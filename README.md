Bitte folgenden Teil des Anpassungsvorschlags umsetzen (die volle Anpassung ist im angehangenen Word Dokument zu finden).

Die Datengrundlage (Implementierung im Interaction Center etc.) entwickelt Michael Ulrich unter #40516.

 

Anpassungsvorschlag:

 

Darstellung und Bearbeitung in der Disposition:

In der Disposition muss eine Spalte für das neue Z-Feld eingefügt werden, in dem die Nummer der Hauptposition enthalten ist. Diese muss auch angezeigt werden können. Die Services können über eine eingespeicherte Sicht nach dieser Nummer gruppiert dargestellt werden (Standardoption).

Da diese Sicht nicht zwangsläufig immer ausgewählt sein muss, soll auch farblich grün hervorgehoben werden, wenn ein Service weitere zusammengehörige Services hat. Ist für einen Service hinterlegt, dass er zu einer Gruppe gehört (unabhängig ob kaufmännisch oder informativ), soll dieser Service in der Farbe „grün“ hervorgehoben werden. Diese Färbung dient als Kennzeichen die gruppierte Sicht auszuwählen. Damit ist sichergestellt, dass die Aufträge zusammen dargestellt werden. Der Disponent kann so sehen, welche Aufträge gemeinsam disponiert werden sollen. Die Entscheidungshoheit bleibt aber bei der Disposition, wann welcher Auftrag gefahren wird.


<img width="460" height="457" alt="image" src="https://github.com/user-attachments/assets/17a0df1d-59e7-4fc2-b184-a433912ee5bb" />


<img width="476" height="225" alt="image" src="https://github.com/user-attachments/assets/6193cd98-cea2-4b24-a12f-8ca53c270cde" />

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
  /plce/tpdsrvcst.wdplantnr,
  /plce/tpdsrvcst.zz_bms_status,
  
  _ExtCustomHelper.wdplantnr as order_wdplantnr
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
  
//  @EndUserText.label: 'Hauptposition'
//  /PLCE/R_PDService._ExtCustom.zz_pobjnr_main,
  
    @EndUserText.label: 'Hauptposition'
  /PLCE/R_PDService._ExtCustom.zz_informational_group,

  @EndUserText.label: 'Entsorgungsanlage'
  @UI.lineItem: [{ position: 132, importance: #HIGH }]
  /PLCE/R_PDService._ExtCustom.order_wdplantnr,
  _TourForRefresh
}



@EndUserText.label: 'Service WR'
@AccessControl.authorizationCheck: #NOT_REQUIRED
@Metadata.allowExtensions: true
@Search.searchable: true
@ObjectModel.usageType:{
  serviceQuality: #X,
  sizeCategory: #XL,
  dataClass: #MIXED
}
define root view entity /PLCE/C_PDMNLServiceWR
  provider contract transactional_query
  as projection on /PLCE/R_PDService
  association [1]    to /PLCE/C_PDMNLService          as _ServiceCore       on _ServiceCore.ServiceUUID = $projection.ServiceUUID
  //association [1] to /PLCE/C_PDServiceC as _ServiceCore on _ServiceCore.ServiceUUID = $projection.ServiceUUID
  association [0..*] to /PLCE/C_PDCFServiceConfNoteWR as _ServiceConfNoteWR on _ServiceConfNoteWR.ServiceUUID = $projection.ServiceUUID
  association [0..*] to /PLCE/C_PDCFServiceDocResWR   as _ServiceDocumentWR on _ServiceDocumentWR.ServiceUUID = $projection.ServiceUUID
{
//          @UI.hidden: true
  key     ServiceUUID,

          @Search:{ defaultSearchElement: true, fuzzinessThreshold: 0.7 }
          //          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDService_VH', element: 'ServiceId' } }]
          ServiceId,
          @Consumption.valueHelpDefinition: [{ entity : { name: '/PLCE/C_PDProfileWA_VH', element: 'Profile' } }]
          Profile,

          @ObjectModel.text.element: ['ServiceTypeDescription']
          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDServiceTypeWA_VH', element: 'ServiceType'  } }]
          ServiceType,

          @UI.hidden: true
          _ServiceType._ServiceTypeText.ServiceTypeDescription                          as ServiceTypeDescription      : localized,


          @EndUserText.label: 'Service Status'
          @ObjectModel.text.element: ['ServiceStatusText']
          @UI.textArrangement: #TEXT_ONLY
          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDServiceStatus_VH', element: 'ServiceStatus' } }]
          ServiceStatus,
          @UI.hidden: true
          _ServiceStatusText.Description                                                as ServiceStatusText           : localized,
          @Consumption.filter.hidden: true
          _ServiceStatusText.IconURL                                                    as ServiceStatusIcon           : localized,

          @Search:{ defaultSearchElement: true  }
          @Consumption.valueHelpDefinition: [{ entity : { name: '/PLCE/C_PDOrderWAWR_VH', element: 'ReferenceId' } }]
          ReferenceId,
          @Consumption.valueHelpDefinition: [{ entity : { name: '/PLCE/C_PDServicePriority_VH', element: 'ServicePriority' } }]
          ServicePriority,

          @UI.hidden: true
          _PlanningStatusText.Description                                               as PlanningStatusText          : localized,

          @ObjectModel.text.element: ['PlanningStatusText']
          @UI.textArrangement: #TEXT_ONLY
          @Consumption.valueHelpDefinition: [{ entity : { name: '/PLCE/C_PDPlanningStatus_VH', element : 'PlanningStatus'} }]
          @Consumption.filter: { defaultValue: '', multipleSelections: true, selectionType: #SINGLE }
          PlanningStatus,

          @UI.hidden: true
          _ServiceLookup.PlanningStatusCriticality                                      as PlanningStatusCriticality,

          @Consumption.filter: { mandatory: true, hidden: false, selectionType: #INTERVAL }
          RequestedDate,
          @Consumption.filter.selectionType: #INTERVAL
          EarliestDate,
          @Consumption.filter.selectionType: #INTERVAL
          LatestDate,

          @Search:{ defaultSearchElement: true, fuzzinessThreshold: 0.7 }
          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDCustomerInfoWA_VH', element: 'CustomerInfo'} }]
          CustomerInfo                                                                  as CustomerInfo,
          @UI.multiLineText: true
          AdditionalText,

          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDFuncLocServiceWA_VH', element: 'FunctionalLocation' } }]
          @EndUserText.label: 'Customer Location'
          FunctionalLocation,
          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDFuncLocServiceWA_VH', element: 'FullAddress' } }]
          _FunctionalLocation.FullAddress                                               as FullAddress,

          @Consumption.valueHelpDefinition: [{ entity: { name : '/PLCE/C_MDPLANT_VH', element: 'Plant' } }]
          PlanningPlant,

          @UI.hidden: true
          @Semantics.unitOfMeasure: true
          cast( _Statistic.TotalDurationUnit as meins preserving type )                 as TotalDurationUnit, //show localized key/allow fiori app value check
          @Semantics.quantity.unitOfMeasure: 'TotalDurationUnit'
          @Consumption.filter.hidden: true
          _Statistic.TotalDuration,

          @EndUserText.label: 'Work Status'
          @ObjectModel.text.element: ['WorkStatusText']
          @UI.textArrangement: #TEXT_ONLY
          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDWorkStatus_VH', element: 'Value' } }]
          _ServiceLookup.WorkStatus,
          @UI.hidden: true
          _ServiceLookup._WorkStatusText.Description                                    as WorkStatusText              : localized,
          @Consumption.filter.hidden: true
          _ServiceLookup._WorkStatusText.IconURL                                        as WorkStatusIcon              : localized,

          @ObjectModel.text.element: ['ServiceWindowText']
          @Consumption.valueHelpDefinition: [{ entity : { name: '/PLCE/C_PDServiceWindow_VH', element : 'ServiceWindow'} }]
          @UI.textArrangement: #TEXT_ONLY
          ServiceWindow,
          @UI.hidden: true
          _ServiceWindowText[1:Language = $session.system_language].Description         as ServiceWindowText,
          @Consumption.filter.selectionType: #SINGLE
          ServiceWindowStartTime,
          @Consumption.filter.selectionType: #SINGLE
          ServiceWindowEndTime,

          @Consumption.filter.hidden: true
          _TourAssignments[1:ServiceUUID = $projection.serviceuuid].TourId              as TourId,

          //Extension Waste
          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_MDMATERIAL_VH', element: 'ResourceKey'} }]
          @Search:{ defaultSearchElement: true } // fuzzinessThreshold: 0.8, ranking: #LOW
          _ExtWaste.Material                                                            as Material,
          @ObjectModel.text.element: ['MaterialText']
          @UI.textArrangement: #TEXT_LAST
          @Consumption.filter.hidden: true
          _ExtWaste.Material                                                            as MaterialWithText,

          @EndUserText.label: 'Material Description'
          @Consumption.filter.hidden: true
          _ExtWaste._Material._Text.MaterialName                                        as MaterialText                : localized,
          @Semantics.quantity.unitOfMeasure:'MaterialWeightUnit'
          @Consumption.filter.hidden: true
          _ExtWaste.MaterialWeight                                                      as MaterialWeight,
          @UI.hidden: true
          cast( _ExtWaste.MaterialWeightUnit as meins preserving type )                 as MaterialWeightUnit, //show localized key/allow fiori app value check
          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_MaterialGroup_VH', element: 'MaterialGroup'} }]
          @ObjectModel.text.element: [ 'MaterialGroupText' ]
          @EndUserText.label: 'Material Group'
          @UI.textArrangement: #TEXT_LAST
          _ExtWaste._Material.MaterialProductGroup                                      as MaterialGroup,
          @EndUserText.label: 'Material Group Description'
          @UI.hidden: true
          _ExtWaste._Material._ProductGroup._ProductGroupText.ProductGroupName          as MaterialGroupText           : localized,

          @ObjectModel.text.element: ['PlantLocationText']
          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDNoCustomerLocWA_VH', element: 'FunctionalLocation' } }]
          @EndUserText.label: 'Plant Location'
          _ExtWaste.PlantLocation                                                       as PlantLocation,
          @UI.hidden: true
          _ExtWaste._PlantLocation._FuctionalLocationText.FunctionalLocationDescription as PlantLocationText           : localized,

          @ObjectModel.text.element: ['ContainerSourceAddress']
          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDFuncLocWA_VH', element: 'FunctionalLocation'} }]
          @EndUserText.label: 'Address - Container Source'
          _ExtWaste.ContainerSourceLocation                                             as ContainerSourceLocation,
          @UI.hidden: true
          _ExtWaste._ContainerSourceLocation.FullAddress                                as ContainerSourceAddress,
          @ObjectModel.text.element: ['ContainerFinalAddress']
          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDFuncLocWA_VH', element: 'FunctionalLocation'} }]
          @EndUserText.label: 'Address - Container Final'
          _ExtWaste.ContainerFinalLocation                                              as ContainerFinalLocation,
          @UI.hidden: true
          _ExtWaste._ContainerFinalLocation.FullAddress                                 as ContainerFinalAddress,

          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_MDTransptPackaggT_VH', element: 'ResourceKey' } }]
          //          @Search:{ defaultSearchElement: true, fuzzinessThreshold: 0.8, ranking: #LOW }
          @ObjectModel.text.element: ['ContainerTypeAtLocationText']
          _ExtWaste.ContainerTypeAtLocation                                             as ContainerTypeAtLocation,
          @ObjectModel.text.element: ['ContainerTypeAtLocationText']
          @UI.textArrangement: #TEXT_LAST
          @Consumption.filter.hidden: true
          _ExtWaste.ContainerTypeAtLocation                                             as ContainerTypeAtLocationWiText,

          @UI.hidden: true
          _ExtWaste._ContainerTypeAtLocation._Text.TransportPackagingTypeName           as ContainerTypeAtLocationText : localized,

          _ExtWaste.ContainerAtLocationCount                                            as ContainerAtLocationCount,
          _ExtWaste.ContainerAtLocationTidnr                                            as ContainerAtLocationTidnr,

          @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_MDTransptPackaggT_VH', element: 'ResourceKey' } }]
          //          @Search:{ defaultSearchElement: true, fuzzinessThreshold: 0.8, ranking: #LOW }
          @ObjectModel.text.element: ['ContainerTypeNewText']
          _ExtWaste.ContainerTypeNew                                                    as ContainerTypeNew,
          @ObjectModel.text.element: ['ContainerTypeNewText']
          @UI.textArrangement: #TEXT_LAST
          @Consumption.filter.hidden: true
          _ExtWaste.ContainerTypeNew                                                    as ContainerTypeNewWithText,

          @UI.hidden: true
          _ExtWaste._ContainerTypeNew._Text.TransportPackagingTypeName                  as ContainerTypeNewText        : localized,
          _ExtWaste.ContainerNewCount                                                   as ContainerNewCount,
          _ExtWaste.ContainerNewTidnr                                                   as ContainerNewTidnr,
          @UI.hidden: true
          _ExtWaste.ServiceFrequency                                                    as ServiceFrequency,
          @EndUserText.label: 'Service Frequency'
          _ExtWaste._ServiceFrequencyText.ServiceFrequencyText                          as ServiceFrequencyText        : localized,

          @EndUserText.label: 'Replanned'
          _ServiceDeviation[1:ReplannedFlag is not initial].ReplannedFlag               as ReplannedFlag,
          //map
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_SERVICE_CALC'
  virtual MapTitle      : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_SERVICE_CALC'
  virtual MapPopup      : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_SERVICE_CALC'
  virtual MapColor      : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_SERVICE_CALC'
  virtual MapHighlight  : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_SERVICE_CALC'
  virtual MapSymbol     : abap.string,

          /* Associations */
          _ServiceTask        : redirected to composition child /PLCE/C_PDMNLServiceTaskWR,
          //_Attachments        : redirected to composition child /PLCE/C_PDMNLSERVICEATTACHMENT,
          _FunctionalLocation : redirected to /PLCE/C_PDMNLFuncLoc,
          _WorkAreaServices,
          _TourAssignments, //    : redirected to /PLCE/C_PDMNLTourServiceAsgWR,
          //_PlanningStatusText,
          _ServiceCore,
          _ServiceConfNoteWR,
          _ServiceDocumentWR
}

@EndUserText.label: 'Tour WR'
@AccessControl.authorizationCheck: #NOT_REQUIRED
@Search.searchable: true
@Metadata.allowExtensions: true
@ObjectModel.usageType:{
  serviceQuality: #X,
  sizeCategory: #XL,
  dataClass: #MIXED
}
define root view entity /PLCE/C_PDMNLTourWR
  provider contract transactional_query
  as projection on /PLCE/R_PDTour
  association [1] to /PLCE/C_PDMNLTour as _TourCore on _TourCore.TourUUID = $projection.TourUUID
{
  key    TourUUID,
         @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDTourWA_VH', element: 'TourId' } }]
         @Search: { defaultSearchElement: true, ranking: #HIGH }
         TourId,

         @EndUserText.label: 'Tour Template'
         @ObjectModel.text.element: ['TourTemplateName']
         @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDTourTemplateWA_VH', element: 'TourTemplate' } }]
         @Search: { defaultSearchElement: true }
         TourTemplate,

         @EndUserText.label: 'Template Description'
         @Consumption.filter.hidden: true
         _TourTemplate._Text.TourTemplateName                                            as TourTemplateName : localized,

         @UI.hidden: true
         _TourTemplate.ColorTour                                                         as ColorTour,

         @EndUserText.label: 'Tour Status'
         @ObjectModel.text.element: ['TourStatusText']
         @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDTourStatus_VH', element: 'TourStatus' } }]
         @Consumption.filter: { multipleSelections: true, selectionType: #SINGLE }
         TourStatus,
         @Semantics.text: true
         @UI.hidden: true
         _PDTourStatusText.Description                                                   as TourStatusText   : localized,
         @UI.hidden: true
         _TourLookup.TourStatusColorValue                                                as TourStatusColorValue,
         @Consumption.filter.hidden: true
         _PDTourStatusText.IconURL                                                       as TourStatusIcon   : localized,

         @Consumption.filter: { mandatory: true, hidden: false, selectionType: #INTERVAL }
         StartDate                                                                       as TourStartDate,
         @Consumption.filter.selectionType: #INTERVAL
         EndDate                                                                         as TourEndDate,
         @Consumption.filter.selectionType: #INTERVAL
         ScheduledDateTimeStart                                                          as ScheduledDateTimeStart,

         @Semantics.quantity.unitOfMeasure: 'CalculatedTourDurationUnit'
         @Consumption.filter.hidden: true
         CalculatedTourDuration                                                          as TourDuration,
         @UI.hidden: true
         cast( CalculatedTourDurationUnit as meins preserving type )                     as CalculatedTourDurationUnit, //show localized key/allow fiori app value check
         @Consumption.filter.hidden: true
         CalculatedTourCapacity                                                          as TourCapacity,
//         @Consumption.filter.hidden: true
         @UI.hidden: true
         _TourLookup.TourCapacityColorValue                                              as TourCapacityColorValue,

         @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDResource_VH', element: 'ResourceId'} }]
         MainResourceId                                                                  as MainResource,

         @Consumption.filter.selectionType: #SINGLE
         EarliestStartTime,
         //LatestStartTime,

         @Semantics.quantity.unitOfMeasure: 'MaximumTourDurationUnit'
         @Consumption.filter.hidden: true
         MaximumTourDuration,
         @UI.hidden: true
         cast( MaximumTourDurationUnit as meins preserving type )                        as MaximumTourDurationUnit, //show localized key/allow fiori app value check

         @EndUserText.label: 'Start Address'
         @Semantics.text: true
         @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDNoCustomerLocWA_VH', element: 'FullAddress' },
                                              label            : 'Start Functional Location'
                                              }]
         _PDStartFunctionalLocation.FullAddress                                          as StartFLocFullAddress,
         @UI.hidden: true
         _PDStartFunctionalLocation._FuctionalLocationText.FunctionalLocationDescription as StartFLocName    : localized,
         @UI.hidden: true //@Consumption.filter.hidden: true
         _PDStartFunctionalLocation.Latitude                                             as StartLatitude,
         @UI.hidden: true //@Consumption.filter.hidden: true
         _PDStartFunctionalLocation.Longitude                                            as StartLongitude,

         @EndUserText.label: 'End Address'
         @Semantics.text: true
         @ObjectModel.text.element: [ 'EndFLocFullAddress' ]
         @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDNoCustomerLocWA_VH', element: 'FullAddress' },
                                              label            : 'End Functional Location'
                                              }]
         _PDEndFunctionalLocation.FullAddress                                            as EndFLocFullAddress,
         @UI.hidden: true
         _PDEndFunctionalLocation._FuctionalLocationText.FunctionalLocationDescription   as EndFLocName      : localized,
         @UI.hidden: true //@Consumption.filter.hidden: true
         _PDEndFunctionalLocation.Latitude                                               as EndLatitude,
         @UI.hidden: true //@Consumption.filter.hidden: true
         _PDEndFunctionalLocation.Longitude                                              as EndLongitude,

          //map
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_TOUR_LOC_S_CALC'
  virtual StartMapTitle     : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_TOUR_LOC_S_CALC'
  virtual StartMapPopup     : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_TOUR_LOC_S_CALC'
  virtual StartMapColor     : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_TOUR_LOC_S_CALC'
  virtual StartMapHighlight : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_TOUR_LOC_S_CALC'
  virtual StartMapSymbol    : abap.string,

          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_TOUR_LOC_E_CALC'
  virtual EndMapTitle       : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_TOUR_LOC_E_CALC'
  virtual EndMapPopup       : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_TOUR_LOC_E_CALC'
  virtual EndMapColor       : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_TOUR_LOC_E_CALC'
  virtual EndMapHighlight   : abap.string,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:/PLCE/CL_PDMNL_TOUR_LOC_E_CALC'
  virtual EndMapSymbol      : abap.string,

         ExternalTourID,

         @UI.hidden: true
         CreatedForSimulation,

         //         PDProfile,
         @EndUserText.label: 'Route available'
         _TourLookup.IsGeoRouteAvailable,
         @EndUserText.label: 'Work Status'
         @UI.textArrangement: #TEXT_ONLY
         @ObjectModel.text.element: ['WorkStatusText']
         @Consumption.valueHelpDefinition: [{ entity: { name: '/PLCE/C_PDWorkStatus_VH', element: 'Value'} }]
         _TourLookup.WorkStatus,
         @UI.hidden: true
         _TourLookup._WorkStatusText.Description                                         as WorkStatusText   : localized,
         @Consumption.filter.hidden: true
         _TourLookup._WorkStatusText.IconURL                                             as WorkStatusIcon   : localized,

         @EndUserText.label: 'Driving Time'
         @Consumption.filter.hidden: true
         _TourLookup.DrivingTime                                                         as DrivingTime,
         @UI.hidden: true
         _TourLookup.DrivingTimeUnit                                                     as DrivingTimeUnit,

         @UI.hidden: true
         _TourLookup._TourCriticality.TourCriticality                                    as TourCriticality,
         @EndUserText.label: 'Deviations on Tour'
         _TourLookup._TourCriticality.DeviationsOnTour                                   as DeviationsOnTour,

         @UI.hidden: true
         _TourLookup._TourLookupWR.WeightUnit                                            as WeightUnit,
         @Consumption.filter.hidden: true
         _TourLookup._TourLookupWR.MaterialWeightInTOSum                                 as MaterialWeightInTOSum,
         @UI.hidden: true
         @Semantics.quantity.unitOfMeasure: 'WeightUnit'
         _TourLookup._TourLookupWR.VehicleCapacityInTOSum                                as VehicleCapacityInTOSum,
         @Consumption.filter.hidden: true
         @UI.hidden: true
         _TourLookup._TourLookupWR.CriticalTonnage                                       as CriticalityTonnage,

         @UI.hidden: true
         _TourLookup._TourLookupWR.CountServices                                         as CountServices,
         @UI.hidden: true
         _TourLookup._TourLookupWR.CountServicesCompleted                                as CountServicesCompleted,

         /*Associations*/
         _TourCore,
         _ServiceAssignments  : redirected to composition child /PLCE/C_PDMNLTourServiceAsgWR,
         _TaskAssignments     : redirected to composition child /PLCE/C_PDMNLTourTaskAsgmtWR,
         _Resources           : redirected to /PLCE/C_PDMNLTourResource,
         _Attachments         : redirected to /PLCE/C_PDMNLTourAttachment,
         _FunctionalLocations : redirected to /PLCE/C_PDMNLTourFuncLoc,
         _GeoRoutes           : redirected to /PLCE/C_PDMNLTourGeoRoute,
         _WorkAreaTours
         //         /PLCE/R_PDTour._Profiles as _Profiles

}

@EndUserText.label: 'Abstract View für Arbeitbereich'

define abstract entity ZAE_D_adjusttime_AB
 
{
  @EndUserText.label: 'Zeitanpassung'
   ZeitInMin : abap.char( 35 ); 
}

@EndUserText.label: 'Abstract View für Entsorgungsanlage'

define abstract entity ZAE_D_ANLAGE_AENDERN_AB
 
{
  @EndUserText.label: 'Entsorgungsanlage'
 @Consumption.valueHelpDefinition: [
    {
      entity         : { name: 'ZAE_D_ENTSORGUNG_ANLAGE_VH', element: 'Entsorgungsanlage' },
      label          : 'Entsorgungsanlage'
    }
  ]
   @ObjectModel.mandatory: true
   anlage : ewael_wdplantnr; 
}

@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Value Help für Entsorgungsanlage'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
@ObjectModel.resultSet.sizeCategory: #XS
define view entity ZAE_D_ENTSORGUNG_ANLAGE_VH as select from ewa_el_wdplant
{
        
   @EndUserText.label: 'Entsorgungsanlage'
   key wdplantnr as Entsorgungsanlage     
}  where zz_available_dispo is not initial
  
@EndUserText.label: 'Service Storno'
define abstract entity ZAE_D_ServiceStorno_AB
  
{
    serviceDoc : char50;
    
}

@EndUserText.label: 'Abstract View für Arbeitbereich'
define abstract entity ZAE_D_TERMINATE_SERVICE
{

   @EndUserText.label: 'Grund: STORNO DURCH DISPO (Bemerkung)'
//  @UI.multiLineText: true
  @ObjectModel.mandatory: true
    @Consumption.valueHelpDefinition: [
    {
      entity            : { name: 'ZAE_D_TERMINATE_SERVICE_VH', element: 'Bemerkung' },
     
     useForValidation: false
    }
  ]
  
  stornogrund : abap.char( 255 );
}

@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Value Help für Storno'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}

define view entity ZAE_D_TERMINATE_SERVICE_VH as select from zwr_pd_storno_bm
{

        @ObjectModel.text.element: [ 'Bemerkung' ]
        @UI.textArrangement: #TEXT_ONLY  
        @UI.hidden: true
        key conftype as Conftype,
        @EndUserText.label: 'Bermerkungen'
        bemerkung as Bemerkung   
}
  

@EndUserText.label: 'Abstract View für Tour Create'

define abstract entity ZAE_D_TOURTOURTEMPLATE_AB

{
//  @EndUserText.label: 'Tour Template'
  @Consumption.valueHelpDefinition: [
     {
       entity   : { name: '/PLCE/C_PDTourTemplate_VH', element: 'TourTemplate' },
       label    : 'TourTemplate',
       useForValidation: true
     }
   ]
  tour_template : /plce/pdtour_template;

//  @EndUserText.label: 'First Tour Date'
  start_date    : zz_tour_start_date;

//  @EndUserText.label: 'Last Tour Date'
  end_date      : zz_tour_end_date;
}

@EndUserText.label: 'Abstract View für Arbeitbereich'

define abstract entity ZAE_D_WORKAREA_AB
 
{
  @EndUserText.label: 'Neue Arbeitsbereich'
//  @UI.defaultValue: #( 'ELEMENT_OF_REFERENCED_ENTITY: ServiceType' )
 @Consumption.valueHelpDefinition: [
    {
      entity         : { name: 'ZZPD_WORKREA_VH', element: 'WorkArea' },
      label          : 'Workarea',
      useForValidation: true
    }
  ]
   WorkArea : /plce/pdwork_area; 
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
  
//  @EndUserText.label: 'Hauptposition'
//  /PLCE/R_PDService._ExtCustom.zz_pobjnr_main,
  
    @EndUserText.label: 'Hauptposition'
  /PLCE/R_PDService._ExtCustom.zz_informational_group,

  @EndUserText.label: 'Entsorgungsanlage'
  @UI.lineItem: [{ position: 132, importance: #HIGH }]
  /PLCE/R_PDService._ExtCustom.order_wdplantnr,
  _TourForRefresh
}

extend view entity /PLCE/C_PDMNLTourWR with {
   /PLCE/R_PDTour._ExtCustom.zz_bms_status , // BMS Status
 
  @ObjectModel.virtualElement: true
  @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_WR_TOUR_EXTEND_CALC'
  @EndUserText.label: 'BMS Criticality'
  @UI.hidden: true
  virtual ZzBmsCriticality : abap.int1,
 
   // 1. Used Capacity (MUST BE DEC FOR PROGRESS BAR TO WORK)
   @ObjectModel.virtualElement: true
   @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_WR_TOUR_EXTEND_CALC'
//   @EndUserText.label: 'Kapazitätsauslastung'
 
//   @UI.hidden: true
   virtual tour_capacity_new : abap.dec( 10, 2 ), 

   // 2. Total Duration (TARGET - Link Unit HERE)
   @ObjectModel.virtualElement: true
   @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_WR_TOUR_EXTEND_CALC'
   @Semantics.quantity.unitOfMeasure: 'zz_duration_unit' 
//   @UI.hidden: true
   virtual tour_duration_min : abap.quan( 10, 2 ),      

   // 3. Unit Field (Holds 'MIN')
   @ObjectModel.virtualElement: true
   @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_WR_TOUR_EXTEND_CALC'
   @Semantics.unitOfMeasure: true
   @UI.hidden: true
   virtual zz_duration_unit : abap.unit( 3 ),

   // 4. Remaining Duration (Standard display)
   @ObjectModel.virtualElement: true
   @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_WR_TOUR_EXTEND_CALC'
//   @EndUserText.label: 'Restdauer'
   @Semantics.quantity.unitOfMeasure: 'zz_duration_unit'
//   @UI.hidden: true
   virtual zz_remaining_duration : abap.quan( 10, 2 ),
   
   
   // 4. MaximumTourDuration converted (Standard display)
   @ObjectModel.virtualElement: true
   @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_WR_TOUR_EXTEND_CALC'
//   @EndUserText.label: 'Converted MaximumTourDuration '
   @Semantics.quantity.unitOfMeasure: 'zz_duration_unit'
   @UI.hidden: true
   virtual MaximumTourDuration_Converted : abap.quan( 10, 2 ),
   
   @ObjectModel.virtualElement: true
   @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_WR_TOUR_EXTEND_CALC'
   @UI.hidden: true
   virtual zz_capacity_criticality : abap.int1,
   
   // Add this to your existing extension:
   @ObjectModel.virtualElement: true
   @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_WR_TOUR_EXTEND_CALC'
   @EndUserText.label: 'Fahrzeit'  // Driving Time
   @Semantics.quantity.unitOfMeasure: 'zz_duration_unit'
   virtual zz_driving_time_min : abap.quan( 10, 2 )
   
}

 @AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Extended Result Work Area - Service'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
serviceQuality: #C,
sizeCategory: #XL,
dataClass: #TRANSACTIONAL
}
define  view entity ZE_P_PDWorkAreaServiceExt
as select from ZE_P_PDWorkAreaServiceFuncExt( p_client: $session.client )
association to parent /PLCE/R_PDServiceExtCustom as _ExtCustom
on $projection.ServiceUUID = _ExtCustom.ServiceUUID
{
@Consumption.valueHelpDefinition: [{
entity: {
name    : '/PLCE/C_PDWorkAreaUser_VH',
element : 'WorkArea'
},
label     : 'Work Area',
useForValidation: true
}]
key work_area    as WorkArea,
key service_uuid as ServiceUUID,
    _ExtCustom.zz_tech_fachbe as Fachbereich,
_ExtCustom  // To-parent association
} where work_area =  _ExtCustom.zz_tech_fachbe


@EndUserText.label: 'Table Function Extension'
@AccessControl.authorizationCheck: #NOT_REQUIRED
@ClientHandling.type: #CLIENT_DEPENDENT
@ClientHandling.algorithm: #SESSION_VARIABLE

define table function ZE_P_PDWorkAreaServiceFuncExt
with parameters
    @Environment.systemField: #CLIENT
    p_client : abap.clnt
returns
{
  mandt : abap.clnt;
  work_area : /plce/pdwork_area;
  service_uuid : /plce/pdservice_uuid;
}
implemented by method
  zcl_pd_workarea_funct_ext=>GET_WORKAREAS_SERVICES_EXT;  // New custom class/method

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
  /plce/tpdsrvcst.wdplantnr,
  /plce/tpdsrvcst.zz_bms_status,
  
  _ExtCustomHelper.wdplantnr as order_wdplantnr
}

extend view entity /PLCE/R_PDTourExtCustom with {
   /plce/tpdtourcst.zz_remaining_duration,
   /plce/tpdtourcst.zz_bms_status
}


@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Aggregated Task Durations per Tour'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
  serviceQuality: #X,
  sizeCategory: #S,
  dataClass: #MIXED
}
define view entity ZI_TOUR_TASK_SUM_WR
  as select from /PLCE/R_PDTourTaskAssignment
{
  key TourUUID,
      
      @Semantics.quantity.unitOfMeasure: 'DurationUnit'
      sum(Duration) as TotalTaskDuration,
      
      DurationUnit
}where TourTaskUUID <> hextobin('00000000000000000000000000000000') and TourTaskUUID is not null
group by
  TourUUID,
  DurationUnit


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
  end ) as zz_pobjnr_main,
  
  max( case when _custom.zz_informational_group is not initial then _custom.zz_informational_group else _order.zz_informational_group end ) as zz_informational_group
}
group by _srv.service_uuid, _order.pobjnr


