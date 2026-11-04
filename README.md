* METHOD precheck_anlageaendern
* Version 1 (2026-11-04): + electronic signature check (validation 4)
*   via ZCL_WR_PD_TOUR_HELPER=>CHECK_NOTE_SIGNATURES, the same rule and
*   the same messages as "Tour an BMS freigeben" (Z_MSG_SVR_TOUR_EXT
*   042, 047–053; &1 = EA-ID, &2 = Scheinnummer, &3 = Scheinart).
*   The helper is called ONCE for all selected EAPs before the loop;
*   inside the loop every service is only looked up by POBJNR.
*   Validations 1–3 unchanged, the AVV check is now validation 5.
* Replace the whole METHOD ... ENDMETHOD.
*======================================================================*
  METHOD precheck_anlageaendern.

    TYPES:
      BEGIN OF ty_pobj,
        pobjnr TYPE j_objnr,
      END OF ty_pobj,
      tt_pobj TYPE SORTED TABLE OF ty_pobj WITH UNIQUE KEY pobjnr.

    DATA:
      lt_pobj        TYPE tt_pobj,
      lt_sig_orders  TYPE zcl_wr_pd_tour_helper=>tt_sig_orders,
      lt_sig_missing TYPE zcl_wr_pd_tour_helper=>tt_sig_missing,
      ls_sig_missing TYPE zcl_wr_pd_tour_helper=>ty_sig_missing,
      lv_pobjnr      TYPE j_objnr.

    " Read ALL services at once
    READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY service
      FIELDS ( serviceuuid referenceinternalid referenceid )
      WITH CORRESPONDING #( keys )
      RESULT DATA(lt_services).

    " ---------------------------------------------------------------------
    " Electronic signatures (UST-S.AA.0006-005) — one mass check for all
    " selected EAPs, same helper as "Tour an BMS freigeben".
    " POBJNR is CHAR 22, REFERENCE_INTERNAL_ID is CHAR 30 and FOR ALL
    " ENTRIES needs identical type and length, hence the key table.
    " NOTEINTNR comes from the DB, so it is already in the internal
    " (ALPHA) format the helper expects.
    " ---------------------------------------------------------------------
    LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_sig_svc>).
      IF <ls_sig_svc>-referenceinternalid IS NOT INITIAL.
        INSERT VALUE #( pobjnr = <ls_sig_svc>-referenceinternalid )
          INTO TABLE lt_pobj.
      ENDIF.
    ENDLOOP.

    IF lt_pobj IS NOT INITIAL.
      SELECT pobjnr,
             /watp/noteintnr AS noteintnr
        FROM ewa_order_object
        FOR ALL ENTRIES IN @lt_pobj
        WHERE pobjnr = @lt_pobj-pobjnr
        INTO TABLE @lt_sig_orders.

      IF lt_sig_orders IS NOT INITIAL.
        lt_sig_missing = zcl_wr_pd_tour_helper=>check_note_signatures( lt_sig_orders ).
      ENDIF.
    ENDIF.

    " Loop through SERVICES (not keys!)
    LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_service>).

      CLEAR: ls_sig_missing, lv_pobjnr.

      " Get the parameter for THIS service
      DATA(ls_param) = keys[ %tky = <ls_service>-%tky ]-%param.

      " Get order details
      SELECT SINGLE *
        FROM ewa_order_object
        WHERE pobjnr = @<ls_service>-referenceinternalid
        INTO @DATA(ls_order).

      IF sy-subrc <> 0.
        CONTINUE.
      ENDIF.

      " ---------------------------------------------------------------------
      " Validation 1: Check for Fixed Disposal Contract
      " ---------------------------------------------------------------------
      IF ls_order-wdplantnr IS NOT INITIAL.
        SELECT SINGLE zz_framework_agreement
          FROM ewa_el_wdplant
          WHERE wdplantnr = @ls_order-wdplantnr
          INTO @DATA(lv_existing_contract).

        IF sy-subrc = 0 AND lv_existing_contract IS NOT INITIAL.
          APPEND VALUE #( %tky = <ls_service>-%tky
                          %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
          APPEND VALUE #( %tky = <ls_service>-%tky
                          %msg = new_message(
                                   id       = 'Z_MSG_SVR_TOUR_EXT'
                                   number   = '010'
                                   severity = if_abap_behv_message=>severity-error
                                   )
                        ) TO reported-service.
          CONTINUE.
        ENDIF.
      ENDIF.

      " ---------------------------------------------------------------------
      " Validation 2: Check if facility is selected
      " ---------------------------------------------------------------------
      IF ls_param-anlage IS INITIAL.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %msg = new_message(
                                 id       = 'Z_MSG_SVR_TOUR_EXT'
                                 number   = '011'
                                 severity = if_abap_behv_message=>severity-error
                                  )
                      ) TO reported-service.
