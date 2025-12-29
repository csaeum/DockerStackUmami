# Changelog

Alle wichtigen Änderungen an diesem Projekt werden in dieser Datei dokumentiert.

Das Format basiert auf [Keep a Changelog](https://keepachangelog.com/de/1.0.0/),
und dieses Projekt folgt [Semantic Versioning](https://semver.org/lang/de/).

## [1.1.0] - 2025-12-29

### Hinzugefügt

- Mehrsprachige Dokumentation (Deutsch, Englisch, Französisch)
- GitHub Actions CI/CD Pipeline mit:
  - Docker Compose Validierung
  - YAML Linting
  - Markdown Linting
  - Secrets Scanning (Gitleaks)
- Automatischer Release-Workflow für GitHub Releases
- .gitattributes für saubere Export-Archive
- Ausführliche Troubleshooting-Sektion in allen README-Versionen

### Geändert

- Passwortgenerierung nutzt nun `openssl rand -hex` statt `-base64`
- URL-sichere Passwörter ohne Sonderzeichen (+, /, =)
- Verbesserte .gitignore mit .claude/ Ausschluss

### Behoben

- "TypeError: Invalid URL" durch URL-unsichere Passwörter

### Dokumentation

- Sponsor-Links und Unterstützungssektion hinzugefügt
- Detaillierte Anleitung für Passwort-Problemlösung

## [1.0.0] - 2025-12-28

### Initiale Veröffentlichung

- Initiale Version des Umami Docker Stacks
- Docker Compose Konfiguration mit Umami, PostgreSQL und Backup-Service
- Traefik Integration mit SSL-Zertifikaten via Let's Encrypt
- Automatische PostgreSQL Backups alle 6 Stunden
- Security Headers (HSTS, X-Frame-Options, CSP)
- Rate Limiting (100 req/s DoS-Schutz)
- Geo-Blocking für 23 Länder
- Gzip/Brotli Kompression
- Health Checks für alle Services
- Umfassende deutsche Dokumentation
- .env.example für einfache Konfiguration
- .gitignore für sichere Versionsverwaltung

### Sicherheit

- Verwendung der offiziellen Umami und PostgreSQL Alpine Images
- Sichere Passwort-Generierung mittels OpenSSL
- APP_SECRET für sichere Session-Verwaltung
- Automatische HTTPS-Weiterleitung
- Moderne TLS-Konfiguration

### Infrastruktur

- PostgreSQL 15 Alpine als Datenbank-Backend
- Automatische Datenbank-Backups mit Aufbewahrungsrichtlinien
- Bridge-Netzwerk für interne Container-Kommunikation
- Externes Traefik-Netzwerk für Reverse Proxy
- Volume-Management für persistente Daten
