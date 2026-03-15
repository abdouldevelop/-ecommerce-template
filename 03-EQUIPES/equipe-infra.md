# Equipe Infrastructure - Tâches et Priorités

## Instructions pour l'agent IA

Ce document couvre toutes les tâches d'infrastructure : Docker, déploiement, reverse proxy, SSL, base de données, backups, monitoring et CI/CD. Les tâches P0 sont indispensables pour la mise en production.

**Légende priorités** : P0 (bloquant), P1 (important), P2 (souhaitable), P3 (futur)
**Légende complexité** : S (< 2h), M (2-4h), L (4-8h), XL (> 8h)

---

## Module 1 : Environnement de développement (P0)

### INFRA-001 : Docker Compose (développement)
- **Priorité** : P0
- **Complexité** : M
- **Description** : Configuration Docker pour le développement local
- **Critères d'acceptation** :
  - [ ] `docker-compose.yml` avec services : postgres, api, web
  - [ ] PostgreSQL avec volume persistant
  - [ ] Hot-reload pour api et web (volumes montés)
  - [ ] Variables d'environnement dans `.env`
  - [ ] `docker compose up` lance tout le projet
  - [ ] Réseau interne entre les containers
- **Fichier** :
```yaml
# docker-compose.yml
version: '3.8'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ecommerce
      POSTGRES_PASSWORD: ecommerce_dev_pass
      POSTGRES_DB: ecommerce
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  api:
    build:
      context: .
      dockerfile: apps/api/Dockerfile
      target: development
    ports:
      - "3001:3001"
    environment:
      DATABASE_URL: postgresql://ecommerce:ecommerce_dev_pass@postgres:5432/ecommerce
      JWT_SECRET: dev-jwt-secret-change-in-production
      JWT_REFRESH_SECRET: dev-refresh-secret-change-in-production
    volumes:
      - ./apps/api/src:/app/apps/api/src
      - ./apps/api/uploads:/app/apps/api/uploads
    depends_on:
      - postgres

  web:
    build:
      context: .
      dockerfile: apps/web/Dockerfile
      target: development
    ports:
      - "3000:3000"
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:3001
    volumes:
      - ./apps/web/app:/app/apps/web/app
      - ./apps/web/components:/app/apps/web/components
      - ./apps/web/lib:/app/apps/web/lib
    depends_on:
      - api

volumes:
  postgres_data:
```

### INFRA-002 : Dockerfile API (multi-stage)
- **Priorité** : P0
- **Complexité** : M
- **Description** : Dockerfile optimisé pour le backend NestJS
- **Critères d'acceptation** :
  - [ ] Multi-stage build : development et production
  - [ ] Stage development : hot-reload avec `nest start --watch`
  - [ ] Stage production : build compilé, node_modules production only
  - [ ] Image finale < 200 Mo
  - [ ] User non-root en production
  - [ ] Healthcheck intégré

### INFRA-003 : Dockerfile Web (multi-stage)
- **Priorité** : P0
- **Complexité** : M
- **Description** : Dockerfile optimisé pour le frontend Next.js
- **Critères d'acceptation** :
  - [ ] Multi-stage build avec `next build` en standalone
  - [ ] Image finale < 150 Mo
  - [ ] Optimisation : `.next/standalone` + `.next/static`
  - [ ] User non-root en production

---

## Module 2 : Serveur de production (P0)

### INFRA-004 : Configuration du serveur
- **Priorité** : P0
- **Complexité** : L
- **Description** : Configuration initiale du VPS de production
- **Critères d'acceptation** :
  - [ ] OS : Ubuntu 22.04 LTS ou Debian 12
  - [ ] Mise à jour système : `apt update && apt upgrade`
  - [ ] Utilisateur non-root créé avec sudo
  - [ ] SSH : clé publique configurée, authentification par mot de passe désactivée
  - [ ] Pare-feu UFW : ports 22, 80, 443 ouverts uniquement
  - [ ] Fail2ban installé et configuré
  - [ ] Fuseau horaire configuré (Africa/Abidjan)
  - [ ] Swap configuré (2 Go si RAM < 4 Go)

