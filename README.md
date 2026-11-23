CLASS lhc_Weighingbridge DEFINITION INHERITING FROM cl_abap_behavior_handler.
  PRIVATE SECTION.
    DATA gs_ls_weighing_result TYPE zwr_swaa_selfweigh_result.
    DATA gv_weigh_transaction_started TYPE abap_bool.
    data gv_ewawa_korsel type ewawa_weightext.
    METHODS get_instance_authorizations FOR INSTANCE AUTHORIZATION
      IMPORTING keys REQUEST requested_authorizations FOR Weighingbridge RESULT result.

    METHODS printSlip FOR READ
      IMPORTING keys FOR FUNCTION Weighingbridge~printSlip RESULT result.

    METHODS determineWeight FOR MODIFY
      IMPORTING keys FOR ACTION Weighingbridge~determineWeight RESULT result.

    METHODS identifyCard FOR MODIFY
      IMPORTING keys FOR ACTION Weighingbridge~identifyCard RESULT result.

    METHODS identifyKorsel FOR MODIFY
      IMPORTING keys FOR ACTION Weighingbridge~identifyKorsel RESULT result.

ENDCLASS.

CLASS lhc_Weighingbridge IMPLEMENTATION.

******************************************************************************************
* Authorization
******************************************************************************************
  METHOD get_instance_authorizations.
    READ ENTITIES OF zi_wr_r_weighbridge IN LOCAL MODE
      ENTITY Weighingbridge
      FIELDS ( SalesDocument ) WITH CORRESPONDING #( keys )
      RESULT DATA(lt_sessions)
      FAILED failed.

    result = VALUE #( FOR ls_session IN lt_sessions
      ( %tky   = ls_session-%tky
*      %create              = if_abap_behv=>auth-allowed
        %update              = if_abap_behv=>auth-allowed
        %action-identifyCard = if_abap_behv=>auth-allowed
        %action-determineWeight  = if_abap_behv=>auth-allowed ) ).
  ENDMETHOD.

******************************************************************************************
* Identification of Card
******************************************************************************************
  METHOD identifyCard.

    DATA lt_success_keys TYPE TABLE FOR READ IMPORT zi_wr_r_weighbridge.
    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ls_key>).
      " Validate contract existence (adjust table name if not VBAK)
*      SELECT SINGLE @abap_true FROM zi_wr_r_weighbridge
*        WHERE SalesDocument = @<ls_key>-%param-vbeln

      SELECT SINGLE @abap_true FROM zi_wr_r_weighbridge
        WHERE IDCardNr = @<ls_key>-%param-Vbeln

        INTO @DATA(lv_exists).
      IF sy-subrc = 0.
        " ✓ Valid: report success message only (no update)
        APPEND VALUE #(
          %tky = <ls_key>-%tky
          %msg = new_message(
                   id = 'ZWR_WEIGHBRIGE_MESS'
                   number = '000'
                   severity = if_abap_behv_message=>severity-success
                   v1 = <ls_key>-%param-vbeln ) )
          TO reported-Weighingbridge.

        " Collect for result
        APPEND VALUE #( %tky = <ls_key>-%tky ) TO lt_success_keys.
      ELSE.
        " ✗ Invalid: report bound error and fail the action
        APPEND VALUE #(
          %tky = <ls_key>-%tky
          %msg = new_message(
                   id = 'ZWR_WEIGHBRIGE_MESS'
                   number = '001'
                   severity = if_abap_behv_message=>severity-error
                   v1 = <ls_key>-%param-vbeln )
          %element-salesdocument = if_abap_behv=>mk-on  " Bind message to Vbeln field
        ) TO reported-Weighingbridge.

        " Fail the function (set failed to prevent result for this key)
        APPEND VALUE #(
          %tky = <ls_key>-%tky
          %fail-cause = if_abap_behv=>cause-unspecific
        ) TO failed-Weighingbridge.
      ENDIF.

      "***************************************************************************
      "Call and check if there is already a Transaction for this Sales Document
      "**************************************************************************
*      ZCL_WEIGHING_HANDLER=>cl_has_open_weighingproc(
*          EXPORTING
*            iv_contractID   = <ls_key>-%param-vbeln                      "Sales Document
*          RECEIVING
*            RV_HAS_WPROC =  DATA(lv_weigh_transaction_started)        "Weighing Transaction: True if there is and false if there is not
*        ).

    ENDLOOP.

