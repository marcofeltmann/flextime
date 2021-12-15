# Ereignisgesteuerte Zeiterfassung

Im Prinzip existieren folgende Komponenten.

- `Irgendwas` erstellt die Events 
	- aktuell: `fish` functions `coming` und `leaving`
- `Irgendwas` ist der Event Store 
	- aktuell: Dateisystem `~/Documents/Arbeitszeiten`
- `App` konvertiert Events aus dem Event Store in ein internes Format und steuert die Ausgabe
	- vermutlich `cmd/app.go`
- `Package` erstellt die gewünschten Reports 
	- vermutlich `github.com/marcofeltmann/flextime`

## Datenformat

Intern brauchen lediglich zwei Informationen vorgehalten werden, die ich hier im Einzelnen näher beleuchte.

### Typ

Diese Aufzählung spezifiziert die Art der Zeitbuchung:

- `coming`: Der Arbeitsplatz wird betreten, die Arbeit aufgenommen.
- `leaving`: Die Arbeit wird beendet, der Arbeitsplatz verlassen.
	
### Zeitstempel

Der Zeitpunkt, an dem `Typ` aufgetreten ist.

## Eingabeformat

Zeit- und Datumsangabe gemäß [RFC3339 Sektion 5](https://datatracker.ietf.org/doc/html/rfc3339#section-5) sollte reichen.

## Ausgabeformat

Das Ausgabeformat der Reports unterscheidet sich je nach Intention.

### Tagesaktuelle (z.B. heutige) Arbeitszeit

Im Prinzip gibt es zwei Zustände, die zu unterschiedlichen Ausgaben führen.

#### Arbeiten abgeschlossen

Ist der letzte Eintrag des gewünschten Tages ein `leaving` und sind die Daten auch relativ konsistent, werden einfach die Zeiträume für jedes `coming`—`leaving`—Paar aufaddiert und angezeigt.

#### Arbeit in Gange

Ist der letzte Eintrag eines gewünschten Tages ein `coming` und sind die Daten auch relativ konsistent, werden die Zeiträume für jedes `coming`—`leaving`—Paar aufaddiert.  
Der Zeitraum des letzten `coming` Events wird bis zum aktuellen Zeitstempel berechnet, zum Bestand addiert und ausgegeben.


## Wochenarbeitszeit

Es wird eine Tabelle von Beginn der Woche bis zum letzten Event zusammengeklöppelt.  
Wie schon bei der tagesaktuellen Arbeitszeit wird ein fehlendes letztes Ereignis durch den aktuellen Zeitstempel aufgefüllt.

Dazu muss natürlich die Wochenarbeitszeit festgehalten werden.  
Wie dies mit Blick auf Kurzarbeit, Urlaub, Krankheit etc. zu realisieren ist muss noch definiert werden…

Beispielsausgabe:

```shell
Wochenarbeitszeit: 40h          KW 50
-------------------------------------
Datum    | Ist       | Soll | Diff
13.12.21 |   7h 35m  |   8h |    25m-
14.12.21 | !10h 12m! |   8h | 2h 12m+
15.12.21 |   8h  5m  |   8h |     5m+
---------+-----------+------+--------
Gesamt   |  25h 52m  |  24h | 1h 52m+
-------------------------------------

Prognose: 15h 8m á 2 Tage ~ 7h 34m Tag

```
