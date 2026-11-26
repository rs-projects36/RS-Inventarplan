# RS-Inventarplan – Prompts zum Nachbauen

Diese Datei enthält die Arbeitsschritte als fertige Prompts, in der Reihenfolge, in der RS - Inventarplan 1:1 entsteht. Nach jedem Schritt kurz testen, dann erst den nächsten schicken.

**Ergebnis:** Ein interaktiver Inventarplan mit Grundriss, Suche, Verlauf und 3D-Ansicht. Er läuft als kleiner Python-Server (nur Standardbibliothek, SQLite) und hat 256 Möbel und Lampen auf zwei Etagen.

**Zutaten vorher bereithalten**

- Die Grundrisse beider Etagen als PDF oder Bild
- Die Inventarliste als Excel oder CSV (Inventarnummer, Bezeichnung, Kategorie, Etage, Raum)
- Das Logo als PNG
- Python 3.9 oder neuer auf dem Rechner

---

## Schritt 1 – Projektgerüst

```
Lege ein Projekt "rs-creative" an. Ordner: app/ (Oberfläche und Daten), app/icons/, python/ (portables Python für Windows, später).
Dateien: server.py, start.bat, start.command, .env.example, README.md.
Regeln für das ganze Projekt:
- Nur Python 3.9+ und die Standardbibliothek, kein pip install.
- Die Oberfläche ist eine einzige HTML-Vorlage (app/index.template.html) mit eingebettetem CSS und JavaScript, ohne CDN und ohne externe Dateien.
- Alle Texte der Oberfläche auf Deutsch.
- Im Code und in der Vorlage steht "Developed by Rujahn.S".
Erstelle zuerst nur das leere Gerüst mit Kommentaren, was in welche Datei kommt.
```

## Schritt 2 – Datenmodell und Startdaten (seed.json)

```
Ich lade meine Inventarliste hoch. Wandle sie in app/seed.json um.
Format: {"items":[...], "removedItems":[]}.
Jeder Eintrag hat diese Felder:
- k: eindeutiger Schlüssel, Etagencode + "-" + Nummer, z. B. "ABEG-12"
- f: Etagencode (ABEG = Altbau EG, ABOG = Altbau OG)
- n: laufende Nummer auf dem Plan (Zahl)
- a: Bezeichnung, z. B. "Tisch"
- c: Kategorie, genau eine von: schreibtisch, buerostuhl, stuhl, tisch, schrank, sofa, lampe, kueche
- r: Raum (Text, z. B. "3", "L", "S")
- inv: Inventarnummer als Text, sechsstellig, beginnend bei 100001
- x, y: Position auf dem Plan in Prozent (0 bis 100, von links oben)
- optional: ang (Drehung in Grad), seat ("chair" oder "arm" bei Sitzmöbeln), cabinet ([x, y, breite, höhe] in Plan-Einheiten bei Schränken), note (Hinweis "Zu prüfen")
- lx, ly, nl, layoutBase: ursprüngliche Plan-Lage, damit man später zurücksetzen kann
Es sind insgesamt 256 Einträge auf zwei Etagen. Melde mir Dubletten bei Schlüssel oder Inventarnummer, statt sie still zu korrigieren.
Gib mir am Ende eine Zählung pro Etage und pro Kategorie.
```

## Schritt 3 – Etagen und Räume (static.json)

```
Lege app/static.json an: {"floors":[...], "notes":[]}.
Pro Etage: code, name, w und h (Breite und Höhe der Zeichnung in Plan-Einheiten, passend zum Seitenverhältnis meines Grundrisses), rooms (Liste mit code, x, y, w: Position des Raumschilds in Prozent und Breite des Schilds) und die Zeichnung selbst.
Die Zeichnung wird aus meinem Grundriss-PDF als Vektor übernommen: Wände, Türen, Fenster, feste Einbauten. Möbel gehören NICHT in die Wandzeichnung, sie werden später aus den Inventardaten gezeichnet.
Räume: Zahlen für Büros, Buchstaben für besondere Räume (L Lounge, S Besprechung). Zeige mir die Raumschilder in einer Vorschau, bevor du weitermachst.
```

