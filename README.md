<img width="1351" height="425" alt="image" src="https://github.com/user-attachments/assets/b94d229b-2875-4cab-8ded-9202c5ba9d41" />
METHOD touranbmsfreigeben.

*----------------------------------------------------------------------*
* TYPES — mirror of InputOrder (BMS API v1, openapi 3.0.4)
*         additionalProperties: false — every component must exist in
*         the spec and nothing may be added.
*----------------------------------------------------------------------*
  TYPES:
    " InputContact — placeOfDelivery, location
    BEGIN OF ty_contact,
      street        TYPE string,
      street_number TYPE string,
      zip_code      TYPE string,
      city          TYPE string,
    END OF ty_contact,

    " InputContactWithName — recycler (no number)
    BEGIN OF ty_contact_name,
      name1         TYPE string,
      name2         TYPE string,
      street        TYPE string,
      street_number TYPE string,
      zip_code      TYPE string,
      city          TYPE string,
    END OF ty_contact_name,

    " InputContactWithNameAndNumber — customer, producer, carrier
    BEGIN OF ty_contact_name_num,
      number        TYPE string,
      name1         TYPE string,
      name2         TYPE string,
      street        TYPE string,
      street_number TYPE string,
      zip_code      TYPE string,
      city          TYPE string,
    END OF ty_contact_name_num,

    " InputContainer.
    " containerMovementTypeInfo and internalRemark are documented as
    " nullable but are REQUIRED by the running API, and [Required]
    " rejects empty strings — both must always carry content.
    BEGIN OF ty_container,
      container_number_old         TYPE string,
      container_number_new         TYPE string,
      movement_type                TYPE string,   " new | change | collect
      customer_owned               TYPE abap_bool,
      container_type_name          TYPE string,
      container_movement_type_info TYPE string,
      internal_remark              TYPE string,
    END OF ty_container,
    tt_containers TYPE STANDARD TABLE OF ty_container WITH EMPTY KEY,

    BEGIN OF ty_bms_order,
      status                     TYPE string,   " ok | cancelled
      external_system_id         TYPE string,
      sort_number                TYPE i,
      order_number               TYPE string,
      order_sheet                TYPE string,
      order_sheet_type           TYPE string,   " LS | ÜS | BS
      customer                   TYPE ty_contact_name_num,
      place_of_delivery          TYPE ty_contact,
      location                   TYPE ty_contact,
      estimated_duration         TYPE i,
      planned_date               TYPE string,
      execution_time_frame_start TYPE string,   " date-span, HH:MM:SS
      execution_time_frame_end   TYPE string,
      notes                      TYPE string,
      special_notes              TYPE string,
      producer                   TYPE ty_contact_name_num,
      recycler                   TYPE ty_contact_name,
      carrier                    TYPE ty_contact_name_num,
      garbage_key                TYPE string,
      garbage_name               TYPE string,
      " ABAP caps component names at 30 chars — patched after serialisation
      coll_consignment_note_nr   TYPE string,
      team                       TYPE string,
      signature_required         TYPE abap_bool,
      containers                 TYPE tt_containers,
    END OF ty_bms_order,

    BEGIN OF ty_ewaobj,
      pobjnr         TYPE j_objnr,
      ordernr        TYPE eordernr,
      smaufnr        TYPE aufnr,
      ordertxt       TYPE text255,
      servloc        TYPE servloc,
      kunnr          TYPE kunnr,
      kunwe          TYPE kunwe,
      beh_type       TYPE beh_type,
      beh_anzahl     TYPE cont_count,
      container      TYPE behaelter,
      order_type     TYPE eorder_type,
      transporter    TYPE ehs_partner,
      disposer       TYPE ehs_partner,
      intappno       TYPE ewa_ehs_appnoint,
      appno          TYPE ewa_ehs_appno,
      planned_time   TYPE pldtim,
      planned_durt   TYPE servdur,
      old_order_date TYPE eorder_date,
      zz_order_date  TYPE dats,
      watp_avvcode   TYPE /watp/davvcode,
      watp_notenr    TYPE /watp/dnotenr,
      watp_noteintnr TYPE /watp/dnoteintnr,
      sdaufnr        TYPE vbeln_va,
      sdposnr        TYPE posnr,
    END OF ty_ewaobj,
    tt_ewaobj TYPE HASHED TABLE OF ty_ewaobj WITH UNIQUE KEY pobjnr,

    BEGIN OF ty_pobj,
      pobjnr TYPE j_objnr,
    END OF ty_pobj,
    tt_pobj TYPE SORTED TABLE OF ty_pobj WITH UNIQUE KEY pobjnr,

    BEGIN OF ty_service,
      service_uuid              TYPE /plce/pdservice_uuid,
      service_id                TYPE /plce/pdservice_id,
      reference_id              TYPE char30,
      reference_internal_id     TYPE char30,
      action                    TYPE /plce/pdaction,
      requested_date            TYPE /plce/date,
      earliest_date             TYPE /plce/date,
      latest_date               TYPE /plce/date,
      service_window_start_time TYPE /plce/time,
      service_window_end_time   TYPE /plce/time,
      service_window            TYPE /plce/pdservice_window,
      additional_text           TYPE /plce/pdadditional_text,
      functional_location       TYPE char30,
    END OF ty_service,
    tt_services TYPE HASHED TABLE OF ty_service WITH UNIQUE KEY service_uuid,

    " MATERIAL + EWC_CODE feed garbageKey; the container fields feed the
    " container array.
    BEGIN OF ty_srvwr,
      service_uuid          TYPE /plce/pdservice_uuid,
      container_atloc_tidnr TYPE char30,
      container_new_tidnr   TYPE char30,
      containertype_atloc   TYPE char30,
      containertype_new     TYPE char30,
      material              TYPE matnr,
      ewc_code              TYPE char20,
    END OF ty_srvwr,
    tt_srvwr TYPE SORTED TABLE OF ty_srvwr WITH NON-UNIQUE KEY service_uuid,

    BEGIN OF ty_bp_addr,
      partner    TYPE bu_partner,
      addrnumber TYPE ad_addrnum,
      name1      TYPE ad_name1,
      name2      TYPE ad_name2,
      street     TYPE ad_street,
      house_num1 TYPE ad_hsnm1,
      post_code1 TYPE ad_pstcd1,
      city1      TYPE ad_city1,
    END OF ty_bp_addr,
    tt_bp_addr TYPE SORTED TABLE OF ty_bp_addr
               WITH NON-UNIQUE KEY partner addrnumber,

    BEGIN OF ty_bp_key,
      partner TYPE bu_partner,
    END OF ty_bp_key,
    tt_bp_key TYPE SORTED TABLE OF ty_bp_key WITH UNIQUE KEY partner,

    " per tour: how many services it has and how many the signature
    " check blocked (STEP 4c) — decides whether BMS is contacted at all
    BEGIN OF ty_sig_count,
      touruuid TYPE /plce/pdtour_uuid,
      services TYPE i,
      blocked  TYPE i,
    END OF ty_sig_count,
    tt_sig_count TYPE HASHED TABLE OF ty_sig_count WITH UNIQUE KEY touruuid.

