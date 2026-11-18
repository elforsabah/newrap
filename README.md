class /PLCP/CL_TA_WA_ORDER_RSLT definition
  public
  inheriting from CL_EEWA_TRANSACTION
  create public .

public section.

  data:
    ORDERRESULTS        type standard table of /PLCP/SWA_WDO_RSLT .
  data:
    ORDERITEMRESULTS    type standard table of /PLCP/SWA_WDOI_RSLT .
  data:
    ORDERRESSRESULTS    type standard table of /PLCP/SWA_WDORESS_RSLT .
  data:
    ORDERWEIGHRESULTS   type standard table of /PLCP/SWA_WEIGH_RSLT .
  data RESULTDATAS type /PLCE/PWR_TOUR_RSLT .
  data:
    ATTACHMENTRESULTS   type standard table of /PLCP/SWA_ATTACHMENT_RSLT .
  constants CTA_ORDER_RESULT type EWAEL_OBJTRANSACTION value '/PLCP/ORDER_RESULT' ##NO_TEXT.
  data USER_ID type SY-UNAME .

  methods BOOK_IMPORT_SERVICE_SEQUENCES
    raising
      CX_EEWA_BASE .
  methods BOOK_ORDER_ITEMS
    importing
      !PAR_SEQUENCE_ONLY type KENNZX default SPACE
    raising
      CX_EEWA_BASE .
  methods BOOK_ORDER_RESSOURCES
    raising
      CX_EEWA_BASE .
  methods BOOK_ATTACHMENTS
    raising
      CX_EEWA_BASE .
  methods BOOK_ORDER_HEADS
    raising
      CX_EEWA_BASE .
  methods BOOK_WEIGHNOTES
    raising
      CX_EEWA_BASE .
  methods BOOK_IMPORT_ITEM_SEQUENCES
    raising
      CX_EEWA_BASE .
  methods BOOK_IMPORT_RESULTS
    raising
      CX_EEWA_BASE .
  class-methods CL_IS_MOVED_ORDER_ITEM
    importing
      !PAR_POBJNR type J_OBJNR
    returning
      value(PAR_IS) type KENNZX .
protected section.

  types:
    begin of TITEMORDERKEY,
      POBJNR                 type J_OBJNR.
      include type /PLCP/SWA_WDOH_KEY.
    types end of TITEMORDERKEY .
  types:
    TITEMORDERKEYS type standard table of TITEMORDERKEY .
  types:
    begin of TITEMMAP,
      SRC type TITEMORDERKEY,
      DST type TITEMORDERKEY,
    end of TITEMMAP .
  types:
    TITEMMAPS type standard table of TITEMMAP .
  types:
    TORDERNRS type standard table of EORDERNR .
  types:
    begin of TORDERPOSMAP,
      POBJNR type J_OBJNR.
      include type EWA_ORDER_OBJECT_IKEY as SRC renaming with suffix _S.
      include type EWA_ORDER_OBJECT_IKEY as DST renaming with suffix _N.
      types end of TORDERPOSMAP .
  types:
    TORDERPOSMAPS type standard table of TORDERPOSMAP .
  types:
    TWDOIDDATAS type standard table of EWA_ORDER_OBJECT .

  data:
    FORDER_KEYS      type standard table of /PLCP/SWA_WDOH_KEY .
  data:
    FEWATDEVLOCS     type standard table of EWATDEVLOC
      with non-unique sorted key SS_ORDER components ORDERNR LAUFNR
      with non-unique sorted key SS_POBJNR components POBJNR .
  data:
    FEWATDEVLOCS_INS type standard table of EWATDEVLOC .
  data:
    FEWATDEVLOCS_DEL type standard table of EWATDEVLOC .
  data FWDOIDEFAULTS type TWDOIDDATAS .
  data FCHANGED_DATAS type TWDOIDDATAS .

  methods GET_CALLBACK_EWAORDER
    importing
      !PAR_EXTNUMBER type BALNREXT
    returning
      value(PAR_CALLBACK) type ref to CL_EEWA_IMPL_BO_CALLBACK_MLOG .
  methods EXECUTE_DATA_UPLOAD
    raising
      CX_EEWA_BASE .
  methods HANDLE_CHANGED_ITEM_DATA
    raising
      CX_EEWA_BASE .
  methods GET_CHANGE_FIELDS
    returning
      value(PAR_FIELDNAMES) type EWAPDDICFIELDNAMES .
  methods CREATE_NOTIFICATION
    importing
      !PAR_USER_ID type SY-UNAME optional
      !PAR_NOTIFKEY type /IWNGW/IF_NOTIF_PROVIDER=>TY_S_NOTIFICATION_TYPE_ID-TYPE_KEY
      !PAR_TOUR_ID type /PLCE/PDTOUR_ID .
  methods BOOK_SERVICE_SEQUENCES
    raising
      CX_EEWA_BASE .
  methods DO_ADDITIONAL_SERVICES
    importing
      !PAR_BO_ORDERHEAD type ref to CL_EEWA_BO_WDORDER
      !PAR_ITEMREF type ref to EWASWDORDERITEMDET
      !PAR_ADDITIONAL_SERVICES type /PLCP/PWA_WDOIADDSRV_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_ORDER_ITEMS
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_ITEM_ATTA
    importing
      !PAR_DATAREF type ref to /PLCE/PWR_SERVICE_ATTA_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
      !PAR_ITEMREF type ref to /PLCP/SWA_WDOI_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_ITEMS
    importing
      !PAR_DATAREF type ref to /PLCE/PWR_SERVICE_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_ITEM
    importing
      !PAR_DATAREF type ref to /PLCE/SWR_SERVICE_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
    exporting
      !PAR_ITEMREF type ref to /PLCP/SWA_WDOI_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_HEAD_WGH
    importing
      !PAR_DATAREF type ref to /PLCE/PWR_WEIGH_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_HEAD_CN
    importing
      !PAR_DATAREF type ref to /PLCE/PWR_CONFNOTE_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_ITEM_WGH
    importing
      !PAR_DATAREF type ref to /PLCE/PWR_WEIGH_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
      !PAR_ITEMREF type ref to /PLCP/SWA_WDOI_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_HEAD_ATTA
    importing
      !PAR_DATAREF type ref to /PLCE/PWR_TOUR_ATTA_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_ITEM_SPEC
    importing
      !PAR_DATAREF type ref to /PLCE/SWR_SERVICE_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
      !PAR_ITEMREF type ref to /PLCP/SWA_WDOI_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_HEAD_ACT
    importing
      !PAR_DATAREF type ref to /PLCE/PWR_ACTIVITY_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_ITEM_AS
    importing
      !PAR_DATAREF type ref to /PLCE/PWR_SERVICE_ADD_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
      !PAR_ITEMREF type ref to /PLCP/SWA_WDOI_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_ITEM_CN
    importing
      !PAR_DATAREF type ref to /PLCE/PWR_CONFNOTE_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
      !PAR_ITEMREF type ref to /PLCP/SWA_WDOI_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_HEAD
    importing
      !PAR_DATAREF type ref to /PLCE/SWR_TOUR_RSLT
    exporting
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
    raising
      CX_EEWA_BASE .
  methods GET_DOC_TYPE
    importing
      !PAR_MIMETYPE type W3CONTTYPE
    returning
      value(PAR_DOCTYPE) type SO_OBJ_TP
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA
    importing
      !PAR_DATAREF type ref to /PLCE/SWR_TOUR_RSLT
    raising
      CX_EEWA_BASE .
  methods PREPARE_RESULT_DATA_HEAD_SPEC
    importing
      !PAR_DATAREF type ref to /PLCE/SWR_TOUR_RSLT
      !PAR_HEADREF type ref to /PLCP/SWA_WDO_RSLT
    raising
      CX_EEWA_BASE .
  methods ADJUST_ORDERWEIGHS
    importing
      !PAR_ORDERPOSMAPS type TORDERPOSMAPS
    raising
      CX_EEWA_BASE .
  methods ADJUST_WEIGHPROCS
    importing
      !PAR_ORDERPOSMAPS type TORDERPOSMAPS
    raising
      CX_EEWA_BASE .
  methods MOVE_ITEM_DETAILS
    importing
      !PAR_SOURCE type ref to CL_EEWA_BO_WDORDERITEM
      !PAR_TARGET type ref to CL_EEWA_BO_WDORDERITEM
    raising
      CX_EEWA_BASE .
  methods READ_EWATDEVLOCS
    importing
      !PAR_ITEM_MAPS type TITEMMAPS
    raising
      CX_EEWA_BASE .
  methods RELEASE_POSITIONS
    importing
      !PAR_TARGET type ref to CL_EEWA_BO_WDORDER
      !PAR_SOURCE type ref to CL_EEWA_BO_WDORDER
      !PAR_POSINDICES type STANDARD TABLE
    exporting
      !PAR_ORDERPOSMAPS type TORDERPOSMAPS
    raising
      CX_EEWA_BASE .
  methods SAVE_ORDERS
    importing
      !PAR_ORDERNRS type TORDERNRS
    raising
      CX_EEWA_BASE .
  methods POSITION_FIND
    importing
      !PAR_BO type ref to CL_EEWA_BO_WDORDER
      !PAR_ITEM type TITEMORDERKEY
    returning
      value(PAR_DETAILINDEX) type INT4
    raising
      CX_EEWA_BASE .
  methods CREATE_ORDER_HEADS
    raising
      CX_EEWA_BASE .
  methods CREATE_WEIGHNOTES
    raising
      CX_EEWA_BASE .
  methods BOOK_MOVE_ITEMS
    exporting
      !PAR_ORDERPOSMAPS type TORDERPOSMAPS
    changing
      !PAR_ITEM_MAPS type TITEMMAPS
    raising
      CX_EEWA_BASE .
  methods CREATE_ORDER_HEAD
    importing
      !PAR_KEY type /PLCP/SWA_WDOH_KEY
    returning
      value(PAR_BO) type ref to CL_EEWA_BO_WDORDER
    raising
      CX_EEWA_BASE .
  methods GET_ORDER_BO
    importing
      !PAR_KEY type EORDERNR optional
      !PAR_LOCK type KENNZX default SPACE
    returning
      value(PAR_BO) type ref to CL_EEWA_BO_WDORDER
    raising
      CX_EEWA_BASE .

  methods CALL_METHOD
    redefinition .
private section.
ENDCLASS.



CLASS /PLCP/CL_TA_WA_ORDER_RSLT IMPLEMENTATION.


  method ADJUST_ORDERWEIGHS.

    data:
      LBO_NEW          type ref to CL_EEWA_BO_WDORDERWEIGH,
      LBO_OLD          type ref to CL_EEWA_BO_WDORDERWEIGH,
      LKEY             type TORDERPOSMAP,
      LORDERWEIGHS     type standard table of EWA_ORDER_WEIGH,
      LORDERWEIGH      type EWA_ORDER_WEIGH,
      LORDER_WEIGH_KEY type EWA_ORDER_WEIGH-KEY.


    if LINES( PAR_ORDERPOSMAPS ) is not initial.

      LBO_OLD ?= CL_EEWA_BO_OBJTYPE=>CL_CREATE_BO( PAR_OBJTYPE = CL_EEWA_BO_WDORDERWEIGH=>COT_WDORDERWEIGH ).
      LBO_OLD->INIT_DATA( ).
      LBO_NEW ?= CL_EEWA_BO_OBJTYPE=>CL_CREATE_BO( PAR_OBJTYPE = CL_EEWA_BO_WDORDERWEIGH=>COT_WDORDERWEIGH ).
      LBO_NEW->INIT_DATA( ).
      loop at PAR_ORDERPOSMAPS into LKEY.
        if LKEY-SRC <> LKEY-DST.
* todo OR Klasse
          clear: LORDERWEIGHS.

* todo OR Klasse
          select * into table LORDERWEIGHS
          from EWA_ORDER_WEIGH
          where POBJNR = LKEY-POBJNR
            and ( STORNO = 'X'  or  PSTNG_DATE is null or PSTNG_DATE = '00000000' ).
          loop at LORDERWEIGHS into LORDERWEIGH.
            LBO_OLD->LOAD_DATA( PAR_KEY = LORDERWEIGH-KEY PAR_LOCK = 'X' ).
            try.
                clear LORDER_WEIGH_KEY.
                LORDER_WEIGH_KEY-ORDERNR = LKEY-DST-ORDERNR.
                LBO_NEW->NEW_DATA( PAR_KEY = LORDER_WEIGH_KEY PAR_LOCK = 'X' ).
                try.
                    LORDER_WEIGH_KEY = LBO_NEW->DATAREF->KEY.  " save key
                    LBO_NEW->DATAREF->* = LBO_OLD->DATAREF->*. " no DATA-Substructure :(
                    LBO_NEW->DATAREF->KEY = LORDER_WEIGH_KEY.  " restore key
                    LBO_OLD->DELETE_DATA( ).
                    LBO_NEW->INSERT_DATA( ).
                  cleanup.
                    LBO_NEW->UNLOCK_FOR_EDIT( ).
                endtry.
                LBO_NEW->UNLOCK_FOR_EDIT( ).
              cleanup.
                LBO_OLD->UNLOCK_FOR_EDIT( ).
            endtry.
            LBO_OLD->UNLOCK_FOR_EDIT( ).
          endloop.
        endif.
      endloop.


    endif.
  endmethod.


  method ADJUST_WEIGHPROCS.

    data:
      LKEY        type TORDERPOSMAP,
      LBO         type ref to CL_EEWA_BO_WEIGHINGPROCESS,
      LBO_ORDER   type ref to CL_EEWA_BO_WDORDER,
      LPOSREF     type ref to EWASWDORDERITEMDET,
      LWEIGHPROCS type standard table of EWA_WA_WEIGHPROC,
      LWEIGHPROC  type EWA_WA_WEIGHPROC.

    if LINES( PAR_ORDERPOSMAPS ) is not initial.

      LBO ?= CL_EEWA_BO_OBJTYPE=>CL_CREATE_BO( PAR_OBJTYPE = CL_EEWA_BO_WEIGHINGPROCESS=>COT_WEIGHINGPROCESS ).
      LBO->INIT_DATA( ).
      loop at PAR_ORDERPOSMAPS into LKEY.
        clear:
          LWEIGHPROCS.

        if LKEY-SRC <> LKEY-DST.

* todo OR Klasse oder WEIGHINGNR into table
          select * into table LWEIGHPROCS from EWA_WA_WEIGHPROC
           where POBJNR = LKEY-POBJNR
             and ( STORNO = 'X'  or  BUCHDATUM is null or BUCHDATUM = '00000000' ). "#EC CI_NOFIELD
          loop at LWEIGHPROCS into LWEIGHPROC.
            LBO->LOAD_DATA( PAR_KEY = LWEIGHPROC-WEIGHINGNR PAR_LOCK = 'X' ).
            try.
                LBO->DATAREF->ORDERNR = LKEY-DST-ORDERNR.
                LBO->CHECK_ORDER( ).
                LBO->SAVE_DATA( ).
              cleanup.
                LBO->UNLOCK_FOR_EDIT( ).
            endtry.
            LBO->UNLOCK_FOR_EDIT( ).
          endloop.
        endif.
      endloop.

    endif.

  endmethod.


  method BOOK_ATTACHMENTS.

    data:
      LBO_DOC         type ref to /PLCP/CL_BO_SO_DOCUMENT,
      LBO             type ref to CL_EEWA_BO_BASE,
      LEXT            type STRING,
      LDB_ATTACHMENTS type standard table of /PLCP/TWRATTA.


    loop at ATTACHMENTRESULTS reference into data(LATTACHMENTREF).

      TA_GETBO_VAR LBO_DOC /PLCP/CL_BO_SO_DOCUMENT=>COT_SO_DOCUMENT SPACE.
      LBO_DOC->INIT_DATA( ).

