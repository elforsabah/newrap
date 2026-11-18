Ursache gefunden. RA und P&D liefern den richtigen Wert (3,5 H). Der Fehler entsteht beim Verbuchen in die Entsorgungsauftragsposition.

Die in IC angelegte Zusatzleistung hat auf der Position eine Menge, aber keine Mengeneinheit. Beim Verbuchen wird deshalb umgerechnet, und ohne Zieleinheit kommt die Menge in Sekunden zurück: 3,5 H × 3600 = 12.600. Die Einheit bleibt leer.

Korrektur in ZCL_PLCP_TA_WA_ORDER_RSLT, Methode DO_ADDITIONAL_SERVICES: Hat die Position keine Einheit, werden Menge und Einheit unverändert aus der RA übernommen. Aus 3,5 H wird dann 3,5 H.
