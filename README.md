METHOD zz_confirm_wdoi.
*#36097 WP-3.1.11 P2C2-25 DEV Automatic confirmation in EWAWDOC
  DATA:
    ltourid      TYPE /plce/pdtour_id,
    lservice     TYPE /plce/swr_service_rslt,
    lresultdatas TYPE /plce/pwr_tour_rslt ##NEEDED.

  CHECK io_bo_wdoi->dataref->/plcp/pd_service_uuid IS NOT INITIAL.

  IF line_exists( orderitemresults[ pobjnr = io_bo_wdoi->dataref->pobjnr ] ).
    ltourid = orderitemresults[ pobjnr = io_bo_wdoi->dataref->pobjnr ]-/plcp/pd_tour_id.
  ENDIF.

  IF ltourid IS NOT INITIAL.
    zcl_wr_eewa_bo_wdorderitem=>zzv_auto_confirm = abap_true.
    TRY.
        io_bo_wdoi->check_book_confirm_pos( ).
        io_bo_wdoi->book_confirm_pos( ).
      CATCH cx_eewa_base INTO DATA(lex).
        "Wird IMMER ausgeführt – auch bei Exception
        zcl_wr_eewa_bo_wdorderitem=>zzv_auto_confirm = abap_false.
        RAISE EXCEPTION lex.
    ENDTRY.
     "WICHTIG: auch im Erfolgsfall zurücksetzen
    zcl_wr_eewa_bo_wdorderitem=>zzv_auto_confirm = abap_false.
  ENDIF.
ENDMETHOD.
