*======================================================================*
* /PLCE/R_PDService — behavior pool, local class LHC_SERVICE
* METHOD precheck_anlageaendern
* Version 2 (2026-11-09): Schein check per ticket (check 1)
*   An EAP whose Schein hangs on a Nachweis that takes part in eANV
*   keeps its disposal facility. New validation 2, right after the
*   framework-agreement check, before the target ESA is looked at.
*   Data comes from ZCL_WR_PD_TOUR_HELPER=>GET_EANV_NOTES (one mass
*   call before the loop, lookup by POBJNR inside). No signature check.
*   One message per blocked EAP, Z_MSG_SVR_TOUR_EXT 057 (new, see README),
*   the IC wording the ticket author kept, 73 characters:
*     EAP &1 mit Schein &2: ESA &3 stimmt nicht mit Entsorger des NW &4 überein
*     &1 = EA-ID, &2 = external Schein number, &3 = chosen ESA,
*     &4 = external Nachweis number (TARBNW-NWNR)
*   Old validations 2–4 are now 3–5, otherwise unchanged.
* Version 1 (2026-11-04): signature check via CHECK_NOTE_SIGNATURES —
*   withdrawn, it blocked on missing signatures and on "no Schein".
* Replace the whole METHOD ... ENDMETHOD.
*======================================================================*
  METHOD precheck_anlageaendern.

    TYPES:
      BEGIN OF ty_pobj,
        pobjnr TYPE j_objnr,
      END OF ty_pobj,
      tt_pobj TYPE SORTED TABLE OF ty_pobj WITH UNIQUE KEY pobjnr.

    DATA:
      lt_pobj       TYPE tt_pobj,
      lt_orders     TYPE zcl_wr_pd_tour_helper=>tt_sig_orders,
      lt_eanv_notes TYPE zcl_wr_pd_tour_helper=>tt_eanv_notes,
      ls_eanv_note  TYPE zcl_wr_pd_tour_helper=>ty_eanv_note,
      lv_pobjnr     TYPE j_objnr.

    " Read ALL services at once
    READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY service
      FIELDS ( serviceuuid referenceinternalid referenceid )
      WITH CORRESPONDING #( keys )
      RESULT DATA(lt_services).

    " ---------------------------------------------------------------------
    " Scheine on an eANV Nachweis — one mass read for all selected EAPs.
    " POBJNR is CHAR 22, REFERENCE_INTERNAL_ID is CHAR 30 and FOR ALL
    " ENTRIES needs identical type and length, hence the key table.
    " NOTEINTNR comes from the DB, so it is already in the internal
    " (ALPHA) format the helper expects.
    " ---------------------------------------------------------------------
    LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_key_svc>).
      IF <ls_key_svc>-referenceinternalid IS NOT INITIAL.
        INSERT VALUE #( pobjnr = <ls_key_svc>-referenceinternalid )
          INTO TABLE lt_pobj.
      ENDIF.
    ENDLOOP.

    IF lt_pobj IS NOT INITIAL.
      SELECT pobjnr,
             /watp/noteintnr AS noteintnr
        FROM ewa_order_object
        FOR ALL ENTRIES IN @lt_pobj
        WHERE pobjnr = @lt_pobj-pobjnr
        INTO TABLE @lt_orders.

      IF lt_orders IS NOT INITIAL.
        lt_eanv_notes = zcl_wr_pd_tour_helper=>get_eanv_notes( lt_orders ).
      ENDIF.
    ENDIF.

    " Loop through SERVICES (not keys!)
    LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_service>).

      CLEAR: ls_eanv_note, lv_pobjnr.

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
      " Validation 2: EAP has a Schein on an eANV Nachweis → no ESA change
      "   Ticket check 1. Only Scheine whose Nachweis takes part in eANV
      "   count (TARBNW-DIGI_SIG, same rule as #42128). An EAP without
      "   Schein, or with a Schein on a paper Nachweis, goes on to the
      "   ESA checks and the AVV check (ticket check 2).
      " ---------------------------------------------------------------------
      lv_pobjnr = <ls_service>-referenceinternalid.

      READ TABLE lt_eanv_notes INTO ls_eanv_note
        WITH TABLE KEY pobjnr = lv_pobjnr.

      IF sy-subrc = 0.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %msg = new_message(
                                 id       = 'Z_MSG_SVR_TOUR_EXT'
                                 number   = '057'
                                 severity = if_abap_behv_message=>severity-error
                                 v1       = condense( |{ <ls_service>-referenceid ALPHA = OUT }| )
                                 v2       = ls_eanv_note-notenr
                                 v3       = CONV #( ls_param-anlage )
                                 v4       = ls_eanv_note-nwnr )
                      ) TO reported-service.
        CONTINUE.
      ENDIF.

      " ---------------------------------------------------------------------
      " Validation 3: Check if facility is selected
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
      " Validation 4: Check facility exists and is available
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