*----------------------------------------------------------------------*
* DATA
*----------------------------------------------------------------------*
  DATA:
    lt_services      TYPE tt_services,
    lt_srvwr         TYPE tt_srvwr,
    lt_pobj          TYPE tt_pobj,
    lt_ewaobj        TYPE tt_ewaobj,
    lt_bp_keys       TYPE tt_bp_key,
    lt_bp_addr       TYPE tt_bp_addr,
    lt_containers    TYPE tt_containers,
    ls_svc           TYPE ty_service,
    ls_ewa           TYPE ty_ewaobj,
    ls_wr            TYPE ty_srvwr,
    ls_cust_bp       TYPE ty_bp_addr,
    ls_kunwe_bp      TYPE ty_bp_addr,
    ls_carrier_bp    TYPE ty_bp_addr,
    ls_recycler_bp   TYPE ty_bp_addr,
    lt_upd_return    TYPE bapiret2_t,
    lv_rfc_msg       TYPE char60,
    lt_sig_missing   TYPE zcl_wr_pd_tour_helper=>tt_sig_missing,
    ls_sig_missing   TYPE zcl_wr_pd_tour_helper=>ty_sig_missing,
    lt_sig_count     TYPE tt_sig_count,
    ls_sig_count     TYPE ty_sig_count,
    lv_cfg_error     TYPE string,
    lv_dest          TYPE rfcdest,
    lv_team          TYPE string,
    lv_pobjnr        TYPE j_objnr,
    lv_bearer_token  TYPE string,
    lv_auth_error    TYPE string,
    lv_movement_type TYPE string,
    lv_ctype_name    TYPE string,
    lv_cont_old      TYPE string,
    lv_cont_new      TYPE string,
    lv_material_desc TYPE maktx,
    lv_http_status   TYPE i,
    lv_response      TYPE string,
    lv_json          TYPE string,
    lv_upd_subrc     TYPE sy-subrc,
    lv_count_ok      TYPE i,
    lv_count_error   TYPE i.

  " ErrorResponse: { "error": { "message": ..., "timestamp": ... } }
  DATA: BEGIN OF ls_err_body,
          BEGIN OF error,
            message   TYPE string,
            timestamp TYPE string,
          END OF error,
        END OF ls_err_body.

*----------------------------------------------------------------------*
* Order of events (since 23.09.2026):
*   STEP 1–4   read tours, services, orders        — no BMS involved
*   STEP 4b/4c signature check and its messages     — no BMS involved
*   CONFIG     ZTOUR_BMS_CFG                        — first BMS-related check
*   STEP 5–10  addresses, login, send, status
* The dispatcher therefore sees every signature problem even when the
* configuration is broken or the BMS is down, and a tour with nothing
* left to send never contacts the BMS.
*----------------------------------------------------------------------*

*----------------------------------------------------------------------*
* STEP 1 — tours
*----------------------------------------------------------------------*
  READ ENTITIES OF /plce/r_pdtour IN LOCAL MODE
    ENTITY tour
      FIELDS ( tourid tourtemplate tourstatus startdate )
      WITH CORRESPONDING #( keys )
    RESULT DATA(lt_tours)
    FAILED DATA(lt_failed).

  LOOP AT lt_failed-tour ASSIGNING FIELD-SYMBOL(<fail>).
    APPEND VALUE #(
      %tky = <fail>-%tky
      %msg = new_message(
               id       = 'Z_MSG_SVR_TOUR_EXT'
               number   = '024'
               severity = if_abap_behv_message=>severity-error )
    ) TO reported-tour.
    APPEND VALUE #( %tky = <fail>-%tky ) TO failed-tour.
  ENDLOOP.

  IF lt_tours IS INITIAL.
    RETURN.
  ENDIF.

*----------------------------------------------------------------------*
* STEP 2 — service assignments
*----------------------------------------------------------------------*
  READ ENTITIES OF /plce/r_pdtour IN LOCAL MODE
    ENTITY tour BY \_serviceassignments
      FIELDS ( touruuid serviceuuid toursequence removed )
      WITH CORRESPONDING #( keys )
    RESULT DATA(lt_asgmts).

  DELETE lt_asgmts WHERE removed IS NOT INITIAL.
  SORT lt_asgmts BY touruuid ASCENDING toursequence ASCENDING.

  IF lt_asgmts IS INITIAL.
    RETURN.
  ENDIF.

*----------------------------------------------------------------------*
* STEP 3 — services and waste extension
*----------------------------------------------------------------------*
  SELECT s~service_uuid,
         s~service_id,
         s~reference_id,
         s~reference_int_id     AS reference_internal_id,
         s~action,
         s~requested_date,
         s~earliest_date,
         s~latest_date,
         s~service_window_start AS service_window_start_time,
         s~service_window_end   AS service_window_end_time,
         s~service_window,
         s~additional_text,
         s~functional_location
    FROM /plce/tpdsrv AS s
    FOR ALL ENTRIES IN @lt_asgmts
    WHERE s~service_uuid = @lt_asgmts-serviceuuid
    INTO CORRESPONDING FIELDS OF TABLE @lt_services.

  SELECT w~service_uuid,
         w~container_atloc_tidnr,
         w~container_new_tidnr,
         w~containertype_atloc,
         w~containertype_new,
         w~material,
         w~ewc_code
    FROM /plce/tpdsrvwr AS w
    FOR ALL ENTRIES IN @lt_asgmts
    WHERE w~service_uuid = @lt_asgmts-serviceuuid
    INTO CORRESPONDING FIELDS OF TABLE @lt_srvwr.

