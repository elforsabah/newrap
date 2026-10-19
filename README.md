max( case
  when _custom.zz_informational_group is not initial
    then _custom.zz_informational_group
  else _order.zz_informational_group
end ) as zz_informational_group
