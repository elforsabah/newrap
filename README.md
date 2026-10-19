Transaction SE11, Database table, enter ZWRTWAALAPRCTP, click Change (pencil). If the system asks for a transport request, use the one your BMS objects are already in, or create a new workbench request named "BMS containerMovementTypeInfo".
On the Fields tab, scroll to the last row and enter in the next empty line:
Field: BMS_QTY_INFO
Data element: XFELD
Press Enter. The row should show type CHAR, length 1, short description "Checkbox".
Save, then Check (Ctrl+F2), then Activate (Ctrl+F3). Activation runs without SE14 because the field is appended at the end and is not a key.
Take a screenshot of the Fields tab after activation, showing all fields and the "aktiv" status in the header.