*----------------------------------------------------------------------*
* STEP 4 — EWA order objects
*          POBJNR is CHAR 22, REFERENCE_INTERNAL_ID is CHAR 30 and
*          FOR ALL ENTRIES needs identical type and length.
*----------------------------------------------------------------------*
  LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<s>).
    IF <s>-reference_internal_id IS NOT INITIAL.
      INSERT VALUE #( pobjnr = <s>-reference_internal_id ) INTO TABLE lt_pobj.
    ENDIF.
  ENDLOOP.

  IF lt_pobj IS NOT INITIAL.
    SELECT pobjnr, ordernr, smaufnr, ordertxt, servloc,
           kunnr, kunwe, beh_type, beh_anzahl, container, order_type,
           transporter, disposer, intappno, appno,
           planned_time, planned_durt, old_order_date, zz_order_date,
           /watp/avvcode   AS watp_avvcode,
           /watp/notenr    AS watp_notenr,
           /watp/noteintnr AS watp_noteintnr,
           sdaufnr, sdposnr
      FROM ewa_order_object
      FOR ALL ENTRIES IN @lt_pobj
      WHERE pobjnr = @lt_pobj-pobjnr
      INTO CORRESPONDING FIELDS OF TABLE @lt_ewaobj.
  ENDIF.


*----------------------------------------------------------------------*
* STEP 4b — electronic signatures (UST-S.AA.0006-005)
*           One mass check for all EAPs of all selected tours. Only
*           EAPs that must NOT go out come back; the service loop does
*           one READ per service. NOTEINTNR comes from the DB, so it is
*           already in the internal (ALPHA) format the helper expects.
*----------------------------------------------------------------------*
  IF lt_ewaobj IS NOT INITIAL.
    lt_sig_missing = zcl_wr_pd_tour_helper=>check_note_signatures(
      VALUE #( FOR <eo> IN lt_ewaobj
               ( pobjnr    = <eo>-pobjnr
                 noteintnr = <eo>-watp_noteintnr ) ) ).
  ENDIF.

*----------------------------------------------------------------------*
* STEP 4c — report the signature results now, per tour, BEFORE the
*           configuration check and the BMS login. Every blocked service
*           gets its message here (Z_MSG_SVR_TOUR_EXT 042, 047–053,
*           &1 = EA-ID, &2/&3 from the helper) and is counted per tour.
*           The service loop later skips these services silently.
*----------------------------------------------------------------------*
  LOOP AT lt_tours INTO DATA(ls_sig_tour).

    CLEAR ls_sig_count.
    ls_sig_count-touruuid = ls_sig_tour-touruuid.

    LOOP AT lt_asgmts INTO DATA(ls_sig_asgmt)
      WHERE touruuid = ls_sig_tour-touruuid.

      ls_sig_count-services = ls_sig_count-services + 1.

      READ TABLE lt_services INTO DATA(ls_sig_svc)
        WITH TABLE KEY service_uuid = ls_sig_asgmt-serviceuuid.
      IF sy-subrc <> 0 OR ls_sig_svc-reference_internal_id IS INITIAL.
        CONTINUE.                 " reported as 025 / 029 in the service loop
      ENDIF.

      lv_pobjnr = ls_sig_svc-reference_internal_id.

      READ TABLE lt_sig_missing INTO ls_sig_missing
        WITH TABLE KEY pobjnr = lv_pobjnr.
      IF sy-subrc <> 0.
        CONTINUE.                 " signature check passed or not applicable
      ENDIF.

      ls_sig_count-blocked = ls_sig_count-blocked + 1.

      APPEND VALUE #(
        %tky = ls_sig_tour-%tky
        %msg = new_message(
                 id       = 'Z_MSG_SVR_TOUR_EXT'
                 number   = ls_sig_missing-msgno
                 severity = if_abap_behv_message=>severity-error
                 v1       = condense( |{ ls_sig_svc-reference_id ALPHA = OUT }| )
                 v2       = ls_sig_missing-v2
                 v3       = ls_sig_missing-v3 )
      ) TO reported-tour.

    ENDLOOP.

    INSERT ls_sig_count INTO TABLE lt_sig_count.
  ENDLOOP.

  CLEAR: lv_pobjnr, ls_sig_missing.

*----------------------------------------------------------------------*
* CONFIG — endpoint comes from an SM59 destination.
*          First point where BMS is involved. The conditions are reported
*          separately: a bare "not configured" message tells the
*          dispatcher nothing. Signature messages are already reported.
*----------------------------------------------------------------------*
  SELECT SINGLE bms_destination, bms_username, bms_password,
                active, default_carrier, default_recycler
    FROM ztour_bms_cfg
    WHERE config_id = 'DEFAULT'
    INTO @DATA(ls_cfg).

  IF sy-subrc <> 0.
    lv_cfg_error = 'kein Eintrag CONFIG_ID = DEFAULT in ZTOUR_BMS_CFG'.
  ELSEIF ls_cfg-active IS INITIAL.
    lv_cfg_error = 'Kennzeichen ACTIVE ist nicht gesetzt'.
  ELSEIF ls_cfg-bms_destination IS INITIAL.
    lv_cfg_error = 'BMS_DESTINATION ist nicht gepflegt'.
  ELSEIF ls_cfg-bms_username IS INITIAL.
    lv_cfg_error = 'BMS_USERNAME ist nicht gepflegt'.
  ENDIF.

  IF lv_cfg_error IS NOT INITIAL.
    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ky>).
      APPEND VALUE #(
        %tky = <ky>-%tky
        %msg = new_message(
                 id       = 'Z_MSG_SVR_TOUR_EXT'
                 number   = '020'
                 severity = if_abap_behv_message=>severity-error
                 v1       = lv_cfg_error )
      ) TO reported-tour.
      APPEND VALUE #( %tky = <ky>-%tky ) TO failed-tour.
    ENDLOOP.
    RETURN.
  ENDIF.

  " typed local — the DDIC field may not be type-compatible with RFCDEST
  lv_dest = ls_cfg-bms_destination.

  DATA(lv_bms_user)     = ls_cfg-bms_username.
  DATA(lv_bms_password) = ls_cfg-bms_password.
  DATA(lv_def_carrier)  = ls_cfg-default_carrier.
  DATA(lv_def_recycler) = ls_cfg-default_recycler.

*----------------------------------------------------------------------*
* STEP 5 — business partner addresses
*          Validity dates live in ADRC, not BUT020.
*----------------------------------------------------------------------*
  LOOP AT lt_ewaobj ASSIGNING FIELD-SYMBOL(<e>).
    INSERT VALUE #( partner = <e>-kunnr )       INTO TABLE lt_bp_keys.
    INSERT VALUE #( partner = <e>-kunwe )       INTO TABLE lt_bp_keys.
    INSERT VALUE #( partner = <e>-transporter ) INTO TABLE lt_bp_keys.
    INSERT VALUE #( partner = <e>-disposer )    INTO TABLE lt_bp_keys.
  ENDLOOP.

  INSERT VALUE #( partner = lv_def_carrier )  INTO TABLE lt_bp_keys.
  INSERT VALUE #( partner = lv_def_recycler ) INTO TABLE lt_bp_keys.
  DELETE lt_bp_keys WHERE partner IS INITIAL.

  IF lt_bp_keys IS NOT INITIAL.
    SELECT ba~partner, ba~addrnumber,
           adr~name1, adr~name2, adr~street,
           adr~house_num1, adr~post_code1, adr~city1
      FROM but020 AS ba
      INNER JOIN adrc AS adr ON  adr~addrnumber = ba~addrnumber
                             AND adr~nation     = @space
      FOR ALL ENTRIES IN @lt_bp_keys
      WHERE ba~partner     = @lt_bp_keys-partner
        AND adr~date_from <= @sy-datum
        AND adr~date_to   >= @sy-datum
      INTO CORRESPONDING FIELDS OF TABLE @lt_bp_addr.
  ENDIF.

