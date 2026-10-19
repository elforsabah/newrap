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
