In Aufgabe #40611 haben wir entwickelt, dass die Entsorgungsanlage (ESA) im P&D pro EAP geändert werden kann.  Es sind nur die ESAs für einen Wechsel zur Verfügung, die einen Haken in /EWAEL04 > “Zusatz” bei “Verfügbar für Dispo” haben. Details siehe #40611 .

In der Schätzaufgabe #39730 wurde bereits erwähnt “ Auftrag mit Nachweis/Schein (das Feld gibt es aktuell noch nicht in P&D → die Einschränkung muss später ergänzt werden → erstmal nur Platzhalter im Coding)” . Mit dieser Aufgabe soll dies nachgebessert werden.

Wenn eine ESA geändert wird sollen folgende Prüfungen greifen:

Falls die EAP einen Schein (BS/ÜS) besitzt, darf die ESA nicht geändert werden. Der ähnliche Fehlermeldetext wie im IC soll auftauchen:
image_20260930103003.png
2. Falls die EAP keinen Schein besitzt, muss geprüft werden, ob der AVV-Code des Auftrages zu den erlaubten Abfällen an der ESA passt ( /EWAEL04 > Allgemein > Abfall) (Beachte: Wenn keine Einträge bei Abfall sind, kann man ALLE Abfälle an die ESA bringen wie zB bei ESA "PROLOGA" in TI4M442). 

Wenn eine ESA ausgewählt wird, wo der AVV nicht passt, dann sollte der Fehlermeldungstext kommen “Abfall (X, Y) des Auftrages passt nicht zur Entsorgungsanlage (Z) !” X = AVV-Code , Y = Material , Z=ausgewählte, nicht zulässige ESA

Zusatz: Insofern möglich (und im Rahmen dieser Entwicklung kein erheblicher Mehraufwand), soll das Feld Scheinart & -nummer bei der EAP im P&D angezeigt werden. 

Ich bitte um grobe Schätzung der Prüfschritte 1&2 und on top dem “Zusatz”. Ein Kommentar auf diese Aufgabe reicht aus.