*----------------------------------------------------------------------*
* STEP 6 — main loop
*----------------------------------------------------------------------*
  LOOP AT lt_tours INTO DATA(ls_tour).

    CLEAR: lv_count_ok, lv_count_error.

    DATA(lv_tour_id_out) = condense( |{ ls_tour-tourid ALPHA = OUT }| ).

    " --- 6a0 Signature results of STEP 4c ---------------------------------
    "     Blocked services already carry their message and count as
    "     errors of this tour. When every service is blocked there is
    "     nothing to send: the tour fails here and BMS is not contacted.
    READ TABLE lt_sig_count INTO ls_sig_count
      WITH TABLE KEY touruuid = ls_tour-touruuid.

    IF sy-subrc = 0.
      lv_count_error = ls_sig_count-blocked.
      IF ls_sig_count-services > 0
         AND ls_sig_count-blocked = ls_sig_count-services.
        APPEND VALUE #( %tky = ls_tour-%tky ) TO failed-tour.
        CONTINUE.
      ENDIF.
    ENDIF.

    " --- 6a Kolonne -------------------------------------------------------
    " The BMS Kolonne IS the tour template. ZTOUR_BMS_KOLONN is only an
    " override for templates whose name in BMS differs — an empty table
    " means "Kolonne = Tourvorlage", so there is nothing to warn about.
    CLEAR lv_team.

    SELECT SINGLE bms_team
      FROM ztour_bms_kolonn
      WHERE tour_template = @ls_tour-tourtemplate
      INTO @lv_team.

    IF lv_team IS INITIAL.
      lv_team = ls_tour-tourtemplate.
    ENDIF.

    lv_team = condense( lv_team ).

    " --- 6b Authentication ------------------------------------------------
    CLEAR: lv_bearer_token, lv_auth_error.

    zcl_wr_pd_tour_helper=>get_bms_bearer_token(
      EXPORTING
        iv_destination = lv_dest
        iv_username    = lv_bms_user
        iv_password    = lv_bms_password
      IMPORTING
        ev_token       = lv_bearer_token
        ev_error       = lv_auth_error ).

    IF lv_auth_error IS NOT INITIAL.
      APPEND VALUE #(
        %tky = ls_tour-%tky
        %msg = new_message_with_text(
                 severity = if_abap_behv_message=>severity-error
                 text     = lv_auth_error )
      ) TO reported-tour.

      MODIFY ENTITIES OF /plce/r_pdtour IN LOCAL MODE
        ENTITY extcustom
          UPDATE FIELDS ( zz_bms_status )
          WITH VALUE #( ( touruuid      = ls_tour-touruuid
                          zz_bms_status = 'ERROR' ) )
        FAILED   DATA(lf_auth_mod)
        REPORTED DATA(lr_auth_mod).

      IF lf_auth_mod IS NOT INITIAL.
        MODIFY ENTITIES OF /plce/r_pdtour IN LOCAL MODE
          ENTITY tour
            CREATE BY \_extcustom
              FIELDS ( zz_bms_status )
              WITH VALUE #( (
                %tky    = ls_tour-%tky
                %target = VALUE #( (
                  %cid          = |BMS_AUTH_TGT_{ ls_tour-touruuid }|
                  zz_bms_status = 'ERROR' ) ) ) )
          FAILED   DATA(lf_auth_crt)
          REPORTED DATA(lr_auth_crt).

        IF lf_auth_crt IS NOT INITIAL.
          APPEND LINES OF lr_auth_crt-tour TO reported-tour.
        ENDIF.
      ENDIF.

      APPEND VALUE #( %tky = ls_tour-%tky ) TO failed-tour.
      CONTINUE.
    ENDIF.

    " --- service loop -----------------------------------------------------
    LOOP AT lt_asgmts INTO DATA(ls_asgmt)
      WHERE touruuid = ls_tour-touruuid.

      " ls_wr MUST be cleared: READ TABLE ... INTO leaves the work area
      " unchanged when nothing is found, so a service without a
      " /PLCE/TPDSRVWR row would inherit the previous service's data.
      CLEAR: ls_svc, ls_ewa, ls_wr, lt_containers,
             ls_cust_bp, ls_kunwe_bp, ls_carrier_bp, ls_recycler_bp,
             lv_movement_type, lv_ctype_name, lv_cont_old, lv_cont_new,
             lv_material_desc, lv_http_status, lv_response, lv_json,
             lv_pobjnr, ls_sig_missing.

      READ TABLE lt_services INTO ls_svc
        WITH TABLE KEY service_uuid = ls_asgmt-serviceuuid.

      IF sy-subrc <> 0.
        lv_count_error = lv_count_error + 1.
        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '025'
                   severity = if_abap_behv_message=>severity-error
                   v1       = lv_tour_id_out )
        ) TO reported-tour.
        CONTINUE.
      ENDIF.

      DATA(lv_svc_ref_out) = condense( |{ ls_svc-reference_id ALPHA = OUT }| ).
      DATA(lv_svc_id_out)  = condense( |{ ls_svc-service_id   ALPHA = OUT }| ).

      lv_pobjnr = ls_svc-reference_internal_id.

      READ TABLE lt_ewaobj INTO ls_ewa
        WITH TABLE KEY pobjnr = lv_pobjnr.

      IF sy-subrc <> 0.
        lv_count_error = lv_count_error + 1.
        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '029'
                   severity = if_abap_behv_message=>severity-error
                   v1       = lv_svc_ref_out )
        ) TO reported-tour.
        CONTINUE.
      ENDIF.

      " --- 6b2 Electronic signature (UST-S.AA.0006-005) --------------------
      "     Blocked by the signature check: message and error count were
      "     already produced in STEP 4c / 6a0, before the BMS login.
      "     Here the service is only skipped.
      READ TABLE lt_sig_missing TRANSPORTING NO FIELDS
        WITH TABLE KEY pobjnr = ls_ewa-pobjnr.

      IF sy-subrc = 0.
        CONTINUE.
      ENDIF.


      READ TABLE lt_srvwr INTO ls_wr
        WITH KEY service_uuid = ls_svc-service_uuid.

      " --- 6c Movement type: new | change | collect -----------------------
      "     CONDENSE matters — a CHAR column with trailing blanks sends
      "     "change " which will not convert to the BMS enum.
      SELECT SINGLE bms_movement_type
        FROM zwrtwaalaprctp
        WHERE service_type = @ls_svc-action
        INTO @lv_movement_type.

      lv_movement_type = to_lower( condense( lv_movement_type ) ).

      IF sy-subrc <> 0 OR lv_movement_type IS INITIAL.
        lv_count_error = lv_count_error + 1.
        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '026'
                   severity = if_abap_behv_message=>severity-error
                   v1       = ls_svc-action
                   v2       = lv_svc_ref_out
                   v3       = lv_tour_id_out )
        ) TO reported-tour.
        CONTINUE.
      ENDIF.

      " --- 6c2 Container type — only containerTypeName exists now ---------
      IF ls_ewa-beh_type IS INITIAL.
        lv_count_error = lv_count_error + 1.
        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '023'
                   severity = if_abap_behv_message=>severity-error
                   v1       = lv_svc_ref_out )
        ) TO reported-tour.
        CONTINUE.
      ENDIF.

      lv_ctype_name = condense( ls_ewa-beh_type ).

      DATA(lv_is_express) = xsdbool( ls_ewa-order_type = '02' ).

      " --- 6d Containers ---------------------------------------------------
      " Spec + runtime rules:
      "   new     (Aufstellung) — containerNumberNew required, old empty
      "   collect (Einzug)      — containerNumberOld required, new empty
      "   change  (Wechsel)     — BOTH required
      " EWA CONTAINER is the container standing at the customer, so it is
      " the OLD one — it only fills "new" for an Aufstellung.
      DATA(lv_cont_fallback) = condense( |{ ls_ewa-container ALPHA = OUT }| ).

      lv_cont_old = condense( ls_wr-container_atloc_tidnr ).
      lv_cont_new = condense( ls_wr-container_new_tidnr ).

      " TEST ONLY — sends the container TYPE where the ident number belongs.
      " Makes message 034 stop firing, but the driver then has no barcode
      " to scan. Enable deliberately, never leave active in production.
