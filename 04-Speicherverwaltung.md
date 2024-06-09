# Speicherverwaltung - Einführung

- Kapazität wird immer größer
  - Programme wachsen aber schneller als verfügbarer Speicher
- Verwaltung der Speicherressourcen -> BS
  ![[zugriffszeiten_speichermedien.png]]

Verschiedenen Speicherkonzepte:
![[speicherverwaltung_annahmen.png]]

- Jeweils nur ein Programm in Ausführung

# Ausführung mehrerer Programme

## Swapping

- Jeweils nur ein Programm im Speicher
  - BS speichert den gesamten Inhalt des Speichers in eine Datei
  - BS ließt eine vorhandene Speicherdatei ein und führt das neue Programm aus

## Hardwareunterstütung durch Schutzschlüssel

- Jeder Speicherblock erhält Schutzschlüssel
- PSW enthält ebenfalls einen Schlüssel
- Hardware prüft beim Zugriff, dass beide Schlüssel identisch sind

## Relokation

- 2 Programme im Speicher aneinanderreihen
  ![[speicher_relokation.png]]
- Problem:
  - Prozess B springt durch `JMP 28` in den Prozess A
- Lösung:
  - Statische Relokation:
    Modifiziere den Prozess B beim Laden in den Speicher ➡ `JMP 28` = `JMP (28+16384)`
  - Problem:
    - `MOV REGISTER1, 28`, aber `28` darf nicht verändert /verschoben werden
    - Wie unterscheidet man `28` von `28`?
    - "Relokierbare Adressen" (dürfen/müssen nicht verschoben werden)

## Konzept: Adressräume

Ziel: Direkter Zugriff auf Speicher vermeiden!

- Schutz
- Relokation

### Speicherabstraktion: Adressräume

💡Virtuelle Adressen werden in physische Adressen umgerechnet

Prozesse: Zeitliches Multiplexing

RAM: Räumliches Multiplexing

### 1. Basis-Limit-Register

- Basis-Register: untere Grenze des physischen Speichers
- Limit-Register: obere Grenze des physischen Speichers
  ![[Basis_Limit_Register.png]]

Zugriff auf Basis-/Limit-Register nur durch BS

**Nachteil**
💡Jeder Speicherzugriff erfordert Addition und Vergleich

### 2. Swapping

- Permanenter Zyklus
  1.  Prozess wird in Speicher geladen
  2.  Prozess läuft einige Zeit
  3.  Prozess wird komplett auf Festplatte gespeichert
- Prozess im Leerlauf können ausgelagert werden
  ![[swapping_fragmentierung.png]]
  ➡Es entstehen Speicherlücken

➡ Prozesse können dynamischen Speicher benötigen
💡Lösung: Segmente

- Logisch zusammenhängender Teil des Adressraums
- Inhalt:
  - Code/Test
  - Daten (inkl. Heap)
  - Stack (Rückkehradressen)

## Frei-Speicher-Verwaltung

### Bitmaps

- Unterteilung in Belegungseinheiten
- Jeder Einheit entspricht ein Bit in Bitmap

- Suche in Bitmap zeitaufwändig
- Größe der Belegungseinheit ist wichtige Entwurfsfrage

### Verkettete Listen

- Eintrag mit Startadresse, Länge und Zeiger auf nächsten Eintrag
- Getrennte Liste für Lücken und Prozesse

Suchalgorithmen

- First Fit
  - Gehe die Liste durch, bis ausreichend große Lücke gefunden ist
- Next Fit
  - Wie First Fit, aber starte bei der letzten gefundenen Lücke
  - Schlechter als First Fit, größere Fragmentierung
- Best Fit
  - Suche die gesamte Liste durch und wähle die kleinste passende Lücke
  - Schlechter als First Fit, erzeugt viele, sehr kleine Fragmente
- Worst Fit
  - Verwende am schlechtesten passenden Bereich
  - Schlechter als First Fit, vernichtet große Freibereiche

# Virtueller Speicher

- Aufteilung des Adressraums eines Programms in kleinere Einheiten (**Seiten, page**)
- Seiten sind aneinander angrenzende Bereiche von Adressen
- Seiten werden dem physikalischen Speicher zugeordnet
  - nicht alle Seiten müssen im physischen Speicher vorhanden sein, damit das Programm läuft
  - **Greift das Programm auf einen Teil des Adressraums zu, der nicht im Hauptspeicher ist, wird das BS aktiv**

## Paging

- Realisierung von virtuellen Speichern
- Trennung zwischen:
  - logischen Adressen
  - Physischen Adressen
