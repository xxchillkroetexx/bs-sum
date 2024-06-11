# Dateien

- als logische Informationseinheit
- persistente Speicherung von Informationen
- werden vom BS verwaltet ➡ Dateisystem als Teil des BS
- Dateien modellieren die Platte (Adressraum modelliert den Arbeitsspeicher)

## Definition

- Dateien sind Abstraktion zum Speichern von Informationen
- Namen sind wichtiges Merkmal

## Struktur

- Byte-Folgen:
  - Maximum an Flexibilität => Desktopsysteme
- innere Struktur:
  - Datensätze mit fester Größe und interner Struktur => Großrechner

## Typen von Dateien

### Reguläre Dateien

- Informationen des Benutzers
- ASCII-Dateien oder Binär-Dateien

### Verzeichnisse

- Systemdateien zur Verwaltung eines Dateisystems

### Zeichendateien

- Für zeichenorientierte E/A (Drucker, Terminal (`/dev/tty`), ... )

### Blockdateien

- Modellierung von Plattenspeicher (`/dev/hda`)

### Ausführbare Datei unter UNIX/Linux

- Struktur ist essenziell
  ![[executable_file_unix.png]]

### Bibliotheken-Dateien

![[library_file_unix.png]]

## Zugriff auf Dateien

### Sequentieller Zugriff (sequential access)

- früher die einzige Art des Zugriffs
- Bytes oder Datensätze einer Datei werden vom Anfang beginnend nacheinander eingelesen
  Bsp.: Magnetband

### Wahlfreier Zugriff (random access)

- Bytes oder Datensätze einer Datei werden in beliebiger Reihenfolge eingelesen
- Unentbehrlich für Anwendungen (Datenbanken)
  Bsp.: Festplatte
- Methode `seek` legt die Position der Datei fest, die danach wieder sequentiell ausgelesen wird

## Dateiattribute

- Informationen über die Dateien == Metadaten
- Inhalt der Attribute variiert von System zu System

## Dateioperationen

... sind realisiert als Systemaufrufe

# Dateiorganisiation

## Flache Organisiation

- Alle Dateien in einer Ebene
- Einfach
- Einsatz:
  - frühe PC-Systeme
  - Embedded Systems

## Hierarchische Organisation

- Über eine Baum-Struktur
- ➡Verzeichnisse
  ![[hierarchie_dateiverwaltung.png]]

## Navigation im Verzeichnisbaum

### Pfadnamen

- **Absolute Pfadnamen**
  - Beginnen immer im Wurzelverzeichnis und enden mit dem Datei- bzw. Verzeichnisnamen
  - Separatoren trennen die einzelnen Bestandteile
    - Windows➡ "\\"
    - Linux➡ "/"
- Arbeitsverzeichnis
  - Aktuelles Verzeichnis durch den Benutzer bestimmt
  - Startpunkt für relative Pfadnamen
- Relative Pfadnamen
  - Beginnen im Arbeitsverzeichnis
  - Spezialeinträge
    - "`.`" bestimmt das aktuelle Verzeichnis
    - "`..`" bestimmt das übergeordnete Verzeichnis

# Dateisysteme

Datenträger wird in Partitionen eingeteilt
![[mbr_partition.png]]

## Master Boot Record (MBR)

- Wichtig beim Systemstart
- Enthält Partitionstabelle
- Enthält Verweis auf die aktive Partition
  - Beim Start: Lese ersten Block (Boot-Block) der aktiven Partition
  - Ein Programm im Boot-Block lädt dann das Betriebssystem

## Layout Datenträger

