<img width="506" height="414" alt="image" src="https://github.com/user-attachments/assets/71782ff2-835c-45b2-a350-32a59209d849" />


<img width="670" height="156" alt="image" src="https://github.com/user-attachments/assets/3cdeb8fe-f5ff-46d3-9449-1fcd569b23a0" />

<img width="1113" height="339" alt="image" src="https://github.com/user-attachments/assets/d414a690-02c8-4c82-9c1a-2a8e3b36d801" />


@EndUserText.label: 'Service Defination for WB'
define service ZSB_WR_WEIGHBRIGE {
  expose ZI_WR_R_WEIGHBRIDGE;
  expose ZI_WR_SALESITEM_CONTRACTVH;
}

managed implementation in class zbp_i_wr_r_weighbridge unique;
strict ( 2 );

define behavior for ZI_WR_R_WEIGHBRIDGE alias Weighingbridge
persistent table ZWR_WEIGHBRIDGE
lock master
authorization master ( instance )
//etag master <field_name>
{

field (readonly) SalesDocument;  //Key must be read only
action identifyCard parameter  ZAE_WR_WEIGHB_IDC result [1] $self ;
action identifyKorsel parameter  ZAE_WR_WEIGHB_KORSEL result [1] $self ;
action determineWeight parameter ZAE_WR_WEIGHB_DW  result [1] $self;
function printSlip parameter ZAE_WR_WEIGHB_PRINT  result [1] $self;
}


@EndUserText.label: 'Abstract View for determine weight'
define abstract entity ZAE_WR_WEIGHB_DW
{
      Vbeln : vbeln_va;
      Loadtype : matnr; 
}


@EndUserText.label: 'Abstract View for Identification'
define abstract entity ZAE_WR_WEIGHB_IDC
  
{
//    key Vbeln : vbeln_va;
      key Vbeln : zz1_idcardnr; 
}


@EndUserText.label: 'Abstract View for Identification'
define abstract entity ZAE_WR_WEIGHB_KORSEL
  
{
        Korselsnr : abap.char( 70 );  
}

@EndUserText.label: 'Print Slip'
define abstract entity ZAE_WR_WEIGHB_PRINT
{
     Vbeln : vbeln_va;
     Loadtype : matnr;
     Weight : abap.char(60);
     WeighUnit: abap.char(3);
}


@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Interface View for weighbridge'
@Metadata.ignorePropagatedAnnotations: true
@Metadata.allowExtensions: true
define root view entity ZI_WR_R_WEIGHBRIDGE
  as select from I_SalesDocument
  association [0..1] to zwr_weighbridge  as _HBRIDGE  on _HBRIDGE.salesdocument = $projection.SalesDocument
  {
   key SalesDocument,
   ZZ1_IDCARDNR_SDH           as IDCardNr,
   SalesOrganization          as Korselsnr,
 //  SalesDocumentType          as Korselsnrindicator,
    ZZ1_KORSELNR_MANDT_SDH     as Korselsnrindicator,
   _HBRIDGE.pdfbase64         as PDFBASE64,
    MasterSalesContract,
   CreatedByUser,
   LastChangedByUser,
   CreationDate,
   CreationTime,
   LastChangeDate,
   LastChangeDateTime  
}
where
      SalesDocumentType = 'ZKM'


 @AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Materials of Contract'
@ObjectModel.dataCategory: #VALUE_HELP
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
serviceQuality: #X,
sizeCategory: #S,
dataClass: #MIXED
}
@ObjectModel.resultSet.sizeCategory: #XS // Make it a Drop Down
define view entity ZI_WR_SALESITEM_CONTRACTVH
with parameters
P_SalesOrder : vbeln_va
as select distinct from I_SalesDocumentItem as salesitem
inner join            I_MaterialText      as _text on  salesitem.Material = _text.Material
and _text.Language     = $session.system_language
association to /watp/tavvmara      as _tavvmara      on _text.Material = _tavvmara.matnr
association to ZI_WR_R_WEIGHBRIDGE as WEIGHBRIDGEDOC on WEIGHBRIDGEDOC.SalesDocument = salesitem.SalesDocument
{
//    @UI.hidden: true
key salesitem.SalesDocument     as SalesOrder,
//    @UI.hidden: true
key salesitem.SalesDocumentItem as SalesOrderitem,
key _tavvmara.avvcode           as Avvcode,
_text.Material              as Material,
_text.MaterialName          as MaterialText,
//    @UI.hidden: true
_text.Language              as Language,
WEIGHBRIDGEDOC.Korselsnr as Korselsnr,
WEIGHBRIDGEDOC.Korselsnrindicator as Korselsnrindicator,
_tavvmara
}
where
salesitem.SalesDocument = $parameters.P_SalesOrder

