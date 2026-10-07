Was wird gemacht	Was der Disponent davon merkt	ET
1	Prüfschritt 1 – Schein vorhanden. Hat die EAP bereits einen Begleit- oder Übernahmeschein, wird der Wechsel der Entsorgungsanlage abgelehnt.	Beim Versuch, die ESA zu ändern, erscheint eine Fehlermeldung mit der externen Scheinnummer und der Scheinart. Die ESA bleibt unverändert.	0,5
2	Prüfschritt 2 – Abfall passt zur Anlage. Diese Prüfung gibt es seit #40611 schon; sie bekommt nur den gewünschten neuen Meldungstext mit AVV-Code, Material und Anlage.	Statt „Für die Entsorgungsanlage … darf AVV … nicht angenommen werden!" steht künftig „Abfall (AVV, Material) des Auftrages passt nicht zur Entsorgungsanlage (…)!". Verhalten sonst unverändert.	0,5
3	Test beider Prüfungen im Testsystem TI4 M442: EAP mit Begleitschein, mit Übernahmeschein, ohne Schein; Anlage mit und ohne Abfall-Liste; Wechsel für mehrere EAPs gleichzeitig.	–	1,0
4	Dokumentation und Nacharbeit nach dem Test.	–	0,5
Zwischensumme Prüfschritte 1 & 2		2,5
5	Zusatz – Schein an der EAP sichtbar machen. Zwei neue Anzeigefelder am Service im P&D: Scheinart und externe Scheinnummer. Die Werte werden beim Öffnen direkt vom Auftrag gelesen, nicht gespeichert, sind also immer aktuell, auch wenn der Schein erst später zugeordnet wird.	In der Detailansicht der EAP im P&D sieht der Disponent sofort, ob und welcher Schein hinterlegt ist, bevor er die ESA ändert.	1,5
