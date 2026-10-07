# Formulardaten
Übertragung von Benutzereingaben aus HTML-Formularen oder der URL an das PHP-Script.

## Formular (HTML)

```html
<form method="post" action="<ziel>.php">
    <input type="text" name="<feld>" value="">
    <button type="submit">Senden</button>
</form>
```

| Attribut        | Bedeutung                                                                             |
| --------------- | ------------------------------------------------------------------------------------- |
| `method="post"` | Daten im Request-Body → `$_POST`. Für alles, was Daten ändert oder sensibel ist       |
| `method="get"`  | Daten in der URL (`?feld=wert`) → `$_GET`. Für Filter, Suche, IDs zum Anzeigen        |
| `action`        | Ziel-Script. Leer/weggelassen = gleiches Script                                       |
| `name`          | Schlüssel im Array (`$_POST["<feld>"]`). **Ohne `name` wird das Feld nicht gesendet** |

## Auslesen

```php
$wert = $_POST["<feld>"] ?? "";          // Standardwert, falls nicht vorhanden
$id   = (int) ($_GET["id"] ?? 0);        // Zahl aus URL
```

> `??` (Null-Coalescing): nimmt rechten Wert, wenn links nicht gesetzt oder `null` → keine Warnung "Undefined index".

Prüfen, ob Formular abgeschickt wurde:
```php
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    // Formular verarbeiten
}
```

## Feldtypen-Besonderheiten

| Feld | Verhalten |
|---|---|
| `checkbox` | **Nicht angehakt → wird gar nicht gesendet**. Prüfen mit `isset($_POST["<feld>"])` |
| `select` | sendet `value` der gewählten `<option>` |
| `radio` | sendet `value` des gewählten Buttons, keiner gewählt → nicht gesendet |
| `hidden` | unsichtbar, z. B. ID beim Bearbeiten: `<input type="hidden" name="id" value="...">` |
| Mehrfachauswahl | `name="<feld>[]"` → kommt als Array an |

> Alle Werte kommen als **String** (oder Array). Zahlen selbst casten / validieren.
> Hidden-Felder und URL-Parameter kann der Benutzer beliebig ändern → genauso prüfen wie sichtbare Felder.

## Werte nach Fehler erneut anzeigen

```php
<input type="text" name="<feld>" value="<?= htmlspecialchars($_POST["<feld>"] ?? "") ?>">
<input type="checkbox" name="<flag>" <?= isset($_POST["<flag>"]) ? "checked" : "" ?>>
```

> `<?= ... ?>` ist Kurzform für `<?php echo ... ?>`.

---

# Datei-Upload
Dateien vom Client entgegennehmen, prüfen und im Dateisystem des Servers ablegen.

## Formular

```html
<form method="post" action="<ziel>.php" enctype="multipart/form-data">
    <input type="file" name="<bild>" accept="image/*">
    <button type="submit">Hochladen</button>
</form>
```

> **`enctype="multipart/form-data"` Pflicht**, sonst kommt keine Datei an.
> `accept` ist nur ein Hinweis für den Browser, kein Schutz.

## `$_FILES`

```php
$_FILES["<bild>"] = [
    "name"     => "foto.jpg",          // Original-Dateiname (vom Client, nicht vertrauen)
    "type"     => "image/jpeg",        // MIME-Typ (vom Client, nicht vertrauen)
    "tmp_name" => "/tmp/phpA1b2C3",    // temporäre Datei auf dem Server
    "error"    => 0,                   // UPLOAD_ERR_OK = 0
    "size"     => 123456,              // Bytes
];
```

| `error` | Bedeutung |
|---|---|
| `UPLOAD_ERR_OK` (0) | erfolgreich |
| `UPLOAD_ERR_INI_SIZE` (1) | größer als `upload_max_filesize` (php.ini) |
| `UPLOAD_ERR_NO_FILE` (4) | keine Datei gewählt |

## Ablauf

