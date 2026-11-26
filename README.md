# RS – Inventarplan

Interaktiver Inventarplan für den Altbau: Altbau EG und Altbau OG, zusammen **256 Möbel und Lampen**. Jedes Möbel hat seine Inventarnummer, seinen Raum und seinen Platz im Grundriss. Der Server speichert alle Änderungen, jede Änderung steht im Verlauf und lässt sich zurücknehmen.

Ohne weitere Einstellung läuft RS_Creative auf **http://127.0.0.1:8080**, nur auf deinem eigenen Rechner und ohne Anmeldung.

*Developed by Rujahn.S*

## Funktionen

- **Grundriss** beider Etagen als Vektorzeichnung, mit Verschieben, Zoomen (Mausrad, Doppelklick, Zwei-Finger-Geste) und klickbaren Nummern
- **Suche** nach Nummer, Artikel oder Raum, dazu Filter nach acht Kategorien und eine Raumübersicht
- **Inventarliste** wie in Excel: sortierbar, pro Etage oder für alle Etagen
- **Karte pro Möbel** mit Inventarnummer, Raum und dem Hinweis, wo das Etikett klebt
- **Zu prüfen**: Einträge und Hinweise, die noch geklärt werden müssen
- **Inventar aktualisieren**: CSV-Import mit Vorschau, neue Möbel auf dem Plan platzieren, Möbel entfernen und wiederherstellen, CSV-Export
- **Möbel ausrichten**: ziehen, mit Pfeiltasten feinjustieren, Position und Drehung als Zahlen eingeben
- **Versionsverlauf**: jede Speicherung ist eine Version, „Wiederherstellen“ legt den alten Stand als neue Version an
- **3D-Vorschau** mit drei Ansichten (Gebäude, Puppenhaus, Rundgang) und drei Einrichtungsstilen
- **Als App installierbar** (PWA), die Programmhülle ist offline verfügbar
- **Anmeldung** lokal mit Benutzern oder über Microsoft Entra ID

## Starten

**Windows (ohne Installation)**

1. ZIP mit Rechtsklick → „Alle extrahieren …“ vollständig entpacken.
2. Im Ordner `start.bat` doppelt anklicken. Der Browser öffnet sich nach zwei Sekunden.
3. Das schwarze Fenster offen lassen, solange du RS_Creative nutzt.

Im Ordner `python` liegt ein portables Python. Es wird nichts installiert.

**Mac**

1. ZIP per Doppelklick entpacken.
2. `start.command` doppelt anklicken. Beim ersten Mal: Rechtsklick → „Öffnen“. Wird sie blockiert, unter Systemeinstellungen → „Datenschutz & Sicherheit“ auf „Dennoch öffnen“ klicken.
3. Es wird ein installiertes Python 3.9 oder neuer gebraucht (der Mac bietet die Installation selbst an).

Klappt der Doppelklick nicht: Im Terminal `sh ` tippen, `start.command` ins Fenster ziehen und Enter drücken.

**Linux oder direkt per Kommandozeile**

```bash
python3 server.py
```

Nötig ist nur Python 3.9 oder neuer, es gibt keine weiteren Pakete.

## Einstellungen (optional)

`.env.example` nach `.env` kopieren und anpassen.

| Schalter | Bedeutung | Standard |
|---|---|---|
| `RSC_USERS` | Benutzer als `name:passwort`, mehrere mit Komma, Passwort mindestens 8 Zeichen | leer (nur lokal) |
| `RSC_HOST` | `127.0.0.1` = nur dieser Rechner, `0.0.0.0` = im Netz (dann sind Benutzer Pflicht) | `127.0.0.1` |
| `RSC_PORT` | Port. Ist 8080 belegt, z. B. `8081` eintragen | `8080` |
| `RSC_DATA` | Ordner der Datenbank (diesen Ordner sichern) | `./data` |
| `RSC_MAX_VERSIONS` | Wie viele Versionen im Verlauf bleiben | `500` |
| `RSC_TRUST_PROXY` | Nur `1`, wenn ein Reverse Proxy mit HTTPS davorsteht | `0` |
| `RSC_HTTPS` | `1` setzt das Secure-Merkmal am Cookie | `0` |
| `RSC_QUIET` | `1` = nur Fehler und Anmeldungen protokollieren | aus |
| `RSC_LOG` | Protokolldatei, ab 5 MB wird zu `.1` rotiert | aus |
| `RSC_BACKUP_KEEP` | Wie viele tägliche Sicherungen in `data/backups` bleiben | `30` |

Sollen mehrere Personen im Netz zugreifen, sind Benutzer Pflicht. Ohne Benutzer startet der Server nur auf `127.0.0.1`.

### Anmeldung mit Microsoft Entra ID

Mit `RSC_AUTH` wählst du die Anmeldeart:

