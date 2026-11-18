class /PLCP/CL_WA_CONFIRMATION definition
  public
  create public .

public section.

  methods CONFIRM_TOURS
    importing
      !IV_PROFILE type /PLCE/PDPROFILE
      !IT_KEYS type /PLCE/PPDTOUR_KEY
    changing
      !CT_BAPIRET type BAPIRETTAB optional
    raising
      /PLCE/CX_BASEEXCEPTION .
  methods CONFIRM_SERVICES
    importing
      !IV_PROFILE type /PLCE/PDPROFILE
      !IT_KEYS type /PLCE/PPDSRV_KEY
    changing
      !CT_BAPIRET type BAPIRETTAB optional
    raising
      /PLCE/CX_BASEEXCEPTION .
  methods SEQUENCE_SERVICES
    importing
      !IV_PROFILE type /PLCE/PDPROFILE
      !IT_KEYS type /PLCE/PPDTASSRVC_KEY
      !IS_SEQUENCE_CONTROL type /PLCE/D_PDTOURTRANSFERSEQUENCE optional
    changing
      !CT_BAPIRET type BAPIRETTAB optional
    raising
      /PLCE/CX_BASEEXCEPTION .
protected section.

  methods READ_TOUR_DATA_CONFIRM
    importing
      !IV_PROFILE type /PLCE/PDPROFILE
      !IT_KEYS type /PLCE/PPDTOUR_KEY
    exporting
      !ET_DATA type /PLCE/PWR_TOUR_RSLT
    raising
      /PLCE/CX_BASEEXCEPTION .
  methods WRITE_DATA_TO_EEWA
    importing
      !IT_TOUR_DATA type /PLCE/PWR_TOUR_RSLT
      !IS_SEQUENCE_CONTROL type /PLCE/D_PDTOURTRANSFERSEQUENCE optional
    changing
      !CT_BAPIRET type BAPIRETTAB optional
    raising
      /PLCE/CX_BASEEXCEPTION .
  methods READ_SERVICE_DATA_CONFIRM
    importing
      !IV_PROFILE type /PLCE/PDPROFILE
      !IT_KEYS type /PLCE/PPDSRV_KEY
    exporting
      !ET_DATA type /PLCE/PWR_TOUR_RSLT
    raising
      /PLCE/CX_BASEEXCEPTION .
  methods READ_SERVICE_DATA_SEQUENCE
    importing
      !IV_PROFILE type /PLCE/PDPROFILE
      !IT_KEYS type /PLCE/PPDTASSRVC_KEY
    exporting
      !ET_DATA type /PLCE/PWR_TOUR_RSLT
    raising
      /PLCE/CX_BASEEXCEPTION .
private section.
ENDCLASS.



CLASS /PLCP/CL_WA_CONFIRMATION IMPLEMENTATION.


  method CONFIRM_SERVICES.

    data:
      LTOURS_RES  type /PLCE/PWR_TOUR_RSLT.

    READ_SERVICE_DATA_CONFIRM(
      exporting
        IV_PROFILE = IV_PROFILE               " Planning and Dispatching Profile
        IT_KEYS    = IT_KEYS                  " Table type for /PLCE/SPDTOUR_KEY
      importing
        ET_DATA    = LTOURS_RES               " WR Tour Results
    ).


    WRITE_DATA_TO_EEWA(
      exporting
        IT_TOUR_DATA = LTOURS_RES                 " WR Tour Results
      changing
        CT_BAPIRET = CT_BAPIRET
    ).

  endmethod.


  method CONFIRM_TOURS.

    data:
      LTOURS_RES  type /PLCE/PWR_TOUR_RSLT.

    READ_TOUR_DATA_CONFIRM(
      exporting
        IV_PROFILE = IV_PROFILE               " Planning and Dispatching Profile
        IT_KEYS    = IT_KEYS                  " Table type for /PLCE/SPDTOUR_KEY
      importing
        ET_DATA    = LTOURS_RES               " WR Tour Results
    ).


    WRITE_DATA_TO_EEWA(
      exporting
        IT_TOUR_DATA = LTOURS_RES                 " WR Tour Results
      changing
        CT_BAPIRET = CT_BAPIRET
    ).

  endmethod.


  method READ_SERVICE_DATA_CONFIRM.

    data:
      LEX         type ref to /PLCP/CX_EEWA_BASE,
      LBAPIRET    type BAPIRET2,
      LDATE       type DATS,
      LTIME       type TIMS,
      LCONFNOTES  type standard table of /PLCE/R_PDConfNote,
      LTOURRESREF     type ref to /PLCE/SWR_TOUR_RSLT,
      LCONST_TOURUUID type /PLCE/PDTOUR_UUID value '24310000000000000000000000000000'.