- Idee: logische Adresse bildet auf physische Adresse ab
  - Unterstützung durch Hardware: MMU (Memory Management Unit)

Vorteile:

- kein Verschieben beim Laden eines Prozesses erforderlich
- Auch kleine freie Speicherbereiche sind nutzbar
- Speicherschutz ergibt sich (fast) automatisch
- Teile des Prozessadressraums können ausgelagert werden
  ![[paging_diagram.png]]

### Einträge in Speichertabelle

![[img/entries_page_table.png]]

| Bit                | Funktion                                 |
| ------------------ | ---------------------------------------- |
| Present/Absent     | Rahmen im Speicher?                      |
| Protection (3 bit) | Lesen, Schreiben, Ausführen              |
| Modified           | Wird von MMU beim Schreibzugriff gesetzt |
| Referenced         | Wird von MMU bei jedem Zugriff gesetzt   |
| Caching            | z.B. bei speicherbasiertem E/A           |

## Funktionsweise

1. Aufteilung der Adressräume in Blöcke fester Größe
   - Seite (page): Block im <u>virtuellen</u> Adressraum
   - Seitenrahmen (page frame): Block im <u>physischen</u> Adressraum
2. Betriebssystem erstellt und verwaltet Seitentabelle - Umsetzungstabelle definiert für jede Seite: - Physische Adresse des entsprechenden Seitenrahmens (falls vorhanden) - Zugriffsrechte
   ![[paging_example.png]]

## Page Fault

- Die Abbildung von Seite auf Seitenrahmen scheitert, weil der Seitenrahmen nicht im Arbeitsspeicher präsent ist
  - ➡Nachladen erforderlich
- Betriebssystem wählt einen anderen Seitenrahmen aus und schreibt dessen Inhalt auf die Festplatte
- Betriebssystem lädt den angeforderten Seitenrahmen
- Betriebssystem ändert die Seitentabelle
- **Betriebssystem führt den Befehl erneut aus**

## Adressumsetzung mit Speichertabelle

- falls einer Seite kein Seitenrahmen zugeordnet wird➡page fault
- Speicherschutz ist automatisch gegeben, da Seitentabelle nur auf Kacheln verweist, die dem Prozess zugeordnet sind
- Zusätzlich gibt es einen Schreibschutz für einzelne Seiten
  - Schutz von Programmcode
  - Bei Verletzung -> Ausnahme erzeugen

**Adressraum ist Abstraktion des Speichers**

## Seitenersetzung

page fault➡BS muss Seiten/Seitenrahmen managen

Vereinfachter Ablauf:

1. Page Fault durch MMU
2. BS ermittelt virtuelle Adresse (Seite S) und Grund der Ausnahme
   1. Falls Schutzverletzung ➡ Prozess abbrechen, Fertig!
   2. Falls kein Seitenrahmen verfügbar:
      1. Bestimme zu verdrängende Seite S'
      2. Falls S' modifiziert wurde (M-Bit): S' auf Platte schreiben
      3. Seitentabelleneintrag von S' aktualisieren (P-Bit = 0)
3. Seite S von Platte in freien Seitenrahmen laden
4. Seitentabelleneintrag von S aktualisieren (P-Bit = 1)
5. Prozess fortsetzen

# Adressräume

### Anforderungen

1. Schnelle Umrechnung von Seite nach Seitenrahmen
2. Seitentabelle muss großen virtuellen Adressraum realisieren können
   - 4 KB Seitengröße und 32 Bit Adressraum ➡$1.04 \cdot 10^6$ Seiten
     - -> 4 MB Speicher notwendig
   - 4 KB Seitengröße und 64 Bit Adressraum ➡$4.5 \cdot 10^{15}$ Seiten
     - -> 32 PB Speicher notwendig
   - ➡ Jeder Prozess hat seine eigenen Seitentabelle

## Ansätze

1. Seitentabelle als Hardware aus Register in CPU
   - Teuer
   - Langsam bei Prozesswechsel
2. Seitentabelle vollständig im Speicher
   - langsam bei Unterbrechung von Speicherreferenzen

## Implementierungen

### 1. Anforderung: Beschleunigung durch <u>Translation Lockaside Buffer</u> (_TLB_)

- Programme greifen oft auf sehr wenige Seiten zu
- Kleine Hardwareeinheit
  - Interner Cache aus Registern
  - Teil der MMU
- Verwaltung des TLB - CISC (x86) Prozessoren: MMU - RISC Prozessoren: Software (BS)
  ![[img/translation_lockaside_buffer.png]]

### 2. Anforderung: Große Speicherbereiche durch mehrstufige Seitentabellen