```php
$errors = [];
$file = $_FILES["<bild>"] ?? null;

if (!$file || $file["error"] !== UPLOAD_ERR_OK) {
    $errors[] = "Bild fehlt oder Upload fehlgeschlagen.";
} else {
    // 1. Größe
    if ($file["size"] > 2 * 1024 * 1024) {
        $errors[] = "Bild zu groß (max. 2 MB).";
    }

    // 2. Echten Typ prüfen (Dateiinhalt, nicht Name/Client-Angabe)
    $erlaubt = ["image/jpeg" => "jpg", "image/png" => "png", "image/webp" => "webp"];
    $mime = (new finfo(FILEINFO_MIME_TYPE))->file($file["tmp_name"]);
    if (!isset($erlaubt[$mime])) {
        $errors[] = "Nur JPG, PNG oder WEBP erlaubt.";
    }
}

if (empty($errors)) {
    // 3. Eigenen, eindeutigen Dateinamen erzeugen
    $dateiname = bin2hex(random_bytes(16)) . "." . $erlaubt[$mime];

    // 4. Aus temp-Ordner in Zielordner verschieben
    if (move_uploaded_file($file["tmp_name"], __DIR__ . "/uploads/" . $dateiname)) {
        $pfad = "uploads/" . $dateiname;     // → in DB speichern
    } else {
        $errors[] = "Bild konnte nicht gespeichert werden.";
    }
}
```

### Regeln
- **Eigenen Dateinamen vergeben**: verhindert Überschreiben, Sonderzeichen und `.php`-Dateien im Upload-Ordner.
- In der DB nur den **relativen Pfad** speichern, Ausgabe: `<img src="<?= htmlspecialchars($pfad) ?>">`.
- Beim **Bearbeiten** ohne neue Datei (`UPLOAD_ERR_NO_FILE`): alten Pfad behalten, kein Fehler.
- Beim **Ersetzen / Löschen** alte Datei entfernen: `unlink(__DIR__ . "/" . $alterPfad);`
- Ordner `uploads/` braucht Schreibrechte für den Webserver-User (`www-data`).
- Limits in php.ini: `upload_max_filesize`, `post_max_size` (Standard oft 2 MB / 8 MB).

---

# Validierung

Prüft, ob Eingaben **fachlich gültig** sind. Fehler gehen an den **Benutzer**.

> HTML-Attribute (`required`, `type="email"`, `maxlength`) sind nur Komfort. Lassen sich umgehen → **Validierung immer serverseitig in PHP**.

## Muster

```php
$errors = [];

$<feld> = trim($_POST["<feld>"] ?? "");

if ($<feld> === "") {
    $errors[] = "<Feld> ist ein Pflichtfeld.";
} elseif (mb_strlen($<feld>) > 100) {
    $errors[] = "<Feld> darf max. 100 Zeichen haben.";
}

if (empty($errors)) {
    // speichern, weiterleiten
}
```

Ausgabe:
```php
<?php if (!empty($errors)): ?>
    <ul>
        <?php foreach ($errors as $e): ?>
            <li><?= htmlspecialchars($e) ?></li>
        <?php endforeach; ?>
    </ul>
<?php endif; ?>
```

> Fehler pro Feld zuordnen: `$errors["<feld>"] = "..."` → Ausgabe direkt beim Feld: `$errors["<feld>"] ?? ""`.

## Prüfungen

| Prüfung              | Code                                                                       |
| -------------------- | -------------------------------------------------------------------------- |
| Pflichtfeld          | `trim($x) === ""`                                                          |
| Länge                | `mb_strlen($x)` (Umlaute = 1 Zeichen, `strlen` zählt Bytes)                |
| E-Mail               | `filter_var($x, FILTER_VALIDATE_EMAIL) !== false`                          |
| Ganzzahl             | `filter_var($x, FILTER_VALIDATE_INT)` → Zahl oder `false`                  |
| Ganzzahl mit Bereich | `filter_var($x, FILTER_VALIDATE_INT, ["options" => ["min_range" => 1]])`   |
| Kommazahl            | `filter_var(str_replace(",", ".", $x), FILTER_VALIDATE_FLOAT)`             |
| Muster               | `preg_match('/^[0-9]{4}$/', $x) === 1`                                     |
| Feste Auswahl        | `in_array($x, ["<a>", "<b>"], true)`                                       |
| Gleichheit           | `$pw !== $pwWiederholung`                                                  |
| Einmalig in DB       | `SELECT COUNT(*)` vorab, siehe `02_Datenbank.md` → Doppelte Werte abfangen |