*    try.
***  Which Services?
    read entity /PLCE/R_PDService
      all fields with corresponding #( IT_KEYS mapping ServiceUUID = SERVICE_UUID )
      result data(LSERVICES)
      by \_Attachments all fields with corresponding #( IT_KEYS mapping ServiceUUID = SERVICE_UUID )
      result data(LSERVATTA).

    delete LSERVICES where Profile <> IV_PROFILE.

    if LINES( LSERVICES ) is not initial.
*** Which Tours ar affected?
      select * from /PLCE/P_PDTourAssignment
        for all entries in @LSERVICES
        where ServiceUUID = @LSERVICES-ServiceUUID
        into table @data(LTOURASSIGNMENTS).

      select * from /PLCE/R_PDMTourServiceResultWR
        for all entries in @LSERVICES
        where ServiceUUID = @LSERVICES-ServiceUUID
        into table @data(LSRVWARESULTS_FULL).

      select * from /PLCE/R_PDMTourWeighing
        for all entries in @LSERVICES
        where ServiceUUID = @LSERVICES-ServiceUUID
        into table @data(LSRVWGHS_FULL).

      select * from /PLCE/R_PDMTourConfNote
        for all entries in @LSERVICES
        where ServiceUUID = @LSERVICES-ServiceUUID
        into table @data(LSRVCFNTS_FULL) ##SELECT_FAE_WITH_LOB[ATTACHMENT].

      select * from /PLCE/R_PDMTourAddResult
        for all entries in @LSERVICES
        where ServiceUUID = @LSERVICES-ServiceUUID
        into table @data(LSRVADDS_FULL).

      select * from /PLCE/R_PDMTourDocResult
        for all entries in @LSERVICES
        where ServiceUUID = @LSERVICES-ServiceUUID
        into table @data(LSRVRESDOCS_FULL).


    endif.

*** Tour Assignment is mandatory! (ORDER_HEAD)
    if LINES( LTOURASSIGNMENTS ) is initial.
      raise exception type /PLCE/CX_BASEEXCEPTION
        message id '/PLCP/MC_WA' type 'W' number 034 ##NUMBER_OK.
    else.

      select TourUUID, ResourceId  from /PLCE/R_PDMTour
        for all entries in @LTOURASSIGNMENTS
        where TourUUID = @LTOURASSIGNMENTS-TourUUID
        into table @data(LT_KEYS).

    endif.

    read entity /PLCE/R_PDMTour
      all fields with corresponding #( LT_KEYS )
        result data(LPDMTOURS) " ?? needed?
      by \_ConfirmationNotes all fields with corresponding #( LT_KEYS )
        result data(LCONFNOTESRSLT)
      by \_ServiceResults all fields with corresponding #( LT_KEYS )
        result data(LSERVICERESULTS)
      by \_Activities all fields with corresponding #( LT_KEYS )
        result data(LACTIVITYRESULTS)
      by \_Weighings all fields with corresponding #( LT_KEYS )
        result data(LWEIGHINGRESULTS)
      by \_AdditionalResults all fields with corresponding #( LT_KEYS )
        result data(LADDRESULTS).

*** only Service data is requested!
    delete LCONFNOTESRSLT where ServiceUUID is initial.
    delete LWEIGHINGRESULTS where ServiceUUID is initial.
    delete LADDRESULTS where ServiceUUID is initial.

*** Filter for relevant services
    loop at LSERVICERESULTS reference into data(LSERVICERESULTREF).
      read table IT_KEYS reference into data(LKEYREF) with key SERVICE_UUID = LSERVICERESULTREF->ServiceUUID.
      if SY-SUBRC is not initial.
        delete LCONFNOTESRSLT where ServiceUUID = LSERVICERESULTREF->ServiceUUID.
        delete LWEIGHINGRESULTS where ServiceUUID = LSERVICERESULTREF->ServiceUUID.
        delete LADDRESULTS where ServiceUUID = LSERVICERESULTREF->ServiceUUID.
        delete LSERVICERESULTS.
        continue.
      endif.
    endloop.

*** Read additional data
    read entity /PLCE/R_PDMTourServiceResult
      by \_ExtWR all fields with corresponding #( LSERVICERESULTS )
      result data(LSERVICEWARESULTS).

    read entity /PLCE/R_PDTour
      all fields with corresponding #( LTOURASSIGNMENTS )
      result data(LTOURS).

    read entity /PLCE/R_MDResource
      all fields with corresponding #( LACTIVITYRESULTS )
      result data(LMDRESSOURCES).

    select ConfirmationNote, ExternalId from /PLCE/R_PDConfNote into corresponding fields of table @LCONFNOTES. "#EC CI_NOWHERE


