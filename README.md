METHOD populateextcustomfromorder.

  READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
    ENTITY service
    FIELDS ( referenceid serviceuuid referenceinternalid )
    WITH CORRESPONDING #( keys )
    RESULT DATA(lt_services)
    FAILED DATA(lt_failed).

  IF lt_services IS INITIAL.
    RETURN.
  ENDIF.

  DATA: lt_ext_create TYPE TABLE FOR CREATE /plce/r_pdservice\_extcustom,
        lt_ext_update TYPE TABLE FOR UPDATE /plce/r_pdservice\\extcustom.

  LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_service>).

    " GUARD CLAUSE: Check if ExtCustom already fully populated
    READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY extcustom
      FIELDS ( serviceuuid wdplantnr zz_tech_fachbe zz_pobjnr_main )
      WITH VALUE #( ( serviceuuid = <ls_service>-serviceuuid ) )
      RESULT DATA(lt_extcustom).

    IF lt_extcustom IS NOT INITIAL.
      DATA(ls_extcustom) = lt_extcustom[ 1 ].

      IF ls_extcustom-wdplantnr IS NOT INITIAL
         AND ls_extcustom-zz_tech_fachbe IS NOT INITIAL
         AND ls_extcustom-zz_pobjnr_main IS NOT INITIAL.
        CLEAR lt_extcustom.
        CONTINUE.
      ENDIF.
    ENDIF.

    " FIXED: Use CDS view to avoid multiple record issue
    SELECT SINGLE zz_tech_fachbe, wdplantnr, zz_pobjnr_main
      FROM ewa_order_object
      WHERE pobjnr = @<ls_service>-referenceinternalid
      INTO @DATA(ls_order_fields).

    IF sy-subrc <> 0.
      CLEAR: ls_order_fields, lt_extcustom.
      CONTINUE.
    ENDIF.

    IF lt_extcustom IS INITIAL.
      " ExtCustom DOESN'T EXIST - create it
      DATA(lv_cid) = |$cid_extcust_{ <ls_service>-serviceuuid }$|.

      DATA(lv_main_position) = zcl_wr_eewa_wdoc_misc=>cl_convert_objnr_out(
                                par_pobjnr = ls_order_fields-zz_pobjnr_main ).

      APPEND VALUE #(
        serviceuuid = <ls_service>-serviceuuid
        %target = VALUE #( (
          %cid = lv_cid
          zz_tech_fachbe = ls_order_fields-zz_tech_fachbe
          wdplantnr = ls_order_fields-wdplantnr
          zz_pobjnr_main = lv_main_position
        ) )
      ) TO lt_ext_create.

    ELSE.
      " ExtCustom EXISTS - update only EMPTY fields
      DATA(lv_needs_update) = abap_false.

      DATA: lv_new_fachbe      TYPE ewa_order_object-zz_tech_fachbe,
            lv_new_wdplantnr   TYPE ewael_wdplantnr,
            lv_new_pobjnr_main TYPE zwr_d_pobjnr_main.

      IF ls_extcustom-zz_tech_fachbe IS INITIAL
         AND ls_order_fields-zz_tech_fachbe IS NOT INITIAL.
        lv_new_fachbe = ls_order_fields-zz_tech_fachbe.
        lv_needs_update = abap_true.
      ELSE.
        lv_new_fachbe = ls_extcustom-zz_tech_fachbe.
      ENDIF.

      IF ls_extcustom-wdplantnr IS INITIAL
         AND ls_order_fields-wdplantnr IS NOT INITIAL.
        lv_new_wdplantnr = ls_order_fields-wdplantnr.
        lv_needs_update = abap_true.
      ELSE.
        lv_new_wdplantnr = ls_extcustom-wdplantnr.
      ENDIF.

      " FIXED: Check zz_pobjnr_main, not wdplantnr!
      IF ls_extcustom-zz_pobjnr_main IS INITIAL
         AND ls_order_fields-zz_pobjnr_main IS NOT INITIAL.
        lv_new_pobjnr_main = zcl_wr_eewa_wdoc_misc=>cl_convert_objnr_out(
                               par_pobjnr = ls_order_fields-zz_pobjnr_main ).
        lv_needs_update = abap_true.
      ELSE.
        lv_new_pobjnr_main = ls_extcustom-zz_pobjnr_main.
      ENDIF.

      IF lv_needs_update = abap_true.
        APPEND VALUE #(
          %key-serviceuuid = <ls_service>-serviceuuid
          zz_tech_fachbe = lv_new_fachbe
          wdplantnr = lv_new_wdplantnr
          zz_pobjnr_main = lv_new_pobjnr_main
        ) TO lt_ext_update.
      ENDIF.
    ENDIF.

    CLEAR: ls_order_fields, lt_extcustom, ls_extcustom, lv_needs_update, lv_main_position.

  ENDLOOP.

  IF lt_ext_create IS NOT INITIAL.
    MODIFY ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY service
      CREATE BY \_extcustom
      AUTO FILL CID SET FIELDS WITH lt_ext_create
      MAPPED DATA(lmapped)
      FAILED DATA(lfailed)
      REPORTED DATA(lreported).
  ENDIF.

  IF lt_ext_update IS NOT INITIAL.
    MODIFY ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY extcustom
      UPDATE FIELDS ( zz_tech_fachbe wdplantnr zz_pobjnr_main )
      WITH lt_ext_update
      FAILED DATA(lt_failed_update)
      REPORTED DATA(lt_reported_update).
  ENDIF.

