# Liftlog

Persönliche Krafttraining-Tracking-App (UI komplett auf Deutsch). Ursprünglich als Claude-Artefakt entstanden, jetzt lokales Projekt. Echte Nutzung als Home-Bildschirm-App auf dem iPhone über GitHub Pages.

## Aufbau
- `index.html`: gesamte App in einer Datei (HTML, CSS, JS inline, ca. 210 KB). Keine Build-Tools, keine Abhängigkeiten.
- Zusatzdateien nur für die Installation als App: `manifest.webmanifest`, `icon-180.png`, `icon-512.png`, `sw.js` (Service Worker: Netz zuerst, Cache nur offline, speichert nie Daten).
- `liftlog-sicherung-*.json`: Sicherungen der Trainingsdaten (Format `{app:'Liftlog', version:1, schema:N, data:{...}}`). Per `.gitignore` ausgeschlossen, das Repo ist öffentlich.

## Datenspeicherung
- Alle Daten laufen über das `kv`-Objekt (Schlüssel in `KV_KEYS`, z. B. `gym-plans`, `gym-sessions`).
- Im Artefakt wurde `window.claude.use('db')` für Cloud-Sync genutzt. Außerhalb von Claude gibt es das nicht: die App fällt automatisch auf `localStorage` zurück (siehe `bootApp()`).
- Der Sicherungs-Button (`#backup-btn`) erscheint nur, wenn `window.claude` fehlt. Damit lassen sich Daten exportieren (iPhone: Teilen-Menü) und importieren. Beim Import wird der alte Stand in `gym-backup-pre-import` abgelegt.

## Echte Daten vs. Entwicklung
- Getrennt über die Adresse (Origin): echte Daten nur auf `https://<name>.github.io/liftlog/` (iPhone-App), Testdaten auf `http://localhost:8000`.
- `IS_DEV` (localhost, http oder Heimnetz-IP) zeigt ein oranges „DEV“-Abzeichen. Der Service Worker wird nur bei https registriert.

## Datenformat ändern (Pflicht)
- `SCHEMA_VERSION` erhöhen und in `MIGRATIONS` einen Eintrag `{ to: N, run() {...} }` ergänzen.
- Migrationen löschen nie Daten, sie wandeln nur um oder ergänzen. Sie müssen mehrfach ausführbar sein.
- `runMigrations()` legt vorher automatisch `gym-backup-pre-v<N>` im localStorage an.

## Regeln
- App-Code bleibt in einer einzelnen `index.html`, solange nichts anderes entschieden wird.
- UI-Texte auf Deutsch, Mobile-first (Safe-Area-Insets beachten), dunkles Design über CSS-Variablen.
- Vor größeren Änderungen die Datei gezielt lesen (Grep), nicht komplett einlesen.

## Lokal starten
`npx serve -l 8000` im Projektordner (Python ist nicht installiert), dann http://localhost:8000 öffnen. Testdaten über den Sicherungs-Button importieren.

## Veröffentlichen
`git commit` + `git push` auf `main` → GitHub Pages ist nach ca. 1 Minute aktuell. Auf dem iPhone die App schließen und neu öffnen.
