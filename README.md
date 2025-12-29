# Umami Analytics - Docker Stack

Umami ist eine einfache, schnelle und datenschutzfreundliche Alternative zu Google Analytics.

## Features

- **Privacy-First Analytics**: DSGVO-konform, keine Cookies, keine persönlichen Daten
- **Einfach & Schnell**: Leichtgewichtige Lösung ohne komplexe Konfiguration
- **Open Source**: Vollständig quelloffen und selbst gehostet
- **PostgreSQL Backend**: Zuverlässige und performante Datenspeicherung
- **Automatische Backups**: PostgreSQL Backups alle 6 Stunden
- **Traefik Integration**: Automatisches SSL-Zertifikat via Let's Encrypt
- **Security Headers**: HSTS, X-Frame-Options, CSP und mehr
- **Rate Limiting**: DoS-Schutz mit 100 req/s
- **Geo-Blocking**: Blockiert 23 Länder für zusätzliche Sicherheit
- **Kompression**: Gzip/Brotli für optimale Performance

## Voraussetzungen

- Docker & Docker Compose installiert
- Laufender Traefik Reverse Proxy mit externem Netzwerk `traefik_proxy_network`
- Domain mit DNS-Eintrag auf den Server

## Installation

### 1. Repository klonen

```bash
git clone <repository-url>
cd DockerStackUmami
```

### 2. Umgebungsvariablen konfigurieren

```bash
cp .env.example .env
nano .env
```

**Wichtige Konfigurationen:**

```bash
# Domain konfigurieren
HOSTRULE=Host(`analytics.example.com`)

# Sichere Passwörter generieren (URL-sicher, ohne Sonderzeichen)
DB_PASSWORD=$(openssl rand -hex 32)
APP_SECRET=$(openssl rand -hex 48)

# Backup-Pfad festlegen
BACKUPVOLUME=/mnt/backups/umami
```

### 3. Verzeichnisse erstellen

```bash
mkdir -p volumes/db
mkdir -p ${BACKUPVOLUME}
```

### 4. Stack starten

```bash
docker compose up -d
```

### 5. Logs prüfen

```bash
docker compose logs -f umami
```

### 6. Erste Anmeldung

Öffne deine konfigurierte Domain (z.B. `https://analytics.example.com`) im Browser.

**Standard-Zugangsdaten:**

- Benutzername: `admin`
- Passwort: `umami`

**⚠️ WICHTIG:** Ändere das Admin-Passwort sofort nach der ersten Anmeldung!

## Verwaltung

### Stack stoppen

```bash
docker compose stop
```

### Stack starten

```bash
docker compose start
```

### Stack neu starten

```bash
docker compose restart
```

### Stack komplett entfernen

```bash
docker compose down
```

### Logs anzeigen

```bash
# Alle Logs
docker compose logs -f

# Nur Umami
docker compose logs -f umami

# Nur Datenbank
docker compose logs -f umami-db

# Nur Backup
docker compose logs -f umami-backup
```

## Backup & Wiederherstellung

### Automatische Backups

Backups werden automatisch alle 6 Stunden erstellt und im konfigurierten `BACKUPVOLUME` gespeichert.

**Aufbewahrungsrichtlinie:**

- Tägliche Backups: 7 Tage
- Wöchentliche Backups: 4 Wochen
- Monatliche Backups: 6 Monate

### Manuelles Backup

```bash
docker compose exec umami-db pg_dump -U umami umami > backup_$(date +%Y%m%d_%H%M%S).sql
```

### Wiederherstellung

```bash
# Stack stoppen
docker compose stop umami

# Backup wiederherstellen
cat backup.sql | docker compose exec -T umami-db psql -U umami umami

# Stack starten
docker compose start umami
```

## Tracking Code

Nach der Einrichtung einer Website in Umami erhältst du einen Tracking-Code:

```html
<script defer src="https://analytics.example.com/script.js" data-website-id="your-website-id"></script>
```

Füge diesen Code in den `<head>` deiner Website ein.

## Netzwerk-Konfiguration