*    IF lv_weigh_transaction_started = abap_true .
*      "When there is already a transaction for this
*      APPEND VALUE #(
*        %tky = <ls_key>-%tky
*        %msg = new_message(
*                 id = 'ZWR_WEIGHBRIGE_MESS'
*                 number = '007'
*                 severity = if_abap_behv_message=>severity-success
*                 v1 = <ls_key>-%param-vbeln ) )
*        TO reported-Weighingbridge.
*
*      " Collect for result
*      APPEND VALUE #( %tky = <ls_key>-%tky ) TO lt_success_keys.
*    ENDIF.

    " Read and return full data for successful instances
    IF lt_success_keys IS NOT INITIAL.
      READ ENTITIES OF zi_wr_r_weighbridge IN LOCAL MODE
        ENTITY Weighingbridge
        ALL FIELDS WITH lt_success_keys
        RESULT DATA(lt_result)
        FAILED DATA(lt_read_failed).
      result = CORRESPONDING #( lt_result ).
    ENDIF.
  ENDMETHOD.


******************************************************************************************
* Identification of Korsel
******************************************************************************************
  METHOD identifykorsel.


    DATA lt_success_keys TYPE TABLE FOR READ IMPORT zi_wr_r_weighbridge.
    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ls_key>).

        SELECT SINGLE * FROM zi_wr_r_weighbridge
        WHERE SalesDocument = @<ls_key>-%key-SalesDocument INTO @data(ls_weighbrigh).

      IF  <ls_key>-%param-Korselsnr is not INITIAL.
        " ✓ Valid: report success message only (no update)
        APPEND VALUE #(
          %tky = <ls_key>-%tky
          %msg = new_message(
                   id = 'ZWR_WEIGHBRIGE_MESS'
                   number = '010'
                   severity = if_abap_behv_message=>severity-success
                   v1 = <ls_key>-%param-Korselsnr ) )
          TO reported-Weighingbridge.
        " Collect for result
        APPEND VALUE #( %tky = <ls_key>-%tky ) TO lt_success_keys.
      ELSE.
*        " ✗ Invalid: report bound error and fail the action
        APPEND VALUE #(
          %tky = <ls_key>-%tky
          %msg = new_message(
                   id = 'ZWR_WEIGHBRIGE_MESS'
                   number = '011'
*                   severity = if_abap_behv_message=>severity-error
                   severity = if_abap_behv_message=>severity-success
                   v1 = <ls_key>-%param-Korselsnr )
          %element-salesdocument = if_abap_behv=>mk-on  " Bind message to Vbeln field
        ) TO reported-Weighingbridge.

        " Fail the function (set failed to prevent result for this key)
        APPEND VALUE #(
          %tky = <ls_key>-%tky
          %fail-cause = if_abap_behv=>cause-unspecific
        ) TO failed-Weighingbridge.
      ENDIF.
    ENDLOOP.
    " Read and return full data for successful instances
    IF lt_success_keys IS NOT INITIAL.
      READ ENTITIES OF zi_wr_r_weighbridge IN LOCAL MODE
        ENTITY Weighingbridge
        ALL FIELDS WITH lt_success_keys
        RESULT DATA(lt_result)
        FAILED DATA(lt_read_failed).
      result = CORRESPONDING #( lt_result ).
    ENDIF.
  ENDMETHOD.
*********************************************************************************************
* Selection of Load type
*********************************************************************************************
  METHOD determineWeight.
    DATA lt_success_keys TYPE TABLE FOR READ IMPORT zi_wr_r_weighbridge.

    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ls_key>).
