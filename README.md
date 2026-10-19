ZI_WR_SERVICE_EXTCUSTOM, inside the braces:

max( _order.zz_informational_group ) as zz_informational_group,
The extension on /PLCE/R_PDServiceExtCustom:

_ExtCustomHelper.zz_informational_group,
The extension on /PLCE/C_PDMNLServiceWR:

@EndUserText.label: 'Informationsgruppe'
/PLCE/R_PDService._ExtCustom.zz_informational_group,