- Stufe 1: Hauptseitentabelle mit 1024 Einträgen
- Stufe 2: Seitentabellen mit je 1024 Einträgen
  ![[img/mehrstufige_seitentabellen.png]]

❗Bei 64-Bit Systemen explodiert die Anzahl der Tabellen
**💡invertierte Tabellen**

- Seitentabelle speichert einen Eintrag nur im physikalischen Seitenrahmen
- Jeder Eintrag enthält (Prozess, Seitennummer)
- Verwaltungsaufwand ist größer
  - Suche durch diese Tabelle bei jedem Zugriff
  - Caching mit TLB möglich

# Seitenersetzungsalgorithmen

Optimale Eigenschaften:

- Zeit zum nächsten Seitenfehler maximieren
  - "Glaskugel": in die Zukunft schauen nicht möglich
  - Vergangenheit durch Statusbits (R(eferenced), M(odified), P(resent))

## Algorithmen

### Not-Recently-Used (NRU)

- Verwende die durch MMU gesetzten Statusbits in Seitentabelle
  - M-Bit: Seite wurde modifiziert
  - R-Bit: Seite wurde referenziert (wird regelmäßig von BS gelöscht)
- Bei Verdrängung: 4 Prioritätsklassen

| _Klasse_ | _referenziert_ | _modifiziert_ |
| -------- | -------------- | :------------ |
| 0        | false          | false         |
| 1        | false          | true          |
| 2        | true           | false         |
| 3        | true           | true          |

Vorteil: einfach, leicht verständlich
Nachteil: nicht optimal, aber oft ausreichend

### First-In-First-Out (FIFO)

- Verdränge die Seite, die am längsten im Hauptspeicher ist.

Einfache, aber schlechte Strategie, da Zugriffsverhalten ignoriert wird
➡nur selten eingesetzt

### Second-Chance

- Variante von FIFO, die das R-Bit verwendet
- Verdränge die älteste Seite, auf die seit dem letzten Seitenwechsel nicht zugegriffen wurde
- Älteste Seite steht an erster Stelle

Wechsel:

1. Falls erste Seite der Liste R=0: Seite entfernen
2. Sonst: Setze R=0, schiebe Seite ans Ende der Liste
3. Gehe zu Schritt 1.

💡Falls zu Begin alle Seiten R=1 hatten wir die älteste Seite beim zweiten Durchlauf verdrängt

### Clock

- Effizientere Implementierung von Second-Chance (Verschieben wird vermieden)
- Seiten in Ringliste angeordnet, Zeiger auf ältestem Eintrag
  ![[clock_page_algorithm.png]]

Bei Page Fault:

1. Betrachte Seite, auf die der Zeiger zeigt:
2. R=0:
   - Verdränge Seite
3. R=1:
   - setze R=0
   - setze Zeiger eins weiter
   - wiederhole von vorn

### Least-Recently-Used (LRU)

- Verdränge die Seite, die am längsten nicht benutzt wurde
- Verkettete Liste mit neustem Eintrag am Anfang und dem am längsten nicht benutzten Eintrag am Ende

Problem: Bei jedem Speicherzugriff muss die Liste aktualisiert werden
➡Sehr Teuer, HW-Unterstützung notwendig, aber selten gegeben

Vorteil: Guter Algorithmus

### Not-Frequently-Used

- Software-Variante von LRU
- Entferne die Seite, die am wenigsten benutzt wird
- Für jede Seiten ∃ Zähler

Problem: Seiten, die zu Beginn einen hohen Wert erhalten, dann aber nicht mehr verwendet werden, werden erst sehr spät verdrängt

### Aging

- Veränderung von NFU
- Zähler werden 1 Bit nach rechts geschoben
- R-Bit wird zum ganz linken Biz des Zählers addiert
- R-Bits löschen
- Bei Seitenfahler wird die Seite mit dem kleinsten Wert verdrängt
- Durch endliche Anzahl von Bits wird der Algorithmus vergesslich

Effiziente implementierbare Näherung an LRU

### Working Set (Arbeitsbereich)

- Bei Page Fault: Lagere eine Seite aus, die nicht zum Arbeitsbereich gehört

Problem: wie findet das BS die Seite, die nicht zum Arbeitsbereich gehört?
Kriterium für Arbeitsbereich:

- Der Arbeitsbereich ist die Menge der Seiten, die seit den letzten $k$ Speicherzugriffen benutzt wurde (siehe [[#Hintergrund]])

#### Bestimmung Arbeitsbereich

Ansatz:

1. Lege $k$ im Voraus fest
2. Schieberegister mit $k$ Einträgen
3. Bei jedem Speicherzugriff wird die aktuelle Seriennummer hinzugefügt
4. Bei einem Seitenfehler das Schieberegister analysieren
   - Sortieren
   - Doppelte Einträge entfernen
5. Wechsel des Arbeitsbereichs durchführen

Nachteil: Schieberegister aktuell zu halten und zu analysieren ist aufwendig!
➡Bessere (Näherungs-)Methoden notwendig

#### Hintergrund

$w(k,t)$: Menge der Seiten zur Zeit $t$, die $P$ bei den letzten $k$ Speicherzugriffen benutzt hat
![[working_set.png]]
Für große $k$ annähernd konstant, aber noch kleiner als der Prozessadressraum

- Arbeitsbereich im Speicher: Prozess läuft ohne viele Seitenfehler bis zur nächsten Phase seiner Ausführung
- Arbeitsbereich passt _nicht_ in Speicher: viele Seitenfehler ➡langsame Ausführung
  - Seitenflattern (Trashing): Seiten werden regelmäßig mit hohen Kosten (Festplatte) ein- und ausgelagert
- Multiprogrammierung ➡Prozesse werden oft ausgelagert

-> [[#Working Set (Arbeitsbereich)]], Prepaging

#### Anwendung

- Virtuelle Zeit $t_V$ = CPU-Zeit, die ein Prozess seit seinem Start benutzt hat
- Arbeitsbereich (neue Definition):
  - Arbeitsbereich ist die Menge der Seiten, auf die innerhalb einer bestimmten virtuellen Zeit $\tau$ zugegriffen wurde (z.B. 100 ms)
- Voraussetzung:
  - Trage die Virtuelle Zeit in jeden Eintrag der Seitentabelle als Zeitpunkt des letzten Eintrags $t_{LE}$ ein
- Lagere Seiten aus, die bei einem Seitenfehler nicht zum Arbeitsbereich gehören
  - 💡R-Bit
- Wenn $R=1$: setze den Zeitpunkt des letzten Eintrags auf den Wert der virtuellen Zeit $t_V$
- Wenn $R=0$:
  - Wenn $(t_V - t_{LE}) \gt \tau$, Seite entfernen
  - Wenn $(t_V - t_{LE}) \le \tau$, kleinsten Zeitpunkt merken
- Wen keine Seite ersetzt wird, ersetze die mit dem kleinsten Zeitpunkt

Bei jedem Seitenfehler wird die gesamte Tabelle durchlaufen.
Guter Algorithmus, aber ineffizient

![[working_set_algo.png]]

### Working-Set-Clock

Verbindung von [[#Clock]] und [[#Working-Set-Clock]]

💡 In realen Systemen weit verbreitet

Ringförmige Liste von Seitenrahmen

Bei Page Fault: Untersuche Seite, auf die der Zeiger zeigt

1. Wenn $R=1$:
   - setze $R=0$ und Zeiger vorrücken und wiederhole den Algorithmus
2. Wenn $R=0$:
   - Wenn $(T_V - T_{LE}) \gt T$
     - Wenn $M=0$, Seite ersetzen
     - Wenn $M=1$, Seite vormerken
   - und Zeiger vorrücken und wiederhole den Algorithmus
3. Wenn die gesamte Liste durchlaufen:
   - Mindestens eine Seite wurde vorgemerkt;
     - Zeiger läuft weiter, bis er eine Seite findet mit $M=0$ (durch BS)
   - Keine Seite vorgemerkt
     - Irgendeine Seite auslagern

![[working_set_clock.png]]

## Programmstart

### Demand-Paging

Seitenwechsel nur auf Anforderung

Nachteil:
Start eines Programms kann lange dauern, bis alle benötigten Seiten geladen sind

Verbesserung:
Die demnächst benötigten Seiten werden gleich in den Speicher geladen

### Lokalitätseigenschaft (locality of reference)

Prozesse beschränken sich in jeder Phase ihrer Ausführung auf einen relativ kleinen Teil ihrer Seiten

➡nur ein Teil der Seiten muss im Speicher sein
![[locality_of_reference.png]]

## Fazit:

- Aufgabe: Bestimmung der Seite, die Verdrängt wird
- Optimale Strategie:
  - Seite, die in Zukunft am längsten nicht gebraucht wird
- NRU: vier Klassen gemäß R- und M-Bit
- FIFO: Seite, die am längsten im Hauptspeicher ist
- Second Chance:
  - älteste Seite, die seit letztem Seitenwechsel nicht benutzt wurde
    - Clock-Algo: effiziente Implementierung
  - LRU: Seite, die am längsten nicht benutzt wurde
    - Aging: effiziente Implementierung
- Working Set: Verwendet virtuelle Zeit und Prepaging
  - WS-Clock: effiziente verbreitete Implementierung

# Paging

## Kriterien

### Zuteilungsstrategie - lokal vs global

- Darf die auszulagernde Seite einem anderen Prozess angehören?
- Lokal: nur die Seiten der Prozesse
  - Fester Speicherbereich für jeden Prozess
  - Führt bei wachsendem Speicher zur Zunahme von Seitenfehlern
    - Trashing, Seitenflattern
- Global: auch Seiten anderer Prozesse dürfen ausgelagert werden - Dynamische Speicherbereiche - Überwachung der Speicherbereiche durch "PFF" (Page Fault Frequency) - 💡mehr oder weniger Speicher zuteilen?
  ![[page_fault_frequency.png]]

### Lastkontrolle

- Zu wenig Speicher im System für $N$ Prozesse
  - Speicherbereich für jede Prozess wird kleiner
  - Reduzierung der Prozesse notwendig
    - Auslagern von Prozessen
    - Reduzierung der PFF auf akzeptables Maß
  - Einfluss auf Scheduling von Prozessen

### Seitengröße

Für kleine Seiten spricht:

- Interne Fragmentierung: für jedes Segment bleibt im Mittel die letzte Seite halb leer
- Programme belegen in verschiedenen Phasen nur Teil des Adressraums

Für große Seiten spricht:

- Optimierte Festplattenzugriffe (nur ein Positionierungsschritt)
- Kurze Seitentabelle

### Trennung von Befehls- und Datenräumen

- Mögliche Strategie bei kleinen Adressräumen
- Befehlsraum (I-Space) für Befehle (Programmtext)
- Datenraum (D-Space) für Daten
- Getrennte Adressräume
  - Eigene Paging Mechanismen

### Gemeinsame Seiten

- Mehrere Benutzer verwenden das gleiche Programm
  - Seiten, die Programmtext enthalten, können gemeinsam genutzt werden
  - Seiten mit Daten nicht
- Verwende getrennte Befehls-Datenräume - Einfluss auf Scheduler: Wenn Prozess 1 beendet wird, aber das Programm von einem anderen Benutzer (Prozess 2) verwendet wird, darf die Programm-Seitentabelle nicht gelöscht werden
  ![[gemeinsame_seiten.png]]

### Bereinigungsstrategien

- Paging braucht genügend freie Seitenrahmen
- Regelmäßiges Bereinigen der Seitenrahmen durch Paging-Daemon
  - Wenn zu wenige freie Seitenrahmen vorhanden sind
  - Speichern von veränderten Seiten auf Festplatte, ohne die Seite aus Speicher zu entfernen
  - Hält die freien Seitenrahmen sauber, was Festplatten-Zugriffe vermindert

## Implementierung

### 1. Erzeugung von Prozessen

1. Seitentabelle erzeugen
2. Platz für ausgelagerte Seiten reservieren (auf Festplatte)
3. Swap-Bereich mit Programmtext und Daten initialisiere (auf Festplatte)
4. Aktualisieren der Prozesstabelle

### 2. Auswählen eines Prozesses durch Scheduler

- MMU aktualisieren
  1.  Spuren des alten Prozesses beseitigen
  2.  Aktuelle Seitentabelle auf Prozess-Seitentabelle setzen
  3.  evtl. Laden einiger Seiten um Seitenfehlerrate zu senken

### 3. Auftreten eines Seitenfehlers (page faults)

1. Feststellen welche virtuelle Adresse den Fehler erzeugt hat
2. Benötigte Seite auf Festplatte finden
3. Neuen Seitenrahmen suchen
4. Befehlszähler auf den verursachenden Befehl zurücksetzen

### 4. Beenden des Prozesses

1. Seitentabelle, Seitenrahmen und Plattenplatz für ausgelagerte Seiten freigeben
2. Evtl. gemeinsame genutzte Seiten erst später löschen

## Swap-Bereich

- Bei Systemstart leer
- Jeder gestartete Prozess kopiert seinen Inhalt auf die Swap-Partition
  - Position und Größe des Swap-Bereichs wird im PCB in der Prozesstabelle abgelegt
- Prozess wird gestoppt ➡Bereich wird freigegeben

**Initialisierung**:

1. Alle Seiten zuerst in Swap-Partition initialisieren und bei Bedarf in den Speicher einlagern
2. Alle Seiten im Speicher initialisieren und bei Bedarf in den Swap-Bereich auslagern

**Implementierung**
![[swap_bereich.png]]

Next:
[Dateien und Dateisysteme](302.2e-DateienDateisysteme.md)