*ATTACHMENTUUID
*COMMENTS
*ATTACHMENT
*MIMETYPE
*FILENAME
*** Check for existing.

      select single UUID from /PLCP/TWRATTA into @data(LDOCID_EXIST)
        where UUID = @LATTACHMENTREF->DATA->ATTACHMENTUUID.

      if SY-SUBRC is not initial.
        if LATTACHMENTREF->DATA->FILENAME is not initial.
          LEXT = TO_UPPER( CL_FS_PATH=>CREATE( LATTACHMENTREF->DATA->FILENAME )->GET_FILE_EXTENSION( ) ).
        else.
          LEXT = GET_DOC_TYPE( PAR_MIMETYPE = LATTACHMENTREF->DATA->MIMETYPE ).
        endif.
        shift LEXT left deleting leading '.'.

        if LEXT is not initial and STRLEN( LEXT ) = 3.

          LBO_DOC->NEW_DATA( ).

          LBO_DOC->CREATE_DOCUMENT(
            PAR_DOC_NAME = 'EXT'
            PAR_DOC_DESC = conv #( LATTACHMENTREF->DATA->COMMENTS )
            PAR_DATA = LATTACHMENTREF->DATA->ATTACHMENT
*        PAR_DOC_TYPE = LDOC_TYPE        "Von Mimetype raten
            PAR_DOC_TYPE = conv #( LEXT )
            PAR_HEX = ABAP_TRUE
          ).

          LBO_DOC->SET_FILENAME( LATTACHMENTREF->DATA->FILENAME ).
          LBO_DOC->INSERT_DATA( ).

          insert value #( UUID = LATTACHMENTREF->DATA->ATTACHMENTUUID DOC_ID = LBO_DOC->DATAREF->DOC_ID ) into table LDB_ATTACHMENTS.


          if LATTACHMENTREF->TOURUUID is not initial.
            read table FORDER_KEYS
              with key
                /PLCP/PD_TOUR_ID = LATTACHMENTREF->/PLCP/PD_TOUR_ID
                ORDER_DATE = LATTACHMENTREF->ORDER_DATE
                ROUTE = LATTACHMENTREF->ROUTE reference into data(LKEYREF) binary search.

            if SY-SUBRC is initial and LKEYREF->ORDERNR is not initial.

              TA_GETBO_VAR LBO CL_EEWA_BO_WDORDER=>COT_WDORDER LKEYREF->ORDERNR.

            else.
              raise exception type /PLCP/CX_EEWA_BASE
                message id '/PLCP/MC_WA'
                type 'E'
                number 031 ##NUMBER_OK
                with LATTACHMENTREF->/PLCP/PD_TOUR_ID.
            endif.

          else.
            select single ORDERNR, ORDER_LAUFNR from EWA_ORDER_OBJECT where POBJNR = @LATTACHMENTREF->POBJNR into @data(LITEMKEY).
            TA_GETBO_VAR LBO CL_EEWA_BO_WDORDERITEM=>COT_WDORDERITEM LITEMKEY.
          endif.

          LBO_DOC->BOOK_LINK_DOCUMENT(
            PAR_MAIN_OBJ_KEY = LBO->GET_GOS_BOR_OBJKEY( )
            PAR_MAIN_OBJ_TYPE = LBO->GET_GOS_BOR_OBJTYPE( )
          ).

          LBO_DOC->SAVE_DATA( ).

        else.
          if LEXT is initial.
            raise exception type /PLCP/CX_EEWA_BASE
              message id '/PLCP/MC_WA'
              type 'E'
              number 032.
          else.
            raise exception type /PLCP/CX_EEWA_BASE
              message id '/PLCP/MC_WA'
              type 'E'
              number 033 ##NUMBER_OK.
          endif.

        endif.
      endif.

    endloop.

    if LINES( LDB_ATTACHMENTS ) is not initial.
      insert /PLCP/TWRATTA from table LDB_ATTACHMENTS.
    endif.

  endmethod.


  method BOOK_IMPORT_ITEM_SEQUENCES.

    define _COMMIT.
      if DO_COMMIT is not initial.
        COMMIT_WORK( PAR_KIND = 'N').
      endif.
    end-of-definition.



    if LINES( RESULTDATAS ) is not initial.

      loop at RESULTDATAS reference into data(LRESULTREF).
        try.
            clear:
              ORDERRESULTS,
              ORDERRESSRESULTS,
              ORDERITEMRESULTS,
              ORDERWEIGHRESULTS.

            PREPARE_RESULT_DATA( LRESULTREF ).
*            _COMMIT.
            BOOK_ORDER_HEADS( ).
            _COMMIT.
*            BOOK_ORDER_RESSOURCES( ).
*            _COMMIT.
            BOOK_ORDER_ITEMS( PAR_SEQUENCE_ONLY = 'X' ).
            _COMMIT.
*            BOOK_WEIGHNOTES( ).
*            _COMMIT.
*            BOOK_ATTACHMENTS( ).
*            _COMMIT.

          catch CX_EEWA_BASE into EXCEPTION.
            if DO_COMMIT is not initial.
              ROLLBACK_WORK( ).
              CALLBACK_REPORT( PAR_KIND = EXCEPTION->MSGTYPE PAR_EXCEPTION = EXCEPTION ).
            else.
              raise exception EXCEPTION.
            endif.
        endtry.

      endloop.

    endif.

  endmethod.


  method BOOK_IMPORT_RESULTS.

    define _COMMIT.
      if DO_COMMIT is not initial.
        COMMIT_WORK( PAR_KIND = 'N').
      endif.
    end-of-definition.



    if LINES( RESULTDATAS ) is not initial.

      loop at RESULTDATAS reference into data(LRESULTREF).
        try.
            clear:
              ORDERRESULTS,
              ORDERRESSRESULTS,
              ORDERITEMRESULTS,
              ORDERWEIGHRESULTS.

            PREPARE_RESULT_DATA( LRESULTREF ).
*            _COMMIT.
            BOOK_ORDER_HEADS( ).
            _COMMIT.
            BOOK_ORDER_RESSOURCES( ).
            _COMMIT.
            BOOK_ORDER_ITEMS( ).
            _COMMIT.
            BOOK_WEIGHNOTES( ).
            _COMMIT.
            BOOK_ATTACHMENTS( ).
            _COMMIT.

          catch CX_EEWA_BASE into EXCEPTION.
            if DO_COMMIT is not initial.
              ROLLBACK_WORK( ).
              CALLBACK_REPORT( PAR_KIND = EXCEPTION->MSGTYPE PAR_EXCEPTION = EXCEPTION ).
            else.
              raise exception EXCEPTION.
            endif.
        endtry.

      endloop.

    endif.

  endmethod.


  method BOOK_IMPORT_SERVICE_SEQUENCES.

    define _COMMIT.
      if DO_COMMIT is not initial.
        COMMIT_WORK( PAR_KIND = 'N').
      endif.
    end-of-definition.

    if LINES( RESULTDATAS ) is not initial.


      loop at RESULTDATAS reference into data(LRESULTREF).
        try.
            clear:
              ORDERRESULTS,
              ORDERRESSRESULTS,
              ORDERITEMRESULTS,
              ORDERWEIGHRESULTS.

            PREPARE_RESULT_DATA( LRESULTREF ).

            BOOK_SERVICE_SEQUENCES( ).
            _COMMIT.
            CREATE_NOTIFICATION( PAR_USER_ID = USER_ID PAR_NOTIFKEY = 'WASTE_SERVICE_SAVE_SUCCESS' PAR_TOUR_ID = lresultref->TOURID ).
            _COMMIT.

          catch CX_EEWA_BASE into EXCEPTION.

            if DO_COMMIT is not initial.
              ROLLBACK_WORK( ).
              CREATE_NOTIFICATION( PAR_USER_ID = USER_ID PAR_NOTIFKEY = 'WASTE_SERVICE_SAVE_FAILED' PAR_TOUR_ID = lresultref->TOURID ).
              CALLBACK_REPORT( PAR_KIND = EXCEPTION->MSGTYPE PAR_EXCEPTION = EXCEPTION ).
            else.
              raise exception EXCEPTION.
            endif.
        endtry.

      endloop.

    endif.

  endmethod.


  method BOOK_MOVE_ITEMS.

    types:
      begin of TSORTDATA,
*      DIFFERENT type CHAR1,
        ORDERNR_TGT type EORDERNR,
        ORDERNR_SRC type EORDERNR,
        SOURCE      type TITEMORDERKEY,
      end of TSORTDATA.


*    sort PAR_ITEM_MAPS by DST-ORDERNR.

*** aquire targets
*** aquire sources per target
***


    data:
      LTARGETORDERS  type TORDERNRS,
      LTARGETORDER   type EORDERNR,
      LSAVEORDERS    type TORDERNRS,
      LMAPITEMREF    type ref to TITEMMAP,
      LSORTDATAREF   type ref to TSORTDATA,
      LSORTDATA      type TSORTDATA,
      LSORTDATAS     type standard table of TSORTDATA,
      LBO_TARGET     type ref to CL_EEWA_BO_WDORDER,
      LBO_SOURCE     type ref to CL_EEWA_BO_WDORDER,
      LDETAILINDEX   type I,
      LDETAILINDICES type standard table of I,
      LORDERPOSMAPS  type TORDERPOSMAPS,
      LPOBJNRS       type BO_T_OBJNR.


    try.

        READ_EWATDEVLOCS( PAR_ITEM_MAPS ).

        loop at PAR_ITEM_MAPS reference into LMAPITEMREF.
          insert LMAPITEMREF->DST-ORDERNR into table LTARGETORDERS.
          insert initial line into table LSORTDATAS reference into LSORTDATAREF.
          LSORTDATAREF->SOURCE = LMAPITEMREF->SRC.
          LSORTDATAREF->ORDERNR_SRC = LMAPITEMREF->SRC-ORDERNR.
          LSORTDATAREF->ORDERNR_TGT = LMAPITEMREF->DST-ORDERNR.
          insert LMAPITEMREF->SRC-POBJNR into table LPOBJNRS.
        endloop.

        sort LTARGETORDERS.
        delete adjacent duplicates from LTARGETORDERS.

        sort LSORTDATAS by ORDERNR_TGT ORDERNR_SRC." SOURCE-ROUTE_SEQUENCE.

        sort LPOBJNRS.
        export POBJNRS = LPOBJNRS to memory id 'WASTEORDERITEM_SORTED_POBJNRS_DISPO'. " this objects are in disposition

        loop at LTARGETORDERS into LTARGETORDER.
          clear:
            LSAVEORDERS,
            LDETAILINDICES,
            LDETAILINDEX,
            FEWATDEVLOCS_DEL,
            FEWATDEVLOCS_INS.

*      try.
          BEGIN_UPDATE( ).
          try.
*              LBO_TARGET ?= GETBO( PAR_OBJTYPE = CL_EEWA_BO_WDORDER=>COT_WDORDER PAR_KEY = LTARGETORDER PAR_LOCK = 'X' PAR_DETAILS = 'ITEM:L,RES' ).
              LBO_TARGET ?= GET_ORDER_BO( PAR_KEY = LTARGETORDER PAR_LOCK = 'X' ).
*

              loop at LSORTDATAS reference into LSORTDATAREF where ORDERNR_TGT = LTARGETORDER.
                if LBO_SOURCE is initial or LBO_SOURCE->DATAREF->ORDERNR <> LSORTDATAREF->SOURCE-ORDERNR.
*                  LBO_SOURCE ?= GETBO( PAR_OBJTYPE = CL_EEWA_BO_WDORDER=>COT_WDORDER PAR_KEY = LSORTDATAREF->SOURCE-ORDERNR PAR_LOCK = 'X' PAR_DETAILS = 'ITEM:L,RES' ).
                  LBO_SOURCE ?= GET_ORDER_BO( PAR_KEY = LSORTDATAREF->SOURCE-ORDERNR PAR_LOCK = 'X' ).
                endif.

                LDETAILINDEX = POSITION_FIND( PAR_BO = LBO_SOURCE PAR_ITEM = LSORTDATAREF->SOURCE ).

                insert LDETAILINDEX into table LDETAILINDICES.

                at end of ORDERNR_SRC ##LOOP_AT_OK.
                  RELEASE_POSITIONS(
                    exporting
                      PAR_TARGET      = LBO_TARGET
                      PAR_SOURCE      = LBO_SOURCE
*                      PAR_TARGETINDEX = LTARGETINDEX
                      PAR_POSINDICES  = LDETAILINDICES
*                    changing
                    importing
                      PAR_ORDERPOSMAPS = LORDERPOSMAPS
                  ).
                  clear LDETAILINDICES.

                  LBO_SOURCE->CHECK_DATA( ).
                  insert LBO_SOURCE->DATAREF->ORDERNR into table LSAVEORDERS.

                endat.

              endloop.

              LBO_TARGET->CHECK_DATA( ).
              insert LBO_TARGET->DATAREF->ORDERNR into table LSAVEORDERS.

              " target should be saved last
              if LBO_TARGET is not initial and LINES( LSAVEORDERS ) > 1.
                read table LSAVEORDERS transporting no fields with key TABLE_LINE = LBO_TARGET->DATAREF->ORDERNR.
                if SY-TABIX <> LINES( LSAVEORDERS ).
                  delete LSAVEORDERS index SY-TABIX.
                  append LBO_TARGET->DATAREF->ORDERNR to LSAVEORDERS.
                endif.
              endif.

              SAVE_ORDERS( LSAVEORDERS ).

            cleanup.
              clear LORDERPOSMAPS.
              END_UPDATE( PAR_FAILED = 'X' ).
          endtry.
          END_UPDATE( ).

*** have to be done after END_UPDATE
          ADJUST_WEIGHPROCS( PAR_ORDERPOSMAPS = LORDERPOSMAPS ).
          ADJUST_ORDERWEIGHS( PAR_ORDERPOSMAPS = LORDERPOSMAPS ).

          insert lines of LORDERPOSMAPS into table PAR_ORDERPOSMAPS.

*        catch CX_EEWA_BASE into EXCEPTION.
*
*          CALLBACK_REPORT(
*            PAR_KIND      = EXCEPTION->MSGTYPE
*            PAR_EXCEPTION = EXCEPTION
*          ).
*
*      endtry.

        endloop.

      cleanup.
        delete from memory id 'WASTEORDERITEM_SORTED_POBJNRS_DISPO'.
    endtry.
    delete from memory id 'WASTEORDERITEM_SORTED_POBJNRS_DISPO'.

  endmethod.


  method BOOK_ORDER_HEADS.

    data:
      LBO   type ref to CL_EEWA_BO_WDORDER,
      LTEXT type STRING.


    CREATE_ORDER_HEADS( ).

*    BEGIN_UPDATE( ).

*    try.
    sort FORDER_KEYS by /PLCP/PD_TOUR_ID ORDER_DATE ROUTE.

    loop at ORDERRESULTS reference into data(LORDERREF).

      read table FORDER_KEYS
        with key
          /PLCP/PD_TOUR_ID = LORDERREF->/PLCP/PD_TOUR_ID
          ORDER_DATE = LORDERREF->ORDER_DATE
          ROUTE = LORDERREF->ROUTE reference into data(LKEYREF) binary search.

      if SY-SUBRC is initial.
*        TA_GETBO_LOCK LBO CL_EEWA_BO_WDORDER=>COT_WDORDER LORDERREF->ORDERNR.
        try.
