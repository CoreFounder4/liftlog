# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Liftlog

Persönliche Krafttraining-Tracking-App (UI komplett auf Deutsch). Ursprünglich als Claude-Artefakt entstanden, jetzt lokales Projekt. Echte Nutzung als Home-Bildschirm-App auf dem iPhone über GitHub Pages (`CoreFounder4/liftlog`).

## Befehle
- Lokal starten: `npx live-server --port=8000 --no-browser` (Python ist nicht installiert), dann http://localhost:8000. Lädt beim Speichern automatisch neu, auch auf dem Handy im WLAN (`http://<PC-IP>:8000`). Im Browser-Pane: `preview_start` mit Name `liftlog` (`.claude/launch.json`).
- Veröffentlichen: `git commit` + `git push` auf `main` → GitHub Pages nach ca. 1 Minute aktuell. Auf dem iPhone die App schließen und neu öffnen.
- Kein Build, kein Linter, keine Tests. Prüfen heißt: im Browser öffnen (mobile Viewport), Konsole auf Fehler checken, Testdaten über den Sicherungs-Button importieren.

## Aufbau
- `index.html`: gesamte App (HTML, CSS, JS inline, ca. 210 KB). Keine Abhängigkeiten.
- Nur für die Installation als App: `manifest.webmanifest`, `icon-180.png`, `icon-512.png`, `sw.js` (Netz zuerst, Cache nur offline, speichert nie Daten; neue Dateien ggf. in `FILES` eintragen).
- `liftlog-sicherung-*.json`: Sicherungen (`{app:'Liftlog', version:1, schema:N, data:{...}}`). Per `.gitignore` ausgeschlossen, das Repo ist öffentlich.

## Architektur von index.html
- **Markup:** Tab-Screens `#menu`, `#calendar`, `#stats`, `#settings` (Umschalten mit `showTab()`), Vollbild-Screens `#editor`, `#workout`, `#summary`, und viele Dialoge als `.overlay` (bzw. `.overlay.popover.bottom-sheet`), die per `hidden` ein-/ausgeblendet werden.
- **Script:** Oben stehen Speicherschicht und Start (`kv`, `KV_KEYS`, `makeCloud()`, `bootApp()`, `runMigrations()`). Danach läuft fast die gesamte App-Logik in einer großen Funktion `startApp()`: Zustand als Closure-Variablen (`plans`, `splits`, `sessions`, `workoutData`, `settings` …). Jedes Stück Zustand hat eine `*_KEY`-Konstante und eine `store…()`-Funktion, die es als JSON in `kv` schreibt. Danach wird neu gerendert (`render()`, `renderStats()` …). `$ = id => document.getElementById(id)`.
- **Datenformen:**
  - Plan: `{id, name, exercises:[{name, sets, range:[min,max]}], defaults}`.
  - Split: `{id, name, planIds, schedule:{mode:'cycle', seq:[planId|'rest']}}`.
  - Session: `{date:'YYYY-MM-DD', planId, plan, at, dur, data:{Übungsname:[{w,r}]}}`. Gewicht und Wiederholungen sind Strings mit Dezimalkomma.
  - `gym-workout-data`: zuletzt eingegebene Werte pro planId (zum Vorbefüllen).
- **Übungen werden über ihren Namen referenziert**, nicht über eine ID: in Sessions, `gym-workout-data`, `gym-steps` und Statistiken. Wer eine Übung umbenennt, braucht eine Migration (siehe Migration v1 „Latzug“ → „Latzug (breit)“).
- Reine UI-Zustände (z. B. `liftlog-collapsed`, `gym-last-backup`) liegen direkt in `localStorage`, nicht in `KV_KEYS`, und werden nicht gesichert.

## Datenspeicherung
- Alle Daten laufen über `kv` (Schlüssel in `KV_KEYS`). Im Claude-Artefakt war das Cloud-Sync über `window.claude.use('db')`. Außerhalb fällt `bootApp()` automatisch auf `localStorage` zurück. Neue persistente Schlüssel müssen in `KV_KEYS`, sonst fehlen sie in Export und Import.
- Der Sicherungs-Button (`#backup-btn`) erscheint nur ohne `window.claude`: Export (iPhone: Teilen-Menü) und Import. Beim Import wird der alte Stand in `gym-backup-pre-import` abgelegt.

## Echte Daten vs. Entwicklung
- Getrennt über die Adresse (Origin): echte Daten nur auf `https://corefounder4.github.io/liftlog/` (iPhone-App), Testdaten auf `localhost:8000`.
- `IS_DEV` (http, localhost oder Heimnetz-IP) zeigt ein oranges „DEV“-Abzeichen. Der Service Worker wird nur bei https registriert.

## Datenformat ändern (Pflicht)
- `SCHEMA_VERSION` erhöhen und in `MIGRATIONS` einen Eintrag `{ to: N, run() {...} }` ergänzen.
- Migrationen löschen nie Daten, sie wandeln nur um oder ergänzen. Sie müssen mehrfach ausführbar sein.
- `runMigrations()` legt vorher automatisch `gym-backup-pre-v<N>` an. Schlägt eine Migration fehl, bleibt die Version stehen.

## Regeln
- App-Code bleibt in einer einzelnen `index.html`, solange nichts anderes entschieden wird.
- UI-Texte auf Deutsch, Mobile-first (Safe-Area-Insets beachten), dunkles Design über CSS-Variablen in `:root`.
- Die Datei gezielt lesen (Grep, Ausschnitte), nicht komplett einlesen.