**** Handle replaned service results
    loop at LSERVICEWARESULTS reference into data(LSERVICEWRRESULTREF).
      delete LSRVWARESULTS_FULL where TourUUID = LSERVICEWRRESULTREF->TourUUID and ResourceId = LSERVICEWRRESULTREF->ResourceId and ServiceUUID = LSERVICEWRRESULTREF->ServiceUUID.
      delete LSRVWGHS_FULL where TourUUID = LSERVICEWRRESULTREF->TourUUID and ResourceId = LSERVICEWRRESULTREF->ResourceId and ServiceUUID = LSERVICEWRRESULTREF->ServiceUUID.
      delete LSRVCFNTS_FULL where TourUUID = LSERVICEWRRESULTREF->TourUUID and ResourceId = LSERVICEWRRESULTREF->ResourceId and ServiceUUID = LSERVICEWRRESULTREF->ServiceUUID.
      delete LSRVADDS_FULL where TourUUID = LSERVICEWRRESULTREF->TourUUID and ResourceId = LSERVICEWRRESULTREF->ResourceId and ServiceUUID = LSERVICEWRRESULTREF->ServiceUUID.
*** ??? LSRVRESDOCS_FULL
*      delete LSRVRESDOCS_FULL where TourUUID = LSERVICEWRRESULTREF->TourUUID and ResourceId = LSERVICEWRRESULTREF->ResourceId and ServiceUUID = LSERVICEWRRESULTREF->ServiceUUID.

    endloop.

    loop at LSRVWARESULTS_FULL reference into data(LSRVWARESULTS_FULLREF).
*      LSRVWARESULTS_FULLREF->TourUUID = '2431'. " $1 as hex
      LSRVWARESULTS_FULLREF->TourUUID = LCONST_TOURUUID.
    endloop.

    loop at LSRVWGHS_FULL reference into data(LSRVWGHS_FULLREF).
*      LSRVWGHS_FULLREF->TourUUID = '2431'. " $1 as hex
      LSRVWGHS_FULLREF->TourUUID = LCONST_TOURUUID.
    endloop.

    loop at LSRVCFNTS_FULL reference into data(LSRVCFNTS_FULLREF).
*      LSRVCFNTS_FULLREF->TourUUID = '2431'. " $1 as hex
      LSRVCFNTS_FULLREF->TourUUID = LCONST_TOURUUID.
    endloop.

    loop at LSRVADDS_FULL reference into data(LSRVADDS_FULLREF).
*      LSRVADDS_FULLREF->TourUUID = '2431'. " $1 as hex
      LSRVADDS_FULLREF->TourUUID = LCONST_TOURUUID.
    endloop.

*    loop at LSRVRESDOCS_FULL reference into data(LSRVRESDOCS_FULLREF).
*      LSRVRESDOCS_FULLREF->TourUUID = '2431'. " $1 as hex
*    endloop.

    LSERVICEWARESULTS = corresponding #( base ( LSERVICEWARESULTS ) LSRVWARESULTS_FULL ).
    LWEIGHINGRESULTS = corresponding #( base ( LWEIGHINGRESULTS ) LSRVWGHS_FULL ).
    LCONFNOTESRSLT = corresponding #( base ( LCONFNOTESRSLT ) LSRVCFNTS_FULL ).
    LADDRESULTS = corresponding #( base ( LADDRESULTS ) LSRVADDS_FULL ).
*    LDOCRESULTS = corresponding #( base ( LDOCRESULTS ) LSRVRESDOCS_FULL ).


*** Create result data
    loop at LTOURS reference into data(LTOURREF).

      insert initial line into table ET_DATA reference into LTOURRESREF.
      LTOURRESREF->TOURID = LTOURREF->TourId.
      LTOURRESREF->TOURTEMPLATE = LTOURREF->TourTemplate.
      select single ExternalTourTemplateID from /PLCE/R_PDTourTemplate into @LTOURRESREF->ExternalTourTemplateID
        where TourTemplate = @LTOURREF->TourTemplate.

**** get order date
      loop at LACTIVITYRESULTS reference into data(LACTREF) using key ID where TourUUID = LTOURREF->TOURUUID and SERVICEUUID is initial and TASKUUID is initial.

        convert time stamp LACTREF->ACTIVITYEND time zone SY-ZONLO
          into date LDATE time LTIME.

        LTOURRESREF->TOURDATE = LACTREF->ACTIVITYEND.
      endloop.

      if LTOURRESREF->TOURDATE is initial.
        convert date LTOURREF->StartDate into time stamp LTOURRESREF->TOURDATE time zone SY-ZONLO.
      endif.