@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Abstract View für Arbeitbereich'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.dataCategory: #VALUE_HELP

@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
define view entity ZZPD_WORKREA_VH as select from /PLCE/R_PDWorkArea
{
   
   key WorkArea as WorkArea
}


@Metadata.layer: #CORE
annotate entity /PLCE/C_PDTourTemplate
    with 
{  @UI.lineItem: [
    { position: 190, importance: #HIGH },
    {
      type                : #FOR_ACTION,
      dataAction          : 'createtourfg',   // bound action name
      label               : 'Tour Anlegen',
      iconUrl: 'sap-icon://wrench',
//      requiresSelection   : false,              // => global action
      invocationGrouping  : #CHANGE_SET,        // optional
      emphasized          : true,               // optional
      position            : 100,
      importance          : #HIGH
      
    }
  ]
 @UI.hidden: true
    CreatedBy;
}


@Metadata.layer: #PARTNER

annotate entity /PLCE/C_PDMNLTourWR
    with 
{
    @UI.lineItem: [{ position: 1 },
                 { hidden: true },
                { type:#FOR_ACTION, dataAction:'GenerateGeoRoute', label:'Spur ermitteln', invocationGrouping: #CHANGE_SET, position: 40  },
                 { type:#FOR_ACTION, dataAction:'releaseTour', label:'Tour Freigeben', invocationGrouping: #CHANGE_SET, position: 30  },
                 { type:#FOR_ACTION, dataAction:'recallTour', label:'Tour zurück rufen', invocationGrouping: #CHANGE_SET, position: 25 },
                 { type:#FOR_ACTION, dataAction:'createtour', label:'Tour anlegen', invocationGrouping: #CHANGE_SET, position: 10 },
                 { type:#FOR_ACTION, dataAction:'tourgenerateDocument', label:'Ducumente drucken', invocationGrouping: #CHANGE_SET, position: 12 },
                 { type:#FOR_ACTION, dataAction:'touranBMSfreigeben', label:'Tour an BMS freigeben', invocationGrouping: #CHANGE_SET, position: 11 },
                 { type:#FOR_ACTION, dataAction:'stornoBMSService', label:'BMS storno', invocationGrouping: #CHANGE_SET, position: 11 },
                 { type:#FOR_ACTION, dataAction:'deleteTour', label:'Tour Löschen', invocationGrouping: #CHANGE_SET, position: 20 },
                 { type:#FOR_ACTION, dataAction:'SequenceAutomated', label: 'Tour automatisch ordnen', position: 50 },
                 { type:#FOR_ACTION, dataAction:'TransferSequence', label:'Servicereihfolge übertragen', invocationGrouping: #CHANGE_SET, position: 70 },
                 { type:#FOR_ACTION, dataAction:'GenerateGeoRoute', label:'Spur ermitteln', invocationGrouping: #CHANGE_SET, qualifier: 'DefaultMap' }
                        
               ]                             
//   
   @UI.hidden: true
    TourUUID; 
 
    // Remaining Duration (e.g. 280)
  @EndUserText.label: 'Restdauer(freie Kapazität)'
  @UI.lineItem: [{ position: 101, importance: #HIGH }]  
  zz_remaining_duration;
 
  // Capacity Used (e.g. 230)
  @EndUserText.label: 'Kapazitätsauslastung(in Min)'
  @UI.lineItem: [
    { position: 102, type: #AS_DATAPOINT, label: 'Kapazitätsauslastung(in Min)', importance: #HIGH },
    { position: 50,  type: #AS_DATAPOINT, label: 'Kapazitätsauslastung(in Min)', importance: #HIGH, qualifier: 'DefaultMap' }
  ]
  @UI.dataPoint: {
    visualization: #PROGRESS,
    targetValueElement: 'MaximumTourDuration_Converted', 
//    criticality: 'TourCapacityColorValue'
    criticality: 'zz_capacity_criticality'
  }
  @UI.fieldGroup: [{position: 110, qualifier: 'DefaultInformation', type: #AS_DATAPOINT }]
  tour_capacity_new;
  
  @UI.hidden: true
  tour_duration_min;
  
  @UI.hidden: true
  @UI.lineItem: [{ position: 450, importance: #HIGH }] 
  TourDuration;

  @UI.hidden: true
  MaximumTourDuration_Converted;
  
  // 1. Hide the standard SAP field (Hours)
//  @UI.hidden: true
  @UI.lineItem: [{ position: 121, importance: #HIGH }]
  DrivingTime;

  // 2. Show your new custom field (Minutes)
  @UI.lineItem: [{ position: 122, importance: #HIGH }]
  @EndUserText.label: 'Fahrzeit'  // Driving Time
  zz_driving_time_min;
  
  
  @UI.lineItem: [{
    position: 995,
    label: 'BMS-Status',
    criticality: 'ZzBmsCriticality',
    criticalityRepresentation: #WITH_ICON
  }]
  @EndUserText.label: 'BMS-Status'
  zz_bms_status;

@UI.hidden: true
  ZzBmsCriticality;
}

@Metadata.layer: #CUSTOMER
@UI: {
lineItem: [{ criticality: 'service_criticality' }]
}
annotate entity /PLCE/C_PDMNLServiceWR with

{
  @UI.lineItem: [
     { position: 190, importance: #HIGH },
     {
       type                : #FOR_ACTION,
       dataAction          : 'assignworkarea',   // bound action name
       label               : 'Arbeitsbereich zuweisen',
       iconUrl: 'sap-icon://wrench',
  //      requiresSelection   : false,              // => global action
       invocationGrouping  : #CHANGE_SET,        // optional
       emphasized          : true,               // optional
       position            : 100,
       importance          : #HIGH

     },
     
     { 
       type                : #FOR_ACTION,
       dataAction          : 'adjusttime',   // bound action name
       label               : 'Zeitanpassung ändern',
       iconUrl: 'sap-icon://wrench',
//        requiresSelection   : false,              // => global action
       invocationGrouping  : #CHANGE_SET,        // optional
       emphasized          : true,               // optional
       position            : 100,
       importance          : #HIGH
       
     }, 
     
     {
       type                : #FOR_ACTION,
       dataAction          : 'terminateservice',   // bound action name
       label               : 'Service stornieren',
       iconUrl: 'sap-icon://wrench',
       invocationGrouping  : #CHANGE_SET,        // optional
       emphasized          : true,               // optional
       position            : 100,
       importance          : #HIGH

     },
     
          {
       type                : #FOR_ACTION,
       dataAction          : 'anlageaendern',   // bound action name
       label               : 'Anlage ändern',
       iconUrl: 'sap-icon://wrench',
       invocationGrouping  : #CHANGE_SET,        // optional
       emphasized          : true,               // optional
       position            : 100,
       importance          : #HIGH

     }
     
   ]
   
   
  
  @EndUserText.label: 'Fachbereich'
  @UI.selectionField: [{position: 20}]
  @Consumption.valueHelpDefinition: [
    {
      entity         : { name: 'ZZPD_WORKREA_VH', element: 'WorkArea' },
      label          : 'Workarea',
      useForValidation: true
    }
  ]
  zz_tech_fachbe;
  @EndUserText.label: 'Termin-Diskrepanz'
  zz_discrepancy; // Termin-Diskrepanz
  @EndUserText.label: 'Reaktionszeit'
  zz_reactiontime; // “Reaktionszeit
  @EndUserText.label: 'Fahrzeuginfo'
  zz_vehicleinfo;  // Fahrzeuginfo
  @EndUserText.label: 'Zeitanpassung in Min'
  @UI.lineItem: [{ position: 131, importance: #HIGH } ]
  zz_timeadjust_display; // Zeitanpassung
  @UI.hidden: true
  service_criticality;
//  @UI.hidden: true
  @EndUserText.label: 'main Criticallity'
  @UI.lineItem: [{ position: 135, importance: #HIGH , hidden: true } ]
  zz_main_pos_criticality;
  @EndUserText.label: 'Entsorgungsanlage'
  @UI.lineItem: [{ position: 132, importance: #HIGH } ]
  wdplantnr;
  @UI.lineItem: [{ position: 137, importance: #HIGH , criticality: 'zz_main_pos_criticality'} ]
  zz_informational_group;
//  zz_pobjnr_main;
   
}


extension for projection implementation in class zbp_e_bp_c_pdmnlservicewr_ext unique;

extend behavior for Service
{
use action assignworkarea;
use action adjusttime;
use action terminateservice ;
use action anlageaendern;


 // Now the compiler will accept this because we are in the UI layer!
  side effects {
    action adjusttime affects entity _TourForRefresh;
    action terminateservice affects $self;
   action anlageaendern affects $self;
  }
}


extension for projection implementation in class zbp_e_bp_c_pdtour unique;

extend behavior for Tour
{

use action createtour;
use action touranBMSfreigeben;
use action stornoBMSService;
use action tourgenerateDocument;
}

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
    action anlageaendern affects $self , entity _ExtCustom;
  }

}

extend behavior for ServiceTask
{
}

extend behavior for ExtCustom
{
  mapping for /plce/tpdsrvcst
    {
      zz_tech_fachbe    = zz_tech_fachbe;
      zz_vehicleinfo    = zz_vehicleinfo;
      zz_discrepancy    = zz_discrepancy;
      zz_reactiontime   = zz_reactiontime;
      zz_order_date     = zz_order_date;
      zz_timeadjustment = zz_timeadjustment;
      zz_pobjnr_main    = zz_pobjnr_main;
      zz_informational_group = zz_informational_group;
      zz_bms_status     = zz_bms_status;
      wdplantnr   = wdplantnr;
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



<img width="1194" height="534" alt="image" src="https://github.com/user-attachments/assets/be563a85-861f-4d05-95e2-71799c90b7d0" />


* regenerated at 24.06.2026 12:34:31
*******************************************************************
*   System-defined Include-files.                                 *
*******************************************************************
  INCLUDE LZTOUR_BMS_KOLOTOP.                " Global Declarations
  INCLUDE LZTOUR_BMS_KOLOUXX.                " Function Modules

*******************************************************************
*   User-defined Include-files (if necessary).                    *
*******************************************************************
* INCLUDE LZTOUR_BMS_KOLOF...                " Subroutines
* INCLUDE LZTOUR_BMS_KOLOO...                " PBO-Modules
* INCLUDE LZTOUR_BMS_KOLOI...                " PAI-Modules
* INCLUDE LZTOUR_BMS_KOLOE...                " Events
* INCLUDE LZTOUR_BMS_KOLOP...                " Local class implement.
* INCLUDE LZTOUR_BMS_KOLOT99.                " ABAP Unit tests
  INCLUDE LZTOUR_BMS_KOLOF00                      . " subprograms
  INCLUDE LZTOUR_BMS_KOLOI00                      . " PAI modules
  INCLUDE LSVIMFXX                                . " subprograms
  INCLUDE LSVIMOXX                                . " PBO modules
  INCLUDE LSVIMIXX                                . " PAI module

  * regenerated at 23.06.2026 13:34:31
*******************************************************************
*   System-defined Include-files.                                 *
*******************************************************************
  INCLUDE LZZ_BMS_CFG_GROUPTOP.              " Global Declarations
  INCLUDE LZZ_BMS_CFG_GROUPUXX.              " Function Modules

*******************************************************************
*   User-defined Include-files (if necessary).                    *
*******************************************************************
* INCLUDE LZZ_BMS_CFG_GROUPF...              " Subroutines
* INCLUDE LZZ_BMS_CFG_GROUPO...              " PBO-Modules
* INCLUDE LZZ_BMS_CFG_GROUPI...              " PAI-Modules
* INCLUDE LZZ_BMS_CFG_GROUPE...              " Events
* INCLUDE LZZ_BMS_CFG_GROUPP...              " Local class implement.
* INCLUDE LZZ_BMS_CFG_GROUPT99.              " ABAP Unit tests
  INCLUDE LZZ_BMS_CFG_GROUPF00                    . " subprograms
  INCLUDE LZZ_BMS_CFG_GROUPI00                    . " PAI modules
  INCLUDE LSVIMFXX                                . " subprograms
  INCLUDE LSVIMOXX                                . " PBO modules
  INCLUDE LSVIMIXX 

  *******************************************************************
*   System-defined Include-files.                                 *
*******************************************************************
  INCLUDE LZZ_BMS_HELPERTOP.                 " Global Declarations
  INCLUDE LZZ_BMS_HELPERUXX.                 " Function Modules

*******************************************************************
*   User-defined Include-files (if necessary).                    *
*******************************************************************
* INCLUDE LZZ_BMS_HELPERF...                 " Subroutines
* INCLUDE LZZ_BMS_HELPERO...                 " PBO-Modules
* INCLUDE LZZ_BMS_HELPERI...                 " PAI-Modules
* INCLUDE LZZ_BMS_HELPERE...                 " Events
* INCLUDE LZZ_BMS_HELPERP...                 " Local class implement.
* INCLUDE LZZ_BMS_HELPERT99.  

* regenerated at 17.03.2026 09:01:29
*******************************************************************
*   System-defined Include-files.                                 *
*******************************************************************
  INCLUDE LZZ_STORNO_GROUPTOP.               " Global Declarations
  INCLUDE LZZ_STORNO_GROUPUXX.               " Function Modules

*******************************************************************
*   User-defined Include-files (if necessary).                    *
*******************************************************************
* INCLUDE LZZ_STORNO_GROUPF...               " Subroutines
* INCLUDE LZZ_STORNO_GROUPO...               " PBO-Modules
* INCLUDE LZZ_STORNO_GROUPI...               " PAI-Modules
* INCLUDE LZZ_STORNO_GROUPE...               " Events
* INCLUDE LZZ_STORNO_GROUPP...               " Local class implement.
* INCLUDE LZZ_STORNO_GROUPT99.               " ABAP Unit tests
  INCLUDE LZZ_STORNO_GROUPF00                     . " subprograms
  INCLUDE LZZ_STORNO_GROUPI00                     . " PAI modules
  INCLUDE LSVIMFXX                                . " subprograms
  INCLUDE LSVIMOXX                                . " PBO modules
  INCLUDE LSVIMIXX                

  CLASS lhc_service DEFINITION INHERITING FROM cl_abap_behavior_handler.

  PRIVATE SECTION.

    METHODS get_global_features FOR GLOBAL FEATURES
      IMPORTING REQUEST requested_features FOR service RESULT result.

    METHODS get_global_authorizations FOR GLOBAL AUTHORIZATION
      IMPORTING REQUEST requested_authorizations FOR service RESULT result.

    METHODS assignworkarea FOR MODIFY
      IMPORTING keys FOR ACTION service~assignworkarea RESULT result.

    METHODS precheck_assignworkarea FOR PRECHECK
      IMPORTING keys FOR ACTION service~assignworkarea.


    METHODS populateextcustomfromorder FOR DETERMINE ON MODIFY
      IMPORTING keys FOR service~populateextcustomfromorder.



    METHODS adjusttime FOR MODIFY
      IMPORTING keys FOR ACTION service~adjusttime RESULT result.
    METHODS precheck_adjusttime FOR PRECHECK
      IMPORTING keys FOR ACTION service~adjusttime.
    METHODS terminateservice FOR MODIFY
      IMPORTING keys FOR ACTION service~terminateservice RESULT result.

    METHODS precheck_terminateservice FOR PRECHECK
      IMPORTING keys FOR ACTION service~terminateservice.
    METHODS anlageaendern FOR MODIFY
      IMPORTING keys FOR ACTION service~anlageaendern RESULT result.

    METHODS precheck_anlageaendern FOR PRECHECK
      IMPORTING keys FOR ACTION service~anlageaendern.

ENDCLASS.

CLASS lhc_service IMPLEMENTATION.

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
      FIELDS ( serviceuuid wdplantnr zz_tech_fachbe zz_pobjnr_main zz_informational_group )   " <-- NEW 1
      WITH VALUE #( ( serviceuuid = <ls_service>-serviceuuid ) )
      RESULT DATA(lt_extcustom).

    IF lt_extcustom IS NOT INITIAL.
      DATA(ls_extcustom) = lt_extcustom[ 1 ].

      IF ls_extcustom-wdplantnr IS NOT INITIAL
         AND ls_extcustom-zz_tech_fachbe IS NOT INITIAL
         AND ls_extcustom-zz_pobjnr_main IS NOT INITIAL
         AND ls_extcustom-zz_informational_group IS NOT INITIAL.   " <-- NEW 2
        CLEAR lt_extcustom.
        CONTINUE.
      ENDIF.
    ENDIF.

    " FIXED: Use CDS view to avoid multiple record issue
    SELECT SINGLE zz_tech_fachbe, wdplantnr, zz_pobjnr_main, zz_informational_group   " <-- NEW 3
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
          zz_informational_group = ls_order_fields-zz_informational_group   " <-- NEW 4
        ) )
      ) TO lt_ext_create.

    ELSE.
      " ExtCustom EXISTS - update only EMPTY fields
      DATA(lv_needs_update) = abap_false.

      DATA: lv_new_fachbe      TYPE ewa_order_object-zz_tech_fachbe,
            lv_new_wdplantnr   TYPE ewael_wdplantnr,
            lv_new_pobjnr_main TYPE zwr_d_pobjnr_main,
            lv_new_info_group  TYPE ewa_order_object-zz_informational_group.   " <-- NEW 5a

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

      " Check zz_pobjnr_main, not wdplantnr!
      IF ls_extcustom-zz_pobjnr_main IS INITIAL
         AND ls_order_fields-zz_pobjnr_main IS NOT INITIAL.
        lv_new_pobjnr_main = zcl_wr_eewa_wdoc_misc=>cl_convert_objnr_out(
                               par_pobjnr = ls_order_fields-zz_pobjnr_main ).
        lv_needs_update = abap_true.
      ELSE.
        lv_new_pobjnr_main = ls_extcustom-zz_pobjnr_main.
      ENDIF.

      " <-5b: informational group, copied as is (already the external number)
      IF ls_extcustom-zz_informational_group IS INITIAL
         AND ls_order_fields-zz_informational_group IS NOT INITIAL.
        lv_new_info_group = ls_order_fields-zz_informational_group.
        lv_needs_update = abap_true.
      ELSE.
        lv_new_info_group = ls_extcustom-zz_informational_group.
      ENDIF.

      IF lv_needs_update = abap_true.
        APPEND VALUE #(
          %key-serviceuuid = <ls_service>-serviceuuid
          zz_tech_fachbe = lv_new_fachbe
          wdplantnr = lv_new_wdplantnr
          zz_pobjnr_main = lv_new_pobjnr_main
          zz_informational_group = lv_new_info_group
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
      UPDATE FIELDS ( zz_tech_fachbe wdplantnr zz_pobjnr_main zz_informational_group )   " <-- NEW 6
      WITH lt_ext_update
      FAILED DATA(lt_failed_update)
      REPORTED DATA(lt_reported_update).
  ENDIF.

ENDMETHOD.

  METHOD get_global_features.
  ENDMETHOD.

  METHOD get_global_authorizations.
  ENDMETHOD.

  METHOD assignworkarea.

    READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
    ENTITY service
    FIELDS ( referenceid serviceuuid ) " // Use CDS casing; add ServiceUUID for key
    WITH CORRESPONDING #( keys )
    RESULT DATA(lt_services)
    FAILED DATA(lt_failed).

    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ls_service>).
*     " // Check if ExtCustom exists; read it first
      READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
        ENTITY extcustom
        FIELDS ( serviceuuid )  "// Minimal to check existence
        WITH VALUE #( ( serviceuuid = <ls_service>-serviceuuid ) )
        RESULT DATA(lt_extcustom).

      IF lt_extcustom IS INITIAL.
*        // Create if missing
        DATA: lv_cid        TYPE string VALUE '$abap_cid1_',
              lt_ext_create TYPE TABLE FOR CREATE /plce/r_pdservice\_extcustom.
        APPEND INITIAL LINE TO lt_ext_create ASSIGNING FIELD-SYMBOL(<ls_ext_create>).
        <ls_ext_create>-serviceuuid = <ls_service>-serviceuuid. " // %tky or key
        <ls_ext_create>-%target = VALUE #( ( %cid = lv_cid ) ).

        MODIFY ENTITIES OF /plce/r_pdservice IN LOCAL MODE
          ENTITY service
          CREATE BY \_extcustom
          AUTO FILL CID SET FIELDS WITH lt_ext_create
          MAPPED DATA(lmapped)
          FAILED DATA(lfailed)
          REPORTED DATA(lreported).
*        // Handle errors if needed
      ENDIF.

*      // Now update the field
      MODIFY ENTITIES OF /plce/r_pdservice IN LOCAL MODE
        ENTITY extcustom
        UPDATE FIELDS ( zz_tech_fachbe )
        WITH VALUE #( ( %key-serviceuuid = <ls_service>-serviceuuid
                        zz_tech_fachbe = <ls_service>-%param-workarea ) )
        FAILED DATA(lt_failed_update)
        REPORTED DATA(lt_reported_update).


      " Step 4: Set action result ($self = Service instances). This ensures that the values are shown on the UI
      READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
        ENTITY service
          ALL FIELDS WITH CORRESPONDING #( keys )
        RESULT DATA(lt_service_result).

      result = VALUE #( FOR ls_service IN lt_service_result ( %tky   = ls_service-%tky
                                                             %param  = ls_service ) ).
      CLEAR:   lt_ext_create,  lt_reported_update,  lt_failed_update,  lreported, lfailed, lmapped.
    ENDLOOP.
  ENDMETHOD.

  METHOD precheck_assignworkarea.

    LOOP AT keys INTO DATA(ls_service).
*      " Check if WorkArea is valid (query value help entity if needed).
*      IF ls_service-%param-WorkArea IS INITIAL.
*        APPEND VALUE #( %key = ls_service-%key
*                        %msg = new_message( id = 'Z_MSG_CLASS' number = '000' severity = if_abap_behv_message=>severity-error ) )
*               TO reported-Service.
*      ENDIF.

    ENDLOOP.
  ENDMETHOD.

  METHOD adjusttime.
    " 1. Loop through the keys (Service Instances)
    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ls_key>).

      " 2. Check if Extension Record exists
      READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
        ENTITY extcustom
        FIELDS ( serviceuuid )
        WITH VALUE #( ( serviceuuid = <ls_key>-serviceuuid ) )
        RESULT DATA(lt_extcustom).

      IF lt_extcustom IS INITIAL.
        " 3. Create Extension if missing
        DATA: lv_cid        TYPE string VALUE '$abap_cid_adj_',
              lt_ext_create TYPE TABLE FOR CREATE /plce/r_pdservice\_extcustom.

        APPEND INITIAL LINE TO lt_ext_create ASSIGNING FIELD-SYMBOL(<ls_ext_create>).
        <ls_ext_create>-serviceuuid = <ls_key>-serviceuuid.
        <ls_ext_create>-%target = VALUE #( ( %cid = lv_cid ) ).

        MODIFY ENTITIES OF /plce/r_pdservice IN LOCAL MODE
          ENTITY service
          CREATE BY \_extcustom
          AUTO FILL CID SET FIELDS WITH lt_ext_create
          MAPPED DATA(lmapped_create)
          FAILED DATA(lfailed_create)
          REPORTED DATA(lreported_create).

        " Optional: Handle creation errors here if needed
      ENDIF.

      " ====================================================================
      " 4. Prepare the Numeric Value
      " ====================================================================
      DATA(lv_raw_input) = <ls_key>-%param-zeitinmin.

      " Assuming your DB field is now an INT4. If it's a DEC, change TYPE i to TYPE p.
      DATA lv_numeric_result TYPE i.

      " Normalize input (replace comma with dot if user typed "37,5")
      DATA lv_calc_val TYPE string.
      lv_calc_val = lv_raw_input.
      REPLACE ALL OCCURRENCES OF ',' IN lv_calc_val WITH '.'.
      CONDENSE lv_calc_val NO-GAPS.

      " Safely convert the input to a pure number
      TRY.
          lv_numeric_result = CONV i( lv_calc_val ).
        CATCH cx_sy_conversion_no_number.
          " Fallback if the user typed garbage text instead of a number
          lv_numeric_result = 0.
      ENDTRY.

      " ====================================================================
      " 5. Update the field with
      " ====================================================================
      MODIFY ENTITIES OF /plce/r_pdservice IN LOCAL MODE
        ENTITY extcustom
        UPDATE FIELDS ( zz_timeadjustment )
        WITH VALUE #( ( %key-serviceuuid = <ls_key>-serviceuuid
                        zz_timeadjustment = lv_numeric_result ) ) " <--- Pure Number
        FAILED DATA(lt_failed_update)
        REPORTED DATA(lt_reported_update).

      " 6. Aggregate errors
      APPEND LINES OF lt_failed_update-extcustom TO failed-extcustom.
      APPEND LINES OF lt_reported_update-extcustom TO reported-extcustom.


      " 7. Read Result to refresh UI
      READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
        ENTITY service
        ALL FIELDS WITH CORRESPONDING #( keys )
        RESULT DATA(lt_service_result).

      LOOP AT lt_service_result ASSIGNING FIELD-SYMBOL(<ls_result>).
        INSERT VALUE #( %tky   = <ls_result>-%tky
                        %param = <ls_result> ) INTO TABLE result.
      ENDLOOP.

    ENDLOOP.

  ENDMETHOD.


  METHOD precheck_adjusttime.

    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ls_key>).

      " ---------------------------------------------------------------------
      " 1. Parse Input & Format Validation
      " ---------------------------------------------------------------------
      DATA(lv_input_str) = CONV string( <ls_key>-%param-zeitinmin ).
      REPLACE ALL OCCURRENCES OF ',' IN lv_input_str WITH '.'.
      CONDENSE lv_input_str NO-GAPS.

      " Regex: Allow integers or decimals, optional minus sign
      IF lv_input_str IS NOT INITIAL.
        IF NOT matches( val = lv_input_str regex = '^-?[0-9]+([.,][0-9]+)?$' ).
          APPEND VALUE #( %tky = <ls_key>-%tky
                          %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
          APPEND VALUE #( %tky = <ls_key>-%tky

                           %msg = new_message(
                       id       = 'Z_MSG_SVR_TOUR_EXT'
                       number   = '005'
                       severity = if_abap_behv_message=>severity-error

                     )

                         ) TO reported-service.
          CONTINUE.
        ENDIF.
      ENDIF.

      " Convert to Number for calculations
      DATA lv_adj_min  TYPE decfloat34.
      TRY.
          lv_adj_min = lv_input_str.
        CATCH cx_sy_conversion_no_number.
          CONTINUE.
      ENDTRY.

      " ---------------------------------------------------------------------
      " 2. Limit Check: Max +/- 600 Minutes (10 Hours)
      " ---------------------------------------------------------------------
      IF lv_adj_min > 600 OR lv_adj_min < -600.
        APPEND VALUE #( %tky = <ls_key>-%tky
                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
        APPEND VALUE #( %tky = <ls_key>-%tky
                           %msg = new_message(
                     id       = 'Z_MSG_SVR_TOUR_EXT'
                     number   = '006'
                     severity = if_abap_behv_message=>severity-error

                   )
                      ) TO reported-service.
        CONTINUE.
      ENDIF.
    ENDLOOP.

  ENDMETHOD.

  METHOD terminateservice.
    DATA: lt_unassign_keys  TYPE TABLE FOR ACTION IMPORT /plce/r_pdtour\\serviceassignment~unassign,
          lt_service_update TYPE TABLE FOR UPDATE /plce/r_pdservice,
          lt_return         TYPE bapiret2_t.

    READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY service
      FIELDS ( serviceuuid referenceinternalid referenceid servicestatus serviceid )
      WITH CORRESPONDING #( keys )
      RESULT DATA(lt_services).

    LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_service>).
      DATA(ls_param) = keys[ %tky = <ls_service>-%tky ]-%param.
      CLEAR lt_return.

      " 1. Do the Waste Order Storno/Delete in the Backend
      CALL FUNCTION 'Z_WR_CANCEL_WASTE_ORDER_RFC' DESTINATION 'NONE'
        EXPORTING
          iv_pobjnr            = CONV char30( <ls_service>-referenceinternalid )
          iv_reason_predefined = 'PLSTORNO'
          iv_reason_text       = CONV string( ls_param-stornogrund )
          iv_serviceuuid       = <ls_service>-serviceuuid
        IMPORTING
          et_return            = lt_return.

      IF lt_return IS INITIAL.
        " --- SUCCESS ---

        " A. Update Status to '01' to keep Fiori happy for the next 3 seconds
        APPEND VALUE #( %tky = <ls_service>-%tky
                        servicestatus = '01' ) TO lt_service_update.

        " C. Prepare Tour Unassignment
        SELECT SINGLE asgmt~serviceuuid, asgmt~touruuid
          FROM /plce/r_pdtourserviceasgmt AS asgmt
          WHERE asgmt~serviceuuid = @<ls_service>-serviceuuid
          INTO @DATA(ls_unassign).

        IF sy-subrc = 0.
          APPEND VALUE #( %key-touruuid    = ls_unassign-touruuid
                          %key-serviceuuid = ls_unassign-serviceuuid ) TO lt_unassign_keys.
        ENDIF.

        " D. Success Message
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %msg = new_message( id       = 'Z_MSG_SVR_TOUR_EXT'
                                            number   = '009'
                                            severity = if_abap_behv_message=>severity-information
                                            v1       = |{ <ls_service>-referenceid ALPHA = OUT }|  )
                      ) TO reported-service.

      ELSE.
        " --- FAILURE ---
        APPEND VALUE #( %tky = <ls_service>-%tky ) TO failed-service.

        LOOP AT lt_return INTO DATA(ls_ret).
          APPEND VALUE #( %tky = <ls_service>-%tky
                          %msg = new_message_with_text(
                                   severity = if_abap_behv_message=>severity-error
                                   text     = CONV #( ls_ret-message ) )
                        ) TO reported-service.
        ENDLOOP.
      ENDIF.
    ENDLOOP.

    " 2. Bulk Modify RAP Entities (Just the temporary status update)
    IF lt_service_update IS NOT INITIAL.
      MODIFY ENTITIES OF /plce/r_pdservice IN LOCAL MODE
        ENTITY service
          UPDATE FIELDS ( servicestatus ) WITH lt_service_update.
    ENDIF.

    " 3. Bulk Execute Tour Unassignment
    IF lt_unassign_keys IS NOT INITIAL.
      MODIFY ENTITIES OF /plce/r_pdtour
        ENTITY serviceassignment
          EXECUTE unassign FROM lt_unassign_keys
        FAILED DATA(srvc_unassign_failed)
        REPORTED DATA(srvc_unassign_reported).
    ENDIF.

    " 4. Return the updated data to the UI
    READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY service ALL FIELDS WITH CORRESPONDING #( keys ) RESULT DATA(lt_service_result).

    result = VALUE #( FOR srv IN lt_service_result ( %tky = srv-%tky %param = srv ) ).

  ENDMETHOD.


  METHOD precheck_terminateservice.
  ENDMETHOD.


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

    TYPES:
      BEGIN OF ty_pobj,
        pobjnr TYPE j_objnr,
      END OF ty_pobj,
      tt_pobj TYPE SORTED TABLE OF ty_pobj WITH UNIQUE KEY pobjnr.

    DATA:
      lt_pobj       TYPE tt_pobj,
      lt_orders     TYPE zcl_wr_pd_tour_helper=>tt_sig_orders,
      lt_eanv_notes TYPE zcl_wr_pd_tour_helper=>tt_eanv_notes,
      ls_eanv_note  TYPE zcl_wr_pd_tour_helper=>ty_eanv_note,
      lv_pobjnr     TYPE j_objnr.

    " Read ALL services at once
    READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
      ENTITY service
      FIELDS ( serviceuuid referenceinternalid referenceid )
      WITH CORRESPONDING #( keys )
      RESULT DATA(lt_services).

    " ---------------------------------------------------------------------
    " Scheine on an eANV Nachweis — one mass read for all selected EAPs.
    " POBJNR is CHAR 22, REFERENCE_INTERNAL_ID is CHAR 30 and FOR ALL
    " ENTRIES needs identical type and length, hence the key table.
    " NOTEINTNR comes from the DB, so it is already in the internal
    " (ALPHA) format the helper expects.
    " ---------------------------------------------------------------------
    LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_key_svc>).
      IF <ls_key_svc>-referenceinternalid IS NOT INITIAL.
        INSERT VALUE #( pobjnr = <ls_key_svc>-referenceinternalid )
          INTO TABLE lt_pobj.
      ENDIF.
    ENDLOOP.

    IF lt_pobj IS NOT INITIAL.
      SELECT pobjnr,
             /watp/noteintnr AS noteintnr
        FROM ewa_order_object
        FOR ALL ENTRIES IN @lt_pobj
        WHERE pobjnr = @lt_pobj-pobjnr
        INTO TABLE @lt_orders.

      IF lt_orders IS NOT INITIAL.
        lt_eanv_notes = zcl_wr_pd_tour_helper=>get_eanv_notes( lt_orders ).
      ENDIF.
    ENDIF.

    " Loop through SERVICES (not keys!)
    LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_service>).

      CLEAR: ls_eanv_note, lv_pobjnr.

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
      " Validation 2: EAP has a Schein on an eANV Nachweis → no ESA change
      "   Ticket check 1. Only Scheine whose Nachweis takes part in eANV
      "   count (TARBNW-DIGI_SIG, same rule as #42128). An EAP without
      "   Schein, or with a Schein on a paper Nachweis, goes on to the
      "   ESA checks and the AVV check (ticket check 2).
      " ---------------------------------------------------------------------
      lv_pobjnr = <ls_service>-referenceinternalid.

      READ TABLE lt_eanv_notes INTO ls_eanv_note
        WITH TABLE KEY pobjnr = lv_pobjnr.

      IF sy-subrc = 0.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
        APPEND VALUE #( %tky = <ls_service>-%tky
                        %msg = new_message(
                                 id       = 'Z_MSG_SVR_TOUR_EXT'
                                 number   = '057'
                                 severity = if_abap_behv_message=>severity-error
                                 v1       = condense( |{ <ls_service>-referenceid ALPHA = OUT }| )
                                 v2       = ls_eanv_note-notenr
                                 v3       = CONV #( ls_param-anlage )
                                 v4       = ls_eanv_note-nwnr )
                      ) TO reported-service.
        CONTINUE.
      ENDIF.

      " ---------------------------------------------------------------------
      " Validation 3: Check if facility is selected
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
      " Validation 4: Check facility exists and is available
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
      " Validation 5: AVV-Code matching
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



*  METHOD precheck_anlageaendern.
*
*    " Read ALL services at once
*    READ ENTITIES OF /plce/r_pdservice IN LOCAL MODE
*      ENTITY service
*      FIELDS ( serviceuuid referenceinternalid referenceid )
*      WITH CORRESPONDING #( keys )
*      RESULT DATA(lt_services).
*
*    " Loop through SERVICES (not keys!)
*    LOOP AT lt_services ASSIGNING FIELD-SYMBOL(<ls_service>).
*
*      " Get the parameter for THIS service
*      DATA(ls_param) = keys[ %tky = <ls_service>-%tky ]-%param.
*
*      " Get order details
*      SELECT SINGLE *
*        FROM ewa_order_object
*        WHERE pobjnr = @<ls_service>-referenceinternalid
*        INTO @DATA(ls_order).
*
*      IF sy-subrc <> 0.
*        CONTINUE.
*      ENDIF.
*
*      " ---------------------------------------------------------------------
*      " Validation 1: Check for Fixed Disposal Contract
*      " ---------------------------------------------------------------------
*      IF ls_order-wdplantnr IS NOT INITIAL.
*        SELECT SINGLE zz_framework_agreement
*          FROM ewa_el_wdplant
*          WHERE wdplantnr = @ls_order-wdplantnr
*          INTO @DATA(lv_existing_contract).
*
*        IF sy-subrc = 0 AND lv_existing_contract IS NOT INITIAL.
*          APPEND VALUE #( %tky = <ls_service>-%tky
*                          %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
*          APPEND VALUE #( %tky = <ls_service>-%tky
*                          %msg = new_message(
*                                   id       = 'Z_MSG_SVR_TOUR_EXT'
*                                   number   = '010'
*                                   severity = if_abap_behv_message=>severity-error
*                                   )
*                        ) TO reported-service.
*          CONTINUE.
*        ENDIF.
*      ENDIF.
*
*      " ---------------------------------------------------------------------
*      " Validation 2: Check if facility is selected
*      " ---------------------------------------------------------------------
*      IF ls_param-anlage IS INITIAL.
*        APPEND VALUE #( %tky = <ls_service>-%tky
*                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
*        APPEND VALUE #( %tky = <ls_service>-%tky
*                        %msg = new_message(
*                                 id       = 'Z_MSG_SVR_TOUR_EXT'
*                                 number   = '011'
*                                 severity = if_abap_behv_message=>severity-error
*                                  )
*                      ) TO reported-service.
**        CONTINUE.
*      ENDIF.
*
*      " ---------------------------------------------------------------------
*      " Validation 3: Check facility exists and is available
*      " ---------------------------------------------------------------------
*      SELECT SINGLE wdplantnr, zz_available_dispo, zz_framework_agreement
*        FROM ewa_el_wdplant
*        WHERE wdplantnr = @ls_param-anlage
*        INTO @DATA(ls_plant).
*
*      IF sy-subrc <> 0.
*        APPEND VALUE #( %tky = <ls_service>-%tky
*                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
*        APPEND VALUE #( %tky = <ls_service>-%tky
*                        %msg = new_message(
*                                 id       = 'Z_MSG_SVR_TOUR_EXT'
*                                 number   = '012'
*                                 severity = if_abap_behv_message=>severity-error
*                                 v1       = CONV #( ls_param-anlage ) )
*                      ) TO reported-service.
*        CONTINUE.
*      ENDIF.
*
*      IF ls_plant-zz_available_dispo IS INITIAL OR ls_plant-zz_available_dispo <> 'X'.
*        APPEND VALUE #( %tky = <ls_service>-%tky
*                        %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
*        APPEND VALUE #( %tky = <ls_service>-%tky
*                        %msg = new_message(
*                                 id       = 'Z_MSG_SVR_TOUR_EXT'
*                                 number   = '013'
*                                 severity = if_abap_behv_message=>severity-error
*                                 v1       = CONV #( ls_param-anlage ) )
*                      ) TO reported-service.
*        CONTINUE.
*      ENDIF.
*
*      " ---------------------------------------------------------------------
*      " Validation 4: AVV-Code matching
*      " ---------------------------------------------------------------------
*      SELECT COUNT(*)
*        FROM /watp/twdplanavv
*        WHERE wdplantnr = @ls_param-anlage
*        INTO @DATA(lv_avv_count).
*
*      IF lv_avv_count > 0.
*        " AVV assignments exist - must validate
*        SELECT SINGLE /watp/avvcode
*          FROM ewa_order_object
*          WHERE pobjnr = @<ls_service>-referenceinternalid
*          INTO @DATA(lv_order_avvcode).
*
*        IF sy-subrc = 0 AND lv_order_avvcode IS NOT INITIAL.
*          SELECT SINGLE avvcode
*            FROM /watp/twdplanavv
*            WHERE wdplantnr = @ls_param-anlage
*              AND avvcode = @lv_order_avvcode
*            INTO @DATA(lv_found_avv).
*
*          IF sy-subrc <> 0.
*            APPEND VALUE #( %tky = <ls_service>-%tky
*                            %fail-cause = if_abap_behv=>cause-unspecific ) TO failed-service.
*            APPEND VALUE #( %tky = <ls_service>-%tky
*                            %msg = new_message(
*                                     id       = 'Z_MSG_SVR_TOUR_EXT'
*                                     number   = '014'
*                                     severity = if_abap_behv_message=>severity-error
*                                     v1       = CONV #( lv_order_avvcode ) )
*                          ) TO reported-service.
*            CONTINUE.
*          ENDIF.
*        ENDIF.
*      ENDIF.
*
*    ENDLOOP.
*  ENDMETHOD.


ENDCLASS.

CLASS lhc_tour DEFINITION INHERITING FROM cl_abap_behavior_handler.
  PRIVATE SECTION.

    METHODS get_global_authorizations FOR GLOBAL AUTHORIZATION
      IMPORTING REQUEST requested_authorizations FOR tour RESULT result.

    METHODS createtour FOR MODIFY
      IMPORTING keys FOR ACTION tour~createtour RESULT result.

    METHODS precheck_createtour FOR PRECHECK
      IMPORTING keys FOR ACTION tour~createtour.

    METHODS assign_earliest_to_latest_date FOR DETERMINE ON MODIFY
      IMPORTING keys FOR tour~assign_earliest_to_latest_date.

    METHODS touranbmsfreigeben FOR MODIFY
      IMPORTING keys FOR ACTION tour~touranbmsfreigeben RESULT result.
    METHODS stornobmsservice FOR MODIFY
      IMPORTING keys FOR ACTION tour~stornobmsservice RESULT result.
    METHODS tourgeneratedocument FOR MODIFY
      IMPORTING keys FOR ACTION tour~tourgeneratedocument RESULT result.

    METHODS remove_obsolete_tour
      IMPORTING iv_touruuid       TYPE /plce/pdtour_uuid
                it_new            TYPE zcl_wr_misc_pd=>tty_attach
      RETURNING VALUE(rv_removed) TYPE i.

    METHODS remove_obsolete_service
      IMPORTING iv_serviceuuid    TYPE /plce/pdservice_uuid
                it_new            TYPE zcl_wr_misc_pd=>tty_attach
      RETURNING VALUE(rv_removed) TYPE i.

ENDCLASS.

CLASS lhc_tour IMPLEMENTATION.

  METHOD assign_earliest_to_latest_date.

    READ ENTITIES OF /plce/r_pdtour IN LOCAL MODE
      ENTITY tour
        FIELDS ( startdate enddate )
        WITH CORRESPONDING #( keys )
      RESULT DATA(lt_tour).

    MODIFY ENTITIES OF /plce/r_pdtour IN LOCAL MODE
      ENTITY tour
        UPDATE FIELDS ( enddate )
        WITH VALUE #(
          FOR ls IN lt_tour
          WHERE ( startdate IS NOT INITIAL AND enddate IS INITIAL )
          ( %tky    = ls-%tky
            enddate = ls-startdate )
        )
      FAILED   DATA(lt_fai)
      REPORTED DATA(lt_rep).

  ENDMETHOD.

  METHOD get_global_authorizations.
  ENDMETHOD.

  METHOD createtour.

    DATA lv_first_date   TYPE /plce/date.
    DATA lv_last_date    TYPE /plce/date.
    DATA lv_current_date TYPE /plce/date.
    DATA lv_exists       TYPE abap_bool.

    TYPES: BEGIN OF ty_msg_sort,
             date TYPE /plce/date,
             msg  TYPE REF TO if_abap_behv_message,
           END OF ty_msg_sort.
    DATA lt_msg_sort TYPE STANDARD TABLE OF ty_msg_sort.

    LOOP AT keys ASSIGNING FIELD-SYMBOL(<key>)
         GROUP BY ( template   = <key>-%param-tour_template
                    first_date = <key>-%param-start_date
                    last_date  = <key>-%param-end_date )
         ASSIGNING FIELD-SYMBOL(<group>).

      lv_first_date = <group>-first_date.
      lv_last_date  = COND /plce/date(
                        WHEN <group>-last_date IS INITIAL
                        THEN lv_first_date
                        ELSE <group>-last_date ).

      IF lv_last_date < lv_first_date.
        DATA(lv_tmp)  = lv_first_date.
        lv_first_date = lv_last_date.
        lv_last_date  = lv_tmp.
      ENDIF.

      lv_current_date = lv_first_date.

      WHILE lv_current_date <= lv_last_date.

        CLEAR lv_exists.

        SELECT SINGLE @abap_true
          FROM /plce/r_pdtour
          WHERE tourtemplate = @<group>-template
            AND startdate    = @lv_current_date
          INTO @lv_exists.

        IF lv_exists = abap_true.

          APPEND VALUE #( date = lv_current_date
                          msg  = new_message(
                                   id       = 'Z_MSG_SVR_TOUR_EXT'
                                   number   = '008'
                                   severity = if_abap_behv_message=>severity-information
                                   v1       = |{ lv_current_date DATE = USER }|
                                   v2       = <group>-template )
                        ) TO lt_msg_sort.

        ELSE.

          DATA(lv_internal_cid) = cl_system_uuid=>create_uuid_x16_static( ).

          MODIFY ENTITIES OF /plce/r_pdtour IN LOCAL MODE
            ENTITY tour
              EXECUTE createtourwithtemplate
              FROM VALUE #(
                ( %cid                 = lv_internal_cid
                  %param-without_draft = 'X'
                  %param-tour_template = <group>-template
                  %param-start_date    = lv_current_date ) )
            MAPPED   DATA(mapped_tour)
            FAILED   DATA(failed_tour)
            REPORTED DATA(reported_tour).

          READ ENTITIES OF /plce/r_pdtour IN LOCAL MODE
            ENTITY tour
              ALL FIELDS
              WITH CORRESPONDING #( mapped_tour-tour )
            RESULT DATA(tours).

          IF lines( tours ) > 0.
            APPEND VALUE #( date = lv_current_date
                            msg  = new_message(
                                     id       = 'Z_MSG_SVR_TOUR_EXT'
                                     number   = '007'
                                     severity = if_abap_behv_message=>severity-success
                                     v1       = |{ lv_current_date DATE = USER }|
                                     v2       = |{ tours[ 1 ]-tourid ALPHA = OUT }| )
                          ) TO lt_msg_sort.

            result = VALUE #( BASE result
                              FOR tour IN tours
                              ( %cid   = lv_internal_cid
                                %param = tour ) ).
          ENDIF.
        ENDIF.

        lv_current_date = lv_current_date + 1.

      ENDWHILE.
    ENDLOOP.

    SORT lt_msg_sort BY date DESCENDING.

    LOOP AT lt_msg_sort INTO DATA(ls_msg_sort).
      INSERT ls_msg_sort-msg INTO reported-%other INDEX 1.
    ENDLOOP.

  ENDMETHOD.


  METHOD remove_obsolete_tour.

    CLEAR rv_removed.

    " which forms are we writing again?
    DATA lt_forms TYPE SORTED TABLE OF /plce/text40 WITH UNIQUE KEY table_line.
    LOOP AT it_new ASSIGNING FIELD-SYMBOL(<ls_new>) WHERE comments CP 'ZWRPRT/*'.
      INSERT <ls_new>-comments INTO TABLE lt_forms.
    ENDLOOP.
    CHECK lt_forms IS NOT INITIAL.

    READ ENTITIES OF /plce/r_pdtour IN LOCAL MODE
      ENTITY tour BY \_attachments
        FIELDS ( attachmentuuid filename comments )
        WITH VALUE #( ( touruuid = iv_touruuid ) )
      RESULT DATA(lt_existing).

    DATA lt_del TYPE TABLE FOR DELETE /plce/r_pdtour\\attachments.

    LOOP AT lt_existing ASSIGNING FIELD-SYMBOL(<ls_att>).
      CHECK <ls_att>-comments CP 'ZWRPRT/*'.                           " 1. ours?
      CHECK line_exists( lt_forms[ table_line = <ls_att>-comments ] ). " 2. rewriting?
      APPEND VALUE #( touruuid       = iv_touruuid
                      attachmentuuid = <ls_att>-attachmentuuid ) TO lt_del.
    ENDLOOP.

    CHECK lt_del IS NOT INITIAL.

    MODIFY ENTITIES OF /plce/r_pdtour IN LOCAL MODE
      ENTITY attachments
        DELETE FROM CORRESPONDING #( lt_del )
      FAILED DATA(lt_fail).

    rv_removed = lines( lt_del ) - lines( lt_fail-attachments ).

  ENDMETHOD.


  METHOD remove_obsolete_service.

    CLEAR rv_removed.

    DATA lt_forms TYPE SORTED TABLE OF /plce/text40 WITH UNIQUE KEY table_line.
    LOOP AT it_new ASSIGNING FIELD-SYMBOL(<ls_new>)
         WHERE serviceuuid = iv_serviceuuid AND comments CP 'ZWRPRT/*'.
      INSERT <ls_new>-comments INTO TABLE lt_forms.
    ENDLOOP.
    CHECK lt_forms IS NOT INITIAL.

    READ ENTITIES OF /plce/r_pdservice
      ENTITY service BY \_attachments
        FIELDS ( attachmentuuid filename comments )
        WITH VALUE #( ( serviceuuid = iv_serviceuuid ) )
      RESULT DATA(lt_existing).

    DATA lt_del TYPE TABLE FOR DELETE /plce/r_pdservice\\attachments.

    LOOP AT lt_existing ASSIGNING FIELD-SYMBOL(<ls_att>).
      CHECK <ls_att>-comments CP 'ZWRPRT/*'.
      CHECK line_exists( lt_forms[ table_line = <ls_att>-comments ] ).
      APPEND VALUE #( serviceuuid    = iv_serviceuuid
                      attachmentuuid = <ls_att>-attachmentuuid ) TO lt_del.
    ENDLOOP.

    CHECK lt_del IS NOT INITIAL.

    MODIFY ENTITIES OF /plce/r_pdservice
      ENTITY attachments
        DELETE FROM CORRESPONDING #( lt_del )
      FAILED DATA(lt_fail).

    rv_removed = lines( lt_del ) - lines( lt_fail-attachments ).

  ENDMETHOD.





  METHOD precheck_createtour.

    DATA(lv_today) = cl_abap_context_info=>get_system_date( ).

    LOOP AT keys ASSIGNING FIELD-SYMBOL(<key>).

      DATA lv_failed   TYPE abap_bool VALUE abap_false.
      DATA lv_start    TYPE /plce/date.
      DATA lv_end      TYPE /plce/date.
      DATA lv_template TYPE /plce/pdtour_template.

      lv_start    = <key>-%param-start_date.
      lv_end      = <key>-%param-end_date.
      lv_template = <key>-%param-tour_template.

      DATA(lv_cid) = cl_system_uuid=>create_uuid_x16_static( ).

      IF lv_start IS INITIAL.
        APPEND VALUE #(
          %cid = <key>-%cid
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '003'
                   severity = if_abap_behv_message=>severity-error
                   v1       = |{ lv_start DATE = USER }| )
        ) TO reported-tour.
        lv_failed = abap_true.
      ENDIF.

      IF lv_end IS NOT INITIAL AND lv_end < lv_start.
        APPEND VALUE #(
          %cid = <key>-%cid
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '002'
                   severity = if_abap_behv_message=>severity-error
                   v1       = |{ lv_start DATE = USER }|
                   v2       = |{ lv_end DATE = USER }| )
        ) TO reported-tour.
        lv_failed = abap_true.
      ENDIF.

      IF lv_template IS INITIAL.
        APPEND VALUE #(
          %cid = <key>-%cid
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '001'
                   severity = if_abap_behv_message=>severity-error )
        ) TO reported-tour.
        lv_failed = abap_true.
      ENDIF.

      IF lv_failed = abap_true.
        APPEND VALUE #( %cid = <key>-%cid ) TO failed-tour.
      ENDIF.

    ENDLOOP.
  ENDMETHOD.

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
      service_type   TYPE service_type,       " W&R Leistungsart (containerMovementTypeInfo)
      mnamin         TYPE ehs_mnamin,         " Abfallmenge (Planung), QUAN 13,3
      mnamqin        TYPE ehs_mnamqin,        " its unit
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

    " text of the W&R service type (TESERVICETYPET), one language,
    " read once for all EAPs — feeds containerMovementTypeInfo
    BEGIN OF ty_svt_text,
      service_type TYPE service_type,
      vtext        TYPE servicetype_txt,     " TESERVICETYPET-VTEXT, CHAR 30
    END OF ty_svt_text,
    tt_svt_text TYPE HASHED TABLE OF ty_svt_text WITH UNIQUE KEY service_type,

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
    lv_count_error   TYPE i,
    lt_svt_text      TYPE tt_svt_text,
    ls_svt_text      TYPE ty_svt_text,
    lv_qty_info      TYPE abap_bool,         " ZWRTWAALAPRCTP-BMS_QTY_INFO
    lv_qty_text      TYPE string,
    lv_unit_ext      TYPE msehi,             " external unit from CUNIT output
    lv_unit_out      TYPE string.

  " ErrorResponse: { "error": { "message": ..., "timestamp": ... } }
  DATA: BEGIN OF ls_err_body,
          BEGIN OF error,
            message   TYPE string,
            timestamp TYPE string,
          END OF error,
        END OF ls_err_body.

  " Decision D2 (19.10.2026): only MNAMIN = 0 is skipped. Type 03 carries a
  " system default of 1 TO on every item; if the Fachbereich ever sets
  " BMS_QTY_INFO for 03, add that value here.
  CONSTANTS lc_qty_default TYPE ehs_mnamin VALUE 0.

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
           sdaufnr, sdposnr,
           service_type, mnamin, mnamqin
      FROM ewa_order_object
      FOR ALL ENTRIES IN @lt_pobj
      WHERE pobjnr = @lt_pobj-pobjnr
      INTO CORRESPONDING FIELDS OF TABLE @lt_ewaobj.
  ENDIF.