ENDMETHOD.

extension implementation in class zbp_e_bp_r_pdmnlservice_ext unique;

extend behavior for Service
{

  action ( precheck, features : global ) assignworkarea parameter ZAE_D_WORKAREA_AB result [0..*] $self;
  action ( precheck, features : global ) adjusttime parameter ZAE_D_adjusttime_AB result [0..*] $self;
  action ( precheck, features : global ) anlageaendern parameter ZAE_D_ANLAGE_AENDERN_AB result [0..*] $self;
  action ( precheck, features : global ) terminateservice
    parameter ZAE_D_TERMINATE_SERVICE
    result [0..*] $self;


  //determination determination PopulateExtCustomFromOrder on modify { create; update; }
  determination PopulateExtCustomFromOrder on modify
  {
    create;
    update;

  }

  side effects
  {

    action adjusttime affects $self, entity _ExtCustom;
    action assignworkarea affects $self;
    action terminateservice affects $self;
    action anlageaendern affects $self, entity _ExtCustom;
  }

}

extend behavior for ServiceTask
{
}

extend behavior for ExtCustom
{
  mapping for /plce/tpdsrvcst
    {
      zz_tech_fachbe         = zz_tech_fachbe;
      zz_vehicleinfo         = zz_vehicleinfo;
      zz_discrepancy         = zz_discrepancy;
      zz_reactiontime        = zz_reactiontime;
      zz_order_date          = zz_order_date;
      zz_timeadjustment      = zz_timeadjustment;
      zz_pobjnr_main         = zz_pobjnr_main;
      zz_informational_group = zz_informational_group;
      zz_bms_status          = zz_bms_status;
      wdplantnr              = wdplantnr;
    }

  field ( readonly : update ) zz_tech_fachbe;
  field ( readonly : update ) zz_vehicleinfo;
  field ( readonly : update ) zz_discrepancy;
  field ( readonly : update ) zz_reactiontime;
  field ( readonly : update ) zz_order_date;
  field ( readonly : update ) zz_timeadjustment;
  field ( readonly : update ) zz_pobjnr_main;
  field ( readonly : update ) zz_informational_group;
  field ( readonly : update ) WDPLANTNR;
}

extend view entity /PLCE/C_PDMNLServiceWR with
  association [0..1] to /PLCE/C_PDMNLTourWR as _TourForRefresh 
    on $projection.TourId = _TourForRefresh.TourId
      