**** Service zu WDOI
      loop at LTOURASSIGNMENTS reference into data(LTOURASSIGNMENTREF) where TourUUID = LTOURREF->TOURUUID.
        read table LSERVICES reference into data(LSERVICEREF) with table key ENTITY components ServiceUUID = LTOURASSIGNMENTREF->ServiceUUID.

        if SY-SUBRC is initial.

          read table LTOURRESREF->SERVICES with key SERVICEUUID = LSERVICEREF->ServiceUUID reference into data(LSERVICEREF2).
          if SY-SUBRC is not initial.
***   new
            insert initial line into table LTOURRESREF->SERVICES reference into LSERVICEREF2.
            LSERVICEREF2->* = corresponding #( LSERVICEREF->* ).
            LSERVICEREF2->TOURSEQUENCE = LTOURASSIGNMENTREF->TourSequence.
          else. " existing
          endif.

*** Servicedaten WR
          loop at LSERVICEWARESULTS reference into LSERVICEWRRESULTREF
*                using key ID
*             where ( TourUUID = LTOURASSIGNMENTREF->TourUUID or TourUUID = '2431' ) and ServiceUUID = LSERVICEREF->ServiceUUID.
            where ( TourUUID = LTOURASSIGNMENTREF->TourUUID or TourUUID = LCONST_TOURUUID ) and ServiceUUID = LSERVICEREF->ServiceUUID.
            if LSERVICEREF2->RESULT is initial.
              LSERVICEREF2->RESULT = corresponding #( LSERVICEWRRESULTREF->* ).
              read table LSERVICERESULTS reference into LSERVICERESULTREF with key ServiceUUID = LSERVICEREF->ServiceUUID TourUUID = LTOURASSIGNMENTREF->TourUUID ##PRIMKEY[ID].
              if SY-SUBRC is initial.
                LSERVICEREF2->RESULT-TIMESTAMP = LSERVICERESULTREF->TIMESTAMP.
              endif.
            else.
              data(LCOMPONENTS) = cast CL_ABAP_STRUCTDESCR( CL_ABAP_STRUCTDESCR=>DESCRIBE_BY_DATA( P_DATA = LSERVICEWRRESULTREF->%DATA ) )->GET_COMPONENTS( ).
              data LPATH type STRING.
              field-symbols:
                <FIELD_SRC> type DATA,
                <FIELD_DST> type DATA.

              loop at LCOMPONENTS reference into data(LCOMPONENTREF) where AS_INCLUDE is initial.
                assign component LCOMPONENTREF->NAME of structure LSERVICERESULTREF->%DATA to <FIELD_DST>.
                LPATH = 'LSERVICEREF2->RESULT-' && LCOMPONENTREF->NAME.
                assign (LPATH) to <FIELD_SRC>.

                if <FIELD_SRC> is assigned and <FIELD_DST> is assigned.
                  if <FIELD_SRC> <> <FIELD_DST>.
                    LSERVICEREF2->INVALID_RESULT = 'X'.
                  endif.
                endif.

              endloop.
            endif.

          endloop.

**** WEIGH
**** CONFNOTES
**** ATTACHMENTS

          loop at LWEIGHINGRESULTS reference into data(LWEIGHREF)
*                using key ID
*            where ( TourUUID = LTOURASSIGNMENTREF->TourUUID or TourUUID = '2431' ) and ServiceUUID = LSERVICEREF->ServiceUUID.
            where ( TourUUID = LTOURASSIGNMENTREF->TourUUID or TourUUID = LCONST_TOURUUID ) and ServiceUUID = LSERVICEREF->ServiceUUID.
*             and TaskUUID is initial.
            insert initial line into table LSERVICEREF2->WEIGHINGS reference into data(LSRVWEIGHREF).
            LSRVWEIGHREF->* = corresponding #( LWEIGHREF->* ).
            if LSRVWEIGHREF->SERVICEUUID is not initial.
              LSRVWEIGHREF->POBJNR = LSERVICEREF2->REFERENCEINTERNALID.
              select single VEH~VEHICLELICENSEPLATE from /PLCE/R_MDVehicle as VEH
                inner join /PLCE/R_MDResource as RES
                  on RES~Equipment = VEH~Vehicle
                where RES~ResourceId = @LWEIGHREF->ResourceId
                into @LSRVWEIGHREF->VEHICLELICENSEPLATE.
            endif.
          endloop.

          sort LCONFNOTESRSLT by TimeStamp.

          loop at LCONFNOTESRSLT reference into data(LCONFNOTEREF)