*      lv_cont_old = condense( ls_wr-containertype_atloc ).
*      lv_cont_new = condense( ls_wr-containertype_new ).

      CASE lv_movement_type.
        WHEN 'new'.
          CLEAR lv_cont_old.
          IF lv_cont_new IS INITIAL.
            lv_cont_new = lv_cont_fallback.
          ENDIF.
        WHEN 'collect'.
          CLEAR lv_cont_new.
          IF lv_cont_old IS INITIAL.
            lv_cont_old = lv_cont_fallback.
          ENDIF.
        WHEN OTHERS.               " change
          IF lv_cont_old IS INITIAL.
            lv_cont_old = lv_cont_fallback.
          ENDIF.
      ENDCASE.

      " Fail here with a readable message rather than letting BMS reject
      " the request for a field we already know is missing.
      DATA(lv_cont_missing) = xsdbool(
        ( lv_movement_type = 'new'     AND lv_cont_new IS INITIAL ) OR
        ( lv_movement_type = 'collect' AND lv_cont_old IS INITIAL ) OR
        ( lv_movement_type = 'change'  AND
          ( lv_cont_old IS INITIAL OR lv_cont_new IS INITIAL ) ) ).

      IF lv_cont_missing = abap_true.
        lv_count_error = lv_count_error + 1.
        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '034'
                   severity = if_abap_behv_message=>severity-error
                   v1       = lv_svc_ref_out
                   v2       = lv_movement_type )
        ) TO reported-tour.
        CONTINUE.
      ENDIF.

      " containerMovementTypeInfo is shown to the driver in the order
      " overview. It and internalRemark are required by the running API
      " and [Required] rejects empty strings, so both always carry content.
      DATA(lv_move_info) = SWITCH string( lv_movement_type
                             WHEN 'new'     THEN 'Aufstellung'
                             WHEN 'collect' THEN 'Einzug'
                             ELSE                'Wechsel' ).

      DATA(lv_internal_remark) = COND string(
        WHEN ls_svc-additional_text IS NOT INITIAL
        THEN condense( CONV string( ls_svc-additional_text ) )
        ELSE 'keine' ).

      APPEND VALUE #(
        container_number_old         = lv_cont_old
        container_number_new         = lv_cont_new
        movement_type                = lv_movement_type
        container_type_name          = lv_ctype_name
        container_movement_type_info = lv_move_info
        internal_remark              = lv_internal_remark
        customer_owned               = abap_false
      ) TO lt_containers.

      " --- 6e Service window — date-span, HH:MM:SS ------------------------
      DATA(lv_win_start) = COND string(
        WHEN ls_svc-service_window_start_time IS NOT INITIAL
        THEN |{ ls_svc-service_window_start_time+0(2) }:| &&
             |{ ls_svc-service_window_start_time+2(2) }:| &&
             |{ ls_svc-service_window_start_time+4(2) }| ).

      DATA(lv_win_end) = COND string(
        WHEN ls_svc-service_window_end_time IS NOT INITIAL
        THEN |{ ls_svc-service_window_end_time+0(2) }:| &&
             |{ ls_svc-service_window_end_time+2(2) }:| &&
             |{ ls_svc-service_window_end_time+4(2) }| ).

      " --- 6f Signature ----------------------------------------------------
      "     was 'UTERSCHR' — missing N, so this never matched
      DATA(lv_sig_req) = xsdbool(
        to_upper( ls_ewa-ordertxt )        CS 'UNTERSCHR' OR
        to_upper( ls_svc-additional_text ) CS 'UNTERSCHR' ).

      " --- 6g Notes ---------------------------------------------------------
      DATA(lv_notes) = CONV string( ls_ewa-ordertxt ).

      " TODO: hardcoded KVV rule — business rule, left unchanged.
      "       KUNWE is numeric, so KUNWE CS 'IS'/'RKT' can never be true.
      IF ls_ewa-smaufnr CS 'IS'  OR ls_ewa-kunwe CS 'IS'
      OR ls_ewa-smaufnr CS 'RKT' OR ls_ewa-kunwe CS 'RKT'
      OR ls_ewa-kunnr = '0000113840'
      OR ls_ewa-kunnr = '0000125669'.
        lv_notes &&= ' AUFTRAG ZU KVV'.
      ENDIF.

      IF ls_ewa-smaufnr IS NOT INITIAL.
        lv_notes &&= | Bestellnummer: { ls_ewa-smaufnr ALPHA = OUT }|.
      ENDIF.

      IF lv_is_express = abap_true.
        lv_notes &&= ' EXPRESSAUFTRAG'.
      ENDIF.

      DATA(lv_cert_nr) = COND string(
        WHEN ls_ewa-appno    IS NOT INITIAL THEN ls_ewa-appno
        WHEN ls_ewa-intappno IS NOT INITIAL THEN ls_ewa-intappno ).
      IF lv_cert_nr IS NOT INITIAL.
        lv_notes &&= | Entsorgungsnachweis: { lv_cert_nr }|.
      ENDIF.

      " --- 6h Business partners --------------------------------------------
      "     Decide the partner FIRST, then take number AND address from
      "     that one decision.
      DATA(lv_carrier_bp_no) = COND bu_partner(
        WHEN ls_ewa-transporter IS NOT INITIAL THEN ls_ewa-transporter
        ELSE lv_def_carrier ).

      DATA(lv_recycler_bp_no) = COND bu_partner(
        WHEN ls_ewa-disposer IS NOT INITIAL THEN ls_ewa-disposer
        ELSE lv_def_recycler ).

      READ TABLE lt_bp_addr INTO ls_cust_bp
        WITH KEY partner = ls_ewa-kunnr.
      READ TABLE lt_bp_addr INTO ls_kunwe_bp
        WITH KEY partner = ls_ewa-kunwe.
      READ TABLE lt_bp_addr INTO ls_carrier_bp
        WITH KEY partner = lv_carrier_bp_no.
      READ TABLE lt_bp_addr INTO ls_recycler_bp
        WITH KEY partner = lv_recycler_bp_no.

      DATA(lv_kunnr_out)    = condense( |{ ls_ewa-kunnr ALPHA = OUT }| ).
      DATA(lv_kunwe_out)    = condense( |{ ls_ewa-kunwe ALPHA = OUT }| ).
      DATA(lv_carrier_out)  = condense( |{ lv_carrier_bp_no  ALPHA = OUT }| ).
      DATA(lv_recycler_out) = condense( |{ lv_recycler_bp_no ALPHA = OUT }| ).

      " Every contact field has minLength: 1 — an empty string fails
      " validation exactly like a missing one. Optional contacts are
      " therefore filled ONLY when complete; incomplete ones are omitted.
      DATA(lv_cust_ok) = xsdbool(
        lv_kunnr_out          IS NOT INITIAL AND
        ls_cust_bp-name1      IS NOT INITIAL AND
        ls_cust_bp-street     IS NOT INITIAL AND
        ls_cust_bp-house_num1 IS NOT INITIAL AND
        ls_cust_bp-post_code1 IS NOT INITIAL AND
        ls_cust_bp-city1      IS NOT INITIAL ).

      IF lv_cust_ok = abap_false.
        lv_count_error = lv_count_error + 1.
        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '030'
                   severity = if_abap_behv_message=>severity-error
                   v1       = lv_svc_ref_out
                   v2       = lv_kunnr_out )
        ) TO reported-tour.
        CONTINUE.
      ENDIF.

      DATA(lv_kunwe_ok) = xsdbool(
        ls_kunwe_bp-street     IS NOT INITIAL AND
        ls_kunwe_bp-house_num1 IS NOT INITIAL AND
        ls_kunwe_bp-post_code1 IS NOT INITIAL AND
        ls_kunwe_bp-city1      IS NOT INITIAL ).

      DATA(lv_carrier_ok) = xsdbool(
        lv_carrier_out           IS NOT INITIAL AND
        ls_carrier_bp-name1      IS NOT INITIAL AND
        ls_carrier_bp-street     IS NOT INITIAL AND
        ls_carrier_bp-house_num1 IS NOT INITIAL AND
        ls_carrier_bp-post_code1 IS NOT INITIAL AND
        ls_carrier_bp-city1      IS NOT INITIAL ).

      DATA(lv_recycler_ok) = xsdbool(
        ls_recycler_bp-name1      IS NOT INITIAL AND
        ls_recycler_bp-street     IS NOT INITIAL AND
        ls_recycler_bp-house_num1 IS NOT INITIAL AND
        ls_recycler_bp-post_code1 IS NOT INITIAL AND
        ls_recycler_bp-city1      IS NOT INITIAL ).

      " --- 6i Waste key and description --------------------------------------
      " garbageKey = Materialnummer + "#" + AVV-Code, e.g. ABK_AZB#200301
      DATA(lv_matnr_out) = condense( |{ ls_wr-material ALPHA = OUT }| ).
      DATA(lv_avv_out)   = condense( |{ ls_wr-ewc_code }| ).

      " fall back to the EWA AVV code when the service carries none
      IF lv_avv_out IS INITIAL.
        lv_avv_out = condense( ls_ewa-watp_avvcode ).
      ENDIF.

      " one part missing → send that part alone, never a dangling "#"
      DATA(lv_garbage_key) = COND string(
        WHEN lv_matnr_out IS NOT INITIAL AND lv_avv_out IS NOT INITIAL
        THEN |{ lv_matnr_out }#{ lv_avv_out }|
        WHEN lv_avv_out   IS NOT INITIAL THEN lv_avv_out
        ELSE                                  lv_matnr_out ).

      " the description belongs to the MATERIAL, not to the AVV code
      IF ls_wr-material IS NOT INITIAL.
        SELECT SINGLE maktx
          FROM makt
          WHERE matnr = @ls_wr-material
            AND spras = @sy-langu
          INTO @lv_material_desc.
      ENDIF.

      " --- 6j Duration HHMMSS -> minutes ------------------------------------
      DATA(lv_duration_min) = COND i(
        WHEN ls_ewa-planned_durt CO ' 0123456789'
        THEN ls_ewa-planned_durt(2) * 60 + ls_ewa-planned_durt+2(2)
        ELSE 0 ).

      " --- 6k Order number — required, minLength 1 --------------------------
      DATA(lv_smaufnr_clean) = condense( |{ ls_ewa-smaufnr ALPHA = OUT }| ).
      DATA(lv_ordernr_clean) = condense( |{ ls_ewa-ordernr ALPHA = OUT }| ).
      DATA(lv_sdaufnr_clean) = condense( |{ ls_ewa-sdaufnr ALPHA = OUT }| ).

      DATA lv_order_number TYPE aufnr.

      lv_order_number = COND string(
        WHEN lv_smaufnr_clean IS NOT INITIAL THEN lv_smaufnr_clean
        WHEN lv_ordernr_clean IS NOT INITIAL THEN lv_ordernr_clean
        WHEN lv_sdaufnr_clean IS NOT INITIAL THEN lv_sdaufnr_clean
        WHEN lv_svc_ref_out   IS NOT INITIAL THEN lv_svc_ref_out
        ELSE                                      lv_svc_id_out ).

      " --- 6l plannedDate — required, date-time -----------------------------
      DATA(lv_planned_date) = COND string(
        WHEN ls_svc-earliest_date  IS NOT INITIAL
        THEN |{ ls_svc-earliest_date DATE = ISO }T00:00:00.000Z|
        WHEN ls_svc-requested_date IS NOT INITIAL
        THEN |{ ls_svc-requested_date DATE = ISO }T00:00:00.000Z|
        WHEN ls_ewa-zz_order_date  IS NOT INITIAL
        THEN |{ ls_ewa-zz_order_date DATE = ISO }T00:00:00.000Z|
        WHEN ls_ewa-old_order_date IS NOT INITIAL
        THEN |{ ls_ewa-old_order_date DATE = ISO }T00:00:00.000Z|
        WHEN ls_tour-startdate     IS NOT INITIAL
        THEN |{ ls_tour-startdate DATE = ISO }T00:00:00.000Z| ).

      IF lv_planned_date IS INITIAL.
        lv_count_error = lv_count_error + 1.
        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '028'
                   severity = if_abap_behv_message=>severity-error
                   v1       = lv_svc_ref_out )
        ) TO reported-tour.
        CONTINUE.
      ENDIF.

      " --- 6m Assemble payload ----------------------------------------------
      DATA(ls_order) = VALUE ty_bms_order(
        status             = 'ok'
        external_system_id = lv_tour_id_out
        sort_number        = ls_asgmt-toursequence
        order_number       = lv_order_number
        order_sheet        = condense( ls_ewa-watp_notenr )
        order_sheet_type   = COND string( WHEN ls_ewa-watp_notenr IS NOT INITIAL
                                          THEN 'LS' )

        customer = VALUE #(
          number        = lv_kunnr_out
          name1         = condense( ls_cust_bp-name1 )
          name2         = condense( ls_cust_bp-name2 )
          street        = condense( ls_cust_bp-street )
          street_number = condense( ls_cust_bp-house_num1 )
          zip_code      = condense( ls_cust_bp-post_code1 )
          city          = condense( ls_cust_bp-city1 ) )

        place_of_delivery = COND #( WHEN lv_kunwe_ok = abap_true
          THEN VALUE #(
            street        = condense( ls_kunwe_bp-street )
            street_number = condense( ls_kunwe_bp-house_num1 )
            zip_code      = condense( ls_kunwe_bp-post_code1 )
            city          = condense( ls_kunwe_bp-city1 ) ) )

        location = COND #( WHEN lv_kunwe_ok = abap_true
          THEN VALUE #(
            street        = condense( ls_kunwe_bp-street )
            street_number = condense( ls_kunwe_bp-house_num1 )
            zip_code      = condense( ls_kunwe_bp-post_code1 )
            city          = condense( ls_kunwe_bp-city1 ) ) )

        estimated_duration         = lv_duration_min
        planned_date               = lv_planned_date
        execution_time_frame_start = lv_win_start
        execution_time_frame_end   = lv_win_end
        notes                      = lv_notes

        special_notes = COND string(
          WHEN lv_is_express = abap_true
          THEN |EXPRESSAUFTRAG| &&
               COND string( WHEN ls_svc-additional_text IS NOT INITIAL
                            THEN | - | && ls_svc-additional_text )
          ELSE ls_svc-additional_text )

        " producer = Auftraggeber, same data as customer
        producer = VALUE #(
          number        = lv_kunnr_out
          name1         = condense( ls_cust_bp-name1 )
          name2         = condense( ls_cust_bp-name2 )
          street        = condense( ls_cust_bp-street )
          street_number = condense( ls_cust_bp-house_num1 )
          zip_code      = condense( ls_cust_bp-post_code1 )
          city          = condense( ls_cust_bp-city1 ) )

        recycler = COND #( WHEN lv_recycler_ok = abap_true
          THEN VALUE #(
            name1         = condense( ls_recycler_bp-name1 )
            name2         = condense( ls_recycler_bp-name2 )
            street        = condense( ls_recycler_bp-street )
            street_number = condense( ls_recycler_bp-house_num1 )
            zip_code      = condense( ls_recycler_bp-post_code1 )
            city          = condense( ls_recycler_bp-city1 ) ) )

        carrier = COND #( WHEN lv_carrier_ok = abap_true
          THEN VALUE #(
            number        = lv_carrier_out
            name1         = condense( ls_carrier_bp-name1 )
            name2         = condense( ls_carrier_bp-name2 )
            street        = condense( ls_carrier_bp-street )
            street_number = condense( ls_carrier_bp-house_num1 )
            zip_code      = condense( ls_carrier_bp-post_code1 )
            city          = condense( ls_carrier_bp-city1 ) ) )

        garbage_key  = lv_garbage_key
        garbage_name = condense( lv_material_desc )

        coll_consignment_note_nr = condense( ls_ewa-watp_noteintnr )

        team               = lv_team
        signature_required = lv_sig_req
        containers         = lt_containers ).