*            LBO ?= GETBO( PAR_OBJTYPE = CL_EEWA_BO_WDORDER=>COT_WDORDER PAR_KEY = LKEYREF->ORDERNR PAR_LOCK = 'X' ).
            LBO ?= GET_ORDER_BO( PAR_KEY = LKEYREF->ORDERNR PAR_LOCK = 'X' ).

            try.
                LBO->DATAREF->/PLCP/SWDO_RSLT_ENH = corresponding #( LORDERREF->* ).
                LBO->DATAREF->ORDERTXT = LORDERREF->ORDERTXT.
                loop at LORDERREF->CONFNOTES reference into data(LCONFNOTEREF).
                  if LBO->DATAREF->/PLCP/CONFIRMATION_NOTE_1 is initial.
                    LBO->DATAREF->/PLCP/CONFIRMATION_NOTE_1 = LCONFNOTEREF->CONFTYPE.
                    LBO->DATAREF->/PLCP/CONF_NOTE_COMMENT = LBO->DATAREF->/PLCP/CONF_NOTE_COMMENT && ` ` && LCONFNOTEREF->CONFNOTE_COMMENT.
                  elseif LBO->DATAREF->/PLCP/CONFIRMATION_NOTE_2 is initial.
                    LBO->DATAREF->/PLCP/CONFIRMATION_NOTE_2 = LCONFNOTEREF->CONFTYPE.
                    LBO->DATAREF->/PLCP/CONF_NOTE_COMMENT = LBO->DATAREF->/PLCP/CONF_NOTE_COMMENT && ` ` && LCONFNOTEREF->CONFNOTE_COMMENT.
                  elseif LBO->DATAREF->/PLCP/CONFIRMATION_NOTE_3 is initial.
                    LBO->DATAREF->/PLCP/CONFIRMATION_NOTE_3 = LCONFNOTEREF->CONFTYPE.
                    LBO->DATAREF->/PLCP/CONF_NOTE_COMMENT = LBO->DATAREF->/PLCP/CONF_NOTE_COMMENT && ` ` && LCONFNOTEREF->CONFNOTE_COMMENT.
                  endif.
                endloop.

                TA_BO_SAVE LBO.
              cleanup.
                TA_BO_UNLOCK LBO.
            endtry.
            TA_BO_UNLOCK LBO.

          catch CX_EEWA_BASE into EXCEPTION.
            TA_LOG_EXCEPTION EXCEPTION.
        endtry.
      else.
        LTEXT = |{ LORDERREF->/PLCP/PD_TOUR_ID alpha = out }|.
        create object EXCEPTION type /PLCP/CX_EEWA_BASE
          exporting
            TEXTID = /PLCP/CX_EEWA_BASE=>NO_CORRESPONDING_ORDER
            TEXT   = CONDENSE( LTEXT ).
        TA_LOG_EXCEPTION EXCEPTION.
      endif.

    endloop.

*      cleanup.
*        END_UPDATE( PAR_FAILED = 'X' ).
*    endtry.
*
*    END_UPDATE( ).

  endmethod.


  method BOOK_ORDER_ITEMS.


* &1 Detailname
* &2 Dataref
* &3 Where clause
    define BO_LOOP_DETAIL_REF_WHERE.
      if LLPDPATH&1 is not initial.
        loop at <LLPDITEMS&1> reference into LLPDREF&1 where (&3).
          assign (LLPDPATH&1) to <LLPDITEM&1>.
          get reference of <LLPDITEM&1> into &2.
    end-of-definition.



    data:
      LITEMKEYS_ACT type standard table of TITEMORDERKEY,
      LITEMKEYREF   type ref to TITEMORDERKEY,
      LORDERNRS     type standard table of EORDERNR,
      LBO           type ref to cl_eewa_BO_WDORDER,
      LBO_ITEM      type ref to CL_EEWA_BO_WDORDERITEM,
      LCALLBACK_ORDER type ref to CL_EEWA_IMPL_BO_CALLBACK_MLOG,
*      LCALLBACK_ORDER type ref to IF_EEWA_BO_CALLBACK_BASE,
      LITEMREF        type ref to EWASWDORDERITEMDET,
      LSERVICECAT   type SERVICE_CAT.


    if LINES( ORDERITEMRESULTS ) is not initial.

**** Get Current keys.
**** Determine Items to move
**** Move Items
**** Fill Result data

*** Move items
      PREPARE_ORDER_ITEMS( ).

*** D&D is done. Targets exists.
*      clear LITEMKEYS_ACT.

      select * into corresponding fields of table LITEMKEYS_ACT from EWA_ORDER_OBJECT
        inner join EWA_ORDER_HEAD on EWA_ORDER_OBJECT~ORDERNR = EWA_ORDER_HEAD~ORDERNR
        for all entries in ORDERITEMRESULTS
        where POBJNR = ORDERITEMRESULTS-POBJNR
          and EWA_ORDER_HEAD~ORDER_DATE = ORDERITEMRESULTS-ORDER_DATE
          and /PLCP/PD_TOUR_ID = ORDERITEMRESULTS-/PLCP/PD_TOUR_ID
          and ROUTE = ORDERITEMRESULTS-ROUTE.