*                using key ID
*            where ( TourUUID = LTOURASSIGNMENTREF->TourUUID or TourUUID = '2431' ) and ServiceUUID = LSERVICEREF->ServiceUUID.
            where ( TourUUID = LTOURASSIGNMENTREF->TourUUID or TourUUID = LCONST_TOURUUID ) and ServiceUUID = LSERVICEREF->ServiceUUID.
*            and TaskUUID is initial.
            insert initial line into table LSERVICEREF2->CONFNOTES reference into data(LSRVCONFNOTEREF).
            LSRVCONFNOTEREF->* = corresponding #( LCONFNOTEREF->* mapping ATTACHMENTUUID = ConfirmationNoteUUID ).
            read table LCONFNOTES reference into data(LC) with key ConfirmationNote = LCONFNOTEREF->ConfirmationNote.
            if SY-SUBRC is initial.
              LSRVCONFNOTEREF->EXTERNALID = LC->ExternalId.
            endif.
            if LSRVCONFNOTEREF->ATTACHMENTUUID is initial or LSRVCONFNOTEREF->ATTACHMENT is initial or LSRVCONFNOTEREF->COMMENTS is initial.
              loop at LSRVRESDOCS_FULL reference into data(LSRVRESDOCS_FULLREF)
                where ServiceUUID = LSERVICEREF->ServiceUUID and ConfirmationNoteUUID = LCONFNOTEREF->ConfirmationNoteUUID.

                  LSRVCONFNOTEREF->ATTACHMENT = LSRVRESDOCS_FULLREF->Attachment.
                if LSRVRESDOCS_FULLREF->Comments is not initial.
                  LSRVCONFNOTEREF->COMMENTS = |{ LSRVCONFNOTEREF->COMMENTS } { LSRVRESDOCS_FULLREF->Comments }|.
                endif.
*                  LSRVCONFNOTEREF->ATTACHMENTUUID = LSRVRESDOCS_FULLREF->DocumentResultUUID. "???
                  LSRVCONFNOTEREF->FILENAME = LSRVRESDOCS_FULLREF->FileName.
                  LSRVCONFNOTEREF->MIMETYPE = LSRVRESDOCS_FULLREF->MimeType.
                  delete LSRVRESDOCS_FULL.
              endloop.
            endif.


          endloop.

          loop at LSERVATTA reference into data(LSRVATTACHMENTREF) using key ENTITY where ServiceUUID = LSERVICEREF->ServiceUUID .
            insert initial line into table LSERVICEREF2->ATTACHMENTS reference into data(LSRVATTATREF).
            LSRVATTATREF->* = corresponding #( LSRVATTACHMENTREF->* ).
          endloop.

          loop at LSRVRESDOCS_FULL reference into LSRVRESDOCS_FULLREF where ServiceUUID = LSERVICEREF->ServiceUUID.
            insert initial line into table LSERVICEREF2->ATTACHMENTS reference into LSRVATTATREF.
            LSRVATTATREF->* = corresponding #( LSRVRESDOCS_FULLREF->* mapping ATTACHMENTUUID = DocumentResultUUID ).
          endloop.


          loop at LADDRESULTS reference into data(LADDRESULTREF)
*                using key ID
*            where ( TOURUUID = LTOURASSIGNMENTREF->TOURUUID or TOURUUID = '2431' ) and SERVICEUUID = LSERVICEREF->SERVICEUUID
            where ( TOURUUID = LTOURASSIGNMENTREF->TOURUUID or TOURUUID = LCONST_TOURUUID ) and SERVICEUUID = LSERVICEREF->SERVICEUUID
            and RESULTTYPE = /PLCE/CL_PD_CONSTANTS=>C_TASK_CATEGORY-ADDITIONAL_SERVICE.

            insert initial line into table LSERVICEREF2->ADDITIONAL_SERVICES reference into data(LADDITIONAL_SERVICEREF).
            LADDITIONAL_SERVICEREF->* = corresponding #( LADDRESULTREF->* mapping AMOUNT = Value AMOUNTUNIT = ValueUnit PRODUCT = ID ).
          endloop.

        else.
*** nop
        endif.

      endloop.

      if LINES( LTOURRESREF->SERVICES ) is initial and LTOURRESREF->* is not initial.
        delete ET_DATA from LTOURRESREF->* ##INDEX_NUM.
      endif.

    endloop.

