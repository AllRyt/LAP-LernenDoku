## Display Errors

### Entwickeln und Testen
Für Fehler im **Code**, die PHP selbst erkennt:
- Syntaxfehler (Parse error)
- Laufzeitfehler (Fatal error)
- Warnung (Warning)
- Veraltet (Deprecated)

Zu beginn jedes Scripts(am besten in einer Datei(debug.php) via `require_once` einbinden):
```php
ini_set('display_errors', '1');          // PHP-Fehler im Browser anzeigen statt weißer Seite
ini_set('display_startup_errors', '1');  // auch Fehler beim Start von PHP anzeigen (selten relevant)
error_reporting(E_ALL);                  // alle Fehlerarten melden, auch Warnungen und Hinweise
```

> Niemals im Live-Betrieb!

---
# Includes
Fügen den Inhalt der Datei an dieser Stelle ins Script ein.

## Aufruf 
`<Statement> '<FileName.php>';` /  `<Statement> __DIR__ . '<Path/FileName.php>';`

> `__DIR__`: Ordner der aktuellen Datei.

## Varianten

| Statement      | Datei fehlt                      | Mehrfach aufgerufen            |
| -------------- | -------------------------------- | ------------------------------ |
| `include`      | Warnung, Script **läuft weiter** | wird jedes Mal neu eingebunden |
| `include_once` | Warnung, Script **läuft weiter** | nur beim ersten Mal            |
| `require`      | Fehler, Script **bricht ab**     | wird jedes Mal neu eingebunden |
| `require_once` | Fehler, Script **bricht ab**     | nur beim ersten Mal            |
## Usecases: 
`require` /`require_once`: Für alles, ohne das die Seite nicht funktioniert: Klassen, Funktionen, Config, DB-Verbindung.

`include`/`include_once`: Für Seitenbausteine, die mehrfach vorkommen dürfen oder deren Fehlen die Seite nicht zerstört, z. B. Header und Footer:

Das `_once` ist bei Klassen und Funktionen wichtig.
Wird dieselbe Datei zweimal eingebunden, bricht PHP mit „Cannot redeclare" ab.

---
# Objekte / Klassen

## Grundbegriffe

Klassendefinitionen beginnen mit `class`.
Klassen sind Blaupausen für Objekte.
Klassen mit nur statischen Methoden dienen als Sammlung von Hilfsfunktionen.

Instanziierung (Erzeugung eines Objekts):
- Ohne Konstruktor-Parameter: `$obj = new <Klassenname>();`
- Mit Konstruktor-Parametern: `$obj = new <Klassenname>(<Eingebe Variable>,...);`

## Aufbau einer Klasse

```php
<?php 
class SimpleClass { 
	// Deklaration einer Variablen Eigenschaft 
	private <Datentyp> $var; 
	const CONSTANT = 'Konstanter Wert';
	
	// Constructor: Initialisier Object
	public function __construct(<Datentyp> $x){
		//initiallisierung der Variablen Eigenschaft
		$this->var = $x;
	}
	
	// Deklaration einer Methode 
	public function displayVar() { 
		echo $this->var;
	}
	
	// Deklaration einer Getter-Methode 
	public function GetVar(): <Datentyp von $var> { 
		return $this->var;
	}
} 
?>
```

### Methoden-Deklaration

`<Access Modifier> function <MethodenName>(<DatenTyp> $<VariablenName>, ...): <RückgaTyp> {...}`

>Namenskonvention: Methoden üblicherweise in camelCase.

### `$this`

`$this` ist eine Pseudovariable.
- Ist verfügbar, wenn eine Methode aus einem Objektkontext heraus aufgerufen wird.
- Referenz auf das aufgerufene Objekt.

## Access Modifier

|Modifier|Zugriff von|
|---|---|
|`public`|überall|
|`protected`|eigene Klasse + Unterklassen|
|`private`|nur eigene Klasse|

| Wo                           | Modifier                                       | 
| -----------------------------| ---------------------------------------------- |
| Eigenschaft in einer Klasse  | **Pflicht** (`public`, `private`, `protected`) |
| Methode                      | optional, ohne Angabe gilt `public`            |

## Zugriff auf Objekte

Aufruf einer Eigenschaft eines Objektes: `$<Onjekt>-><EigenschaftName>;`

Aufruf einer Methode:
- Normale Methode: `$<Onjekt>-><Methode>(<eingabe Variablen>,...);` 
- Statische Methode: `<Klasse>::<Methode>(<eingabe Variablen>,...);`

> Der zugriff vom `$obj->var ...` (nur bei `public`). 

## Statisch (`static`)

Klasseneigenschaften oder -methoden als statisch zu deklarieren, macht diese zugänglich, ohne dass man die Klasse instantisieren muss.

Statische Eigenschaft `Klasse::$eigenschaft` (mit `$`), Konstante `Klasse::CONSTANT`. Innerhalb der Klasse `self::` statt `$this->`.

Auf statische Eigenschaften kann nicht über den Objektoperator (`->`) darauf zugegriffen werden.


## Vererbung

```php
<?php 
class <BaseClass> { 
	protected $Eigenschaft1;
	
	function __construct($Imput1) {           // BaseClass-Konstruktor
		$this->Eigenschaft1 = $Imput1;
	} 
} 

class <SubClass> extends <BaseClass> { 
	Private $Eigenschaft2;
	
	function __construct($Imput1, $Imput2) {  // SubClass-Konstruktor
		parent::__construct($Imput1); 
		$this->Eigenschaft2 = $Imput2;
	} 
} 

$BaseObj = new <BaseClass>($Imput1);           // ruft BaseClass-Konstruktor auf
$SubObj = new <SubClass>($Imput1, $Imput2);    // ruft SubClass-Konstruktor auf
?>
```