*
TRY.
        ZCL_WR_WEIGHBRIDGE_HELPER=>cl_weighing(
          EXPORTING
            iv_contractid = <ls_key>-%param-Vbeln " Sales Document
            iv_wasteid = <ls_key>-%param-Loadtype "Waste = MATNR
*           * iv_licence_plate = " License plate number
            iv_simulation = abap_false
            iv_user = SY-UNAME " User Name in User Master Record
            iv_commit = abap_false " No need for explicit commit. Commit is handle by RAP framework
          IMPORTING
            iv_weighing_result = gs_ls_weighing_result " returning information from weighing process
        ).
      CATCH cx_eewa_base INTO DATA(lx_error).
        APPEND VALUE #(
          %tky = <ls_key>-%tky
          %msg = new_message(
                   id = 'ZWR_WEIGHBRIGE_MESS'  " Use your custom message class
                   number = '012'  " Assume a message number for weighing error
                   severity = if_abap_behv_message=>severity-error
                   v1 = lx_error->get_text( ) )
          %element-salesdocument = if_abap_behv=>mk-on  " Bind to relevant field if needed
        ) TO reported-weighingbridge.
        APPEND VALUE #(
          %tky = <ls_key>-%tky
          %fail-cause = if_abap_behv=>cause-unspecific
        ) TO failed-weighingbridge.
        CONTINUE.  " Skip further processing for this failed key
      ENDTRY.


      " Read the current session data
      READ ENTITIES OF zi_wr_r_weighbridge IN LOCAL MODE
        ENTITY Weighingbridge
        ALL FIELDS WITH VALUE #( ( %tky = <ls_key>-%tky ) )
        RESULT DATA(lt_sessions)
        FAILED DATA(lt_failed).

      IF lt_sessions IS NOT INITIAL.
        DATA(ls_session) = lt_sessions[ 1 ].
        DATA lv_unit TYPE weight_unit.
        " Example: Assume weight is calculated or fetched (e.g., from a device or table)
        IF gs_ls_weighing_result-bruttoweight IS NOT INITIAL.
          DATA(lv_grossweight) = gs_ls_weighing_result-bruttoweight. "
          lv_unit   = gs_ls_weighing_result-weightunit.
        ENDIF.
        IF gs_ls_weighing_result-taraweight IS NOT INITIAL.
          DATA(lv_teraweight)  = gs_ls_weighing_result-taraweight. "
          lv_unit   = gs_ls_weighing_result-weightunit.
        ENDIF.

        lv_unit = 'KG'.
        IF lv_grossweight IS INITIAL.
          " Simulate fetching weight
          lv_grossweight =  5674367." Example value in kg
*        " Update the entity with the weight
        ENDIF.

        IF lv_teraweight IS INITIAL.
          " Simulate fetching weight
*         lv_teraweight = 3998765. " Example value in kg
*        " Update the entity with the weight
        ENDIF.

        IF lv_grossweight IS NOT INITIAL.
          " Report success of Gross Weight
          APPEND VALUE #(
            %tky = <ls_key>-%tky
            %msg = new_message(
                     id = 'ZWR_WEIGHBRIGE_MESS'
                     number = '002'
                     severity = if_abap_behv_message=>severity-success
                     v1 = |{ lv_grossweight }  { lv_unit }|
                      )
          ) TO reported-weighingbridge.

        ENDIF.

        IF lv_teraweight IS NOT INITIAL.
          " Report success of Tera weight
          APPEND VALUE #(
            %tky = <ls_key>-%tky
            %msg = new_message(
                     id = 'ZWR_WEIGHBRIGE_MESS'
                     number = '009'
                     severity = if_abap_behv_message=>severity-success
                     v1 = |{ lv_teraweight  }  { lv_unit }| )
          ) TO reported-weighingbridge.

        ENDIF.
        " Collect for result
        APPEND VALUE #( %tky = <ls_key>-%tky ) TO lt_success_keys.
      ELSE.
        " Report error if session not found
        APPEND VALUE #(
          %tky = <ls_key>-%tky
          %msg = new_message(
                   id = 'ZWR_WEIGHBRIGE_MESS'
                   number = '003'
                   severity = if_abap_behv_message=>severity-error
                   v1 = 'Session not found' )
        ) TO reported-weighingbridge.
        APPEND VALUE #( %tky = <ls_key>-%tky %fail-cause = if_abap_behv=>cause-not_found ) TO failed-weighingbridge.
      ENDIF.
    ENDLOOP.

  ENDMETHOD.

*********************************************************************************************
* Printing of slip
*********************************************************************************************
  METHOD printslip.
    DATA lv_base64 TYPE string.
    DATA lv_b64 TYPE string.
    DATA lv_b64_v2 TYPE string.
    DATA ls_wa TYPE ewa_wa_weighproc.
    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ls_key>).

      " Fetch the base64 PDF using the helper method
      ls_wa-brutto_weight  = gs_ls_weighing_result-bruttoweight.   "
      ls_wa-tare_weight    = gs_ls_weighing_result-taraweight.

      ls_wa-brutto_weight  = 5674367.
      ls_wa-tara_weight    = 3998765.
      ls_wa-brutto_unit    = 'KG'.
      ls_wa-tara_unit      = 'KG'.
      ls_wa-vbeln          = <ls_key>-%param-Vbeln.
      ls_wa-waste          = <ls_key>-%param-Loadtype.

      zcl_wr_weighbridge_helper=>form_to_base64(
        EXPORTING
          iv_form   = 'IS_U_WA_SF_WEIGHINGPROCESS'
          iv_lang   = sy-langu
*         iv_show   = abap_true
*         iv_pdf    = abap_false
*         iv_b64    = abap_false
*         iv_pdfpth =
*         iv_b64pth =
*         iv_prevln = 200
          Iv_WA     = ls_wa
        RECEIVING
          r_base64  = lv_base64    "--> This is a string
      ).


      IF   lv_base64 IS NOT INITIAL.
        " Report success message
        APPEND VALUE #(
          %tky = <ls_key>-%tky
          %msg = new_message(
                   id = 'ZWR_WEIGHBRIGE_MESS'
                   number = '004'  " Assume a new message for PDF generation success
                   severity = if_abap_behv_message=>severity-success
                   v1 = 'PDF generated successfully' )
        ) TO reported-weighingbridge.

        " Directly add to result (combine %tky with result entity fields)
        APPEND VALUE #( %tky = <ls_key>-%tky
                        %param = VALUE #( pdfbase64 = lv_base64 ) ) TO result.
      ELSE.
        " Report error if base64 generation failed
        APPEND VALUE #(
          %tky = <ls_key>-%tky
          %msg = new_message(
                   id = 'ZWR_WEIGHBRIGE_MESS'
                   number = '006'
                   severity = if_abap_behv_message=>severity-error
                   v1 = 'Failed to generate PDF' )
        ) TO reported-weighingbridge.
        APPEND VALUE #( %tky = <ls_key>-%tky %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-weighingbridge.
      ENDIF.
    ENDLOOP.
  ENDMETHOD.