*      catch /PLCP/CX_EEWA_BASE into LEX.
*        LEX->GET_AS_BAPIRET( changing PAR_BAPIRET2 = LBAPIRET ).
**        insert LBAPIRET into table CT_BAPIRET.
*    endtry.

  endmethod.


  method READ_SERVICE_DATA_SEQUENCE.

    data:
      LEX         type ref to /PLCP/CX_EEWA_BASE,
      LBAPIRET    type BAPIRET2,
      LDATE       type DATS,
      LTIME       type TIMS,
      LCONFNOTES  type standard table of /PLCE/R_PDConfNote,
      LTOURRESREF type ref to /PLCE/SWR_TOUR_RSLT.

     read entities of /PLCE/R_PDTour
      entity ServiceAssignment
*        all fields with corresponding #( IT_KEYS mapping ServiceUUID = SERVICE_UUID TourUUID = TOUR_UUID )
        fields ( TourUUID ServiceUUID TourSequence Removed ) with corresponding #( IT_KEYS mapping ServiceUUID = SERVICE_UUID TourUUID = TOUR_UUID )
        result data(LTOURASSIGNMENTS).

    sort LTOURASSIGNMENTS by ServiceUUID Removed descending.

*** Tour Assignment is mandatory! (ORDER_HEAD)
    if LINES( LTOURASSIGNMENTS ) is initial.
      raise exception type /PLCE/CX_BASEEXCEPTION
        message id '/PLCP/MC_WA' type 'W' number 034 ##NUMBER_OK.
    else.
      read entity /PLCE/R_PDService
        all fields with corresponding #( IT_KEYS mapping ServiceUUID = SERVICE_UUID )
        result data(LSERVICES).

      delete LSERVICES where Profile <> IV_PROFILE.

      read entity /PLCE/R_PDTour
        all fields with corresponding #( LTOURASSIGNMENTS )
        result data(LTOURS).
    endif.

*** Create result data
    loop at LTOURS reference into data(LTOURREF).

      insert initial line into table ET_DATA reference into LTOURRESREF.
      LTOURRESREF->TOURID = LTOURREF->TourId.
      LTOURRESREF->TOURTEMPLATE = LTOURREF->TourTemplate.
      select single ExternalTourTemplateID from /PLCE/R_PDTourTemplate into @LTOURRESREF->ExternalTourTemplateID
        where TourTemplate = @LTOURREF->TourTemplate.

**** get order date
      if LTOURRESREF->TOURDATE is initial.
        convert date LTOURREF->StartDate into time stamp LTOURRESREF->TOURDATE time zone SY-ZONLO.
      endif.

**** Service zu WDOI
      loop at LTOURASSIGNMENTS reference into data(LTOURASSIGNMENTREF) using key ENTITY where TourUUID = LTOURREF->TOURUUID.
        read table LSERVICES reference into data(LSERVICEREF) with table key ENTITY components ServiceUUID = LTOURASSIGNMENTREF->ServiceUUID.

        if SY-SUBRC is initial.

          read table LTOURRESREF->SERVICES with key SERVICEUUID = LSERVICEREF->ServiceUUID reference into data(LSERVICERESREF).
          if SY-SUBRC is not initial.
***   new
            insert initial line into table LTOURRESREF->SERVICES reference into LSERVICERESREF.
            LSERVICERESREF->* = corresponding #( LSERVICEREF->* ).
            LSERVICERESREF->TOURSEQUENCE = LTOURASSIGNMENTREF->TourSequence.
            LSERVICERESREF->TOURASSIGN_REMOVED = LTOURASSIGNMENTREF->Removed.
          else. " existing
          endif.

        else.
*** nop
        endif.

      endloop.

      if LINES( LTOURRESREF->SERVICES ) is initial and LTOURRESREF->* is not initial.
        delete ET_DATA from LTOURRESREF->* ##INDEX_NUM.
      endif.

    endloop.

  endmethod.


  method READ_TOUR_DATA_CONFIRM.

    data:
      LDATE       type DATS,
      LTIME       type TIMS,
      LCONFNOTES  type standard table of /PLCE/R_PDConfNote,
      LTOURRESREF type ref to /PLCE/SWR_TOUR_RSLT.

    if LINES( IT_KEYS ) is not initial.
      select TourUUID, ResourceId  from /PLCE/R_PDMTour
        for all entries in @IT_KEYS
        where TourUUID = @IT_KEYS-TOUR_UUID
        into table @data(LT_KEYS).

      if LINES( LT_KEYS ) is not initial.

        read entity /PLCE/R_PDMTour
          all fields with corresponding #( LT_KEYS )
          result data(LPDMTOURS)
          by \_ConfirmationNotes all fields with corresponding #( LT_KEYS )
          result data(LCONFNOTESRSLT)
*      by \_ServiceResults all fields with corresponding #( LT_KEYS )
*      result data(LSR)
          by \_Activities all fields with corresponding #( LT_KEYS )
          result data(LACTIVITYRESULTS)