**   todo Status

      LORDERNRS = value #( for L in LITEMKEYS_ACT ( L-ORDERNR ) ).

      sort LORDERNRS.
      delete adjacent duplicates from LORDERNRS.

      loop at LORDERNRS reference into data(LORDERNRREF).
        try.

            if LBO is initial or LBO->DATAREF->ORDERNR <> LORDERNRREF->*.
              LBO ?= GET_ORDER_BO( PAR_KEY = LORDERNRREF->* PAR_LOCK = 'X' ).

              LCALLBACK_ORDER = GET_CALLBACK_EWAORDER( exporting PAR_EXTNUMBER = conv #( LORDERNRREF->* ) ).

              LBO->SET_CALLBACK( PAR_INTERFACE = LCALLBACK_ORDER ).

              LBO->RELOAD_DATA( ).
            endif.


            BO_DATA_LOOP_DETAIL_VAR LBO ITEM.

            try.
                loop at LITEMKEYS_ACT reference into LITEMKEYREF where ORDERNR = LBO->DATAREF->ORDERNR.

                  BO_LOOP_DETAIL_REF_WHERE ITEM LITEMREF 'POBJNR = LITEMKEYREF->POBJNR'.

                  FWDOIDEFAULTS = value #( base FWDOIDEFAULTS ( corresponding #( LITEMREF->* ) ) ).

                  loop at ORDERITEMRESULTS reference into data(LRESULTREF) where POBJNR = LITEMKEYREF->POBJNR.


                    if PAR_SEQUENCE_ONLY is initial.
**** Move results
                      LSERVICECAT = CL_EEWA_BO_SERVICETYPE=>CL_GET_SERVICE_CAT( PAR_SERVICE_TYPE = LITEMREF->SERVICE_TYPE ).

*CSC_EMPTY
*CSC_OTHERS
*CSC_REMOVE
*CSC_SWITCH
*CSC_TRANSPORT

                      case LSERVICECAT.
                        when CL_EEWA_BO_SERVICETYPE=>CSC_TRANSPORT.

                          LITEMREF->BEHTYP_RM = LRESULTREF->BEH_TYPE_NEW.
                          LITEMREF->BEH_ANZAHL_RM = LRESULTREF->BEH_ANZAHL_NEW.
                          LITEMREF->/PLCP/TIDNR_RM = LRESULTREF->TIDNR_NEW.
                          if LRESULTREF->TIDNR_NEW is not initial.
                            select single SERNR from EQUI as EQ inner join EQUZ as EZ on EZ~EQUNR = EQ~EQUNR
                                where EQ~MATNR = @LITEMREF->BEHTYP_RM
                                  and ( EZ~DATBI <= @LITEMREF->ORDER_DATE and EZ~DATAB >= @LITEMREF->ORDER_DATE )
                                  into @LITEMREF->SERNR_RM_NEW. "REMSERNR
                          endif.
** SERNR

                        when CL_EEWA_BO_SERVICETYPE=>CSC_SWITCH.

                          LITEMREF->BEHTYP_RM = LRESULTREF->BEH_TYPE_NEW.
                          LITEMREF->BEH_ANZAHL_RM = LRESULTREF->BEH_ANZAHL_NEW.
                          LITEMREF->/PLCP/TIDNR_NEW_RM = LRESULTREF->TIDNR_NEW.
                          LITEMREF->/PLCP/TIDNR_RM = LRESULTREF->TIDNR.

                          if LRESULTREF->TIDNR_NEW is not initial.
                            select single SERNR from EQUI as EQ inner join EQUZ as EZ on EZ~EQUNR = EQ~EQUNR
                                where EQ~MATNR = @LITEMREF->BEHTYP_RM
                                  and ( EZ~DATBI <= @LITEMREF->ORDER_DATE and EZ~DATAB >= @LITEMREF->ORDER_DATE )
                                  into @LITEMREF->SERNR_RM_NEW.
                          endif.

                        when others.

                          LITEMREF->BEHTYP_RM = LRESULTREF->BEH_TYPE.
                          LITEMREF->BEH_ANZAHL_RM = LRESULTREF->BEH_ANZAHL.
                          LITEMREF->BEH_TYPE_NEW = LRESULTREF->BEH_TYPE_NEW.
                          if LRESULTREF->BEH_ANZAHL_NEW is not initial.
                            LITEMREF->BEH_ANZAHL_RM = LRESULTREF->BEH_ANZAHL_NEW.
                          endif.
                          if LRESULTREF->TIDNR_NEW is not initial.
                            LITEMREF->/PLCP/TIDNR_NEW_RM = LRESULTREF->TIDNR_NEW.
                            select single SERNR from EQUI as EQ inner join EQUZ as EZ on EZ~EQUNR = EQ~EQUNR
                              where EQ~MATNR = @LITEMREF->BEH_TYPE_NEW
                                and ( EZ~DATBI <= @LITEMREF->ORDER_DATE and EZ~DATAB >= @LITEMREF->ORDER_DATE )
                                into @LITEMREF->SERNR_RM_NEW.
                          endif.
                          if LRESULTREF->TIDNR is not initial.
                            LITEMREF->/PLCP/TIDNR_RM = LRESULTREF->TIDNR.
                          endif.

                      endcase.

*                    LITEMREF->WASTE = LRESULTREF->WASTE_TYPE.
                      LITEMREF->WASTE_TYPE = LRESULTREF->WASTE_TYPE.
                      LITEMREF->WASTE = LRESULTREF->WASTE_TYPE.
                      LITEMREF->WASTE_WEIGHT = LRESULTREF->WASTE_WEIGHT.
                      LITEMREF->WEIGHT_UNIT = LRESULTREF->WEIGHT_UNIT.

                      LBO_ITEM ?= LBO->GETDETAILBO(
                        exporting
                          PAR_NAME        = 'ITEM'
                          PAR_DETAILINDEX = LITEMREF->DETAIL_INDEX
                      ).
                      if LITEMREF->WASTE_WEIGHT is not initial.
                        LBO_ITEM->DATAREF_FILLK->WASTE_WEIGHT = CL_EEWA_ORDER_FILLUP_RESULT=>C_FILLKIND-MANUALLY.
                      else.
                        LBO_ITEM->DATAREF_FILLK->WASTE_WEIGHT = CL_EEWA_ORDER_FILLUP_RESULT=>C_FILLKIND-UNSET.
                      endif.

***** todo SERNR NEW based on TIDNR!

                      loop at LRESULTREF->CONFNOTES reference into data(LCONFNOTEREF).
                        if LITEMREF->CONFTYPE1 is initial.
                          LITEMREF->CONFTYPE1 = LCONFNOTEREF->CONFTYPE.
                          LITEMREF->TEXT = LITEMREF->TEXT && ` ` && LCONFNOTEREF->CONFNOTE_COMMENT.
                        elseif LITEMREF->CONFTYPE2 is initial.
                          LITEMREF->CONFTYPE2 = LCONFNOTEREF->CONFTYPE.
                          LITEMREF->TEXT = LITEMREF->TEXT && ` ` && LCONFNOTEREF->CONFNOTE_COMMENT.
                        elseif LITEMREF->CONFTYPE3 is initial.
                          LITEMREF->CONFTYPE3 = LCONFNOTEREF->CONFTYPE.
                          LITEMREF->TEXT = LITEMREF->TEXT && ` ` && LCONFNOTEREF->CONFNOTE_COMMENT.
                        else.
                          call function 'CONVERSION_EXIT_EWAPO_OUTPUT'
                            exporting
                              INPUT  = LRESULTREF->POBJNR
                            importing
                              OUTPUT = LRESULTREF->POBJNR.

                          create object EXCEPTION type /PLCP/CX_EEWA_BASE
                            exporting
                              TEXTID     = /PLCP/CX_EEWA_BASE=>WDOI_NO_COMMENT_TRANSFERED
                              MSGTYPE    = 'W'
                              T100_MSGID = '/PLCP/MC_BASE'
                              T100_MSGNO = 021
                              T100_MSGV1 = CONDENSE( |{ LRESULTREF->POBJNR }| )
                              T100_MSGV2 = CONDENSE( |{ LCONFNOTEREF->CONFTYPE }| ).

                          LCALLBACK_ORDER->IF_EEWA_PROTOCOL~REPORT(
                            PAR_KIND        = 'W'
                            PAR_EXCEPTION   = EXCEPTION
                          ).

*                          LCALLBACK_ORDER->MSGLOG->ADD(
*                            exporting
*                              PAR_MSGTY = 'W'
*                              PAR_MSGID = '/PLCP/MC_BASE'
*                              PAR_MSGNO = 021
*                              PAR_MSGV1 = LRESULTREF->POBJNR
*                              PAR_MSGV2 = LCONFNOTEREF->CONFTYPE
*                              PAR_MSGV3 = LCONFNOTEREF->CONFNOTE_COMMENT
*                          ).
                        endif.
                      endloop.

                      convert time stamp LRESULTREF->TIMESTAMP time zone SY-ZONLO
                        into time LITEMREF->ACTUAL_TIME.

                    endif.

**** Fields which has to be updated always
                    LITEMREF->/PLCP/PD_SERVICE_UUID = LRESULTREF->/PLCP/PD_SERVICE_UUID.
                    LITEMREF->ROUTE_SEQUENCE = LRESULTREF->ROUTE_SEQUENCE.

                    DO_ADDITIONAL_SERVICES(
                      exporting
                        PAR_BO_ORDERHEAD = LBO
                        PAR_ITEMREF = LITEMREF
                        PAR_ADDITIONAL_SERVICES = LRESULTREF->ADDITIONAL_SERVICES
                      ).

                  endloop.
                  if SY-SUBRC is initial.
                    LBO->ITEM_CHANGED( PAR_DETAILINDEX = LITEMREF->DETAIL_INDEX ).
                  endif.
                  BO_ENDLOOP_DETAIL.

                endloop.

                TA_BO_SAVE LBO.

              cleanup.
                TA_BO_UNLOCK LBO.
            endtry.

            TA_BO_UNLOCK LBO.

          catch CX_EEWA_BASE into EXCEPTION.
            delete FWDOIDEFAULTS where ORDERNR = LORDERNRREF->*.
            if LCALLBACK_ORDER is not initial.
              LCALLBACK_ORDER->IF_EEWA_PROTOCOL~REPORT(
                PAR_KIND        = EXCEPTION->MSGTYPE
                PAR_EXCEPTION   = EXCEPTION
              ).
            else.
              CALLBACK_REPORT(
              PAR_KIND      = EXCEPTION->MSGTYPE
              PAR_EXCEPTION = EXCEPTION
            ).
            endif.
        endtry.
        if LCALLBACK_ORDER->MSGLOG->EXISTS_LOG( ) is not initial.
          LCALLBACK_ORDER->MSGLOG->SAVE( PAR_AUTONOM = 'X' ).
        endif.

      endloop.

      HANDLE_CHANGED_ITEM_DATA( ).

    endif.

  endmethod.


  method BOOK_ORDER_RESSOURCES.

    types:
      begin of _TTIMES,
        ORDERNR type EORDERNR,
        DATE    type DATS,
        START   type TIMS,
        END     type TIMS,
      end of _TTIMES.

    data:
      LBO         type ref to CL_EEWA_BO_WDORDER,
      LDETAILREF  type ref to EWA_ORDER_RESOBJ,
*      LDATE       type DATS,
*      LTIME       type TIMS,
      LTEXT       type STRING,
      LINDEX      type I,
      LDATAREF    type ref to DATA,
      LORDERTIMES type standard table of _TTIMES,
      LORDERTIME  type _TTIMES,
      LORDERNRS   type standard table of EORDERNR,
      LORDERNR    type EORDERNR,
      LSTART      type TIMS,
      LEND        type TIMS.

    if LINES( ORDERRESSRESULTS ) is not initial.

      loop at ORDERRESSRESULTS reference into data(LORDERRESREF).

        clear: LINDEX, LORDERTIME.

        sort FORDER_KEYS by /PLCP/PD_TOUR_ID ORDER_DATE ROUTE.

        read table FORDER_KEYS
          with key
            /PLCP/PD_TOUR_ID = LORDERRESREF->/PLCP/PD_TOUR_ID
            ORDER_DATE = LORDERRESREF->ORDER_DATE
            ROUTE = LORDERRESREF->ROUTE reference into data(LKEYREF) binary search.

        if SY-SUBRC is initial.

          try.

              LBO ?= GET_ORDER_BO( PAR_KEY = LKEYREF->ORDERNR PAR_LOCK = 'X' ).

              try.

                  LORDERTIME-ORDERNR = LKEYREF->ORDERNR.
                  insert LKEYREF->ORDERNR into table LORDERNRS.
                  convert time stamp LORDERRESREF->ACTIVITY_START time zone SY-ZONLO
                      into date LORDERTIME-DATE time LORDERTIME-START.

                  convert time stamp LORDERRESREF->ACTIVITY_END time zone SY-ZONLO
                      into time LORDERTIME-END.


                  BO_DATA_LOOP_DETAIL_VAR LBO RES.
                  BO_LOOP_DETAIL_REF RES LDETAILREF.
                  if LDETAILREF->/PLCP/PD_ACTIVITY_UUID = LORDERRESREF->/PLCP/PD_ACTIVITYUUID.
                    LINDEX = SY-TABIX.
                    exit. "loop
                  endif.

                  BO_ENDLOOP_DETAIL.



                  if LINDEX is initial.
                    if LORDERRESREF->RESSOURCE_TYPE = 'F'.
                      LBO->RES_INSERT(
                        exporting
                          PAR_FLEETEQUNR = LORDERRESREF->EQUNR
                        importing
                          PAR_RES        = LDETAILREF
                      ).

                    elseif LORDERRESREF->RESSOURCE_TYPE = 'P'.
                      LBO->RES_INSERT(
                        exporting
                          PAR_PERNR = LORDERRESREF->PERSON
                        importing
                          PAR_RES        = LDETAILREF
                      ).
                    else.
**              Warning
                    endif.
                  else.
                    LBO->RES_CHANGING(
                      exporting
                        PAR_INDEX = LINDEX
                      importing
                        PAR_ITEM  = LDATAREF
                    ).
                    LDETAILREF ?= LDATAREF.
                  endif.

                  LDETAILREF->/PLCP/PD_ACTIVITY_UUID = LORDERRESREF->/PLCP/PD_ACTIVITYUUID.

                  if TO_UPPER( LORDERRESREF->ACTIVITYTYPE ) <> 'BREAK'.
                    if LORDERRESREF->DURATION_UNIT <> 'MIN'.
                      try.
                          /PLCE/CL_BASE_MISC=>UNIT_CONVERSION_SIMPLE(
                            exporting
                              IV_INPUT    = LORDERRESREF->DURATION
                              IV_UNIT_IN  = LORDERRESREF->DURATION_UNIT
                              IV_UNIT_OUT = 'MIN'
                            importing
                              EV_OUTPUT   = LDETAILREF->IDAUR
                          ).
                          LDETAILREF->IDAURE = 'MIN'.
                        catch /PLCE/CX_BASEEXCEPTION into data(LEX_PLCE).
                          /PLCE/CL_BASE_MISC=>EXCEPTION_TO_SYMSG( IR_EXCEPTION = LEX_PLCE ).
                          raise exception type /PLCP/CX_EEWA_BASE using message.
                      endtry.
                    else.
                      LDETAILREF->IDAUR = LORDERRESREF->DURATION.
                      LDETAILREF->IDAURE = LORDERRESREF->DURATION_UNIT.
                    endif.
                    LDETAILREF->EWA_ORDER_RESOURCE_RESULT-FAHRST = LORDERRESREF->DISTANCE.
                    LDETAILREF->EWA_ORDER_RESOURCE_RESULT-FAHRSTE = LORDERRESREF->DISTANCE_UNIT.
*                    convert time stamp LORDERRESREF->ACTIVITY_START time zone SY-ZONLO
*                      into date LDATE time LTIME.
*
*                    LDETAILREF->EWA_ORDER_RESOURCE_RESULT-ACTUAL_START = LTIME.
                    LDETAILREF->EWA_ORDER_RESOURCE_RESULT-ACTUAL_START = LORDERTIME-START.

*                    convert time stamp LORDERRESREF->ACTIVITY_END time zone SY-ZONLO
*                      into date LDATE time LTIME.
*
*                    LDETAILREF->EWA_ORDER_RESOURCE_RESULT-ACTUAL_END = LTIME.
                    LDETAILREF->EWA_ORDER_RESOURCE_RESULT-ACTUAL_END = LORDERTIME-END.

                  else.
                    LDETAILREF->PAUSE = LORDERRESREF->DURATION. " UNIT?
                  endif.


                  if LINDEX is not initial.
                    LBO->RES_CHANGED( PAR_INDEX = LINDEX ).
                  endif.

                  TA_BO_SAVE LBO.
                  if LORDERTIME-START is not initial and LORDERTIME-END is not initial.
                    insert LORDERTIME into table LORDERTIMES.
                  endif.
                cleanup.
                  TA_BO_UNLOCK LBO.
              endtry.
              TA_BO_UNLOCK LBO.

            catch CX_EEWA_BASE into EXCEPTION.
              TA_LOG_EXCEPTION EXCEPTION.
          endtry.
        else.
*          LTEXT = |Tour { LORDERRESREF->/PLCP/PD_TOUR_ID }: No corresponding Order exist!|.
          LTEXT = |{ LORDERRESREF->/PLCP/PD_TOUR_ID alpha = out }|.
*          condense LTEXT.
          create object EXCEPTION type /PLCP/CX_EEWA_BASE
            exporting
              TEXTID = /PLCP/CX_EEWA_BASE=>NO_CORRESPONDING_ORDER
              TEXT   = CONDENSE( LTEXT ).
          TA_LOG_EXCEPTION EXCEPTION.
        endif.

      endloop.


**** Order duration
      sort LORDERNRS.
      delete adjacent duplicates from LORDERNRS.
      delete LORDERNRS where TABLE_LINE is initial.

      loop at LORDERNRS into LORDERNR.
        clear:
          LSTART, LEND.
        LBO ?= GET_ORDER_BO( PAR_KEY = LORDERNR PAR_LOCK = 'X' ).

        try.
            try.

                loop at LORDERTIMES into LORDERTIME where ORDERNR = LORDERNR and DATE = LBO->DATAREF->ORDER_DATE.

                  if LSTART > LORDERTIME-START or LSTART is initial.
                    LSTART = LORDERTIME-START.
                  endif.

                  if LEND < LORDERTIME-END or LEND is initial.
                    LEND = LORDERTIME-END.
                  endif.

                endloop.

                LBO->DATAREF->ACTUAL_START = LSTART.
                LBO->DATAREF->ACTUAL_END = LEND.

                TA_BO_SAVE LBO.
              cleanup.
                TA_BO_UNLOCK LBO.
            endtry.
            TA_BO_UNLOCK LBO.

          catch CX_EEWA_BASE into EXCEPTION.
            TA_LOG_EXCEPTION EXCEPTION.
        endtry.

      endloop.

    endif.

  endmethod.


  method BOOK_SERVICE_SEQUENCES.

    types:
      begin of _TORDER,
        SERVLOC       type SERVLOC,
        SERVFREQNR    type EOBJNR,
        SERVFREQ_LFNR type LAUFNR,
        POBJNR        type J_OBJNR,
        ORDERNR       type EORDERNR,
        ORDER_LAUFNR  type ORDER_LAUFNR,
        TOUR          type /PLCP/PDTOUR_ID,
        TOUR_SEQUENCE type ROUTE_SEQUENCE,
      end of _TORDER.

    data:
      LDATAREF type ref to _TORDER,
      LDATAS   type standard table of _TORDER,
      LBO      type ref to CL_EEWA_BO_SERVFREQHEAD,
*      LITEMREF type ref to EWASSERVFREQITEM_DET,
      LBO_ITEM type ref to CL_EEWA_BO_SERVFREQITEM,
      LBO_TS   type ref to CL_EEWA_BO_SERVFREQTIMESLICE.

    if ( ORDERITEMRESULTS[] is not initial ).
      select * into corresponding fields of table LDATAS from EWA_ORDER_OBJECT
*        inner join EWA_ORDER_HEAD on EWA_ORDER_OBJECT~ORDERNR = EWA_ORDER_HEAD~ORDERNR
        for all entries in ORDERITEMRESULTS
        where POBJNR = ORDERITEMRESULTS-POBJNR.
*          and EWA_ORDER_HEAD~ORDER_DATE = ORDERITEMRESULTS-ORDER_DATE
*          and /PLCP/PD_TOUR_ID = ORDERITEMRESULTS-/PLCP/PD_TOUR_ID
*          and ROUTE = ORDERITEMRESULTS-ROUTE.
    endif.

    loop at LDATAS reference into LDATAREF.
      read table ORDERITEMRESULTS reference into data(LORDERITMRESREF) with key POBJNR = LDATAREF->POBJNR.
      if SY-SUBRC is initial.
        read table ORDERRESULTS reference into data(LORDERRESREF) with key /PLCP/PD_TOUR_ID = LORDERITMRESREF->/PLCP/PD_TOUR_ID.
        if SY-SUBRC is initial.
          LDATAREF->TOUR = LORDERRESREF->ROUTE.
          LDATAREF->TOUR_SEQUENCE = LORDERITMRESREF->ROUTE_SEQUENCE.
        else.
***        should never happen
        endif.
      else.
***        should never happen
      endif.

    endloop.

    sort LDATAS.


    loop at LDATAS reference into LDATAREF.
      at new SERVFREQNR.
*        TA_GETBO_LOCK .
        LBO ?= GETBO( PAR_OBJTYPE = CL_EEWA_BO_SERVFREQHEAD=>COT_SERVFREQHEAD PAR_KEY = LDATAREF->SERVFREQNR PAR_LOCK = 'X' ).
      endat.


      LBO->ITEM_FIND(
        exporting
          PAR_LAUFNR      = LDATAREF->SERVFREQ_LFNR
        importing
*          PAR_DETAILINDEX =                  " Natural Number
          PAR_ITEM        = LBO_ITEM
      ).

      LBO_ITEM->TS_FIND(
        exporting
*          PAR_DATE             =                  " Date
*          PAR_BIS              =                  " Date On Which a Time Slice Expires
          PAR_LAST             = 'X'                 " Find Last Time Slice
*          PAR_FIRST            =                  " Find First Time Slice
*          PAR_GUID             =                  " Link GUID
*          PAR_IGNORE_STARTDATE = ABAP_FALSE
        importing
          PAR_TIMESLICE        = LBO_TS                 " Service Frequency Time Slice
*          PAR_DETAILINDEX      =                  " Natural Number
      ).

      LBO_TS->DATAREF->ROUTE = LDATAREF->TOUR.
      LBO_TS->DATAREF->ROUTE_SEQUENCE = LDATAREF->TOUR_SEQUENCE.

      LBO_TS->CHANGED( ).
      LBO_ITEM->CHANGED( ).
*      catch CX_EEWA_BASE. " Basis Class: All Exceptions for EWA

      at end of SERVFREQNR.
        TA_BO_SAVE_UNLOCK LBO.
      endat.

    endloop.

  endmethod.


  method BOOK_WEIGHNOTES.

    data:
      LBO        type ref to CL_EEWA_BO_WDORDERWEIGH,
      LDETAILREF type ref to EWA_ORDER_RESOBJ,
      LDATE      type DATS,
      LTIME      type TIMS,
      LTEXT      type STRING.

    if LINES( ORDERWEIGHRESULTS ) is not initial.

      LBO ?= CL_EEWA_BO_OBJTYPE=>CL_CREATE_BO( PAR_OBJTYPE = CL_EEWA_BO_WDORDERWEIGH=>COT_WDORDERWEIGH ).
      LBO->INIT_DATA( ).


      sort FORDER_KEYS by /PLCP/PD_TOUR_ID ORDER_DATE ROUTE.

      loop at ORDERWEIGHRESULTS reference into data(LWEIGHREF).

        read table FORDER_KEYS
          with key
            /PLCP/PD_TOUR_ID = LWEIGHREF->/PLCP/PD_TOUR_ID
            ORDER_DATE = LWEIGHREF->ORDER_DATE
            ROUTE = LWEIGHREF->ROUTE reference into data(LKEYREF) binary search.

        if SY-SUBRC is initial.
*        TA_GETBO_LOCK LBO CL_EEWA_BO_WDORDER=>COT_WDORDER LORDERREF->ORDERNR.
          try.
*            LBO ?= GETBO( PAR_OBJTYPE = CL_EEWA_BO_WDORDER=>COT_WDORDER PAR_KEY = LKEYREF->ORDERNR PAR_LOCK = 'X' ).
*              LBO ?= GET_ORDER_BO( PAR_KEY = LKEYREF->ORDERNR PAR_LOCK = 'X' ).

              select count( * ) from EWA_ORDER_WEIGH
                where ORDERNR = LKEYREF->ORDERNR
                 and /PLCP/PD_WEIGHING_UUID = LWEIGHREF->/PLCP/PD_WEIGHING_UUID.

              if SY-DBCNT is initial.
                LBO->CLEAR_DATA( ).
                try.
                    LBO->DATAREF->ORDERNR = LKEYREF->ORDERNR.
                    LBO->NEW_DATA( PAR_LOCK = 'X' ).

                    LBO->DATAREF->POBJNR = LWEIGHREF->POBJNR.
                    LBO->DATAREF->LICENSE_NUM = LWEIGHREF->LICENSE_NUM.
                    LBO->DATAREF->WASTE = LWEIGHREF->WASTE.
                    LBO->DATAREF->WDPLANT = LWEIGHREF->WDPLANT.
*                LBO->DATAREF->EXT_ID = LWEIGHREF->EXT_ID.
                    LBO->DATAREF->TARE_WDATE = LWEIGHREF->ORDER_DATE.
                    LBO->DATAREF->GROSS_WDATE = LBO->DATAREF->TARE_WDATE.
*                LBO->DATAREF->TARE_WTIME = LWEIGHREF->TARE_WTIME.
*                LBO->DATAREF->GROSS_WTIME = LWEIGHREF->GROSS_WTIME.
*                  convert time stamp LORDERRESREF->ACTIVITY_START time zone SY-ZONLO
*                    into date LDATE time LTIME.
*** todo CHECK UNIT.
                    LBO->DATAREF->WEIGHT_UNIT = LWEIGHREF->WEIGHT_UNIT.
                    if LWEIGHREF->NET_WEIGHT is not initial.
                      LBO->DATAREF->NET_WEIGHT = LWEIGHREF->NET_WEIGHT.
                    endif.
                    if LWEIGHREF->GROSS_WEIGHT is not initial.
                      LBO->DATAREF->GROSS_WEIGHT = LWEIGHREF->GROSS_WEIGHT.
                    endif.
                    if LWEIGHREF->TARE_WEIGHT is not initial.
                      LBO->DATAREF->TARE_WEIGHT = LWEIGHREF->TARE_WEIGHT.
                    endif.

                    if LWEIGHREF->QUANTITY is not initial.
                      LBO->DATAREF->QUANTITY = LWEIGHREF->QUANTITY.
                    endif.

                    if LWEIGHREF->QUANTITY_UNIT is not initial.
                      LBO->DATAREF->QUANTITY_UNIT = LWEIGHREF->QUANTITY_UNIT.
                    endif.

                    LBO->DATAREF->/PLCP/PD_WEIGHING_UUID = LWEIGHREF->/PLCP/PD_WEIGHING_UUID.
                    LBO->DATAREF->EXT_ID = LWEIGHREF->EXT_ID.

                    LBO->INSERT_DATA( ).

                  cleanup.
                    LBO->UNLOCK_FOR_EDIT( ).
                endtry.
                LBO->UNLOCK_FOR_EDIT( ).
              endif.

            catch CX_EEWA_BASE into EXCEPTION.
              TA_LOG_EXCEPTION EXCEPTION.
          endtry.
        else.
          LTEXT = |{ LWEIGHREF->/PLCP/PD_TOUR_ID alpha = out }|.
          create object EXCEPTION type /PLCP/CX_EEWA_BASE
            exporting
              TEXTID = /PLCP/CX_EEWA_BASE=>NO_CORRESPONDING_ORDER
              TEXT   = CONDENSE( LTEXT ).
          TA_LOG_EXCEPTION EXCEPTION.
        endif.

      endloop.

    endif.

  endmethod.


  method CALL_METHOD.
    TA_CALL_METHOD.
  endmethod.


  method CL_IS_MOVED_ORDER_ITEM.

    data:
      LPOBJNRS_DISPO type BO_T_OBJNR.

    clear: PAR_IS.

    import POBJNRS = LPOBJNRS_DISPO from memory id 'WASTEORDERITEM_SORTED_POBJNRS_DISPO'.

    read table LPOBJNRS_DISPO with key TABLE_LINE = PAR_POBJNR transporting no fields binary search.
    if SY-SUBRC is initial.
      PAR_IS = 'X'.
    endif.

  endmethod.


  method CREATE_NOTIFICATION.

    data:
      LTNOTIFICATIONS         type /PLCE/IF_NOTIFICATION_API=>TY_T_NOTIFICATION,
      LNOTIFICATION_PROCESSOR type ref to /PLCE/CL_BGP_PDNOTIFICATION,
      LUUID                   type SYSUUID_X16,
      LLOGHANDLE              type BALLOGHNDL,
      LLOG_EEWA               type ref to CL_EEWA_IMPL_BO_CALLBACK_MLOG,
      LLOG_PLCE               type ref to /PLCE/CL_APPLLOG_HELPER.

    if PAR_USER_ID is not initial.

****      Log handle
      if FCALLBACK is not initial and FCALLBACK is instance of CL_EEWA_IMPL_BO_CALLBACK_MLOG.
        LLOG_EEWA ?= FCALLBACK.
        LLOGHANDLE = LLOG_EEWA->MSGLOG->HANDLE.
        create object LLOG_PLCE exporting IV_LOG_HANDLE = LLOGHANDLE.
      endif.

      try.
          LUUID = CL_UUID_FACTORY=>CREATE_SYSTEM_UUID( )->CREATE_UUID_X16( ).

          insert value #(
            ID = LUUID
            TYPE_KEY = PAR_NOTIFKEY
            TYPE_VERSION = '0.0.5'
            PRIORITY = /PLCE/IF_NOTIFICATION_API=>GCS_PRIORITIES-NEUTRAL
            RECIPIENTS = value #( ( ID = PAR_USER_ID ) )
            PARAMETERS = value #( ( LANGUAGE = SY-LANGU PARAMETERS = value #( ( NAME = 'TourId' VALUE = |{ PAR_TOUR_ID alpha = out }| )  ) ) )
          )
          into table LTNOTIFICATIONS.

          create object LNOTIFICATION_PROCESSOR.
          LNOTIFICATION_PROCESSOR->NOTIFICATIONS = LTNOTIFICATIONS.
          /PLCE/CL_BGPROCESSING_HELPER=>SCHEDULE( I_BGUNIT = LNOTIFICATION_PROCESSOR ).

        catch CX_UUID_ERROR.