@EndUserText.label : 'Weighing Bridge'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
@AbapCatalog.tableCategory : #TRANSPARENT
@AbapCatalog.deliveryClass : #A
@AbapCatalog.dataMaintenance : #ALLOWED
define table zwr_weighbridge {

  key mandt            : mandt not null;
  key salesdocument    : vbeln_va not null;
  sddocumentcategory   : vbtypl;
  salesdocumenttype    : auart;
  pdfbase64            : dstring;
  ernam                : ernam;
  last_changed_by_user : last_changed_by_user;
  aedat                : aedat;
  erzet                : erzet;
  upd_tmstmp           : timestampl;
  mahdt                : mahdt;

}


@EndUserText.label : 'Weighing Session Draft'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
@AbapCatalog.tableCategory : #TRANSPARENT
@AbapCatalog.deliveryClass : #A
@AbapCatalog.dataMaintenance : #ALLOWED
define table zwr_weighb_dd {

  key mandt       : mandt not null;
  key sessionid   : zwr_sessionid not null;
  vbeln           : vbeln_va;
  loadtype        : matnr;
  pdfbase64       : dstring;
  @Semantics.quantity.unitOfMeasure : '/watp/tbcwghbill.grossweightunit'
  grossweight     : /watp/dbcgrossweight;
  grossweightunit : /watp/dbcweightunit;
  @Semantics.quantity.unitOfMeasure : '/watp/tbcwghbill.tareweightunit'
  tareweight      : /watp/dbctareweight;
  tareweightunit  : /watp/dbcweightunit;
  @Semantics.quantity.unitOfMeasure : '/watp/tbcwghbill.netweightunit'
  netweight       : /watp/dbcnetweight;
  netweightunit   : /watp/dbcweightunit;
  step            : zwr_sessionstep;
  step1ok         : boole_d;
  step2ok         : boole_d;
  issummited      : boole_d;
  include sych_bdl_draft_admin_inc;


@EndUserText.label : 'Weighing Session'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
@AbapCatalog.tableCategory : #TRANSPARENT
@AbapCatalog.deliveryClass : #A
@AbapCatalog.dataMaintenance : #ALLOWED
define table zwr_weighsession {

  key mandt       : mandt not null;
  key sessionid   : zwr_sessionid not null;
  vbeln           : vbeln_va;
  loadtype        : matnr;
  pdfbase64       : dstring;
  @Semantics.quantity.unitOfMeasure : '/watp/tbcwghbill.grossweightunit'
  grossweight     : /watp/dbcgrossweight;
  grossweightunit : /watp/dbcweightunit;
  @Semantics.quantity.unitOfMeasure : '/watp/tbcwghbill.tareweightunit'
  tareweight      : /watp/dbctareweight;
  tareweightunit  : /watp/dbcweightunit;
  @Semantics.quantity.unitOfMeasure : '/watp/tbcwghbill.netweightunit'
  netweight       : /watp/dbcnetweight;
  netweightunit   : /watp/dbcweightunit;
  step            : zwr_sessionstep;
  step1ok         : boole_d;
  step2ok         : boole_d;
  issummited      : boole_d;

}

@EndUserText.label : 'Weighing Session Draft'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
@AbapCatalog.tableCategory : #TRANSPARENT
@AbapCatalog.deliveryClass : #A
@AbapCatalog.dataMaintenance : #ALLOWED
define table zwr_weighsessi_d {

  key mandt          : mandt not null;
  key sessionid      : zwr_sessionid not null;
  salesdocument      : vbeln_va;
  sddocumentcategory : vbtypl;
  salesdocumenttype  : auart;
  lastchangeon       : aedat;
  include sych_bdl_draft_admin_inc;

}


