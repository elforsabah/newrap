Step 1. SE16 in 442, two tables:

/PLCE/TPDSRV, REFERENCE_INT_ID = WE00000000000002478798. Note how many rows and their SERVICE_UUID values.
ZBMS_API_LOG, POBJNR = WE00000000000002478798, newest rows. Note the SERVICE_UUID column and the request text ({"BmsStatus":"…"} (was "…")).
/PLCE/TPDSRVCST for every SERVICE_UUID from 1: does a row exist, and what is in ZZ_BMS_STATUS?
Send me the three results. If the log's UUID matches a row that did change, it is case one and the fix is the service selection. If no row changed at all, it is case two, and step 2 is a direct test of ZWR_BMS_UPDATE_SERVICE in SE37 with that UUID to see whether the RAP part persists outside the endpoint.



<img width="1880" height="228" alt="image" src="https://github.com/user-attachments/assets/f188ae4b-3871-44af-a35a-4e2c3e56e642" />
<img width="1367" height="864" alt="image" src="https://github.com/user-attachments/assets/b78921eb-e444-4a57-a71a-c12a652bba89" />
<img width="1357" height="915" alt="image" src="https://github.com/user-attachments/assets/2c590eb9-ccb1-489a-a55a-70b688504a10" />