## Schritt 4 – Grundriss-Ansicht

```
Baue in app/index.template.html die Hauptansicht.
Kopfzeile: Logo, Titel "Inventarplan", Untertitel mit Zählung (Möbel und Lampen), Etagen-Umschalter als Tabs.
Hauptfläche: Grundriss als SVG.
- Verschieben per Ziehen, Zoomen per Mausrad, Doppelklick und Zwei-Finger-Geste. Drei Knöpfe: Vergrößern, Verkleinern, Ganzer Plan.
- Jedes Möbel ist ein klickbarer Marker mit seiner Nummer. Sitzmöbel als kleine runde Marker, Tische und Schreibtische als Pillen, Lampen als Kreis mit Kreuz, Schränke als Rechtecke.
- Raumschilder als Pillen mit "Raum X".
- Farben nach Kategorie: Schreibtisch #6B6B6B, Bürostuhl #1F1F1F, Stuhl #D4D4D4, Tisch #6AC169, Schrank #A6A6A6, Sofa #F0567F, Lampe #FFFFFF, Küche #000000.
- Auswahl hebt das Möbel hervor und blendet eine Verbindungslinie zur Karte ein.
Die Daten kommen über den Platzhalter /*RSC_DATA*/null in der Vorlage (genau einmal im Skript, `const D = /*RSC_DATA*/null;`). Der Server ersetzt ihn später durch JSON.
Tastatur: Tab und Enter wählen Nummern, die Taste / springt in die Suche. Beachte "prefers-reduced-motion".
Hell/Dunkel nicht nötig, Schrift ist die Systemschrift.
```

## Schritt 5 – Seitenleiste

```
Ergänze rechts (auf dem Handy unter dem Plan) eine Seitenleiste:
1. Suchfeld "Nummer, Artikel oder Raum" mit Trefferliste (max. 12, danach Hinweis "weitere Treffer, Suche genauer eingeben").
2. Kategorien als Filter-Chips mit Farbpunkt, Anzahl und Haken. Klick blendet die Kategorie aus oder ein. Knopf "Alle zeigen".
3. Räume: Tabs "Räume" und "Zu prüfen" (mit Zähler). Räume zeigen ihre Möbel, Klick auf einen Raum zoomt hinein. "Zu prüfen" listet Einträge mit Hinweis (note) und allgemeine Hinweise (notes) mit "Im Plan zeigen".
4. Inventarliste wie eine Excel-Tabelle: Spalten Inv.-Nr., Büro, Stockw., Artikel. Sortierbar per Klick auf die Spaltenköpfe, Umschalter "Diese Etage" und "Alle Etagen", Gruppenköpfe pro Etage und Raum. Klick auf eine Zeile wählt das Möbel und zoomt hin.
5. Kurzer Hilfetext am Ende und die Zeile "Developed by Rujahn.S".
Räume werden so sortiert: erst Buchstaben, dann Zahlen aufsteigend.
```

## Schritt 6 – Detailkarte mit Klebeposition

```
Beim Auswählen eines Möbels erscheint eine Karte mit: Inventarnummer, Bezeichnung, Kategorie, Raum und Etage, und einem Hinweis, wo das Inventaretikett klebt.
Die Klebeposition kommt aus einer Tabelle INV_POS, abhängig von Kategorie und Bezeichnung:
- Bürostuhl: An der Seite oder unter dem Stuhl
- Stuhl und Hochstuhl: Unten am Stuhl
- Tisch und Schreibtisch: Unten, vorne oder an der Seite des Tisches
- Lampe: Oben an der Lampe
- Sideboard: Zwischen Schreibtisch und Sideboard
- Theke: Unten an der Theke
- Schrank: Vorne oder an der Seite
- Sessel: Unten am Sessel
- sonst: Noch nicht hinterlegt
Die Tabelle steht an einer Stelle im Code und ist leicht änderbar.
Die Auswahl steht in der Adresse (history.replaceState), damit ein Link wieder dasselbe Möbel zeigt.
```

