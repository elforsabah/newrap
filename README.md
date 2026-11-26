    TYPES:
      tt_bms_service_uuid TYPE STANDARD TABLE OF /plce/pdservice_uuid WITH EMPTY KEY,
      BEGIN OF ty_bms_started,
        service_uuid TYPE /plce/pdservice_uuid,
        reference_id TYPE /plce/tpdsrv-reference_id,
      END OF ty_bms_started,
      tt_bms_started TYPE SORTED TABLE OF ty_bms_started WITH UNIQUE KEY service_uuid.

    CONSTANTS gc_bms_status_started TYPE /plce/tpdsrvcst-zz_bms_status VALUE 'BEGONNEN'.

    "! Services of the input that BMS has reported as started (ZZ_BMS_STATUS = BEGONNEN).
    "! Used by the prechecks of the own actions and by the two validations.
    CLASS-METHODS get_bms_started_services
      IMPORTING it_service_uuid   TYPE tt_bms_service_uuid
      RETURNING VALUE(rt_started) TYPE tt_bms_started.


        METHOD get_bms_started_services.

    IF it_service_uuid IS INITIAL.
      RETURN.
    ENDIF.

    SELECT cst~service_uuid, srv~reference_id
      FROM /plce/tpdsrvcst AS cst
      INNER JOIN /plce/tpdsrv AS srv
        ON srv~service_uuid = cst~service_uuid
      FOR ALL ENTRIES IN @it_service_uuid
      WHERE cst~service_uuid  = @it_service_uuid-table_line
        AND cst~zz_bms_status = @gc_bms_status_started
      INTO TABLE @rt_started.

  ENDMETHOD.

  