METHOD anlageaendern.

  " =========================================================================
  " Declare ALL variables at the top
  " =========================================================================
  DATA: lt_ext_create    TYPE TABLE FOR CREATE /plce/r_pdservice\_extcustom,
        lt_ext_update    TYPE TABLE FOR UPDATE /plce/r_pdservice\\extcustom,
        lt_extwr_update  TYPE TABLE FOR UPDATE /plce/r_pdservice\\ExtWaste,
        lt_tour_uuids    TYPE TABLE OF /plce/r_pdtour-TourUUID,
        lt_tour_keys     TYPE TABLE FOR ACTION IMPORT /plce/r_pdtour~generategeoroute,
        lv_tour_uuid     TYPE /plce/r_pdtour-TourUUID,
        lv_check_tour    TYPE /plce/r_pdtour-TourUUID.

  " =========================================================================
  " Read ALL services at once
  " =========================================================================
  READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
    ENTITY service
    FIELDS ( serviceuuid referenceinternalid referenceid )
    WITH CORRESPONDING #( keys )
    RESULT DATA(lt_services).

  " =========================================================================
  " Loop through SERVICES - prepare ExtCustom + ExtWR create/update
  " =========================================================================
  LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_service>).

    DATA(ls_param) = keys[ %tky = <ls_service>-%tky ]-%param.

    " Get TPLNR from disposal facility
    IF ls_param-anlage IS NOT INITIAL.
      SELECT SINGLE wdplantnr, tplnr
        FROM ewa_el_wdplant
        WHERE wdplantnr = @ls_param-anlage
        INTO @DATA(ls_facility).

      IF sy-subrc = 0 AND ls_facility-tplnr IS NOT INITIAL.
        APPEND VALUE #(
          %key-serviceuuid = <ls_service>-serviceuuid
          plantlocation    = ls_facility-tplnr
        ) TO lt_extwr_update.
      ENDIF.
    ELSE.
      " Clear plant_location if facility is cleared
      APPEND VALUE #(
        %key-serviceuuid = <ls_service>-serviceuuid
        plantlocation    = ''
      ) TO lt_extwr_update.
    ENDIF.

    " Check if ExtCustom exists
    READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY extcustom
      FIELDS ( serviceuuid )
      WITH VALUE #( ( serviceuuid = <ls_service>-serviceuuid ) )
      RESULT DATA(lt_extcustom).

    IF lt_extcustom IS INITIAL.
      DATA(lv_cid) = |$cid_{ <ls_service>-serviceuuid }$|.

      APPEND VALUE #( serviceuuid = <ls_service>-serviceuuid
                      %target = VALUE #( ( %cid = lv_cid ) ) )
        TO lt_ext_create.
    ENDIF.

    APPEND VALUE #( %key-serviceuuid = <ls_service>-serviceuuid
                    wdplantnr        = ls_param-anlage )
      TO lt_ext_update.

    CLEAR: lt_extcustom, ls_facility.

  ENDLOOP.

  " =========================================================================
  " Bulk update ExtWR plant_location
  " =========================================================================
  IF lt_extwr_update IS NOT INITIAL.
    MODIFY ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY ExtWaste
      UPDATE FIELDS ( plantlocation )
      WITH lt_extwr_update
      FAILED DATA(lt_extwr_failed)
      REPORTED DATA(lt_extwr_reported).

    " Success message for plant_location update
    IF lt_extwr_failed IS INITIAL.
      LOOP AT lt_services ASSIGNING <ls_service>.
        DATA(ls_param_extwr) = keys[ %tky = <ls_service>-%tky ]-%param.

        IF ls_param_extwr-anlage IS NOT INITIAL.
          APPEND VALUE #( %tky = <ls_service>-%tky
                          %msg = new_message(
                                   id       = 'Z_MSG_SVR_TOUR_EXT'
                                   number   = '018'
                                   severity = if_abap_behv_message=>severity-information
                                   v1       = <ls_service>-referenceid )
                        ) TO reported-service.
        ENDIF.
      ENDLOOP.
    ENDIF.
  ENDIF.

  " =========================================================================
  " Bulk create missing ExtCustom records
  " =========================================================================
  IF lt_ext_create IS NOT INITIAL.
    MODIFY ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY service
      CREATE BY \_extcustom
      AUTO FILL CID SET FIELDS WITH lt_ext_create
      MAPPED DATA(lmapped)
      FAILED DATA(lfailed)
      REPORTED DATA(lreported).

    IF lfailed IS NOT INITIAL.
      APPEND LINES OF lfailed-extcustom TO failed-extcustom.
      APPEND LINES OF lreported-extcustom TO reported-extcustom.
    ENDIF.
  ENDIF.

  " =========================================================================
  " Bulk update disposal facility
  " =========================================================================
  IF lt_ext_update IS NOT INITIAL.
    MODIFY ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY extcustom
      UPDATE FIELDS ( wdplantnr )
      WITH lt_ext_update
      FAILED DATA(lt_failed_update)
      REPORTED DATA(lt_reported_update).

    IF lt_failed_update IS NOT INITIAL.
      APPEND LINES OF lt_failed_update-extcustom TO failed-extcustom.
      APPEND LINES OF lt_reported_update-extcustom TO reported-extcustom.
    ELSE.
      " ---------------------------------------------------------------------
      " Success messages for wdplantnr
      " ---------------------------------------------------------------------
      LOOP AT lt_services ASSIGNING <ls_service>.
        DATA(ls_param_msg) = keys[ %tky = <ls_service>-%tky ]-%param.

        IF ls_param_msg-anlage IS NOT INITIAL.
          APPEND VALUE #( %tky = <ls_service>-%tky
                          %msg = new_message(
                                   id       = 'Z_MSG_SVR_TOUR_EXT'
                                   number   = '015'
                                   severity = if_abap_behv_message=>severity-information
                                   v1       = <ls_service>-referenceid )
                        ) TO reported-service.
        ELSE.
          APPEND VALUE #( %tky = <ls_service>-%tky
                          %msg = new_message(
                                   id       = 'Z_MSG_SVR_TOUR_EXT'
                                   number   = '016'
                                   severity = if_abap_behv_message=>severity-information
                                   v1       = <ls_service>-referenceid )
                        ) TO reported-service.
        ENDIF.
      ENDLOOP.

      " ---------------------------------------------------------------------
      " Recalculate Tour Route (Spur ermitteln) for assigned tours
      " ---------------------------------------------------------------------
      IF lt_services IS NOT INITIAL .

       if lt_services[ 1 ]-PlanningStatus = 'X'.
        SELECT DISTINCT tour_uuid
          FROM /plce/tpdsrvtsk
          FOR ALL ENTRIES IN @lt_services
          WHERE service_uuid = @lt_services-serviceuuid
            AND tour_uuid IS NOT INITIAL
          INTO TABLE @lt_tour_uuids.

        " Everything tour-related INSIDE this check
        " Guarantees tour code NEVER runs when no tours found
        IF lt_tour_uuids IS NOT INITIAL.

          " Prepare keys for GenerateGeoRoute action
          LOOP AT lt_tour_uuids INTO lv_tour_uuid.
            APPEND VALUE #( %tky-touruuid = lv_tour_uuid ) TO lt_tour_keys.
          ENDLOOP.

          " Call GenerateGeoRoute action
          " Inline DATA() - correct type for EXECUTE action response
          MODIFY ENTITIES OF /plce/r_pdtour
            ENTITY tour
              EXECUTE generategeoroute FROM lt_tour_keys
            FAILED DATA(lt_tour_failed)
            REPORTED DATA(lt_tour_reported).

          " Only reached when tours were actually processed
          IF lt_tour_failed-tour IS INITIAL.
            " Add message ONLY for services that have a tour
            LOOP AT lt_services ASSIGNING <ls_service>.

              SELECT SINGLE tour_uuid
                FROM /plce/tpdsrvtsk
                WHERE service_uuid = @<ls_service>-serviceuuid
                  AND tour_uuid IS NOT INITIAL
                INTO @lv_check_tour.

              IF sy-subrc = 0.
                APPEND VALUE #( %tky = <ls_service>-%tky
                                %msg = new_message(
                                         id       = 'Z_MSG_SVR_TOUR_EXT'
                                         number   = '017'
                                         severity = if_abap_behv_message=>severity-information
                                         v1       = <ls_service>-referenceid )
                              ) TO reported-service.
              ENDIF.

              CLEAR lv_check_tour.
            ENDLOOP.
          ENDIF.

        ENDIF. " lt_tour_uuids IS NOT INITIAL

      ENDIF. " lt_services IS NOT INITIAL
    ENDIF.
    ENDIF.
  ENDIF.

  " =========================================================================
  " Return updated data to UI
  " =========================================================================
  READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
    ENTITY service ALL FIELDS WITH CORRESPONDING #( keys ) RESULT DATA(lt_service_result).

  result = VALUE #( FOR srv IN lt_service_result ( %tky = srv-%tky %param = srv ) ).

ENDMETHOD.




  METHOD precheck_anlageaendern.

    " Read ALL services at once
    READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY service
      FIELDS ( serviceuuid referenceinternalid referenceid )
      WITH CORRESPONDING #( keys )
      RESULT DATA(lt_services).

    " Loop through SERVICES (not keys!)
    LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_service>).

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
      " Validation 4: AVV-Code matching
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

