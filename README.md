Method: ZZ_IS_AUTO_CONFIRM_VEHICLE
Level: Instance Method, Visibility: Private
Description: e.g. "Order has front or rear loader"
Parameters:

Parameter	Type	Typing	Associated type
IV_ORDERNR	Importing	Type	EORDERNR
RV_RESULT	Returning	Type	ABAP_BOOL
Implementation:


METHOD zz_is_auto_confirm_vehicle.
* Only orders with a front loader (6000) or rear loader (6004) are confirmed
* automatically. No vehicle on the order -> no automatic confirmation.

  SELECT SINGLE @abap_true
    FROM ewa_order_resobj AS r
    INNER JOIN equi  AS e ON e~equnr = r~fleet_equnr
    INNER JOIN fleet AS f ON f~objnr = e~objnr
    WHERE r~ordernr = @iv_ordernr
      AND f~fleet_cat IN ( '6000', '6004' )
    INTO @rv_result.

ENDMETHOD.