*      by \_AdditionalResults all fields with corresponding #( IT_KEYS mapping TourUUID = TOUR_UUID )
*      result data(LADDRESULTS)
          by \_Tour all fields with corresponding #( LT_KEYS )
          result data(LTOURS)
          by \_Weighings all fields with corresponding #( LT_KEYS )
          result data(LWEIGHRESULTS)
          by \_DocumentResults all fields with corresponding #( LT_KEYS )
          result data(LPDMDOCRES).


        read entity /PLCE/R_MDResource
          all fields with corresponding #( LACTIVITYRESULTS )
          result data(LMDRESSOURCES).

        read entity /PLCE/R_PDTour
          by \_Attachments all fields with corresponding #( LPDMTOURS )
          result data(LTOURATTACHMENTS).

        if lines( LPDMDOCRES ) is not initial.
          delete LPDMDOCRES where ServiceUUID is not initial.
          LTOURATTACHMENTS = corresponding #( base ( LTOURATTACHMENTS ) LPDMDOCRES
            mapping
              Attachment = Attachment
              AttachmentUUID = DocumentResultUUID
              Comments = Comments
              Filename = FileName
              MimeType = MimeType
              ).
        endif.


        select ConfirmationNote, ExternalId from /PLCE/R_PDConfNote into table @LCONFNOTES.

        loop at LTOURS reference into data(LTOURREF).
          insert initial line into table ET_DATA reference into LTOURRESREF.
          LTOURRESREF->TOURID = LTOURREF->TOURID.
          LTOURRESREF->ADDITIONALTEXT = LTOURREF->ADDITIONALTEXT.
          LTOURRESREF->TOURTEMPLATE = LTOURREF->TourTemplate.
          select single ExternalTourTemplateID from /PLCE/R_PDTourTemplate into @LTOURRESREF->ExternalTourTemplateID
            where TourTemplate = @LTOURREF->TourTemplate.

**** get order date
          loop at LACTIVITYRESULTS reference into data(LACTREF) using key ID where TourUUID = LTOURREF->TOURUUID and SERVICEUUID is initial and TASKUUID is initial.
            insert initial line into table LTOURRESREF->ACTIVITIES reference into data(LACTIVITYREF).

            convert time stamp LACTREF->ACTIVITYEND time zone SY-ZONLO
              into date LDATE time LTIME.

            LACTIVITYREF->* = corresponding #( LACTREF->*  ).
            LACTIVITYREF->ACTIVITYDATE = LDATE.

*** todo mehrere Tage auf einer Tour!!!
            LTOURRESREF->TOURDATE = LACTREF->ACTIVITYEND.

            read table LMDRESSOURCES reference into data(LMDRESSOURCEREF) with table key ENTITY components ResourceId = LACTREF->ResourceId.
            if SY-SUBRC is initial.
              LACTIVITYREF->RESSOURCETYPE = LMDRESSOURCEREF->SemanticContext.
              LACTIVITYREF->EQUIPMENT = LMDRESSOURCEREF->Equipment.
              LACTIVITYREF->PARTNER = LMDRESSOURCEREF->Partner.
*        LACTIVITYREF->PERSON =

              if LACTIVITYREF->PARTNER is not initial.
                select single cast( BPIdentificationNumber as numc( 8 ) ) from I_BPUsrExternalID
                  into @LACTIVITYREF->PERSON
                  where BusinessPartner = @LACTIVITYREF->PARTNER
                  and BPIdentificationType = 'HCM001'.
              endif.
            endif.
          endloop.

          if LTOURRESREF->TOURDATE is initial.
*      LTOURRESREF->TOURDATE = LTOURREF->StartDate.
            convert date LTOURREF->StartDate into time stamp LTOURRESREF->TOURDATE time zone SY-ZONLO.
          endif.

**** Kopfwiegenoten
          loop at LWEIGHRESULTS reference into data(LWEIGHREF) using key ID where TourUUID = LTOURREF->TOURUUID and ServiceUUID is initial and TaskUUID is initial.
            insert initial line into table LTOURRESREF->WEIGHINGS reference into data(LTOURWEIGHREF).
            LTOURWEIGHREF->* =  corresponding #( LWEIGHREF->* ).
            select single VEH~VEHICLELICENSEPLATE from /PLCE/R_MDVehicle as VEH
              inner join /PLCE/R_MDResource as RES
                on RES~Equipment = VEH~Vehicle
              where RES~ResourceId = @LWEIGHREF->ResourceId
              into @LTOURWEIGHREF->VEHICLELICENSEPLATE.

          endloop.