ENDCLASS.



class ZCL_WEIGHING_HANDLER definition
  public
  final
  create public .

public section.

  class-methods CL_GET_CONTRACT
    importing
      !IV_CARDID type STRING
    returning
      value(RV_CONTRACT) type VBELN_VA .
  class-methods CL_HAS_OPEN_WEIGHINGPROC
    importing
      !IV_CARDID type STRING optional
      !IV_CONTRACTID type VBELN_VA optional
    returning
      value(RV_HAS_WPROC) type ABAP_BOOL .
  class-methods CL_WEIGHING
    importing
      !IV_CONTRACTID type VBELN_VA
      !IV_WASTEID type WASTE_TYPE optional
      !IV_LICENCE_PLATE type LICENSE_NUM optional
      !IV_SIMULATION type ABAP_BOOL default ABAP_FALSE
      !IV_USER type XUBNAME
      !IV_COMMIT type ABAP_BOOL optional
      !IV_DELIVERYTYPE type EWADWA_WEIGHPROCINTEXT default 'I'
      !IV_WDPLANT type EWAEL_WDPLANTNR optional
      !IV_DEVICEGROUPNR type EWAWA_DEVICEGROUPNR optional
    exporting
      !IV_WEIGHING_RESULT type ZWR_SWAA_SELFWEIGH_RESULT
    raising
      CX_EEWA_BASE .
  class-methods GET_OPEN_WEIGHPROC
    importing
      !IV_CONTRACTID type VBELN_VA
    returning
      value(RV_WEIGHINGID) type EWAWA_WEIGHINGNR .
  PROTECTED SECTION.

  PRIVATE SECTION.
    TYPES:
      ty_weighing_request TYPE zcl_wr_waa_ta_selfweighing=>sty_weighing_request,
      ty_weighing_result  TYPE zwr_swaa_selfweigh_result,
      ty_hw_profile       TYPE eewa_scale_user.

    CLASS-METHODS get_hw_profile
      IMPORTING
        iv_user           TYPE xubname
      RETURNING
        VALUE(rs_profile) TYPE ty_hw_profile.

    CLASS-METHODS execute_transaction
      IMPORTING
        is_process       TYPE ty_weighing_request
        iv_commit        TYPE abap_bool
      RETURNING
        VALUE(rs_result) TYPE ty_weighing_result
      RAISING
        cx_eewa_base.

ENDCLASS.



