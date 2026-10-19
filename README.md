Das Feld containerMovementTypeInfo auf dem Fahrer-Tablet zeigt künftig den Text der W&R-Leistungsart, zum Beispiel "Sofortbefüllung" oder "Tauschen". Der Text kommt immer aus der Standard-Texttabelle und hängt nicht vom Kennzeichen ab.

Das Kennzeichen entscheidet nur, ob dahinter die geplante Abfallmenge mit Einheit angehängt wird. Drei Beispiele:

Auftrag 7363, Leistungsart 06, Menge 4 TO: Kennzeichen gesetzt, Menge wird angehängt. Der Fahrer sieht "Sofortbefüllung 4 t".
Auftrag 9, Leistungsart 03, Menge 1 TO: kein Kennzeichen, Menge bleibt weg. Der Fahrer sieht "Tauschen". Ohne das Kennzeichen stünde dort "Tauschen 1 t", eine Zahl, die niemand eingegeben hat, sondern der Systemvorschlag.
Auftrag 735, Leistungsart 06, Menge 0: Kennzeichen gesetzt, aber Menge 0. Sie bleibt weg, der Fahrer sieht "Sofortbefüllung".
Soll später eine weitere Leistungsart ihre Menge anzeigen, etwa 05, setzt der Fachbereich das Häkchen in SM30 (View ZWRVWAALAPRCTP). Kein Programmänderung, kein Transport.