> `filter_var(...)` liefert bei `0` als gültige Zahl `0` → Vergleich immer mit `=== false`, nicht `!$x`.

## Validator-Klasse

Hilfsfunktionen als statische Methoden, wiederverwendbar in allen Formularen:

```php
<?php
class Validator
{
    public static function required(string $wert): bool
    {
        return trim($wert) !== "";
    }

    public static function maxLength(string $wert, int $max): bool
    {
        return mb_strlen($wert) <= $max;
    }

    public static function email(string $wert): bool
    {
        return filter_var($wert, FILTER_VALIDATE_EMAIL) !== false;
    }
}

// Nutzung
if (!Validator::email($email)) {
    $errors["email"] = "Ungültige E-Mail-Adresse.";
}
```

## Zahlungsdaten (Kreditkarte)

- In der Prüfung nur **simuliert**: Format prüfen, keine echte Zahlung.
- **Kartennummer, Ablaufdatum, CVC nicht in der DB speichern.** Höchstens Zahlungsart + ggf. letzte 4 Ziffern.

---

# Ausgabe absichern
Schutz vor Angriffen, die über Benutzereingaben eingeschleust werden.

## XSS (Cross-Site-Scripting)

Benutzereingaben werden ungefiltert ins HTML ausgegeben und vom Browser als **HTML/JavaScript** ausgeführt.
```php
// NIE SO
echo "Hallo " . $_GET["name"];
// Eingabe: <script>...</script>   →   Script läuft im Browser anderer Besucher
```

**Schutz:** Bei **jeder Ausgabe** von Daten, die von außen kommen (Formular, URL, **auch aus der DB**), maskieren.

```php
echo htmlspecialchars($wert, ENT_QUOTES, "UTF-8");
```

### Maskierung (Escaping)
HTML-Sonderzeichen werden in **HTML-Entities** umgewandelt → Browser zeigt sie als Text an, statt sie als HTML/JS auszuführen.

```ts
htmlspecialchars(string $string, int $flags, ?string $encoding): string
```
- `$string`: umzuwandelnder Text.
- `$flags`: Bitmaske für Umgang mit Anführungszeichen, ungültigen Zeichenfolgen und Dokumenttyp.
  - Default (ab PHP 8.1): `ENT_QUOTES`
  maskiert `"` **und** `'` (wichtig in Attributen).
- `$encoding`: Zeichenkodierung des Textes. Default:  (ab PHP 8.1): `UTF-8`


| Zeichen | wird zu |
|---|---|
| `<` / `>` | `&lt;` / `&gt;` |
| `"` / `'` | `&quot;` / `&#039;` |
| `&` | `&amp;` |

Hilfsfunktion (z. B. in `includes/`):
```php
function e(?string $wert): string
{
    return htmlspecialchars($wert ?? "", ENT_QUOTES, "UTF-8");
}

// Nutzung
<p><?= e($produkt["<spalte>"]) ?></p>
<input value="<?= e($_POST["<feld>"] ?? "") ?>">
<a href="produkt.php?id=<?= (int) $produkt["id"] ?>">
```

### Regeln
- **Bei der Ausgabe** maskieren, nicht beim Speichern (DB enthält Originalwerte).
- Gilt nicht nur für Text zwischen Tags (`<p>…</p>`), sondern **auch innerhalb von Attributwerten** (`value="…"`, `href`, `src`, `alt`).
  > Im Attribut reicht schon ein `"`, um aus dem Wert auszubrechen: `" onfocus="alert(1)` → deshalb `ENT_QUOTES`.
