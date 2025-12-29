# Umami Analytics - Stack Docker

Umami est une alternative simple, rapide et respectueuse de la vie privée à Google Analytics.

## Fonctionnalités

- **Analyse axée sur la confidentialité** : Conforme au RGPD, sans cookies, sans données personnelles
- **Simple et rapide** : Solution légère sans configuration complexe
- **Open Source** : Entièrement open source et auto-hébergé
- **Backend PostgreSQL** : Stockage de données fiable et performant
- **Sauvegardes automatiques** : Sauvegardes PostgreSQL toutes les 6 heures
- **Intégration Traefik** : Certificat SSL automatique via Let's Encrypt
- **En-têtes de sécurité** : HSTS, X-Frame-Options, CSP et plus
- **Limitation de débit** : Protection DoS avec 100 req/s
- **Géo-blocage** : Bloque 23 pays pour plus de sécurité
- **Compression** : Gzip/Brotli pour des performances optimales

## Prérequis

- Docker & Docker Compose installés
- Reverse Proxy Traefik en cours d'exécution avec réseau externe `traefik_proxy_network`
- Domaine avec enregistrement DNS pointant vers le serveur

## Installation

### 1. Cloner le dépôt

```bash
git clone <repository-url>
cd DockerStackUmami
```

### 2. Configurer les variables d'environnement

```bash
cp .env.example .env
nano .env
```

**Configurations importantes :**

```bash
# Configurer le domaine
HOSTRULE=Host(`analytics.example.com`)

# Générer des mots de passe sécurisés (sûrs pour les URL, sans caractères spéciaux)
DB_PASSWORD=$(openssl rand -hex 32)
APP_SECRET=$(openssl rand -hex 48)

# Définir le chemin de sauvegarde
BACKUPVOLUME=/mnt/backups/umami
```

### 3. Créer les répertoires

```bash
mkdir -p volumes/db
mkdir -p ${BACKUPVOLUME}
```

### 4. Démarrer la stack

```bash
docker compose up -d
```

### 5. Vérifier les logs

```bash
docker compose logs -f umami
```

### 6. Première connexion

Ouvrez votre domaine configuré (par exemple `https://analytics.example.com`) dans votre navigateur.

**Identifiants par défaut :**
- Nom d'utilisateur : `admin`
- Mot de passe : `umami`

**⚠️ IMPORTANT :** Changez le mot de passe admin immédiatement après la première connexion !

## Gestion

### Arrêter la stack

```bash
docker compose stop
```

### Démarrer la stack

```bash
docker compose start
```

### Redémarrer la stack

```bash
docker compose restart
```

### Supprimer complètement la stack

```bash
docker compose down
```

### Afficher les logs

```bash
# Tous les logs
docker compose logs -f

# Umami uniquement
docker compose logs -f umami

# Base de données uniquement
docker compose logs -f umami-db

# Sauvegarde uniquement
docker compose logs -f umami-backup
```

## Sauvegarde et restauration

### Sauvegardes automatiques

Les sauvegardes sont automatiquement créées toutes les 6 heures et enregistrées dans le `BACKUPVOLUME` configuré.

**Politique de conservation :**
- Sauvegardes quotidiennes : 7 jours
- Sauvegardes hebdomadaires : 4 semaines
- Sauvegardes mensuelles : 6 mois

### Sauvegarde manuelle

```bash
docker compose exec umami-db pg_dump -U umami umami > backup_$(date +%Y%m%d_%H%M%S).sql
```

### Restauration

```bash
# Arrêter la stack
docker compose stop umami

# Restaurer la sauvegarde
cat backup.sql | docker compose exec -T umami-db psql -U umami umami

# Démarrer la stack
docker compose start umami
```

## Code de suivi

Après avoir configuré un site web dans Umami, vous recevrez un code de suivi :

```html
<script defer src="https://analytics.example.com/script.js" data-website-id="your-website-id"></script>
```

Ajoutez ce code dans le `<head>` de votre site web.

## Configuration réseau