- `lokal` (Standard): eigene Anmeldeseite mit `RSC_USERS`
- `oidc` (empfohlen): RS_Creative meldet selbst bei Microsoft Entra ID an. Nötig sind `RSC_OIDC_TENANT`, `RSC_OIDC_CLIENT_ID`, `RSC_OIDC_CLIENT_SECRET` und genau eine `RSC_PUBLIC_URL` (https, nur localhost darf http sein).
- `entra`: Die Anmeldung übernimmt eine vorgeschaltete Schicht (Azure App Service, Entra Application Proxy oder ein anderer Reverse Proxy). RS_Creative liest den Benutzer aus einer Kopfzeile.

Mit `RSC_EDIT_ROLE` dürfen nur Personen mit dieser Rolle bearbeiten, alle anderen sehen den Plan nur an. Alle Schalter stehen mit Erklärung in `.env.example`, die Einrichtung steht in `ENTRA.md`.

## Daten und Sicherung

Die Datenbank liegt in `data/rs-creative.db` (der Ordner entsteht beim ersten Start).

```bash
python server.py backup sicherung.db      # Sicherung in eine Datei
python server.py backup                   # Sicherung nach data/backups, die letzten 30 bleiben
python server.py logout-all               # alle Sitzungen beenden
```

Zusätzlich lädt `/api/export.csv` die Inventarliste als CSV (Semikolon, UTF-8).

## Bearbeiten

- Änderungen werden nach etwa einer Sekunde automatisch gespeichert.
- Unter „Inventar aktualisieren“ steht der Versionsverlauf. „Wiederherstellen“ legt den alten Stand als neue Version an, nichts geht verloren.
- Hat jemand anderes inzwischen gespeichert, erscheint eine Meldung. Es wird nie still überschrieben.
- Neue Möbel kommen per CSV (`Inventarnummer; Bezeichnung; Kategorie; Etage; Raum`). Du siehst vorher eine Vorschau und platzierst sie danach auf dem Plan.

## App installieren

Chrome und Edge: Installieren-Symbol in der Adressleiste. iPhone und iPad (Safari): Teilen → „Zum Home-Bildschirm“. Das geht nur über HTTPS oder `localhost`.

## Projektstruktur

```
rs-creative/
├── server.py                  Server (nur Python-Standardbibliothek, SQLite)
├── start.bat                  Start unter Windows
├── start.command              Start unter Mac
├── .env.example               Einstellungen, nach .env kopieren
├── ENTRA.md                   Anleitung Microsoft Entra ID
├── app/
│   ├── index.template.html    Oberfläche (HTML, CSS, JavaScript, three.js, in einer Datei)
│   ├── seed.json              Startdaten: alle Möbel mit Nummer, Raum, Position
│   ├── static.json            Etagen, Räume und Wandzeichnung
│   ├── manifest.webmanifest   App-Beschreibung
│   ├── sw.js                  Service Worker
│   └── icons/                 App-Symbole und Logo
├── python/                    portables Python für Windows (nicht im Git, nur im Release-ZIP)
└── data/                      Datenbank (entsteht beim Start, nicht im Git)
```

## Kategorien

Schreibtisch, Bürostuhl, Stuhl, Tisch, Schrank und Ablage, Sofa und Sessel, Lampe, Küche und Theke.

## Schnittstellen

| Methode | Pfad | Zweck |
|---|---|---|
| GET | `/api/health` | Lebenszeichen |
| GET | `/api/version` | aktuelle Version |
| GET | `/api/me` | Benutzer, Bearbeitungsrecht, Anmeldeart |
| GET | `/api/inventory` | aktueller Stand |
| PUT | `/api/inventory` | neuen Stand speichern (mit `baseVersion`, bei Konflikt `409`) |
| GET | `/api/history` | Versionsliste |
| GET | `/api/history/<version>` | ein alter Stand |
| POST | `/api/restore/<version>` | alten Stand als neue Version anlegen |
| GET | `/api/export.csv` | Inventarliste als CSV |

## Sicherheit

- Nur Standardbibliothek, keine fremden Pakete
- Strenge Prüfung jedes gespeicherten Stands (Felder, Kategorien, Positionen, Größe)
- Content-Security-Policy erlaubt nur eigene Quellen, nichts wird von fremden Servern geladen
- Same-Origin-Prüfung bei allen schreibenden Anfragen
- Sitzungen mit zufälligem Token, in der Datenbank nur als Hash, Cookie `HttpOnly` und `SameSite`
- Sperre nach 8 Fehlversuchen in 5 Minuten pro IP
- Ohne Benutzer nur lokal erreichbar

## Nachbauen

Die Arbeitsschritte als fertige Prompts stehen in [`PROMPTS.md`](PROMPTS.md).

---

© RS_Creative · Developed by Rujahn.S