*----------------------------------------------------------------------*
* STEP 4d — texts of the W&R service types (containerMovementTypeInfo)
*           One read for all EAPs. Language 'D': the driver tablet is
*           German and every other payload text is German, independent
*           of the dispatcher's logon language.
*----------------------------------------------------------------------*
  IF lt_ewaobj IS NOT INITIAL.
    SELECT service_type, vtext
      FROM teservicetypet
      FOR ALL ENTRIES IN @lt_ewaobj
      WHERE spras        = 'D'
        AND service_type = @lt_ewaobj-service_type
      INTO CORRESPONDING FIELDS OF TABLE @lt_svt_text.
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
      "     BMS_QTY_INFO (SM30 flag) decides whether the planned quantity
      "     is appended to containerMovementTypeInfo in 6d.
      CLEAR lv_qty_info.

      SELECT SINGLE bms_movement_type, bms_qty_info
        FROM zwrtwaalaprctp
        WHERE service_type = @ls_svc-action
        INTO ( @lv_movement_type, @lv_qty_info ).

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
      " overview. Content, in this order of precedence:
      "   1) TESERVICETYPET-VTEXT of the EAP's service type, first letter
      "      upper, rest lower ("SOFORTBEFÜLLUNG" -> "Sofortbefüllung")
      "   2) the movement word, because the running API rejects an
      "      empty string (old codes ST / LE / F have no VTEXT)
      " followed by the planned waste quantity and unit when the service
      " type asks for it (BMS_QTY_INFO) and MNAMIN is not 0.
      DATA(lv_move_info) = SWITCH string( lv_movement_type
                             WHEN 'new'     THEN 'Aufstellung'
                             WHEN 'collect' THEN 'Einzug'
                             ELSE                'Wechsel' ).

      READ TABLE lt_svt_text INTO ls_svt_text
        WITH TABLE KEY service_type = ls_ewa-service_type.
      IF sy-subrc <> 0.
        READ TABLE lt_svt_text INTO ls_svt_text
          WITH TABLE KEY service_type = CONV service_type( ls_svc-action ).
      ENDIF.
      IF sy-subrc = 0 AND ls_svt_text-vtext IS NOT INITIAL.
        DATA(lv_vtext) = condense( to_lower( ls_svt_text-vtext ) ).
        lv_move_info = |{ to_upper( lv_vtext(1) ) }{ lv_vtext+1 }|.
      ENDIF.

      " planned quantity + unit, German notation: 2,5 t / 3 t / 0,75 m3
      CLEAR: lv_qty_text, lv_unit_out, lv_unit_ext.

      IF lv_qty_info = abap_true
         AND ls_ewa-mnamin IS NOT INITIAL
         AND ls_ewa-mnamin <> lc_qty_default.

        lv_qty_text = |{ ls_ewa-mnamin NUMBER = RAW }|.         " 2.500
        IF lv_qty_text CS '.'.
          SHIFT lv_qty_text RIGHT DELETING TRAILING '0'.        " 2.5
          SHIFT lv_qty_text RIGHT DELETING TRAILING '.'.        " 3
          CONDENSE lv_qty_text NO-GAPS.
          REPLACE '.' IN lv_qty_text WITH ','.                  " 2,5
        ENDIF.

        " external unit (t, kg, m3) instead of the internal key (TO, KG, M3)
        CALL FUNCTION 'CONVERSION_EXIT_CUNIT_OUTPUT'
          EXPORTING
            input    = ls_ewa-mnamqin
            language = 'D'
          IMPORTING
            output   = lv_unit_ext
          EXCEPTIONS
            OTHERS   = 1.
        lv_unit_out = COND string( WHEN sy-subrc = 0 AND lv_unit_ext IS NOT INITIAL
                                   THEN condense( lv_unit_ext )
                                   ELSE condense( ls_ewa-mnamqin ) ).

        lv_move_info = condense( |{ lv_move_info } { lv_qty_text } { lv_unit_out }| ).
      ENDIF.

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

METHOD stornobmsservice.

*----------------------------------------------------------------------*
* There is no storno endpoint in the BMS API. Cancellation is the SAME
* create-order-halle call with status "cancelled". The payload is
* replayed from ZBMS_API_LOG so BMS sees exactly the order it accepted.
*
* Prerequisite: ZBMS_API_LOG-REQUEST_PAYLOAD must be type STRING.
* At CHAR 1333 the stored JSON is truncated and replays as invalid.
*----------------------------------------------------------------------*
  TYPES:
    BEGIN OF ty_svc_storno,
      service_uuid     TYPE /plce/pdservice_uuid,
      service_id       TYPE /plce/pdservice_id,
      reference_id     TYPE char30,
      reference_int_id TYPE char30,
    END OF ty_svc_storno,
    tt_svc_storno TYPE HASHED TABLE OF ty_svc_storno
                  WITH UNIQUE KEY service_uuid.

  DATA:
    lt_services     TYPE tt_svc_storno,
    ls_svc          TYPE ty_svc_storno,
    ls_srvcst       TYPE /plce/tpdsrvcst,
    lv_dest         TYPE rfcdest,
    lv_token        TYPE string,
    lv_error        TYPE string,
    lv_http_status  TYPE i,
    lv_response     TYPE string,
    lv_json         TYPE string,
    lv_storno_ok    TYPE i,
    lv_storno_err   TYPE i,
    lv_pobjnr       TYPE j_objnr,
    lv_sto_subrc    TYPE sy-subrc,
    lv_order_number TYPE aufnr.

  DATA: BEGIN OF ls_err_body,
          BEGIN OF error,
            message   TYPE string,
            timestamp TYPE string,
          END OF error,
        END OF ls_err_body.

*----------------------------------------------------------------------*
* CONFIG — endpoint comes from an SM59 destination
*----------------------------------------------------------------------*
  SELECT SINGLE bms_destination, bms_username, bms_password, active
    FROM ztour_bms_cfg
    WHERE config_id = 'DEFAULT'
    INTO @DATA(ls_cfg).

  IF sy-subrc <> 0 OR ls_cfg-active is  INITIAL
     OR ls_cfg-bms_destination IS INITIAL.
    LOOP AT keys ASSIGNING FIELD-SYMBOL(<ky>).
      APPEND VALUE #(
        %tky = <ky>-%tky
        %msg = new_message(
                 id       = 'Z_MSG_SVR_TOUR_EXT'
                 number   = '020'
                 severity = if_abap_behv_message=>severity-error )
      ) TO reported-tour.
      APPEND VALUE #( %tky = <ky>-%tky ) TO failed-tour.
    ENDLOOP.
    RETURN.
  ENDIF.

  lv_dest = ls_cfg-bms_destination.

  DATA(lv_bms_user)     = ls_cfg-bms_username .
  DATA(lv_bms_password) =  ls_cfg-bms_password .

*----------------------------------------------------------------------*
* Tours
*----------------------------------------------------------------------*
  READ ENTITIES OF /plce/r_pdtour IN LOCAL MODE
    ENTITY tour
      FIELDS ( tourid touruuid )
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
* Service assignments
*----------------------------------------------------------------------*
  READ ENTITIES OF /plce/r_pdtour IN LOCAL MODE
    ENTITY tour BY \_serviceassignments
      FIELDS ( touruuid serviceuuid removed )
      WITH CORRESPONDING #( keys )
    RESULT DATA(lt_asgmts).

  DELETE lt_asgmts WHERE removed IS NOT INITIAL.

  IF lt_asgmts IS INITIAL.
    RETURN.
  ENDIF.

  SELECT service_uuid, service_id, reference_id, reference_int_id
    FROM /plce/tpdsrv
    FOR ALL ENTRIES IN @lt_asgmts
    WHERE service_uuid = @lt_asgmts-serviceuuid
    INTO CORRESPONDING FIELDS OF TABLE @lt_services.