**        nop
        catch /plce/cx_bgp into data(lex_bgp).
        if llog_plce is not initial.
            LLOG_PLCE->ADD_EXCEPTION(
              IV_SEVERITY   = IF_BALI_CONSTANTS=>C_SEVERITY_WARNING
              IRO_EXCEPTION = lex_bgp
            ).
        else.
**        nop
        endif.
*            create object EXCEPTION.
*            EXCEPTION->PUT_SYMSG( ).

      endtry.
    endif.

  endmethod.


  method CREATE_ORDER_HEAD.


*    LBO_HEAD ?= FBOCACHE->GETBO( PAR_OBJTYPE = CL_EEWA_BO_WDORDER=>COT_WDORDER PAR_DETAILS = 'ITEM,RES' PAR_LOCK = SPACE ).
*    PAR_BO ?= GETBO( PAR_OBJTYPE = CL_EEWA_BO_WDORDER=>COT_WDORDER PAR_LOCK = SPACE PAR_DETAILS = 'ITEM,RES' ).
    PAR_BO ?= GET_ORDER_BO( ).
    PAR_BO->NEW_DATA( PAR_LOCK = 'X' ).
    try.

        PAR_BO->BOOK_CHANGEORDERDATE( PAR_ORDERDATE = PAR_KEY-ORDER_DATE ).
        if PAR_KEY-ROUTE is not initial.
          PAR_BO->BOOK_CHANGEROUTE( PAR_ROUTE = PAR_KEY-ROUTE ). " PAR_ADAPTRESOURCES = SPACE ??
          PAR_BO->DATAREF->ACTUAL_ROUTE = PAR_KEY-ROUTE.
        endif.
        if PAR_KEY-/PLCP/PD_TOUR_ID is not initial.
          PAR_BO->DATAREF->/PLCP/PD_TOUR_ID = PAR_KEY-/PLCP/PD_TOUR_ID.
        endif.

        TA_BO_SAVE PAR_BO.

      cleanup.
        TA_BO_UNLOCK PAR_BO.
    endtry.

    TA_BO_UNLOCK PAR_BO.

  endmethod.


  method CREATE_ORDER_HEADS.

*** todo... Modularize for customer  enhancements.

***  Input
    types:
      begin of _TINPUT,
        TOURID     type /PLCP/PDTOUR_ID,
        ORDER_DATE type EORDER_DATE,
        ROUTE      type EROUTE,
      end of _TINPUT.

    data:
*      LEXISTING_ORDERS type standard table of _TINPUT,
*      LINPUT           type standard table of /PLCP/SWA_WDOH_KEY, " _TINPUT,
      LRESULTREF    type ref to /PLCP/SWA_WDO_RSLT,
      LORDERCREATES type standard table of /PLCP/SWA_WDOH_KEY, "/PLCP/SWA_WDO_RSLT,
      LORDERREF     type ref to /PLCP/SWA_WDOH_KEY,
      LBO_HEAD      type ref to CL_EEWA_BO_WDORDER,
      LORDER_KEYS   type standard table of /PLCP/SWA_WDOH_KEY.


    clear:
      FORDER_KEYS.

    if LINES( ORDERRESULTS ) is not initial.

*      AQUIRE_EXISTING_ORDERS( ).
      select ORDERNR, ORDER_DATE, ROUTE, /PLCP/PD_TOUR_ID from EWA_ORDER_HEAD into corresponding fields of table @FORDER_KEYS
        for all entries in @ORDERRESULTS
        where ORDER_DATE = @ORDERRESULTS-ORDER_DATE
          and ROUTE = @ORDERRESULTS-ROUTE
          and /PLCP/PD_TOUR_ID = @ORDERRESULTS-/PLCP/PD_TOUR_ID
        and not exists ( select OBJNR from JEST
                       where OBJNR = EWA_ORDER_HEAD~HOBJNR
                         and STAT = @CL_EEWA_CONSTANTS_CONFIRMATION=>CWDOCNFSTATUS_HEAD-CONFIRMED_POSITIVE
                         and INACT = @SPACE ). "#EC CI_NOFIELD


      if SY-SUBRC is initial. " Search for missing entries
        sort FORDER_KEYS by /PLCP/PD_TOUR_ID ORDER_DATE ROUTE.

        loop at ORDERRESULTS reference into LRESULTREF.
          read table FORDER_KEYS
            with key
              /PLCP/PD_TOUR_ID = LRESULTREF->/PLCP/PD_TOUR_ID
              ORDER_DATE = LRESULTREF->ORDER_DATE
              ROUTE = LRESULTREF->ROUTE transporting no fields binary search.

          if SY-SUBRC is not initial.
            insert LRESULTREF->/PLCP/SWA_WDOH_KEY into table LORDERCREATES.
          endif.
        endloop.
      else. " all entries have to be created

        LORDERCREATES = corresponding #( ORDERRESULTS ).

      endif.

    endif.

    if LINES( LORDERCREATES ) is not initial.
***  Create orders.
      sort LORDERCREATES.
      delete adjacent duplicates from LORDERCREATES.

      BEGIN_UPDATE( ).
      try.
          loop at LORDERCREATES reference into LORDERREF.

            LBO_HEAD ?= CREATE_ORDER_HEAD( LORDERREF->* ).

            LORDER_KEYS = value #( base LORDER_KEYS (
              ORDERNR = LBO_HEAD->DATAREF->ORDERNR
              ORDER_DATE = LBO_HEAD->DATAREF->ORDER_DATE
              ROUTE = LBO_HEAD->DATAREF->ROUTE
              /PLCP/PD_TOUR_ID = LBO_HEAD->DATAREF->/PLCP/PD_TOUR_ID )
              ).

          endloop.

        cleanup.
          END_UPDATE( PAR_FAILED = 'X' ).
      endtry.

      END_UPDATE( ).

      insert lines of LORDER_KEYS into table FORDER_KEYS.

    endif.

  endmethod.


  method CREATE_WEIGHNOTES.
  endmethod.


  method DO_ADDITIONAL_SERVICES.

** 3 Methods
** 1. Find in DMRPOS
** 2. Find in SDPOS and ADD to DMRPOS
** 3. Insert plane in DMRPOS


    types:
      begin of _TDMRPOSREF,
        LDMRPOSREF type ref to EWASORDERDMRPOS_DET,
      end of _TDMRPOSREF,
      begin of _TVBAPREF,
        LVBAPREF type ref to EWASWDORDERITEM_VBAP,
      end of _TVBAPREF.

    data:
      LDMRPOSREF     type ref to EWASORDERDMRPOS_DET,
      LBO_ITEM       type ref to CL_EEWA_BO_WDORDERITEM,
      LQUANT         type DZMENG,
      LDMRDETAILREF  type ref to _TDMRPOSREF,
      LDMRDETAILREFS type standard table of _TDMRPOSREF,
      LVBAPDETREF    type ref to _TVBAPREF,
      LVBAPDETREFS   type standard table of _TVBAPREF,
      LVBAPREF       type ref to EWASWDORDERITEM_VBAP,
      LEX            type ref to CX_EEWA_BASE.

    if LINES( PAR_ADDITIONAL_SERVICES ) is not initial.
      LBO_ITEM ?= PAR_BO_ORDERHEAD->GETDETAILBO(
        exporting
          PAR_NAME        = 'ITEM'
          PAR_DETAILINDEX = PAR_ITEMREF->DETAIL_INDEX
      ).

      LBO_ITEM->REQUIRE_DETAILS( PAR_DETAILS = 'DMR_POS,VBAP' ).

      BO_DATA_LOOP_DETAIL_VAR LBO_ITEM DMR_POS.
      BO_DATA_LOOP_DETAIL_VAR LBO_ITEM VBAP.

      loop at PAR_ADDITIONAL_SERVICES reference into data(LADDSRVREF).

        try.
            clear:
              LDMRDETAILREFS, LDMRPOSREF, LDMRDETAILREF,
              LVBAPDETREF, LVBAPDETREFS.

            BO_LOOP_DETAIL_REF_WHERE DMR_POS LDMRPOSREF 'MATNR = LADDSRVREF->MATNR'.
*** All details for the material
            LDMRDETAILREFS = value #( base LDMRDETAILREFS ( LDMRPOSREF = LDMRPOSREF ) ).
            BO_ENDLOOP_DETAIL.

            if LINES( LDMRDETAILREFS ) is not initial. "-> already existing --> Add values to existing
              loop at LDMRDETAILREFS reference into LDMRDETAILREF.
                clear: LQUANT.

                if LDMRDETAILREF->LDMRPOSREF->VBELN is not initial and LDMRDETAILREF->LDMRPOSREF->SRC_POSNR is not initial.
                  BO_LOOP_DETAIL_REF_WHERE VBAP LVBAPREF 'VBELN = LDMRDETAILREF->LDMRPOSREF->VBELN and POSNR = LDMRDETAILREF->LDMRPOSREF->SRC_POSNR'.
*** needed for Contshare
                  BO_ENDLOOP_DETAIL.
                else.
                  clear: LVBAPREF.
                endif.

                if LDMRDETAILREF->LDMRPOSREF->ZMENG is not initial and LDMRDETAILREF->LDMRPOSREF->ZIEME <> LADDSRVREF->ZIEME.
                  CL_EEWA_BO_MEASUREMENT=>CL_UNIT_CONVERSION2(
                    exporting
                      PAR_QUANIN  = LADDSRVREF->ZMENG
                      PAR_UNITIN  = LADDSRVREF->ZIEME
                      PAR_UNITOUT = LDMRPOSREF->ZIEME
                    importing
                      PAR_QUANOUT = LQUANT
                  ).
                else.
                  LQUANT = LADDSRVREF->ZMENG.
                endif.

                if LVBAPREF is not initial.
                  LQUANT = LQUANT * LVBAPREF->CONTSHARE.
                endif.

                LDMRDETAILREF->LDMRPOSREF->ZMENG = LDMRDETAILREF->LDMRPOSREF->ZMENG + LQUANT.
                LBO_ITEM->DMR_POS_CHANGED( PAR_DETAILINDEX = LDMRDETAILREF->LDMRPOSREF->DETAIL_INDEX ).
              endloop.
            else.

              clear: LVBAPREF.
              BO_LOOP_DETAIL_REF_WHERE VBAP LVBAPREF 'MATNR = LADDSRVREF->MATNR'.

              LBO_ITEM->DMR_POS_INSERT(
                importing
                  PAR_DETAIL      = LDMRPOSREF
              ).

              LDMRPOSREF->* = corresponding #( base ( LDMRPOSREF->* ) LVBAPREF->* mapping POSNR = UEPOS SRC_POSNR = POSNR except ZMENG POBJNR POS_ORDER ). " das Detail arbeitet nur mit UEPOS!!

              CL_EEWA_EL_SD_INTERFACE=>GET_FOLLOWON_DOCTYPE(
                exporting
                  PAR_VBELN   = LVBAPREF->VBELN                 " Verkaufsbeleg
                  PAR_POSNR   = LVBAPREF->POSNR                 " Verkaufsbelegposition
                importing
                  PAR_POSTYPE = LDMRPOSREF->PSTYV                   " Positionstyp Vertriebsbeleg
              ).

              try .
                  CL_EEWA_BO_MEASUREMENT=>CL_UNIT_CONVERSION2(
                    exporting
                      PAR_QUANIN  = LADDSRVREF->ZMENG
                      PAR_UNITIN  = LADDSRVREF->ZIEME
                      PAR_UNITOUT = LDMRPOSREF->ZIEME
                    importing
                      PAR_QUANOUT = LDMRPOSREF->ZMENG
                  ).

                catch CX_EEWA_BASE.
*              if this is the case, store the values and save only. the confirmation will raise the message and it has to be corrected manually
                  LDMRPOSREF->ZMENG = LADDSRVREF->ZMENG.
                  LDMRPOSREF->ZIEME = LADDSRVREF->ZIEME.
              endtry.

              LDMRPOSREF->ZMENG = LDMRPOSREF->ZMENG * LVBAPREF->CONTSHARE.

              BO_ENDLOOP_DETAIL.

              if LVBAPREF is initial.
                LBO_ITEM->DMR_POS_INSERT(
                  importing
                    PAR_DETAIL      = LDMRPOSREF
                ).
                LDMRPOSREF->DATA = corresponding #( LADDSRVREF->* ).
              endif.

            endif.

          catch CX_EEWA_BASE into LEX.
            CALLBACK_REPORT(
              PAR_KIND      = LEX->MSGTYPE                 " PAR_KIND
              PAR_EXCEPTION = LEX                 " PAR_EXCEPTION
            ).
*            catch CX_EEWA_BASE. " CX_EEWA_BASE
        endtry.
      endloop.
    endif.

  endmethod.


  method EXECUTE_DATA_UPLOAD.

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

          new /PLCP/CL_WA_WDOI_PDSERVICE( LLOGHANDLE )->CREATE_BY_PRINTDATA(
            exporting
              IS_PRINT_DATA = LEXPORTDATA
          ).
        endif.
      endif.
    endif.

  endmethod.


  method GET_CHANGE_FIELDS.

    clear PAR_FIELDNAMES.

    PAR_FIELDNAMES = value #( ( '/WATP/NOTEINTNR' ) ).

  endmethod.