CLASS ZCL_WEIGHING_HANDLER IMPLEMENTATION.


  method CL_GET_CONTRACT.
    RV_CONTRACT = IV_CARDID. "design definition.
  endmethod.


  method CL_HAS_OPEN_WEIGHINGPROC.
    data:
      LV_CONTRACT     type VBELN_VA.

    RV_HAS_WPROC = ABAP_FALSE.
    LV_CONTRACT = IV_CONTRACTID.
    if IV_CARDID is not initial and LV_CONTRACT is initial.
      LV_CONTRACT = CL_GET_CONTRACT(
        exporting
          IV_CARDID   = IV_CARDID
      ).
    endif.

    if LV_CONTRACT is not initial and
       GET_OPEN_WEIGHPROC( exporting IV_CONTRACTid   = LV_CONTRACT ) is not initial.
      RV_HAS_WPROC = ABAP_TRUE.
    endif.
  endmethod.


  METHOD cl_weighing.
    DATA: ls_process TYPE ty_weighing_request,
          lv_commit  TYPE abap_bool.

    CLEAR iv_weighing_result.

    ls_process-devicegroup = iv_devicegroupnr.
    ls_process-facilityid  = iv_wdplant.

    IF ls_process-devicegroup IS INITIAL.
      DATA(ls_hw_profile) = get_hw_profile( iv_user ).
      IF ls_hw_profile IS NOT INITIAL.
        ls_process-devicegroup = ls_hw_profile-devicegroupnr.
        ls_process-facilityid  = ls_hw_profile-wdplant.
      ENDIF.
    ENDIF.

    IF ls_process-devicegroup IS NOT INITIAL.
      ls_process-weighingid   = get_open_weighproc( iv_contractid ).
      ls_process-processtype  = cl_eewa_bo_wdorderweigh=>cprocesstype_inbound.
      ls_process-simulation   = iv_simulation.
      ls_process-wasteid      = iv_wasteid.
      ls_process-deliverytype = iv_deliverytype.

      lv_commit = iv_commit.

      iv_weighing_result = execute_transaction(
                             is_process = ls_process
                             iv_commit  = lv_commit ).
    ENDIF.
  ENDMETHOD.


  METHOD execute_transaction.
    DATA(lo_ta) = NEW zcl_wr_waa_ta_selfweighing( ).

    lo_ta->gs_process_data = is_process.

    TRY.
        lo_ta->zz_book_weighing( ).
      CATCH cx_eewa_base INTO DATA(lx_error).
        ROLLBACK WORK.
        RAISE EXCEPTION lx_error.
    ENDTRY.

    rs_result = lo_ta->gs_result_data.

    IF iv_commit = abap_true.
      COMMIT WORK AND WAIT.
    ELSE.
      " No commit; changes may be buffered or not persisted depending on BO implementation
    ENDIF.
  ENDMETHOD.


  METHOD get_hw_profile.
    SELECT SINGLE *
      FROM eewa_scale_user
      INTO @rs_profile
      WHERE xuser = @iv_user.
  ENDMETHOD.


  METHOD get_open_weighproc.
    TYPES:
      BEGIN OF ty_weighinglead,
        weighingnr_lead TYPE ewa_wa_weighproc-weighingnr_lead,
      END OF ty_weighinglead.

    DATA:
      lr_ranges     TYPE REF TO ewapranges,
      lt_weighingnr TYPE STANDARD TABLE OF ty_weighinglead,
      lv_max        TYPE int4 VALUE 1,
      lv_grossid    TYPE ewa_wa_weighproc-gross_id,
      lv_tareid     TYPE ewa_wa_weighproc-tare_id,
      lv_weighinglead TYPE ewa_wa_weighproc-weighingnr_lead.

    IF iv_contractid IS INITIAL.
      RETURN.
    ENDIF.

    CREATE DATA lr_ranges.
    FIELD-SYMBOLS: <lt_ranges> TYPE ewapranges.
    ASSIGN lr_ranges->* TO <lt_ranges>.

    " For CONTRACT_VBELN
    TYPES tt_vbeln_range TYPE RANGE OF vbeln_va.
    DATA(lt_contract_range) = VALUE tt_vbeln_range( ( sign = 'I' option = 'EQ' low = iv_contractid ) ).
    DATA(lr_contract_ref) = REF #( lt_contract_range ).
    APPEND VALUE #( name = 'CONTRACT_VBELN' range = lr_contract_ref ) TO <lt_ranges>.

    " For GROSS_ID
    TYPES tt_gross_range TYPE RANGE OF ewa_wa_weighproc-gross_id.
    DATA(lt_gross_range) = VALUE tt_gross_range( ( sign = 'I' option = 'NE' low = lv_grossid ) ).
    DATA(lr_gross_ref) = REF #( lt_gross_range ).
    APPEND VALUE #( name = 'GROSS_ID' range = lr_gross_ref ) TO <lt_ranges>.

    " For TAREID
    TYPES tt_tare_range TYPE RANGE OF ewa_wa_weighproc-tare_id.
    DATA(lt_tare_range) = VALUE tt_tare_range( ( sign = 'I' option = 'EQ' low = lv_tareid ) ).
    DATA(lr_tare_ref) = REF #( lt_tare_range ).
    APPEND VALUE #( name = 'TAREID' range = lr_tare_ref ) TO <lt_ranges>.

    " For WEIGHINGNR_LEAD
    TYPES tt_lead_range TYPE RANGE OF ewa_wa_weighproc-weighingnr_lead.
    DATA(lt_lead_range) = VALUE tt_lead_range( ( sign = 'I' option = 'NE' low = lv_weighinglead ) ).
    DATA(lr_lead_ref) = REF #( lt_lead_range ).
    APPEND VALUE #( name = 'WEIGHINGNR_LEAD' range = lr_lead_ref ) TO <lt_ranges>.

    cl_eewa_or_weighingprocess=>cl_read(
      EXPORTING
        par_objtype = cl_eewa_bo_weighingprocess=>cot_weighingprocess
        par_domain  = 'OPEN'
        par_ranges  = lr_ranges
        par_maxrows = lv_max
      IMPORTING
        par_table   = lt_weighingnr
    ).

    IF line_exists( lt_weighingnr[ 1 ] ).
      rv_weighingid = lt_weighingnr[ 1 ]-weighingnr_lead.
    ENDIF.
    
  ENDMETHOD.