## Schritt 7 – Server und Speicherung

```
Schreibe server.py mit ThreadingHTTPServer und SQLite (WAL, synchronous NORMAL), nur Standardbibliothek.
Tabelle versions(version, updated_at, user, note, summary, payload). Payload ist {"items","removedItems"} als JSON, zlib-komprimiert.
Beim ersten Start wird seed.json geprüft und als Version 1 ("Erster Stand aus der Vorlage", Benutzer "System") gespeichert.
Strenge Prüfung (validate_snapshot) bei jedem Speichern:
- items und removedItems sind Listen, zusammen höchstens 10000 Einträge
- k eindeutig und höchstens 60 Zeichen, f muss eine bekannte Etage sein, c eine der acht Kategorien, a nicht leer, n eine Zahl
- x und y zwischen -5 und 105, außer bei placed=false
- Schachtelungstiefe begrenzt, Body höchstens 8 MB
Seiten-Auslieferung: Die Vorlage wird gelesen, /*RSC_DATA*/null wird durch static.json plus aktuelle Daten plus Serverinfo (Version, Benutzer, canEdit) ersetzt. "<" wird als \u003c maskiert. Antwort gzip-komprimiert, Seiten im Cache nach Version und Benutzer.
Sicherheitskopfzeilen: Content-Security-Policy mit default-src 'self', script-src 'self' 'unsafe-inline', style-src 'self' 'unsafe-inline', img-src 'self' data: blob:, connect-src 'self', frame-ancestors 'none', base-uri 'none'.
Konfiguration nur über Umgebungsvariablen RSC_* und eine .env-Datei: RSC_HOST (Standard 127.0.0.1), RSC_PORT (8080), RSC_DATA (./data), RSC_MAX_VERSIONS (500).
Zeige mir die Endpunkte vor dem Schreiben des Codes als Tabelle.
```

## Schritt 8 – API, Verlauf und Konflikte

```
Baue diese Endpunkte:
- GET /api/health → {"ok":true}
- GET /api/version → aktuelle Version, Zeit, Benutzer
- GET /api/me → Benutzer, canEdit, Rollen, Modus
- GET /api/inventory → aktueller Stand
- GET /api/history?limit=50 (1 bis 200) → Liste mit Version, Zeit, Benutzer, Notiz, Zusammenfassung
- GET /api/history/<version> → ein alter Stand
- PUT /api/inventory → speichert einen neuen Stand. Der Body enthält baseVersion. Ist die Basis nicht die aktuelle Version, antwortet der Server mit 409 und der aktuellen Version, es wird nie still überschrieben. Mit ?force=1 wird bewusst überschrieben (Notiz "Überschrieben").
- POST /api/restore/<version> → legt den alten Stand als NEUE Version an (Notiz "Wiederhergestellt aus Version N"), nichts wird gelöscht
- GET /api/export.csv → Inventarliste, Semikolon, UTF-8 mit BOM, alle Felder in Anführungszeichen, Zellen die mit = + @ - beginnen bekommen ein Apostroph davor. Spalten: Inventarnummer, Bezeichnung, Kategorie, Etage, Raum
- DELETE ist nicht erlaubt (405)
Jede Version bekommt eine automatische Zusammenfassung: "3 geändert, 1 neu oder wiederhergestellt, 2 entfernt" (Vergleich nach Wert, 12 und 12.0 sind gleich). Ältere Versionen über RSC_MAX_VERSIONS hinaus werden gelöscht.
Speichern läuft in einer Transaktion (BEGIN IMMEDIATE), damit zwei gleichzeitige Speicherungen sich nicht überholen.
```

