Zusammenfassung:

Die Touren der Stadtreinigung (Fachbereich TER) werden im BMS-System verwaltet und ausgeführt. Es werden hierfür keine Entsorgungsaufträge (EAP) zwischen SAP und BMS ausgetauscht.
Das BMS meldet gefahrene Tagestouren mit den erbrachten Leistungen an SAP. Daraus soll im SAP jeweils direkt eine Lastschriftanforderung (SD Sales Order) zum zugehörigen SD-Vertrag (Sales Contract) erzeugt werden.
Separat davon kann das BMS-System ein Auftragsdokument (PDF-Datei) erzeugen und auf einem Document Management System (d3) ablegen. Der Link zu dem Dokument soll an SAP übermittelt werden und dort als Anhang/Archivelink zur Lastschriftanforderung hinzugefügt werden.
Aus diesem Grundansatz werden zwei Endpunkte benötigt. Siehe unten nochmal einzeln beschrieben.
OData V2 oder V4


 #39534 - BMS-Schnittstelle TER - Entwicklung Endpunkte