Der Stack nutzt zwei Netzwerke:

- **default**: Internes Bridge-Netzwerk für die Kommunikation zwischen den Containern
- **traefik_proxy_network**: Externes Netzwerk für die Traefik-Integration

## Sicherheit

### Aktive Middleware

- **security-headers**: HSTS, X-Frame-Options, CSP, etc.
- **compression**: Gzip/Brotli-Kompression
- **rate-limit**: 100 Anfragen pro Sekunde
- **geo-block**: Blockiert 23 Länder
- **redirect-to-https**: Automatische HTTPS-Weiterleitung

### Empfohlene Sicherheitsmaßnahmen

1. Ändere das Admin-Passwort sofort nach der Installation
2. Verwende starke, zufällig generierte Passwörter
3. Aktiviere 2FA für Admin-Accounts (falls verfügbar)
4. Halte die Container-Images aktuell
5. Überwache die Logs regelmäßig

## Updates

### Container-Images aktualisieren

```bash
docker compose pull
docker compose up -d
```

Umami führt Datenbank-Migrationen automatisch beim Start durch.

## Troubleshooting

### Umami startet wiederholt neu / "TypeError: Invalid URL"

**Problem:** Die Umami-App startet kontinuierlich neu mit einem "Invalid URL" Fehler.

**Ursache:** Das Datenbank-Passwort enthält Sonderzeichen (`+`, `/`, `=`), die in URLs nicht erlaubt sind.

**Lösung:** Generiere ein neues Passwort ohne problematische Sonderzeichen:

```bash
# Neues URL-sicheres Passwort generieren
openssl rand -hex 32

# Dann in der .env Datei eintragen und Container neu starten
docker compose down
docker compose up -d
```

### Umami startet nicht

```bash
# Logs prüfen
docker compose logs umami

# Datenbank-Status prüfen
docker compose exec umami-db pg_isready -U umami
```

### Verbindungsprobleme

```bash
# Netzwerk prüfen
docker network ls | grep traefik

# Container-Status prüfen
docker compose ps
```

### Backup-Fehler

```bash
# Backup-Logs prüfen
docker compose logs umami-backup

# Backup-Verzeichnis-Berechtigung prüfen
ls -la ${BACKUPVOLUME}
```

## Technische Details

### Services

| Service | Image | Port | Beschreibung |
|---------|-------|------|--------------|
| umami | ghcr.io/umami-software/umami:postgresql-latest | 3000 | Umami Anwendung |
| umami-db | postgres:15-alpine | 5432 | PostgreSQL Datenbank |
| umami-backup | prodrigestivill/postgres-backup-local:15-alpine | - | Automatische Backups |

### Volumes

- `./volumes/db`: PostgreSQL Datenbankdaten
- `${BACKUPVOLUME}`: Backup-Speicherort (Host-Pfad)

### Healthchecks

Alle Services verfügen über Healthchecks für automatisches Dependency-Management und Überwachung.

## Links

- [Umami Website](https://umami.is/)
- [Umami Dokumentation](https://umami.is/docs)
- [Umami GitHub](https://github.com/umami-software/umami)
- [PostgreSQL Dokumentation](https://www.postgresql.org/docs/)

## Lizenz

Dieses Projekt steht unter der GPL-3.0 Lizenz. Siehe [LICENSE](LICENSE) für Details.

## Support

Bei Problemen oder Fragen erstelle bitte ein Issue im Repository.

---

**Made with ❤️ by WSC - Web SEO Consulting**

Dieses Projekt ist kostenlos und Open Source (GPL-3.0-or-later). Wenn es dir geholfen hat, freue ich mich über deine Unterstützung:

[![Buy Me a Coffee](https://img.shields.io/badge/Buy%20Me%20a%20Coffee-ffdd00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/csaeum)
[![GitHub Sponsors](https://img.shields.io/badge/GitHub%20Sponsors-ea4aaa?style=for-the-badge&logo=github-sponsors&logoColor=white)](https://github.com/sponsors/csaeum)
[![PayPal](https://img.shields.io/badge/PayPal-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/csaeum)