## Schritt 9 – Autospeichern und Verlaufsanzeige in der Oberfläche

```
Verbinde die Oberfläche mit dem Server:
- Änderungen werden nach etwa einer Sekunde automatisch gespeichert (PUT mit baseVersion).
- Statusanzeige: "Gespeichert", "Speichert …", "Nicht gespeichert", Fehler.
- Bei 409 erscheint eine Meldung mit Wahl: Neuen Stand laden oder bewusst überschreiben. Es wird nie still überschrieben.
- Ein Knopf "Inventar aktualisieren" öffnet einen Dialog, darin der Versionsverlauf (Version, Zeit, Benutzer, Zusammenfassung) mit "Wiederherstellen" pro Zeile und der Frage "wiederherstellen? Der aktuelle Stand bleibt im Verlauf erhalten."
- Bei Leserechten (canEdit=false) sind alle Bearbeiten-Knöpfe ausgeblendet.
- Ungespeicherte Änderungen werden zusätzlich im localStorage gesichert (Schlüssel rs_creative.inventory.v1, schema 1, pending, baseVersion) und beim Laden streng geprüft, bevor sie übernommen werden.
```

## Schritt 10 – Anmeldung (lokal) und Schutz

```
Baue die lokale Anmeldung:
- Ohne RSC_USERS läuft die App ohne Anmeldung, aber nur auf 127.0.0.1. Sobald RSC_HOST etwas anderes ist (z. B. 0.0.0.0), sind Benutzer Pflicht, sonst bricht der Start mit einer klaren Meldung ab.
- RSC_USERS=name:passwort,name2:passwort2, Passwort mindestens 8 Zeichen.
- Eigene Anmeldeseite (Deutsch) mit "Angemeldet bleiben". Sitzung 12 Stunden, mit Haken 30 Tage.
- Sitzungstoken zufällig (secrets.token_urlsafe(32)), in der Datenbank nur als SHA-256-Hash gespeichert. Cookie rsc_session: HttpOnly, SameSite=Strict, Secure wenn RSC_HTTPS=1.
- Passwortvergleich mit hmac.compare_digest, auch bei unbekanntem Benutzer (gleiche Zeit).
- Nach 8 Fehlversuchen in 5 Minuten pro IP sperren, Meldung "Zu viele Fehlversuche."
- Same-Origin-Prüfung bei allen schreibenden Anfragen (Origin muss zum Host passen).
- Weiterleitung nach Login nur auf eigene Pfade (kein //, kein \, nicht /login oder /logout).
- Abmelden per POST /logout.
- RSC_TRUST_PROXY=1 nur hinter einem Reverse Proxy, dann zählt X-Forwarded-For und X-Forwarded-Host.
- Kommandos: python server.py logout-all beendet alle Sitzungen.
```

## Schritt 11 – Inventar aktualisieren: Import, Platzieren, Entfernen

