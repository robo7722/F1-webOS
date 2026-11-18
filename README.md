# F1 Hub für LG webOS

Der [F1 Hub](https://github.com/robo7722/F1) als App für LG-Fernseher. Alle Termine von Formel 1 und Formel 2, die WM-Tabellen, die Ergebnisse und die News, gebaut für die Fernbedienung.

- **App-ID:** `com.robo7722.f1hub`
- **Ziel:** webOS 6 und neuer. Auf webOS 4/5 startet die App ebenfalls, das Layout kann dort aber einfacher aussehen.
- **Auflösung:** fest 1920×1080 mit Rand für die Bildschirmkanten. Auf HD-Modellen wird die Ansicht automatisch verkleinert.

## Aufbau

Die App folgt dem Muster bekannter TV-Apps: **Menü links, Inhalt als Kartenreihen rechts, Tastenhilfe unten.**

| Bereich | Inhalt |
|---|---|
| **Start** | Große Karte „Als Nächstes“ mit Countdown und Wetter, darunter Reihen mit den nächsten Sessions, den neuesten News und den letzten Ergebnissen |
| **Termine** | Oben die Details des gewählten Wochenendes (alle Sessions, Wetter, Streckenkarte), darunter die Reihen Rennwochenenden, Anzeige-Filter und Saison 2027 |
| **WM-Stand** | Vier Karten (F1/F2, Fahrer/Teams). Die Tabelle darunter wechselt, sobald eine Karte fokussiert ist. |
| **Ergebnisse** | Oben das Podium des gewählten Rennens mit Pole und schnellster Runde. Mit OK öffnen sich alle Ergebnisse (Rennen, Qualifikation, Sprint, Sprint-Quali). |
| **News** | Links die Schlagzeilen, rechts die gewählte Meldung mit **QR-Code zum Weiterlesen auf dem Handy**. Mit OK öffnet sich der Artikel im TV-Browser. |

## Bedienung

| Taste | Im Menü | Im Inhalt |
|---|---|---|
| ▲ ▼ | Bereich wählen (wird sofort angezeigt) | Reihe wechseln |
| ◀ ▶ | ▶ springt in den Inhalt | Innerhalb der Reihe bewegen. ◀ am Anfang springt zurück ins Menü. |
| OK | Bereich öffnen | Aktion der Karte. Was OK gerade tut, steht unten in der Tastenhilfe. |
| Zurück | Zur Startseite, auf der Startseite: App beenden | Zurück ins Menü (eine offene Ergebnis-Ansicht wird zuerst geschlossen) |

Beim Start liegt der Fokus direkt auf der Karte „Als Nächstes“. Die Magic Remote funktioniert auch als Zeiger. Einen Bearbeiten-Modus gibt es auf dem TV nicht, weil keine Tastatur vorhanden ist.

## Daten

- **News:** kommen weiterhin aus dem Original-Repo, über `https://raw.githubusercontent.com/robo7722/F1/main/news.json`. Die GitHub Action dort aktualisiert sie alle 20 Minuten. Fällt der Abruf aus, zeigt die App die zuletzt gespeicherten News.
- **Zeitplan, Tabellen, Ergebnisse:** Jolpica F1 (Ergast-Nachfolger), OpenF1 und Wikipedia (F2)
- **Wetter:** Open-Meteo
- **Streckenkarten:** bacinger/f1-circuits
- **Flaggen:** flagcdn.com
- **QR-Codes:** [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator) von Kazuhiko Arase (MIT), liegt unter `app/lib/`

Alles wird lokal auf dem Fernseher zwischengespeichert, damit die App auch ohne Netz etwas anzeigt.

## Installation

Die fertige IPK liegt bei jedem Release unter [Releases](../../releases), z. B. `com.robo7722.f1hub_1.0.0_all.ipk`.

### Variante A: Developer Mode (ohne Root)

1. Auf dem TV mit einem LG-Konto anmelden und im LG Content Store die App **Developer Mode** installieren.
2. In der App **Dev Mode Status** auf *ON* stellen. Der TV startet neu.
3. Die App erneut öffnen und **Key Server** einschalten. Die angezeigte IP-Adresse und Passphrase notieren.
4. Auf dem PC die webOS CLI installieren (Node.js nötig):
   ```bash
   npm install -g @webos-tools/cli
   ```
5. Den TV als Gerät einrichten. Als Name `tv` vergeben, die IP aus Schritt 3 eintragen und Port `9922` sowie Benutzer `prisoner` übernehmen:
   ```bash
   ares-setup-device
   ```
6. Den Schlüssel holen. Dabei die Passphrase aus Schritt 3 eingeben:
   ```bash
   ares-novacom --device tv --getkey
   ```
7. Installieren und starten:
   ```bash
   ares-install --device tv com.robo7722.f1hub_1.0.0_all.ipk
   ```
   ```bash
   ares-launch --device tv com.robo7722.f1hub
   ```

Hinweis: Die Developer-Mode-Sitzung läuft nach etwa 50 Stunden ab. Danach verschwinden selbst installierte Apps. Die Laufzeit lässt sich in der Developer-Mode-App mit **Extend** verlängern.

### Variante B: gerooteter TV (z. B. mit Homebrew Channel)

Am einfachsten installierst du die IPK im **Homebrew Channel**. Per SSH geht es so:

1. Die IPK auf den TV kopieren:
   ```bash
   scp com.robo7722.f1hub_1.0.0_all.ipk root@<TV-IP>:/tmp/f1hub.ipk
   ```
2. Auf dem TV (per `ssh root@<TV-IP>`) installieren:
   ```bash
   luna-send -n 1 -f luna://com.webos.appInstallService/dev/install '{"id":"com.ares.defaultName","ipkUrl":"/tmp/f1hub.ipk","subscribe":true}'
   ```
   Warte auf `"state": "installed"` und brich dann mit Strg+C ab.
3. Starten:
   ```bash
   luna-send -n 1 luna://com.webos.applicationManager/launch '{"id":"com.robo7722.f1hub"}'
   ```

Auf gerooteten Geräten bleibt die App dauerhaft installiert, ein Ablauf wie im Developer Mode gibt es dort nicht.

### Deinstallieren

```bash
ares-install --device tv --remove com.robo7722.f1hub
```

Oder auf dem TV selbst: App in der Startleiste lange drücken und auf **Entfernen** gehen.

## Entwickeln

- Die App besteht aus `app/index.html`, `app/appinfo.json`, den Icons und `app/lib/qrcode.js`. Es gibt keinen Build-Schritt.
- Testen im Browser: `app/index.html` öffnen und das Fenster auf 1920×1080 stellen (kleinere Fenster werden skaliert). Pfeiltasten, Enter und Esc/Backspace entsprechen der Fernbedienung.
- Bewusst ES5 und konservatives CSS: kein `grid`, kein `gap` bei Flexbox, kein `inset`, kein `Intl…formatToParts`. So läuft die App ab Chromium 53 (webOS 4).
- Lokal packen:
  ```bash
  ares-package app -o dist
  ```

## Release

Einen Tag `v*` pushen, z. B.:

```bash
git tag v1.0.0
```
```bash
git push origin v1.0.0
```

Die Action [`.github/workflows/release.yml`](.github/workflows/release.yml) übernimmt die Version aus dem Tag in `appinfo.json`, baut die IPK mit `ares-package` und hängt sie an ein GitHub-Release. Bei Pull Requests baut sie die IPK nur als Artefakt.
