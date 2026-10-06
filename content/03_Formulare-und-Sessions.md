# Formulardaten 
($_GET / $_POST)

---

# Datei-Upload
(Produktbild)

---

# Validierung

---

# Ausgabe absichern

---

# Passwörter
(hash / verify)

---

#
---

#

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
- `unset()` - (`unset($_SESSION["<VARIABLE>"])`)
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

## Login/Logout

## Weiterleitung

## Warenkorb

---

# E-Mail-Versand