*----------------------------------------------------------------------*
* Main loop
*----------------------------------------------------------------------*
  LOOP AT lt_tours INTO DATA(ls_tour).

    CLEAR: lv_storno_ok, lv_storno_err, lv_token, lv_error.

    DATA(lv_tour_id_out) = condense( |{ ls_tour-tourid ALPHA = OUT }| ).

    zcl_wr_pd_tour_helper=>get_bms_bearer_token(
      EXPORTING
        iv_destination = lv_dest
        iv_username    = lv_bms_user
        iv_password    = lv_bms_password
      IMPORTING
        ev_token       = lv_token
        ev_error       = lv_error ).

    IF lv_error IS NOT INITIAL.
      APPEND VALUE #(
        %tky = ls_tour-%tky
        %msg = new_message_with_text(
                 severity = if_abap_behv_message=>severity-error
                 text     = lv_error )
      ) TO reported-tour.
      APPEND VALUE #( %tky = ls_tour-%tky ) TO failed-tour.
      CONTINUE.
    ENDIF.

    LOOP AT lt_asgmts INTO DATA(ls_asgmt)
      WHERE touruuid = ls_tour-touruuid.

      CLEAR: ls_svc, lv_http_status, lv_response, lv_json,
             lv_pobjnr, lv_order_number, lv_sto_subrc.

      READ TABLE lt_services INTO ls_svc
        WITH TABLE KEY service_uuid = ls_asgmt-serviceuuid.

      IF sy-subrc <> 0.
        lv_storno_err = lv_storno_err + 1.
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

      " Only cancel what was actually released
      SELECT SINGLE zz_bms_status
        FROM /plce/tpdsrvcst
        WHERE service_uuid = @ls_asgmt-serviceuuid
        INTO @DATA(lv_cur_status).

      IF sy-subrc <> 0 OR lv_cur_status <> 'FREIGEGEBEN'.
        CONTINUE.   " never released, or already cancelled — nothing to do
      ENDIF.

      " Replay the payload BMS accepted
      SELECT SINGLE order_number, request_payload
        FROM zbms_api_log
        WHERE tour_uuid    = @ls_tour-touruuid
          AND service_uuid = @ls_asgmt-serviceuuid
        INTO ( @lv_order_number, @lv_json ).

      IF sy-subrc <> 0 OR lv_json IS INITIAL.
        lv_storno_err = lv_storno_err + 1.
        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '027'
                   severity = if_abap_behv_message=>severity-error
                   v1       = lv_svc_ref_out )
        ) TO reported-tour.
        CONTINUE.
      ENDIF.

      " Switch the status. This check is critical: if the pattern does not
      " match, the payload still says "ok" and posting it would CREATE the
      " order again instead of cancelling it.
      REPLACE FIRST OCCURRENCE OF '"status":"ok"'
        IN lv_json WITH '"status":"cancelled"'.

      IF sy-subrc <> 0.
        lv_storno_err = lv_storno_err + 1.
        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '031'
                   severity = if_abap_behv_message=>severity-error
                   v1       = lv_svc_ref_out )
        ) TO reported-tour.
        CONTINUE.
      ENDIF.

      lv_pobjnr = ls_svc-reference_int_id.

      zcl_wr_pd_tour_helper=>post_bms_order(
        EXPORTING
          iv_destination  = lv_dest
          iv_bearer_token = lv_token
          iv_json         = lv_json
        IMPORTING
          ev_http_status  = lv_http_status
          ev_response     = lv_response ).

      zcl_wr_pd_tour_helper=>log_bms_call(
        iv_tour_uuid    = ls_tour-touruuid
        iv_service_uuid = ls_asgmt-serviceuuid
        iv_order_number = lv_order_number
        iv_endpoint     = '/api/container/create-order-halle (CANCEL)'
        iv_http_status  = lv_http_status
        iv_request      = lv_json
        iv_response     = lv_response
        iv_pobjnr       = lv_pobjnr ).

      IF lv_http_status = 200 OR lv_http_status = 201.

        lv_storno_ok = lv_storno_ok + 1.

        CALL FUNCTION 'ZWR_BMS_UPDATE_SERVICE'
          DESTINATION 'NONE'
          EXPORTING
            service_uuid          = ls_asgmt-serviceuuid
            pobjnr                = lv_pobjnr
            zz_bms_status         = 'STORNIERT'
          IMPORTING
            ev_subrc              = lv_sto_subrc
          EXCEPTIONS
            communication_failure = 1
            system_failure        = 2
            OTHERS                = 3.

        IF sy-subrc <> 0 OR lv_sto_subrc <> 0.
          CLEAR ls_srvcst.
          ls_srvcst-service_uuid  = ls_asgmt-serviceuuid.
          ls_srvcst-zz_bms_status = 'STORNIERT'.
          INSERT /plce/tpdsrvcst FROM ls_srvcst.
        ENDIF.

        APPEND VALUE #(
          %tky = ls_tour-%tky
          %msg = new_message(
                   id       = 'Z_MSG_SVR_TOUR_EXT'
                   number   = '032'
                   severity = if_abap_behv_message=>severity-success
                   v1       = lv_order_number )
        ) TO reported-tour.

      ELSE.

        lv_storno_err = lv_storno_err + 1.

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
                   number   = '033'
                   severity = if_abap_behv_message=>severity-error
                   v1       = lv_order_number
                   v2       = lv_bms_msg )
        ) TO reported-tour.

      ENDIF.

    ENDLOOP.   " services

*----------------------------------------------------------------------*
* Nothing was cancelled, but something was attempted.
*     Both counters zero = no service was in FREIGEGEBEN, a legitimate
*     no-op that must not be reported as a failure.
*----------------------------------------------------------------------*
    IF lv_storno_ok = 0 AND lv_storno_err > 0.
      APPEND VALUE #( %tky = ls_tour-%tky ) TO failed-tour.
      CONTINUE.
    ENDIF.

    DATA(lv_tour_status) = COND /plce/char20(
      WHEN lv_storno_ok > 0 AND lv_storno_err = 0 THEN 'STORNIERT'
      WHEN lv_storno_ok > 0                       THEN 'PARTIAL'
      WHEN lv_storno_err > 0                      THEN 'ERROR'
      ELSE                                             'STORNIERT' ).

    MODIFY ENTITIES OF /plce/r_pdtour IN LOCAL MODE
      ENTITY extcustom
        UPDATE FIELDS ( zz_bms_status )
        WITH VALUE #( ( touruuid      = ls_tour-touruuid
                        zz_bms_status = lv_tour_status ) )
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
                %cid          = |BMS_STORNO_TGT_{ ls_tour-touruuid }|
                zz_bms_status = lv_tour_status ) ) ) )
        FAILED   DATA(lf_crt)
        REPORTED DATA(lr_crt).

      IF lf_crt IS NOT INITIAL.
        APPEND LINES OF lr_crt-tour TO reported-tour.
      ENDIF.
    ENDIF.

  ENDLOOP.   " tours

ENDMETHOD.


  METHOD tourgeneratedocument.

    DATA:
      lt_srv_all       TYPE zpd_service_tt,
      lt_services_tour TYPE zpd_service_tt,
      lt_tour_create   TYPE TABLE FOR CREATE /plce/r_pdtour\_Attachments,
      lt_srv_create    TYPE TABLE FOR CREATE /plce/r_pdservice\_Attachments,
      lv_ok            TYPE i,
      lv_err           TYPE i,
      lv_removed       TYPE i,
      lv_cid           TYPE i.

    DATA(lo_misc) = zcl_wr_misc_pd=>get_instance( ).

*----------------------------------------------------------------------*
* The complete print configuration goes in. build_for_tour() reduces it
* per tour template via ZWR_PDFORMS; create_manual_service_attm()
* resolves the service forms itself via ZWR_PDFORMS_SRV.
*----------------------------------------------------------------------*
    SELECT * FROM zc_printconfig INTO TABLE @DATA(lt_all_config).

    IF lt_all_config IS INITIAL.
      LOOP AT keys ASSIGNING FIELD-SYMBOL(<ls_k>).
        APPEND VALUE #( %tky = <ls_k>-%tky ) TO failed-tour.
        APPEND VALUE #( %tky = <ls_k>-%tky
                        %msg = new_message( id       = 'Z_MSG_SVR_TOUR_EXT'
                                            number   = '041'
                                            severity = if_abap_behv_message=>severity-error )
                      ) TO reported-tour.
      ENDLOOP.
      RETURN.
    ENDIF.

*--- 1  the selected tours ---------------------------------------------
    READ ENTITIES OF /plce/r_pdtour IN LOCAL MODE
      ENTITY tour
        FIELDS ( touruuid tourid tourtemplate )
        WITH CORRESPONDING #( keys )
      RESULT DATA(lt_tours)
      FAILED DATA(lt_read_failed).

    " an error in REPORTED is downgraded to a warning unless the key is
    " also in FAILED — same trap as in touranBMSfreigeben
    LOOP AT lt_read_failed-tour ASSIGNING FIELD-SYMBOL(<ls_rf>).
      APPEND VALUE #( %tky = <ls_rf>-%tky ) TO failed-tour.
      APPEND VALUE #( %tky = <ls_rf>-%tky
                      %msg = new_message( id       = 'Z_MSG_SVR_TOUR_EXT'
                                          number   = '024'
                                          severity = if_abap_behv_message=>severity-error )
                    ) TO reported-tour.
    ENDLOOP.

    CHECK lt_tours IS NOT INITIAL.

*--- 2  the services of all selected tours, read once -------------------
    READ ENTITIES OF /plce/r_pdtour IN LOCAL MODE
      ENTITY tour BY \_serviceassignments
        FIELDS ( touruuid serviceuuid removed )
        WITH CORRESPONDING #( keys )
      RESULT DATA(lt_asgmts).

    DELETE lt_asgmts WHERE removed IS NOT INITIAL.

    IF lt_asgmts IS NOT INITIAL.

      READ ENTITIES OF /plce/r_pdservice
        ENTITY service
          FIELDS ( serviceuuid servicetype )
          WITH VALUE #( FOR ls_asg IN lt_asgmts ( serviceuuid = ls_asg-serviceuuid ) )
        RESULT DATA(lt_srv).

      READ ENTITIES OF /plce/r_pdservice
        ENTITY service BY \_extwaste
          FIELDS ( serviceuuid material )
          WITH CORRESPONDING #( lt_srv )
        RESULT DATA(lt_mat).

      lt_srv_all = VALUE #(
        FOR ls_s IN lt_srv
        ( service_uuid = ls_s-serviceuuid
          service_type = ls_s-servicetype
          material     = VALUE #( lt_mat[ serviceuuid = ls_s-serviceuuid ]-material OPTIONAL ) ) ).

    ENDIF.

*--- 3  per tour --------------------------------------------------------
    LOOP AT lt_tours ASSIGNING FIELD-SYMBOL(<ls_tour>).

      CLEAR: lv_ok, lv_err, lv_removed,
             lt_services_tour, lt_tour_create, lt_srv_create.

      DATA(lv_tour_out) = condense( |{ <ls_tour>-tourid ALPHA = OUT }| ).

*     3a  tour documents
      DATA(ls_conf) = lo_misc->build_for_tour(
                        is_tour       = CORRESPONDING #( <ls_tour> )
                        it_sel_config = lt_all_config ).

      IF ls_conf IS NOT INITIAL.

        " render FIRST — nothing is deleted until there is a replacement
        DATA(lt_tour_attm) = lo_misc->create_manual_tour_attm(
                               is_tour_with_conf = ls_conf ).

        IF lt_tour_attm IS NOT INITIAL.

         lv_removed = remove_obsolete_tour( iv_touruuid = <ls_tour>-touruuid
                                            it_new      = lt_tour_attm ).

          " build one CREATE-BY row for this tour, with all its documents
          DATA ls_tour_create LIKE LINE OF lt_tour_create.
          CLEAR ls_tour_create.
          ls_tour_create-%tky = <ls_tour>-%tky.

          LOOP AT lt_tour_attm ASSIGNING FIELD-SYMBOL(<ls_ta>).
            lv_cid = lv_cid + 1.
            APPEND VALUE #( %cid       = |T{ lv_cid }|
                            filename   = <ls_ta>-filename
                            mimetype   = <ls_ta>-mimetype
                            attachment = <ls_ta>-attachment
                            comments   = <ls_ta>-comments ) TO ls_tour_create-%target.
          ENDLOOP.

          APPEND ls_tour_create TO lt_tour_create.

          MODIFY ENTITIES OF /plce/r_pdtour IN LOCAL MODE
            ENTITY tour
              CREATE BY \_Attachments
                FIELDS ( filename mimetype attachment comments )
                WITH lt_tour_create
            FAILED DATA(lt_fail_t).

          " test the whole structure — the child alias cannot be named safely
          IF lt_fail_t IS NOT INITIAL.
            lv_err = lv_err + 1.
            APPEND VALUE #( %tky = <ls_tour>-%tky
                            %msg = new_message( id       = 'Z_MSG_SVR_TOUR_EXT'
                                                number   = '038'
                                                severity = if_abap_behv_message=>severity-error
                                                v1       = lv_tour_out )
                          ) TO reported-tour.
          ELSE.
            lv_ok = lv_ok + lines( lt_tour_attm ).
            APPEND VALUE #( %tky = <ls_tour>-%tky
                            %msg = new_message( id       = 'Z_MSG_SVR_TOUR_EXT'
                                                number   = '036'
                                                severity = if_abap_behv_message=>severity-success
                                                v1       = |{ lines( lt_tour_attm ) }|
                                                v2       = lv_tour_out )
                          ) TO reported-tour.
          ENDIF.

          IF lv_removed > 0.
            APPEND VALUE #( %tky = <ls_tour>-%tky
                            %msg = new_message( id       = 'Z_MSG_SVR_TOUR_EXT'
                                                number   = '039'
                                                severity = if_abap_behv_message=>severity-information
                                                v1       = |{ lv_removed }|
                                                v2       = lv_tour_out )
                          ) TO reported-tour.
          ENDIF.

        ENDIF.
      ENDIF.

*     3b  service documents
      lt_services_tour = VALUE #(
        FOR ls_asg2 IN lt_asgmts WHERE ( touruuid = <ls_tour>-touruuid )
        ( VALUE #( lt_srv_all[ service_uuid = ls_asg2-serviceuuid ] OPTIONAL ) ) ).

      DELETE lt_services_tour WHERE service_uuid IS INITIAL.

      IF lt_services_tour IS NOT INITIAL.

        DATA(lt_srv_attm) = lo_misc->create_manual_service_attm(
                              it_selected_services = lt_services_tour ).

        IF lt_srv_attm IS NOT INITIAL.

          DATA ls_srv_create LIKE LINE OF lt_srv_create.

          LOOP AT lt_srv_attm ASSIGNING FIELD-SYMBOL(<ls_sa>)
               GROUP BY ( uuid = <ls_sa>-serviceuuid )
               ASSIGNING FIELD-SYMBOL(<grp_srv>).

           lv_removed = lv_removed + remove_obsolete_service(
                                       iv_serviceuuid = <grp_srv>-uuid
                                       it_new         = lt_srv_attm ).

            CLEAR ls_srv_create.
            ls_srv_create-serviceuuid = <grp_srv>-uuid.

            LOOP AT GROUP <grp_srv> ASSIGNING FIELD-SYMBOL(<ls_member>).
              lv_cid = lv_cid + 1.
              APPEND VALUE #( %cid       = |S{ lv_cid }|
                              filename   = <ls_member>-filename
                              mimetype   = <ls_member>-mimetype
                              attachment = <ls_member>-attachment
                              comments   = <ls_member>-comments ) TO ls_srv_create-%target.
            ENDLOOP.

            APPEND ls_srv_create TO lt_srv_create.

          ENDLOOP.

          MODIFY ENTITIES OF /plce/r_pdservice
            ENTITY service
              CREATE BY \_Attachments
                FIELDS ( filename mimetype attachment comments )
                WITH lt_srv_create
            FAILED DATA(lt_fail_s).

          IF lt_fail_s IS NOT INITIAL.
            lv_err = lv_err + 1.
            APPEND VALUE #( %tky = <ls_tour>-%tky
                            %msg = new_message( id       = 'Z_MSG_SVR_TOUR_EXT'
                                                number   = '040'
                                                severity = if_abap_behv_message=>severity-error
                                                v1       = lv_tour_out )
                          ) TO reported-tour.
          ELSE.
            lv_ok = lv_ok + lines( lt_srv_attm ).
            APPEND VALUE #( %tky = <ls_tour>-%tky
                            %msg = new_message( id       = 'Z_MSG_SVR_TOUR_EXT'
                                                number   = '037'
                                                severity = if_abap_behv_message=>severity-success
                                                v1       = |{ lines( lt_srv_attm ) }|
                                                v2       = lv_tour_out )
                          ) TO reported-tour.
          ENDIF.

        ENDIF.
      ENDIF.

*     3c  nothing configured — information, not an error
      IF lv_ok = 0 AND lv_err = 0.
        APPEND VALUE #( %tky = <ls_tour>-%tky
                        %msg = new_message( id       = 'Z_MSG_SVR_TOUR_EXT'
                                            number   = '035'
                                            severity = if_abap_behv_message=>severity-information
                                            v1       = <ls_tour>-tourtemplate
                                            v2       = lv_tour_out )
                      ) TO reported-tour.
        CONTINUE.
      ENDIF.

*     3d  outcome — a partial result deliberately does NOT fail
      IF lv_ok = 0 AND lv_err > 0.
        APPEND VALUE #( %tky = <ls_tour>-%tky ) TO failed-tour.
      ELSE.
        APPEND VALUE #( %tky   = <ls_tour>-%tky
                        %param = CORRESPONDING #( <ls_tour> ) ) TO result.
      ENDIF.

    ENDLOOP.

  ENDMETHOD.



ENDCLASS.

CLASS zcl_bms_inbound_handler DEFINITION
  PUBLIC FINAL
  CREATE PUBLIC.

  PUBLIC SECTION.
    INTERFACES if_http_extension.

ENDCLASS.