*        CONTINUE.
      ENDIF.

      " ---------------------------------------------------------------------
      " Validation 3: Check facility exists and is available
      " ---------------------------------------------------------------------
      SELECT SINGLE wdplantnr, zz_available_dispo, zz_framework_agreement
        FROM ewa_el_wdplant
        WHERE wdplantnr = @ls_param-anlage
        INTO @DATA(ls_plant).

      IF sy-subrc <> 0.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %msg = new_message(
                                 id       = 'Z_MSG_SVR_TOUR_EXT'
                                 number   = '012'
                                 severity = if_abap_behv_message=>severity-error
                                 v1       = CONV #( ls_param-anlage ) )
                      ) TO reported-service.
        CONTINUE.
      ENDIF.

      IF ls_plant-zz_available_dispo IS INITIAL OR ls_plant-zz_available_dispo <> 'X'.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %msg = new_message(
                                 id       = 'Z_MSG_SVR_TOUR_EXT'
                                 number   = '013'
                                 severity = if_abap_behv_message=>severity-error
                                 v1       = CONV #( ls_param-anlage ) )
                      ) TO reported-service.
        CONTINUE.
      ENDIF.

      " ---------------------------------------------------------------------
      " Validation 4: Electronic signature (UST-S.AA.0006-005)
      "   Every EAP the helper returns must not have its facility changed.
      "   The helper delivers the message number and &2/&3; &1 = EA-ID is
      "   added here, exactly as in "Tour an BMS freigeben".
      "   NOTE: the helper also returns EAPs WITHOUT a Schein (message 047).
      "   If an EAP without Schein is supposed to pass on to the AVV check
      "   instead, add to the IF below:
      "     AND ls_sig_missing-msgno <> zcl_wr_pd_tour_helper=>gc_sig_msg-no_note
      " ---------------------------------------------------------------------
      lv_pobjnr = <ls_service>-referenceinternalid.

      READ TABLE lt_sig_missing INTO ls_sig_missing
        WITH TABLE KEY pobjnr = lv_pobjnr.

      IF sy-subrc = 0.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %msg = new_message(
                                 id       = 'Z_MSG_SVR_TOUR_EXT'
                                 number   = ls_sig_missing-msgno
                                 severity = if_abap_behv_message=>severity-error
                                 v1       = condense( |{ <ls_service>-referenceid ALPHA = OUT }| )
                                 v2       = ls_sig_missing-v2
                                 v3       = ls_sig_missing-v3 )
                      ) TO reported-service.
        CONTINUE.
      ENDIF.

      " ---------------------------------------------------------------------
      " Validation 5: AVV-Code matching
      " ---------------------------------------------------------------------
      SELECT COUNT(*)
        FROM /watp/twdplanavv
        WHERE wdplantnr = @ls_param-anlage
        INTO @DATA(lv_avv_count).

      IF lv_avv_count > 0.
        " AVV assignments exist - must validate
        SELECT SINGLE /watp/avvcode
          FROM ewa_order_object
          WHERE pobjnr = @<ls_service>-referenceinternalid
          INTO @DATA(lv_order_avvcode).

        IF sy-subrc = 0 AND lv_order_avvcode IS NOT INITIAL.
          SELECT SINGLE avvcode
            FROM /watp/twdplanavv
            WHERE wdplantnr = @ls_param-anlage
              AND avvcode = @lv_order_avvcode
            INTO @DATA(lv_found_avv).

          IF sy-subrc <> 0.
            APPEND VALUE #( %tky = <ls_service>-%tky
                            %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
            APPEND VALUE #( %tky = <ls_service>-%tky
                            %msg = new_message(
                                     id       = 'Z_MSG_SVR_TOUR_EXT'
                                     number   = '014'
                                     severity = if_abap_behv_message=>severity-error
                                     v1       = CONV #( lv_order_avvcode ) )
                          ) TO reported-service.
            CONTINUE.
          ENDIF.
        ENDIF.
      ENDIF.

    ENDLOOP.
  ENDMETHOD.