ENDCLASS.


class ZCL_WR_WEIGHBRIDGE_HELPER definition
  public
  create public .

public section.

  class-methods CL_GET_CONTRACT
    importing
      !IV_CARDID type STRING
    returning
      value(RV_CONTRACT) type VBELN_VA .
  class-methods CL_GET_OPEN_WEIGHPROC
    importing
      !IV_CONTRACT type VBELN_VA
    returning
      value(RV_WEIGHINGNR) type EWAWA_WEIGHINGNR .
  class-methods CL_HAS_OPEN_WEIGHINGPROC
    importing
      !IV_CARDID type STRING optional
      !IV_CONTRACTID type VBELN_VA optional
    returning
      value(RV_HAS_WPROC) type ABAP_BOOL .
  class-methods CL_WEIGHING
    importing
      !IV_CONTRACTID type VBELN_VA
      !IV_WASTEID type WASTE_TYPE optional
      !IV_LICENCE_PLATE type LICENSE_NUM optional
      !IV_SIMULATION type ABAP_BOOL default ABAP_FALSE
      !IV_USER type XUBNAME default SY-UNAME
      !IV_COMMIT type ABAP_BOOL optional
      !IV_DELIVERYTYPE type EWADWA_WEIGHPROCINTEXT default 'I'
      !IV_WDPLANT type EWAEL_WDPLANTNR optional
      !IV_DEVICEGROUPNR type EWAWA_DEVICEGROUPNR optional
      !IV_EWAWA_KORSEL type EWAWA_WEIGHTEXT optional
    exporting
      !IV_WEIGHING_RESULT type ZWR_SWAA_SELFWEIGH_RESULT
    raising
      CX_EEWA_BASE .
  class-methods FORM_TO_BASE64
    importing
      !IV_FORM type CHAR30 default 'IS_U_WA_SF_WEIGHINGPROCESS'          " SFP form name
      !IV_LANG type SYLANGU default SY-LANGU
      !IV_SHOW type ABAP_BOOL default ABAP_TRUE          " Show Base64 preview
      !IV_PDF type ABAP_BOOL default ABAP_FALSE           " Save PDF file
      !IV_B64 type ABAP_BOOL default ABAP_FALSE           " Save Base64 text
      !IV_PDFPTH type STRING optional "DEFAULT 'C:\Temp\adobe_form.pdf'
      !IV_B64PTH type STRING optional
      !IV_PREVLN type I default 200        " Preview chars
      !IV_WA type EWA_WA_WEIGHPROC
    returning
      value(R_BASE64) type STRING .  " Return the Base64 string
protected section.
private section.
ENDCLASS.



CLASS ZCL_WR_WEIGHBRIDGE_HELPER IMPLEMENTATION.


  METHOD form_to_base64.
    DATA: ls_outparams          TYPE sfpoutputparams,
          ls_docparams          TYPE sfpdocparams,
          lv_fm_name            TYPE rs38l_fnam,
          ls_formoutput         TYPE fpformoutput,
          lv_pdf_xstr           TYPE xstring,
          lv_pdf_b64            TYPE string,
          lv_job_opened         TYPE abap_bool,
          ls_ARCHIVE_INDEX      TYPE  toa_dara,
          ls_ARCHIVE_INDEX_TAB  TYPE  tsfdara,
          ls_ARCHIVE_PARAMETERS TYPE  arc_params,
          ls_CONTROL_PARAMETERS TYPE  ssfctrlop,
          ls_MAIL_APPL_OBJ      TYPE  swotobjid,
          ls_MAIL_RECIPIENT     TYPE  swotobjid,
          ls_MAIL_SENDER        TYPE  swotobjid.

    ls_outparams-nodialog = abap_true.
    ls_outparams-getpdf = abap_true.
    ls_docparams-langu = iv_lang.

    " Fill application data for the form interface
    " Example values - adjust based on your form's requirements

    TRY.
        " Open FP job
        CALL FUNCTION 'FP_JOB_OPEN'
          CHANGING
            ie_outputparams = ls_outparams.
            lv_job_opened = abap_true.

        " Resolve generated FM for form name
        CALL FUNCTION 'FP_FUNCTION_MODULE_NAME'
          EXPORTING
            i_name     = iv_form
          IMPORTING
            e_funcname = lv_fm_name.

        " === If your SFP interface needs additional application data, fill it here ===
        " ================================================================

        " Call the form FM
        CALL FUNCTION lv_fm_name
          EXPORTING
            /1bcdwb/docparams  = ls_docparams
            archive_index      = ls_archive_index
            archive_index_tab  = ls_archive_index_tab
            archive_parameters = ls_archive_parameters
            control_parameters = ls_control_parameters
            mail_appl_obj      = ls_mail_appl_obj
            mail_recipient     = ls_mail_recipient
            mail_sender        = ls_mail_sender
            wa                 = iv_wa
          IMPORTING
            /1bcdwb/formoutput = ls_formoutput
          EXCEPTIONS
            usage_error        = 1
            system_error       = 2
                                 internal_error= 3
            OTHERS             = 4.
        IF sy-subrc <> 0.
