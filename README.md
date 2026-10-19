@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Helper CDS for EWA-ORDER_ObJECT'
@Metadata.ignorePropagatedAnnotations: true
define root view entity ZI_WR_EWA_ORDER_OBJECT as select distinct from ewa_order_object as _EWA_ORDER_OBJECT
association [0..1] to /PLCE/R_PDService  as _ServiceCore on _ServiceCore.ReferenceInternalId = $projection.pobjnr

{
    key pobjnr,
    _ServiceCore.ServiceUUID,
    wdplantnr , //Entsorgunganlage,
    zz_pobjnr_main , // Hauptposition
    zz_informational_group,
    zz_order_date , //
    _ServiceCore.ReferenceId,
    zz_reactiontime,
    zz_tech_fachbe
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
  end ) as zz_pobjnr_main

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
  /plce/tpdsrvcst.zz_bms_status,
  /plce/tpdsrvcst.wdplantnr,
  
  _ExtCustomHelper.wdplantnr as order_wdplantnr
}

}
group by _srv.service_uuid, _order.pobjnr
