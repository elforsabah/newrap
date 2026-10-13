The direct run works: EV_SUBRC empty, ET_RETURN empty, and /PLCE/TPDSRVCST now shows BEGONNEN. So the module is correct in a dialog session. The difference is the endpoint's call: the module then runs in the RFC session opened by DESTINATION 'NONE', and there the PROLOGA half silently does not land while the EEWA half does.

The most likely mechanism: PROLOGA's save writes /PLCE/TPDSRVCST in the update task. COMMIT ENTITIES commits asynchronously, so a failed update is not visible to the module (the EEWA half uses COMMIT WORK AND WAIT, which would have reported it). In a dialog session a failed update pops up as an express message; in an RFC session it only lands in SM13.

Step 3: two lookups, both in client 442.

SM13: user HWSB10035, from the date of your endpoint tests, status Fehler / abgebrochen ticked. Any entries? Open one: which update module (/PLCE/… or similar) and which error text.
SE16, ZBMS_API_LOG, POBJNR = WE00000000000002478798, newest rows: the SERVICE_UUID column must show 005056B8C72B1FD1AEEEE6FFCD0B4000. If it shows something else or is empty, the endpoint handed the module a wrong service key and the RAP half updated nothing.
Send me both results. If SM13 is clean and the UUID is right, step 4 is an external breakpoint on COMMIT ENTITIES in the module (RFC sessions of your own user honour it) to read ls_failed, ls_c_failed and sy-subrc inside the endpoint's call.
