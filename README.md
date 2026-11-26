  validation checkBmsNotStarted on save { update; }

  draft determine action Prepare { validation checkBmsNotStarted; }


    METHOD checkbmsnotstarted.

    " Lock in BMS status BEGONNEN (07_Change_Lock 3.2): a started service cannot be saved.
    " Fires only for changes on Service itself, not on ExtCustom, so the status
    " write of ZWR_BMS_UPDATE_SERVICE and PopulateExtCustomFromOrder pass.
    READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY service
      FIELDS ( serviceuuid referenceid )
      WITH CORRESPONDING #( keys )
      RESULT DATA(lt_services).

    DATA(lt_bms_started) = zcl_wr_pd_tour_helper=>get_bms_started_services(
      VALUE #( FOR ls_s IN lt_services ( ls_s-serviceuuid ) ) ).

    LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_service>).

      READ TABLE lt_bms_started ASSIGNING FIELD-SYMBOL(<ls_started>)
        WITH TABLE KEY service_uuid = <ls_service>-serviceuuid.
      IF sy-subrc <> 0.
        CONTINUE.
      ENDIF.

      APPEND VALUE #( %tky = <ls_service>-%tky ) TO failed-service.
      APPEND VALUE #( %tky = <ls_service>-%tky
                      %msg = new_message(
                               id       = 'Z_MSG_SVR_TOUR_EXT'
                               number   = '058'
                               severity = if_abap_behv_message=>severity-error
                               v1       = condense( |{ <ls_started>-reference_id ALPHA = OUT }| ) )
                    ) TO reported-service.

    ENDLOOP.

  ENDMETHOD.
