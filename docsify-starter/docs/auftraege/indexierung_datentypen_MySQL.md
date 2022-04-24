# Indexierung und Datentypen bei MySQL

## Datentypen / Attribute

### Datentypen in MySQL

| Datentyp | Speicherplatz | Optionen | Beschreibung |
|---|---|---|---|
| TINYINT | 1 Byte | [(M)] [U] [Z] | Ganzzahlen von 0 bis 255 oder von -128 bis 127. |
| SMALLINT | 2 Bytes | [(M)] [U] [Z] | Ganzzahlen von 0 bis 65.535 oder von -32.768 bis 32.767. |
| MEDIUMINT | 3 Bytes |   [(M)] [U] [Z] | Ganzzahlen von 0 bis 16.777.215 oder von -8.388.608 bis 8.388.607. |
| INT | 4 Bytes | [(M)] [U] [Z] | Ganzzahlen von 0 bis ~4,3 Mill. oder von -2.147.483.648 bis 2.147.483.647. |
| INTEGER | 4 Bytes | [(M)] [U] [Z] | Alias für INT. |
| BIGINT | 8 Bytes | [(M)] [U] [Z] | Ganzzahlen von 0 bis 2^64-1 oder von -(2^63) bis (2^63)-1. |
| FLOAT | 4 Bytes | [(M,D)] [U] [Z] | Fließkommazahl mit Vorzeichen. Wertebereich von -(3,402823466×1038) bis -(1,175494351×10-38), 0 und 1,175494351×10-38 bis 3,402823466×1038. |
| DOUBLE | 8 Bytes | [(M,D)] [U] [Z] | Fließkommazahl mit Vorzeichen. Wertebereich von -(1,79769×10308) bis -(2.22507×10-308), 0 und 2.22507×10-308 bis 1,79769×10308 |
| REAL | 8 Bytes | [(M,D)] [U] [Z] | Alias für DOUBLE. |
| DECIMAL | M+x Bytes | [(M,D)] [U] [Z] | Fließkommazahl mit Vorzeichen. Speicherbedarf: x=1 wenn D=0, sonst x=2. Ab MySQL 5.1 binär gespeichert, zuvor als String. |
| NUMERIC | M+x Bytes | [(M,D)] [U] [Z] | Alias für DECIMAL. |
| DATE | 3 Bytes | - | Datum im Format 'YYYY-MM-DD'. Wertebereich von 01.01.1000 bis 31.12.9999. |
| DATETIME | 8 Bytes | - | Datumsangabe im Format 'YYYY-MM-DD hh:mm:ss'. Wertebereich entspricht DATE. |
| TIMESTAMP | 4 Bytes | - | Zeitstempel. Wertebereich: 1.1.1970 bis 19.01.2038. Das Format variiert in den MySQL-Versionen. |
| TIME | 3 Bytes | - | Zeit zwischen -838:59:59 und 839:59:59. Ausgabe: hh:mm:ss. |
| YEAR | 1 Byte | [(2|4)] | Jahr zwischen 1901 bis 2155 bei (4) und zwischen 1970 bis 2069 bei (2). |
| CHAR | M Byte(s) | (M) [BINARY] | Zeichenkette fester Länge M. Wertebereich für M: 0 bis 255. |
| VARCHAR | L+1 Bytes | (M) [BINARY] | Zeichenkette variabler Länge, Maximum ist M. Wertebereich für M: 0 bis 255. |
| BINARY | M Bytes | (M) | Zum Speichern binärer Strings, unabhängig vom Zeichensatz. Wertebereich für M: 0 bis 255. Weiterer Typ: VARBINARY |
| BLOB | L+2 Bytes | (M) | Binäres Objekt mit variablen Daten. Weitere Typen: TINYBLOB, MEDIUMBLOB und LONGBLOB. M ist ab Version 4.1 definierbar. |
| TEXT | L+2 Bytes | (M) | Wie BLOB. Ignoriert beim Sortieren & Vergleichen Groß- und Kleinschreibung. Weitere Typen: TINYTEXT, MEDIUMTEXT, LONGTEXT. M ist ab Version 4.1 definierbar. |
| ENUM | 1 oder 2 Bytes | ('val1', 'val2', ...) | Liste von Werten (val1, val2, ...). 65.535 eineindeutige Elemente sind maximal möglich. |
| SET | x Bytes | ('val1', 'val2', ...) | String-Objekt mit verschiedenen Variablen. 64 sind maximal möglich. Speicherbedarf: x ist 1, 2, 3, 4 oder 8. |

