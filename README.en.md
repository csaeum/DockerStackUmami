# Umami Analytics - Docker Stack

Umami is a simple, fast, and privacy-friendly alternative to Google Analytics.

## Features

- **Privacy-First Analytics**: GDPR compliant, no cookies, no personal data
- **Simple & Fast**: Lightweight solution without complex configuration
- **Open Source**: Fully open source and self-hosted
- **PostgreSQL Backend**: Reliable and performant data storage
- **Automatic Backups**: PostgreSQL backups every 6 hours
- **Traefik Integration**: Automatic SSL certificate via Let's Encrypt
- **Security Headers**: HSTS, X-Frame-Options, CSP and more
- **Rate Limiting**: DoS protection with 100 req/s
- **Geo-Blocking**: Blocks 23 countries for additional security
- **Compression**: Gzip/Brotli for optimal performance

## Prerequisites

- Docker & Docker Compose installed
- Running Traefik Reverse Proxy with external network `traefik_proxy_network`
- Domain with DNS record pointing to the server

## Installation

### 1. Clone Repository

```bash
git clone <repository-url>
cd DockerStackUmami
```

### 2. Configure Environment Variables

```bash
cp .env.example .env
nano .env
```

**Important Configurations:**

```bash
# Configure domain
HOSTRULE=Host(`analytics.example.com`)

# Generate secure passwords (URL-safe, without special characters)
DB_PASSWORD=$(openssl rand -hex 32)
APP_SECRET=$(openssl rand -hex 48)

# Set backup path
BACKUPVOLUME=/mnt/backups/umami
```

### 3. Create Directories

```bash
mkdir -p volumes/db
mkdir -p ${BACKUPVOLUME}
```

### 4. Start Stack

```bash
docker compose up -d
```

### 5. Check Logs

```bash
docker compose logs -f umami
```

### 6. First Login

Open your configured domain (e.g., `https://analytics.example.com`) in your browser.

**Default Credentials:**
- Username: `admin`
- Password: `umami`

**⚠️ IMPORTANT:** Change the admin password immediately after first login!

## Management

### Stop Stack

```bash
docker compose stop
```

### Start Stack

```bash
docker compose start
```

### Restart Stack

```bash
docker compose restart
```

### Remove Stack Completely

```bash
docker compose down
```

### View Logs

```bash
# All logs
docker compose logs -f

# Umami only
docker compose logs -f umami

# Database only
docker compose logs -f umami-db

# Backup only
docker compose logs -f umami-backup
```

## Backup & Restore

### Automatic Backups

Backups are automatically created every 6 hours and saved in the configured `BACKUPVOLUME`.

**Retention Policy:**
- Daily backups: 7 days
- Weekly backups: 4 weeks
- Monthly backups: 6 months

### Manual Backup

```bash
docker compose exec umami-db pg_dump -U umami umami > backup_$(date +%Y%m%d_%H%M%S).sql
```

### Restore

```bash
# Stop stack
docker compose stop umami

# Restore backup
cat backup.sql | docker compose exec -T umami-db psql -U umami umami

# Start stack
docker compose start umami
```

## Tracking Code

After setting up a website in Umami, you will receive a tracking code:

```html
<script defer src="https://analytics.example.com/script.js" data-website-id="your-website-id"></script>
```

Add this code to the `<head>` of your website.

## Network Configuration

The stack uses two networks:

- **default**: Internal bridge network for communication between containers
- **traefik_proxy_network**: External network for Traefik integration

## Security

### Active Middleware

- **security-headers**: HSTS, X-Frame-Options, CSP, etc.
- **compression**: Gzip/Brotli compression
- **rate-limit**: 100 requests per second
- **geo-block**: Blocks 23 countries
- **redirect-to-https**: Automatic HTTPS redirect

### Recommended Security Measures

1. Change the admin password immediately after installation
2. Use strong, randomly generated passwords
3. Enable 2FA for admin accounts (if available)
4. Keep container images up to date
5. Monitor logs regularly

## Updates

### Update Container Images

```bash
docker compose pull
docker compose up -d
```

Umami performs database migrations automatically on startup.

## Troubleshooting

### Umami Restarts Repeatedly / "TypeError: Invalid URL"

**Problem:** The Umami app restarts continuously with an "Invalid URL" error.

**Cause:** The database password contains special characters (`+`, `/`, `=`) that are not allowed in URLs.

**Solution:** Generate a new password without problematic special characters:

```bash
# Generate new URL-safe password
openssl rand -hex 32

# Then enter it in the .env file and restart containers
docker compose down
docker compose up -d
```

### Umami Won't Start

```bash
# Check logs
docker compose logs umami

# Check database status
docker compose exec umami-db pg_isready -U umami
```

### Connection Problems

```bash
# Check network
docker network ls | grep traefik

# Check container status
docker compose ps
```

### Backup Errors

```bash
# Check backup logs
docker compose logs umami-backup

# Check backup directory permissions
ls -la ${BACKUPVOLUME}
```

## Technical Details

### Services

| Service | Image | Port | Description |
|---------|-------|------|-------------|
| umami | ghcr.io/umami-software/umami:postgresql-latest | 3000 | Umami Application |
| umami-db | postgres:15-alpine | 5432 | PostgreSQL Database |
| umami-backup | prodrigestivill/postgres-backup-local:15-alpine | - | Automatic Backups |

### Volumes

- `./volumes/db`: PostgreSQL database data
- `${BACKUPVOLUME}`: Backup storage location (host path)

### Healthchecks

All services have healthchecks for automatic dependency management and monitoring.

## Links

- [Umami Website](https://umami.is/)
- [Umami Documentation](https://umami.is/docs)
- [Umami GitHub](https://github.com/umami-software/umami)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

## License

This project is licensed under the GPL-3.0 License. See [LICENSE](LICENSE) for details.

## Support

For issues or questions, please create an issue in the repository.

---

**Made with ❤️ by WSC - Web SEO Consulting**

This project is free and open source (GPL-3.0-or-later). If it helped you, I appreciate your support:

[![Buy Me a Coffee](https://img.shields.io/badge/Buy%20Me%20a%20Coffee-ffdd00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/csaeum)
[![GitHub Sponsors](https://img.shields.io/badge/GitHub%20Sponsors-ea4aaa?style=for-the-badge&logo=github-sponsors&logoColor=white)](https://github.com/sponsors/csaeum)
[![PayPal](https://img.shields.io/badge/PayPal-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/csaeum)