- Zahlen per `(int)` / `(float)` gecastet müssen nicht maskiert werden (enthalten nur Ziffern). Gilt nur nach dem Cast, nicht für Zahlen-Strings (z. B. `DECIMAL` aus der DB).
- `htmlspecialchars` ≠ Schutz vor SQL-Injection → dafür Prepared Statements.

| Angriff | Schutz | Wann |
|---|---|---|
| SQL-Injection | Prepared Statements | beim DB-Zugriff |
| XSS | `htmlspecialchars()` | bei der Ausgabe |
| CSRF | Token | beim Absenden von Formularen |

## CSRF (Cross-Site Request Forgery)

Fremde Seite schickt im Namen eines eingeloggten Users ein Formular auf deine Webseite (z. B.datensätze löschen).
**Schutz:** Geheimes Token in der Session, das jedes Formular mitschicken muss.

```php
// Token erzeugen (einmal pro Session, z. B. in includes/auth.php)
if (empty($_SESSION["csrf"])) {
    $_SESSION["csrf"] = bin2hex(random_bytes(32));
}

// Im Formular
<input type="hidden" name="csrf" value="<?= e($_SESSION["csrf"]) ?>">

// Beim Verarbeiten
if (!hash_equals($_SESSION["csrf"], $_POST["csrf"] ?? "")) {
    die("Ungültige Anfrage.");
}
```

> Daten ändernde Aktionen (Löschen, Freigeben) **nur per POST**, nicht über Links (`?delete=5`).

**Abgrenzung zum Zugriffsschutz** (zwei Ebenen, unterschiedliche Angriffe):

| Schutz | Prüft | Schützt gegen |
|---|---|---|
| Zugriffsschutz (`requireLogin` / `requireAdmin`) | **wer** sendet (Session) | Unbefugte |
| CSRF-Token | **woher** die Anfrage kommt (eigenes Formular) | fremde Seite, die den Browser eines **eingeloggten** Users missbraucht |

> Der Browser schickt das Session-Cookie automatisch mit → eine gefälschte Anfrage besteht den Login-Check. Nur das Token unterscheidet sie.

**Wo:** jedes Formular, das für eingeloggte User **Daten ändert** (Admin-Aktionen, Bestellung, Adressen). Nicht nötig bei reinen GET-Formularen (Suche, Filter).

---

# Passwörter
Sichere Speicherung und Prüfung von Passwörtern über Hashes.

Passwörter **nie im Klartext** speichern. Nur einen Hash (Einweg, nicht zurückrechenbar).

```php
// Registrierung: Hash erzeugen und speichern
$hash = password_hash($passwort, PASSWORD_DEFAULT);

// Login: Eingabe gegen gespeicherten Hash prüfen
if (password_verify($passwort, $user["password"])) {
    // korrekt
}
```

| Funktion | Aufgabe |
|---|---|
| `password_hash($pw, PASSWORD_DEFAULT)` | erzeugt Hash inkl. zufälligem Salt. Gleiches Passwort → jedes Mal anderer Hash |
| `password_verify($pw, $hash)` | prüft Passwort gegen Hash → `true` / `false` |

### Regeln
- Spalte `VARCHAR(255)`.
- Nicht selbst vergleichen (`$hash === password_hash(...)` funktioniert nie, wegen Salt).
- Kein `md5()` / `sha1()` für Passwörter.
- Login-Fehlermeldung allgemein halten: „E-Mail oder Passwort falsch", nicht verraten, welches von beiden.
- Passwort beim Validieren **nicht** `trim()`en.

---

# Session
Eine Session ist ein temporärer Datenspeicher pro Besucher, der über mehrere Seitenaufrufe erhalten bleibt.
Daten werden Serverseitig gespeichert. Im Browser liegt nur ein Cookie mit der Session-ID, keine Daten.

