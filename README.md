
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Planning and Dispatching Service'
@ObjectModel.semanticKey: ['ServiceId']
@ObjectModel.usageType:{
  serviceQuality: #D,
  sizeCategory: #XL,
  dataClass: #TRANSACTIONAL
}
define root view entity /PLCE/R_PDService
  as select from /plce/tpdsrv

  //base compositions
  composition [0..*] of /PLCE/R_PDServiceTask          as _ServiceTask
  composition [0..*] of /PLCE/R_PDServiceAttachment    as _Attachments
  composition [0..*] of /PLCE/R_PDServiceNote          as _Notes
  composition [0..*] of /PLCE/R_PDServiceStatusHistory as _StatusHistory

  //extensions
  composition [0..1] of /PLCE/R_PDServiceExtUNI        as _ExtUniversal
  composition [0..1] of /PLCE/R_PDServiceExtWR         as _ExtWaste
  composition [0..1] of /PLCE/R_PDServiceExtCustom     as _ExtCustom
  //associations
  association [1..1] to /PLCE/R_PDProfile              as _Profile            on $projection.Profile = _Profile.Profile
  association [1..1] to /PLCE/R_PDAction               as _Action             on $projection.Action = _Action.Action
  association [1..1] to /PLCE/R_PDServiceType          as _ServiceType        on $projection.ServiceType = _ServiceType.ServiceType
  association [1]    to /PLCE/R_PDService_L            as _ServiceLookup      on $projection.ServiceUUID = _ServiceLookup.ServiceUUID
  association [0..*] to /PLCE/P_PDPlanningStatusT      as _PlanningStatusText on $projection.PlanningStatus = _PlanningStatusText.Value
  association [0..*] to /PLCE/P_PDServiceWindowT       as _ServiceWindowText  on $projection.ServiceWindow = _ServiceWindowText.Value
  association [1]    to /PLCE/R_PDServiceStatistic     as _Statistic          on $projection.ServiceUUID = _Statistic.ServiceUUID
  association [0..1] to /PLCE/R_PDFunctionalLocation   as _FunctionalLocation on $projection.FunctionalLocation = _FunctionalLocation.FunctionalLocation

  association [0..*] to /PLCE/P_PDWorkAreaService      as _WorkAreaServices   on $projection.ServiceUUID = _WorkAreaServices.ServiceUUID
  association [0..*] to /PLCE/P_PDTourAssignment       as _TourAssignments    on $projection.ServiceUUID = _TourAssignments.ServiceUUID

  association [0..*] to /PLCE/R_PDMTourServiceResult   as _ServiceResults     on $projection.ServiceUUID = _ServiceResults.ServiceUUID
  association [0..*] to /PLCE/P_PDServiceStatusT       as _ServiceStatusText  on _ServiceStatusText.Value = $projection.ServiceStatus
  association [0..*] to /PLCE/P_PDCFDeviation_L        as _ServiceDeviation   on _ServiceDeviation.ServiceUUID = $projection.ServiceUUID
{
      @Semantics.uuid: true
  key service_uuid          as ServiceUUID,
      @EndUserText.label: 'Service Id'
      service_id            as ServiceId,
      @ObjectModel.foreignKey.association: '_Profile'
      @EndUserText.label: 'Profile'
      profile               as Profile,
      @ObjectModel.foreignKey.association: '_Action'
      @EndUserText.label: 'Action'
      action                as Action,
      @ObjectModel.foreignKey.association: '_ServiceType'
      @EndUserText.label: 'Service Type'
      service_type          as ServiceType,

      service_status        as ServiceStatus,

      @EndUserText.label: 'Order'
      reference_id          as ReferenceId,
      reference_int_id      as ReferenceInternalId,

      @EndUserText.label: 'Service Priority'
      service_priority      as ServicePriority,
      @EndUserText.label: 'Planning Status'
      planning_status       as PlanningStatus,

      @EndUserText.label: 'Requested Date'
      requested_date        as RequestedDate,
      @EndUserText.label: 'Earliest Date'
      earliest_date         as EarliestDate,
      @EndUserText.label: 'Latest Date'
      latest_date           as LatestDate,

      @EndUserText.label: 'Customer'
      customer_info         as CustomerInfo,
      @EndUserText.label: 'Additional Information'
      additional_text       as AdditionalText,
      //note                         as Note, //todo raus
      @ObjectModel.foreignKey.association: '_FunctionalLocation'
      functional_location   as FunctionalLocation,
      //TODO: Do we need a field for the PDCategory of a FunctionalLocation here as well?
      @EndUserText.label: 'Planning Plant'
      planning_plant        as PlanningPlant,

      @EndUserText.label: 'Service Window'
      service_window as ServiceWindow,
      @EndUserText.label: 'Service Window Start Time'
      service_window_start as ServiceWindowStartTime,
      @EndUserText.label: 'Service Window End Time'
      service_window_end as ServiceWindowEndTime,
//      concat_with_space( service_window, concat_with_space( service_window_start, service_window_end, 1 ), 1) as ServiceWindowFull,

//      service_freq          as ServiceFrequency,

      @Semantics.user.createdBy: true
      @EndUserText.label: 'Created By'
      created_by            as CreatedBy,
      @Semantics.systemDateTime.createdAt: true
      @EndUserText.label: 'Created At'
      created_at            as CreatedAt,
      @Semantics.user.lastChangedBy: true
      @EndUserText.label: 'Last Changed By'
      last_changed_by       as LastChangedBy,
      @Semantics.systemDateTime.lastChangedAt: true
      @EndUserText.label: 'Last Changed At'
      last_changed_at       as LastChangedAt,
      @Semantics.systemDateTime.localInstanceLastChangedAt: true
      @EndUserText.label: 'Local Last Changed At'
      local_last_changed_at as LocalLastChangedAt,


      //Association
      _ServiceTask,
      _Attachments,
      _Profile,
      _Action,
      _ServiceType,
      _FunctionalLocation,
      _ServiceLookup,
      _PlanningStatusText,
      _Statistic,
      _WorkAreaServices,
      _TourAssignments,
      _ServiceResults,
      _ServiceStatusText,
      _ServiceWindowText,
      _ServiceDeviation,
      _Notes,
      _StatusHistory,

      _ExtUniversal,
      _ExtWaste,
      _ExtCustom

}



