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



    METHOD DO_ADDITIONAL_SERVICES.

** 3 Methods
** 1. Find in DMRPOS
** 2. Find in SDPOS and ADD to DMRPOS
** 3. Insert plane in DMRPOS


    TYPES:
      BEGIN OF _TDMRPOSREF,
        LDMRPOSREF TYPE REF TO EWASORDERDMRPOS_DET,
      END OF _TDMRPOSREF,
      BEGIN OF _TVBAPREF,
        LVBAPREF TYPE REF TO EWASWDORDERITEM_VBAP,
      END OF _TVBAPREF.

    DATA:
      LDMRPOSREF     TYPE REF TO EWASORDERDMRPOS_DET,
      LBO_ITEM       TYPE REF TO CL_EEWA_BO_WDORDERITEM,
      LQUANT         TYPE DZMENG,
      LDMRDETAILREF  TYPE REF TO _TDMRPOSREF,
      LDMRDETAILREFS TYPE STANDARD TABLE OF _TDMRPOSREF,
      LVBAPDETREF    TYPE REF TO _TVBAPREF,
      LVBAPDETREFS   TYPE STANDARD TABLE OF _TVBAPREF,
      LVBAPREF       TYPE REF TO EWASWDORDERITEM_VBAP,
      LEX            TYPE REF TO CX_EEWA_BASE.

    IF LINES( PAR_ADDITIONAL_SERVICES ) IS NOT INITIAL.
      LBO_ITEM ?= PAR_BO_ORDERHEAD->GETDETAILBO(
        EXPORTING
          PAR_NAME        = 'ITEM'
          PAR_DETAILINDEX = PAR_ITEMREF->DETAIL_INDEX
      ).

      LBO_ITEM->REQUIRE_DETAILS( PAR_DETAILS = 'DMR_POS,VBAP' ).

      BO_DATA_LOOP_DETAIL_VAR LBO_ITEM DMR_POS.
      BO_DATA_LOOP_DETAIL_VAR LBO_ITEM VBAP.

      LOOP AT PAR_ADDITIONAL_SERVICES REFERENCE INTO DATA(LADDSRVREF).

        TRY.
            CLEAR:
              LDMRDETAILREFS, LDMRPOSREF, LDMRDETAILREF,
              LVBAPDETREF, LVBAPDETREFS.

            BO_LOOP_DETAIL_REF_WHERE DMR_POS LDMRPOSREF 'MATNR = LADDSRVREF->MATNR'.
*** All details for the material
            LDMRDETAILREFS = VALUE #( BASE LDMRDETAILREFS ( LDMRPOSREF = LDMRPOSREF ) ).
            BO_ENDLOOP_DETAIL.

            IF LINES( LDMRDETAILREFS ) IS NOT INITIAL. "-> already existing --> Add values to existing
              LOOP AT LDMRDETAILREFS REFERENCE INTO LDMRDETAILREF.
                CLEAR: LQUANT.

                IF LDMRDETAILREF->LDMRPOSREF->VBELN IS NOT INITIAL AND LDMRDETAILREF->LDMRPOSREF->SRC_POSNR IS NOT INITIAL.
                  BO_LOOP_DETAIL_REF_WHERE VBAP LVBAPREF 'VBELN = LDMRDETAILREF->LDMRPOSREF->VBELN and POSNR = LDMRDETAILREF->LDMRPOSREF->SRC_POSNR'.
*** needed for Contshare
                  BO_ENDLOOP_DETAIL.
                ELSE.
                  CLEAR: LVBAPREF.
                ENDIF.

                IF LDMRDETAILREF->LDMRPOSREF->ZMENG IS NOT INITIAL AND LDMRDETAILREF->LDMRPOSREF->ZIEME <> LADDSRVREF->ZIEME.
                  CL_EEWA_BO_MEASUREMENT=>CL_UNIT_CONVERSION2(
                    EXPORTING
                      PAR_QUANIN  = LADDSRVREF->ZMENG
                      PAR_UNITIN  = LADDSRVREF->ZIEME
                      PAR_UNITOUT = LDMRPOSREF->ZIEME
                    IMPORTING
                      PAR_QUANOUT = LQUANT
                  ).
                ELSE.
                  LQUANT = LADDSRVREF->ZMENG.
                ENDIF.

                IF LVBAPREF IS NOT INITIAL.
                  LQUANT = LQUANT * LVBAPREF->CONTSHARE.
                ENDIF.

                LDMRDETAILREF->LDMRPOSREF->ZMENG = LQUANT.
                LBO_ITEM->DMR_POS_CHANGED( PAR_DETAILINDEX = LDMRDETAILREF->LDMRPOSREF->DETAIL_INDEX ).
              ENDLOOP.
            ELSE.

              CLEAR: LVBAPREF.
              BO_LOOP_DETAIL_REF_WHERE VBAP LVBAPREF 'MATNR = LADDSRVREF->MATNR'.

              LBO_ITEM->DMR_POS_INSERT(
                IMPORTING
                  PAR_DETAIL      = LDMRPOSREF
              ).

              LDMRPOSREF->* = CORRESPONDING #( BASE ( LDMRPOSREF->* ) LVBAPREF->* MAPPING POSNR = UEPOS SRC_POSNR = POSNR EXCEPT ZMENG POBJNR POS_ORDER ). " das Detail arbeitet nur mit UEPOS!!

              CL_EEWA_EL_SD_INTERFACE=>GET_FOLLOWON_DOCTYPE(
                EXPORTING
                  PAR_VBELN   = LVBAPREF->VBELN                 " Verkaufsbeleg
                  PAR_POSNR   = LVBAPREF->POSNR                 " Verkaufsbelegposition
                IMPORTING
                  PAR_POSTYPE = LDMRPOSREF->PSTYV                   " Positionstyp Vertriebsbeleg
              ).

              TRY .
                  CL_EEWA_BO_MEASUREMENT=>CL_UNIT_CONVERSION2(
                    EXPORTING
                      PAR_QUANIN  = LADDSRVREF->ZMENG
                      PAR_UNITIN  = LADDSRVREF->ZIEME
                      PAR_UNITOUT = LDMRPOSREF->ZIEME
                    IMPORTING
                      PAR_QUANOUT = LDMRPOSREF->ZMENG
                  ).

                CATCH CX_EEWA_BASE.
*              if this is the case, store the values and save only. the confirmation will raise the message and it has to be corrected manually
                  LDMRPOSREF->ZMENG = LADDSRVREF->ZMENG.
                  LDMRPOSREF->ZIEME = LADDSRVREF->ZIEME.
              ENDTRY.

              LDMRPOSREF->ZMENG = LDMRPOSREF->ZMENG * LVBAPREF->CONTSHARE.

              BO_ENDLOOP_DETAIL.

              IF LVBAPREF IS INITIAL.
                LBO_ITEM->DMR_POS_INSERT(
                  IMPORTING
                    PAR_DETAIL      = LDMRPOSREF
                ).
                LDMRPOSREF->DATA = CORRESPONDING #( LADDSRVREF->* ).
              ENDIF.

            ENDIF.

          CATCH CX_EEWA_BASE INTO LEX.
            CALLBACK_REPORT(
              PAR_KIND      = LEX->MSGTYPE                 " PAR_KIND
              PAR_EXCEPTION = LEX                 " PAR_EXCEPTION
            ).
*            catch CX_EEWA_BASE. " CX_EEWA_BASE
        ENDTRY.
      ENDLOOP.
    ENDIF.

  ENDMETHOD.