```
Erweitere den Dialog "Inventar aktualisieren":
1. CSV-Import (Semikolon oder Komma). Erkannte Spaltennamen, egal ob Groß/Klein und mit oder ohne Umlaute und Leerzeichen: Inventarnummer/InvNr/Inv, Bezeichnung/Artikel, Kategorie, Etage/Stockwerk, Raum/Büro-Nr. Jede dieser fünf Spalten muss genau einmal vorkommen. Höchstens 5000 Zeilen. Ein Apostroph vor = + @ - wird entfernt.
2. Vorschau vor dem Übernehmen als Tabelle (Aktion, Inv.-Nr., Bezeichnung, Etage/Raum), mit Zählung "N neue Möbel · N Änderungen · N Wiederherstellungen". Fehler nennen die Zeile, z. B. "Zeile 7: Die Anzahl der Spalten stimmt nicht."
3. Prüfregeln pro Zeile:
   - Inventarnummer: 1 bis 40 Zeichen, erlaubt sind Buchstaben, Ziffern, Punkt, Bindestrich, Unterstrich, Schrägstrich. Doppelte Nummern in der Datei sind ein Fehler.
   - Bezeichnung: 1 bis 160 Zeichen.
   - Kategorie: Kennung oder Klartext (z. B. "Bürostuhl"), Groß/Klein egal. Unbekannt ist ein Fehler.
   - Etage: Code oder Name (ABEG oder "Altbau EG"). Unbekannt ist ein Fehler.
   - Raum: "Raum " am Anfang wird entfernt, Großbuchstaben. Der Raum muss in dieser Etage existieren.
4. Aktionen: "Neu", "Aktualisieren", "Gleich" und "Wiederherstellen" (wenn die Nummer unter den entfernten Möbeln steht). Vorhandene Möbel werden nie gelöscht. Bei einem Möbel, das schon im Plan gezeichnet ist, darf nur die Bezeichnung geändert werden. Weichen Etage, Raum oder Kategorie ab, ist das ein Fehler mit dem Hinweis, Standort oder Kategorie separat zu bearbeiten.
5. Neue Möbel bekommen den Schlüssel "NEU-<n>" (n = nächste freie Nummer), added=true, placed=false und noch keine Position. Sitzmöbel (Stuhl, Bürostuhl) bekommen seat="chair". Sie erscheinen im Kasten "Möbel noch nicht platziert" mit "Neue Möbel platzieren".
6. Platzieren: Klick auf den Plan setzt das Möbel, alternativ "In Planmitte platzieren" oder "Abbrechen". Neue Möbel werden mit einem passenden Symbol gezeichnet (Stuhl, Tisch, Schrank, Lampe, Sofa) und erscheinen auch in der 3D-Ansicht.
7. Entfernen: Der Knopf "Entfernen" verschiebt ein Möbel nach removedItems. Dort steht eine Liste mit "Wiederherstellen". Es wird nichts hart gelöscht.
8. Nach dem Import werden die Raumlisten aus den neuen Daten ergänzt.
9. Im selben Dialog: "Inventarliste als CSV laden" im Format von /api/export.csv (Semikolon, UTF-8 mit BOM, alle Felder in Anführungszeichen, Apostroph vor = + @ -).
10. Läuft die Seite ohne Server (Datei direkt geöffnet), speichert "Plan speichern" stattdessen eine neue HTML-Datei mit den aktuellen Daten zum Herunterladen.
```

## Schritt 12 – Möbel genau ausrichten

```
Ergänze eine Leiste "Möbel verschieben und ausrichten":
- Starten über die Karte eines Möbels. Das Möbel lässt sich ziehen oder durch Klick auf den Zielpunkt setzen. Ziehen auf freier Fläche verschiebt weiterhin den Plan.
- Pfeiltasten verschieben fein, Umschalt + Pfeil zehnfach. Vier Pfeilknöpfe, Auswahl "Schrittweite" (Sehr fein 0,1 / Fein 1 / Grob 5).
- Felder X (%), Y (%), Drehung (°) mit Zahlen von 0 bis 100 beziehungsweise 0 bis 360, Schrittweite 0,001 beziehungsweise 0,01. Knöpfe ↶ 15° und ↷ 15°.
- Knöpfe "Übernehmen", "Plan speichern", "Abbrechen". Abbrechen stellt die Ausgangslage wieder her.
- Position ist in Prozent von der linken oberen Planecke, keine Zentimeter. Die Raumzuordnung bleibt gleich.
- Die Änderung wirkt sofort im Plan und in der 3D-Ansicht.
```

## Schritt 13 – 3D-Ansicht