CLASS ZCL_BMS_INBOUND_HANDLER IMPLEMENTATION.


  METHOD if_http_extension~handle_request.

    "-----------------------------------------------------------------------
    " Inbound REST endpoint for BMS → SAP status callbacks.
    " BMS POSTs JSON: { "orderNumber": "...", "status": "BEGONNEN" }
    " SAP looks up the service UUID and updates ZZ_BMS_STATUS.
    " Registered in SICF: /sap/bc/zbms/status_update
    "-----------------------------------------------------------------------

    " Only accept POST
    IF server->request->get_method( ) <> 'POST'.
      server->response->set_status( code = 405 reason = 'Method Not Allowed' ).
      server->response->set_cdata( '{"error":"POST required"}' ).
      RETURN.
    ENDIF.

    " Read and validate body
    DATA(lv_body) = server->request->get_cdata( ).
    IF lv_body IS INITIAL.
      server->response->set_status( code = 400 reason = 'Bad Request' ).
      server->response->set_cdata( '{"error":"Empty body"}' ).
      RETURN.
    ENDIF.

    " Deserialize JSON callback
    " Full BEENDET payload may also carry actualDuration, wasteWeight etc.
    TYPES: BEGIN OF ty_callback,
             order_number       TYPE string,
             status             TYPE string,
             actual_duration    TYPE i,
             waste_weight       TYPE decfloat16,
             weight_unit        TYPE string,
             execution_time     TYPE string,
             signature_obtained TYPE abap_bool,
           END OF ty_callback.
    DATA ls_cb TYPE ty_callback.

    /ui2/cl_json=>deserialize(
      EXPORTING
        json        = lv_body
        pretty_name = /ui2/cl_json=>pretty_mode-camel_case
      CHANGING
        data        = ls_cb ).

    " Validate orderNumber
    IF ls_cb-order_number IS INITIAL.
      server->response->set_status( code = 400 reason = 'Bad Request' ).
      server->response->set_cdata( '{"error":"orderNumber is required"}' ).
      RETURN.
    ENDIF.

    " Validate status value
    IF ls_cb-status <> 'BEGONNEN'
    AND ls_cb-status <> 'BEENDET'
    AND ls_cb-status <> 'STORNIERT'.
      server->response->set_status( code = 400 reason = 'Bad Request' ).
      server->response->set_cdata(
        |\{ "error": "Invalid status '{ ls_cb-status }'. | &&
        |Expected BEGONNEN, BEENDET or STORNIERT" \}| ).
      RETURN.
    ENDIF.

    " Convert orderNumber to internal format (ALPHA=IN)
    DATA lv_smaufnr TYPE aufnr.
    lv_smaufnr = |{ ls_cb-order_number ALPHA = IN }|.

    " Look up POBJNR via EWA_ORDER_OBJECT
    SELECT SINGLE pobjnr
      FROM ewa_order_object
      WHERE smaufnr = @lv_smaufnr
      INTO @DATA(lv_pobjnr).

    IF sy-subrc <> 0.
      server->response->set_status( code = 404 reason = 'Not Found' ).
      server->response->set_cdata(
        |\{ "error": "Order '{ ls_cb-order_number }' not found in SAP" \}| ).
      RETURN.
    ENDIF.

    " Find service UUID via /plce/tpdsrv
    SELECT SINGLE service_uuid
      FROM /plce/tpdsrv
      WHERE reference_int_id = @lv_pobjnr
      INTO @DATA(lv_service_uuid).

    IF sy-subrc <> 0.
      server->response->set_status( code = 404 reason = 'Not Found' ).
      server->response->set_cdata(
        |\{ "error": "Service for order '{ ls_cb-order_number }' not found" \}| ).
      RETURN.
    ENDIF.

    " Update ZZ_BMS_STATUS on /plce/tpdsrvcst
    UPDATE /plce/tpdsrvcst
      SET zz_bms_status = @ls_cb-status
      WHERE service_uuid = @lv_service_uuid.

    IF sy-subrc <> 0.
      " Row does not exist yet — insert it
      DATA ls_srvcst TYPE /plce/tpdsrvcst.
      CLEAR ls_srvcst.
*      ls_srvcst-mandt         = sy-mandt.
      ls_srvcst-service_uuid  = lv_service_uuid.
      ls_srvcst-zz_bms_status = ls_cb-status.
      INSERT /plce/tpdsrvcst FROM ls_srvcst.
    ENDIF.

" Log the inbound call
    DATA lv_empty_uuid TYPE /plce/pdtour_uuid.
    CLEAR lv_empty_uuid.

    zcl_wr_pd_tour_helper=>log_bms_call(
      iv_tour_uuid    = lv_empty_uuid       " ← initial UUID, not empty string
      iv_service_uuid = lv_service_uuid
      iv_order_number = lv_smaufnr
      iv_endpoint     = '/sap/bc/zbms/status_update'
      iv_http_status  = 200
      iv_request      = lv_body
      iv_response     = 'OK'
      iv_direction    = 'INBOUND' ).


    " Respond with success
    server->response->set_status( code = 200 reason = 'OK' ).
    server->response->set_header_field(
      name  = 'Content-Type'
      value = 'application/json' ).
    server->response->set_cdata(
      |\{ "result": "OK", | &&
      |"orderNumber": "{ ls_cb-order_number }", | &&
      |"status": "{ ls_cb-status }" \}| ).

  ENDMETHOD.
ENDCLASS.

class ZCL_IM_WR_PD_WA_ORD_PD_DWNLD definition
  public
  final
  create public .

public section.

  interfaces IF_BADI_INTERFACE .
  interfaces /PLCP/IF_WA_ORDER_DOWNLOAD .
protected section.
private section.
ENDCLASS.



CLASS ZCL_IM_WR_PD_WA_ORD_PD_DWNLD IMPLEMENTATION.


  method /PLCP/IF_WA_ORDER_DOWNLOAD~DATA_DISPATCH.
     new ZCL_WR_PD_WA_WDOI_PDSERVICE( X_LOG_HANDLE )->CREATE_BY_PRINTDATA(
      exporting
        IS_PRINT_DATA = X_PRINT_DATA
    ).
  endmethod.
ENDCLASS.

class ZCL_PD_PROC_WASTE_ORDERS definition
  public
  inheriting from /PLCE/CL_PD_PROC_WASTE_ORDERS
  create public .

public section.

  types:
    begin of ZZ_TY_DATA,
    SRV_UPD      type table for update /PLCE/R_PDSERVICE,
    SRV_UPD_UN   type table for update /PLCE/R_PDSERVICEEXTUNI,
    SRV_UPD_WA   type table for update /PLCE/R_PDSERVICEEXTWR,
    SRV_UPD_CUST type table for update /PLCE/R_PDServiceExtCustom,
    TSKS_DELE    type table for delete /PLCE/R_PDSERVICETASK,
    TSKS_DELE_UN type table for delete /PLCE/R_PDSERVICETASK,
    TSKS_DELE_WA type table for delete /PLCE/R_PDSERVICETASK,
    TSKS_CREA    type table for create /PlCE/R_PDSERVICE\_SERVICETASK,
    TSKS_CREA_UN type table for create /PLCE/R_PDSERVICETASK\_EXTUNIVERSAL,
    TSKS_CREA_WA type table for create /PLCE/R_PDSERVICETASK\_EXTWASTE.
  types end of ZZ_TY_DATA .
  types:
    begin of ZTP_SERVICE_EXT,
        SERVICE      type ref to /PLCE/R_PDSERVICE,
        EXTUNIVERSAL type ref to /PLCE/R_PDServiceExtUNI,
        EXTWASTE     type ref to /PLCE/R_PDServiceExtWR,
        EXTCustom    type ref to /PLCE/R_PDServiceExtCustom,
      end of ZTP_SERVICE_EXT .
  types:
    begin of ZTS_MODIFY_SERVICE,
        CID            type ABP_BEHV_CID,
        SERVICE        type /PLCE/R_PDSERVICE,
        EXTUNIVERSAL   type /PLCE/R_PDServiceExtUNI,
        EXTWASTE       type /PLCE/R_PDServiceExtWR,
        EXTCustom    type ref to /PLCE/R_PDServiceExtCustom,
        T_SERVICE_TASK type TP_SERVICE_TASK,
      end of ZTS_MODIFY_SERVICE .
  types:
    ZTP_MODIFY_SERVICE type sorted table of ZTS_MODIFY_SERVICE with unique key CID .
  types:
    begin of ZTS_MODIFY_SERVICE_EXT,
        CID            type ABP_BEHV_CID,
        SERVICE        type /PLCE/R_PDSERVICE,
        EXTUNIVERSAL   type /PLCE/R_PDServiceExtUNI,
        EXTWASTE       type /PLCE/R_PDServiceExtWR,
        EXTCUSTOM      type /PLCE/R_PDServiceExtCustom,
        ATTACHMENTS    type TP_MODIFY_SERVICE_ATTACHEMNT,
        T_SERVICE_TASK type TP_MODIFY_SERVICE_TASK_EXT,
        T_SERVICE_NOTES type TP_MODIFY_SERVICE_NOTE_EXT,
      end of ZTS_MODIFY_SERVICE_EXT .
  types:
    ZTP_MODIFY_SERVICE_EXT type sorted table of ZTS_MODIFY_SERVICE_EXT with unique key CID .

  class-methods ZZ_CHANGE_SERVICE_TYPE
    importing
      !PAR_KEY type /PLCE/SPDSRV_KEY
      !PAR_SVTPTO type /PLCE/PDSERVICE_TYPE
      !PAR_LOG type ref to /PLCE/CL_APPLLOG_HELPER optional
      !PAR_COMMIT type KENNZX optional
      !PAR_SIMULATE type KENNZX optional
    raising
      /PLCE/CX_BASEEXCEPTION .
  class-methods ZZ_FILL_SERVICE_TASK
    importing
      !PAR_KEY type /PLCE/SPDSRV_KEY
      !PAR_SVTPTO type /PLCE/PDSERVICE_TYPE
      !PAR_LOG type ref to /PLCE/CL_APPLLOG_HELPER optional
    returning
      value(PAR_RESULT) type ZZ_TY_DATA
    raising
      /PLCE/CX_BASEEXCEPTION .

  methods PROCESS_WASTEORDERS
    redefinition .
protected section.

  data AD_SERVICE type ref to ZTP_SERVICE_EXT .
  data AT_SERVICE_TO_CREATE type ZTP_MODIFY_SERVICE_EXT .

  methods FILL_SERVICES
    redefinition .
  methods FILL_SERVICE_DATA
    redefinition .
  methods FILL_SERVICE_TASK_DATA
    redefinition .
  methods PROCESS_UPDATE
    redefinition .
  methods SAVE_SERVICE
    redefinition .
private section.
ENDCLASS.



CLASS ZCL_PD_PROC_WASTE_ORDERS IMPLEMENTATION.


  method FILL_SERVICES.

    data:
        LSEQUENCE type I.

*    try.

    loop at IT_WASTE_ORDER_ITEM_DATA reference into FR_WASTE_ORDER_ITEM.
      clear LSEQUENCE.
*          if SHOULD_SERVICE_BE_CREATED( ).
      data(LV_ACTION) = GET_ACTION( ).
      if LV_ACTION is initial.
        FO_MSGLOG->/PLCE/IF_PROTOCOL~REPORT_MESSAGE(
          exporting
            IV_SEVERITY   = FO_MSGLOG->/PLCE/IF_PROTOCOL~C_SEVERITY-ERROR
            IV_ID         = '/PLCE/MC_PD_MSG'
            IV_NUMBER     = '110'
            IV_VARIABLE_1 = conv #( FR_WASTE_ORDER_ITEM->WASTE_TYPE )
        ).
      endif.
      data(LST_TEMPLS) = READ_SERVICE_TEMPLATES( exporting IV_ACTION = LV_ACTION ).
      loop at LST_TEMPLS assigning field-symbol(<SERVICETMPL>)
        where SemanticTemplateName <> 'ST_SERVICE_NOTE'.
        insert value #( CID = GET_NEXT_CID( ) ) into table AT_SERVICE_TO_CREATE assigning field-symbol(<LS_SERVICE_TO_CREATE>).
        create data AD_SERVICE.
        AD_SERVICE->SERVICE = ref #( <LS_SERVICE_TO_CREATE>-SERVICE ).
        AD_SERVICE->EXTUNIVERSAL = ref #( <LS_SERVICE_TO_CREATE>-EXTUNIVERSAL ).
        AD_SERVICE->EXTWASTE = ref #( <LS_SERVICE_TO_CREATE>-EXTWASTE ).
        AD_SERVICE->EXTCUSTOM = ref #( <LS_SERVICE_TO_CREATE>-EXTCUSTOM ).
*              append initial line to FT_SERVICE_TO_CREATE reference into AD_SERVICE.
        FILL_SERVICE_DATA(
          exporting
            IS_SERVICE_TEMPLATE = <SERVICETMPL>
        ).
        if LINES( FR_WASTE_ORDER_ITEM->NOTES ) is not initial.
          loop at FR_WASTE_ORDER_ITEM->NOTES reference into data(LWASTESERVICENOTEREF).
            insert initial line into table <LS_SERVICE_TO_CREATE>-T_SERVICE_NOTES reference into data(LSERVICENOTEREF).
            FO_ACCESS_POOL->SET_CONTEXT(
              IV_SEMANTIC_CONTEXT = 'EWA_WASTE_SERVICE_NOTE' "/PLCE/CL_SEMANTIC_CONSTANTS=>C_SEMANTIC_CONTEXT-???
              IR_DATA             = LWASTESERVICENOTEREF
            ).
            FO_ACCESS_POOL->SET_CONTEXT(
              IV_SEMANTIC_CONTEXT = 'SERVICE_NOTE' "/PLCE/CL_SEMANTIC_CONSTANTS=>C_SEMANTIC_CONTEXT-???
              IR_DATA             = ref #( LSERVICENOTEREF->SERVICE_NOTE )
            ).

            FO_MAPPER->EXECUTE_SEMANTIC_TEMPL_CONFIG(
              IV_PROFILE                = GET_PROFILE( )
              IV_SEMANTIC_TEMPLATE_NAME = 'ST_SERVICE_NOTE'
            ).

          endloop.

        endif.


        if LINES( FR_WASTE_ORDER_ITEM->ATTACHMENTS ) is not initial.
          loop at FR_WASTE_ORDER_ITEM->ATTACHMENTS reference into data(LWASTEATTACHMENTREF).
            insert initial line into table <LS_SERVICE_TO_CREATE>-ATTACHMENTS reference into data(LSERVICEATTAREF).
*                FO_ACCESS_POOL->SET_CONTEXT(
*                  IV_SEMANTIC_CONTEXT = 'EWA_WASTE_SERVICE_NOTE' "/PLCE/CL_SEMANTIC_CONSTANTS=>C_SEMANTIC_CONTEXT-???
*                  IR_DATA             = LWASTESERVICENOTEREF
*                ).
*                FO_ACCESS_POOL->SET_CONTEXT(
*                  IV_SEMANTIC_CONTEXT = 'SERVICE_NOTE' "/PLCE/CL_SEMANTIC_CONSTANTS=>C_SEMANTIC_CONTEXT-???
*                  IR_DATA             = ref #( LSERVICENOTEREF->SERVICE_NOTE )
*                ).
*
*                FO_MAPPER->EXECUTE_SEMANTIC_TEMPL_CONFIG(
*                  IV_PROFILE                = GET_PROFILE( )
*                  IV_SEMANTIC_TEMPLATE_NAME = 'ST_SERVICE_NOTE'
*                ).
            LSERVICEATTAREF->ATTACHMENT-Filename = LWASTEATTACHMENTREF->FILENAME.
            LSERVICEATTAREF->ATTACHMENT-Attachment = LWASTEATTACHMENTREF->ATTACHMENT.
            LSERVICEATTAREF->ATTACHMENT-MimeType = LWASTEATTACHMENTREF->MIMETYPE.
            LSERVICEATTAREF->ATTACHMENT-CreatedByUpload = 'X'.

          endloop.

        endif.


        data(LSTT_TEMPLS) = READ_SERVICE_TASK_TEMPLATES( AD_SERVICE->SERVICE->SERVICETYPE ).
        loop at LSTT_TEMPLS assigning field-symbol(<LS_SERVICE_TASK_TEMPLATE>)
          where TaskType <> 'ADDITIONAL_SERVICE'.
          append initial line to <LS_SERVICE_TO_CREATE>-T_SERVICE_TASK assigning field-symbol(<SERVICE_TASK_TO_CREATE>).
          create data FR_SERVICE_TASK.
          FR_SERVICE_TASK->SERVICETASK = ref #( <SERVICE_TASK_TO_CREATE>-SERVICETASK ).
          FR_SERVICE_TASK->EXTUNIVERSAL = ref #( <SERVICE_TASK_TO_CREATE>-EXTUNIVERSAL ).
          FR_SERVICE_TASK->EXTWASTE = ref #( <SERVICE_TASK_TO_CREATE>-EXTWASTE ).
          FILL_SERVICE_TASK_DATA( <LS_SERVICE_TASK_TEMPLATE> ).
        endloop.
        if LINES( FR_WASTE_ORDER_ITEM->ADDITIONAL_SERVICES ) is not initial.
          LSEQUENCE = <SERVICE_TASK_TO_CREATE>-SERVICETASK-SequenceNumber.

          loop at FR_WASTE_ORDER_ITEM->ADDITIONAL_SERVICES reference into data(LADDSERVICEREF).
            loop at LSTT_TEMPLS assigning <LS_SERVICE_TASK_TEMPLATE>
              where TaskType = 'ADDITIONAL_SERVICE'.

              append initial line to <LS_SERVICE_TO_CREATE>-T_SERVICE_TASK assigning <SERVICE_TASK_TO_CREATE>.
              create data FR_SERVICE_TASK.
              FR_SERVICE_TASK->SERVICETASK = ref #( <SERVICE_TASK_TO_CREATE>-SERVICETASK ).
              FR_SERVICE_TASK->EXTUNIVERSAL = ref #( <SERVICE_TASK_TO_CREATE>-EXTUNIVERSAL ).
              FR_SERVICE_TASK->EXTWASTE = ref #( <SERVICE_TASK_TO_CREATE>-EXTWASTE ).

              FO_ACCESS_POOL->SET_CONTEXT(
                IV_SEMANTIC_CONTEXT = 'EWA_WASTE_ADDITIONAL_SERVICE' "/PLCE/CL_SEMANTIC_CONSTANTS=>C_SEMANTIC_CONTEXT-???
                IR_DATA             = LADDSERVICEREF
              ).
              FILL_SERVICE_TASK_DATA( <LS_SERVICE_TASK_TEMPLATE> ).
              LSEQUENCE = LSEQUENCE + 10.
              <SERVICE_TASK_TO_CREATE>-SERVICETASK-SequenceNumber = LSEQUENCE.

*                  FO_MAPPER->EXECUTE_SEMANTIC_TEMPL_CONFIG(
*                    IV_PROFILE                = GET_PROFILE( )
*                    IV_SEMANTIC_TEMPLATE_NAME = <LS_SERVICE_TASK_TEMPLATE>-SemanticTemplateName
*                  ).

            endloop.
          endloop.
        endif.
      endloop.
*          endif.
    endloop.

*      catch /PLCE/CX_BASEEXCEPTION into data(LBASEEX).
*        FO_MSGLOG->/PLCE/IF_PROTOCOL~REPORT_EXCEPTION(
*          exporting
**           IV_SEVERITY  =
*            IV_EXCEPTION = LBASEEX
*        ).
*    endtry.

  endmethod.


  method FILL_SERVICE_DATA.
    field-symbols:
      <FIELD_FROM> type DATA,
      <FIELD_TO>   type DATA.

*    AD_SERVICE->* = corresponding #( IS_SERVICE_TEMPLATE mapping SequenceNumber = SqncNr ).
    AD_SERVICE->SERVICE->* = corresponding #( IS_SERVICE_TEMPLATE ).

    if FR_WASTE_ORDER_ITEM is not initial.
*      AD_SERVICE->SERVICE->ReferenceId         = |{ FR_WASTE_ORDER_ITEM->ORDERNR alpha = out }/{ FR_WASTE_ORDER_ITEM->ORDER_LAUFNR }|. condense AD_SERVICE->SERVICE->ReferenceId.
      AD_SERVICE->SERVICE->ReferenceId         = |{ FR_WASTE_ORDER_ITEM->WDOITEMID alpha = out }|. condense AD_SERVICE->SERVICE->ReferenceId.
      AD_SERVICE->SERVICE->ReferenceInternalId = FR_WASTE_ORDER_ITEM->POBJNR.
*      AD_SERVICE->SERVICE->ExternalOrderType  = AD_SERVICE_ORDER_ITEM->ServiceOrderItemCategory. "ServiceDocumentItemObjectType
      AD_SERVICE->SERVICE->Profile = GET_PROFILE( ).

      FO_ACCESS_POOL->SET_CONTEXT(
        IV_SEMANTIC_CONTEXT = /PLCE/CL_SEMANTIC_CONSTANTS=>C_SEMANTIC_CONTEXT-SERVICE
        IR_DATA             = AD_SERVICE
      ).
      FO_ACCESS_POOL->SET_CONTEXT(
        IV_SEMANTIC_CONTEXT = 'EWA_WASTE_ORDER_ITEM' "/PLCE/CL_SEMANTIC_CONSTANTS=>C_SEMANTIC_CONTEXT-??
        IR_DATA             = FR_WASTE_ORDER_ITEM
      ).
      FO_MAPPER->EXECUTE_SEMANTIC_TEMPL_CONFIG(
        IV_PROFILE                = GET_PROFILE( )
        IV_SEMANTIC_TEMPLATE_NAME = IS_SERVICE_TEMPLATE-SemanticTemplateName
      ).
    endif.
  endmethod.


  method FILL_SERVICE_TASK_DATA.

    FR_SERVICE_TASK->SERVICETASK->* = corresponding #( IS_SRVC_TSK_TMPL mapping SequenceNumber = SqncNr ).
    FR_SERVICE_TASK->SERVICETASK->ServiceUUID = AD_SERVICE->SERVICE->ServiceUUID.

    FO_ACCESS_POOL->SET_CONTEXT(
      IV_SEMANTIC_CONTEXT = /PLCE/CL_SEMANTIC_CONSTANTS=>C_SEMANTIC_CONTEXT-SERVICE
      IR_DATA             = AD_SERVICE
    ).
    FO_ACCESS_POOL->SET_CONTEXT(
      IV_SEMANTIC_CONTEXT = 'EWA_WASTE_ORDER_ITEM' "/PLCE/CL_SEMANTIC_CONSTANTS=>C_SEMANTIC_CONTEXT-???
      IR_DATA             = FR_WASTE_ORDER_ITEM
    ).
    FO_ACCESS_POOL->SET_CONTEXT(
      IV_SEMANTIC_CONTEXT = /PLCE/CL_SEMANTIC_CONSTANTS=>C_SEMANTIC_CONTEXT-SERVICE_TASK
      IR_DATA             = FR_SERVICE_TASK
    ).
    FO_MAPPER->EXECUTE_SEMANTIC_TEMPL_CONFIG(
      IV_PROFILE                = GET_PROFILE( )
      IV_SEMANTIC_TEMPLATE_NAME = IS_SRVC_TSK_TMPL-SemanticTemplateName
    ).
  endmethod.


  method PROCESS_UPDATE.
    types:
      begin of _TATTAKEY,
        ServiceUUID    type /PLCE/PDSERVICE_UUID,
        AttachmentUUID type /PLCE/PDATTACHMENT_UUID,
      end of _TATTAKEY.

    data:
      LSRVKEYS    type /PLCE/PPDSRV_KEY,
      LSRVTSKKEYS type /PLCE/PPDSRVTSK_KEY.


    data:
      LRS_UPD       type table for update /PLCE/R_PDSERVICE,
      LRSUNI_UPD    type table for update /PLCE/R_PDSERVICEEXTUNI,
      LRSWA_UPD     type table for update /PLCE/R_PDSERVICEEXTWR,
      LRSCT_UPD     type table for update /PLCE/R_PDServiceExtCustom,
      LRSTSK_UPD    type table for update /PLCE/R_PDSERVICETASK,
      LRSTSKUNI_UPD type table for update /PLCE/R_PDSERVICETASKEXTUNI,
      LRSTSKWA_UPD  type table for update /PLCE/R_PDSERVICETASKEXTWR,
      LRS_ATTA_CREA type table for create /PLCE/R_PDSERVICE\_Attachments.
*      LRS_ATTA_CREA type table for create /PLCE/R_PDServiceAttachment.

    data:
      LSRVKEYS_DEL        type /PLCE/PPDSRV_KEY,
      LSRVKEYS_UNI_DEL    type /PLCE/PPDSRV_KEY,
      LSRVKEYS_WA_DEL     type /PLCE/PPDSRV_KEY,
      LSRVKEYS_CT_DEL     type /PLCE/PPDSRV_KEY,
      LSRVTSKKEYS_DEL     type /PLCE/PPDSRVTSK_KEY,
      LSRVTSKKEYS_UNI_DEL type /PLCE/PPDSRVTSK_KEY,
      LSRVTSKKEYS_WA_DEL  type /PLCE/PPDSRVTSK_KEY,
      LSRATTAKEYS_DEL     type standard table of _TATTAKEY.

    types:
      begin of _TPOBJNR,
        POBJNR type C length 30,
      end of _TPOBJNR.

    data:
      LPOBJNRS type standard table of _TPOBJNR.
****  FILL_KEYS
    if LINES( FT_WASTE_ORDER_ITEM_UPD ) is not initial.
      LPOBJNRS = corresponding #( FT_WASTE_ORDER_ITEM_UPD ).

      if LINES( LPOBJNRS ) is not initial.
        select SERVICEUUID from /PLCE/R_PDSERVICE
          for all entries in @LPOBJNRS
          where REFERENCEINTERNALID = @LPOBJNRS-POBJNR
          into table @LSRVKEYS.
      endif.
      if LINES( LSRVKEYS ) is not initial.

        select SERVICETASKUUID from /PLCE/R_PDSERVICETASK
          for all entries in @LSRVKEYS
          where SERVICEUUID = @LSRVKEYS-SERVICE_UUID
          into table @LSRVTSKKEYS.

      endif.

**** READ_ENTITIES

      read entities of /PLCE/R_PDSERVICE
        entity SERVICE
          all fields
          with value #( for LLINE in LSRVKEYS ( %KEY-SERVICEUUID = LLINE-SERVICE_UUID ) )
*    with corresponding #( LKEYS mapping from entity )
          result data(LRS)
*      result ( LRS )
          by \_EXTUNIVERSAL all fields with value #( for LLINE in LSRVKEYS ( %KEY-SERVICEUUID = LLINE-SERVICE_UUID ) )
          result data(LRSUNI)
          by \_EXTWASTE all fields with value #( for LLINE in LSRVKEYS ( %KEY-SERVICEUUID = LLINE-SERVICE_UUID ) )
          result data(LRSWA)
          by \_ExtCustom all fields with value #( for LLINE in LSRVKEYS ( %KEY-SERVICEUUID = LLINE-SERVICE_UUID ) )
          result data(LRSCT)
          by \_SERVICETASK all fields with value #( for LLINE in LSRVKEYS ( %KEY-SERVICEUUID = LLINE-SERVICE_UUID ) )
          result data(LRSTSK)
          by \_ATTACHMENTS all fields with value #( for LLINE in LSRVKEYS ( %KEY-SERVICEUUID = LLINE-SERVICE_UUID ) )
          result data(LRSATTA).

      read entities of /PLCE/R_PDSERVICE
        entity SERVICETASK
          by \_EXTWASTE all fields with corresponding #( LRSTSK )  "value #( for L in LRSTSK ( %KEY-ServiceTaskUUID = L-%KEY-ServiceTaskUUID ) )
          result data(LRSTSKWA)
          by \_EXTUNIVERSAL all fields with corresponding #( LRSTSK )  "value #( for L in LRSTSK ( %KEY-ServiceTaskUUID = L-%KEY-ServiceTaskUUID ) )
          result data(LRSTSKUNI).


      LRS_UPD = corresponding #( LRS ).
      LRSUNI_UPD = corresponding #( LRSUNI ).
      LRSWA_UPD = corresponding #( LRSWA ).
      LRSCT_UPD = corresponding #( LRSWA ).
      LRSTSK_UPD = corresponding #( LRSTSK ).
      LRSTSKUNI_UPD = corresponding #( LRSTSKUNI ).
      LRSTSKWA_UPD = corresponding #( LRSTSKWA ).

**** READ_ENTITIES

**** MAP_DATA

      data:
        LTSRV     type standard table of /PLCE/TPDSRV,
        LSRV_CREA type /PLCE/TPDSRV,
        LSRV_UPD  type /PLCE/TPDSRV.

      LTSRV = corresponding #( LRS_UPD mapping from entity  ).

      loop at LRS_UPD reference into data(LRSUPDREF).

*    LRSUPDREF->

**** Eigentlich kann es nur einen geben!!!!
        loop at AT_SERVICE_TO_CREATE reference into data(LSERVICECREATEREF)
          where SERVICE-REFERENCEINTERNALID = LRSUPDREF->REFERENCEINTERNALID. "#EC CI_SORTSEQ

**** Base entity
          LSRV_CREA = corresponding #( LSERVICECREATEREF->SERVICE mapping from entity ).
          LSRV_UPD = corresponding #( LRSUPDREF->* mapping from entity ).

          clear:
            LSRV_UPD-SERVICE_UUID, LSRV_UPD-SERVICE_ID.

          if LSRV_UPD-DATA <> LSRV_CREA-DATA.
* match!
            LRSUPDREF->%DATA = corresponding #( base ( LRSUPDREF->%DATA ) LSERVICECREATEREF->SERVICE except %KEY SERVICEID ).
            /PLCE/CL_BASE_MISC=>SET_MODIFY_CONTROL_FIELDS( IR_STRUCT = LRSUPDREF IV_SET_ALL_FIELDS = ABAP_TRUE ).

          endif.

*** EXTUNI
          read table LRSUNI_UPD reference into data(LRSUNI_UPDREF)
            with table key ENTITY components %KEY-SERVICEUUID = LRSUPDREF->%KEY-SERVICEUUID.

          if SY-SUBRC is initial.
            LRSUNI_UPDREF->%DATA = corresponding #( base ( LRSUNI_UPDREF->%DATA ) LSERVICECREATEREF->EXTUNIVERSAL except %KEY ).
            /PLCE/CL_BASE_MISC=>SET_MODIFY_CONTROL_FIELDS( IR_STRUCT = LRSUNI_UPDREF IV_SET_ALL_FIELDS = ABAP_TRUE ).
          else.
*** new relation
*          insert LSERVICECREATEREF->EXTUNIVERSAL into table ...
          endif.

*** EXTWA
          read table LRSWA_UPD reference into data(LRSWA_UPDREF)
            with table key ENTITY components %KEY-SERVICEUUID = LRSUPDREF->%KEY-SERVICEUUID.

          if SY-SUBRC is initial.
            LRSWA_UPDREF->%DATA = corresponding #( base ( LRSWA_UPDREF->%DATA ) LSERVICECREATEREF->EXTWASTE except %KEY ).
            /PLCE/CL_BASE_MISC=>SET_MODIFY_CONTROL_FIELDS( IR_STRUCT =  LRSWA_UPDREF IV_SET_ALL_FIELDS = ABAP_TRUE ).
*            LRSWA_UPDREF->* = corresponding #( base ( LRSWA_UPDREF->* ) LSERVICECREATEREF->EXTWASTE changing control except SERVICEUUID %KEY ).
          else.
*** new relation
*          insert LSERVICECREATEREF->EXTWASTE into table ...
          endif.

          read table LRSCT_UPD reference into data(LRSCT_UPDREF)
            with table key ENTITY components %KEY-SERVICEUUID = LRSUPDREF->%KEY-SERVICEUUID.

          if SY-SUBRC is initial.
            LRSCT_UPDREF->%DATA = corresponding #( base ( LRSCT_UPDREF->%DATA ) LSERVICECREATEREF->EXTCUSTOM except %KEY ).
            /PLCE/CL_BASE_MISC=>SET_MODIFY_CONTROL_FIELDS( IR_STRUCT =  LRSCT_UPDREF IV_SET_ALL_FIELDS = ABAP_TRUE ).
*            LRSWA_UPDREF->* = corresponding #( base ( LRSWA_UPDREF->* ) LSERVICECREATEREF->EXTWASTE changing control except SERVICEUUID %KEY ).
          else.
*** new relation
*          insert LSERVICECREATEREF->EXTWASTE into table ...
          endif.

**** Attachments <- always
          if LINES( LSERVICECREATEREF->ATTACHMENTS ) is not initial.
            insert initial line into table LRS_ATTA_CREA assigning field-symbol(<ATTA_CREA>).
            loop at LSERVICECREATEREF->ATTACHMENTS reference into data(LATTAREF).
              insert initial line into table <ATTA_CREA>-%TARGET assigning field-symbol(<ATTA_CREA_LINE>).
              <ATTA_CREA_LINE> = corresponding #( LATTAREF->ATTACHMENT ).
              <ATTA_CREA_LINE>-ServiceUUID = LRSUPDREF->ServiceUUID.
              <ATTA_CREA_LINE>-%CID = GET_NEXT_CID( ).

              /PLCE/CL_BASE_MISC=>SET_MODIFY_CONTROL_FIELDS( IR_STRUCT = ref #( <ATTA_CREA_LINE> ) ).
              <ATTA_CREA_LINE>-%CONTROL-ServiceUUID = IF_ABAP_BEHV=>MK-OFF.
            endloop.
            <ATTA_CREA>-ServiceUUID = LRSUPDREF->ServiceUUID.
*            <ATTA_CREA>-%CID_REF = LRSUPDREF->ServiceUUID.
          endif.

***** Tasks

**** upd & delete

***  update/insert
          loop at LSERVICECREATEREF->T_SERVICE_TASK reference into data(LSERVICETASKCREATEREF).

            read table LRSTSK_UPD reference into data(LRSTSK_UPDREF)
              with key
                       SERVICEUUID = LRSUPDREF->SERVICEUUID  " todo --- Fehler nachstellen
                       TASKTYPE = LSERVICETASKCREATEREF->SERVICETASK-TASKTYPE
                       SEQUENCENUMBER = LSERVICETASKCREATEREF->SERVICETASK-SEQUENCENUMBER.

            if SY-SUBRC is initial.

              LRSTSK_UPDREF->%DATA = corresponding #( base ( LRSTSK_UPDREF->%DATA ) LSERVICETASKCREATEREF->SERVICETASK
                except %KEY SERVICEUUID PLANNINGSTATUS TOURUUID ).
              /PLCE/CL_BASE_MISC=>SET_MODIFY_CONTROL_FIELDS( IR_STRUCT = LRSTSK_UPDREF IV_SET_ALL_FIELDS = ABAP_TRUE ).
              LRSTSK_UPDREF->%CONTROL-ServiceUUID = IF_ABAP_BEHV=>MK-OFF.
              LRSTSK_UPDREF->%CONTROL-SequenceNumber = IF_ABAP_BEHV=>MK-OFF.

              read table LRSTSKUNI_UPD reference into data(LRSTSKUNI_UPDREF)
                with table key ENTITY components SERVICETASKUUID = LRSTSK_UPDREF->SERVICETASKUUID.

              if SY-SUBRC is initial.
                LRSTSKUNI_UPDREF->%DATA = corresponding #( base ( LRSTSKUNI_UPDREF->%DATA ) LSERVICETASKCREATEREF->EXTUNIVERSAL except %KEY SERVICEUUID ).
                /PLCE/CL_BASE_MISC=>SET_MODIFY_CONTROL_FIELDS( IR_STRUCT = LRSTSKUNI_UPDREF IV_SET_ALL_FIELDS = ABAP_TRUE ).
                LRSTSKUNI_UPDREF->%CONTROL-ServiceUUID = IF_ABAP_BEHV=>MK-OFF.
              else.
***           insert new UNI Task
              endif.

              read table LRSTSKWA_UPD reference into data(LRSTSKWA_UPDREF)
                with table key ENTITY components SERVICETASKUUID = LRSTSK_UPDREF->SERVICETASKUUID.

              if SY-SUBRC is initial.
                LRSTSKWA_UPDREF->%DATA = corresponding #( base ( LRSTSKWA_UPDREF->%DATA ) LSERVICETASKCREATEREF->EXTWASTE except %KEY SERVICEUUID ).
                /PLCE/CL_BASE_MISC=>SET_MODIFY_CONTROL_FIELDS( IR_STRUCT = LRSTSKWA_UPDREF IV_SET_ALL_FIELDS = ABAP_TRUE ).
                LRSTSKWA_UPDREF->%CONTROL-ServiceUUID = IF_ABAP_BEHV=>MK-OFF.
              else.
***           insert new WA Task
              endif.


            else.
***       alle Task Teile müssen angelegt werden.

            endif.

          endloop.


****  Delete

          if LSERVICETASKCREATEREF->EXTUNIVERSAL is initial.
            read table LRSUNI_UPD reference into LRSUNI_UPDREF
              with table key ENTITY components %KEY = corresponding #( LRSUPDREF->%KEY ).

            if SY-SUBRC is initial.
              if LSERVICECREATEREF->EXTUNIVERSAL is initial.
                insert corresponding #( LRSUNI_UPDREF->%KEY ) into table LSRVKEYS_UNI_DEL.
              endif.
            endif.
          endif.

          if LSERVICETASKCREATEREF->EXTWASTE is initial.
            read table LRSWA_UPD reference into LRSWA_UPDREF
              with table key ENTITY components %KEY = corresponding #( LRSUPDREF->%KEY ).

            if SY-SUBRC is initial.
              if LSERVICECREATEREF->EXTWASTE is initial.
                insert corresponding #( LRSWA_UPDREF->%KEY ) into table LSRVKEYS_WA_DEL.
              endif.
            endif.
          endif.


          loop at LRSTSK_UPD reference into LRSTSK_UPDREF where SERVICEUUID = LRSUPDREF->SERVICEUUID.
            read table LSERVICECREATEREF->T_SERVICE_TASK  reference into LSERVICETASKCREATEREF "transporting no fields
              with key "SERVICETASK-SERVICEUUID = LRSTSK_UPDREF->SERVICEUUID
                       SERVICETASK-TASKTYPE = LRSTSK_UPDREF->TASKTYPE
                       SERVICETASK-SEQUENCENUMBER = LRSTSK_UPDREF->SEQUENCENUMBER.

            if SY-SUBRC is not initial.
              insert corresponding #( LRSTSK_UPDREF->%KEY ) into table LSRVTSKKEYS_DEL.
            else.

              read table LRSTSKUNI_UPD reference into LRSTSKUNI_UPDREF
                with table key ENTITY components %KEY = corresponding #( LRSTSK_UPDREF->%KEY ).

              if SY-SUBRC is initial.
                if LSERVICETASKCREATEREF->EXTUNIVERSAL is initial.
                  insert corresponding #( LRSTSK_UPDREF->%KEY ) into table LSRVTSKKEYS_UNI_DEL.
                endif.
              endif.

              read table LRSTSKWA_UPD reference into LRSTSKWA_UPDREF
                  with table key ENTITY components %KEY = corresponding #( LRSTSK_UPDREF->%KEY ).

              if SY-SUBRC is initial.
                if LSERVICETASKCREATEREF->EXTWASTE is initial.
                  insert corresponding #( LRSTSK_UPDREF->%KEY ) into table LSRVTSKKEYS_WA_DEL.
                endif.
              endif.

            endif.

          endloop.

        endloop.
      endloop.

*** Attachments
      LSRATTAKEYS_DEL = value #( for LATTALINE in LRSATTA where ( CreatedByUpload is not initial )
        ( corresponding #( LATTALINE ) ) ).

**** SAVE
      delete:
        LRS_UPD where %CONTROL is initial,
        LRSUNI_UPD where %CONTROL is initial,
        LRSWA_UPD where %CONTROL is initial,
        LRSCT_UPD where %CONTROL is initial,
        LRSTSK_UPD where %CONTROL is initial,
        LRSTSKWA_UPD where %CONTROL is initial,
        LRSTSKUNI_UPD where %CONTROL is initial.


      modify entities of /PLCE/R_PDSERVICE
        entity SERVICE
          update from LRS_UPD
*          update set fields with corresponding #( LRS_UPD ) "set fields with LRS_UPD  ##SETFIELDS_OK
          delete from corresponding #( LSRVKEYS_DEL )
          create by \_Attachments from corresponding #( LRS_ATTA_CREA )
        entity EXTUNIVERSAL
          update from LRSUNI_UPD
*          update set fields with corresponding #( LRSUNI_UPD ) ##SETFIELDS_OK
          delete from corresponding #( LSRVKEYS_UNI_DEL )
        entity EXTWASTE
*          update set fields with corresponding #( LRSWA_UPD ) ##SETFIELDS_OK
          update from LRSWA_UPD
          delete from corresponding #( LSRVKEYS_WA_DEL )
        entity ExtCustom
*          update set fields with corresponding #( LRSWA_UPD ) ##SETFIELDS_OK
          update from LRSCT_UPD
          delete from corresponding #( LSRVKEYS_CT_DEL )
        entity Attachments
          delete from corresponding #( LSRATTAKEYS_DEL )
        entity SERVICETASK
          update from LRSTSK_UPD
*          update set fields with corresponding #( LRSTSK_UPD except SERVICEUUID SEQUENCENUMBER ) ##SETFIELDS_OK
          delete from corresponding #( LSRVTSKKEYS_DEL )
        entity EXTTASKUNIVERSAL
          update from LRSTSKUNI_UPD
*          update set fields with corresponding #( LRSTSKUNI_UPD except SERVICEUUID ) ##SETFIELDS_OK
          delete from corresponding #( LSRVTSKKEYS_UNI_DEL )
        entity EXTTASKWASTE
          update from LRSTSKWA_UPD
*          update set fields with corresponding #( LRSTSKWA_UPD except SERVICEUUID  ) ##SETFIELDS_OK
          delete from corresponding #( LSRVTSKKEYS_WA_DEL )
        mapped data(MAPPED_UPDATE)
        failed data(FAILED_UPDATE)
        reported data(REPORTED_UPDATE).

      if FAILED_UPDATE is initial.
*        modify entities of /PLCE/R_PDSERVICE
*          entity SERVICE
**            update from LRS_UPD
**  *          update set fields with corresponding #( LRS_UPD ) "set fields with LRS_UPD  ##SETFIELDS_OK
**            delete from corresponding #( LSRVKEYS_DEL )
*            create by \_Attachments from corresponding #( LRS_ATTA_CREA )
*          mapped data(MAPPED_UPDATE_ATTA)
*          failed data(FAILED_UPDATE_ATTA)
*          reported data(REPORTED_UPDATE_ATTA).

*        if FAILED_UPDATE_ATTA is initial.
          commit entities
            response of /PLCE/R_PDSERVICE
              failed data(FAILED_COMMIT_SRVC)
              reported data(REPORTED_COMMIT_SRVC).
*        else.
*          rollback entities.                          "#EC CI_ROLLBACK.
*        endif.
      else.
        rollback entities.                            "#EC CI_ROLLBACK.
      endif.

      /PLCE/CL_BASE_MISC=>CHECK_RESPONSE(
        exporting
          IS_RESPONSE = REPORTED_UPDATE
          IR_LOG      = FO_MSGLOG                 " APPLOG
      ).

    endif.

  endmethod.


  method PROCESS_WASTEORDERS.
    clear:
      AT_SERVICE_TO_CREATE.

    super->PROCESS_WASTEORDERS(
      exporting
        IV_PROFILE     = IV_PROFILE
        IT_WASTEORDER  = IT_WASTEORDER
        IO_MSGLOG      = IO_MSGLOG
      importing
        ET_SRV_MAPPING = ET_SRV_MAPPING
        ET_SRV_KEY     = ET_SRV_KEY
    ).
  endmethod.


  method SAVE_SERVICE.
    check AT_SERVICE_TO_CREATE is not initial.

    data: LT_SERVICE_TO_CREATE         type table for create /PLCE/R_PDSERVICE,
          LT_SERVICE_TASK_TO_CREATE    type table for create /PLCE/R_PDSERVICE\_SERVICETASK,
          LT_SERVICE_UNIVERSAL         type table for create /PLCE/R_PDSERVICE\_EXTUNIVERSAL,
          LT_SERVICE_WASTE             type table for create /PLCE/R_PDSERVICE\_EXTWASTE,
          LT_SERVICE_CUSTOM            type table for create /PLCE/R_PDSERVICE\_EXTCUSTOM,
          LT_SERVICE_TASK_TO_CREATE_UN type table for create /PLCE/R_PDSERVICETASK\_EXTUNIVERSAL,
          LT_SERVICE_TASK_TO_CREATE_WA type table for create /PLCE/R_PDSERVICETASK\_EXTWASTE,
          LT_SERVICE_NOTES_TO_CREATE   type table for create /PLCE/R_PDSERVICE\_Notes,
          LT_SERVICE_ATTAS_TO_CREATE   type table for create /PLCE/R_PDSERVICE\_Attachments,
          LV_SERVICETASKCID            type ABP_BEHV_CID,
          LINDEX                       type N length 8.

    loop at AT_SERVICE_TO_CREATE reference into data(LR_SERVICE_TO_CREATE).
      insert value #( %CID = LR_SERVICE_TO_CREATE->CID %DATA = LR_SERVICE_TO_CREATE->SERVICE ) into table LT_SERVICE_TO_CREATE.

      if LR_SERVICE_TO_CREATE->EXTUNIVERSAL is not initial.
        insert value #( %CID_REF = LR_SERVICE_TO_CREATE->CID ) into table LT_SERVICE_UNIVERSAL assigning field-symbol(<LS_SERVICE_UNIVERSAL>).
        insert value #( %CID = |{ LR_SERVICE_TO_CREATE->CID }_EXTUN| %DATA = LR_SERVICE_TO_CREATE->EXTUNIVERSAL ) into table <LS_SERVICE_UNIVERSAL>-%TARGET.
      endif.
      if LR_SERVICE_TO_CREATE->EXTWASTE is not initial.
        insert value #( %CID_REF = LR_SERVICE_TO_CREATE->CID ) into table LT_SERVICE_WASTE assigning field-symbol(<LS_SERVICE_WASTE>).
        insert value #( %CID = |{ LR_SERVICE_TO_CREATE->CID }_EXTWA| %DATA = LR_SERVICE_TO_CREATE->EXTWASTE ) into table <LS_SERVICE_WASTE>-%TARGET.
      endif.

      if LR_SERVICE_TO_CREATE->EXTCUSTOM is not initial.
        insert value #( %CID_REF = LR_SERVICE_TO_CREATE->CID ) into table LT_SERVICE_CUSTOM assigning field-symbol(<LS_SERVICE_CUSTOM>).
        insert value #( %CID = |{ LR_SERVICE_TO_CREATE->CID }_EXTCUSTOM| %DATA = LR_SERVICE_TO_CREATE->EXTCUSTOM  ) into table <LS_SERVICE_CUSTOM>-%TARGET.
      endif.

*      insert value #( %CID = LR_SERVICE_TO_CREATE->CID %DATA = LR_SERVICE_TO_CREATE->SERVICE ) into table LT_SERVICE_TO_CREATE.

      if LINES( LR_SERVICE_TO_CREATE->T_SERVICE_TASK ) > 0.
        insert value #( %CID_REF = LR_SERVICE_TO_CREATE->CID ) into table LT_SERVICE_TASK_TO_CREATE assigning field-symbol(<LS_SERVICE_TASK_TO_CREATE>).
        loop at LR_SERVICE_TO_CREATE->T_SERVICE_TASK reference into data(LR_SERVICE_TASK).
          concatenate LR_SERVICE_TO_CREATE->CID '_' LR_SERVICE_TASK->SERVICETASK-SEQUENCENUMBER into LV_SERVICETASKCID.
          insert value #( %CID = LV_SERVICETASKCID %DATA = LR_SERVICE_TASK->SERVICETASK ) into table <LS_SERVICE_TASK_TO_CREATE>-%TARGET.

          if LR_SERVICE_TASK->EXTUNIVERSAL is not initial.
            insert value #( %CID_REF = LV_SERVICETASKCID ) into table LT_SERVICE_TASK_TO_CREATE_UN assigning field-symbol(<LS_SERVICETASK_UNIVERSAL>).
            insert value #( %CID = |{ LV_SERVICETASKCID }_EXTUN| %DATA = LR_SERVICE_TASK->EXTUNIVERSAL ) into table <LS_SERVICETASK_UNIVERSAL>-%TARGET.
          endif.
          if LR_SERVICE_TO_CREATE->EXTWASTE is not initial.
            insert value #( %CID_REF = LV_SERVICETASKCID ) into table LT_SERVICE_TASK_TO_CREATE_WA assigning field-symbol(<LS_SERVICETASK_WASTE>).
            insert value #( %CID = |{ LV_SERVICETASKCID }_EXTWA| %DATA = LR_SERVICE_TASK->EXTWASTE ) into table <LS_SERVICETASK_WASTE>-%TARGET.
          endif.

        endloop.
      endif.
      if LINES( LR_SERVICE_TO_CREATE->T_SERVICE_NOTES ) > 0.
        insert value #( %CID_REF = LR_SERVICE_TO_CREATE->CID ) into table LT_SERVICE_NOTES_TO_CREATE assigning field-symbol(<LS_SERVICE_NOTES_TO_CREATE>).
        loop at LR_SERVICE_TO_CREATE->T_SERVICE_NOTES reference into data(LR_SERVICE_NOTE).
          LINDEX = LINDEX + 1.
          concatenate LR_SERVICE_TO_CREATE->CID '_NOTE' LINDEX into LV_SERVICETASKCID.
          insert value #( %CID = LV_SERVICETASKCID %DATA = LR_SERVICE_NOTE->SERVICE_NOTE ) into table <LS_SERVICE_NOTES_TO_CREATE>-%TARGET.
        endloop.
      endif.
      if LINES( LR_SERVICE_TO_CREATE->ATTACHMENTS ) > 0.
        insert value #( %CID_REF = LR_SERVICE_TO_CREATE->CID ) into table LT_SERVICE_ATTAS_TO_CREATE assigning field-symbol(<LS_SERVICE_ATTAS_TO_CREATE>).
        loop at LR_SERVICE_TO_CREATE->ATTACHMENTS reference into data(LR_SERVICE_ATTAREF).
          LINDEX = LINDEX + 1.
          concatenate LR_SERVICE_TO_CREATE->CID '_ATTA' LINDEX into LV_SERVICETASKCID.
          insert value #( %CID = LV_SERVICETASKCID %DATA = LR_SERVICE_ATTAREF->ATTACHMENT ) into table <LS_SERVICE_ATTAS_TO_CREATE>-%TARGET.
        endloop.
      endif.
    endloop.

    modify entities of /PLCE/R_PDSERVICE
      entity SERVICE
        create set fields with LT_SERVICE_TO_CREATE  ##SETFIELDS_OK
        create by \_SERVICETASK set fields with LT_SERVICE_TASK_TO_CREATE
*        create by \_ServiceTask ExtUniversal set fields with LT_SERVICE_TASK_TO_CREATE
        create by \_EXTUNIVERSAL set fields with LT_SERVICE_UNIVERSAL
        create by \_EXTWASTE set fields with LT_SERVICE_WASTE
        create by \_ExtCustom set fields with LT_SERVICE_CUSTOM
        create by \_Notes set fields with LT_SERVICE_NOTES_TO_CREATE
        create by \_Attachments set fields with LT_SERVICE_ATTAS_TO_CREATE
      entity SERVICETASK
        create by \_EXTUNIVERSAL set fields with LT_SERVICE_TASK_TO_CREATE_UN
        create by \_EXTWASTE set fields with LT_SERVICE_TASK_TO_CREATE_WA
      mapped data(MAPPED_CREATE)
      failed data(FAILED_CREATE)
      reported data(REPORTED_CREATE).


    loop at FAILED_CREATE-SERVICE assigning field-symbol(<FAILED_CREATE_SRVC>).
      FO_MSGLOG->/PLCE/IF_PROTOCOL~REPORT_MESSAGE(
        exporting
          IV_SEVERITY   = FO_MSGLOG->/PLCE/IF_PROTOCOL~C_SEVERITY-ERROR
          IV_ID         = '/PLCE/MC_PD_MSG'
          IV_NUMBER     = '104'
          IV_VARIABLE_1 = conv #( text-005 )
          IV_VARIABLE_2 = conv #( <FAILED_CREATE_SRVC>-SERVICEUUID )
      ).
    endloop.

    commit entities
      response of /PLCE/R_PDSERVICE
        failed data(FAILED_COMMIT_SRVC)
        reported data(REPORTED_COMMIT_SRVC).

    if FAILED_COMMIT_SRVC is not initial or sy-subrc is not initial.
      loop at FAILED_COMMIT_SRVC-SERVICE assigning field-symbol(<FAILED_COMMIT_SRVC>).
        FO_MSGLOG->/PLCE/IF_PROTOCOL~REPORT_MESSAGE(
          exporting
            IV_SEVERITY   = FO_MSGLOG->/PLCE/IF_PROTOCOL~C_SEVERITY-ERROR
            IV_ID         = '/PLCE/MC_PD_MSG'
            IV_NUMBER     = '106'
            IV_VARIABLE_1 = conv #( text-005 )
            IV_VARIABLE_2 = conv #( <FAILED_COMMIT_SRVC>-SERVICEUUID )
        ).
      endloop.
      rollback entities.                               "#EC CI_ROLLBACK
*      clear ET_SERVICE_KEY.
    else.
      if ET_SERVICE_KEY is requested or ET_SRV_MAPPING is requested.

        ET_SERVICE_KEY = corresponding #( MAPPED_CREATE-SERVICE ).
        loop at MAPPED_CREATE-SERVICE assigning field-symbol(<MAPPED_CREATE_SRVC>).
          ET_SERVICE_KEY = value #( base ET_SERVICE_KEY ( SERVICE_UUID = <MAPPED_CREATE_SRVC>-%KEY-SERVICEUUID ) ).
        endloop.

      endif.

      if ET_SRV_MAPPING is requested.

        read entities of /PLCE/R_PDSERVICE
          entity SERVICE
          fields ( SERVICEUUID REFERENCEINTERNALID )
*          with value #( for LLINE in MAPPED_CREATE-SERVICE ( %KEY-SERVICEUUID = LLINE-%KEY-SERVICEUUID ) )
          with corresponding #( MAPPED_CREATE-SERVICE )
          result data(LRS).

        ET_SRV_MAPPING = corresponding #( LRS mapping POBJNR = ReferenceInternalId SERVICE_UUID = ServiceUUID ).

      endif.
    endif.
  endmethod.


method ZZ_CHANGE_SERVICE_TYPE.
  data:
    LDATA type ZZ_TY_DATA.

  try.
      LDATA = ZZ_FILL_SERVICE_TASK(
        exporting
          PAR_KEY    = PAR_KEY
          PAR_SVTPTO = PAR_SVTPTO
          PAR_LOG    = PAR_LOG ).

* -------------------------------------------------------------------
      modify entities of /PLCE/R_PDService
        entity SERVICE
          update from corresponding #( LDATA-SRV_UPD )
          create by \_SERVICETASK set fields with LDATA-TSKS_CREA
        entity SERVICETASK
          delete from corresponding #( LDATA-TSKS_DELE )
          create by \_EXTWASTE set fields with LDATA-TSKS_CREA_WA
        entity EXTTASKUNIVERSAL
          delete from corresponding #( LDATA-TSKS_DELE_UN )
        entity EXTTASKWASTE
          delete from corresponding #( LDATA-TSKS_DELE_WA )
        mapped   data(LT_SRV_MAP)
        reported data(LT_SRV_REP)
        failed   data(LT_SRV_FAI) ##SETFIELDS_OK.

      if LT_SRV_FAI is initial.
        if PAR_SIMULATE is initial.
          commit entities
            response of /PLCE/R_PDSERVICE
              failed   data(SRV_COMMIT_FAI)
              reported data(SRV_COMMIT_REP).
        endif.
      endif.

      /PLCE/CL_BASE_MISC=>CHECK_RESPONSE(
        exporting
          IS_RESPONSE = LT_SRV_REP
          IR_LOG      = PAR_LOG  ).               " APPLOG

    catch /PLCE/CX_BASEEXCEPTION into data(LEX_PD).
      PAR_LOG->ADD_EXCEPTION( IV_SEVERITY = 'W' IRO_EXCEPTION = LEX_PD ).
*    catch CX_SADL_SHORTDUMP into data(LEX_SADL).
*      data(LX_ROOT_CAUSE) = new CX_SADL_CONTRACT_VIOLATION( ).
*      CL_ABAP_UNIT_ASSERT=>ASSERT_EQUALS( ACT = LEX_SADL->PREVIOUS
*                                          EXP = LX_ROOT_CAUSE ).
  endtry.

endmethod.


method ZZ_FILL_SERVICE_TASK.
  data:
    LINDEX   type SYINDEX,
    LSRVKEYS type /PLCE/PPDSRV_KEY,
    LSEQNR   type /PLCE/SEQUENCE_NUMBER,
    LEX_PD   type ref to /PLCE/CX_BASEEXCEPTION,
    LPROC    type ref to ZCL_PD_PROC_WASTE_ORDERS.

  data:
    LDATA         type ZZ_TY_DATA,
    LACTION      type /PLCE/PDACTION,
    LWDO         type /PLCE/SWR_WASTEORDERITEM_IMP,
    LWDOS        type /PLCE/PWR_WASTEORDERITEM_IMP,
    LTSKS_CREA_X type /PLCE/CL_PD_PROC_WASTE_ORDERS=>TP_MODIFY_SERVICE_TASK_EXT ##NEEDED,
    LTSKS_WDOI   type standard table of /Plce/R_PDServicetask
                  with non-unique sorted key SEC_KEY components ServiceUUID SequenceNumber.

  try.
      LPROC = new #( ).
      LSRVKEYS = value #( ( PAR_KEY ) ).

* -------------------------------------------------------------------
      read entities of /PLCE/R_PDSERVICE
      entity SERVICE
        all fields "with corresponding #( LSRVKEYS mapping from entity )
          with value #( for LLINE in LSRVKEYS ( %KEY-SERVICEUUID = LLINE-SERVICE_UUID ) )
          result data(LSRV)
        by \_EXTUNIVERSAL all fields
          with value #( for LLINE in LSRVKEYS ( %KEY-SERVICEUUID = LLINE-SERVICE_UUID ) )
          result data(LSRV_UN)
        by \_EXTWASTE all fields
          with value #( for LLINE in LSRVKEYS ( %KEY-SERVICEUUID = LLINE-SERVICE_UUID ) )
          result data(LSRV_WA)
        by \_ExtCustom all fields
          with value #( for LLINE in LSRVKEYS ( %KEY-SERVICEUUID = LLINE-SERVICE_UUID ) )
          result data(LSRV_CT)
        by \_SERVICETASK all fields
          with value #( for LLINE in LSRVKEYS ( %KEY-SERVICEUUID = LLINE-SERVICE_UUID ) )
          result data(LRSTSK).

      read entities of /PLCE/R_PDSERVICE
        entity SERVICETASK
          by \_EXTWASTE all fields with corresponding #( LRSTSK )
          result data(LRSTSK_WA)
          by \_EXTUNIVERSAL all fields with corresponding #( LRSTSK )
          result data(LRSTSK_UN).

      PAR_RESULT-SRV_UPD    = corresponding #( LSRV ).
      PAR_RESULT-SRV_UPD_UN = corresponding #( LSRV_UN ).
      PAR_RESULT-SRV_UPD_WA = corresponding #( LSRV_WA ).
      PAR_RESULT-SRV_UPD_CUST = corresponding #( LSRV_CT ).

* -------------------------------------------------------------------
      create data LPROC->AD_SERVICE.
      create data LPROC->AD_SERVICE->SERVICE.
      if LINE_EXISTS( LSRV[ 1 ] ).
        LPROC->AD_SERVICE->SERVICE->* = corresponding #(  LSRV[ 1 ] ).
        LPROC->SET_PROFILE( LSRV[ 1 ]-PROFILE ).
        LWDO-POBJNR = LSRV[ 1 ]-REFERENCEINTERNALID. "POBJNR.
        LWDOS = value #( ( LWDO ) ).
        LPROC->SET_DATA( LWDOS ) .

        select single TMPL~ACTION from /PLCE/CPDSTMPL as TMPL "/PLCE/R_PDServiceTemplate
          inner join /PLCE/CPDACTION as ACT on TMPL~ACTION = ACT~ACTION "Z Action
          where PROFILE      eq @LPROC->FV_PROFILE
            and SERVICE_TYPE eq @PAR_SVTPTO
          into @LACTION.
        data(LST_TEMPLS)      = LPROC->READ_SERVICE_TEMPLATES( exporting IV_ACTION = LACTION ).
        data(LSTT_TEMPLS_OLD) = LPROC->READ_SERVICE_TASK_TEMPLATES( LPROC->AD_SERVICE->SERVICE->SERVICETYPE ).
        data(LSTT_TEMPLS_NEW) = LPROC->READ_SERVICE_TASK_TEMPLATES( PAR_SVTPTO ).
      endif.

      loop at PAR_RESULT-SRV_UPD assigning field-symbol(<S_UPD>) .
        <S_UPD>-ServiceType = PAR_SVTPTO.
        <S_UPD>-%CONTROL-ServiceType = IF_ABAP_BEHV=>MK-ON.
        <S_UPD>-Action = LACTION.
        <S_UPD>-%CONTROL-Action      = IF_ABAP_BEHV=>MK-ON.
      endloop.

* -------------------------------------------------------------------
      if 1 = 2.  "Renew service completly
        loop at LST_TEMPLS assigning field-symbol(<SERVICETMPL>)
          where SemanticTemplateName <> 'ST_SERVICE_NOTE'.
          LPROC->FILL_SERVICE_DATA(
            exporting
              IS_SERVICE_TEMPLATE = <SERVICETMPL> ).
        endloop.
      endif.

      loop at LRSTSK assigning field-symbol(<TASK>).
        if LINE_EXISTS( LSTT_TEMPLS_OLD[ SqncNr = <TASK>-SequenceNumber TaskType = <TASK>-TaskType ] ).
          PAR_RESULT-TSKS_DELE    = value #( base PAR_RESULT-TSKS_DELE ( corresponding #( <TASK> ) ) ).
          if LRSTSK_UN is not initial.
            PAR_RESULT-TSKS_DELE_UN = value #( base PAR_RESULT-TSKS_DELE_UN ( corresponding #( <TASK> ) ) ).
          endif.
          if LRSTSK_WA is not initial.
            PAR_RESULT-TSKS_DELE_WA = value #( base PAR_RESULT-TSKS_DELE_WA ( corresponding #( <TASK> ) ) ).
          endif.
        else.
          LTSKS_WDOI = value #( base LTSKS_WDOI ( corresponding #( <TASK> ) ) ).
        endif.
      endloop.

      select * from /PLCE/R_MDFuncLocMDCat
        into table @data(FUNCLOCCATS). "#EC CI_ALL_FIELDS_NEEDED "#EC CI_NOWHERE

      data(LWDOI_TASK_CNT) = LINES( LTSKS_WDOI ) .
      if LWDOI_TASK_CNT > 0.
        read table LTSKS_WDOI index 1 reference into data(TASK_WDOI_MIN).
        if LWDOI_TASK_CNT > 1.
          LINDEX = LWDOI_TASK_CNT.
          read table LTSKS_WDOI index LINDEX reference into data(TASK_WDOI_MAX).
        endif.
      endif.

      if 1 = 2. "Errors
        loop at LSTT_TEMPLS_NEW assigning field-symbol(<TASKTEMLATE_X>)
*        where TaskType <> 'ADDITIONAL_SERVICE'.
          where IsServiceTaskOptional is initial.

          append initial line to LTSKS_CREA_X assigning field-symbol(<TASK_CREA_X>).
          create data LPROC->FR_SERVICE_TASK.
          LPROC->FR_SERVICE_TASK->SERVICETASK  = ref #( <TASK_CREA_X>-SERVICETASK ).
          LPROC->FR_SERVICE_TASK->EXTUNIVERSAL = ref #( <TASK_CREA_X>-EXTUNIVERSAL ).
          LPROC->FR_SERVICE_TASK->EXTWASTE     = ref #( <TASK_CREA_X>-EXTWASTE ).
          LPROC->FILL_SERVICE_TASK_DATA( <TASKTEMLATE_X> ).
        endloop.
      endif.

      loop at LSTT_TEMPLS_NEW assigning field-symbol(<TASKTEMLATE>)
        where IsServiceTaskOptional is initial.

        append initial line to PAR_RESULT-TSKS_CREA assigning field-symbol(<TASK_CREA>).
        insert initial line into table <TASK_CREA>-%TARGET assigning field-symbol(<TARGET>).
        <TARGET> = corresponding #( <TASKTEMLATE> mapping SequenceNumber = SqncNr ).

        if LWDOI_TASK_CNT is not initial and
           <TARGET>-SequenceNumber >= TASK_WDOI_MIN->SequenceNumber and
           <TARGET>-SequenceNumber <= TASK_WDOI_MAX->SequenceNumber.

          if LSEQNR is initial.
            LSEQNR = TASK_WDOI_MAX->SequenceNumber + 10.
          else.
            LSEQNR += 10.
          endif.
          <TARGET>-SequenceNumber   = LSEQNR.
        endif.

        case <TARGET>-FUNCTIONALLOCATIONCATEGORY.
          when /PLCE/IF_MD_CONSTANTS=>C_FUNCLOC_MDCATEGORIES-CUSTOMERLOCATION.
*           "filled already
          when /PLCE/IF_MD_CONSTANTS=>C_FUNCLOC_MDCATEGORIES-MATERIALDEPOT or
               /PLCE/IF_MD_CONSTANTS=>C_FUNCLOC_MDCATEGORIES-TRANSPORTPACKAGINGDEPOT.
            data(FLC) = value #( FUNCLOCCATS[ FUNCLOCMDCATEGORY = <TARGET>-FUNCTIONALLOCATIONCATEGORY ] optional ).
            if SY-SUBRC is initial.
              <TARGET>-DURATION     = FLC-STAYDURATION.
              <TARGET>-DURATIONUNIT = FLC-STAYDURATIONUNIT.
            endif.
          when others.
        endcase.

        <TARGET>-%CID = LPROC->GET_NEXT_CID( ).
*        <TARGET>-ServiceUUID = LPROC->FR_SERVICE->SERVICE->ServiceUUID. "must be empty for create
        <TASK_CREA>-ServiceUUID = LPROC->AD_SERVICE->SERVICE->ServiceUUID.

        /PLCE/CL_BASE_MISC=>SET_MODIFY_CONTROL_FIELDS( IR_STRUCT = ref #( <TARGET> ) ).
*        <TARGET>-%CONTROL-ServiceUUID = IF_ABAP_BEHV=>MK-OFF.
      endloop.

*LDATA-SRV_UPD
*LDATA-TSKS_CREA
*LDATA-TSKS_CREA_WA
*LDATA-TSKS_DELE
*LDATA-TSKS_DELE_UN
*LDATA-TSKS_DELE_WA

    catch /PLCE/CX_BASEEXCEPTION into LEX_PD.
      PAR_LOG->ADD_EXCEPTION( IV_SEVERITY = 'W' IRO_EXCEPTION = LEX_PD ).
*    catch CX_SADL_SHORTDUMP into data(LEX_SADL).
*      data(LX_ROOT_CAUSE) = new CX_SADL_CONTRACT_VIOLATION( ).
*      CL_ABAP_UNIT_ASSERT=>ASSERT_EQUALS( ACT = LEX_SADL->PREVIOUS
*                                          EXP = LX_ROOT_CAUSE ).
  endtry.

endmethod.
ENDCLASS.


CLASS zcl_pd_workarea_funct_ext DEFINITION
  PUBLIC
  FINAL
  CREATE PUBLIC .

  PUBLIC SECTION.
  interfaces if_amdp_marker_hdb.
    class-methods get_workareas_services_ext for table function ZE_P_PDWorkAreaServiceFuncExt.

  PROTECTED SECTION.
  PRIVATE SECTION.
ENDCLASS.

CLASS zcl_pd_workarea_funct_ext IMPLEMENTATION.

METHOD get_workareas_services_ext
 by database function for hdb
        language sqlscript
        options read-only
        using  /PLCE/P_PDWORKAREASERVICEFUNC.

        return select distinct mandt, work_area, service_uuid
    from (
*    // Call the standard computed assignments (wrapper select)
    select mandt, work_area, service_uuid
    from "/PLCE/P_PDWORKAREASERVICEFUNC"( p_client => :p_client )
    );
ENDMETHOD.

ENDCLASS.

class ZCL_PLCP_TA_WA_ORDER_RSLT definition
  public
  inheriting from /PLCP/CL_TA_WA_ORDER_RSLT
  create public .

public section.
protected section.

  methods EXECUTE_DATA_UPLOAD
    redefinition .
private section.
ENDCLASS.



CLASS ZCL_PLCP_TA_WA_ORDER_RSLT IMPLEMENTATION.


  method EXECUTE_DATA_UPLOAD.

  include EEWA_EL_MACRO_RANGE.
  data:
      LORDERNRS   type standard table of EORDERNR,
      LORDERDATAS type standard table of EWA_ORDER_HEAD,
      LEXPORTDATA type ISU_WA_ORDERDOWN_S_EROUTE,
      LLOGHANDLE  type BALLOGHNDL,
      LLOG        type ref to CL_EEWA_IMPL_BO_CALLBACK_MLOG.

    LORDERNRS = value #( for L in FCHANGED_DATAS ( L-ORDERNR ) ).

    if LINES( LORDERNRS ) is not initial.

      sort LORDERNRS.
      delete adjacent duplicates from LORDERNRS.
      sort FCHANGED_DATAS by KEY.

      DATA_BO_RANGE X.
      SET_BO_RANGE_KEYS X __KEYS LORDERNRS.

      CL_EEWA_OR_WDORDER=>CL_READ(
        exporting
          PAR_OBJTYPE = CL_EEWA_BO_WDORDER=>COT_WDORDER
          PAR_RANGES  = LRANGEXS
        importing
          PAR_TABLE   = LORDERDATAS
      ).

      if LINES( LORDERDATAS ) is not initial.

        loop at LORDERDATAS reference into data(LORDERDATARF).
          insert initial line into table LEXPORTDATA-T_EORDER reference into data(LORDEREXPORTREF).
          LORDEREXPORTREF->WA_EORDER = LORDERDATARF->*.
          loop at FCHANGED_DATAS reference into data(LITEMDATARF) where ORDERNR = LORDERDATARF->ORDERNR.
            insert initial line into table LORDEREXPORTREF->T_ORDER_OBJ reference into data(LITMEEXPORTREF).
            LITMEEXPORTREF->WA_ORDER_OBJ = corresponding #( LITEMDATARF->* ).
          endloop.
        endloop.

        if LEXPORTDATA is not initial.
****      Log handle
          if FCALLBACK is not initial and FCALLBACK is instance of CL_EEWA_IMPL_BO_CALLBACK_MLOG.
            LLOG ?= FCALLBACK.
            LLOGHANDLE = LLOG->MSGLOG->HANDLE.
          endif.

          new ZCL_WR_PD_WA_WDOI_PDSERVICE( LLOGHANDLE )->CREATE_BY_PRINTDATA(
            exporting
              IS_PRINT_DATA = LEXPORTDATA
          ).
        endif.
      endif.
    endif.
  endmethod.
ENDCLASS.

"! Sets EWA_ORDER_OBJECT-ZZ_BMS_STATUS through the waste order item business
"! object — the same way ZCL_WR_PD_ORDER_CANCELATION->EXECUTE_CANCEL changes an EAP:
"!   getbo( lock ) -> dataref -> save_bo -> unlock_for_edit
"! No confirmation step is executed and no COMMIT is done here: the caller
"! (ZWR_BMS_UPDATE_SERVICE, own LUW via DESTINATION 'NONE') commits, exactly as
"! Z_WR_CANCEL_WASTE_ORDER_RFC does for the cancellation.
CLASS zcl_wr_pd_bms_eap_status DEFINITION
  PUBLIC
  INHERITING FROM /plcp/cl_ta_wa_order_rslt
  FINAL
  CREATE PUBLIC.

  PUBLIC SECTION.
    "! Returns abap_false when no EAP with this POBJNR exists.
    "! A foreign lock or a BO error raises cx_eewa_base (the BO is unlocked first).
    CLASS-METHODS set_bms_status
      IMPORTING iv_pobjnr       TYPE j_objnr
                iv_bms_status   TYPE ewa_order_object-zz_bms_status
      RETURNING VALUE(rv_found) TYPE abap_bool
      RAISING   cx_eewa_base.

  PRIVATE SECTION.
    METHODS execute_set_status
      IMPORTING iv_pobjnr       TYPE j_objnr
                iv_bms_status   TYPE ewa_order_object-zz_bms_status
      RETURNING VALUE(rv_found) TYPE abap_bool
      RAISING   cx_eewa_base.
ENDCLASS.



CLASS ZCL_WR_PD_BMS_EAP_STATUS IMPLEMENTATION.


  METHOD execute_set_status.
    DATA: ls_item_key TYPE ewa_order_object_ikey,
          lo_bo_item  TYPE REF TO zcl_wr_eewa_bo_wdorderitem.

    rv_found = abap_false.

    " 1. POBJNR -> item key (ORDERNR, ORDER_LAUFNR), as in EXECUTE_CANCEL
    SELECT SINGLE ordernr, order_laufnr
      FROM ewa_order_object
      WHERE pobjnr = @iv_pobjnr
      INTO @ls_item_key.
    IF sy-subrc <> 0.
      RETURN.
    ENDIF.
    rv_found = abap_true.

    " 2. waste order item BO with lock (inherited GETBO). A foreign lock
    "    raises cx_eewa_base — nothing to unlock yet, let it propagate.
    lo_bo_item ?= getbo( par_objtype = cl_eewa_bo_wdorderitem=>cot_wdorderitem
                         par_key     = ls_item_key
                         par_lock    = abap_true ).

    " 3. the one change. Same value already there -> release the lock, no save.
    IF lo_bo_item->dataref->zz_bms_status = iv_bms_status.
      lo_bo_item->unlock_for_edit( ).
      RETURN.
    ENDIF.

    lo_bo_item->dataref->zz_bms_status = iv_bms_status.

    " 4. save through the BO and unlock — as in EXECUTE_CANCEL
    TRY.
        save_bo( lo_bo_item ).
        lo_bo_item->unlock_for_edit( ).
      CATCH cx_eewa_base INTO DATA(lx_bo).
        TRY.
            lo_bo_item->unlock_for_edit( ).
          CATCH cx_eewa_base.
            " already unlocked or lock lost — nothing sensible to do
        ENDTRY.
        RAISE EXCEPTION lx_bo.
    ENDTRY.
  ENDMETHOD.


  METHOD set_bms_status.
    " static wrapper — the inherited GETBO / SAVE_BO are instance methods
    rv_found = NEW zcl_wr_pd_bms_eap_status( )->execute_set_status(
                 iv_pobjnr     = iv_pobjnr
                 iv_bms_status = iv_bms_status ).
  ENDMETHOD.
ENDCLASS.

class ZCL_WR_PD_IMP_WASTE_ORDERS definition
  public
  inheriting from /PLCE/CL_PD_IMP_WASTE_ORDERS
  create public .

public section.
protected section.

  methods PROCESS_SERVICES
    redefinition .
private section.
ENDCLASS.



CLASS ZCL_WR_PD_IMP_WASTE_ORDERS IMPLEMENTATION.


  method PROCESS_SERVICES.
    data:
      LPROFILEREF  type ref to TS_WASTEORDER_PROFILE,
      LOBJ         type ref to ZCL_PD_PROC_WASTE_ORDERS,
      LSRVKEYS     type /PLCE/PPDSRV_KEY,
      LMAPPINGKEYS type TP_WDOI_SRV_MAPPING.

    create object LOBJ exporting IO_MSGLOG = FO_MSGLOG.

    loop at IT_PROFILE_DATA reference into LPROFILEREF.
      clear:
        LSRVKEYS,
        LMAPPINGKEYS.

      LOBJ->PROCESS_WASTEORDERS(
        exporting
          IV_PROFILE     = LPROFILEREF->PROFILE
          IT_WASTEORDER  = LPROFILEREF->WASTEORDERS
*          IO_MSGLOG      =
        importing
          ET_SRV_MAPPING = LMAPPINGKEYS
          ET_SRV_KEY    = LSRVKEYS
      ).
      insert lines of LSRVKEYS into table ET_SERVICE_KEY.
      insert lines of LMAPPINGKEYS into table ET_SRV_MAPPING.

    endloop.
  endmethod.
ENDCLASS.

class ZCL_WR_PD_ORDER_CANCELATION definition
  public
  inheriting from /PLCP/CL_TA_WA_ORDER_RSLT
  final
  create public .

public section.

    " Static wrapper for easy calling from RAP / Interaction Center
  class-methods CANCEL_WASTE_ORDER_ITEM
    importing
      !IV_POBJNR type CHAR30
      !IV_REASON_PREDEFINED type CHAR8 optional
      !IV_REASON_TEXT type STRING optional
      !IV_SERVICEUUID type /PLCE/PDSERVICE_UUID optional
    raising
      CX_EEWA_BASE .
private section.

    " Instance method to actually execute the logic using inherited methods
  methods EXECUTE_CANCEL
    importing
      !IV_POBJNR type CHAR30
      !IV_REASON_PREDEFINED type CHAR8 optional
      !IV_REASON_TEXT type STRING optional
      !IV_SERVICEUUID type /PLCE/PDSERVICE_UUID optional
    raising
      CX_EEWA_BASE .
ENDCLASS.



CLASS ZCL_WR_PD_ORDER_CANCELATION IMPLEMENTATION.


  METHOD CANCEL_WASTE_ORDER_ITEM.
    " 1. Instantiate this class to access the inherited instance methods
    DATA(lo_api) = NEW zcl_wr_pd_order_cancelation( ).
    " 2. Call the instance method
    lo_api->execute_cancel(
      iv_pobjnr      = iv_pobjnr
      IV_REASON_PREDEFINED = iv_reason_predefined
      iv_reason_text = iv_reason_text
      iv_serviceuuid = iv_serviceuuid
    ).
  ENDMETHOD.


  METHOD EXECUTE_CANCEL.
    DATA: ls_item_key TYPE ewa_order_object_ikey,
          lo_bo_item  TYPE REF TO zcl_wr_eewa_bo_wdorderitem.

    " 1. Finde die Bestellposten-Schlüssel basierend auf POBJNR
    SELECT SINGLE ordernr, order_laufnr
      FROM ewa_order_object
      INTO @ls_item_key
      WHERE pobjnr = @iv_pobjnr.

    CHECK sy-subrc = 0.

  "  2. Instantiieren Sie das IS-U Waste Order Item BO mit dem geerbten GETBO
    lo_bo_item ?= getbo(
        par_objtype = cl_eewa_bo_wdorderitem=>cot_wdorderitem
        par_key     = ls_item_key
        par_lock    = 'X' ).

*   3. Wenden Sie die Storno-Aktualisierungen auf die BO-Datenreferenz an
    lo_bo_item->dataref->conftype1 = iv_reason_predefined.  " Vordefinierte Grund
    lo_bo_item->dataref->text = iv_reason_text." Gegebene Grund
    lo_bo_item->dataref->eorder_status = 03. "Negative Rückmeldung
    TRY.
*        Standard-Item-BO-Methoden zur Überprüfung und Ausführung der Bestätigung
        lo_bo_item->check_book_confirm_neg( ). "Negative Rückmeldung überprüfung
        lo_bo_item->book_confirm_neg( ).       "Negative Rückmeldung

        " -----------------------------------------------------------------
        ""TRIGGER HINTERGRUNDAUFGABE HIER!
       " Dies ist völlig für RAP verborgen und völlig sicher.
        " -----------------------------------------------------------------
        IF iv_serviceuuid IS NOT INITIAL.
          CALL FUNCTION 'Z_WR_DELAYED_HARD_DELETE_RFC' IN BACKGROUND TASK
            EXPORTING
              iv_serviceuuid = iv_serviceuuid.
        ENDIF.
        " -----------------------------------------------------------------

*       5. Speichern Sie mit dem geerbten SAVE_BO und entsperren Sie
        save_bo( lo_bo_item ).
        lo_bo_item->unlock_for_edit( ).

      CATCH cx_eewa_base INTO DATA(lex).
*        Stellen Sie sicher, dass wir entsperren, bevor der Fehler an Fiori weitergegeben wird
        lo_bo_item->unlock_for_edit( ).
        RAISE EXCEPTION lex.
    ENDTRY.

  ENDMETHOD.
ENDCLASS.

class ZCL_WR_PD_TOUR_HELPER definition
  public
  create public .

public section.

  types:
  " input: one line per EAP that is about to be released
    BEGIN OF ty_sig_order,
    pobjnr    TYPE j_objnr,
    noteintnr TYPE /watp/dnoteintnr,     " EWA_ORDER_OBJECT-/WATP/NOTEINTNR
  END OF ty_sig_order .
  types:
    tt_sig_orders TYPE STANDARD TABLE OF ty_sig_order WITH EMPTY KEY .
  types:
  " output: one line per EAP that must NOT be released
    BEGIN OF ty_sig_missing,
    pobjnr    TYPE j_objnr,
    noteintnr TYPE /watp/dnoteintnr,     " the CHECKED Schein (the SBS for an ÜS); empty = EAP has no Schein
    msgno     TYPE symsgno,              " message in Z_MSG_SVR_TOUR_EXT; &1 = EA-ID is added by the caller
    v2        TYPE string,               " &2 = external Schein number, readable
    v3        TYPE string,               " &3 = Scheinart (CERTTYPE, e.g. BS_DE)
  END OF ty_sig_missing .
  types:
    tt_sig_missing TYPE HASHED TABLE OF ty_sig_missing WITH UNIQUE KEY pobjnr .
  types:
  " output of GET_EANV_NOTES: one line per EAP whose Schein hangs on an eANV Nachweis
    BEGIN OF ty_eanv_note,
    pobjnr    TYPE j_objnr,
    noteintnr TYPE /watp/dnoteintnr,     " the Schein on the EAP (internal number)
    notenr    TYPE string,               " external Schein number, readable (msg 057 &2)
    certtype  TYPE string,               " Scheinart (CERTTYPE, e.g. BS_DE), not in the message today
    nwnr      TYPE string,               " external Nachweis number, e.g. SNFG12312344 (msg 057 &4)
  END OF ty_eanv_note .
  types:
    tt_eanv_notes TYPE HASHED TABLE OF ty_eanv_note WITH UNIQUE KEY pobjnr .

  constants:
    BEGIN OF gc_certtype,
    bs  TYPE /watp/dcerttype VALUE 'BS_DE', "zcl_wr_eewa_bo_wdorderitem=>zzc_certtype_bs,
    ues TYPE /watp/dcerttype VALUE 'UES_DE', " zcl_wr_eewa_bo_wdorderitem=>zzc_certtype_ues,
    sbs TYPE /watp/dcerttype VALUE 'SBS_DE', " zcl_wr_eewa_bo_wdorderitem=>zzc_certtype_sbs,
  END OF gc_certtype .
  constants:
  " messages in Z_MSG_SVR_TOUR_EXT, one per reason (&1 EA-ID, &2 Schein, &3 Scheinart)
    BEGIN OF gc_sig_msg,
    erz_missing  TYPE symsgno VALUE '042',   " Erzeuger-Signatur fehlt
    no_note      TYPE symsgno VALUE '047',   " EAP hat keinen Schein
    no_doc       TYPE symsgno VALUE '048',   " noch kein eANV-Dokument
    no_sbs       TYPE symsgno VALUE '049',   " ÜS ohne SBS
    not_found    TYPE symsgno VALUE '050',   " Scheinnummer existiert nicht
    bad_type     TYPE symsgno VALUE '051',   " unbekannte Scheinart
    bef_missing  TYPE symsgno VALUE '052',   " Beförderer-Signatur fehlt
    both_missing TYPE symsgno VALUE '053',   " beide fehlen
  END OF gc_sig_msg .
  " name returned by ZCL_WR_AXNS_INTERFACE_EANV->GET_INTERFACENAME
  constants GC_PROVIDER_AXIANS type STRING value 'AXIANS' ##NO_TEXT.

"! Electronic-signature check per Schein type (UST-S.AA.0006-005).
"! Returns every EAP that must not be released, with the message that
"! tells the dispatcher why. Rule: an EAP is sent only when its
"! Begleitschein / Sammelbegleitschein is proven signed (or receipted)
"! by producer AND carrier. Everything else is returned: no Schein,
"! unknown Schein, ÜS without SBS, unknown Scheinart, no eANV document,
"! missing signature. Only Scheine whose Nachweis takes part in eANV
"! (TARBNW-DIGI_SIG) are checked, and only when the active eANV
"! provider of the system is AXIANS.
  class-methods CHECK_NOTE_SIGNATURES
    importing
      !IT_ORDERS type TT_SIG_ORDERS
    returning
      value(RT_MISSING) type TT_SIG_MISSING .
"! Returns every EAP that carries a Schein (BS / ÜS / SBS) whose
"! Nachweis takes part in eANV (TARBNW-DIGI_SIG = 'X'). Nothing about
"! signatures is checked. Used by "Anlage ändern": such an EAP must
"! keep its disposal facility.
"! NOT returned: EAP without Schein, Schein on a paper Nachweis,
"! Schein number that does not exist, Nachweis flagged for deletion.
"! Everything is a mass read — call once per action call, not per service.
  class-methods GET_EANV_NOTES
    importing
      !IT_ORDERS type TT_SIG_ORDERS
    returning
      value(RT_NOTES) type TT_EANV_NOTES .
    "-----------------------------------------------------------------------
    " Helper: obtain a Bearer token from the BMS authentication endpoint.
    " Called once per tour (not once per service) to avoid redundant
    " auth calls. Token is valid for the duration of the tour loop.
    "-----------------------------------------------------------------------
  class-methods GET_BMS_BEARER_TOKEN
    importing
      !IV_DESTINATION type RFCDEST
      !IV_USERNAME type CHAR60
      !IV_PASSWORD type CHAR60
    exporting
      !EV_TOKEN type STRING
      !EV_ERROR type STRING .
    "-----------------------------------------------------------------------
    " Helper: POST a single serialized JSON order to the BMS endpoint.
    " Completely independent of the RAP framework — can be called and
    " tested directly without triggering the full action.
    "-----------------------------------------------------------------------
  class-methods POST_BMS_ORDER
    importing
      !IV_DESTINATION type RFCDEST
      !IV_BEARER_TOKEN type STRING
      !IV_JSON type STRING
    exporting
      !EV_HTTP_STATUS type I
      !EV_RESPONSE type STRING .
  class-methods LOG_BMS_CALL
    importing
      !IV_TOUR_UUID type /PLCE/PDTOUR_UUID
      !IV_SERVICE_UUID type /PLCE/PDSERVICE_UUID
      !IV_ORDER_NUMBER type AUFNR
      !IV_ENDPOINT type STRING
      !IV_HTTP_STATUS type I
      !IV_REQUEST type STRING
      !IV_RESPONSE type STRING
      !IV_DIRECTION type CHAR10 default 'OUTBOUND'
      !IV_POBJNR type J_OBJNR .
  PROTECTED SECTION.
  PRIVATE SECTION.

      "-----------------------------------------------------------------------
    " Path prefix of the BMS API (e.g. /BmsApiSapTest) for a destination,
    " from ZTOUR_BMS_CFG. Empty when not maintained. No trailing slash.
    "-----------------------------------------------------------------------
    CLASS-METHODS get_path_prefix
      IMPORTING iv_destination   TYPE rfcdest
      RETURNING VALUE(rv_prefix) TYPE string.

ENDCLASS.



CLASS ZCL_WR_PD_TOUR_HELPER IMPLEMENTATION.


  METHOD check_note_signatures.
*----------------------------------------------------------------------*
* Rule — identical to ZCL_WR_AXNS_INTERFACE_EANV->CHECK_IS_LAST_LAYER_SIGNED,
* restricted to the two layers producer (ERZ) and carrier (BEF):
*   producer OK = SIGNATUR_ERZ or QUITTUNG_ERZ filled
*   carrier  OK = SIGNATUR_BE1 or QUITTUNG_BEF filled
*
* Which Schein is checked:
*   BS_DE   the Schein on the EAP itself
*   UES_DE  NOT the ÜS — the SBS behind it. TARBNOTES-COLLECT_NOTE holds
*           the SBS's INTERNAL number (domain /WATP/NOTEINTNR).
*   SBS_DE  itself (not expected directly on an EAP, handled anyway)
*
* Decision 23.09.2026 (design A1/A9/A13): nothing that cannot be proven
* signed is released. Returned with their own message:
*   047  EAP without Schein
*   050  NOTEINTNR points to no Schein
*   049  ÜS without SBS
*   051  Scheinart not BS/ÜS/SBS
*   048  BS/SBS without eANV (ZKS) document
*   042 / 052 / 053  Erzeuger / Beförderer / both not signed
*
* Scope (AK HWS 21.09.2026 "nur eANV-teilnehmende NW"):
*   A11  only when the active eANV implementation of the system is
*        AXIANS (/WATP/CL_BO_ARBEANVCFG). The provider is a system-wide
*        setting, not a Nachweis attribute.
*   A12  only Scheine whose Nachweis (TARBNOTES-NWINTNR → TARBNW) has
*        DIGI_SIG = 'X'. A paper Nachweis never gets an electronic
*        signature → not checked, passes.
*
* Document choice (A6): the newest non-deleted ZKS document of the
* Schein (LOEVM = space, max ADM_AETIMESTAMP, then ADM_ERTIMESTAMP,
* then DOC_ID). A superseded signed document does not rescue an
* unsigned current one.
*
* Everything is a mass read — call once per action call, not per service.
*----------------------------------------------------------------------*
    TYPES:
      BEGIN OF ty_key,
        noteintnr TYPE /watp/dnoteintnr,
      END OF ty_key,
      tt_keys TYPE SORTED TABLE OF ty_key WITH UNIQUE KEY noteintnr,

      BEGIN OF ty_nw_key,
        nwintnr TYPE /watp/darbnwintnr,
      END OF ty_nw_key,
      tt_nw_keys TYPE SORTED TABLE OF ty_nw_key WITH UNIQUE KEY nwintnr,

      BEGIN OF ty_note,
        noteintnr    TYPE /watp/dnoteintnr,
        certtype     TYPE /watp/dcerttype,
        notenr       TYPE /watp/dnotenr,
        collect_note TYPE /watp/darbcollectnote,
        nwintnr      TYPE /watp/darbnwintnr,
      END OF ty_note,
      tt_notes TYPE HASHED TABLE OF ty_note WITH UNIQUE KEY noteintnr,

      BEGIN OF ty_nw,
        nwintnr  TYPE /watp/darbnwintnr,
        digi_sig TYPE /watp/darbdigi_sig,
      END OF ty_nw,
      tt_nw TYPE HASHED TABLE OF ty_nw WITH UNIQUE KEY nwintnr,

      BEGIN OF ty_doc,
        noteintnr       TYPE /watp/dnoteintnr,
        doc_id          TYPE /watp/darbzksdoc_id,
        adm_aetimestamp TYPE timestampl,
        adm_ertimestamp TYPE timestampl,
        signatur_erz    TYPE /watp/tarbzksdoc-signatur_erz,
        quittung_erz    TYPE /watp/tarbzksdoc-quittung_erz,
        signatur_be1    TYPE /watp/tarbzksdoc-signatur_be1,
        quittung_bef    TYPE /watp/tarbzksdoc-quittung_bef,
      END OF ty_doc,
      tt_docs TYPE STANDARD TABLE OF ty_doc WITH EMPTY KEY,

      " EAP → the Schein that has to be checked for it
      BEGIN OF ty_target,
        pobjnr    TYPE j_objnr,
        noteintnr TYPE /watp/dnoteintnr,
      END OF ty_target,
      tt_targets TYPE STANDARD TABLE OF ty_target WITH EMPTY KEY.

    DATA:
      lt_keys     TYPE tt_keys,
      lt_sbs_keys TYPE tt_keys,
      lt_doc_keys TYPE tt_keys,
      lt_nw_keys  TYPE tt_nw_keys,
      lt_notes    TYPE tt_notes,
      lt_sbs      TYPE tt_notes,
      lt_nw       TYPE tt_nw,
      lt_docs     TYPE tt_docs,
      lt_targets  TYPE tt_targets,
      lt_checked  TYPE tt_targets,
      lo_eanv     TYPE REF TO /watp/if_arb_interface_eanv,
      lv_provider TYPE string,
      lv_erz_ok   TYPE abap_bool,
      lv_bef_ok   TYPE abap_bool,
      lv_msgno    TYPE symsgno.

    FIELD-SYMBOLS:
      <o>  TYPE ty_sig_order,
      <n>  TYPE ty_note,
      <nw> TYPE ty_nw,
      <t>  TYPE ty_target,
      <d>  TYPE ty_doc,
      <m>  TYPE ty_sig_missing.

    " --- 0. system-wide guard: only when the active eANV provider is AXIANS (A11)
    "     No active implementation (exception) → nothing can be e-signed in
    "     this system → no check.
    TRY.
        lo_eanv     = /watp/cl_bo_arbeanvcfg=>cl_get_interface_eanv( ).
        lv_provider = to_upper( lo_eanv->get_interfacename( ) ).
      CATCH cx_root.
        CLEAR lv_provider.
    ENDTRY.

    IF lv_provider <> gc_provider_axians.
      RETURN.
    ENDIF.

    " --- 1. distinct Schein numbers of the EAPs ---------------------------
    "     A1: an EAP without a Schein has no signature that could be
    "     checked → it is not released either (message 047).
    LOOP AT it_orders ASSIGNING <o>.
      IF <o>-noteintnr IS INITIAL.
        INSERT VALUE #( pobjnr = <o>-pobjnr
                        msgno  = gc_sig_msg-no_note ) INTO TABLE rt_missing.
      ELSE.
        INSERT VALUE #( noteintnr = <o>-noteintnr ) INTO TABLE lt_keys.
      ENDIF.
    ENDLOOP.

    IF lt_keys IS INITIAL.
      RETURN.                     " no Schein on any EAP → all already rejected
    ENDIF.

    " --- 2. Schein master ---------------------------------------------------
    SELECT noteintnr, certtype, notenr, collect_note, nwintnr
      FROM /watp/tarbnotes
      FOR ALL ENTRIES IN @lt_keys
      WHERE noteintnr = @lt_keys-noteintnr
      INTO TABLE @lt_notes.

    " --- 3. the SBS behind each ÜS (second read, only the ones not loaded) --
    LOOP AT lt_notes ASSIGNING <n>
      WHERE certtype = gc_certtype-ues AND collect_note IS NOT INITIAL.
      IF NOT line_exists( lt_notes[ noteintnr = <n>-collect_note ] ).
        INSERT VALUE #( noteintnr = <n>-collect_note ) INTO TABLE lt_sbs_keys.
      ENDIF.
    ENDLOOP.

    IF lt_sbs_keys IS NOT INITIAL.
      SELECT noteintnr, certtype, notenr, collect_note, nwintnr
        FROM /watp/tarbnotes
        FOR ALL ENTRIES IN @lt_sbs_keys
        WHERE noteintnr = @lt_sbs_keys-noteintnr
        INTO TABLE @lt_sbs.

      LOOP AT lt_sbs ASSIGNING <n>.
        INSERT <n> INTO TABLE lt_notes.
      ENDLOOP.
    ENDIF.

    " --- 4. resolve: which Schein is checked for which EAP ------------------
    LOOP AT it_orders ASSIGNING <o> WHERE noteintnr IS NOT INITIAL.

      ASSIGN lt_notes[ noteintnr = <o>-noteintnr ] TO <n>.
      IF sy-subrc <> 0.
        " the EAP points to a Schein that does not exist (A10) → 050
        INSERT VALUE #( pobjnr    = <o>-pobjnr
                        noteintnr = <o>-noteintnr
                        msgno     = gc_sig_msg-not_found ) INTO TABLE rt_missing.
        CONTINUE.
      ENDIF.

      CASE <n>-certtype.

        WHEN gc_certtype-bs OR gc_certtype-sbs.
          APPEND VALUE #( pobjnr    = <o>-pobjnr
                          noteintnr = <n>-noteintnr ) TO lt_targets.

        WHEN gc_certtype-ues.
          IF <n>-collect_note IS INITIAL
             OR NOT line_exists( lt_notes[ noteintnr = <n>-collect_note ] ).
            " ÜS without a Sammelbegleitschein → cannot be signed (A8) → 049
            INSERT VALUE #( pobjnr    = <o>-pobjnr
                            noteintnr = <n>-noteintnr
                            msgno     = gc_sig_msg-no_sbs ) INTO TABLE rt_missing.
            CONTINUE.
          ENDIF.
          APPEND VALUE #( pobjnr    = <o>-pobjnr
                          noteintnr = <n>-collect_note ) TO lt_targets.

        WHEN OTHERS.
          " a Scheinart this check cannot prove signed → not released (A9) → 051
          " CERTTYPE has exactly three values today, so this is a safety net.
          INSERT VALUE #( pobjnr    = <o>-pobjnr
                          noteintnr = <n>-noteintnr
                          msgno     = gc_sig_msg-bad_type ) INTO TABLE rt_missing.

      ENDCASE.
    ENDLOOP.

    IF lt_targets IS NOT INITIAL.

      " --- 4b. only Scheine whose Nachweis takes part in eANV (A12) ---------
      LOOP AT lt_targets ASSIGNING <t>.
        ASSIGN lt_notes[ noteintnr = <t>-noteintnr ] TO <n>.
        IF sy-subrc = 0 AND <n>-nwintnr IS NOT INITIAL.
          INSERT VALUE #( nwintnr = <n>-nwintnr ) INTO TABLE lt_nw_keys.
        ENDIF.
      ENDLOOP.

      IF lt_nw_keys IS NOT INITIAL.
        SELECT nwintnr, digi_sig
          FROM /watp/tarbnw
          FOR ALL ENTRIES IN @lt_nw_keys
          WHERE nwintnr = @lt_nw_keys-nwintnr
            AND loevm   = @space
          INTO TABLE @lt_nw.
      ENDIF.

      LOOP AT lt_targets ASSIGNING <t>.
        ASSIGN lt_notes[ noteintnr = <t>-noteintnr ] TO <n>.
        CHECK sy-subrc = 0 AND <n>-nwintnr IS NOT INITIAL.

        READ TABLE lt_nw ASSIGNING <nw> WITH TABLE KEY nwintnr = <n>-nwintnr.
        CHECK sy-subrc = 0 AND <nw>-digi_sig = abap_true.

        APPEND <t> TO lt_checked.
      ENDLOOP.
    ENDIF.

    IF lt_checked IS NOT INITIAL.

      " --- 5. newest non-deleted ZKS document per checked Schein (A6) -------
      LOOP AT lt_checked ASSIGNING <t>.
        INSERT VALUE #( noteintnr = <t>-noteintnr ) INTO TABLE lt_doc_keys.
      ENDLOOP.

      SELECT noteintnr, doc_id, adm_aetimestamp, adm_ertimestamp,
             signatur_erz, quittung_erz, signatur_be1, quittung_bef
        FROM /watp/tarbzksdoc
        FOR ALL ENTRIES IN @lt_doc_keys
        WHERE noteintnr = @lt_doc_keys-noteintnr
          AND loevm     = @space
        INTO TABLE @lt_docs.

      SORT lt_docs BY noteintnr       ASCENDING
                      adm_aetimestamp DESCENDING
                      adm_ertimestamp DESCENDING
                      doc_id          DESCENDING.
      DELETE ADJACENT DUPLICATES FROM lt_docs COMPARING noteintnr.

      " --- 6. the rule ------------------------------------------------------
      LOOP AT lt_checked ASSIGNING <t>.

        CLEAR: lv_msgno, lv_erz_ok, lv_bef_ok.

        READ TABLE lt_docs ASSIGNING <d>
          WITH KEY noteintnr = <t>-noteintnr
          BINARY SEARCH.

        IF sy-subrc <> 0.
          lv_msgno = gc_sig_msg-no_doc.                            " A7 → 048
        ELSE.
          lv_erz_ok = xsdbool( <d>-signatur_erz IS NOT INITIAL
                            OR <d>-quittung_erz IS NOT INITIAL ).
          lv_bef_ok = xsdbool( <d>-signatur_be1 IS NOT INITIAL
                            OR <d>-quittung_bef IS NOT INITIAL ).

          lv_msgno = COND #(
            WHEN lv_erz_ok = abap_false AND lv_bef_ok = abap_false THEN gc_sig_msg-both_missing
            WHEN lv_erz_ok = abap_false                            THEN gc_sig_msg-erz_missing
            WHEN lv_bef_ok = abap_false                            THEN gc_sig_msg-bef_missing ).
        ENDIF.

        IF lv_msgno IS NOT INITIAL.
          INSERT VALUE #( pobjnr    = <t>-pobjnr
                          noteintnr = <t>-noteintnr
                          msgno     = lv_msgno ) INTO TABLE rt_missing.
        ENDIF.

      ENDLOOP.
    ENDIF.

    " --- 7. message variables &2 / &3 for every rejected EAP with a Schein ---
    "     &2 = external Schein number (NOTENR, else the internal number),
    "          ALPHA = OUT so the dispatcher never sees leading zeros
    "     &3 = Scheinart (CERTTYPE, customizing /WATP/TTCERTTYP), the
    "          same value the spec's message shows as "Scheinart X"
    LOOP AT rt_missing ASSIGNING <m> WHERE noteintnr IS NOT INITIAL.
      ASSIGN lt_notes[ noteintnr = <m>-noteintnr ] TO <n>.
      IF sy-subrc <> 0.
        " message 050: the Schein does not exist, only its number is known
        <m>-v2 = condense( |{ <m>-noteintnr ALPHA = OUT }| ).
        CONTINUE.
      ENDIF.

      <m>-v2 = COND #( WHEN <n>-notenr IS NOT INITIAL
                       THEN condense( |{ <n>-notenr    ALPHA = OUT }| )
                       ELSE condense( |{ <n>-noteintnr ALPHA = OUT }| ) ).
      <m>-v3 = condense( <n>-certtype ).
    ENDLOOP.

  ENDMETHOD.


  METHOD get_bms_bearer_token.

    CLEAR: ev_token, ev_error.

    DATA lo_http  TYPE REF TO if_http_client.
    DATA lv_code  TYPE i.
    DATA lv_subrc TYPE sy-subrc.

    " Serialise rather than concatenate — a password containing " or \
    " would otherwise produce invalid JSON.
    DATA: BEGIN OF ls_auth_req,
            username TYPE string,
            password TYPE string,
          END OF ls_auth_req.

    ls_auth_req-username = condense( CONV string( iv_username ) ).
    ls_auth_req-password = condense( CONV string( iv_password ) ).

    DATA(lv_auth_json) = /ui2/cl_json=>serialize(
      data        = ls_auth_req
      compress    = abap_false
      pretty_name = /ui2/cl_json=>pretty_mode-camel_case ).

    CLEAR ls_auth_req.

    cl_http_client=>create_by_destination(
      EXPORTING
        destination              = iv_destination
      IMPORTING
        client                   = lo_http
      EXCEPTIONS
        argument_not_found       = 1
        destination_not_found    = 2
        destination_no_authority = 3
        plugin_not_active        = 4
        internal_error           = 5
        OTHERS                   = 6 ).

    IF sy-subrc <> 0.
      lv_subrc = sy-subrc.
      MESSAGE e054(z_msg_svr_tour_ext) WITH iv_destination lv_subrc INTO ev_error.
      RETURN.
    ENDIF.

    " Path prefix from SM59, see POST_BMS_ORDER