*** Confirmation notes
*** Additional Result
*** Attachment von Tour und Confirmation Note

          loop at LCONFNOTESRSLT reference into data(LCONFNOTEREF) using key ID where TourUUID = LTOURREF->TOURUUID and ServiceUUID is initial. " and TaskUUID is initial.
            insert initial line into table LTOURRESREF->CONFNOTES reference into data(LTOURCONFNOTEREF).
            LTOURCONFNOTEREF->* = corresponding #( LCONFNOTEREF->* mapping ATTACHMENTUUID = ConfirmationNoteUUID ).
            read table LCONFNOTES reference into data(LC) with key ConfirmationNote = LCONFNOTEREF->ConfirmationNote.
            if SY-SUBRC is initial.
              LTOURCONFNOTEREF->EXTERNALID = LC->ExternalId.
            endif.
          endloop.

          loop at LTOURATTACHMENTS using key Entity reference into data(LATTACHMENTREF) where TourUUID = LTOURREF->TOURUUID.
            insert initial line into table LTOURRESREF->ATTACHMENTS reference into data(LTOURATTACHMENTREF).
            LTOURATTACHMENTREF->* = corresponding #( LATTACHMENTREF->* ).
          endloop.

        endloop.

      endif.
    endif.

  endmethod.


  method SEQUENCE_SERVICES.

    data:
      LTOURS_RES  type /PLCE/PWR_TOUR_RSLT.

    READ_SERVICE_DATA_SEQUENCE(
      exporting
        IV_PROFILE = IV_PROFILE               " Planning and Dispatching Profile
        IT_KEYS    = IT_KEYS                  " Table type for /PLCE/SPDTOUR_KEY
      importing
        ET_DATA    = LTOURS_RES               " WR Tour Results
    ).


    WRITE_DATA_TO_EEWA(
      exporting
        IT_TOUR_DATA = LTOURS_RES                 " WR Tour Results
*        IV_SEQUENCE_ONLY = 'X'
        IS_SEQUENCE_CONTROL = IS_SEQUENCE_CONTROL
      changing
        CT_BAPIRET = CT_BAPIRET
    ).

  endmethod.


  method WRITE_DATA_TO_EEWA.

    data:
      LCALLBACK type ref to CL_EEWA_IMPL_BO_CALLBACK_MLOG,
      LBAPIRET  type BAPIRET2.

    create object LCALLBACK exporting PAR_INIT = SPACE.
    if LINES( IT_TOUR_DATA ) is not initial.
      LCALLBACK->MSGLOG->HEAD-EXTNUMBER = |{ IT_TOUR_DATA[ 1 ]-TOURID } { IT_TOUR_DATA[ 1 ]-TOURDATE }|.
    endif.
    LCALLBACK->INIT( ).
    LCALLBACK->MSGLOG->SETCUMULATE( PAR_CUMULATE = 'X' ).

    try.

        TA_BEGIN_NAMED_VAR TA /PLCP/CL_TA_WA_ORDER_RSLT=>CTA_ORDER_RESULT.
        TA_DO_COMMIT TA.
*        TA_NO_COMMIT TA.
        TA_DO_RAISE_EXCEPTION TA.
        TA_SET_CALLBACK TA LCALLBACK.
        TA_SET_PARAM TA RESULTDATAS IT_TOUR_DATA.
        TA_SET_PARAM TA USER_ID IS_SEQUENCE_CONTROL-USER_ID.
        if IS_SEQUENCE_CONTROL is initial.
          TA_EXECUTE TA BOOK_IMPORT_RESULTS.
        else.
          if IS_SEQUENCE_CONTROL-SEQUENCE_OPERATIONAL is not initial.
             TA_EXECUTE TA BOOK_IMPORT_ITEM_SEQUENCES.
          endif.
          if IS_SEQUENCE_CONTROL-SEQUENCE_MASTERDATA is not initial.
            TA_EXECUTE TA BOOK_IMPORT_SERVICE_SEQUENCES.
          endif.
        endif.
        TA_END TA.

      catch CX_EEWA_BASE into data(LEX).
        data(LTEXT) = LEX->GET_TEXT( ).
        LEX->GET_AS_BAPIRET(
          changing
            PAR_BAPIRET2    = LBAPIRET
        ).
        insert LBAPIRET into table CT_BAPIRET.
    endtry.

    if LCALLBACK->MSGLOG->EXISTS_LOG( ) is not initial.
      LCALLBACK->MSGLOG->READ_INTO_BAPIRET(
        importing
          PAR_BAPIRET2       = CT_BAPIRET
      ).
      LCALLBACK->MSGLOG->SAVE( ).
    endif.

  endmethod.
ENDCLASS.