*          MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
*                  WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
        ENDIF.

      CLEANUP.
        IF lv_job_opened = abap_true.
          " Always close job
          CALL FUNCTION 'FP_JOB_CLOSE'
            EXCEPTIONS
              OTHERS = 1.
        ENDIF.
    ENDTRY.

    " Get PDF as XSTRING (newer) or convert from SOLIX (older)
    IF ls_formoutput-pdf IS NOT INITIAL.
      lv_pdf_xstr = ls_formoutput-pdf.
    " XSTRING -> Base64
      lv_pdf_b64 = cl_http_utility=>if_http_utility~encode_x_base64( lv_pdf_xstr ).
    ELSE.
      MESSAGE 'No PDF returned by ADS.' TYPE 'E'.
    ENDIF.
    " Optional: display a short preview of Base64
    IF iv_show = abap_true.
      DATA(lv_len) = iv_prevln.
      IF lv_len <= 0 OR lv_len > strlen( lv_pdf_b64 ).
        lv_len = strlen( lv_pdf_b64 ).
      ENDIF.
      WRITE: / 'Base64 (preview):'.
      ULINE.
      WRITE: / lv_len.
      SKIP.
      WRITE: / |Total Base64 length: { strlen( lv_pdf_b64 ) } chars|.
    ENDIF.

    " Optional: save PDF to frontend
    IF iv_pdf = abap_true.
      DATA(lt_solix) = cl_bcs_convert=>xstring_to_solix( lv_pdf_xstr ).
      cl_gui_frontend_services=>gui_download(
        EXPORTING
          filename = iv_pdfpth
          filetype = 'BIN'
        CHANGING
          data_tab = lt_solix
        EXCEPTIONS
          OTHERS = 1 ).
      IF sy-subrc = 0.
        WRITE: / |PDF saved to: { iv_pdfpth }|.
      ELSE.
        WRITE: / 'Could not save PDF to frontend.' COLOR COL_NEGATIVE.
      ENDIF.
    ENDIF.

    " Optional: save Base64 text to frontend
    IF iv_b64 = abap_true.
      DATA(lt_lines) = VALUE stringtab( ( lv_pdf_b64 ) ).
      cl_gui_frontend_services=>gui_download(
        EXPORTING
          filename = iv_b64pth
          filetype = 'ASC'
        CHANGING
          data_tab = lt_lines
        EXCEPTIONS
          OTHERS = 1 ).
      IF sy-subrc = 0.
        WRITE: / |Base64 saved to: { iv_b64pth }|.
      ELSE.
        WRITE: / 'Could not save Base64 to frontend.' COLOR COL_NEGATIVE.
      ENDIF.
    ENDIF.

    IF iv_pdf = abap_false AND iv_b64 = abap_false AND iv_show = abap_false.
      WRITE: / 'Done. (Nothing selected to show or save.)'.
    ENDIF.