*    DATA(lv_uri) = lo_http->request->get_header_field( '~request_uri' ).
*    IF lv_uri IS NOT INITIAL
*       AND substring( val = lv_uri off = strlen( lv_uri ) - 1 ) = '/'.
*      lv_uri = substring( val = lv_uri len = strlen( lv_uri ) - 1 ).
*    ENDIF.

DATA(lv_uri) = get_path_prefix( iv_destination ).

    cl_http_utility=>set_request_uri(
      request = lo_http->request
      uri     = |{ lv_uri }/api/user/authenticate/user| ).

    lo_http->request->set_method( if_http_request=>co_request_method_post ).
    lo_http->request->set_content_type( 'application/json' ).
    lo_http->request->set_cdata( lv_auth_json ).
    CLEAR lv_auth_json.

    lo_http->send( EXCEPTIONS OTHERS = 4 ).
    IF sy-subrc <> 0.
      lo_http->get_last_error( IMPORTING message = ev_error ).
      lo_http->close( ).
      RETURN.
    ENDIF.

    lo_http->receive( EXCEPTIONS OTHERS = 4 ).
    IF sy-subrc <> 0.
      lo_http->get_last_error( IMPORTING message = ev_error ).
      lo_http->close( ).
      RETURN.
    ENDIF.

    lo_http->response->get_status( IMPORTING code = lv_code ).
    DATA(lv_body) = lo_http->response->get_cdata( ).
    lo_http->close( ).

    IF lv_code <> 200.
      MESSAGE e055(z_msg_svr_tour_ext) WITH lv_code lv_body INTO ev_error.
      RETURN.
    ENDIF.

    TYPES: BEGIN OF ty_auth_data,
             access_token  TYPE string,
             refresh_token TYPE string,
           END OF ty_auth_data.
    TYPES: BEGIN OF ty_auth_response,
             data TYPE ty_auth_data,
           END OF ty_auth_response.

    DATA ls_auth TYPE ty_auth_response.

    /ui2/cl_json=>deserialize(
      EXPORTING json        = lv_body
                pretty_name = /ui2/cl_json=>pretty_mode-camel_case
      CHANGING  data        = ls_auth ).

    IF ls_auth-data-access_token IS INITIAL.
      MESSAGE e056(z_msg_svr_tour_ext) INTO ev_error.
      RETURN.
    ENDIF.

    ev_token = |Bearer { ls_auth-data-access_token }|.

  ENDMETHOD.


  METHOD post_bms_order.

    CLEAR: ev_http_status, ev_response.

    DATA lo_http  TYPE REF TO if_http_client.
    DATA lv_subrc TYPE sy-subrc.

    cl_http_client=>create_by_destination(
      EXPORTING
        destination              = iv_destination
      IMPORTING
        client                   = lo_http
      EXCEPTIONS
        argument_not_found       = 1
        destination_not_found    = 2
        destination_no_authority = 3
        plugin_not_active        = 4
        internal_error           = 5
        OTHERS                   = 6 ).

    IF sy-subrc <> 0.
      lv_subrc = sy-subrc.
      MESSAGE e054(z_msg_svr_tour_ext) WITH iv_destination lv_subrc INTO ev_response.
      RETURN.
    ENDIF.
    " The path prefix (/BmsApiSapTest in TI4) comes from ZTOUR_BMS_CFG,
    " field BMS_PATH_PREFIX of the destination's row, not from the coding.