- `session_start()` 
  Startet eine neue Session oder setzt die bestehende fort.
  Benötigt jedes Script was `$_SESSION` benutzt. 
  Bei erstmaliges aufrufen ist `$_SESSION` leer und muss befüllt werden.
  Bei Aufruf eines späteren Scripts sind die Werte wieder verfügbar.
  Ohne `session_start()` existiert `$_SESSION` in diesem Script nicht (Zugriff → Warnung "Undefined variable").
- `$_SESSION["<VARIABLE>"]` 
  Globale Variable - Array mit Schlüsseln.
  Speichern und zugreifen der Session Variablen.
- `unset()` - (`unset($_SESSION["<VARIABLE>"])`)
  Entfernt sofort eine spezifische Variable. der Schlüssel wird gelöscht.
- `session_unset()` 
  Entfernt alle Variablen aus `$_SESSION` sofort. 
  Alle Schlüssel werden gelöscht -> `$_SESSION = [];`.
  Session bleibt die selbe.
- `session_destroy()` 
  Session ist beendet.
  `$_SESSION` existiert nach `session_destroy()` im **selben Script noch mit den alten Werten**.
  Auf der nächsten Seite -> `$_SESSION = [];`.
  nächstes `session_start()` Erzeugt eine Komplet neue Session.

Sessions Werden Beendet durch:
- session_destroy()
- **Inaktivität:** Der Server löscht die Daten nach einer gewissen Zeit ohne Aufruf.
- **Browser schließen:** Das Cookie wird standardmäßig gelöscht. Die Daten liegen dann zwar noch auf dem Server, sind aber nicht mehr erreichbar.

> `session_start()` muss **vor jeder Ausgabe** stehen (auch Leerzeichen/HTML vor `<?php`).

## Login/Logout

### Login
```php
session_start();
$errors = [];

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $email    = trim($_POST["email"] ?? "");
    $passwort = $_POST["passwort"] ?? "";

    $stmt = $conn->prepare("SELECT * FROM <user-tabelle> WHERE email = ?");
    $stmt->execute([$email]);
    $user = $stmt->fetch();

    if ($user && password_verify($passwort, $user["password"])) {
        session_regenerate_id(true);              // neue Session-ID nach Login
        $_SESSION["user_id"]  = (int) $user["id"];
        $_SESSION["is_admin"] = (bool) $user["<admin-flag>"];
        header("Location: index.php");
        exit;
    }
    $errors[] = "E-Mail oder Passwort falsch.";
}
```

> `session_regenerate_id(true)`: Schutz gegen Session-Fixation (Angreifer schiebt eine bekannte Session-ID unter).
> In der Session nur **IDs und Flags** speichern, kein Passwort/Hash.

### Zugriffsschutz (`includes/auth.php`)
```php
<?php
session_start();

function requireLogin(): void
{
    if (!isset($_SESSION["user_id"])) {
        header("Location: login.php");
        exit;
    }
}

function requireAdmin(): void
{
    requireLogin();
    if (empty($_SESSION["is_admin"])) {
        http_response_code(403);
        die("Kein Zugriff.");
    }
}
```

```php
// Ganz oben in jeder geschützten Seite
require_once __DIR__ . '/../includes/auth.php';
requireAdmin();
```

> Jede Admin-Seite **einzeln** schützen. Nur den Link zu verstecken reicht nicht, die URL ist direkt aufrufbar.
> Pfad zu `login.php` aus `admin/` heraus: `../login.php`.

### Logout
```php
session_start();
$_SESSION = [];
session_destroy();
header("Location: login.php");
exit;
```

## Weiterleitung

```php
header("Location: <ziel>.php");
exit;
```

- **Vor jeder Ausgabe** (sonst Fehler "headers already sent").
- **`exit` immer direkt danach**: `header()` beendet das Script nicht, der Code darunter läuft sonst weiter.