```
Baue eine 3D-Ansicht "3D Vorschau" (Knopf neben dem Titel, Knopf "Zum Grundriss" zum Zurückkehren).
Technik: three.js r128 und OrbitControls direkt in die HTML-Vorlage eingebettet (als eigene <script>-Blöcke vor dem App-Skript). Kein CDN, die Content-Security-Policy erlaubt nur 'self'.
Die Wände werden aus den Vektordaten der Grundrisse (Außenlinien und Löcher) extrudiert und als MODEL3D im Skript abgelegt, ebenso die Möbelliste (m3: Kisten, Zylinder, Stühle, Schränke, Lampen).
Drei Modi, umschaltbar:
1. Gebäude: beide Etagen übereinander, automatisch drehend (nicht bei prefers-reduced-motion), Etagenschilder, Klick auf eine Etage öffnet sie als Grundriss.
2. Puppenhaus: eine Etage in echter Raumhöhe von oben, Decke ausgeblendet.
3. Rundgang: Ich-Perspektive mit Maus und Tasten (W A S D, Pfeile, Mauszeiger sperren), Kollision mit Wänden und Möbeln.
Drei Einrichtungsstile zur Wahl: "Standard" (grau, Teppich), "Skandinavisch" (Holz hell, grüner Akzent), "Warm" (Holz dunkel). Böden und Tischplatten bekommen kleine Texturen aus einem Canvas.
Möbel und Lampen sind anklickbar (Raycaster) und öffnen im Grundriss die passende Karte. Neu platzierte oder verschobene Möbel erscheinen sofort.
Falls WebGL nicht verfügbar ist, zeige eine freundliche Meldung statt eines Fehlers.
```

## Schritt 14 – App-Installation (PWA)

```
Mach die Seite installierbar:
- app/manifest.webmanifest: name "Inventarplan", short_name "RS", lang de, start_url "/", display standalone, background_color und theme_color #000000, Icons 192, 512 und maskable 512.
- app/sw.js: Service Worker mit Cache "rs-creative-shell-v4". Beim Installieren Manifest und die zwei Icons cachen. API-Aufrufe (/api/) nie cachen, Daten kommen immer vom Server. Seitenaufrufe zuerst übers Netz, bei Offline die zuletzt geladene Startseite, sonst "Offline. Bitte später erneut öffnen." (503).
- Symbole in app/icons/: icon-192.png, icon-512.png, icon-maskable-512.png, apple-touch-icon.png und logo.png (aus meinem Logo erzeugen).
- Meta-Tags für iPhone (apple-mobile-web-app-capable, Titel, Statusleiste) und viewport-fit=cover.
- Der Server liefert /manifest.webmanifest und /sw.js ohne Anmeldung aus (no-cache, Service-Worker-Allowed: /), die Icons mit einem Tag Cache.
Hinweis in der README: Installieren geht nur über HTTPS oder localhost.
```

## Schritt 15 – Startskripte und portables Python

```
Schreibe die Startskripte:
- start.bat (Windows): wechselt in den eigenen Ordner, nimmt python\python.exe wenn vorhanden, sonst "py -3", sonst "python". Fehlt alles, erscheint eine deutsche Meldung (ZIP vollständig entpacken). Öffnet nach zwei Sekunden http://127.0.0.1:8080 im Browser, startet server.py und zeigt bei Fehlern den Hinweis auf den Port (RSC_PORT=8081 in .env). Fenster bleibt offen (pause).
- start.command (Mac): #!/bin/sh, wechselt in den Ordner, entfernt die Quarantäne (xattr), nimmt ./python/bin/python3 oder python3 ab Version 3.9, sonst Hinweis zur Installation. Öffnet den Browser per "open".
- In server.py: Unter Windows den QuickEdit-Modus der Konsole abschalten, sonst friert der Server ein, sobald jemand ins Fenster klickt.
- Im Ordner python/ liegt ein portables Python 3.12 (python-build-standalone, ohne Tcl/Tk, Tests, pip, Header). Lege eine Datei python/HERKUNFT.txt mit Quelle, Dateiname und SHA-256 der Originaldatei an. Der Ordner ist nur für Windows und gehört nicht ins Git-Repository.
```