*    DATA(lv_uri) = lo_http->request->get_header_field( '~request_uri' ).
*    IF lv_uri IS NOT INITIAL
*       AND substring( val = lv_uri off = strlen( lv_uri ) - 1 ) = '/'.
*      lv_uri = substring( val = lv_uri len = strlen( lv_uri ) - 1 ).
*    ENDIF.

 DATA(lv_uri) = get_path_prefix( iv_destination ).

    cl_http_utility=>set_request_uri(
      request = lo_http->request
      uri     = |{ lv_uri }/api/container/create-order-halle| ).

    lo_http->request->set_method( if_http_request=>co_request_method_post ).
    lo_http->request->set_content_type( 'application/json' ).
    lo_http->request->set_header_field( name  = 'Authorization'
                                        value = iv_bearer_token ).
    lo_http->request->set_cdata( iv_json ).

    lo_http->send( EXCEPTIONS OTHERS = 4 ).
    IF sy-subrc <> 0.
      lo_http->get_last_error( IMPORTING message = ev_response ).
      lo_http->close( ).
      RETURN.
    ENDIF.

    lo_http->receive( EXCEPTIONS OTHERS = 4 ).
    IF sy-subrc <> 0.
      lo_http->get_last_error( IMPORTING message = ev_response ).
      lo_http->close( ).
      RETURN.
    ENDIF.

    lo_http->response->get_status( IMPORTING code = ev_http_status ).
    ev_response = lo_http->response->get_cdata( ).
    lo_http->close( ).

  ENDMETHOD.


  METHOD log_bms_call.

    DATA lv_created_date TYPE dats.
    DATA lv_log_message  TYPE string.

    lv_created_date = sy-datum.

    " Extract the message field from a JSON error response if present.
    " BMS error responses follow the pattern {"error":{"message":"..."}}.
    " For success responses the full response is logged as-is.
    lv_log_message = iv_response.

    DATA(lv_msg_start) = find( val = iv_response sub = '"message":"' ).
    IF lv_msg_start >= 0.
      lv_msg_start = lv_msg_start + strlen( '"message":"' ).
      DATA(lv_msg_end) = find( val = iv_response sub = '"' off = lv_msg_start ).
      IF lv_msg_end > lv_msg_start.
        lv_log_message = substring( val = iv_response
                                    off = lv_msg_start
                                    len = lv_msg_end - lv_msg_start ).
      ENDIF.
    ENDIF.

    " DIRECTION is the caller's: OUTBOUND (default) for the release and the
    " storno, INBOUND for the confirmation endpoint (ZCL_BMS_EAP_CONF).
    CALL FUNCTION 'ZWR_BMS_LOG_WRITE'
      DESTINATION 'NONE'
      EXPORTING
        iv_mandt        = sy-mandt
        iv_created_date = lv_created_date
        iv_created_time = sy-uzeit
        iv_created_by   = sy-uname
        iv_direction    = iv_direction
        iv_tour_uuid    = iv_tour_uuid
        iv_service_uuid = iv_service_uuid
        iv_order_number = iv_order_number
        iv_endpoint     = iv_endpoint
        iv_http_status  = iv_http_status
        iv_request      = iv_request
        iv_response     = lv_log_message
        iv_pobjnr       = iv_pobjnr.

  ENDMETHOD.


  METHOD get_eanv_notes.