### INFRA-005 : Installation des dépendances serveur
- **Priorité** : P0
- **Complexité** : M
- **Description** : Installer Node.js, PostgreSQL, Nginx, PM2, Certbot
- **Critères d'acceptation** :
  - [ ] Node.js 20 LTS (via nvm ou NodeSource)
  - [ ] PostgreSQL 16 installé et sécurisé
  - [ ] Nginx installé
  - [ ] PM2 installé globalement (`npm i -g pm2`)
  - [ ] Certbot installé (Let's Encrypt)
  - [ ] Git installé
  - [ ] Vérification : `node -v`, `psql --version`, `nginx -v`, `pm2 -v`

### INFRA-006 : Configuration PostgreSQL (production)
- **Priorité** : P0
- **Complexité** : M
- **Description** : Configuration et optimisation de PostgreSQL
- **Critères d'acceptation** :
  - [ ] Utilisateur dédié créé (pas postgres en direct)
  - [ ] Base de données créée
  - [ ] Permissions restreintes (seulement l'utilisateur dédié)
  - [ ] Authentification MD5 ou SCRAM-SHA-256
  - [ ] Ecoute sur localhost uniquement (pas d'accès externe)
  - [ ] Paramètres optimisés pour le VPS :
    - `shared_buffers` = 25% de la RAM
    - `effective_cache_size` = 75% de la RAM
    - `work_mem` = 4-8 Mo
    - `maintenance_work_mem` = 64 Mo
    - `max_connections` = 100
  - [ ] Logs activés (log_min_duration_statement = 1000ms)

---

## Module 3 : Reverse Proxy et SSL (P0)

### INFRA-007 : Configuration Nginx
- **Priorité** : P0
- **Complexité** : L
- **Description** : Nginx comme reverse proxy pour le frontend et l'API
- **Critères d'acceptation** :
  - [ ] Redirection HTTP → HTTPS
  - [ ] Proxy vers Next.js (port 3000) pour `/`
  - [ ] Proxy vers NestJS (port 3001) pour `/api/`
  - [ ] Servir les fichiers statiques uploadés (`/uploads/`)
  - [ ] Compression gzip activée
  - [ ] Cache des assets statiques (images, CSS, JS) : 30 jours
  - [ ] Timeouts proxy : 120 secondes
  - [ ] `client_max_body_size` : 10M
  - [ ] Headers de sécurité : X-Frame-Options, X-Content-Type-Options, etc.
  - [ ] Rate limiting sur /api/auth/ (10 req/min)

### INFRA-008 : Certificat SSL (Let's Encrypt)
- **Priorité** : P0
- **Complexité** : S
- **Description** : Installation et renouvellement automatique du certificat SSL
- **Critères d'acceptation** :
  - [ ] Certificat obtenu avec Certbot
  - [ ] Renouvellement automatique (cron ou timer systemd)
  - [ ] Test : `curl -I https://votredomaine.com` retourne 200
  - [ ] Note SSL Labs : A ou A+
  - [ ] HSTS activé

---

## Module 4 : Déploiement (P0)

### INFRA-009 : Configuration PM2
- **Priorité** : P0
- **Complexité** : M
- **Description** : Gestion des processus Node.js avec PM2
- **Critères d'acceptation** :
  - [ ] Fichier `ecosystem.config.js` :
```javascript
module.exports = {
  apps: [
    {
      name: 'ecommerce-api',
      cwd: '/var/www/ecommerce/apps/api',
      script: 'dist/main.js',
      instances: 2,
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3001,
      },
      max_memory_restart: '500M',
      error_file: '/var/log/pm2/api-error.log',
      out_file: '/var/log/pm2/api-out.log',
    },
    {
      name: 'ecommerce-web',
      cwd: '/var/www/ecommerce/apps/web',
      script: 'node_modules/.bin/next',
      args: 'start -p 3000',
      instances: 1,
      env: {
        NODE_ENV: 'production',
      },
      max_memory_restart: '500M',
      error_file: '/var/log/pm2/web-error.log',
      out_file: '/var/log/pm2/web-out.log',
    },
  ],
};
```
  - [ ] `pm2 startup` pour démarrage automatique au boot
  - [ ] `pm2 save` pour persister la configuration
  - [ ] Monitoring : `pm2 monit` fonctionne

### INFRA-010 : Script de déploiement
- **Priorité** : P0
- **Complexité** : M
- **Description** : Script bash pour déployer les mises à jour
- **Critères d'acceptation** :
  - [ ] Script `deploy.sh` :
```bash
#!/bin/bash
set -e

PROJECT_DIR="/var/www/ecommerce"
echo "=== Déploiement E-Commerce ==="
echo "Date : $(date)"

# 1. Pull du code
cd $PROJECT_DIR
git pull origin main

# 2. Installation des dépendances
npm install --production

# 3. Migrations Prisma
cd apps/api
npx prisma migrate deploy
npx prisma generate
cd ../..

# 4. Build
npm run build

# 5. Restart PM2
pm2 restart ecosystem.config.js

# 6. Vérification
sleep 3
pm2 status
curl -s http://localhost:3001/api/health | jq .

echo "=== Déploiement terminé ==="
```
  - [ ] Permissions d'exécution (`chmod +x deploy.sh`)
  - [ ] Rollback possible : tag Git avant chaque déploiement

---

## Module 5 : DNS (P0)

### INFRA-011 : Configuration DNS
- **Priorité** : P0
- **Complexité** : S
- **Description** : Configuration des enregistrements DNS
- **Critères d'acceptation** :
  - [ ] Enregistrement A : `votredomaine.com` → IP du serveur
  - [ ] Enregistrement A : `www.votredomaine.com` → IP du serveur
  - [ ] TTL : 3600 (1 heure)
  - [ ] Redirection www → non-www (ou inverse, au choix du client)
  - [ ] Vérification : `dig votredomaine.com` retourne l'IP correcte
  - [ ] Propagation complète (24-48h, vérifier sur dnschecker.org)

---

## Module 6 : Backups (P0)

### INFRA-012 : Backup base de données
- **Priorité** : P0
- **Complexité** : M
- **Description** : Sauvegardes automatiques de PostgreSQL
- **Critères d'acceptation** :
  - [ ] Script `backup-db.sh` :
```bash
#!/bin/bash
BACKUP_DIR="/var/backups/postgresql"
DB_NAME="ecommerce"
DB_USER="ecommerce_user"
DATE=$(date +%Y%m%d_%H%M%S)
FILENAME="${DB_NAME}_${DATE}.sql.gz"

mkdir -p $BACKUP_DIR
pg_dump -U $DB_USER $DB_NAME | gzip > "$BACKUP_DIR/$FILENAME"

# Supprimer les backups de plus de 30 jours
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup créé : $FILENAME ($(du -sh $BACKUP_DIR/$FILENAME | cut -f1))"
```
  - [ ] Cron : tous les jours à 3h du matin
  - [ ] Rétention : 30 jours
  - [ ] Test de restauration effectué au moins une fois
  - [ ] Espace disque vérifié

### INFRA-013 : Backup fichiers uploadés
- **Priorité** : P1
- **Complexité** : S
- **Description** : Sauvegarde des images et fichiers uploadés
- **Critères d'acceptation** :
  - [ ] Rsync vers un répertoire de backup : `rsync -a /var/www/ecommerce/apps/api/uploads/ /var/backups/uploads/`
  - [ ] Cron : une fois par jour
  - [ ] Rétention : dernière copie uniquement (rsync incrémental)

---

## Module 7 : Monitoring (P1)

### INFRA-014 : Monitoring applicatif
- **Priorité** : P1
- **Complexité** : M
- **Description** : Surveillance de la santé du système
- **Critères d'acceptation** :
  - [ ] Script de vérification :
```bash
#!/bin/bash
# Vérification API
API_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3001/api/health)
if [ "$API_STATUS" != "200" ]; then
  echo "ALERTE : API down (status: $API_STATUS)" | mail -s "E-Commerce API Down" admin@email.com
  pm2 restart ecommerce-api
fi

# Vérification Frontend
WEB_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3000)
if [ "$WEB_STATUS" != "200" ]; then
  echo "ALERTE : Frontend down (status: $WEB_STATUS)" | mail -s "E-Commerce Web Down" admin@email.com
  pm2 restart ecommerce-web
fi

# Vérification espace disque
DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
if [ "$DISK_USAGE" -gt 85 ]; then
  echo "ALERTE : Disque à ${DISK_USAGE}%" | mail -s "E-Commerce Disk Warning" admin@email.com
fi
```
  - [ ] Cron : toutes les 5 minutes
  - [ ] Alertes par email ou WhatsApp
  - [ ] PM2 : restart automatique si crash

### INFRA-015 : Monitoring des logs
- **Priorité** : P1
- **Complexité** : S
- **Description** : Rotation et gestion des logs
- **Critères d'acceptation** :
  - [ ] Logrotate configuré pour les logs PM2
  - [ ] Rotation hebdomadaire, compression, rétention 4 semaines
  - [ ] Logs Nginx : rotation automatique (déjà configuré par défaut)
  - [ ] Logs PostgreSQL : vérification de la rotation

---

## Module 8 : Sécurité serveur (P0)

### INFRA-016 : Durcissement serveur
- **Priorité** : P0
- **Complexité** : M
- **Description** : Sécurisation du serveur
- **Critères d'acceptation** :
  - [ ] SSH : port changé (optionnel), clé publique only, root login désactivé
  - [ ] Pare-feu UFW activé et configuré
  - [ ] Fail2ban : protection SSH, Nginx, et rate limiting
  - [ ] Mises à jour automatiques de sécurité (`unattended-upgrades`)
  - [ ] Permissions fichiers : applications en user non-root
  - [ ] PostgreSQL : écoute localhost uniquement
  - [ ] `.env` : permissions 600 (lecture propriétaire uniquement)

### INFRA-017 : Headers de sécurité Nginx
- **Priorité** : P0
- **Complexité** : S
- **Description** : Headers HTTP de sécurité
- **Critères d'acceptation** :
  - [ ] `X-Frame-Options: SAMEORIGIN`
  - [ ] `X-Content-Type-Options: nosniff`
  - [ ] `X-XSS-Protection: 1; mode=block`
  - [ ] `Referrer-Policy: strict-origin-when-cross-origin`
  - [ ] `Strict-Transport-Security: max-age=31536000; includeSubDomains`
  - [ ] `Content-Security-Policy` : adapté au projet
  - [ ] Test sur securityheaders.com : note A minimum

---

## Module 9 : CI/CD (P2)

### INFRA-018 : GitHub Actions - CI
- **Priorité** : P2
- **Complexité** : L
- **Description** : Pipeline d'intégration continue
- **Critères d'acceptation** :
  - [ ] Workflow déclenché sur push/PR vers `main`
  - [ ] Steps : install, lint, type-check, test, build
  - [ ] PostgreSQL en service pour les tests
  - [ ] Cache des node_modules pour la performance
  - [ ] Notification en cas d'échec

### INFRA-019 : Déploiement automatique (CD)
- **Priorité** : P2
- **Complexité** : L
- **Description** : Déploiement automatique après merge sur main
- **Critères d'acceptation** :
  - [ ] Webhook GitHub ou GitHub Actions avec SSH
  - [ ] Déploiement automatique après CI réussi
  - [ ] Notification de succès/échec
  - [ ] Rollback automatique si le health check échoue

---

## Module 10 : Performance (P1)

### INFRA-020 : Optimisation Nginx
- **Priorité** : P1
- **Complexité** : S
- **Description** : Optimisations de performance Nginx
- **Critères d'acceptation** :
  - [ ] Compression gzip : text/html, text/css, application/javascript, application/json, image/svg+xml
  - [ ] Cache statique : images 30j, CSS/JS 7j
  - [ ] `worker_processes auto;`
  - [ ] `keepalive_timeout 65;`
  - [ ] `sendfile on;`
  - [ ] HTTP/2 activé

### INFRA-021 : Optimisation Node.js
- **Priorité** : P1
- **Complexité** : S
- **Description** : Optimisations Node.js en production
- **Critères d'acceptation** :
  - [ ] `NODE_ENV=production`
  - [ ] PM2 en mode cluster (2 instances pour l'API)
  - [ ] `max_old_space_size` ajusté selon la RAM disponible
  - [ ] Garbage collection optimisé

---

## Checklist de production

### Avant le déploiement initial

- [ ] Serveur provisionné et accessible en SSH
- [ ] Nom de domaine configuré (DNS propagé)
- [ ] PostgreSQL installé et sécurisé
- [ ] Node.js 20 installé
- [ ] Nginx installé et configuré
- [ ] PM2 installé et configuré
- [ ] SSL (Let's Encrypt) installé
- [ ] Pare-feu configuré (UFW : 22, 80, 443)
- [ ] Fail2ban actif
- [ ] Backup automatique configuré
- [ ] Monitoring en place
- [ ] Script de déploiement testé
- [ ] Variables d'environnement de production définies
- [ ] Seed initial exécuté (admin + catégories)
- [ ] Health check accessible : `https://domaine.com/api/health`

### Objectifs de performance

| Métrique | Cible | Outil de mesure |
|---|---|---|
| Temps de chargement page | < 3 secondes | Google PageSpeed |
| TTFB (Time to First Byte) | < 500ms | WebPageTest |
| Score PageSpeed (mobile) | > 70 | Google PageSpeed |
| Score PageSpeed (desktop) | > 85 | Google PageSpeed |
| Temps de réponse API | < 500ms | curl / Postman |
| Uptime | > 99.5% | Monitoring |
| Taille page accueil | < 2 Mo | DevTools Network |

---

## Résumé des tâches

| ID | Tâche | Priorité | Complexité | Statut |
|---|---|---|---|---|
| INFRA-001 | Docker Compose dev | P0 | M | TODO |
| INFRA-002 | Dockerfile API | P0 | M | TODO |
| INFRA-003 | Dockerfile Web | P0 | M | TODO |
| INFRA-004 | Config serveur | P0 | L | TODO |
| INFRA-005 | Dépendances serveur | P0 | M | TODO |
| INFRA-006 | Config PostgreSQL | P0 | M | TODO |
| INFRA-007 | Config Nginx | P0 | L | TODO |
| INFRA-008 | SSL Let's Encrypt | P0 | S | TODO |
| INFRA-009 | Config PM2 | P0 | M | TODO |
| INFRA-010 | Script déploiement | P0 | M | TODO |
| INFRA-011 | Config DNS | P0 | S | TODO |
| INFRA-012 | Backup BDD | P0 | M | TODO |
| INFRA-013 | Backup fichiers | P1 | S | TODO |
| INFRA-014 | Monitoring applicatif | P1 | M | TODO |
| INFRA-015 | Monitoring logs | P1 | S | TODO |
| INFRA-016 | Durcissement serveur | P0 | M | TODO |
| INFRA-017 | Headers sécurité | P0 | S | TODO |
| INFRA-018 | CI (GitHub Actions) | P2 | L | TODO |
| INFRA-019 | CD automatique | P2 | L | TODO |
| INFRA-020 | Opti Nginx | P1 | S | TODO |
| INFRA-021 | Opti Node.js | P1 | S | TODO |

**Total estimé P0** : ~30h
**Total estimé P1** : ~8h
**Total estimé P2** : ~16h
