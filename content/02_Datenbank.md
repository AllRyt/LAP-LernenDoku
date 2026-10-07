# Datenbank-Design

> phpMyAdmin hat einen **Designer** (unter "Datenbankansicht" -> "Mehr"/"Designer)

## Skizzen Elemente

|Element|Beispiel|
|---|---|
|Tabellen|`users`, `products`, `orders`|
|Spalten mit Datentyp|`name VARCHAR(100)`|
|Primärschlüssel (PK)|`id`|
|Fremdschlüssel (FK)|`orders.user_id → users.id`|
|Beziehungen mit Kardinalität|1 User : n Bestellungen|

### Normalisierung
- **1NF:** Atomare Werte, Primärschlüssel
- **2NF:** Kein partieller Abhängigkeit (alle Attribute hängen vom ganzen PK ab)
  > Nur bei zusammengesetztem PK relevant (z. B. Zwischentabelle).
- **3NF:** Keine transitive Abhängigkeit (Nicht-PK-Attribute hängen nur vom PK ab)

## Beziehungen
### Beziehungstypen 
`1:1` mit Unique Constraint über Fremdschlüssel
`1:n` über Fremdschlüssel
`n:m` über Zwischentabelle

### Child-Parent-Beziehungen
**Parent-Table**: Tabelle hält die Hauptdaten bereit.
**Child-Table**: Referenziert den **Primärschlüssel** eines Datensatzes des Parent-Table über eine Spalte mit **Fremdschlüssel**

## Constraints
- `NOT NULL`: Sicherstellen, dass Spalte nicht `NULL` als Wert beinhalten kann.
- `UNIQUE`: Sicherstellen, dass alle Werte in den Spalte einmalig vorkommen.
- `PRIMARY KEY`: einmalig Identifikations-Spalte für jede Zeile(ist `NOT NULL` und `UNIQUE`)
  > Kann aus mehreren Spalten bestehen (zusammengesetzter Schlüssel), typisch bei Zwischentabellen.
- `FOREIGN KEY`: Gewährleistet eine Verknüpfung der Daten zweier Tabellen.
- `CHECK`: Gewährleistet, dass die Werte einer Spalte bestimmte Bedingung erfüllt. (Versionsabhängig)
- `DEFAULT`: Setzt den Standard-Wert einer Spalte, wenn kein spezifischer Wert angegeben wird.

**Index (kein Constraint)**
- `CREATE INDEX`: Erstellt Indexe an Spalten um Daten aus der Datenbank schneller abrufen.
  > `UNIQUE` erzeugt automatisch einen Index.
  
### Referential triggered action

#### Referential Rule / Event Clauses
Das **Trigger-Event** welches Zustand im System definiert, bei dem die Datenbank aktiv eine **Referential Action** ausführt.
- `ON DELETE`: Löschung eines **Parent-Record**.
- `ON UPDATE`: Änderung des referenzierten Schlüsselwerts (meist PK) im **Parent-Record**.

#### Referential Action
Aktion, die beim Event auf die abhängigen Datensätze der **Child-Table** ausgeführt wird.
- `ON X CASCADE`: Änderung wird "durchgereicht". Löscht/ändert automatisch alle verknüpften Zeilen der **Child-Table(s)**, wenn zugehörige Zeile in der **Parent-Table** gelöscht/geändert wird.
- `ON X SET NULL`: Setzt `FOREIGN KEY` in **Child-Table** automatisch auf `NULL`, wenn zugehöriger **Parent-Record** gelöscht/geändert wird.
  > FK-Spalte darf nicht `NOT NULL` sein.
- `ON X SET DEFAULT`: Setzt die `FOREIGN KEY` im **Child-Table** auf `DEFAULT` der Spalte, wenn **Parent-Record** gelöscht/geändert wird.
  > In MySQL/MariaDB (InnoDB) **nicht unterstützt**.
- `ON X RESTRICT`: Verbietet das Löschen/ändern des **Parent-Record**, solange verknüpfte Zeilen in **Child-Table(s)** existieren. Die Datenbank blockiert Löschbefehl.
- `ON X NO ACTION`: Nach Löschung/änderung wird geprüft ob verknüpfte **Child-Table** existieren. Falls Ja wird Löschen rückgängig gemacht.
  > In MySQL/MariaDB identisch mit `RESTRICT`. Standardverhalten, wenn nichts angegeben ist.

## Datenintegrität
**Soft Delete / Status-Flag**
Statt löschen eine Spalte wie `aktiv` oder `freigegeben`.
Löschen ist problematisch, wenn Datensätze noch referenziert werden.

**Snapshot-Werte**
Speichert Werte, die sich ändern können, zum Zeitpunkt des Vorgangs.
Rechnungen dürfen sich nicht nachträglich ändern, wenn z. B. ein Preis angepasst wird.

## Datentypen (MySQL/MariaDB)

| Typ                      | Verwendung                                               |
| ------------------------ | -------------------------------------------------------- |
| `INT AUTO_INCREMENT`     | IDs                                                      |
| `VARCHAR(n)`             | kurze Texte                                              |
| `TEXT`                   | lange Texte                                              |
| `DECIMAL(10,2)`          | Geldbeträge, **nie `FLOAT`** (Rundungsfehler)            |
| `DATETIME` / `TIMESTAMP` | Zeitpunkte                                               |
| `TINYINT(1)` / `BOOLEAN` | Ja/Nein-Flags                                            |
| `ENUM('a','b')`          | feste Auswahl, z. B. Rolle oder Zahlungsart              |
| `VARCHAR(255)`           | Passwort-Hash (Länge von `password_hash()` kann wachsen) |

---

# SQL

> Queries testen: phpMyAdmin → Datenbank wählen → Reiter **SQL**.

## Sprachbereiche

| Kürzel | Name | Betrifft | Befehle |
|---|---|---|---|
| **DDL** | Data Definition Language | **Struktur** | `CREATE`, `ALTER`, `DROP` |
| **DML** | Data Manipulation Language | **Daten** | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** | Data Control Language | **Rechte** | `GRANT`, `REVOKE` |
| **TCL** | Transaction Control Language | **Transaktionen** | `START TRANSACTION`, `COMMIT`, `ROLLBACK` |

## Syntax-Grundregeln

| Element | Schreibweise | Beispiel |
|---|---|---|
| Bezeichner (Tabelle, Spalte) | ohne Zeichen oder in Backticks | `` `products` `` |
| Text-Werte | einfache Anführungszeichen | `'Text'` |
| Zahlen | ohne Zeichen | `42`, `9.99` |
| Statement-Ende | `;` | |
| Kommentar | `-- ` oder `/* */` | |
| Alias | `AS` | `SELECT name AS produktname` |

> Schlüsselwörter sind case-insensitive. Konvention: GROSS.

## Struktur (DDL)

### Datenbank
```sql
CREATE DATABASE <db> CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
DROP DATABASE <db>;
```

### Tabelle erstellen
```sql
CREATE TABLE <tabelle> (
    id            INT AUTO_INCREMENT PRIMARY KEY,
    <spalte>      VARCHAR(100)  NOT NULL,
    <code>        VARCHAR(50)   NOT NULL UNIQUE,
    <betrag>      DECIMAL(10,2) NOT NULL CHECK (<betrag> >= 0),
    <flag>        TINYINT(1)    NOT NULL DEFAULT 0,
    <erstellt>    DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    <parent>_id   INT           NOT NULL,
    FOREIGN KEY (<parent>_id) REFERENCES <parent>(id) ON DELETE RESTRICT
) ENGINE=InnoDB;
```
> **Storage Engine:** Art der internen Tabellenspeicherung der Datenbank. Angabe pro Tabelle.
> **InnoDB:** Speichermodell, unterstützt Fremdschlüssel und Transaktionen. Standard, Angabe optional.

Zwischentabelle (`n:m`) mit zusammengesetztem PK:
```sql
CREATE TABLE <a>_<b> (
    <a>_id  INT NOT NULL,
    <b>_id  INT NOT NULL,
    PRIMARY KEY (<a>_id, <b>_id),
    FOREIGN KEY (<a>_id) REFERENCES <a>(id) ON DELETE CASCADE,
    FOREIGN KEY (<b>_id) REFERENCES <b>(id)
) ENGINE=InnoDB;
```
> Hat die Zwischentabelle eigene Daten (z. B. Menge), bekommt sie oft eine eigene `id` als PK:
```sql
CREATE TABLE <a>_<b> (
    id      INT AUTO_INCREMENT PRIMARY KEY,
    <a>_id  INT NOT NULL,
    <b>_id  INT NOT NULL,
    menge   INT NOT NULL,
    UNIQUE (<a>_id, <b>_id),                    -- optional: Kombination trotzdem einmalig
    FOREIGN KEY (<a>_id) REFERENCES <a>(id),
    FOREIGN KEY (<b>_id) REFERENCES <b>(id)
) ENGINE=InnoDB;
```

### Tabelle ändern / löschen
```sql
ALTER TABLE <tabelle> ADD COLUMN <spalte> VARCHAR(100);
ALTER TABLE <tabelle> MODIFY COLUMN <spalte> VARCHAR(200) NOT NULL;
ALTER TABLE <tabelle> DROP COLUMN <spalte>;
ALTER TABLE <tabelle> ADD UNIQUE (<spalte>);
ALTER TABLE <tabelle> ADD FOREIGN KEY (<parent>_id) REFERENCES <parent>(id);

DROP TABLE IF EXISTS <tabelle>;
```

### Setup-Script (`.sql`-Datei)

| Variante | Tabelle existiert bereits |
|---|---|
| `CREATE TABLE` | Fehler, Script bricht ab |
| `CREATE TABLE IF NOT EXISTS` | wird übersprungen, **Strukturänderungen in der Datei wirken nicht** |

Datei als einzige Wahrheit → vorher alles löschen (**Daten gehen verloren**):
```sql
DROP DATABASE IF EXISTS <db>;
CREATE DATABASE <db> CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE <db>;
-- CREATE TABLE ...  (Parent vor Child)
-- INSERT INTO ...   (Testdaten)
```
> Einzelne Tabellen löschen: Child vor Parent (sonst blockiert der `FOREIGN KEY`).
> Ausführen: phpMyAdmin → **Importieren** oder Inhalt in Reiter **SQL** einfügen.
> Script läuft nur beim **manuellen** Ausführen, nicht automatisch.

**Arbeitsweise**
- Strukturänderungen **immer in der Datei** machen, dann Script neu ausführen → Datei bleibt aktuell.
- Nicht parallel in phpMyAdmin / per `ALTER TABLE` ändern → nächster Reset überschreibt es.
- Sobald echte Daten drin sind (gegen Ende): Script **nicht mehr ausführen**. Änderungen nur noch per `ALTER TABLE`, und in der Datei nachziehen.

**Vor Abgabe**
- Stand sichern: phpMyAdmin → Datenbank → **Exportieren** (Struktur + Daten als `.sql`).

### Index
```sql
CREATE INDEX idx_<spalte> ON <tabelle>(<spalte>);
DROP INDEX idx_<spalte> ON <tabelle>;
```
> **Index:** Sortierte Nachschlagestruktur (**B-Baum**) mit Bezug auf eine spezifische Tabelle. Nicht als Tabelle sichtbar.
> Besteht pro Eintrag aus: **Spaltenwert(e)** + **Verweis auf die Zeile** (in InnoDB: der PK).
> Schnelles `WHERE` / `JOIN` / `ORDER BY` statt alle Zeilen zu durchsuchen. Kostet Speicher und etwas Schreibgeschwindigkeit.
> `PRIMARY KEY`, `UNIQUE` und `FOREIGN KEY` stellen selbst einen Index da.
> Indexierung auf Strings möglich (z. B. `email`). PK als String ebenfalls möglich, üblich ist aber `INT AUTO_INCREMENT`.

## Daten (DML)

### SELECT
```sql
SELECT <spalte1>, <spalte2>          -- * = alle Spalten
FROM <tabelle>
WHERE <bedingung>
ORDER BY <spalte> ASC|DESC
LIMIT <anzahl> OFFSET <start>;
```

> `ASC`: aufsteigende Sortierung (Standard). `DESC`: absteigende Sortierung.
> Ausführungsreihenfolge: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`

### Bedingungen (`WHERE`)

| Operator | Bedeutung |
|---|---|
| `=`, `<>` / `!=`, `<`, `>`, `<=`, `>=` | Vergleich |
| `AND`, `OR`, `NOT` | Verknüpfung |
| `IN (<w1>, <w2>)` | Wert in Liste |
| `BETWEEN <a> AND <b>` | Bereich (inklusive Grenzen) |
| `LIKE '%text%'` | Muster: `%` = beliebig viele Zeichen, `_` = genau ein Zeichen |
| `IS NULL` / `IS NOT NULL` | Prüfung auf `NULL` (**nicht** `= NULL`) |
| `EXISTS (<subquery>)` / `NOT EXISTS` | Subquery liefert mind. eine Zeile |

`LIKE`: Mustervergleich für Texte

| Muster | Trifft zu auf |
|---|---|
| `'Bohr%'` | beginnt mit „Bohr" |
| `'%hammer'` | endet mit „hammer" |
| `'%akku%'` | enthält „akku" |
| `'B_hr'` | `_` = genau ein Zeichen |

> Mit `_ci`-Kollation wird Groß-/Kleinschreibung ignoriert.

### Aggregation
```sql
SELECT <spalte>, COUNT(*) AS anzahl, SUM(<betrag>) AS summe
FROM <tabelle>
GROUP BY <spalte>
HAVING COUNT(*) > 1;
```

| Funktion | Ergebnis |
|---|---|
| `COUNT(*)` / `COUNT(DISTINCT <spalte>)` | Anzahl Zeilen / unterschiedlicher Werte |
| `SUM()`, `AVG()` | Summe, Durchschnitt |
| `MIN()`, `MAX()` | kleinster, größter Wert |

> `WHERE` filtert Zeilen **vor** der Gruppierung, `HAVING` filtert Gruppen **danach**.
> Jede Spalte im `SELECT`, die nicht aggregiert ist, gehört ins `GROUP BY`.

Nützliche Funktionen: `NOW()`, `YEAR(<datum>)`, `MONTH(<datum>)`, `CONCAT(<a>, ' ', <b>)`, `ROUND(<wert>, 2)`

### Joins
```sql
SELECT c.<spalte>, p.<spalte>
FROM <child> c
INNER JOIN <parent> p ON c.<parent>_id = p.id;
```

| Join | Ergebnis |
|---|---|
| `INNER JOIN` | nur Zeilen mit Treffer in **beiden** Tabellen |
| `LEFT JOIN` | **alle** Zeilen links + Treffer rechts (sonst `NULL`) |
| `RIGHT JOIN` | umgekehrt, selten, lässt sich als `LEFT JOIN` schreiben |

> Mehrere Joins hintereinander möglich (z. B. Zwischentabelle → beide Seiten).
> Bei gleichen Spaltennamen immer mit Tabellen-Alias (`c.id`, `p.id`).

### INSERT
```sql
INSERT INTO <tabelle> (<spalte1>, <spalte2>) VALUES (<wert1>, <wert2>);
INSERT INTO <tabelle> (<spalte1>, <spalte2>) VALUES (<w1>, <w2>), (<w3>, <w4>);   -- mehrere Zeilen
```

### UPDATE
```sql
UPDATE <tabelle> SET <spalte1> = <wert1>, <spalte2> = <wert2> WHERE id = <id>;
```

### DELETE
```sql
DELETE FROM <tabelle> WHERE id = <id>;
```

> **`UPDATE` und `DELETE` ohne `WHERE` betreffen alle Zeilen!**

### Transaktionen
```sql
START TRANSACTION;
-- mehrere Statements
COMMIT;      -- alles übernehmen
ROLLBACK;    -- alles verwerfen
```
> Für zusammengehörige Änderungen über mehrere Tabellen. Nur mit `InnoDB`.

---

# SQL Connection

Ein **PDO-Objekt**, die offene Verbindung zur Datenbank. Darüber laufen alle Abfragen.

Setzen des Zugangs
```php
<?php
// config.php
return [
    "host"     => "localhost",
    "dbname"   => "<db>",
    "username" => "<user>",
    "password" => "<pw>",
];
// Zentralisiert außerhalb des Webroots Speichen
// kann aus Git ausgeschlossen werden(`.gitignore`).
```

```php
<?php
class Database
{
    private PDO $conn;

    public function __construct()
    {
        // Pfad relativ zu dieser Datei: /var/www/html/shop/classes/ → /var/www/config.php
        $config = require __DIR__ . '/../../../config.php';

        try {
            $this->conn = new PDO(
                "mysql:host={$config['host']};dbname={$config['dbname']};charset=utf8mb4",
                $config['username'],
                $config['password']
            );
            $this->conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
            $this->conn->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
            $this->conn->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
        } catch (PDOException $e) {
            die("Datenbankverbindung fehlgeschlagen: " . $e->getMessage());
        }
    }

    public function getConn(): PDO
    {
        return $this->conn;
    }
}
```

>`ERRMODE_EXCEPTION`: SQL-Fehler werfen eine Exception.
>`FETCH_ASSOC`: Ergebnisse kommen als `["spalte" => wert]`, ohne doppelte Zahlen-Indizes.
>`EMULATE_PREPARES false`: Echte Prepared Statements in der DB, `LIMIT ?` funktioniert.

```php
// FETCH_BOTH (Standard)
["name" => "Bohrer", 0 => "Bohrer", "preis" => "9.99", 1 => "9.99"]
// FETCH_ASSOC
["name" => "Bohrer", "preis" => "9.99"]
```

> `prepare` + `execute` funktionieren in **beiden** Modi gleich.
> `EMULATE_PREPARES true` (Standard): **PDO** setzt die Werte als maskierte Strings in den SQL-String ein und schickt fertiges SQL → `LIMIT ?` scheitert (`LIMIT '10'`).
> `EMULATE_PREPARES false`: SQL mit Platzhaltern und Werte gehen **getrennt** an die DB.

Include:
```php
require_once __DIR__ . '/classes/Database.php';

$db   = new Database();
$conn = $db->getConn();
```

---

# Prepared Statements / CRUD

## SQL-Injection

Benutzereingabe direkt im SQL-String wird als **SQL-Code** interpretiert.
```php
// NIE SO
$sql = "SELECT * FROM <tabelle> WHERE email = '$email'";
// Eingabe: ' OR '1'='1   →   WHERE email = '' OR '1'='1'   →   liefert alle Zeilen
```

**Schutz:** Prepared Statements. SQL und Werte werden getrennt an die DB geschickt, Werte werden nie als SQL ausgeführt.

## Methoden

```ts
// Typen als Schema
PDO.prepare(sql: string): PDOStatement          // bei ERRMODE_EXCEPTION: Fehler → Exception
PDOStatement.execute(params?: array): bool      // führt aus
PDOStatement.bindValue(param, value, type): bool
PDOStatement.fetch() / fetchAll() / rowCount() ...
```

> `prepare`: Rückgabe: Objekt der Klasse `PDOStatement`.
> `PDOStatement`: vorbereitetes Statement für Methoden `execute`, `fetch`, ...

Reihenfolge:
```
prepare()  →  [bindValue()]  →  execute()  →  fetch() / fetchAll() / rowCount()
```
> Vor `execute` gibt es kein Ergebnis (`fetch()` → `false`).
> `fetch()` liest zeilenweise, jeder Aufruf liefert die nächste Zeile. Nach `fetchAll()` ist alles gelesen.

## Platzhalter

```php
// Positionell: ?
$stmt = $conn->prepare("SELECT * FROM <tabelle> WHERE (id = ? AND <flag> = ?)");
$stmt->execute([$id, 1]);

// Benannt: :name
$stmt = $conn->prepare("SELECT * FROM <tabelle> WHERE email = :email");
$stmt->execute(["email" => $email]);
```
`execute([...])`:
- Array-Elemente ersetzen in **gleicher Reihenfolge** die `?` wie im prepare-Statement.
- Assoziatives Array: `:<Schlüsselwort>` aus prepare-Statement gezielt mit Wert ersetzen: `["<Schlüsselwort>" => wert]` (Doppelpunkt im Schlüssel optional).

> Beide Arten nicht in einem Statement mischen.
> Gleicher benannter Platzhalter zweimal im Statement ist unter `EMULATE_PREPARES false` **nicht erlaubt** → unterschiedliche Namen (`:x1`, `:x2`).

Typ explizit setzen (z. B. bei `LIMIT`):
```php
$stmt = $conn->prepare("SELECT * FROM <tabelle> LIMIT :limit");
$stmt->bindValue(":limit", $limit, PDO::PARAM_INT);
$stmt->execute();
```
> `bindValue(":limit", $limit, PDO::PARAM_INT)` bindet den Wert mit Typ an den Platzhalter.

### Regeln
- Platzhalter **nur für Werte**. Nicht für Tabellen-, Spaltennamen oder `ASC`/`DESC` → dafür feste Auswahl (Whitelist) im PHP-Code.
- `LIKE`: Wildcards in den Wert, nicht ins SQL:
  ```php
  $stmt = $conn->prepare("SELECT * FROM <tabelle> WHERE <spalte> LIKE ?");
  $stmt->execute(["%$suche%"]);
  ```
- `IN (...)` mit variabler Anzahl:
  ```php
  $platzhalter = implode(',', array_fill(0, count($ids), '?'));   // "?,?,?"
  $stmt = $conn->prepare("SELECT * FROM <tabelle> WHERE id IN ($platzhalter)");
  $stmt->execute($ids);
  ```
- `$conn->query("...")` nur für Statements **ohne** Benutzereingaben.

## Ergebnisse

| Methode | Rückgabe |
|---|---|
| `$stmt->fetch()` | eine Zeile als Array, `false` wenn keine |
| `$stmt->fetchAll()` | Array aller Zeilen, `[]` wenn keine |
| `$stmt->fetchColumn()` | Wert der ersten Spalte (z. B. `COUNT(*)`), `false` wenn keine |
| `$stmt->rowCount()` | Anzahl betroffener Zeilen bei `INSERT` / `UPDATE` / `DELETE` |
| `$conn->lastInsertId()` | ID des zuletzt eingefügten Datensatzes (als String) |

## CRUD

```php
// Create
$stmt = $conn->prepare("INSERT INTO <tabelle> (<spalte1>, <spalte2>) VALUES (?, ?)");
$stmt->execute([$wert1, $wert2]);
$id = (int) $conn->lastInsertId();

// Read: ein Datensatz
$stmt = $conn->prepare("SELECT * FROM <tabelle> WHERE id = ?");
$stmt->execute([$id]);
$row = $stmt->fetch();          // false, wenn nicht gefunden

// Read: Liste
$stmt = $conn->prepare("SELECT * FROM <tabelle> WHERE <flag> = ? ORDER BY <spalte>");
$stmt->execute([1]);
$rows = $stmt->fetchAll();

// Update
$stmt = $conn->prepare("UPDATE <tabelle> SET <spalte1> = ?, <spalte2> = ? WHERE id = ?");
$stmt->execute([$wert1, $wert2, $id]);

// Delete
$stmt = $conn->prepare("DELETE FROM <tabelle> WHERE id = ?");
$stmt->execute([$id]);

// Soft Delete / Freigabe
$stmt = $conn->prepare("UPDATE <tabelle> SET <flag> = ? WHERE id = ?");
$stmt->execute([0, $id]);
```

## Doppelte Werte abfangen (`UNIQUE`)

**Vorab prüfen** (für verständliche Fehlermeldung):
```php
$stmt = $conn->prepare("SELECT COUNT(*) FROM <tabelle> WHERE <code> = ?");
$stmt->execute([$code]);
if ($stmt->fetchColumn() > 0) {
    $errors[] = "<Wert> existiert bereits.";
}
```

**Absicherung über die DB** (falls trotzdem doppelt):
```php
try {
    $stmt->execute([...]);
} catch (PDOException $e) {
    if ($e->errorInfo[1] === 1062) {          // MySQL/MariaDB: Duplicate entry
        $errors[] = "<Wert> existiert bereits.";
    } else {
        throw $e;
    }
}
```
> Beim Bearbeiten den eigenen Datensatz ausschließen: `... WHERE (<code> = ? AND id <> ?)`

## Transaktionen

Mehrere Statements werden zu einer Einheit zusammengefasst: Entweder werden **alle** übernommen (`COMMIT`) oder **keines** (`ROLLBACK`).
Schlägt ein Statement in der Mitte fehl, bleibt die DB im Zustand von vorher, es entstehen keine halben Datensätze.
Typisch: Ein Datensatz mit abhängigen Einträgen in einer zweiten Tabelle, die nur gemeinsam Sinn ergeben.

| Methode | Wirkung |
|---|---|
| `beginTransaction()` | schaltet Autocommit **aus**. Ab hier wird nichts endgültig gespeichert. |
| `commit()` | speichert alles seit `beginTransaction()` endgültig, **beendet** die Transaktion, Autocommit ist wieder an |
| `rollBack()` | verwirft alles seit `beginTransaction()`, beendet die Transaktion ebenfalls |

> Endet das Script ohne `commit()`, wird automatisch zurückgerollt.

```php
try {
    $conn->beginTransaction();

    // mehrere zusammengehörige Statements
    $stmt = $conn->prepare("INSERT INTO <parent> (...) VALUES (...)");
    $stmt->execute([...]);
    $parentId = (int) $conn->lastInsertId();

    $stmt = $conn->prepare("INSERT INTO <child> (<parent>_id, ...) VALUES (?, ...)");
    foreach ($items as $item) {
        $stmt->execute([$parentId, ...]);
    }

    $conn->commit();
} catch (PDOException $e) {
    $conn->rollBack();
    throw $e;
}
```
> Entweder alle Statements werden übernommen oder keines. Usecase: Datensatz + abhängige Positionen.

---

# OR-Mapping

Objektrelationales Mapping: Datenbank-Zeilen werden in Objekte übersetzt und umgekehrt.

| Datenbank | PHP |
|---|---|
| Tabelle | Klasse |
| Zeile | Objekt |
| Spalte | Eigenschaft |

## Aufteilung

| Klasse | Aufgabe | Datei |
|---|---|---|
| **Entity** | hält die Daten eines Datensatzes | `classes/<Entity>.php` |
| **Repository** | DB-Zugriff (CRUD) für diese Entity | `classes/<Entity>Repository.php` |

**Entity**
Reines Datenobjekt. Bildet **einen** Datensatz einer Tabelle ab: eine Eigenschaft pro Spalte, dazu Getter/Setter.
Kennt die Datenbank nicht, enthält kein SQL.
Wird überall im Code herumgereicht (Anzeige, Formulare, Validierung).

**Repository**
Einzige Stelle, an der SQL für diese Tabelle steht.
Nimmt Entities entgegen und speichert sie (`insert`, `update`) oder liest Zeilen aus der DB und gibt sie als Entities zurück (`findById`, `findAll`).
Der restliche Code ruft nur Methoden auf (`$repo->findAll()`) und schreibt selbst kein SQL.

→ Trennung: **Entity = Was** (die Daten), **Repository = Wie** (laden/speichern).

> Repository nur bei **Hauptobjekten**.
> Zwischentabellen: kein eigenes Repository, übernimmt eine der Haupttabellen. Eigene Entity bei Bedarf.

> Alternative (Active Record): CRUD-Methoden direkt in der Entity (`$obj->save()`). Weniger Dateien, aber Daten und DB-Zugriff vermischt.

## Entity

```php
<?php
class <Entity>
{
    private ?int $id;                 // null = noch nicht in der DB
    private string $<eigenschaft>;
    private bool $<flag>;

    public function __construct(?int $id, string $<eigenschaft>, bool $<flag>)
    {
        $this->id = $id;
        $this-><eigenschaft> = $<eigenschaft>;
        $this-><flag> = $<flag>;
    }

    // Zeile → Objekt
    public static function fromRow(array $row): self
    {
        return new self(
            (int) $row["id"],
            $row["<spalte>"],
            (bool) $row["<flag>"]
        );
    }

    public function getId(): ?int { return $this->id; }
    public function setId(int $id): void { $this->id = $id; }

    public function get<Eigenschaft>(): string { return $this-><eigenschaft>; }
    public function set<Eigenschaft>(string $wert): void { $this-><eigenschaft> = $wert; }

    public function is<Flag>(): bool { return $this-><flag>; }
    public function set<Flag>(bool $wert): void { $this-><flag> = $wert; }
}
```
> Werte aus der DB beim Mapping casten (`(int)`, `(bool)`, `(float)`). `DECIMAL` kommt als String.
> Spalten (`snake_case`) und Eigenschaften (`camelCase`) können unterschiedlich heißen, das Mapping in `fromRow()` übersetzt.

## Repository

```php
<?php
class <Entity>Repository
{
    private PDO $conn;

    public function __construct(PDO $conn)
    {
        $this->conn = $conn;
    }

    public function findById(int $id): ?<Entity>
    {
        $stmt = $this->conn->prepare("SELECT * FROM <tabelle> WHERE id = ?");
        $stmt->execute([$id]);
        $row = $stmt->fetch();
        return $row ? <Entity>::fromRow($row) : null;
    }

    /** @return <Entity>[] */
    public function findAll(): array
    {
        $stmt = $this->conn->query("SELECT * FROM <tabelle> ORDER BY <spalte>");
        return array_map(fn($row) => <Entity>::fromRow($row), $stmt->fetchAll());
    }

    public function insert(<Entity> $obj): void
    {
        $stmt = $this->conn->prepare("INSERT INTO <tabelle> (<spalte>, <flag>) VALUES (?, ?)");
        $stmt->execute([$obj->get<Eigenschaft>(), (int) $obj->is<Flag>()]);
        $obj->setId((int) $this->conn->lastInsertId());
    }

    public function update(<Entity> $obj): void
    {
        $stmt = $this->conn->prepare("UPDATE <tabelle> SET <spalte> = ?, <flag> = ? WHERE id = ?");
        $stmt->execute([$obj->get<Eigenschaft>(), (int) $obj->is<Flag>(), $obj->getId()]);
    }

    public function delete(int $id): void
    {
        $stmt = $this->conn->prepare("DELETE FROM <tabelle> WHERE id = ?");
        $stmt->execute([$id]);
    }
}
```
> `PDO` wird von außen übergeben → eine Verbindung für alle Repositories.
> Repository hält **Referenz auf das PDO-Objekt** (keine Kopie).
> Namenskonvention Getter bei `bool`: `is<Flag>()`.

## Nutzung

### Bootstrap-Datei
Eine Datei erledigt Debug, Session, Includes und Verbindung. Jede Seite bindet nur diese ein.
```php
<?php
// includes/bootstrap.php
require_once __DIR__ . '/debug.php';               // zuerst → Fehler der anderen werden angezeigt
if (session_status() === PHP_SESSION_NONE) session_start();

require_once __DIR__ . '/../classes/Database.php';
require_once __DIR__ . '/../classes/<Entity>.php';
require_once __DIR__ . '/../classes/<Entity>Repository.php';

$conn = (new Database())->getConn();
```

```php
// jede Seite, ganz oben
require_once __DIR__ . '/includes/bootstrap.php';        // aus admin/: '/../includes/bootstrap.php'
$repo = new <Entity>Repository($conn);
```

### Ohne Bootstrap
```php
require_once __DIR__ . '/classes/Database.php';
require_once __DIR__ . '/classes/<Entity>.php';
require_once __DIR__ . '/classes/<Entity>Repository.php';

$conn = (new Database())->getConn();
$repo = new <Entity>Repository($conn);

// Lesen
$obj  = $repo->findById($id);        // null, wenn nicht gefunden
$list = $repo->findAll();

// Neu anlegen
$neu = new <Entity>(null, $wert, true);
$repo->insert($neu);                 // $neu->getId() ist danach gesetzt

// Ändern
$obj->set<Eigenschaft>($neuerWert);
$repo->update($obj);
```