method GET_DOC_TYPE.

  data: LEXTS type standard table of SDOKMIME,
        LEXT  type SDOKMIME.

  "Versucht aus Mimetype die Extension to finden
  "Problem: Diese Extension kann länger als 3 CHARs sein, aber PAR_DOC_TYPE ist nur max. 3 lang

  "Großteil der Anfragen:
  case TO_UPPER( PAR_MIMETYPE ).
    when 'IMAGE/JPEG'.
      PAR_DOCTYPE = 'JPG'.
    when 'IMAGE/JPG'.
      PAR_DOCTYPE = 'JPG'.
    when 'IMAGE/PNG'.
      PAR_DOCTYPE = 'PNG'.
    when 'APPLICATION/PDF'.
      PAR_DOCTYPE = 'PDF'.
  endcase.
  if PAR_DOCTYPE is not initial.
    return.
  endif.

  "Andererseits versuche zu raten:
  select *
    from SDOKMIME
    into table LEXTS
    where SDOKMIME~TYPE = PAR_MIMETYPE order by primary key.

  if SY-SUBRC is not initial.
    PAR_DOCTYPE = 'RAW'. "Kann aber generell nicht geöffnet werden
    return.
  endif.

  loop at LEXTS into LEXT.
    condense LEXT-EXTENSION no-gaps.
    if STRLEN( LEXT-EXTENSION ) > 3.
      delete LEXTS.
    endif.
  endloop.

  "Man kann es auch theoretisch mit der folgenden Methode probieren:
  "(Man erhält für JPEG dann aber auch zB. immer 'JPEG' zurück was länger als 3 Zeichen ist und nicht geht)
  "CL_RSOD_UTILITIES=>MIMETYPE_TO_EXTENSION( I_MIMETYPE = PAR_MIMETYPE ).

  if LINES( LEXTS ) > 0.
    "Damit sollten z.B. PNG, PDF möglich sein
    "Geht nicht: Für Word oder Excel Dokumente gibt es keine passende 3 stellige Extension (heutzutage .docx oder .xlsx)
    "            PC kann '.DOC' wahrscheinlich nicht öffnen
    read table LEXTS index 1 into LEXT. "#EC CI_NOORDER
    PAR_DOCTYPE = conv #( LEXT-EXTENSION ).
  else.
    PAR_DOCTYPE = 'RAW'. "Kann aber generell nicht geöffnet werden
  endif.

endmethod.


  method GET_ORDER_BO.

    PAR_BO ?= GETBO( PAR_OBJTYPE = CL_EEWA_BO_WDORDER=>COT_WDORDER PAR_KEY = PAR_KEY PAR_LOCK = PAR_LOCK PAR_DETAILS = 'ITEM:L,RES' ).

  endmethod.


  method HANDLE_CHANGED_ITEM_DATA.
*** check if specific data is changed -> then reupload service
*** first field is /WATP/NOTEINTNR
    data:
      LFIELDNAMES    type EWAPDDICFIELDNAMES,
      LFIELDNAME     type FIELDNAME,
      LKEYS          type standard table of EWA_ORDER_OBJECT_IKEY,
      LACTUALDATAS   type standard table of EWA_ORDER_OBJECT,
      LCHANGED_DATAS type standard table of EWA_ORDER_OBJECT,
      LCHANGED       type KENNZX.

    field-symbols:
      <FIELD_SRC> type DATA,
      <FIELD_DST> type DATA.



    LFIELDNAMES = GET_CHANGE_FIELDS( ).

    if LINES( LFIELDNAMES ) is not initial.

      LKEYS = corresponding #( FWDOIDEFAULTS ).
      sort LKEYS.
      delete adjacent duplicates from LKEYS.

      DATA_BO_RANGE X.
      SET_BO_RANGE_KEYS X __KEYS LKEYS.

      CL_EEWA_OR_WDORDERITEM=>CL_READ(
        exporting
          PAR_OBJTYPE = CL_EEWA_BO_WDORDERITEM=>COT_WDORDERITEM
          PAR_RANGES  = LRANGEXS
        importing
          PAR_TABLE   = LACTUALDATAS
      ).

      sort:
        FWDOIDEFAULTS by KEY,
        LACTUALDATAS by KEY.

      loop at FWDOIDEFAULTS reference into data(LWDOIDEFAULTSREF).
        clear LCHANGED.

        read table LACTUALDATAS reference into data(LACTUALDATASREF)
          with key KEY = LWDOIDEFAULTSREF->KEY binary search.
        if SY-SUBRC is initial.
*** find changed fields -> end afte the first match
          loop at LFIELDNAMES into LFIELDNAME.
            assign component LFIELDNAME of structure LWDOIDEFAULTSREF->* to <FIELD_SRC> else unassign.
            if <FIELD_SRC> is assigned.
              assign component LFIELDNAME of structure LACTUALDATASREF->* to <FIELD_DST> else unassign.
              if <FIELD_DST> is assigned.
                if <FIELD_SRC> <> <FIELD_DST>.
                  LCHANGED = 'X'.
                  exit. "loop
                endif.
              endif.
            endif.

          endloop.

        endif.

        if LCHANGED is not initial.
          insert LACTUALDATASREF->* into table FCHANGED_DATAS.
        endif.

      endloop.

      if LINES( FCHANGED_DATAS ) is not initial.
        EXECUTE_DATA_UPLOAD( ).
      endif.

    endif.

  endmethod.


  method MOVE_ITEM_DETAILS.

    data:
      LBULKYREF     type ref to EWA_ORDER_BULKY,
      LBULKY        type EWA_ORDER_BULKY,
      LDMRPOSREF    type ref to EWASORDERDMRPOS_DET,
      LSUBDMRPOSREF type ref to EWASSUB_WDOISUB_DMRPOS_DET,
      LSUBSDPOREF   type ref to EWAS_SUB_WDOI_PO_DET,
      LWDOITEMSDREF type ref to EWA_ORDER_OBJ_SD,
      LDEVLOCNEW    type EWATDEVLOC.


    if PAR_SOURCE->GETDETAILUSED( PAR_NAME = 'BULKY' ) is not initial.
      BO_DATA_LOOP_DETAIL_VAR PAR_SOURCE BULKY.
      BO_LOOP_DETAIL_REF BULKY LBULKYREF.

      LBULKY = LBULKYREF->*.
      clear: LBULKY-ORDERNR, LBULKY-ORDER_LAUFNR.

      PAR_TARGET->BULKY_INSERT(
        exporting
          PAR_DFLT    = LBULKY ).

      BO_ENDLOOP_DETAIL.

      BO_LOOP_DETAIL_REF BULKY LBULKYREF.
        PAR_SOURCE->BULKY_DELETE( PAR_INDEX = SY-TABIX ).
      BO_ENDLOOP_DETAIL.

    endif.

    if PAR_SOURCE->GETDETAILUSED( PAR_NAME = 'DMR_POS' ) is not initial.

      BO_DATA_LOOP_DETAIL_VAR PAR_SOURCE DMR_POS.
      BO_LOOP_DETAIL_REF DMR_POS LDMRPOSREF.

      PAR_TARGET->DMR_POS_INSERT(
        exporting
          PAR_DEFAULT     = LDMRPOSREF->*
      ).

      BO_ENDLOOP_DETAIL.
      BO_LOOP_DETAIL_REF DMR_POS LDMRPOSREF.
        PAR_SOURCE->DMR_POS_DELETE( PAR_DETAILINDEX = LDMRPOSREF->DETAIL_INDEX ).
      BO_ENDLOOP_DETAIL.

    endif.

    if PAR_SOURCE->GETDETAILUSED( PAR_NAME = 'SUBDMR_POS' ) is not initial.

      BO_DATA_LOOP_DETAIL_VAR PAR_SOURCE SUBDMR_POS.
      BO_LOOP_DETAIL_REF SUBDMR_POS LSUBDMRPOSREF.

      PAR_TARGET->SUBDMR_POS_INSERT(
        exporting
          PAR_DEFAULT     = LSUBDMRPOSREF->*
      ).

      BO_ENDLOOP_DETAIL.

      PAR_SOURCE->SUBDMR_POSITIONS_DELETE( ).

    endif.

    if PAR_SOURCE->GETDETAILUSED( PAR_NAME = 'SUBSD_PO' ) is not initial.

      BO_DATA_LOOP_DETAIL_VAR PAR_SOURCE SUBSD_PO.
      BO_LOOP_DETAIL_REF SUBSD_PO LSUBSDPOREF.

      PAR_TARGET->SUBSD_PO_INSERT(
        exporting
          PAR_DEFAULT     = LSUBSDPOREF->*
      ).

      BO_ENDLOOP_DETAIL.

      PAR_SOURCE->SUBSD_PO_DELETE_ALL( ).

    endif.

    if PAR_SOURCE->GETDETAILUSED( PAR_NAME = 'WDOITEM_SD' ) is not initial.

      BO_DATA_LOOP_DETAIL_VAR PAR_SOURCE WDOITEM_SD.
      BO_LOOP_DETAIL_REF WDOITEM_SD LWDOITEMSDREF.

      PAR_TARGET->WDOITEM_SD_INSERT(
        exporting
*          PAR_VBELN     =                  " Sales Document
*          PAR_POSNR     =                  " Sales document item
*          PAR_CONTSHARE =                  " Container Portion
          PAR_DFLT      = LWDOITEMSDREF->*
      ).

      BO_ENDLOOP_DETAIL.

      BO_LOOP_DETAIL_REF WDOITEM_SD LWDOITEMSDREF.
        PAR_SOURCE->WDOITEM_SD_DELETE( PAR_INDEX = SY-TABIX ).
      BO_ENDLOOP_DETAIL.

    endif.

*** EWATDEVLOC

    read table FEWATDEVLOCS into LDEVLOCNEW with table key SS_ORDER
      components ORDERNR = PAR_SOURCE->DATAREF->ORDERNR LAUFNR = PAR_SOURCE->DATAREF->ORDER_LAUFNR.
    if SY-SUBRC is not initial.
      read table FEWATDEVLOCS into LDEVLOCNEW with table key SS_POBJNR
        components POBJNR = PAR_SOURCE->DATAREF->POBJNR.
    endif.

    if LDEVLOCNEW is not initial.
      insert LDEVLOCNEW into table FEWATDEVLOCS_DEL.
      LDEVLOCNEW-ORDERNR = PAR_TARGET->DATAREF->ORDERNR.
      LDEVLOCNEW-LAUFNR = PAR_TARGET->DATAREF->ORDER_LAUFNR.
      LDEVLOCNEW-POBJNR = PAR_TARGET->DATAREF->POBJNR.
      insert LDEVLOCNEW into table FEWATDEVLOCS_INS.
    endif.

  endmethod.


  method POSITION_FIND.
* &1 Detailname
* &2 Dataref
* &3 Where clause
    define BO_LOOP_DETAIL_REF_WHERE.
      if LLPDPATH&1 is not initial.
        loop at <LLPDITEMS&1> reference into LLPDREF&1 where (&3).
          assign (LLPDPATH&1) to <LLPDITEM&1>.
          get reference of <LLPDITEM&1> into &2.
    end-of-definition.

    data:
      LITEMREF    type ref to EWASWDORDERITEMDET.

    BO_DATA_LOOP_DETAIL_VAR PAR_BO ITEM.

    BO_LOOP_DETAIL_REF_WHERE ITEM LITEMREF 'POBJNR = PAR_ITEM-POBJNR'.
      PAR_DETAILINDEX = LITEMREF->DETAIL_INDEX.
    BO_ENDLOOP_DETAIL.

  endmethod.


  method PREPARE_ORDER_ITEMS.

    data:
      LITEMKEYS_ACT type standard table of TITEMORDERKEY,
      LITEMKEYS_DST type standard table of TITEMORDERKEY,
      LITEMKEYREF   type ref to TITEMORDERKEY,
      LITEMKEYREF2  type ref to TITEMORDERKEY,
      LITEMMAP      type TITEMMAP,
      LITEMMAPS     type TITEMMAPS.


    if LINES( ORDERITEMRESULTS ) is not initial.

**** Get Current keys.
**** Determine Items to move
**** Move Items

      clear:
        FORDER_KEYS.

      FORDER_KEYS = corresponding #( ORDERITEMRESULTS ).

      sort FORDER_KEYS.
      delete adjacent duplicates from FORDER_KEYS.

      if LINES( FORDER_KEYS ) is not initial.

        select * from EWA_ORDER_HEAD into corresponding fields of table @FORDER_KEYS
          for all entries in @FORDER_KEYS
          where ORDER_DATE = @FORDER_KEYS-ORDER_DATE
            and /PLCP/PD_TOUR_ID = @FORDER_KEYS-/PLCP/PD_TOUR_ID
            and ROUTE = @FORDER_KEYS-ROUTE
          and not exists ( select OBJNR from JEST
                             where OBJNR = EWA_ORDER_HEAD~HOBJNR
                               and STAT = @CL_EEWA_CONSTANTS_CONFIRMATION=>CWDOCNFSTATUS_HEAD-CONFIRMED_POSITIVE
                               and INACT = @SPACE ).    "#EC CI_NOFIELD
      endif.

*** todo --- missing order heads.

*** get actual item keys.
      LITEMKEYS_DST = corresponding #( ORDERITEMRESULTS ).


***** adjust destination keys of non planned items / unassigned

      data:
        LPOBJNRS    type standard table of J_OBJNR,
        LORDERNRS   type standard table of EORDERNR,
        LORDERDATES type standard table of EORDER_DATE.


      loop at ORDERITEMRESULTS reference into data(LORDERITEMRESULTREF) where ROUTEASSIGN_REMOVED is not initial.
        insert LORDERITEMRESULTREF->POBJNR into table LPOBJNRS.
      endloop.

      if LINES( LPOBJNRS ) is not initial.
        select POBJNR, ORDERNR, ORDER_DATE, REF_ORDERNR from EWA_ORDER_OBJECT
          for all entries in @LPOBJNRS
          where POBJNR = @LPOBJNRS-TABLE_LINE
          into table @data(LOBJ).

        loop at LOBJ reference into data(LOBJREF).
          insert LOBJREF->ORDER_DATE into table LORDERDATES.
        endloop.

        sort LORDERDATES.
        delete adjacent duplicates from LORDERDATES.

        if LINES( LORDERDATES ) is not initial.

          types:
            begin of _TORDER,
              ORDERNR    type EORDERNR,
              ORDER_DATE type EORDER_DATE,
              ITEM_CNT   type ORDER_LAUFNR,
            end of _TORDER.

          data:
            LORDERS   type standard table of _TORDER,
            LORDERREF type ref to _TORDER.


          select H~ORDERNR, H~ORDER_DATE, max( I~ORDER_LAUFNR ) as ITEM_CNT
            from EWA_ORDER_HEAD as H
            inner join @LORDERDATES as D on H~ORDER_DATE = D~TABLE_LINE
            left outer join EWA_ORDER_OBJECT as I on H~ORDERNR = I~ORDERNR
            where H~ROUTE is initial
            and not exists ( select OBJNR from JEST
                               where OBJNR = H~HOBJNR
                                 and ( STAT = @CL_EEWA_CONSTANTS_CONFIRMATION=>CWDOCNFSTATUS_HEAD-CONFIRMED_POSITIVE
                                 and INACT = @SPACE )
                                 and ( STAT = @CL_EEWA_CONSTANTS_CONFIRMATION=>CWDOCNFSTATUS_HEAD-CANCELED
                                 and INACT = @SPACE )
                                 )
            group by H~ORDERNR, H~ORDER_DATE ", I~ORDER_LAUFNR
            into corresponding fields of table @LORDERS ##ITAB_KEY_IN_SELECT.
        endif.

        loop at LOBJ reference into LOBJREF.
          clear LORDERREF.

          if LOBJREF->REF_ORDERNR is not initial.
            read table LORDERS reference into LORDERREF with key ORDERNR = LOBJREF->REF_ORDERNR.
          endif.
          if LORDERREF is initial or LORDERREF->ITEM_CNT >= 9999.
            read table LORDERS reference into LORDERREF with key ORDER_DATE = LOBJREF->ORDER_DATE.
            if SY-SUBRC is not initial or LORDERREF->ITEM_CNT >= 9999.
              data(LBO) = CREATE_ORDER_HEAD(
                exporting
                  PAR_KEY = value #( ORDER_DATE = LOBJREF->ORDER_DATE )
              ).
              insert initial line into table LORDERS reference into LORDERREF.
              LORDERREF->ORDERNR = LBO->DATAREF->ORDERNR.
              LORDERREF->ORDER_DATE = LBO->DATAREF->ORDER_DATE.
            endif.
          endif.

          if LORDERREF is not initial.
            read table LITEMKEYS_DST reference into LITEMKEYREF  with key POBJNR = LOBJREF->POBJNR.
            if SY-SUBRC is initial.
              LITEMKEYREF->ORDER_DATE = LORDERREF->ORDER_DATE.
              LITEMKEYREF->ORDERNR = LORDERREF->ORDERNR.
              clear:
                LITEMKEYREF->ROUTE,
                LITEMKEYREF->/PLCP/PD_TOUR_ID.

              FORDER_KEYS = value #( base FORDER_KEYS
                ( ORDER_DATE = LITEMKEYREF->ORDER_DATE
                  ORDERNR = LITEMKEYREF->ORDERNR
                  /PLCP/PD_TOUR_ID = LITEMKEYREF->/PLCP/PD_TOUR_ID
                  ROUTE = LITEMKEYREF->ROUTE ) ).

            else.