| _Layout-Bereich_         | Funktion                                                                                            |
| ------------------------ | --------------------------------------------------------------------------------------------------- |
| _Boot-Block_             | Programm zum Laden des Betriebssystems                                                              |
| _SuperBlock_             | Schlüsselparameter des Dateisystems (Typ, MagischeZahl, Anzahl der Blöcke, ...)                     |
| _Freispeicherverwaltung_ | Informationen über die freien Blöcke im Dateisystem                                                 |
| _i-Nodes_                | Liste von Datenstrukturen, die alle Informationen über je eine Datei enthalten (siehe [[#i-Nodes]]) |
| _Wurzelverzeichnis_      | Einstiegspunkt in die Baumstruktur des Dateisystems                                                 |
| _Dateien_                | Datenablage                                                                                         |

## Implementierung von Dateien

### Ansatz 1: Zusammenhängende Belegung

- Abspeichern als zusammenhängende Menge von Blöcken
  ![[implementierung_blockdateien.png]]

_**Vorteile:**_

- Einfache Implementierung
- Hervorragende Leseleistung

_**Nachteile:**_

- Beim Löschen beginnt die Platte zu fragmentieren

**_mögl. Lösungen:_**

- Verdichten
  - Alle Dateien verschieben, um Lücken zu schließen
  - Sehr Teuer
- Liste mit freien Blöcken verwalten zur Wiederverwendung
  - Schon bei der Erzeugung der Datei muss die Größe bekannt sein
  - unrealistisch

*Verwendung*➡CD-ROMs

### Ansatz 2: Belegung durch Verkettete Liste

- Datei ist verkettete Liste
  ![[implementierung_verkettete_liste.png]]

**_Vorteile:_**

- keine Fragmentierung
- Als Parameter reicht die Nummer des 1. Block aus

**_Nachteile:_**

- Speicherplatz pro Block wird um den Zeiger verringert
- Viele Systemaufrufe zur Positionierung
  - Sequentielles Lesen ist akzeptabel
  - Wahlfreier Zugriff ist sehr teuer

**_mögl. Lösungen:_**

- lege die Zeiger in Tabelle in Arbeitsspeicher ab

### Ansatz 3: Belegung durch Verkettete Listen im Arbeitsspeicher

- Dateiallokationstabelle (File Allocation Table = FAT)
  ![[implementierung_FAT.png]]

**_Vorteile:_**

- Der gesamte Block steht zur Verfügung
- Wahlfreier Zugriff geht schneller, da im Arbeitsspeicher gesucht wird ohne Plattenzugriffe davor
- Immer noch nur ein Parameter für die Datei

**_Nachteile:_**

- Tabelle benötigt Arbeitsspeicher
- Linear mit der Größe der Platte
  - 1 kB Blockgröße, 200 GB
  - ➡ 200.000.000 Einträge mit 4 Byte/Eintrag -> 800 MB
- Nicht bei großen Platten geeignet

### Ansatz 4: i-Nodes

- Datenstruktur mit: - Dateiattributen - Adressen aller Blöcke
  ![[implementierung_i-nodes.png]]

**_Vorteile:_**

- Mit i-Node sind alle Blöcke bekannt
- Die i-Node müssen nur im Speicher sein, wenn die Datei geöffnet ist
  - Reservierter Speicher für i-Nodes hängt von der Anzahl geöffneter Dateien ab
- Umgang mit großen Dateien, die viele Blöcke haben
  - Zeiger auf einen Block mit weiteren Adressen

## I-Nodes

### Implementierung von Verzeichnissen

- Verzeichnis = Datei mit Tabelle mit Verzeichniseinträgen
- Verzeichniseintrag = Informationen zum Auffinden von Plattenblöcken und evtl. über Attribute der Datei - Abbilden von ASCII-Namen auf Datei-Information - Bei zusammenhängender Belegung die Adresse und Größe - Bei verkettenden Listen die Nummer des ersten Blocks - Bei i-Nodes die Nummer des i-Node
  ![[attribute_inode_verzeichnis.png]]

### Erzeugung einer Datei unter Linux

1. I-Node für die Datei erzeugen
2. I-Node für das Verzeichnis aktualisieren (Attribute)
3. Neuer Eintrag in den Verzeichnisblock eintragen
4. "Datei selbst schreiben"

➡ Bei vielen kleinen Dateien viel Overhead
❕Wichtig: Diese Aktionen sollen zeitnah ausgeführt werden
❗Gefahr: Inkonsistenz-Probleme durch Systemabstürze während der Erstellung

## Blöcke

### Blockgröße

- Große Blöcke verschwenden Platz
- kleine Blöcke verschwenden Zeit
  ➡ Optimum hängt von der "üblichen Größe" der Dateien ab
  ![[blockgroesse_vs_datenrate.png]]

### Freie Blöcke

#### Ansatz 1: Verkettete Liste von Plattenblöcken

- Bei leerer Platte sehr groß
- Aber verwende einfach die freien Blöcke und verbrauche damit keinen zusätzlichen Speicher

Blockgröße: 1 kB => 256 32-Bit Plattenblocknummern

#### Ansatz 2: Bitmap

- Nur bei voller Platte verbraucht sie mehr Platz als die verkettete Liste
  - 500 GB => 488 Mio. Bit => 60.000 kB
- Kann als virtueller Speicher verwaltet und seitenweise nachgeladen werden (vgl. [Paging]([[302.2d-Speicherverwaltung#Paging]]))

## Performanz

- hängt von der Größe der zu lesenden/schreibenden Informationen ab
- Schreiben geht langsamer als lesen
- Mechanik der Festplatte muss berücksichtigt werden

### Optimierungen

- Cache
  - Blöcke werden gleichzeitig im Speicher gehalten, um schnell darauf zuzugreifen
  - (vgl. [Paging]([[302.2d-Speicherverwaltung#Paging]]))
- Vorausschauendes Lesen von Blöcken
  - (Lese $i$, und falls $i+1$ nicht im Cache ist, lese auch $i+1$)
  - Beobachtung: Dateien werden in der Regel sequentiell gelesen
    - Nützlich bei sequenziellem Zugriff
    - Gefährlich bei zufälligem Zugriff
- Reduzierung der Bewegung des Plattenarms - Integration der Optimierung im Festplatten-Controller
  ![[reduzierung_bewegung_plattenarm.png]]

### Problem

- Festplattenzugriffszeit blieb in der Vergangenheit nahezu konstant, während alle Komponenten im System schneller wurden
- Dateisystem wurde Engpass im System

➡ Alternative Ansätze notwendig!

- Log-structured File System LFS
- Journaling-File-System

### Log-structured File System (LFS)

- Schreibaufträge werden im Arbeitsspeicher gepuffert
  - Werden gemeinsam und regelmäßig als ein Segment an das Ende eines Log auf die Platte geschrieben
- Zuordnung erfolgt über eine I-Node-Map
  - Diese sind komplizierter
  - Muss ständig aktualisiert werden

Bsp.: Öffnen einer Datei

1. Suchen nach der I-Node der Datei in der I-Node-Map
2. I-Node bestimmt die Blöcke der Datei
   - Diese Blöcke können auch noch in einem Segment liegen

💡Mit Segmenten von ca. 10 MB kann die gesamte Bandbreite der Platte ausgenutzt werden.

- Segmente enthalten auch alte Blöcke
- Gefahr, dass die Log-Datei zu groß wird
- Cleaner-Thread startet regelmäßig um I-Nodes und Blöcke wieder freizugeben

💡LFS ist schneller bei Schreibzugriffen mit kleinen Blöcken (10x)
❗Nicht weit verbreitet, da inkompatibel zu existierenden Dateisystemen

### Journaling-File-System (JFS; bspw. NTFS, ext3, ReiserFS)

- Verbindet Aspekte von LFS mit existierenden Implementierungen

Ablauf:

1. Geplante Operationen werden in einer Datei geloggt (und gleich auf Platte geschrieben)
2. Dann werden die Operationen durchgeführt
3. Wenn erfolgreich, dann werden die Operationen aus dem Log gelöscht

➡Bei Systemabsturz werden die noch im Log stehenden Aufgaben zuerst abgearbeitet
💡 Struktur des Dateisystems überlebt Systemabstürze

## Gemeinsam genutzte Dateien

- Mehrere Personen arbeiten an einer Datei gleichzeitig
  - Datei kommt gleichzeitig in Verzeichnissen verschiedener Benutzer vor
  - Eher ein DAG (Directed Acyclic Graph) als ein Baum
  - Verwendung von Links

**Problem:**

- Wenn die Adressen im Verzeichnis gespeichert sind, dann müssen Änderungen in allen Verzeichnissen durchgeführt werden

**Lösung 1:**

- Attribute in der Metadaten‐Struktur (wie I‐Nodes)
- Problem mit dem „Besitzer“ der Datei

**Lösung 2:**

- Symbolischer Link (auf ursprünglichen Pfad der Datei)
- Problem beim Löschen, da ein Link ins Leere zeigen kann

❗Vorsicht bei der Verwendung von gemeinsam genutzten Dateien

## Konsistenz

**Problem:**

- System stürzt ab, bevor alle veränderten Blöcke geschrieben wurden
- ➡inkonsistentes Dateisystem

### Konsistenzprüfungen:

- Findet auf Ebene der <u>Blöcke</u> statt und erstellt:
  - _Tabelle 1_ für das Vorkommen eines Blockes in einer Datei
  - _Tabelle 2_ für das Vorkommen eines Blockes in der Freibereichsliste
  - Schleife über alle I-Nodes und den Freibereich
  - Alles OK: Jeder Block ist **nur einmal** entweder in Tabelle 1 oder Tabelle 2

#### ... auf Dateien

- Tabelle A für die Verwendung eines I-Nodes
- Inhalt von Tabelle A wird mit I-Node internen Zählern verglichen
- Bei Unterschieden wird der Link-Zähler angeglichen

Mögliche weitere Checks:

- Schreibrechte
- Dateien im Benutzerverzeichnis und Eigentümer root
- ...

### Problemfälle

1. Fehlender Block
   - Kein Schaden
   - Reduziert die Festplattenkapazität
   - 💡Korrektur: Block zur Freibereichsliste hinzufügen
2. Doppelter Block im Freibereich
   - Treten nur bei der "Liste" auf
   - ❗Gefahr: Vergabe eines Blockes an zwei Dateien
   - 💡Korrektur: Neuaufbau der Freibereichsliste
3. Doppelter Datenblock
   - Datenblock ist Teil von zwei Dateien ➡ eine der Dateien ist verstümmelt
   - 💡Korrektur: Rette eine Datei
     - Kopiere den Block auf einen freien Block
     - Eingliedern des neuen Blocks in eine der Dateien

## Verschiedene Dateisysteme

### ISO-9660

- Eine lange Spur sequentieller Daten
- logische Blockgröße: 3252 Byte
  - Metadaten
  - 2048 Byte für Daten
- Persistenter Datenspeicher, nur Lesezugriff
  ![[iso-9660.png]]

### MS-DOS Dateisystem (FAT)

![[FAT.png]]

### UNIX-V7

- Verwendung von I-Node
  ![[unix-v7_inode.png]]

Dateieintrag in UNIX-V7:
![[unix-v7_dateieintrag.png]]

💡Im Wurzelverzeichnis stehen die I-Nodes für die Verzeichnisse `/root`, `/bin`, `/home`, ...

Aufruf von `ls /usr/box/mbox`:
![[unix-v7_i-node_verwendung.png]]

Next:
[IO-Hardware](302.2f-IO-Hardware.md)