*----------------------------------------------------------------------*
    TYPES:
      BEGIN OF ty_key,
        noteintnr TYPE /watp/dnoteintnr,
      END OF ty_key,
      tt_keys TYPE SORTED TABLE OF ty_key WITH UNIQUE KEY noteintnr,

      BEGIN OF ty_nw_key,
        nwintnr TYPE /watp/darbnwintnr,
      END OF ty_nw_key,
      tt_nw_keys TYPE SORTED TABLE OF ty_nw_key WITH UNIQUE KEY nwintnr,

      BEGIN OF ty_note,
        noteintnr    TYPE /watp/dnoteintnr,
        certtype     TYPE /watp/dcerttype,
        notenr       TYPE /watp/dnotenr,
        collect_note TYPE /watp/darbcollectnote,
        nwintnr      TYPE /watp/darbnwintnr,
      END OF ty_note,
      tt_notes TYPE HASHED TABLE OF ty_note WITH UNIQUE KEY noteintnr,

      BEGIN OF ty_nw,
        nwintnr  TYPE /watp/darbnwintnr,
        nwnr     TYPE /watp/tarbnw-nwnr,
        digi_sig TYPE /watp/darbdigi_sig,
      END OF ty_nw,
      tt_nw TYPE HASHED TABLE OF ty_nw WITH UNIQUE KEY nwintnr.

    DATA:
      lt_keys     TYPE tt_keys,
      lt_sbs_keys TYPE tt_keys,
      lt_nw_keys  TYPE tt_nw_keys,
      lt_notes    TYPE tt_notes,
      lt_sbs      TYPE tt_notes,
      lt_nw       TYPE tt_nw.

    FIELD-SYMBOLS:
      <o>  TYPE ty_sig_order,
      <n>  TYPE ty_note,
      <s>  TYPE ty_note,
      <nw> TYPE ty_nw.

    " --- 1. distinct Schein numbers; an EAP without Schein is not returned
    LOOP AT it_orders ASSIGNING <o> WHERE noteintnr IS NOT INITIAL.
      INSERT VALUE #( noteintnr = <o>-noteintnr ) INTO TABLE lt_keys.
    ENDLOOP.

    IF lt_keys IS INITIAL.
      RETURN.
    ENDIF.

    " --- 2. Schein master -----------------------------------------------------
    SELECT noteintnr, certtype, notenr, collect_note, nwintnr
      FROM /watp/tarbnotes
      FOR ALL ENTRIES IN @lt_keys
      WHERE noteintnr = @lt_keys-noteintnr
      INTO TABLE @lt_notes.

    IF lt_notes IS INITIAL.
      RETURN.                     " every number points to no Schein
    ENDIF.

    " --- 3. SBS behind an ÜS that has no Nachweis link of its own ------------
    LOOP AT lt_notes ASSIGNING <n>
      WHERE certtype     = gc_certtype-ues
        AND nwintnr      IS INITIAL
        AND collect_note IS NOT INITIAL.
      IF NOT line_exists( lt_notes[ noteintnr = <n>-collect_note ] ).
        INSERT VALUE #( noteintnr = <n>-collect_note ) INTO TABLE lt_sbs_keys.
      ENDIF.
    ENDLOOP.

    IF lt_sbs_keys IS NOT INITIAL.
      SELECT noteintnr, certtype, notenr, collect_note, nwintnr
        FROM /watp/tarbnotes
        FOR ALL ENTRIES IN @lt_sbs_keys
        WHERE noteintnr = @lt_sbs_keys-noteintnr
        INTO TABLE @lt_sbs.

      LOOP AT lt_sbs ASSIGNING <s>.
        INSERT <s> INTO TABLE lt_notes.
      ENDLOOP.
    ENDIF.

    " inherit the SBS's Nachweis into the ÜS (non-key field, in place)
    LOOP AT lt_notes ASSIGNING <n>
      WHERE certtype     = gc_certtype-ues
        AND nwintnr      IS INITIAL
        AND collect_note IS NOT INITIAL.
      READ TABLE lt_notes ASSIGNING <s> WITH TABLE KEY noteintnr = <n>-collect_note.
      IF sy-subrc = 0.
        <n>-nwintnr = <s>-nwintnr.
      ENDIF.
    ENDLOOP.

    " --- 4. does the Nachweis take part in eANV? (A12) -----------------------
    LOOP AT lt_notes ASSIGNING <n> WHERE nwintnr IS NOT INITIAL.
      INSERT VALUE #( nwintnr = <n>-nwintnr ) INTO TABLE lt_nw_keys.
    ENDLOOP.

    IF lt_nw_keys IS INITIAL.
      RETURN.
    ENDIF.

    SELECT nwintnr, nwnr, digi_sig
      FROM /watp/tarbnw
      FOR ALL ENTRIES IN @lt_nw_keys
      WHERE nwintnr = @lt_nw_keys-nwintnr
        AND loevm   = @space
      INTO TABLE @lt_nw.

    " --- 5. result: EAP → its Schein, with the readable message values ------
    LOOP AT it_orders ASSIGNING <o> WHERE noteintnr IS NOT INITIAL.

      READ TABLE lt_notes ASSIGNING <n> WITH TABLE KEY noteintnr = <o>-noteintnr.
      CHECK sy-subrc = 0 AND <n>-nwintnr IS NOT INITIAL.

      READ TABLE lt_nw ASSIGNING <nw> WITH TABLE KEY nwintnr = <n>-nwintnr.
      CHECK sy-subrc = 0 AND <nw>-digi_sig = abap_true.

      INSERT VALUE #(
        pobjnr    = <o>-pobjnr
        noteintnr = <n>-noteintnr
        notenr    = COND #( WHEN <n>-notenr IS NOT INITIAL
                            THEN condense( |{ <n>-notenr    ALPHA = OUT }| )
                            ELSE condense( |{ <n>-noteintnr ALPHA = OUT }| ) )
        certtype  = condense( <n>-certtype )
        nwnr      = condense( <nw>-nwnr )
      ) INTO TABLE rt_notes.

    ENDLOOP.

  ENDMETHOD.


  METHOD get_path_prefix.

    CLEAR rv_prefix.

    " Same row the action reads: CONFIG_ID = DEFAULT. The destination is
    " only checked when the row names one, so a wrong destination in the
    " call cannot silently pick up the wrong prefix.
    SELECT SINGLE bms_destination, bms_path_prefix
      FROM ztour_bms_cfg
      WHERE config_id = 'DEFAULT'
      INTO @DATA(ls_cfg).

    IF sy-subrc <> 0.
      RETURN.
    ENDIF.

    IF ls_cfg-bms_destination IS NOT INITIAL
       AND ls_cfg-bms_destination <> iv_destination.
      RETURN.
    ENDIF.

    rv_prefix = condense( CONV string( ls_cfg-bms_path_prefix ) ).

    IF rv_prefix IS NOT INITIAL
       AND substring( val = rv_prefix off = strlen( rv_prefix ) - 1 ) = '/'.
      rv_prefix = substring( val = rv_prefix len = strlen( rv_prefix ) - 1 ).
    ENDIF.

  ENDMETHOD.
ENDCLASS.

class ZCL_WR_PD_WA_WDOI_PDSERVICE definition
  public
  inheriting from /PLCP/CL_WA_WDOI_PDSERVICE
  create public .

public section.

  methods CREATE_BY_PRINTDATA
    redefinition .
  methods IMPORT_EXT_DATA
    redefinition .
protected section.
private section.
ENDCLASS.



CLASS ZCL_WR_PD_WA_WDOI_PDSERVICE IMPLEMENTATION.


  method CREATE_BY_PRINTDATA.
        define _CHECK_BAPIRET.

      loop at LBAPIRETTAB into LBAPIRET.

        if LBAPIRET-ID is not initial and LBAPIRET-NUMBER is not initial.
          LS_MESSAGE_DATA-MSGTY     = LBAPIRET-TYPE.
          LS_MESSAGE_DATA-MSGID     = LBAPIRET-ID.
          LS_MESSAGE_DATA-MSGNO     = LBAPIRET-NUMBER.
          LS_MESSAGE_DATA-MSGV1     = LBAPIRET-MESSAGE_V1.
          LS_MESSAGE_DATA-MSGV2     = LBAPIRET-MESSAGE_V2.
          LS_MESSAGE_DATA-MSGV3     = LBAPIRET-MESSAGE_V3.
          LS_MESSAGE_DATA-MSGV4     = LBAPIRET-MESSAGE_V4.

          LOG_MESSAGE( LS_MESSAGE_DATA ).
        else.
          LOG_FREE_TEXT( exporting IV_MESSAGE_TYPE = LBAPIRET-TYPE IV_TEXT = LBAPIRET-MESSAGE ).
        endif.

      endloop.

    end-of-definition.


    data:
      LOBJ            type ref to ZCL_WR_PD_IMP_WASTE_ORDERS,
      LBAPIRETTAB     type BAPIRETTAB,
      LBAPIRET        type BAPIRET2,
      LS_MESSAGE_DATA type BAL_S_MSG.


    try.
        CLEAR( ).
        CONVERT_PRINTDATA( IS_PRINT_DATA = IS_PRINT_DATA ).

        PROVIDE_MASTERDATA( importing ET_BAPIRETTAB = LBAPIRETTAB ).

        _CHECK_BAPIRET.

        clear: LBAPIRETTAB.

        if LINES( FT_EXTERNAL_DATA ) is not initial.
          create object LOBJ.
          LOBJ->IMPORT_WASTE_ORDERS( exporting IT_WASTE_ORDER = corresponding #( deep FT_EXTERNAL_DATA ) importing ET_BAPIRETTAB = LBAPIRETTAB ).

          _CHECK_BAPIRET.
        else.
*        TODO: Meldung keine Daten zum exportieren
          LOG_MESSAGE(
            exporting
              IS_MESSAGE_DATA = value #( MSGID = '/PLCP/MC_WA' MSGNO = '037' MSGTY = 'S' )
            ).
        endif.

      catch /PLCP/CX_BASE into data(LO_EX).
        LOG_EXCEPTION( IO_EXCEPTION = LO_EX IV_MESSAGE_TYPE = C_MESSAGE_TYPE-ERROR ).
      catch /PLCE/CX_BASEEXCEPTION into data(LOE_EX).
        LOG_EXCEPTION( IO_EXCEPTION = LOE_EX IV_MESSAGE_TYPE = C_MESSAGE_TYPE-ERROR ).
    endtry.
  endmethod.


  method IMPORT_EXT_DATA.
    data:
      LOBJ            type ref to ZCL_WR_PD_IMP_WASTE_ORDERS,
      LBAPIRETTAB     type BAPIRETTAB,
      LS_MESSAGE_DATA type BAL_S_MSG,
      LBAPIRET        type BAPIRET2.



    try.
        CLEAR( ).
*        FS_PRINT_DATA = IS_PRINT_DATA.
*        CONVERT_PRINTDATA( IS_PRINT_DATA = IS_PRINT_DATA ).
        FT_EXTERNAL_DATA = corresponding #( IT_WDOI_DATA ).

****  todo somthing lookup

        create object LOBJ.
*new ZCL_PD_IMPORT_WASTEORDERS( )->IMPORT_WASTE_ORDERS( IT_WASTE_ORDER = LDATAS ).
        LOBJ->IMPORT_WASTE_ORDERS( exporting IT_WASTE_ORDER = corresponding #( FT_EXTERNAL_DATA ) importing ET_BAPIRETTAB = ET_BAPIRET ).

      catch /PLCP/CX_BASE into data(LO_EX).
        LOG_EXCEPTION( IO_EXCEPTION = LO_EX IV_MESSAGE_TYPE = C_MESSAGE_TYPE-ERROR ).
        LBAPIRET = LO_EX->GET_AS_BAPIRET( ).
        insert LBAPIRET into table ET_BAPIRET.


      catch /PLCE/CX_BASEEXCEPTION into data(LOE_EX).
        LOG_EXCEPTION( IO_EXCEPTION = LOE_EX IV_MESSAGE_TYPE = C_MESSAGE_TYPE-ERROR ).
        LBAPIRET = LOE_EX->GET_AS_BAPIRET( ).
        insert LBAPIRET into table ET_BAPIRET.
    endtry.


  endmethod.
ENDCLASS.

CLASS zcl_wr_service_extend_calc DEFINITION
  PUBLIC
  CREATE PUBLIC .

  PUBLIC SECTION.
    INTERFACES if_sadl_exit .
    INTERFACES if_sadl_exit_calc_element_read .
  PROTECTED SECTION.
  PRIVATE SECTION.
ENDCLASS.



CLASS zcl_wr_service_extend_calc IMPLEMENTATION.


  METHOD if_sadl_exit_calc_element_read~calculate.
    FIELD-SYMBOLS: <ls_original>             TYPE any,
                   <ls_calculated>           TYPE any,
                   <lv_refid>                TYPE any,
                   <lv_service_criticality>  TYPE int1,
                   <lv_time_adj>             TYPE any,
                   <lv_entsorgungs>          TYPE any,
                   <lv_entsorgungs_original> TYPE any,
                   <lv_time_disp>            TYPE any,
                   <lv_main_position>        TYPE any,
                   <lv_main_pos_crit>        TYPE int1.

    DATA: lv_reactiontime TYPE zwr_d_ewmd_reactiontime,
          lv_time_num     TYPE i,
          lv_time_str     TYPE string.

*   ct_calculated_data = CORRESPONDING #(  it_original_data ).
    CLEAR ct_calculated_data.
    LOOP AT it_original_data ASSIGNING <ls_original>.
      " Prepare the calculated row
      APPEND INITIAL LINE TO ct_calculated_data ASSIGNING <ls_calculated>.
      MOVE-CORRESPONDING <ls_original> TO <ls_calculated>.

      " =======================================================================
      " Service Criticality
      " =======================================================================
*      ASSIGN COMPONENT 'REFERENCEID' OF STRUCTURE <ls_original> TO <lv_refid>.
      ASSIGN COMPONENT 'REFERENCEID' OF STRUCTURE <ls_original> TO <lv_refid>.
      IF sy-subrc = 0 AND <lv_refid> IS NOT INITIAL.

        CLEAR lv_reactiontime.
        " Fetch via compliant CDS
        SELECT SINGLE * "aliased field for ZZPOINT_ORIGIN_WDOI
          FROM zi_wr_ewa_order_object
          WHERE referenceid = @<lv_refid> INTO @DATA(ls_wr_ewa_order_object).
        lv_reactiontime  = ls_wr_ewa_order_object-zz_reactiontime.

        IF sy-subrc = 0.
          ASSIGN COMPONENT 'SERVICE_CRITICALITY' OF STRUCTURE <ls_calculated> TO <lv_service_criticality>.
          IF sy-subrc = 0.
            " Map to 1 (red/critical if replanned) or 0 (neutral)
            <lv_service_criticality> = COND #( WHEN lv_reactiontime IS NOT INITIAL THEN 1 WHEN  ls_wr_ewa_order_object-zz_informational_group IS NOT INITIAL "            ls_wr_ewa_order_object-zz_pobjnr_main  IS NOT INITIAL
                                                                                   THEN 3 ELSE 0 ).
          ENDIF.

        ENDIF.

        " =======================================================================
        "  Main Position Criticality (Green for Grouped Services)
*
*        ZZ_POBJNR_MAIN IS INITIAL ->Not grouped at all
*        POBJNR = ZZ_POBJNR_MAINMain -> position of a group
*        POBJNR ≠ ZZ_POBJNR_MAIN  -> Sub-position


        ASSIGN COMPONENT 'ZZ_MAIN_POS_CRITICALITY' OF STRUCTURE <ls_calculated> TO <lv_main_pos_crit>.
        IF sy-subrc = 0.
          IF ls_wr_ewa_order_object-zz_informational_group IS NOT INITIAL. "ls_wr_ewa_order_object-zz_pobjnr_main IS NOT INITIAL AND
            " Belongs to a group (main or sub) -> Green
            <lv_main_pos_crit> = 3.
          ELSE.
            " No group -> Neutral
            <lv_main_pos_crit> = 0.
          ENDIF.
        ENDIF.

        " =======================================================================
        " Time Adjustment Formatting (+37 MIN)
        " =======================================================================
        ASSIGN COMPONENT 'ZZ_TIMEADJUSTMENT' OF STRUCTURE <ls_original> TO <lv_time_adj>.
        IF sy-subrc = 0 AND <lv_time_adj> IS NOT INITIAL.
          lv_time_num = <lv_time_adj>.
        ELSE.
          lv_time_num = 0.
        ENDIF.

        " Format the string based on positive, negative, or zero
        IF lv_time_num > 0.
          lv_time_str = |+{ lv_time_num } MIN|.
        ELSEIF lv_time_num < 0.
          " Negative sign is automatically included in the variable, so just append ' MIN'
          lv_time_str = |{ lv_time_num } MIN|.
        ELSE.
          lv_time_str = |0 MIN|.
        ENDIF.

        " Assign the formatted string to the Virtual Display Field
        ASSIGN COMPONENT 'ZZ_TIMEADJUST_DISPLAY' OF STRUCTURE <ls_calculated> TO <lv_time_disp>.
        IF sy-subrc = 0.
          <lv_time_disp> = lv_time_str.
        ENDIF.


      ENDIF.
    ENDLOOP.
  ENDMETHOD.


  METHOD if_sadl_exit_calc_element_read~get_calculation_info.
    " Please always add them in an alphabetical order
    APPEND 'REFERENCEID'              TO    et_requested_orig_elements.
    APPEND 'ZZ_TIMEADJUSTMENT'        TO    et_requested_orig_elements.
  ENDMETHOD.
ENDCLASS.

class ZCL_WR_TOUR_EXTEND_CALC definition
  public
  create public .

public section.

  interfaces IF_SADL_EXIT .
  interfaces IF_SADL_EXIT_CALC_ELEMENT_READ .
  PROTECTED SECTION.
private section.
ENDCLASS.



CLASS ZCL_WR_TOUR_EXTEND_CALC IMPLEMENTATION.


  METHOD if_sadl_exit_calc_element_read~get_calculation_info.
    " Request the base Tour fields needed for math and matching
    " Make sure they are added in an alphabetic order
    APPEND 'DRIVINGTIME'             TO et_requested_orig_elements.
     " --- NEW: Request the MaximumTourDuration and its Unit ---
    APPEND 'MAXIMUMTOURDURATION'     TO et_requested_orig_elements.
    APPEND 'MAXIMUMTOURDURATIONUNIT' TO et_requested_orig_elements.
    APPEND 'TOURCAPACITY'            TO et_requested_orig_elements.
    APPEND 'TOURDURATION'            TO et_requested_orig_elements.
    APPEND 'TOURUUID'                TO et_requested_orig_elements.
    APPEND 'ZZ_BMS_STATUS'           TO et_requested_orig_elements.
  ENDMETHOD.


   METHOD if_sadl_exit_calc_element_read~calculate.
    " -------------------------------------------------------------------------
    " 1. Data Declarations
    " -------------------------------------------------------------------------
    TYPES: BEGIN OF ty_tour_key,
             touruuid TYPE sysuuid_x16,
           END OF ty_tour_key.

    TYPES: BEGIN OF ty_service_data,
             touruuid             TYPE sysuuid_x16,
             totalserviceduration TYPE p LENGTH 10 DECIMALS 2,
             totaltimeadjustment  TYPE p LENGTH 10 DECIMALS 2,
             TotalTaskDuration  TYPE p LENGTH 10 DECIMALS 2,
           END OF ty_service_data.

    DATA: lt_tour_keys   TYPE STANDARD TABLE OF ty_tour_key,
          ls_tour_key    TYPE ty_tour_key,
          lt_service_agg TYPE STANDARD TABLE OF ty_service_data.

    DATA: lv_tour_uuid    TYPE sysuuid_x16,
          lv_tour_dur_h   TYPE p LENGTH 10 DECIMALS 2,
          lv_driving_h    TYPE p LENGTH 10 DECIMALS 2,

          lv_tour_dur_min TYPE p LENGTH 10 DECIMALS 2,
          lv_driving_min  TYPE p LENGTH 10 DECIMALS 2,

          lv_svc_dur_h    TYPE p LENGTH 10 DECIMALS 2,
          lv_svc_dur_min  TYPE p LENGTH 10 DECIMALS 2,
          lv_svc_adj_min  TYPE p LENGTH 10 DECIMALS 2,
          lv_svc_task_min  TYPE p LENGTH 10 DECIMALS 2,

          lv_new_dur_min  TYPE p LENGTH 10 DECIMALS 2,
          lv_rem_min      TYPE p LENGTH 10 DECIMALS 2,

          lv_max_dur      TYPE p LENGTH 10 DECIMALS 3,
          lv_max_dur_min  TYPE p LENGTH 10 DECIMALS 2. " Maximum capacity in minutes

    FIELD-SYMBOLS: <ls_orig>             TYPE any,
                   <ls_calc>             TYPE any,
                   <lv_val_uuid>         TYPE any,
                   <lv_val_dur>          TYPE any,
                   <lv_val_cap>          TYPE any,
                   <lv_val_drive>        TYPE any,

                   <lv_val_max_dur>      TYPE any,
                   <lv_val_max_dur_unit> TYPE any,

                   <lv_set_cap>          TYPE any,
                   <lv_set_rem>          TYPE any,
                   <lv_set_tot>          TYPE any,
                   <lv_set_unit>         TYPE any,
                   <lv_set_max_conv>     TYPE any.

    " -------------------------------------------------------------------------
    " 2. Collect all Tour UUIDs for our fast DB query
    " -------------------------------------------------------------------------
    LOOP AT it_original_data ASSIGNING <ls_orig>.
      ASSIGN COMPONENT 'TOURUUID' OF STRUCTURE <ls_orig> TO <lv_val_uuid>.
      IF sy-subrc = 0 AND <lv_val_uuid> IS NOT INITIAL.
        ls_tour_key-touruuid = <lv_val_uuid>.
        APPEND ls_tour_key TO lt_tour_keys.
      ENDIF.
    ENDLOOP.

    " Remove duplicates to make the query even faster
    SORT lt_tour_keys BY touruuid.
    DELETE ADJACENT DUPLICATES FROM lt_tour_keys COMPARING touruuid.

    " -------------------------------------------------------------------------
    " 3. Fetch Aggregated Service Data from our new CDS View
    " -------------------------------------------------------------------------
    IF lt_tour_keys IS NOT INITIAL.
      SELECT touruuid,
             totalserviceduration,
             totaltimeadjustment,
             TotalTaskDuration
        FROM zi_tour_svc_sum_wr   " <--- Must match your new CDS view name
        FOR ALL ENTRIES IN @lt_tour_keys
        WHERE touruuid = @lt_tour_keys-touruuid
        INTO TABLE @lt_service_agg.
    ENDIF.

    " -------------------------------------------------------------------------
    " 4. Perform math and assign to virtual fields row-by-row
    " -------------------------------------------------------------------------
    LOOP AT it_original_data ASSIGNING <ls_orig>.
      READ TABLE ct_calculated_data ASSIGNING <ls_calc> INDEX sy-tabix.
      CHECK sy-subrc = 0.

      " --- A. Get Source Tour Fields ---
      ASSIGN COMPONENT 'DRIVINGTIME'             OF STRUCTURE <ls_orig> TO <lv_val_drive>.
      ASSIGN COMPONENT 'TOURDURATION'            OF STRUCTURE <ls_orig> TO <lv_val_dur>.
      ASSIGN COMPONENT 'TOURCAPACITY'            OF STRUCTURE <ls_orig> TO <lv_val_cap>.
      ASSIGN COMPONENT 'TOURUUID'                OF STRUCTURE <ls_orig> TO <lv_val_uuid>.

      ASSIGN COMPONENT 'MAXIMUMTOURDURATION'     OF STRUCTURE <ls_orig> TO <lv_val_max_dur>.
      ASSIGN COMPONENT 'MAXIMUMTOURDURATIONUNIT' OF STRUCTURE <ls_orig> TO <lv_val_max_dur_unit>.

      IF sy-subrc <> 0. CONTINUE. ENDIF.

      lv_tour_uuid  = <lv_val_uuid>.
      lv_tour_dur_h = COND #( WHEN <lv_val_dur> IS ASSIGNED THEN <lv_val_dur> ELSE 0 ).
      lv_driving_h  = COND #( WHEN <lv_val_drive> IS ASSIGNED THEN <lv_val_drive> ELSE 0 ).

      " --- B. Convert Maximum Tour Duration to Minutes FIRST ---
      IF <lv_val_max_dur> IS ASSIGNED AND <lv_val_max_dur_unit> IS ASSIGNED.
        lv_max_dur = <lv_val_max_dur>.
        IF <lv_val_max_dur_unit> = 'H'.
          lv_max_dur_min = lv_max_dur * 60.
        ELSE.
          lv_max_dur_min = lv_max_dur. " Assume already MIN if not H
        ENDIF.
      ELSE.
        lv_max_dur_min = 0.
      ENDIF.

      " Convert standard Tour fields to Minutes
      lv_tour_dur_min = lv_tour_dur_h * 60.
      lv_driving_min  = lv_driving_h * 60.

      " --- C. Read pre-aggregated Service Data from our DB result ---
      READ TABLE lt_service_agg INTO DATA(ls_agg) WITH KEY touruuid = lv_tour_uuid.
      IF sy-subrc = 0.
        lv_svc_dur_h   = ls_agg-totalserviceduration.
        lv_svc_adj_min = ls_agg-totaltimeadjustment. " Already in minutes
        lv_svc_task_min = ls_agg-totaltaskduration.  " Task
      ELSE.
        CLEAR: lv_svc_dur_h, lv_svc_adj_min, lv_svc_task_min.
      ENDIF.

      " Convert Service TotalDuration to Minutes
      lv_svc_dur_min = lv_svc_dur_h * 60.

      " --- D. THE MATH ---
      IF lv_max_dur_min = 0.
        " If Maximum Duration is 0, force everything to 0 for a clean '0 von 0' UI
        lv_new_dur_min  = 0.
        lv_rem_min      = 0.
        lv_tour_dur_min = 0. " Overwrites target to 0
      ELSE.
        " 1. NewDuration = Service Duration(Min) + DrivingTime(Min) + Adjustment(Min)
        lv_new_dur_min = lv_svc_dur_min + lv_driving_min + lv_svc_adj_min + lv_svc_task_min.

        " 2. Remaining = Maximum Duration(Min) - NewDuration
        " (This correctly caps the available time to the Maximum Duration)
        lv_rem_min = lv_max_dur_min - lv_new_dur_min.
      ENDIF.

        " --- NEW: Determine Color (Criticality) ---
      DATA lv_crit TYPE int1.
      IF lv_max_dur_min = 0.
        lv_crit = 0. " Neutral/Grey
      ELSEIF lv_new_dur_min > lv_max_dur_min.
        lv_crit = 1. " 1 = RED (Over capacity)
      ELSEIF lv_new_dur_min >= ( lv_max_dur_min * '0.75' ).
        lv_crit = 2. " 2 = YELLOW (Warning: 90% or more full)
      ELSE.
        lv_crit = 3. " 3 = GREEN (Good)
      ENDIF.

      " Map the color to the UI
      ASSIGN COMPONENT 'ZZ_CAPACITY_CRITICALITY' OF STRUCTURE <ls_calc> TO FIELD-SYMBOL(<lv_set_crit>).
      IF sy-subrc = 0. <lv_set_crit> = lv_crit. ENDIF.

      " Map 1: tour_capacity_new = NewDuration (Used Capacity)
      ASSIGN COMPONENT 'TOUR_CAPACITY_NEW' OF STRUCTURE <ls_calc> TO <lv_set_cap>.
      IF sy-subrc = 0. <lv_set_cap> = lv_new_dur_min. ENDIF.

      " Map 2: tour_duration_min = TourDuration (Target for Progress Bar)
      ASSIGN COMPONENT 'TOUR_DURATION_MIN' OF STRUCTURE <ls_calc> TO <lv_set_tot>.
      IF sy-subrc = 0. <lv_set_tot> = lv_tour_dur_min. ENDIF.

      " Map 3: zz_remaining_duration = Remaining Time
      ASSIGN COMPONENT 'ZZ_REMAINING_DURATION' OF STRUCTURE <ls_calc> TO <lv_set_rem>.
      IF sy-subrc = 0. <lv_set_rem> = lv_rem_min. ENDIF.

      " Map 4: Force Unit to 'MIN' for the UI
      ASSIGN COMPONENT 'ZZ_DURATION_UNIT' OF STRUCTURE <ls_calc> TO <lv_set_unit>.
      IF sy-subrc = 0. <lv_set_unit> = 'MIN'. ENDIF.

      " Map 5: MaximumTourDuration_Converted
      ASSIGN COMPONENT 'MAXIMUMTOURDURATION_CONVERTED' OF STRUCTURE <ls_calc> TO <lv_set_max_conv>.
      IF sy-subrc = 0. <lv_set_max_conv> = lv_max_dur_min. ENDIF.

      " Map 6: Driving Time in Minutes
      ASSIGN COMPONENT 'ZZ_DRIVING_TIME_MIN' OF STRUCTURE <ls_calc> TO FIELD-SYMBOL(<lv_set_drive_min>).
      IF sy-subrc = 0. <lv_set_drive_min> = lv_driving_min. ENDIF.

       " Map 7: BMS criticality from ZZ_BMS_STATUS
      ASSIGN COMPONENT 'ZZ_BMS_STATUS' OF STRUCTURE <ls_orig>
        TO FIELD-SYMBOL(<lv_bms_status>).

      IF sy-subrc = 0.
        DATA lv_bms_crit TYPE int1.
        CASE <lv_bms_status>.
          WHEN 'SENT'        OR 'FREIGEGEBEN' OR 'BEENDET'.
            lv_bms_crit = 3.    " green
          WHEN 'PARTIAL'     OR 'BEGONNEN'.
            lv_bms_crit = 2.    " orange
          WHEN 'ERROR'.
            lv_bms_crit = 1.    " red
          WHEN OTHERS.
            lv_bms_crit = 0.    " grey — initial / STORNIERT
        ENDCASE.

        ASSIGN COMPONENT 'ZZBMSCRITICALITY' OF STRUCTURE <ls_calc>
          TO FIELD-SYMBOL(<lv_set_bms_crit>).
        IF sy-subrc = 0.
          <lv_set_bms_crit> = lv_bms_crit.
        ENDIF.
      ENDIF.


    ENDLOOP.
  ENDMETHOD.
ENDCLASS.


<img width="862" height="780" alt="image" src="https://github.com/user-attachments/assets/478f0d7b-dda2-4ba9-88d5-4555320a804a" />


{
  "formatVersion": "1",
  "header": {
    "description": "BMS connection configuration",
    "originalLanguage": "de"
  },
  "clientDependent": true,
  "languageDependent": "noLanguageKey",
  "category": "customizing",
  "transportRecording": "notSupported",
  "adaptableInProduction": false,
  "bcSetAdjustment": "automatic",
  "tableItems": [
    {
      "tableName": "ZTOUR_BMS_CFG",
      "systemControlled": true,
      "isPrimaryTable": true
    }
  ]
}

{
  "formatVersion": "1",
  "header": {
    "description": "Tour Template → BMS Kolonne Mapping",
    "originalLanguage": "de"
  },
  "clientDependent": true,
  "languageDependent": "noLanguageKey",
  "category": "customizing",
  "transportRecording": "automatic",
  "adaptableInProduction": false,
  "bcSetAdjustment": "automatic",
  "tableItems": [
    {
      "tableName": "ZTOUR_BMS_KOLONN",
      "systemControlled": true,
      "isPrimaryTable": true
    }
  ]
}

{
  "formatVersion": "1",
  "header": {
    "description": "Bermekungen für Storno",
    "originalLanguage": "de"
  },
  "clientDependent": true,
  "languageDependent": "noLanguageKey",
  "category": "customizing",
  "transportRecording": "notSupported",
  "adaptableInProduction": false,
  "tableItems": [
    {
      "tableName": "ZWR_PD_STORNO_BM",
      "systemControlled": true,
      "isPrimaryTable": true
    }
  ]
}

<img width="1412" height="820" alt="image" src="https://github.com/user-attachments/assets/979379cd-2100-46a3-a403-f6c8672c3eb6" />
<img width="1379" height="794" alt="image" src="https://github.com/user-attachments/assets/99960d99-7375-4c08-bc8c-6807818b8346" />