*              ??? should never happens
            endif.


          else.
***         EXC
          endif.

        endloop.

        sort FORDER_KEYS.
        delete adjacent duplicates from FORDER_KEYS.

      endif.

*****

      select * into corresponding fields of table LITEMKEYS_ACT from EWA_ORDER_OBJECT as OBJ
        inner join EWA_ORDER_HEAD as HDR on HDR~ORDERNR = OBJ~ORDERNR
        for all entries in ORDERITEMRESULTS
        where POBJNR = ORDERITEMRESULTS-POBJNR.
**   todo Status

*** find out differences.

      sort: LITEMKEYS_ACT by POBJNR, LITEMKEYS_DST.

      loop at LITEMKEYS_DST reference into LITEMKEYREF.

        read table LITEMKEYS_ACT reference into LITEMKEYREF2 with key POBJNR = LITEMKEYREF->POBJNR binary search.
        if SY-SUBRC is initial.
          if LITEMKEYREF2->ORDERNR is initial
            or LITEMKEYREF->/PLCP/PD_TOUR_ID <> LITEMKEYREF2->/PLCP/PD_TOUR_ID
            or LITEMKEYREF->ORDER_DATE <> LITEMKEYREF2->ORDER_DATE
            or ( LITEMKEYREF->ORDERNR is not initial and LITEMKEYREF->ORDERNR <> LITEMKEYREF2->ORDERNR )
            or LITEMKEYREF->ROUTE <> LITEMKEYREF2->ROUTE.

            clear LITEMMAP.
            LITEMMAP-SRC = LITEMKEYREF2->*.
            LITEMMAP-DST = LITEMKEYREF->*.

            read table FORDER_KEYS reference into data(LORDERKEYREF)
              with key
                /PLCP/PD_TOUR_ID = LITEMKEYREF->/PLCP/PD_TOUR_ID
                ORDER_DATE = LITEMKEYREF->ORDER_DATE
                ROUTE = LITEMKEYREF->ROUTE.

            if SY-SUBRC is initial.

**** EXCLUDE unassigned items
              read table LPOBJNRS with key TABLE_LINE = LITEMMAP-DST-POBJNR transporting no fields.

              if SY-SUBRC is not initial or LITEMMAP-DST-ORDERNR is initial.
                LITEMMAP-DST-ORDERNR = LORDERKEYREF->ORDERNR.
              endif.

              insert LITEMMAP into table LITEMMAPS.

            else.
*            EXC
            endif.
          endif.
        endif.

      endloop.