*    r_base64 = lv_pdf_xstr. " Return the Base64 string
    r_base64 = lv_pdf_b64. " Return the Base64 string

  ENDMETHOD.


  method CL_GET_CONTRACT.
    RV_CONTRACT = IV_CARDID. "design definition.
  endmethod.


  method CL_HAS_OPEN_WEIGHINGPROC.
    data:
      LV_CONTRACT     type VBELN_VA.

    RV_HAS_WPROC = ABAP_FALSE.
    LV_CONTRACT = IV_CONTRACTID.
    if IV_CARDID is not initial and LV_CONTRACT is initial.
      LV_CONTRACT = CL_GET_CONTRACT(
        exporting
          IV_CARDID   = IV_CARDID
      ).
    endif.

    if LV_CONTRACT is not initial and
       CL_GET_OPEN_WEIGHPROC( exporting IV_CONTRACT   = LV_CONTRACT ) is not initial.
      RV_HAS_WPROC = ABAP_TRUE.
    endif.
  endmethod.


  METHOD cl_weighing.
    DATA:
      ls_process        TYPE zcl_wr_waa_ta_selfweighing=>sty_weighing_request,
      ls_process_result TYPE zwr_swaa_selfweigh_result.

    CLEAR:
      iv_weighing_result.

    ls_process-devicegroup = iv_devicegroupnr.
    ls_process-facilityid  = iv_wdplant.
    ls_process-ewawa_korsel = iv_ewawa_korsel.

    IF ls_process-devicegroup IS INITIAL.
      SELECT SINGLE *
        INTO @DATA(ls_hw_profile)
        FROM eewa_scale_user
        WHERE xuser = @iv_user.

      IF sy-subrc IS INITIAL.
        ls_process-devicegroup = ls_hw_profile-devicegroupnr.
        ls_process-facilityid  = ls_hw_profile-wdplant.
      ENDIF.
    ENDIF.

    IF ls_process-devicegroup IS NOT INITIAL.
      ls_process-weighingid = cl_get_open_weighproc(
        EXPORTING
          iv_contract   = iv_contractid
      ).
      ls_process-processtype = cl_eewa_bo_wdorderweigh=>cprocesstype_inbound.
      ls_process-simulation  = iv_simulation.
      ls_process-wasteid     = iv_wasteid.
      ls_process-deliverytype = cl_eewa_bo_multiweighprochead=>cdt_external.

      ta_begin_named_var ta zcl_wr_waa_ta_selfweighing=>c_ta_name.
      ta_do_raise_exception ta.
      IF iv_commit = abap_true.
        ta_do_commit ta.
      ELSE.
        ta_no_commit ta.
      ENDIF.
      ta_set_param ta gs_process_data ls_process.
      ta_execute ta zz_book_weighing.
      ta_get_param ta gs_result_data ls_process_result.
      ta_end ta.

    ENDIF.
    iv_weighing_result = ls_process_result.
  ENDMETHOD.


  method CL_GET_OPEN_WEIGHPROC.
    types:
      begin of TY_WEIGHINGLEAD,
        WEIGHINGNR_LEAD type EWA_WA_WEIGHPROC-WEIGHINGNR_LEAD,
      end of TY_WEIGHINGLEAD.
    data:
      LV_CONTRACT     type VBELN_VA,
      LV_GROSSID      type EWA_WA_WEIGHPROC-GROSS_ID,
      LV_TAREID       type EWA_WA_WEIGHPROC-TARE_ID,
      LV_WEIGHINGLEAD type EWA_WA_WEIGHPROC-WEIGHINGNR_LEAD,
      LV_MAX          type INT4 value 1,
      LT_WEIGHINGNR   type standard table of TY_WEIGHINGLEAD.

    clear RV_WEIGHINGNR.
    if IV_CONTRACT is not initial.
      DATA_BO_RANGE X.
      SET_BO_RANGE X CONTRACT_VBELN IV_CONTRACT VBELN_VA.
      SET_BO_RANGE_CMPLT X GROSS_ID LV_GROSSID EWA_WA_WEIGHPROC-GROSS_ID I NE.
      SET_BO_RANGE_CMPLT X TAREID LV_TAREID EWA_WA_WEIGHPROC-TARE_ID I EQ.
      SET_BO_RANGE_CMPLT X WEIGHINGNR_LEAD LV_WEIGHINGLEAD EWA_WA_WEIGHPROC-WEIGHINGNR_LEAD I NE.

      CL_EEWA_OR_WEIGHINGPROCESS=>CL_READ(
        exporting
          PAR_OBJTYPE = CL_EEWA_BO_WEIGHINGPROCESS=>COT_WEIGHINGPROCESS
          PAR_DOMAIN  = 'OPEN'
          PAR_RANGES  = LRANGEXS
          PAR_MAXROWS = LV_MAX
        importing
          PAR_TABLE   = LT_WEIGHINGNR
      ).
      if LINES( LT_WEIGHINGNR ) > 0.
        RV_WEIGHINGNR = LT_WEIGHINGNR[ 1 ]-WEIGHINGNR_LEAD.
      endif.
    endif.
  endmethod.
ENDCLASS.






