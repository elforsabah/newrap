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
}
group by _srv.service_uuid, _order.pobjnr