### Post/Redirect/Get (PRG)
Nach erfolgreichem POST **immer weiterleiten**, statt direkt eine Seite auszugeben.
Sonst sendet ein Neuladen (F5) das Formular erneut → doppelte Datensätze.
```
POST verarbeiten → Erfolg → header("Location: ...") → GET-Seite anzeigen
                 → Fehler → Formular mit Fehlern direkt anzeigen (keine Weiterleitung)
```

### Flash-Message
Meldung über eine Weiterleitung hinweg mitgeben, einmal anzeigen, dann löschen:
```php
// vor der Weiterleitung
$_SESSION["flash"] = "<Meldung>";
header("Location: <ziel>.php");
exit;

// auf der Zielseite
if (isset($_SESSION["flash"])) {
    echo "<p>" . e($_SESSION["flash"]) . "</p>";
    unset($_SESSION["flash"]);
}
```

## Warenkorb

In der Session als Array: **Produkt-ID → Menge**.
```php
$_SESSION["cart"] = [
    <produkt_id> => <menge>,
    ...
];
```

```php
$_SESSION["cart"] ??= [];                                   // initialisieren, falls leer

// Hinzufügen (Menge erhöhen)
$id = (int) $_POST["product_id"];
$_SESSION["cart"][$id] = ($_SESSION["cart"][$id] ?? 0) + 1;

// Menge setzen
$_SESSION["cart"][$id] = max(1, (int) $_POST["menge"]);

// Entfernen
unset($_SESSION["cart"][$id]);

// Leeren (nach Bestellung)
unset($_SESSION["cart"]);

// Anzahl Artikel
$anzahl = array_sum($_SESSION["cart"]);
```

Anzeige: Produktdaten zu den IDs aus der DB laden:
```php
$ids = array_keys($_SESSION["cart"]);
if (!empty($ids)) {
    $platzhalter = implode(',', array_fill(0, count($ids), '?'));
    $stmt = $conn->prepare("SELECT * FROM <produkt-tabelle> WHERE id IN ($platzhalter)");
    $stmt->execute($ids);
    foreach ($stmt->fetchAll() as $p) {
        $menge = $_SESSION["cart"][$p["id"]];
        $zeilensumme = $p["<preis>"] * $menge;
    }
}
```

### Regeln
- In der Session **nur ID + Menge**, keine Preise. Preis immer aus der DB → kann vom Benutzer nicht manipuliert werden.
- Beim Bestellen erneut prüfen: Produkt existiert noch und ist freigegeben.
- Bestellung speichern mit Transaktion (Bestellung + Positionen), Preis als **Snapshot** in die Positionen, danach Warenkorb leeren.

---

# E-Mail-Versand
Versand von Nachrichten (z. B. Bestellbestätigung, Rechnung) direkt aus PHP.

```php
$an      = "<empfaenger@domain>";
$betreff = "<Betreff>";
$text    = "<Nachricht>";
$headers = [
    "From"         => "<shop@domain>",
    "Content-Type" => "text/plain; charset=UTF-8",
];

if (mail($an, $betreff, $text, $headers)) {
    // übergeben
} else {
    // fehlgeschlagen
}
```

HTML-Mail (z. B. Rechnung):
```php
$headers["Content-Type"] = "text/html; charset=UTF-8";
$text = "<h1>Rechnung</h1><p>...</p>";        // Benutzerdaten darin mit e() maskieren
```

### Regeln
- `mail()` braucht einen **konfigurierten Mailserver** auf dem Linux-Server (z. B. `sendmail` / `postfix`). Ob das in der Prüfungs-VM eingerichtet ist, ist unklar.
- `true` heißt nur: an den Mailserver übergeben, **nicht** dass die Mail angekommen ist.
- Fällt der Versand weg: Mail-Inhalt trotzdem erzeugen und z. B. anzeigen oder in Datei schreiben (`file_put_contents()`), damit die Funktion nachweisbar ist.
- E-Mail-Adresse vorher validieren (`FILTER_VALIDATE_EMAIL`), keine Zeilenumbrüche in Betreff/Empfänger zulassen.
- Bibliotheken wie PHPMailer brauchen Composer/Internet → in der Prüfung unsicher.