*----------------------------------------------------------------------*
* STEP 7 — Serialize
*          compress = abap_true: optional contacts left initial must be
*          OMITTED, not sent as {} or with empty strings — every field
*          in them has minLength: 1.
*----------------------------------------------------------------------*
      lv_json = /ui2/cl_json=>serialize(
        data        = ls_order
        compress    = abap_true
        pretty_name = /ui2/cl_json=>pretty_mode-camel_case ).

      " ABAP component names cap at 30 chars, so this one needs patching
      REPLACE ALL OCCURRENCES OF '"collConsignmentNoteNr"'
        IN lv_json WITH '"collectiveConsignmentNoteNumber"'.

      " Safety net: an empty object would fail required-field validation.
      " Check the logged payload — if these never fire, delete them.
      REPLACE ALL OCCURRENCES OF '"placeOfDelivery":{},' IN lv_json WITH ``.
      REPLACE ALL OCCURRENCES OF ',"placeOfDelivery":{}' IN lv_json WITH ``.
      REPLACE ALL OCCURRENCES OF '"location":{},'        IN lv_json WITH ``.
      REPLACE ALL OCCURRENCES OF ',"location":{}'        IN lv_json WITH ``.
      REPLACE ALL OCCURRENCES OF '"recycler":{},'        IN lv_json WITH ``.
      REPLACE ALL OCCURRENCES OF ',"recycler":{}'        IN lv_json WITH ``.
      REPLACE ALL OCCURRENCES OF '"carrier":{},'         IN lv_json WITH ``.
      REPLACE ALL OCCURRENCES OF ',"carrier":{}'         IN lv_json WITH ``.

