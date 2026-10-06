
# Ausgangspunkt 

**Ausgangspunkt der Praxisprüfung**  
Man arbeitet an einem Prüfungs-PC mit vorinstalliertem **VirtualBox** und zwei VMs. 
Für beide VMs bekommt man Zugangsdaten.

## Linux-VM

Die **Ubuntu-Server-VM** fungiert als webserver und stellt den LAMP-Stack bereit. 
Man startet sie nur und meldet sich an.
**LAMP** läuft automatisch über Systemdienste(systemd) auf Ubuntu.
> LAMP = **L**inux + **A**pache + **MariaDB + **P**HP

Dienste Prüfen:
```Linux
  systemctl status apache2
  systemctl status mysql         # bzw. mariadb
  sudo systemctl start apache2   # falls gestoppt
```

## Windows-VM

Die **Windows-VM** ist **WinSCP** (Dateiübertragung).
Ein Editor der eigenen Wahl kann nachinstalliert werden, wobei die dafür
erforderliche Installationszeit zur Prüfungszeit zählt.
 
Von Windows aus verbindet man sich über WinSCP per **SFTP** mit dem Linux-Server.
```
Host: [IPv4 des Ubuntu-Servers]               # In Linux: hostname -I
Port: 22(oder bewust anders Konfiguriert)     # In Linux: sudo ss -tlnp | grep ssh
Benutzer und Passwort: in Zugangsdaten 
```

**links (lokal):** dein Projektordner auf Windows
**rechts (remote):** das Zielverzeichnis im Webroot   (typischerweise `/var/www/html/...`)
### Daten auf Webserver aktuell halten
**Optional**
In WinSCP Nach verbinden zum Webserver:
Befehle(Menü-Leiste) → Entferntes Verzeichnis aktuell halten

## Datenbank Berechtigung

unter `http://<IPv4 des Ubuntu-Servers>/phpmyadmin` -> `Benuterkonten` muss ein Datenbank-User Erstellt werden.
```
host= localhost ODER <IPv4 des Ubuntu-Servers> 
Rechte: Daten(SELECT, INSERT, UPDATE, DELETE) + Struktur(CREATE, ALTER, DROP, INDEX …)
```

Der Datenbank-User-Account sind die Zugangsdaten für die Datenbankverbindung der PHP-Anwendung.

```php
$pdo = new PDO('mysql:host=localhost;dbname=<db>;charset=utf8mb4', '<user>', '<pw>');
```


---

# Prüfungsziel

Webshop mit **Datenbank** (SQL), **Backend** (PHP) und **Frontend** (HTML).
CSS und JavaScript nicht gefordert.

## Datenbank
- Skizze des Datenbank-Designs
- Datenanbindung via Datenbank
  
## Webseiten Inhalte
- Produkte mit Produktdetails
- Produktliste: Name, Beschreibung, Bild
- (Niedrige Priorität) Nutzbar auf beliebigen Geräten

## Verwaltung (Admin)
- Sicherer Login / Logout
- Produkt hinzufügen
- Produkte freigeben / bearbeiten / löschen
- Validierung gegen Fehleingaben und verhindern Doppelte Eingaben (z. B. Artikelcode)
- (niedrige Priorität) Statistiken zu ...

## User Interface (Kunde)
Einfaches, sicheres, logisches User Interface
- Registrierung / Login / Logout 
- Warenkorb
- Bestellvorgang: 
  - Rechnungsadresse, Lieferadresse
  - Zahlungsart: Rechnung / Kreditkarte
  - Rechnungserstellung
  - E-Mail-Versand
- Validierung gegen Fehleingaben


---

# Verzeichnisstruktur

```
/var/www/
├── config.php                  # DB-Zugangsdaten (außerhalb Webroot)
└── html/shop/
    ├── index.php               # Produktliste
    ├── produkt.php             # Produktdetails
    ├── registrieren.php
    ├── login.php               # Kunde + Admin
    ├── logout.php
    ├── warenkorb.php
    ├── bestellung.php          # Adressen, Zahlungsart, Abschluss
    ├── rechnung.php
    │
    ├── admin/
    │   ├── index.php           # Produktübersicht: freigeben / löschen
    │   ├── produkt_form.php    # hinzufügen / bearbeiten
    │   └── statistik.php
    │
    ├── classes/
    │   ├── Database.php
    │   ├── Product.php
    │   ├── User.php
    │   ├── Order.php
    │   └── Validator.php
    │
    ├── includes/
    │   ├── debug.php
    │   ├── header.php
    │   ├── footer.php
    │   └── auth.php            # Login-/Admin-Prüfung
    │
    └── uploads/                # Produktbilder
```