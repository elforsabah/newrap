  METHOD precheck_assignworkarea.

    " Lock in BMS status BEGONNEN (07_Change_Lock): a started service is not changed
    DATA(lt_bms_started) = zcl_wr_pd_tour_helper=>get_bms_started_services(
      VALUE #( FOR ls_k IN keys ( ls_k-serviceuuid ) ) ).

    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ls_key>).

      READ TABLE lt_bms_started ASSIGNING FIELD-SYMBOL(<ls_started>)
        WITH TABLE KEY service_uuid = <ls_key>-serviceuuid.
      IF sy-subrc <> 0.
        CONTINUE.
      ENDIF.

      APPEND VALUE #( %tky        = <ls_key>-%tky
                      %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
      APPEND VALUE #( %tky = <ls_key>-%tky
                      %msg = new_message(
                               id       = 'Z_MSG_SVR_TOUR_EXT'
                               number   = '058'
                               severity = if_abap_behv_message=>severity-error
                               v1       = condense( |{ <ls_started>-reference_id ALPHA = OUT }| ) )
                    ) TO reported-service.

    ENDLOOP.

  ENDMETHOD.


    METHOD precheck_terminateservice.

    " Lock in BMS status BEGONNEN (07_Change_Lock): a started service is not cancelled from SAP
    DATA(lt_bms_started) = zcl_wr_pd_tour_helper=>get_bms_started_services(
      VALUE #( FOR ls_k IN keys ( ls_k-serviceuuid ) ) ).

    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ls_key>).

      READ TABLE lt_bms_started ASSIGNING FIELD-SYMBOL(<ls_started>)
        WITH TABLE KEY service_uuid = <ls_key>-serviceuuid.
      IF sy-subrc <> 0.
        CONTINUE.
      ENDIF.

      APPEND VALUE #( %tky        = <ls_key>-%tky
                      %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
      APPEND VALUE #( %tky = <ls_key>-%tky
                      %msg = new_message(
                               id       = 'Z_MSG_SVR_TOUR_EXT'
                               number   = '058'
                               severity = if_abap_behv_message=>severity-error
                               v1       = condense( |{ <ls_started>-reference_id ALPHA = OUT }| ) )
                    ) TO reported-service.

    ENDLOOP.

  ENDMETHOD.


    METHOD precheck_adjusttime.

    " Lock in BMS status BEGONNEN (07_Change_Lock): checked before the input checks
    DATA(lt_bms_started) = zcl_wr_pd_tour_helper=>get_bms_started_services(
      VALUE #( FOR ls_k IN keys ( ls_k-serviceuuid ) ) ).

    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ls_key>).

      READ TABLE lt_bms_started ASSIGNING FIELD-SYMBOL(<ls_started>)
        WITH TABLE KEY service_uuid = <ls_key>-serviceuuid.
      IF sy-subrc = 0.
        APPEND VALUE #( %tky        = <ls_key>-%tky
                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
        APPEND VALUE #( %tky = <ls_key>-%tky
                        %msg = new_message(
                                 id       = 'Z_MSG_SVR_TOUR_EXT'
                                 number   = '058'
                                 severity = if_abap_behv_message=>severity-error
                                 v1       = condense( |{ <ls_started>-reference_id ALPHA = OUT }| ) )
                      ) TO reported-service.
        CONTINUE.
      ENDIF.

      " ---------------------------------------------------------------------
      " 1. Parse Input & Format Validation
      " ---------------------------------------------------------------------
      DATA(lv_input_str) = CONV string( <ls_key>-%param-zeitinmin ).
      REPLACE ALL OCCURRENCES OF ',' IN lv_input_str WITH '.'.
      CONDENSE lv_input_str NO-GAPS.

      " Regex: Allow integers or decimals, optional minus sign
      IF lv_input_str IS NOT INITIAL.
        IF NOT matches( val = lv_input_str regex = '^-?[0-9]+([.,][0-9]+)?$' ).
          APPEND VALUE #( %tky = <ls_key>-%tky
                          %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
          APPEND VALUE #( %tky = <ls_key>-%tky
                          %msg = new_message(
                                   id       = 'Z_MSG_SVR_TOUR_EXT'
                                   number   = '005'
                                   severity = if_abap_behv_message=>severity-error )
                        ) TO reported-service.
          CONTINUE.
        ENDIF.
      ENDIF.

      " Convert to Number for calculations
      DATA lv_adj_min  TYPE decfloat34.
      TRY.
          lv_adj_min = lv_input_str.
        CATCH cx_sy_conversion_no_number.
          CONTINUE.
      ENDTRY.

      " ---------------------------------------------------------------------
      " 2. Limit Check: Max +/- 600 Minutes (10 Hours)
      " ---------------------------------------------------------------------
      IF lv_adj_min > 600 OR lv_adj_min < -600.
        APPEND VALUE #( %tky = <ls_key>-%tky
                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
        APPEND VALUE #( %tky = <ls_key>-%tky
                        %msg = new_message(
                                 id       = 'Z_MSG_SVR_TOUR_EXT'
                                 number   = '006'
                                 severity = if_abap_behv_message=>severity-error )
                      ) TO reported-service.
        CONTINUE.
      ENDIF.
    ENDLOOP.

  ENDMETHOD.


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

    " Lock in BMS status BEGONNEN (07_Change_Lock): one read for all selected services
    DATA(lt_bms_started) = zcl_wr_pd_tour_helper=>get_bms_started_services(
      VALUE #( FOR ls_s IN lt_services ( ls_s-serviceuuid ) ) ).

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

      " ---------------------------------------------------------------------
      " Validation 0: started in BMS → no change at all
      " ---------------------------------------------------------------------
      READ TABLE lt_bms_started ASSIGNING FIELD-SYMBOL(<ls_started>)
        WITH TABLE KEY service_uuid = <ls_service>-serviceuuid.
      IF sy-subrc = 0.
        APPEND VALUE #( %tky        = <ls_service>-%tky
                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %msg = new_message(
                                 id       = 'Z_MSG_SVR_TOUR_EXT'
                                 number   = '058'
                                 severity = if_abap_behv_message=>severity-error
                                 v1       = condense( |{ <ls_started>-reference_id ALPHA = OUT }| ) )
                      ) TO reported-service.
        CONTINUE.
      ENDIF.

      " Get the parameter for THIS service
      DATA(ls_param) = keys[ %tky = <ls_service>-%tky ]-%param.
