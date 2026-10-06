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

## SQL Syntax / Queries-Anatomy

…


## SQL Connection

Ein **PDO-Objekt**, die offene Verbindung zur Datenbank. Darüber laufen alle Abfragen:

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

``` php
<?php
class Database
{

    private PDO $conn;
    
    public function __construct()
    {
	    $config = require __DIR__ . '/config.php';
        try {
            $this->conn = new PDO(
                "mysql:host=$host;dbname=$dbname;charset=utf8mb4",
                $username,
                $password
            );
            $this->conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
            $this->conn->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
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
>`FETCH_ASSOC`: Ergebnisse kommen als `["spalte" => wert]`, ohne doppelte Zahlen-Indizes

Include:
```
require_once __DIR__ . '/Database.php';

$db   = new Database();
$conn = $db->getConn();
```

---

# Prepared Statements / CRUD

---

# OR-Mapping