*----------------------------------------------------------------------*
* STEP 8 — POST via the SM59 destination
*----------------------------------------------------------------------*
      zcl_wr_pd_tour_helper=>post_bms_order(
        EXPORTING
          iv_destination  = lv_dest
          iv_bearer_token = lv_bearer_token
          iv_json         = lv_json
        IMPORTING
          ev_http_status  = lv_http_status
          ev_response     = lv_response ).

      zcl_wr_pd_tour_helper=>log_bms_call(
        iv_tour_uuid    = ls_tour-touruuid
        iv_service_uuid = ls_asgmt-serviceuuid
        iv_order_number = lv_order_number
        iv_endpoint     = '/api/container/create-order-halle'
        iv_http_status  = lv_http_status
        iv_request      = lv_json
        iv_response     = lv_response
        iv_pobjnr       = ls_ewa-pobjnr ).

*----------------------------------------------------------------------*
* STEP 9 — Evaluate response
*----------------------------------------------------------------------*
      IF lv_http_status = 200 OR lv_http_status = 201.

        lv_count_ok = lv_count_ok + 1.

        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '021'
                   severity = if_abap_behv_message=>severity-success
                   v1       = lv_svc_ref_out
                   v2       = lv_order_number )
        ) TO reported-tour.

      ELSE.

        lv_count_error = lv_count_error + 1.

        " 400 carries {"errors":{...}} or {"error":{"message":...}};
        " 500 carries only a generic title plus a traceId. The complete
        " body is always in ZBMS_API_LOG.
        CLEAR ls_err_body.
        TRY.
            /ui2/cl_json=>deserialize(
              EXPORTING json = lv_response
              CHANGING  data = ls_err_body ).
          CATCH cx_root.
            CLEAR ls_err_body.
        ENDTRY.

        DATA(lv_bms_msg) = COND string(
          WHEN ls_err_body-error-message IS NOT INITIAL
          THEN ls_err_body-error-message
          WHEN strlen( lv_response ) > 80
          THEN |HTTP { lv_http_status }: { lv_response(80) }|
          WHEN lv_response IS NOT INITIAL
          THEN |HTTP { lv_http_status }: { lv_response }|
          ELSE |HTTP { lv_http_status }| ).

        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '022'
                   severity = if_abap_behv_message=>severity-error
                   v1       = lv_svc_ref_out
                   v2       = lv_bms_msg ) ) TO reported-tour.

      ENDIF.

      " --- per-service status -----------------------------------------------
      "     ZWR_BMS_UPDATE_SERVICE runs in its own LUW (DESTINATION 'NONE'):
      "     /PLCE/TPDSRVCST by UPDATE, EWA_ORDER_OBJECT through the item BO.
      "     The HTTP call has already happened — a failed status write is
      "     reported to the dispatcher as a warning, never swallowed.
      DATA(lv_svc_bms_status) = COND string(
        WHEN lv_http_status = 200 OR lv_http_status = 201
        THEN 'FREIGEGEBEN'
        ELSE 'ERROR' ).

      CLEAR: lv_upd_subrc, lt_upd_return, lv_rfc_msg.

      CALL FUNCTION 'ZWR_BMS_UPDATE_SERVICE'
        DESTINATION 'NONE'
        EXPORTING
          iv_service_uuid       = ls_asgmt-serviceuuid
          iv_pobjnr             = ls_ewa-pobjnr
          iv_zz_bms_status      = CONV ZZ_D_BMS_STATUS( lv_svc_bms_status )
        IMPORTING
          ev_subrc              = lv_upd_subrc
          et_return             = lt_upd_return
        EXCEPTIONS
          communication_failure = 1 MESSAGE lv_rfc_msg
          system_failure        = 2 MESSAGE lv_rfc_msg
          OTHERS                = 3.

      IF sy-subrc <> 0 OR lv_upd_subrc <> 0.
        DATA(lv_status_err) = COND string(
          WHEN lines( lt_upd_return ) > 0 THEN lt_upd_return[ 1 ]-message
          WHEN lv_rfc_msg IS NOT INITIAL   THEN lv_rfc_msg
          ELSE |RFC { sy-subrc }| ).

        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '046'
                   severity = if_abap_behv_message=>severity-warning
                   v1       = lv_svc_ref_out
                   v2       = lv_svc_bms_status
                   v3       = lv_status_err )
        ) TO reported-tour.
      ENDIF.

    ENDLOOP.   " services

