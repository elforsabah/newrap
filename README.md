

In Aufgabe #40611 haben wir entwickelt, dass die Entsorgungsanlage (ESA) im P&D pro EAP geändert werden kann.  Es sind nur die ESAs für einen Wechsel zur Verfügung, die einen Haken in /EWAEL04 > “Zusatz” bei “Verfügbar für Dispo” haben. Details siehe #40611 .

In der Schätzaufgabe #39730 wurde bereits erwähnt “ Auftrag mit Nachweis/Schein (das Feld gibt es aktuell noch nicht in P&D → die Einschränkung muss später ergänzt werden → erstmal nur Platzhalter im Coding)” . Mit dieser Aufgabe soll dies nachgebessert werden.

Wenn eine ESA geändert wird sollen folgende Prüfungen greifen:

Falls die EAP einen Schein (BS/ÜS) besitzt, darf die ESA nicht geändert werden. Der ähnliche Fehlermeldetext wie im IC soll auftauchen:
image_20260930103003.png
2. Falls die EAP keinen Schein besitzt, muss geprüft werden, ob der AVV-Code des Auftrages zu den erlaubten Abfällen an der ESA passt ( /EWAEL04 > Allgemein > Abfall) (Beachte: Wenn keine Einträge bei Abfall sind, kann man ALLE Abfälle an die ESA bringen wie zB bei ESA "PROLOGA" in TI4M442). 

Wenn eine ESA ausgewählt wird, wo der AVV nicht passt, dann sollte der Fehlermeldungstext kommen “Abfall (X, Y) des Auftrages passt nicht zur Entsorgungsanlage (Z) !” X = AVV-Code , Y = Material , Z=ausgewählte, nicht zulässige ESA

Zusatz: Insofern möglich (und im Rahmen dieser Entwicklung kein erheblicher Mehraufwand), soll das Feld Scheinart & -nummer bei der EAP im P&D angezeigt werden. 

Ich bitte um grobe Schätzung der Prüfschritte 1&2 und on top dem “Zusatz”. Ein Kommentar auf diese Aufgabe reicht aus.