La stack utilise deux réseaux :

- **default** : Réseau bridge interne pour la communication entre les conteneurs
- **traefik_proxy_network** : Réseau externe pour l'intégration Traefik

## Sécurité

### Middleware actifs

- **security-headers** : HSTS, X-Frame-Options, CSP, etc.
- **compression** : Compression Gzip/Brotli
- **rate-limit** : 100 requêtes par seconde
- **geo-block** : Bloque 23 pays
- **redirect-to-https** : Redirection HTTPS automatique

### Mesures de sécurité recommandées

1. Changez le mot de passe admin immédiatement après l'installation
2. Utilisez des mots de passe forts générés aléatoirement
3. Activez la 2FA pour les comptes admin (si disponible)
4. Gardez les images de conteneurs à jour
5. Surveillez régulièrement les logs

## Mises à jour

### Mettre à jour les images de conteneurs

```bash
docker compose pull
docker compose up -d
```

Umami effectue automatiquement les migrations de base de données au démarrage.

## Dépannage

### Umami redémarre en boucle / "TypeError: Invalid URL"

**Problème :** L'application Umami redémarre continuellement avec une erreur "Invalid URL".

**Cause :** Le mot de passe de la base de données contient des caractères spéciaux (`+`, `/`, `=`) qui ne sont pas autorisés dans les URL.

**Solution :** Générez un nouveau mot de passe sans caractères spéciaux problématiques :

```bash
# Générer un nouveau mot de passe sûr pour les URL
openssl rand -hex 32

# Ensuite, saisissez-le dans le fichier .env et redémarrez les conteneurs
docker compose down
docker compose up -d
```

### Umami ne démarre pas

```bash
# Vérifier les logs
docker compose logs umami

# Vérifier l'état de la base de données
docker compose exec umami-db pg_isready -U umami
```

### Problèmes de connexion

```bash
# Vérifier le réseau
docker network ls | grep traefik

# Vérifier l'état des conteneurs
docker compose ps
```

### Erreurs de sauvegarde

```bash
# Vérifier les logs de sauvegarde
docker compose logs umami-backup

# Vérifier les permissions du répertoire de sauvegarde
ls -la ${BACKUPVOLUME}
```

## Détails techniques

### Services

| Service | Image | Port | Description |
|---------|-------|------|-------------|
| umami | ghcr.io/umami-software/umami:postgresql-latest | 3000 | Application Umami |
| umami-db | postgres:15-alpine | 5432 | Base de données PostgreSQL |
| umami-backup | prodrigestivill/postgres-backup-local:15-alpine | - | Sauvegardes automatiques |

### Volumes

- `./volumes/db` : Données de la base de données PostgreSQL
- `${BACKUPVOLUME}` : Emplacement de stockage des sauvegardes (chemin hôte)

### Healthchecks

Tous les services disposent de healthchecks pour la gestion automatique des dépendances et la surveillance.

## Liens

- [Site web Umami](https://umami.is/)
- [Documentation Umami](https://umami.is/docs)
- [Umami GitHub](https://github.com/umami-software/umami)
- [Documentation PostgreSQL](https://www.postgresql.org/docs/)

## Licence

Ce projet est sous licence GPL-3.0. Voir [LICENSE](LICENSE) pour plus de détails.

## Support

Pour des problèmes ou des questions, veuillez créer une issue dans le dépôt.

---

**Fait avec ❤️ par WSC - Web SEO Consulting**

Ce projet est gratuit et open source (GPL-3.0-or-later). S'il vous a aidé, j'apprécie votre soutien :

[![Buy Me a Coffee](https://img.shields.io/badge/Buy%20Me%20a%20Coffee-ffdd00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/csaeum)
[![GitHub Sponsors](https://img.shields.io/badge/GitHub%20Sponsors-ea4aaa?style=for-the-badge&logo=github-sponsors&logoColor=white)](https://github.com/sponsors/csaeum)
[![PayPal](https://img.shields.io/badge/PayPal-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/csaeum)