*----------------------------------------------------------------------*
* Nothing went out for this tour
*     RAP downgrades an error in REPORTED to a warning when the key is
*     not also in FAILED — that is what produced the "proceed anyway"
*     dialog instead of a plain error list.
*----------------------------------------------------------------------*
    IF lv_count_ok = 0.
      APPEND VALUE #( %tky = ls_tour-%tky ) TO failed-tour.
      CONTINUE.
    ENDIF.

*----------------------------------------------------------------------*
* STEP 10 — Tour status
*----------------------------------------------------------------------*
    DATA(lv_bms_status) = COND /plce/char20(
      WHEN lv_count_ok > 0 AND lv_count_error = 0 THEN 'SENT'
      WHEN lv_count_ok > 0                        THEN 'PARTIAL'
      ELSE                                             'ERROR' ).

    MODIFY ENTITIES OF /plce/r_pdtour IN LOCAL MODE
      ENTITY extcustom
        UPDATE FIELDS ( zz_bms_status )
        WITH VALUE #( ( touruuid      = ls_tour-touruuid
                        zz_bms_status = lv_bms_status ) )
      FAILED   DATA(lf_mod)
      REPORTED DATA(lr_mod).

    IF lf_mod IS NOT INITIAL.
      MODIFY ENTITIES OF /plce/r_pdtour IN LOCAL MODE
        ENTITY tour
          CREATE BY \_extcustom
            FIELDS ( zz_bms_status )
            WITH VALUE #( (
              %tky    = ls_tour-%tky
              %target = VALUE #( (
                %cid          = |BMS_STAT_TGT_{ ls_tour-touruuid }|
                zz_bms_status = lv_bms_status ) ) ) )
        FAILED   DATA(lf_crt)
        REPORTED DATA(lr_crt).

      IF lf_crt IS NOT INITIAL.
        APPEND LINES OF lr_crt-tour TO reported-tour.
      ENDIF.
    ENDIF.

  ENDLOOP.   " tours

ENDMETHOD.
