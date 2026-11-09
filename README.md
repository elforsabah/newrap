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


* <SIGNATURE>---------------------------------------------------------------------------------------+
* | Static Public Method ZCL_WR_PD_TOUR_HELPER=>CHECK_NOTE_SIGNATURES
* +-------------------------------------------------------------------------------------------------+
* | [--->] IT_ORDERS                      TYPE        TT_SIG_ORDERS
* | [<-()] RT_MISSING                     TYPE        TT_SIG_MISSING
* +--------------------------------------------------------------------------------------</SIGNATURE>
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


* <SIGNATURE>---------------------------------------------------------------------------------------+
* | Static Public Method ZCL_WR_PD_TOUR_HELPER=>GET_EANV_NOTES
* +-------------------------------------------------------------------------------------------------+
* | [--->] IT_ORDERS                      TYPE        TT_SIG_ORDERS
* | [<-()] RT_NOTES                       TYPE        TT_EANV_NOTES
* +--------------------------------------------------------------------------------------</SIGNATURE>
  METHOD get_eanv_notes.
*----------------------------------------------------------------------*
* "Anlage ändern" (follow-up to #40611): an EAP whose Schein hangs on a
* Nachweis that takes part in eANV keeps its disposal facility. This
* method only answers "which EAPs are those". No signature is checked.
* Same data path and the same "nur eANV-teilnehmende NW" rule (A12) as
* CHECK_NOTE_SIGNATURES (#42128):
*   EWA_ORDER_OBJECT-/WATP/NOTEINTNR
*     → /WATP/TARBNOTES (CERTTYPE, NOTENR, COLLECT_NOTE, NWINTNR)
*     → /WATP/TARBNW-DIGI_SIG = 'X', LOEVM = space
*
* Which Nachweis is looked at:
*   BS_DE / SBS_DE  the Schein's own NWINTNR
*   UES_DE          its own NWINTNR; when that is empty, the NWINTNR of
*                   the Sammelbegleitschein behind it (TARBNOTES-
*                   COLLECT_NOTE holds the SBS's INTERNAL number).
*                   The signature check always goes to the SBS for an
*                   ÜS, so the SBS is the safe fallback here as well.
*
* Alternative the ticket mentions ("Scheine, die in /WATP/TARBZKSDOC
* vorkommen"): deliberately NOT used. A Schein on an eANV Nachweis that
* has no ZKS document yet would slip through there, and DIGI_SIG is the
* attribute #42128 already agreed on with the department.
*
* Result per EAP carries the readable values for message 057:
*   notenr   = external Schein number (NOTENR, else internal), ALPHA = OUT
*   certtype = Scheinart (CERTTYPE)
*   nwnr     = external Nachweis number (TARBNW-NWNR)
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


* <SIGNATURE>---------------------------------------------------------------------------------------+
* | Static Public Method ZCL_WR_PD_TOUR_HELPER=>GET_BMS_BEARER_TOKEN
* +-------------------------------------------------------------------------------------------------+
* | [--->] IV_DESTINATION                 TYPE        RFCDEST
* | [--->] IV_USERNAME                    TYPE        CHAR60
* | [--->] IV_PASSWORD                    TYPE        CHAR60
* | [<---] EV_TOKEN                       TYPE        STRING
* | [<---] EV_ERROR                       TYPE        STRING
* +--------------------------------------------------------------------------------------</SIGNATURE>
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


* <SIGNATURE>---------------------------------------------------------------------------------------+
* | Static Public Method ZCL_WR_PD_TOUR_HELPER=>POST_BMS_ORDER
* +-------------------------------------------------------------------------------------------------+
* | [--->] IV_DESTINATION                 TYPE        RFCDEST
* | [--->] IV_BEARER_TOKEN                TYPE        STRING
* | [--->] IV_JSON                        TYPE        STRING
* | [<---] EV_HTTP_STATUS                 TYPE        I
* | [<---] EV_RESPONSE                    TYPE        STRING
* +--------------------------------------------------------------------------------------</SIGNATURE>
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


* <SIGNATURE>---------------------------------------------------------------------------------------+
* | Static Public Method ZCL_WR_PD_TOUR_HELPER=>LOG_BMS_CALL
* +-------------------------------------------------------------------------------------------------+
* | [--->] IV_TOUR_UUID                   TYPE        /PLCE/PDTOUR_UUID
* | [--->] IV_SERVICE_UUID                TYPE        /PLCE/PDSERVICE_UUID
* | [--->] IV_ORDER_NUMBER                TYPE        AUFNR
* | [--->] IV_ENDPOINT                    TYPE        STRING
* | [--->] IV_HTTP_STATUS                 TYPE        I
* | [--->] IV_REQUEST                     TYPE        STRING
* | [--->] IV_RESPONSE                    TYPE        STRING
* | [--->] IV_DIRECTION                   TYPE        CHAR10 (default ='OUTBOUND')
* | [--->] IV_POBJNR                      TYPE        J_OBJNR
* +--------------------------------------------------------------------------------------</SIGNATURE>
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


* <SIGNATURE>---------------------------------------------------------------------------------------+
* | Static Private Method ZCL_WR_PD_TOUR_HELPER=>GET_PATH_PREFIX
* +-------------------------------------------------------------------------------------------------+
* | [--->] IV_DESTINATION                 TYPE        RFCDEST
* | [<-()] RV_PREFIX                      TYPE        STRING
* +--------------------------------------------------------------------------------------</SIGNATURE>
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