**Legende**

| Zeichen | Bedeutung |
|---|---|
| ^ | Potenzzeichen |
| [ ] | Optionaler Parameter |
| BINARY | Attribut für die Sortierung |
| D | Anzahl der Kommastellen bei einer Dezimalzahl |
| L | Stringlänge (Berechnung Speicherbedarf) |
| M | Maximale Anzahl der gezeigten Stellen |
| Mill. | Milliarden |
| U | UNSIGNED (Zahl ohne Vorzeichen) |
| Z | Zerofill |

### Spezialtyp: JSON

**JSON Arrays** beginnen mit [ und enden mit ]. Sie enthalten verschiedene Werte, welche durch Kommas voneinander getrennt werden:

```json
["abc", 10, null, true, false]
```

**JSON Objekte** beginnen mit { und Enden mit }. Sie enthalten verschiedene Werte als Key-value (Schlüssel-Werte), welche auch durch eine Komma voneinander getrennt werden:

```json
{"k1": "value", "k2": 10}
```

**Beispiele:**

Tabelle mit einem Attribut vom Datentyp JSON erstellen:

```sql
CREATE TABLE t1 (jdoc JSON);
```

Datensatz mit JSON-Wert hinzufügen:

```sql
INSERT INTO t1 VALUES('{"key1": "value1", "key2": "value2"}');
```

### Spezialtyp: ENUM

Der Datentyp **ENUM** enthält String-Objekte, welche nur Werte aus einer definierten Liste erlaubt. Diese Liste an zulässigen Werten wird in der Spaltenspezifikation bei der Tabellenerstellung definiert.

**Beispiele:**

Tabelle mit einem Attribut vom Datentyp ENUM erstellen:

```sql
CREATE TABLE employee (
    name VARCHAR(40),
    department ENUM('IT', 'Purchasing', Production')
);
```

Datensatz mit ENUM-Wert hinzufügen:

```sql
INSERT INTO employee (name, department)
VALUES ('Max','IT'), ('Tom', 'Production');
```

## Indexierung

### Allgemein

In MySQL ist ein Index eine spezielle Datenstruktur. Diese beschleunigt die Abfrage von Datensätzen, da sonst ohne Index mittels WHERE sämtliche Einträge einer Tabelle überprüft werden müssten. Durch den Index werden Einträge einer Spalte indiziert. Dies bedeutet, dass zusätzlich eine Datenstruktur erstellt wird, die das schnelle Finden von Datensätzen ermöglicht.
Indizes sind vor allem bei grossen Tabellen mit über 10'000 Datensätzen, bei denen die Spalte häufig mit WHERE oder ORDER Statements abgefragt wird.

### Primary Key

Für jede Tabelle ist nur ein Primary Key möglich. Dieser ist typischerweise eine ID und muss in dieser Spalte einzigartig sein.

### Unique

Der Index-Typ Unique sorgt dafür, dass ein Wert innerhalb einer Spalte einzigartig ist. Im Gegensatz zum Primary Key sind mehrere Spalten des Typs Unique in einer Tabelle möglich. Dies könnte man z.B. für eine Spalte E-Mail Adresse verwenden, da diese für jeden Benutze einzigartig sein muss.

### Index

Spalten, die mit Index gekennzeichnet sind, erlauben auch doppelte Werte in dieser Spalte.

### Fulltext

Der Typ Fulltext eignet sich für Spalten die längere Texte enthalten. Dabei wird jedes Wort einzeln indiziert. Dadurch wird die Suche nach einzelnen (Teil-)Wörtern ermöglicht. Dieser Index-Typ ist deshalb sinnvoll, wenn man mit der Volltextsuche von MySQL arbeiten möchte.