@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Planning and Dispatching Tour'
@ObjectModel.semanticKey: ['TourId']
@ObjectModel.usageType:{
  serviceQuality: #D,
  sizeCategory: #XL,
  dataClass: #TRANSACTIONAL
}
define root view entity /PLCE/R_PDTour
  as select from /plce/tpdtour
  // compositions
  composition [0..*] of /PLCE/R_PDTourTaskAssignment as _TaskAssignments
  composition [0..*] of /PLCE/R_PDTourServiceAsgmt   as _ServiceAssignments
  composition [0..*] of /PLCE/R_PDTourResource       as _Resources
  composition [0..*] of /PLCE/R_PDTourAttachment     as _Attachments
  composition [0..*] of /PLCE/R_PDTourTask           as _TourTasks
  composition [0..*] of /PLCE/R_PDTourProductGroup   as _ProductGroups
  composition [0..*] of /PLCE/R_PDTourFuncLoc        as _FunctionalLocations
  composition [0..*] of /PLCE/R_PDTourPlant          as _Plants
  composition [0..*] of /PLCE/R_PDTourProfile        as _Profiles
  composition [0..1] of /PLCE/R_PDTourGeoRoute       as _GeoRoutes
  composition [0..*] of /PLCE/R_PDTourStatusHistory  as _StatusHistory
  //extensions  
  composition [0..1] of /PLCE/R_PDTourExtCustom      as _ExtCustom

  // associations
  //  association [1]    to /PLCE/R_PDProfile               as _Profile                on  $projection.PDProfile = _Profile.Profile
  association [1]    to /PLCE/R_PDTourTemplate       as _TourTemplate              on  $projection.TourTemplate = _TourTemplate.TourTemplate
  association [0..1] to /PLCE/R_PDFunctionalLocation as _PDStartFunctionalLocation on  $projection.StartFunctionalLocation = _PDStartFunctionalLocation.FunctionalLocation
  association [0..1] to /PLCE/R_PDFunctionalLocation as _PDEndFunctionalLocation   on  $projection.EndFunctionalLocation = _PDEndFunctionalLocation.FunctionalLocation
  association [0..*] to /PLCE/P_PDTourStatusT        as _PDTourStatusText          on  $projection.TourStatus = _PDTourStatusText.Value
  association [0..1] to /PLCE/R_PDTourResource       as _MainResource              on  $projection.MainResourceId = _MainResource.ResourceId
                                                                                   and $projection.TourUUID       = _MainResource.TourUUID
  association [1..1] to /PLCE/R_PDTour_L             as _TourLookup                on  $projection.TourUUID = _TourLookup.TourUUID
  //TODO anpassen nach AssignmentUmbau
  association [1]    to /PLCE/P_PDTourStatistic      as _Statistic                 on  _Statistic.TourUuid = $projection.TourUUID
  association [0..*] to /PLCE/P_PDWorkAreaTour       as _WorkAreaTours             on  _WorkAreaTours.TourUUID = $projection.TourUUID

  association [0..*] to /PLCE/R_PDMTourServiceResult as _TourResults               on  $projection.TourUUID = _TourResults.TourUUID

  association [0..*] to /PLCE/R_BGProcessingEntity   as _PendingProcessings        on  _PendingProcessings.EntityType = 'TOUR'
                                                                                   and _PendingProcessings.EntityUUID = $projection.TourUUID
{
      @Semantics.uuid: true
      @UI.hidden: true
  key tour_uuid                      as TourUUID,
      @EndUserText.label: 'Tour ID'
      tour_id                        as TourId,
      @ObjectModel.foreignKey.association: '_TourTemplate'
      tour_template                  as TourTemplate,
      //      tour_type                  as TourType,
//      @ObjectModel.text.association: '_PDTourStatusText'
      tour_status                    as TourStatus,

      @EndUserText.label: 'Scheduled Start'
      @Semantics.dateTime: true
      date_time_start                as ScheduledDateTimeStart, //TODO default calculation / later optimizer
      @EndUserText.label: 'Tour Start Date'
      date_start                     as StartDate,
      @EndUserText.label: 'Tour End Date'
      date_end                       as EndDate,
      earliest_start_time            as EarliestStartTime,
      latest_start_time              as LatestStartTime,
      start_functional_location      as StartFunctionalLocation,
      end_functional_location        as EndFunctionalLocation,
      @EndUserText.label: 'Maximum Tour Duration'
      @Semantics.quantity.unitOfMeasure: 'MaximumTourDurationUnit'
      maximum_tour_duration          as MaximumTourDuration,
      maximum_tour_duration_unit     as MaximumTourDurationUnit,

      @Semantics.quantity.unitOfMeasure: 'CalculatedTourDurationUnit'
      calculated_tour_duration       as CalculatedTourDuration,
      calculated_tour_duration_unit  as CalculatedTourDurationUnit,
      @EndUserText.label: 'Capacity used (in %)'
      calculated_tour_capacity       as CalculatedTourCapacity,

      external_tour_id               as ExternalTourID,
      created_for_simulation         as CreatedForSimulation,
      //      pd_profile                                           as PDProfile,

      @EndUserText.label: 'Main Resource'
      main_resource_id               as MainResourceId,

      @EndUserText.label: 'Additional Information'
      additional_text                as AdditionalText,

      @EndUserText.label: 'Resource Assignment Changed At'
      resource_assignment_changed_at as ResourceAssignmentChangedAt,
      // is updated when a /PLCE/R_PDTourTaskAssignment was changed 
      @EndUserText.label: 'Service Assignment Changed At'
      service_assignments_changed_at as ServiceAssignmentChangedAt,

      @EndUserText.label: 'Sequence Mode'
      sequence_mode                  as SequenceMode,


      @Semantics.user.createdBy: true
      @EndUserText.label: 'Created By'
      created_by                     as CreatedBy,
      @Semantics.systemDateTime.createdAt: true
      @EndUserText.label: 'Created At'
      created_at                     as CreatedAt,
      @Semantics.user.lastChangedBy: true
      @EndUserText.label: 'Last Changed By'
      last_changed_by                as LastChangedBy,
      @Semantics.systemDateTime.lastChangedAt: true
      @EndUserText.label: 'Last Changed At'
      last_changed_at                as LastChangedAt,
      @Semantics.systemDateTime.localInstanceLastChangedAt: true
      @EndUserText.label: 'Local Last Changed At'
      local_last_changed_at          as LocalLastChangedAt,

      //only for controlling process flow / no persistence
      cast('' as /plce/char20 )      as SessionProcessFlag,

      //Compositions
      _Resources,
      _Attachments,

      //Association
      _Statistic,
      _MainResource,
      _TourTasks,
      _TourTemplate,
      _TourLookup,
      _ProductGroups,
      _FunctionalLocations,
      _Plants,
      _PDStartFunctionalLocation,
      _PDEndFunctionalLocation,
      _PDTourStatusText,
      //      _Profile,
      _TaskAssignments,
      _ServiceAssignments,
      _WorkAreaTours,
      _Profiles,
      _TourResults,
      _GeoRoutes,
      _StatusHistory,
      _PendingProcessings,
      
      //Extension
      _ExtCustom
}