{
  /PLCE/R_PDService._ExtCustom.zz_tech_fachbe,
  /PLCE/R_PDService._ExtCustom.zz_discrepancy,
   
  @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_WR_SERVICE_EXTEND_CALC'
  virtual service_criticality : abap.int1,
  
  /PLCE/R_PDService._ExtCustom.zz_reactiontime,
  /PLCE/R_PDService._ExtCustom.zz_vehicleinfo,
  
  @UI.hidden: true
  /PLCE/R_PDService._ExtCustom.zz_timeadjustment,
   
  @ObjectModel.virtualElement: true
  @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_WR_SERVICE_EXTEND_CALC'
  @EndUserText.label: 'Zeitanpassung in Min'
  virtual zz_timeadjust_display : abap.char(35),
  
  @ObjectModel.virtualElement: true
  @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_WR_SERVICE_EXTEND_CALC'
  @EndUserText.label: 'Hauptposition Kritikalität'
  virtual zz_main_pos_criticality : abap.int1,
  
  @EndUserText.label: 'Hauptposition'
  /PLCE/R_PDService._ExtCustom.zz_pobjnr_main,
  
    @EndUserText.label: 'Hauptposition 2'
  /PLCE/R_PDService._ExtCustom.zz_informational_group,

  @EndUserText.label: 'Entsorgungsanlage'
  @UI.lineItem: [{ position: 132, importance: #HIGH }]
  /PLCE/R_PDService._ExtCustom.order_wdplantnr,
  
  @EndUserText.label: 'BMS Status'
  /PLCE/R_PDService._ExtCustom.zz_bms_status,
  
  _TourForRefresh
}


extend view entity /PLCE/R_PDServiceExtCustom with
  association [0..1] to ZI_WR_SERVICE_EXTCUSTOM as _ExtCustomHelper
    on _ExtCustomHelper.ServiceUUID = $projection.ServiceUUID
{
  /plce/tpdsrvcst.zz_tech_fachbe,
  /plce/tpdsrvcst.zz_discrepancy,
  /plce/tpdsrvcst.zz_reactiontime,
  /plce/tpdsrvcst.zz_vehicleinfo,
  /plce/tpdsrvcst.zz_order_date,
  /plce/tpdsrvcst.zz_timeadjustment,
  _ExtCustomHelper.zz_pobjnr_main,
  _ExtCustomHelper.zz_informational_group,
  /plce/tpdsrvcst.zz_bms_status,
  /plce/tpdsrvcst.wdplantnr,
  
  _ExtCustomHelper.wdplantnr as order_wdplantnr
}


@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Service Entsorgungsanlage with fallback'
define view entity ZI_WR_SERVICE_EXTCUSTOM
  as select from /plce/tpdsrv as _srv
  left outer join /plce/tpdsrvcst as _custom
    on _custom.service_uuid = _srv.service_uuid
  left outer join ewa_order_object as _order
    on  _order.pobjnr    = _srv.reference_int_id
    and _order.wdplantnr <> ''
{
  key _srv.service_uuid as ServiceUUID,
  _order.pobjnr,
  max( case
    when _custom.wdplantnr is not initial
      then _custom.wdplantnr
    else _order.wdplantnr
  end ) as wdplantnr,

  max( case

    when _custom.zz_pobjnr_main is not initial
      and _custom.zz_pobjnr_main not like 'WE%'
      and _custom.zz_pobjnr_main not like '0%'
        then _custom.zz_pobjnr_main
    when _order.zz_pobjnr_main is not initial
      and _order.zz_pobjnr_main like 'WE%'
        then cast(
               cast(
                 substring( _order.zz_pobjnr_main, 3, 20 )
               as abap.int8 )
             as abap.char(20) )
    else ''
  end ) as zz_pobjnr_main,
  
  max( _order.zz_informational_group ) as zz_informational_group
}
group by _srv.service_uuid, _order.pobjnr


