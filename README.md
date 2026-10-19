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
      FIELDS ( serviceuuid wdplantnr zz_tech_fachbe zz_pobjnr_main )
      WITH VALUE #( ( serviceuuid = <ls_service>-serviceuuid ) )
      RESULT DATA(lt_extcustom).

    IF lt_extcustom IS NOT INITIAL.
      DATA(ls_extcustom) = lt_extcustom[ 1 ].

      IF ls_extcustom-wdplantnr IS NOT INITIAL
         AND ls_extcustom-zz_tech_fachbe IS NOT INITIAL
         AND ls_extcustom-zz_pobjnr_main IS NOT INITIAL.
        CLEAR lt_extcustom.
        CONTINUE.
      ENDIF.
    ENDIF.

    " FIXED: Use CDS view to avoid multiple record issue
    SELECT SINGLE zz_tech_fachbe, wdplantnr, zz_pobjnr_main
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
        ) )
      ) TO lt_ext_create.

    ELSE.
      " ExtCustom EXISTS - update only EMPTY fields
      DATA(lv_needs_update) = abap_false.

      DATA: lv_new_fachbe      TYPE ewa_order_object-zz_tech_fachbe,
            lv_new_wdplantnr   TYPE ewael_wdplantnr,
            lv_new_pobjnr_main TYPE zwr_d_pobjnr_main.

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

      " FIXED: Check zz_pobjnr_main, not wdplantnr!
      IF ls_extcustom-zz_pobjnr_main IS INITIAL
         AND ls_order_fields-zz_pobjnr_main IS NOT INITIAL.
        lv_new_pobjnr_main = zcl_wr_eewa_wdoc_misc=>cl_convert_objnr_out(
                               par_pobjnr = ls_order_fields-zz_pobjnr_main ).
        lv_needs_update = abap_true.
      ELSE.
        lv_new_pobjnr_main = ls_extcustom-zz_pobjnr_main.
      ENDIF.

      IF lv_needs_update = abap_true.
        APPEND VALUE #(
          %key-serviceuuid = <ls_service>-serviceuuid
          zz_tech_fachbe = lv_new_fachbe
          wdplantnr = lv_new_wdplantnr
          zz_pobjnr_main = lv_new_pobjnr_main
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
      UPDATE FIELDS ( zz_tech_fachbe wdplantnr zz_pobjnr_main )
      WITH lt_ext_update
      FAILED DATA(lt_failed_update)
      REPORTED DATA(lt_reported_update).
  ENDIF.

ENDMETHOD.