*** if differences exist... move items.

      if LINES( LITEMMAPS ) is not initial.

        BOOK_MOVE_ITEMS( changing PAR_ITEM_MAPS = LITEMMAPS ).

      endif.

    endif.

  endmethod.


  method PREPARE_RESULT_DATA.

    data:
      LHEADREF type ref to /PLCP/SWA_WDO_RSLT.

    PREPARE_RESULT_DATA_HEAD( exporting PAR_DATAREF = PAR_DATAREF importing PAR_HEADREF = LHEADREF ).
    PREPARE_RESULT_DATA_HEAD_ACT( exporting PAR_DATAREF = ref #( PAR_DATAREF->ACTIVITIES ) PAR_HEADREF = LHEADREF ).
    PREPARE_RESULT_DATA_HEAD_WGH( exporting PAR_DATAREF = ref #( PAR_DATAREF->WEIGHINGS ) PAR_HEADREF = LHEADREF ).
    PREPARE_RESULT_DATA_HEAD_CN( exporting PAR_DATAREF = ref #( PAR_DATAREF->CONFNOTES ) PAR_HEADREF = LHEADREF ).
    PREPARE_RESULT_DATA_HEAD_ATTA( exporting PAR_DATAREF = ref #( PAR_DATAREF->ATTACHMENTS ) PAR_HEADREF = LHEADREF ).
    PREPARE_RESULT_DATA_HEAD_SPEC( exporting PAR_DATAREF = PAR_DATAREF PAR_HEADREF = LHEADREF ).

    PREPARE_RESULT_DATA_ITEMS( exporting PAR_DATAREF = ref #( PAR_DATAREF->SERVICES ) PAR_HEADREF = LHEADREF ).

*    PREPARE_RESULT_DATA_HEAD_SPEC( exporting PAR_DATAREF = PAR_DATAREF PAR_HEADREF = LHEADREF ).

  endmethod.


  method PREPARE_RESULT_DATA_HEAD.
    data:
      LDATE type EORDER_DATE,
      LTIME type TIMS.

    insert initial line into table ORDERRESULTS reference into PAR_HEADREF.
    PAR_HEADREF->/PLCP/PD_TOUR_ID = PAR_DATAREF->TOURID.
*    LORDERRESULTREF->ROUTE = PAR_DATAREF->TOURTEMPLATE.
    PAR_HEADREF->ROUTE = PAR_DATAREF->EXTERNALTOURTEMPLATEID.
    PAR_HEADREF->ORDERTXT = PAR_DATAREF->ADDITIONALTEXT.

    convert time stamp PAR_DATAREF->TOURDATE time zone SY-ZONLO
        into date LDATE time LTIME.

    PAR_HEADREF->ORDER_DATE = LDATE.

  endmethod.


  method PREPARE_RESULT_DATA_HEAD_ACT.

    loop at PAR_DATAREF->* reference into data(LACTIVITYREF).
      insert initial line into table ORDERRESSRESULTS reference into data(LRESSOURCEREF).
      LRESSOURCEREF->/PLCP/SWA_WDOH_KEY = PAR_HEADREF->/PLCP/SWA_WDOH_KEY.

      case LACTIVITYREF->RESSOURCETYPE.
        when 'EMPLOYEE'.
          LRESSOURCEREF->RESSOURCE_TYPE = 'P'.
          LRESSOURCEREF->PERSON = LACTIVITYREF->PERSON.
        when 'VEHICLE'.
          LRESSOURCEREF->RESSOURCE_TYPE = 'F'.
          LRESSOURCEREF->EQUNR = LACTIVITYREF->EQUIPMENT.
        when others.
**     Warning, skip entry
          delete ORDERRESSRESULTS from LRESSOURCEREF->* ##INDEX_NUM.
          PAR_HEADREF->INVALID_RESULT = 'X'.
          continue.
      endcase.
      LRESSOURCEREF->* = corresponding #( base ( LRESSOURCEREF->* ) LACTIVITYREF->*
        mapping
          DISTANCE_UNIT = DISTANCEUNIT
          DURATION_UNIT = DURATIONUNIT
          ACTIVITY_START = ACTIVITYSTART
          ACTIVITY_END = ACTIVITYEND
          /PLCP/PD_ACTIVITYUUID = ACTIVITYUUID ).

    endloop.


  endmethod.


  method PREPARE_RESULT_DATA_HEAD_ATTA.

    loop at PAR_DATAREF->* reference into data(LTOURATTACHMENTREF).
      insert initial line into table ATTACHMENTRESULTS reference into data(LTOURATTAREF).
      LTOURATTAREF->/PLCP/SWA_WDOH_KEY = PAR_HEADREF->/PLCP/SWA_WDOH_KEY.
      LTOURATTAREF->TOURUUID = LTOURATTACHMENTREF->TOURUUID.
      LTOURATTAREF->DATA = ref #( LTOURATTACHMENTREF->/PLCE/SWR_ATTACHMENT_DATA ).
    endloop.

  endmethod.


  method PREPARE_RESULT_DATA_HEAD_CN.

    loop at PAR_DATAREF->* reference into data(LCONFNOTEREF).
      if LCONFNOTEREF->EXTERNALID is not initial.
        insert initial line into table PAR_HEADREF->CONFNOTES reference into data(LORDERCONFNOTEREF).
        LORDERCONFNOTEREF->CONFTYPE = LCONFNOTEREF->EXTERNALID.
        LORDERCONFNOTEREF->CONFNOTE_COMMENT = LCONFNOTEREF->COMMENTS.
      else. "Log
        PAR_HEADREF->INVALID_RESULT = 'X'.

      endif.

      if LCONFNOTEREF->/PLCE/SWR_ATTACHMENT_DATA-ATTACHMENT is not initial.
        insert initial line into table ATTACHMENTRESULTS reference into data(LCONFTOURATTAREF).
        LCONFTOURATTAREF->/PLCP/SWA_WDOH_KEY = PAR_HEADREF->/PLCP/SWA_WDOH_KEY..
        LCONFTOURATTAREF->TOURUUID = LCONFNOTEREF->TOURUUID.
        LCONFTOURATTAREF->DATA = ref #( LCONFNOTEREF->/PLCE/SWR_ATTACHMENT_DATA ).
      endif.
    endloop.

  endmethod.


  method PREPARE_RESULT_DATA_HEAD_SPEC.

  endmethod.


  method PREPARE_RESULT_DATA_HEAD_WGH.

    loop at PAR_DATAREF->* reference into data(LWEIGHREF).
      insert initial line into table ORDERWEIGHRESULTS reference into data(LORDERWEIGHREF).
*      LORDERWEIGHREF->POBJNR = LWEIGHREF->
      LORDERWEIGHREF->/PLCP/SWA_WDOH_KEY = PAR_HEADREF->/PLCP/SWA_WDOH_KEY.
      LORDERWEIGHREF->POBJNR = LWEIGHREF->POBJNR.
      LORDERWEIGHREF->GROSS_WEIGHT = LWEIGHREF->GROSSWEIGHT.
      LORDERWEIGHREF->NET_WEIGHT = LWEIGHREF->NETWEIGHT.
      LORDERWEIGHREF->QUANTITY = LWEIGHREF->AMOUNT.
      LORDERWEIGHREF->QUANTITY_UNIT = LWEIGHREF->AMOUNTUNIT.
      LORDERWEIGHREF->TARE_WEIGHT = LWEIGHREF->TAREWEIGHT.
      LORDERWEIGHREF->WASTE = LWEIGHREF->PRODUCT.
      LORDERWEIGHREF->LICENSE_NUM = LWEIGHREF->VEHICLELICENSEPLATE.
*      LORDERWEIGHREF->WDPLANT
      if LWEIGHREF->FUNCTIONALLOCATION is not initial.
        select single WDPLANTNR from EWA_EL_WDPLANT into LORDERWEIGHREF->WDPLANT
          where TPLNR = LWEIGHREF->FUNCTIONALLOCATION. "#EC CI_NOORDER
        if SY-SUBRC is not initial.
*** EXC
          PAR_HEADREF->INVALID_RESULT = 'X'.
          LORDERWEIGHREF->INVALID_RESULT = 'X'.
        endif.
      endif.
      LORDERWEIGHREF->WEIGHT_UNIT = LWEIGHREF->WEIGHTUNIT.
      LORDERWEIGHREF->EXT_ID = LWEIGHREF->WEIGHINGID.
      LORDERWEIGHREF->/PLCP/PD_WEIGHING_UUID = LWEIGHREF->WEIGHINGUUID.


    endloop.

  endmethod.


  method PREPARE_RESULT_DATA_ITEM.

    insert initial line into table ORDERITEMRESULTS reference into PAR_ITEMREF.
    PAR_ITEMREF->POBJNR = PAR_DATAREF->REFERENCEINTERNALID.
    PAR_ITEMREF->/PLCP/PD_SERVICE_UUID = PAR_DATAREF->SERVICEUUID.
    PAR_ITEMREF->/PLCP/PD_TOUR_ID = PAR_HEADREF->/PLCP/PD_TOUR_ID.
    PAR_ITEMREF->ROUTE_SEQUENCE = conv #( PAR_DATAREF->TOURSEQUENCE+1 ).
    PAR_ITEMREF->ROUTEASSIGN_REMOVED = PAR_DATAREF->TOURASSIGN_REMOVED.
    PAR_ITEMREF->ORDER_DATE = PAR_HEADREF->ORDER_DATE.
    PAR_ITEMREF->ROUTE = PAR_HEADREF->ROUTE.

    PAR_ITEMREF->BEH_TYPE = PAR_DATAREF->RESULT-CONTAINERTYPEATLOCATION.
    PAR_ITEMREF->BEH_ANZAHL = PAR_DATAREF->RESULT-CONTAINERATLOCATIONCOUNT.
    PAR_ITEMREF->TIDNR = PAR_DATAREF->RESULT-CONTAINERATLOCATIONTIDNR.

    PAR_ITEMREF->BEH_TYPE_NEW = PAR_DATAREF->RESULT-CONTAINERTYPENEW.
    PAR_ITEMREF->BEH_ANZAHL_NEW = PAR_DATAREF->RESULT-CONTAINERNEWCOUNT.
    PAR_ITEMREF->TIDNR_NEW = PAR_DATAREF->RESULT-CONTAINERNEWTIDNR.

    PAR_ITEMREF->WASTE_TYPE = PAR_DATAREF->RESULT-MATERIAL.
    PAR_ITEMREF->TIMESTAMP = PAR_DATAREF->RESULT-TIMESTAMP.
    PAR_ITEMREF->WASTE_WEIGHT = PAR_DATAREF->RESULT-AMOUNT.
    PAR_ITEMREF->WEIGHT_UNIT = PAR_DATAREF->RESULT-AMOUNTUNIT.

  endmethod.


  method PREPARE_RESULT_DATA_ITEMS.

    data:
      LSERVICEREF type ref to /PLCE/SWR_SERVICE_RSLT,
      LITEMREF    type ref to /PLCP/SWA_WDOI_RSLT.

    loop at PAR_DATAREF->* reference into LSERVICEREF.

      PREPARE_RESULT_DATA_ITEM( exporting PAR_DATAREF = LSERVICEREF PAR_HEADREF = PAR_HEADREF importing PAR_ITEMREF = LITEMREF ).
      PREPARE_RESULT_DATA_ITEM_WGH( exporting PAR_DATAREF = ref #( LSERVICEREF->WEIGHINGS ) PAR_HEADREF = PAR_HEADREF PAR_ITEMREF = LITEMREF ).
      PREPARE_RESULT_DATA_ITEM_ATTA( exporting PAR_DATAREF = ref #( LSERVICEREF->ATTACHMENTS ) PAR_HEADREF = PAR_HEADREF PAR_ITEMREF = LITEMREF ).
      PREPARE_RESULT_DATA_ITEM_CN( exporting PAR_DATAREF = ref #( LSERVICEREF->CONFNOTES ) PAR_HEADREF = PAR_HEADREF PAR_ITEMREF = LITEMREF ).
      PREPARE_RESULT_DATA_ITEM_AS( exporting PAR_DATAREF = ref #( LSERVICEREF->ADDITIONAL_SERVICES ) PAR_HEADREF = PAR_HEADREF PAR_ITEMREF = LITEMREF ).
      PREPARE_RESULT_DATA_ITEM_SPEC( exporting PAR_DATAREF = LSERVICEREF PAR_HEADREF = PAR_HEADREF PAR_ITEMREF = LITEMREF ).

    endloop.

  endmethod.


  method PREPARE_RESULT_DATA_ITEM_AS.

    loop at PAR_DATAREF->* reference into data(LSRVADDREF).
      if LSRVADDREF->AMOUNT is not initial.
        insert initial line into table PAR_ITEMREF->ADDITIONAL_SERVICES reference into data(LITEMADDSRVREF).
        LITEMADDSRVREF->* = corresponding #( LSRVADDREF->* mapping MATNR = PRODUCT ZMENG = AMOUNT ZIEME = AMOUNTUNIT ).
*        LITEMCONFNOTEREF->CONFTYPE = LSRVCONFNOTEREF->EXTERNALID.
*        LITEMCONFNOTEREF->CONFNOTE_COMMENT = LSRVCONFNOTEREF->COMMENTS.
      else. "Log
        PAR_HEADREF->INVALID_RESULT = 'X'.
      endif.
    endloop.

  endmethod.


  method PREPARE_RESULT_DATA_ITEM_ATTA.

    loop at PAR_DATAREF->* reference into data(LSRVATTACHMENTREF).
      insert initial line into table ATTACHMENTRESULTS reference into data(LSRVATTAREF).
      LSRVATTAREF->/PLCP/SWA_WDOH_KEY = PAR_HEADREF->/PLCP/SWA_WDOH_KEY.
      LSRVATTAREF->POBJNR = PAR_ITEMREF->POBJNR.
      LSRVATTAREF->SERVICEUUID = LSRVATTACHMENTREF->SERVICEUUID.
      LSRVATTAREF->DATA = ref #( LSRVATTACHMENTREF->/PLCE/SWR_ATTACHMENT_DATA ).
    endloop.

  endmethod.


  method PREPARE_RESULT_DATA_ITEM_CN.

    loop at PAR_DATAREF->* reference into data(LSRVCONFNOTEREF).
      if LSRVCONFNOTEREF->EXTERNALID is not initial.
        insert initial line into table PAR_ITEMREF->CONFNOTES reference into data(LITEMCONFNOTEREF).
        LITEMCONFNOTEREF->CONFTYPE = LSRVCONFNOTEREF->EXTERNALID.
        LITEMCONFNOTEREF->CONFNOTE_COMMENT = LSRVCONFNOTEREF->COMMENTS.
      else. "Log
        PAR_HEADREF->INVALID_RESULT = 'X'.
      endif.

      if LSRVCONFNOTEREF->/PLCE/SWR_ATTACHMENT_DATA-ATTACHMENT is not initial.
        insert initial line into table ATTACHMENTRESULTS reference into data(LCONFSRVATTAREF).
*          LCONFSRVATTAREF->/PLCP/SWA_WDOH_KEY = LORDERRESULTREF->/PLCP/SWA_WDOH_KEY.
        LCONFSRVATTAREF->/PLCP/SWA_WDOH_KEY = PAR_ITEMREF->/PLCP/SWA_WDOH_KEY.
*          LCONFSRVATTAREF->TOURUUID = LSRVCONFNOTEREF->TOURUUID.
        LCONFSRVATTAREF->SERVICEUUID = LSRVCONFNOTEREF->SERVICEUUID.
        LCONFSRVATTAREF->POBJNR = PAR_ITEMREF->POBJNR.
        LCONFSRVATTAREF->DATA = ref #( LSRVCONFNOTEREF->/PLCE/SWR_ATTACHMENT_DATA ).
      endif.
    endloop.

  endmethod.


  method PREPARE_RESULT_DATA_ITEM_SPEC.

  endmethod.


  method PREPARE_RESULT_DATA_ITEM_WGH.

    if PAR_DATAREF->* is not initial.
      clear PAR_ITEMREF->WASTE_WEIGHT.
      clear PAR_ITEMREF->WEIGHT_UNIT.
    endif.

    loop at PAR_DATAREF->* reference into DATA(LWEIGHREF).

      insert initial line into table ORDERWEIGHRESULTS reference into data(LORDERWEIGHREF).
      LORDERWEIGHREF->/PLCP/SWA_WDOH_KEY = PAR_HEADREF->/PLCP/SWA_WDOH_KEY.
      LORDERWEIGHREF->POBJNR = LWEIGHREF->POBJNR.
      LORDERWEIGHREF->GROSS_WEIGHT = LWEIGHREF->GROSSWEIGHT.
      LORDERWEIGHREF->NET_WEIGHT = LWEIGHREF->NETWEIGHT.
      LORDERWEIGHREF->QUANTITY = LWEIGHREF->AMOUNT.
      LORDERWEIGHREF->QUANTITY_UNIT = LWEIGHREF->AMOUNTUNIT.
      LORDERWEIGHREF->TARE_WEIGHT = LWEIGHREF->TAREWEIGHT.
      LORDERWEIGHREF->WASTE = LWEIGHREF->PRODUCT.
      LORDERWEIGHREF->LICENSE_NUM = LWEIGHREF->VEHICLELICENSEPLATE.
      LORDERWEIGHREF->/PLCP/PD_WEIGHING_UUID = LWEIGHREF->WEIGHINGUUID.
      LORDERWEIGHREF->EXT_ID = LWEIGHREF->WEIGHINGID.
      LORDERWEIGHREF->/PLCP/PD_WEIGHING_UUID = LWEIGHREF->WEIGHINGUUID.

      if LWEIGHREF->FUNCTIONALLOCATION is not initial.
        select single WDPLANTNR from EWA_EL_WDPLANT into LORDERWEIGHREF->WDPLANT
          where TPLNR = LWEIGHREF->FUNCTIONALLOCATION. "#EC CI_NOORDER
        if SY-SUBRC is not initial.
*** EXC
          PAR_ITEMREF->INVALID_RESULT = 'X'.
          LORDERWEIGHREF->INVALID_RESULT = 'X'.
        endif.
      endif.
      LORDERWEIGHREF->WEIGHT_UNIT = LWEIGHREF->WEIGHTUNIT.

    endloop.

  endmethod.


  method READ_EWATDEVLOCS.

    types:
      begin of _TORDERKEY.
        include type EWA_ORDER_OBJECT_IKEY.
    types POBJNR type J_OBJNR.
    types end of _TORDERKEY.
    types _TORDERKEYS type standard table of _TORDERKEY.


    data:
      LORDERKEYS type _TORDERKEYS,
      LEWATDEVLOCS type standard table of EWATDEVLOC.

    clear:
      FEWATDEVLOCS,
      FEWATDEVLOCS_DEL,
      FEWATDEVLOCS_INS.

    if LINES( PAR_ITEM_MAPS ) is not initial.

      select * from EWA_ORDER_OBJECT into corresponding fields of table LORDERKEYS
        for all entries in PAR_ITEM_MAPS
        where POBJNR = PAR_ITEM_MAPS-SRC-POBJNR.

      sort LORDERKEYS.
      delete adjacent duplicates from LORDERKEYS.

      if LINES( LORDERKEYS ) is not initial.

        select * from EWATDEVLOC into corresponding fields of table FEWATDEVLOCS
          for all entries in LORDERKEYS
          where ( ORDERNR = LORDERKEYS-ORDERNR and LAUFNR = LORDERKEYS-ORDER_LAUFNR )
            or POBJNR = LORDERKEYS-POBJNR.

      endif.
    endif.

  endmethod.


  method RELEASE_POSITIONS.

*    PAR_SOURCE->ITEM_DELETE( ).



**** ADOPT_POSITIONS
    data:
      LDETAILINDEX     type I,
      LDETAILINDEX_NEW type I,
      LITEMREF_SRC     type ref to EWASWDORDERITEMDET,
      LITEMREF_DST     type ref to EWASWDORDERITEMDET,
      LBO_ITEM_SRC     type ref to CL_EEWA_BO_WDORDERITEM,
      LBO_ITEM_DST     type ref to CL_EEWA_BO_WDORDERITEM,
      LDETAILENRY_OLD  type CL_EEWA_BO_BASE=>TDETAILENTRY,
      LDETAILRECORD    type ref to CL_EEWA_BO_BASE=>TDETAILRECORD,
      LBO              type ref to CL_EEWA_BO_BASE,
      LSTATUS          type ref to CL_EEWA_SAPSTATUS,
      LCL_BO_HELPER    type ref to LCL_BO_HELPER,
      LORDERPOSMAP     type TORDERPOSMAP,
      LDETAILSNAMES    type STRING.


    loop at PAR_POSINDICES into LDETAILINDEX.
      clear:
        LITEMREF_SRC,
        LITEMREF_DST,
        LBO_ITEM_DST,
        LBO_ITEM_SRC,
        LDETAILINDEX_NEW,
        LORDERPOSMAP.

      try.

          PAR_SOURCE->ITEM_CHANGING(
            exporting
*         PAR_INDEX       =
              PAR_DETAILINDEX = LDETAILINDEX
            importing
*         PAR_ITEM        =
*         PAR_DATAREF     =
              PAR_WDOITEM     = LITEMREF_SRC
          ).

          LORDERPOSMAP-POBJNR = LITEMREF_SRC->POBJNR.
          LORDERPOSMAP-ORDERNR_S = LITEMREF_SRC->ORDERNR.
          LORDERPOSMAP-ORDER_LAUFNR_S = LITEMREF_SRC->ORDER_LAUFNR.
***  Check for Status


** BULKY
** SDSHARE
** TDEVLOC
          LBO_ITEM_SRC ?= PAR_SOURCE->GETDETAILBO(
            exporting
              PAR_NAME        = 'ITEM'
              PAR_DETAILINDEX = LDETAILINDEX                 " Detailed Data Record for Business Object
          ).

          if LDETAILSNAMES is initial.
            LBO_ITEM_SRC->GET_INITDATA(
              importing
                PAR_INITDATA = data(LINITDATA)
            ).

            loop at LINITDATA reference into data(LINITDATAREF) where KIND = 'D'.
              if LBO_ITEM_SRC->GETDETAILUSED( PAR_NAME = LINITDATAREF->NAME ) is initial.
                if LDETAILSNAMES is initial.
                  LDETAILSNAMES = LINITDATAREF->NAME.
                else.
                  LDETAILSNAMES = LDETAILSNAMES && ',' && LINITDATAREF->NAME.
                endif.
              endif.
            endloop.
          endif.

          if LDETAILSNAMES is not initial.
            LBO_ITEM_SRC->REQUIRE_DETAILS( PAR_DETAILS = LDETAILSNAMES ).
          endif.

          LBO ?= LBO_ITEM_SRC.

*          clear:
*            LBO->FBULKDATAACCESS.


          LBO ?= PAR_TARGET.
          data(LDETAILENTRY) = LBO->GETDETAILENTRY( exporting PAR_NAME = 'ITEM' ).
          LDETAILENRY_OLD = LDETAILENTRY->*.
          clear:
            LDETAILENTRY->BO_CLASSNAME, LDETAILENTRY->BO_GROUP_NAME, LDETAILENTRY->BO_OBJTYPE.

          PAR_TARGET->ITEM_INSERT(
            importing
              PAR_WDOITEM     = LITEMREF_DST
              PAR_DETAILINDEX = LDETAILINDEX_NEW
          ).

          LBO->GET_DETAILRECORD(
            exporting
              PAR_NAME        = 'ITEM'
              PAR_DETAILINDEX = LDETAILINDEX_NEW
            importing
              PAR_RECORD = LDETAILRECORD
          ).

          LDETAILRECORD->STATUS = 'X'.

          LDETAILENTRY->BO_CLASSNAME = LDETAILENRY_OLD-BO_CLASSNAME.
          LDETAILENTRY->BO_GROUP_NAME = LDETAILENRY_OLD-BO_GROUP_NAME.
          LDETAILENTRY->BO_OBJTYPE = LDETAILENRY_OLD-BO_OBJTYPE.
          LDETAILRECORD->BO_PENDING = 'X'.

          LBO_ITEM_DST ?= PAR_TARGET->GETDETAILBO(
            exporting
              PAR_NAME        = 'ITEM'
              PAR_DETAILINDEX = LDETAILINDEX_NEW
          ).

          if LDETAILSNAMES is not initial.
            LBO_ITEM_DST->REQUIRE_DETAILS( PAR_DETAILS = LDETAILSNAMES ).
          endif.

          LDETAILRECORD->STATUS = 'I'.

          LBO ?= LBO_ITEM_DST.
*          clear:
*            LBO->FBULKDATAACCESS.


          create object LCL_BO_HELPER.
          LCL_BO_HELPER->ADJUST_SAPSTATUS(
            exporting
*          PAR_BO = LBO
              PAR_BO_ITEM = LBO_ITEM_DST
              PAR_OBJNR = LITEMREF_SRC->POBJNR
              ).

          LBO_ITEM_DST->NEW_DATA( ).

          LITEMREF_DST->EWA_ORDER_OBJECT = corresponding #( base ( LITEMREF_DST->EWA_ORDER_OBJECT ) LITEMREF_SRC->EWA_ORDER_OBJECT except KEY ).
          LITEMREF_DST->ORDER_DATE = PAR_TARGET->DATAREF->ORDER_DATE.


          if LBO_ITEM_DST is not initial and LBO_ITEM_SRC is not initial.

            MOVE_ITEM_DETAILS( exporting PAR_SOURCE = LBO_ITEM_SRC PAR_TARGET = LBO_ITEM_DST ).

          endif.

          LBO_ITEM_DST->CHECK_DATA( ).


*          clear:
*            LITEMREF_SRC->POBJNR. " prevent deletion of status object.
          PAR_SOURCE->ITEM_DELETE( PAR_DETAILINDEX = LDETAILINDEX ).

          LORDERPOSMAP-ORDERNR_N = LITEMREF_DST->ORDERNR.
          LORDERPOSMAP-ORDER_LAUFNR_N = LITEMREF_DST->ORDER_LAUFNR.
          insert LORDERPOSMAP into table PAR_ORDERPOSMAPS.

        catch CX_EEWA_BASE into EXCEPTION.
          CALLBACK_REPORT(
            PAR_KIND      = EXCEPTION->MSGTYPE
            PAR_EXCEPTION = EXCEPTION
          ).
          raise exception type /PLCP/CX_EEWA_BASE
            message id '/PLCP/MC_WA'
            type 'E'
            number 015 ##NUMBER_OK
            with LITEMREF_SRC->ORDERNR LITEMREF_SRC->ORDER_LAUFNR.

      endtry.

    endloop.

  endmethod.


  method SAVE_ORDERS.

    data:
      LBO      type ref to CL_EEWA_BO_WDORDER,
      LORDERNR type EORDERNR.

    loop at PAR_ORDERNRS into LORDERNR.
      LBO ?= GET_ORDER_BO( PAR_KEY = LORDERNR PAR_LOCK = 'X' ).
      LBO->BOOK_PLAN( ).
      SAVE_BO( PAR_BO = LBO ).
    endloop.

*** EWATDEVLOCS
    if LINES( FEWATDEVLOCS_DEL ) is not initial.
      delete EWATDEVLOC from table FEWATDEVLOCS_DEL.
    endif.
    if LINES( FEWATDEVLOCS_INS ) is not initial.
*      insert EWATDEVLOC from table FEWATDEVLOCS_INS.  " done by BO itself
    endif.

  endmethod.


  method GET_CALLBACK_EWAORDER.

    create object PAR_CALLBACK exporting PAR_INIT = SPACE.
    PAR_CALLBACK->MSGLOG->HEAD-OBJECT = 'EEWA'.
    PAR_CALLBACK->MSGLOG->HEAD-SUBOBJECT = 'LOGISTICS'.
    PAR_CALLBACK->MSGLOG->HEAD-EXTNUMBER = PAR_EXTNUMBER.
    PAR_CALLBACK->MSGLOG->INIT( ).
    PAR_CALLBACK->CALLBACK = FCALLBACK.

  endmethod.
ENDCLASS.
