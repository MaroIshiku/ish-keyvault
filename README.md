# Keyku - Key Vault

Self-hosted Steam-Key-Vault für Docker und ZimaOS.

## Kurzbeschreibung

Keyku verwaltet eine gemeinsame Steam-Key-Liste mit Login, Adminbereich, Share-Links und Reaktivierungsanfragen. Die App nutzt ein kleines Flask-Backend, ein statisches Vanilla-Frontend und persistiert Daten im gemounteten Datenordner.

## Teil der ishiku-Familie

Keyku folgt dem gemeinsamen Pixel Soft Utility Designsystem für ishiku Apps: ruhig, rund, praktisch und für Self-hosting gebaut. Themes, App-Shell, Setup-Verhalten und Admin-Info-Bereiche sind appübergreifend konsistent.

Unterstützt werden die sechs gemeinsamen Themes Lavender, Mint, Sky, Amber, Rose und Graphite sowie System-, Hell- und Dunkelmodus.

## Funktionen

- Gemeinsamer Steam-Key-Vault mit Suche, Statusfilter und Sortierung
- Login mit HttpOnly-Session-Cookie
- First-Run-Setup für den ersten Adminaccount
- Setup-Secret per Docker Secret oder Env-Fallback
- Admin-Verwaltung für Keys, Benutzer, Anfragen und Wartung
- Share-Links pro Key ohne Login, geschützt über HMAC-Token
- Key-Reveal, Kopieren, Einlösen, Steam-Suche und SteamDB-Link
- Reaktivierungsanfragen für bereits benutzte Keys
- Passwort-Reset-Anfragen über den Adminbereich
- Health- und Readiness-Endpunkte für Containerbetrieb

## Tech Stack

- Python 3.12
- Flask
- Gunicorn
- Vanilla HTML, CSS und JavaScript
- Pixel Soft Utility Designsystem
- Docker / Docker Compose

## Installation

### Docker Compose

Für ZimaOS ist die Hauptdatei auf absolute Pfade ausgelegt:

```bash
mkdir -p /DATA/AppData/keyku/data
mkdir -p /DATA/AppData/keyku/secrets
openssl rand -base64 48 > /DATA/AppData/keyku/secrets/setup_secret.txt
docker compose pull
docker compose up -d
```

Die App läuft danach standardmäßig auf:

```text
http://<server-ip>:8080
```

### Erstes Starten

Beim ersten Start prüft Keyku, ob ein Adminaccount existiert. Wenn nicht, erscheint sofort das Setup-Fenster. Ohne konfiguriertes Setup-Secret bleibt die App geschlossen und zeigt den fehlenden Konfigurationsschlüssel.

### Adminaccount erstellen

Im Setup-Fenster werden Setup-Secret, Anzeigename, Admin-Benutzername und Passwort eingetragen. Nach erfolgreicher Erstellung ist öffentliche Registrierung geschlossen. Weitere Konten werden anschließend durch Admins in der App erstellt.

## Konfiguration

### Umgebungsvariablen

| Variable | Beschreibung |
| --- | --- |
| `TZ` | Zeitzone, empfohlen `Europe/Berlin` |
| `ISHIKU_APP_URL` | Öffentliche URL hinter Reverse Proxy, für Share-Links |
| `ISHIKU_BASE_PATH` | Optionaler Basis-Pfad, Standard `/` |
| `ISHIKU_DATA_DIR` | Persistenter Datenordner im Container, Standard `/data` |
| `ISHIKU_LOG_LEVEL` | Log-Level, Standard `info` |
| `ISHIKU_TRUST_PROXY` | `true`, wenn ein vertrauenswürdiger Reverse Proxy genutzt wird |
| `ISHIKU_SETUP_SECRET_FILE` | Pfad zum Docker Secret, Standard `/run/secrets/ishiku_setup_secret` |
| `ISHIKU_SETUP_SECRET` | Fallback, wenn Docker Secrets nicht genutzt werden |
| `PORT` | Interner HTTP-Port, Standard `3000` |

### Docker Secrets

Bevorzugt wird ein Docker Secret:

```yaml
secrets:
  ishiku_setup_secret:
    file: ./secrets/setup_secret.txt
```

Das Secret wird nur für die erste Admin-Erstellung verwendet und nicht in der App-Datenbank gespeichert.

### Persistente Daten

Keyku legt im Datenordner unter anderem diese Dateien an:

- `keys.csv`
- `users.json`
- `reactivation-requests.json`
- `password-reset-requests.json`
- `session-secret.txt`
- `setup-state.json`

Diese Dateien sollten zusammen gesichert werden.

## Sicherheit

- Das Setup-Secret ist nur für die First-Run-Registrierung gedacht.
- Das Admin-Passwort darf nicht mit dem Setup-Secret übereinstimmen.
- Passwörter werden als PBKDF2/SHA-256 Hashes mit Salt gespeichert.
- Öffentliche Registrierung ist nach dem ersten Adminaccount geschlossen.
- Sessions laufen über signierte HttpOnly-Cookies mit SameSite=Lax.
- Normale Key-Listenantworten enthalten keine Klartext-Keys.
- Share-Links sind öffentlich, aber kryptisch und HMAC-basiert.
- Secret-Werte gehören nicht in README, Logs, Images oder Git.

## Updates und Backup

```bash
docker compose pull
docker compose up -d
docker compose logs -f
```

Für Backups den kompletten persistenten Datenordner sichern. Vor Wartungsaktionen wie dem Löschen benutzter Keys empfiehlt sich ein aktuelles Backup.

## Entwicklung

Lokal kann das Backend mit Flask oder über Docker gestartet werden. Die Frontend-Dateien liegen statisch in `public/`, das Backend in `python/app.py`.

```bash
docker build -f python/Dockerfile -t keyku:local .
docker run --rm -p 3000:3000 \
  -e ISHIKU_SETUP_SECRET=replace-with-a-long-random-setup-secret \
  -v keyku-data:/data \
  keyku:local
```

## Erstellt mit ChatGPT Codex

Dieses Projekt wurde mit Unterstützung von ChatGPT Codex überarbeitet und implementiert. Codex ist nicht Eigentümer oder Betreiber des Projekts.

## Status und Lizenz

Status: aktiv in Entwicklung.

Eine Lizenzdatei ist aktuell nicht enthalten.
