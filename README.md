" old
  IF ltourid IS NOT INITIAL.

" new
  IF ltourid IS NOT INITIAL
     AND zz_is_auto_confirm_vehicle( io_bo_wdoi->dataref->ordernr ) = abap_true.
