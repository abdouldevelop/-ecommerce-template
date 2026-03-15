# Haute Disponibilite et Robustesse - E-Commerce

## Instructions pour l'agent IA

Ce document est le guide complet pour rendre la plateforme e-commerce hautement disponible, performante et resiliente. Il couvre l'architecture multi-serveur, les bases de donnees, le cache, le monitoring, la reprise apres sinistre, la securite en production, la performance et le CI/CD. Chaque section contient des recommandations concretes avec des exemples de configuration.

**Niveaux d'implementation** :
- ESSENTIEL : A mettre en place avant la mise en production
- RECOMMANDE : A planifier dans les 30 premiers jours
- AVANCE : Pour les plateformes a fort trafic (> 10 000 visiteurs/jour)

---

## 1. Architecture multi-serveur

### Schema d'architecture cible

```
                         ┌─────────────────────────┐
                         │     DNS (Cloudflare)     │
                         │   CDN + DDoS Protection  │
                         └────────────┬────────────┘
                                      │ HTTPS
                         ┌────────────▼────────────┐
                         │     Load Balancer        │
                         │  (HAProxy / Nginx / CLB) │
                         │   SSL Termination        │
                         └──────┬───────────┬──────┘
                                │           │
                    ┌───────────▼──┐  ┌─────▼──────────┐
                    │  App Server 1 │  │  App Server 2  │
                    │  Next.js SSR  │  │  Next.js SSR   │
                    │  NestJS API   │  │  NestJS API    │
                    │  PM2 Cluster  │  │  PM2 Cluster   │
                    └───────┬──────┘  └──────┬─────────┘
                            │                │
                    ┌───────▼────────────────▼─────────┐
                    │          Redis Sentinel           │
                    │   Sessions, Cache, Rate Limit     │
                    │   BullMQ (file d'attente)         │
                    ├──────────┬────────────────────────┤
                    │  Master  │  Replica + Sentinel    │
                    └──────────┴────────────────────────┘
                            │
                    ┌───────▼──────────────────────────┐
                    │      PostgreSQL Cluster           │
                    ├──────────┬───────────────────────┤
                    │  Primary │    Read Replica        │
                    │  (Write) │    (Read)              │
                    └──────────┴───────────────────────┘
                            │
                    ┌───────▼──────────────────────────┐
                    │      Stockage                     │
                    │  S3/Backblaze B2 (images)         │
                    │  Volume SSD local (uploads)       │
                    └──────────────────────────────────┘
```

### Load Balancer

**Option 1 : HAProxy (recommande pour le self-hosted)**

```haproxy
# /etc/haproxy/haproxy.cfg
global
    log /dev/log local0
    maxconn 4096
    user haproxy
    group haproxy
    daemon
    tune.ssl.default-dh-param 2048

defaults
    log global
    mode http
    option httplog
    option dontlognull
    option forwardfor
    option http-server-close
    timeout connect 5s
    timeout client 30s
    timeout server 120s
    retries 3

frontend http_front
    bind *:80
    redirect scheme https code 301 if !{ ssl_fc }

frontend https_front
    bind *:443 ssl crt /etc/ssl/certs/ecommerce.pem
    http-request set-header X-Forwarded-Proto https

    # Route statique vers CDN (si pas Cloudflare)
    acl is_static path_beg /uploads/ /_next/static/

    # Route API
    acl is_api path_beg /api/

    use_backend api_servers if is_api
    default_backend web_servers

backend web_servers
    balance roundrobin
    option httpchk GET /api/health
    http-check expect status 200
    cookie SERVERID insert indirect nocache

    server web1 10.0.1.10:3000 check cookie web1 maxconn 500
    server web2 10.0.1.11:3000 check cookie web2 maxconn 500

backend api_servers
    balance leastconn
    option httpchk GET /api/health
    http-check expect status 200

    server api1 10.0.1.10:3001 check maxconn 300
    server api2 10.0.1.11:3001 check maxconn 300
```

**Option 2 : Nginx upstream**

```nginx
# /etc/nginx/conf.d/upstream.conf
upstream web_backend {
    least_conn;
    server 10.0.1.10:3000 max_fails=3 fail_timeout=30s;
    server 10.0.1.11:3000 max_fails=3 fail_timeout=30s;
    keepalive 32;
}

upstream api_backend {
    least_conn;
    server 10.0.1.10:3001 max_fails=3 fail_timeout=30s;
    server 10.0.1.11:3001 max_fails=3 fail_timeout=30s;
    keepalive 32;
}

server {
    listen 443 ssl http2;
    server_name votredomaine.com;

    ssl_certificate /etc/letsencrypt/live/votredomaine.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/votredomaine.com/privkey.pem;

    location / {
        proxy_pass http://web_backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_next_upstream error timeout http_502 http_503;
    }

    location /api/ {
        proxy_pass http://api_backend/;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 120s;
        proxy_send_timeout 120s;
        proxy_read_timeout 120s;
        proxy_next_upstream error timeout http_502 http_503;
    }
}
```

**Option 3 : Cloud Load Balancer (si hebergement cloud)**

| Fournisseur | Service | Avantages |
|---|---|---|
| AWS | ALB (Application Load Balancer) | Integre avec EC2, auto-scaling |
| GCP | Cloud Load Balancing | Global, anycast IP |
| OVH | Load Balancer | Bon rapport qualite/prix, datacenters FR |
| Hetzner | Load Balancer | Tres bon prix, datacenters EU |

### Infrastructure minimale recommandee

| Composant | Minimum (MVP) | Recommande (production) | Avance (fort trafic) |
|---|---|---|---|
| Serveurs app | 1 (PM2 cluster) | 2 serveurs + LB | 3+ serveurs + auto-scaling |
| Base de donnees | 1 PostgreSQL | Primary + 1 Read Replica | Cluster Patroni (3 noeuds) |
| Cache | Redis standalone | Redis Sentinel (3 noeuds) | Redis Cluster (6 noeuds) |
| CDN | Cloudflare (gratuit) | Cloudflare Pro | CloudFront + S3 |
| Stockage images | Local (disque) | S3/Backblaze B2 | S3 + CDN + resize on-the-fly |

---

## 2. Base de donnees

### PostgreSQL Streaming Replication

**Configuration du Primary** (`postgresql.conf`) :

```ini
# /etc/postgresql/16/main/postgresql.conf (Primary)
listen_addresses = 'localhost,10.0.1.10'
port = 5432
max_connections = 200

# WAL et Replication
wal_level = replica
max_wal_senders = 5
wal_keep_size = '1GB'
hot_standby = on
synchronous_commit = on

# Performance
shared_buffers = '1GB'                    # 25% de la RAM (4 Go)
effective_cache_size = '3GB'              # 75% de la RAM
work_mem = '8MB'
maintenance_work_mem = '256MB'
random_page_cost = 1.1                    # SSD
effective_io_concurrency = 200            # SSD

# WAL archiving pour PITR
archive_mode = on
archive_command = 'cp %p /var/backups/postgresql/wal_archive/%f'

# Logging
log_min_duration_statement = 500          # Log les requetes > 500ms
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on

# Statistiques
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.track = all
```

**Configuration du Replica** :

```bash
# Sur le replica, initialiser depuis le primary
pg_basebackup -h 10.0.1.10 -U replication_user -D /var/lib/postgresql/16/main -Fp -Xs -P -R
```

```ini
# /etc/postgresql/16/main/postgresql.conf (Replica)
hot_standby = on
primary_conninfo = 'host=10.0.1.10 port=5432 user=replication_user password=xxx'
```

### Connection Pooling (PgBouncer)

```ini
# /etc/pgbouncer/pgbouncer.ini
[databases]
ecommerce = host=127.0.0.1 port=5432 dbname=ecommerce

[pgbouncer]
listen_addr = 127.0.0.1
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 500
default_pool_size = 25
min_pool_size = 5
reserve_pool_size = 5
reserve_pool_timeout = 3
server_idle_timeout = 600
log_connections = 1
log_disconnections = 1
stats_period = 60
```

**Mise a jour de Prisma pour utiliser PgBouncer** :

```env
# .env (production)
DATABASE_URL="postgresql://ecommerce_user:password@localhost:6432/ecommerce?pgbouncer=true&connection_limit=10"
```

### Strategie de backup

```
┌─────────────────────────────────────────────────────┐
│              Strategie de backup 3-2-1              │
│                                                     │
│  3 copies : production + backup local + off-site    │
│  2 supports : disque SSD + stockage objet (S3/B2)   │
│  1 copie off-site : dans un autre datacenter/region │
└─────────────────────────────────────────────────────┘
```

**Script de backup complet** :

```bash
#!/bin/bash
# /usr/local/bin/backup-ecommerce.sh

set -euo pipefail

# Configuration
DB_NAME="ecommerce"
DB_USER="ecommerce_user"
BACKUP_DIR="/var/backups/postgresql"
WAL_ARCHIVE="/var/backups/postgresql/wal_archive"
S3_BUCKET="s3://ecommerce-backups"
DATE=$(date +%Y%m%d_%H%M%S)
DAY_OF_WEEK=$(date +%u)
RETENTION_DAILY=7
RETENTION_WEEKLY=30
RETENTION_MONTHLY=365

# 1. Dump quotidien (format custom pour pg_restore)
DUMP_FILE="${BACKUP_DIR}/daily/${DB_NAME}_${DATE}.dump"
mkdir -p "${BACKUP_DIR}/daily" "${BACKUP_DIR}/weekly" "${BACKUP_DIR}/monthly"

echo "[$(date)] Debut du backup..."
pg_dump -U $DB_USER -Fc -Z 6 -f "$DUMP_FILE" $DB_NAME
echo "[$(date)] Dump cree : $DUMP_FILE ($(du -sh $DUMP_FILE | cut -f1))"

# 2. Copie hebdomadaire (dimanche)
if [ "$DAY_OF_WEEK" = "7" ]; then
    cp "$DUMP_FILE" "${BACKUP_DIR}/weekly/${DB_NAME}_weekly_${DATE}.dump"
    echo "[$(date)] Copie hebdomadaire creee"
fi

# 3. Copie mensuelle (1er du mois)
if [ "$(date +%d)" = "01" ]; then
    cp "$DUMP_FILE" "${BACKUP_DIR}/monthly/${DB_NAME}_monthly_${DATE}.dump"
    echo "[$(date)] Copie mensuelle creee"
fi

# 4. Upload off-site (S3 ou Backblaze B2)
if command -v aws &> /dev/null; then
    aws s3 cp "$DUMP_FILE" "${S3_BUCKET}/daily/" --storage-class STANDARD_IA
    echo "[$(date)] Upload S3 termine"
elif command -v b2 &> /dev/null; then
    b2 upload-file ecommerce-backups "$DUMP_FILE" "daily/$(basename $DUMP_FILE)"
    echo "[$(date)] Upload B2 termine"
fi

# 5. Nettoyage par retention
find "${BACKUP_DIR}/daily" -name "*.dump" -mtime +$RETENTION_DAILY -delete
find "${BACKUP_DIR}/weekly" -name "*.dump" -mtime +$RETENTION_WEEKLY -delete
find "${BACKUP_DIR}/monthly" -name "*.dump" -mtime +$RETENTION_MONTHLY -delete

# 6. Nettoyage WAL archive (garder 7 jours)
find "$WAL_ARCHIVE" -type f -mtime +7 -delete 2>/dev/null || true

# 7. Verification
DUMP_SIZE=$(du -sb "$DUMP_FILE" | cut -f1)
if [ "$DUMP_SIZE" -lt 1024 ]; then
    echo "ALERTE : Backup suspect (taille < 1 Ko)" | mail -s "Backup E-Commerce ERREUR" admin@votredomaine.com
    exit 1
fi

echo "[$(date)] Backup termine avec succes"
```

**Cron** :

```cron
# /etc/cron.d/ecommerce-backup
# Backup quotidien a 3h du matin
0 3 * * * postgres /usr/local/bin/backup-ecommerce.sh >> /var/log/ecommerce-backup.log 2>&1

# Verification de l'integrite du backup (dimanche a 5h)
0 5 * * 0 postgres /usr/local/bin/verify-backup.sh >> /var/log/ecommerce-backup-verify.log 2>&1
```

### Failover automatique (Patroni)

Patroni gere le failover automatique du cluster PostgreSQL. En cas de panne du primary, un replica est automatiquement promu.

```yaml
# /etc/patroni/config.yml
scope: ecommerce-cluster
name: pg-node1

restapi:
  listen: 0.0.0.0:8008
  connect_address: 10.0.1.10:8008

etcd:
  hosts: 10.0.1.10:2379,10.0.1.11:2379,10.0.1.12:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      parameters:
        max_connections: 200
        shared_buffers: 1GB
        wal_level: replica
        hot_standby: on
        max_wal_senders: 5

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 10.0.1.10:5432
  data_dir: /var/lib/postgresql/16/main
  authentication:
    superuser:
      username: postgres
      password: xxx
    replication:
      username: replication_user
      password: xxx
```

### Monitoring base de donnees

**Requetes essentielles** :

```sql
-- Requetes lentes en cours
SELECT pid, now() - pg_stat_activity.query_start AS duration, query, state
FROM pg_stat_activity
WHERE state != 'idle'
  AND now() - pg_stat_activity.query_start > interval '5 seconds'
ORDER BY duration DESC;

-- Top 10 requetes les plus lentes (pg_stat_statements)
SELECT
    calls,
    mean_exec_time::numeric(10,2) AS avg_ms,
    total_exec_time::numeric(10,2) AS total_ms,
    rows,
    LEFT(query, 100) AS query
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Taux de cache hit (doit etre > 99%)
SELECT
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) * 100 AS cache_hit_ratio
FROM pg_statio_user_tables;

-- Connexions actives par etat
SELECT state, count(*)
FROM pg_stat_activity
GROUP BY state;

-- Taille des tables
SELECT
    relname AS table,
    pg_size_pretty(pg_total_relation_size(relid)) AS total_size,
    pg_size_pretty(pg_relation_size(relid)) AS data_size,
    pg_size_pretty(pg_indexes_size(relid)) AS index_size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC;
```

---

## 3. Cache

### Redis : Sessions, Panier, Rate Limiting

**Configuration Redis Sentinel (HA)** :

```conf
# /etc/redis/redis.conf (Master)
bind 10.0.1.10 127.0.0.1
port 6379
requirepass ecommerce_redis_strong_password
maxmemory 512mb
maxmemory-policy allkeys-lru
save 900 1
save 300 10
save 60 10000
```

```conf
# /etc/redis/sentinel.conf
port 26379
sentinel monitor ecommerce-master 10.0.1.10 6379 2
sentinel auth-pass ecommerce-master ecommerce_redis_strong_password
sentinel down-after-milliseconds ecommerce-master 5000
sentinel failover-timeout ecommerce-master 10000
sentinel parallel-syncs ecommerce-master 1
```

**Utilisation dans NestJS** :

```typescript
// apps/api/src/common/redis/redis.module.ts
import { Module, Global } from '@nestjs/common';
import { Redis } from 'ioredis';

@Global()
@Module({
  providers: [
    {
      provide: 'REDIS_CLIENT',
      useFactory: () => {
        // Mode Sentinel pour HA
        const redis = new Redis({
          sentinels: [
            { host: '10.0.1.10', port: 26379 },
            { host: '10.0.1.11', port: 26379 },
            { host: '10.0.1.12', port: 26379 },
          ],
          name: 'ecommerce-master',
          password: process.env.REDIS_PASSWORD,
          db: 0,
          retryDelayOnFailover: 100,
          maxRetriesPerRequest: 3,
        });

        redis.on('error', (err) => console.error('Redis error:', err));
        redis.on('+failover', () => console.warn('Redis failover detected'));

        return redis;
      },
    },
  ],
  exports: ['REDIS_CLIENT'],
})
export class RedisModule {}
```

### Strategie de cache multi-niveaux

```
┌──────────────────────────────────────────────────────────────┐
│                 Strategie de cache                           │
│                                                              │
│  Niveau 1 : CDN (Cloudflare)          TTL: 30j (statique)   │
│  ├── Images produits, CSS, JS                                │
│  ├── Pages statiques (CGV, A propos)                         │
│  └── Headers: Cache-Control: public, max-age=2592000         │
│                                                              │
│  Niveau 2 : Next.js ISR               TTL: 60-300s          │
│  ├── Pages produits (/products/[slug])                       │
│  ├── Page d'accueil (produits vedettes)                      │
│  └── Pages categories                                        │
│                                                              │
│  Niveau 3 : Redis                     TTL: 120-600s          │
│  ├── Categories (TTL: 5 min)                                 │
│  ├── Parametres site (TTL: 5 min)                            │
│  ├── Produits featured (TTL: 2 min)                          │
│  ├── Dashboard stats (TTL: 30s)                              │
│  ├── Sessions utilisateur (TTL: 30 min)                      │
│  └── Rate limiting counters (TTL: 60s)                       │
│                                                              │
│  Niveau 4 : In-memory (NestJS)        TTL: 30-120s          │
│  └── Parametres systeme (TTL: 2 min)                         │
└──────────────────────────────────────────────────────────────┘
```

### Cache invalidation

```typescript
// apps/api/src/common/cache/cache.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { Redis } from 'ioredis';

@Injectable()
export class CacheService {
  constructor(@Inject('REDIS_CLIENT') private redis: Redis) {}

  // Prefixes pour organiser les cles
  private PREFIXES = {
    CATEGORIES: 'cache:categories',
    SETTINGS: 'cache:settings',
    PRODUCTS_FEATURED: 'cache:products:featured',
    PRODUCT: 'cache:product',
    DASHBOARD: 'cache:dashboard',
  };

  async get<T>(key: string): Promise<T | null> {
    const data = await this.redis.get(key);
    return data ? JSON.parse(data) : null;
  }

  async set(key: string, value: any, ttlSeconds: number): Promise<void> {
    await this.redis.setex(key, ttlSeconds, JSON.stringify(value));
  }

  // Invalidation par pattern (quand un produit est modifie)
  async invalidatePattern(pattern: string): Promise<void> {
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }

  // Invalidation ciblee
  async invalidateProducts(): Promise<void> {
    await this.invalidatePattern('cache:product:*');
    await this.redis.del(this.PREFIXES.PRODUCTS_FEATURED);
    await this.redis.del(this.PREFIXES.DASHBOARD);
  }

  async invalidateCategories(): Promise<void> {
    await this.redis.del(this.PREFIXES.CATEGORIES);
  }

  async invalidateSettings(): Promise<void> {
    await this.redis.del(this.PREFIXES.SETTINGS);
  }
}
```

### Next.js ISR (Incremental Static Regeneration)

```typescript
// apps/web/app/(shop)/products/[slug]/page.tsx
export const revalidate = 300; // Revalide toutes les 5 minutes

export async function generateStaticParams() {
  const products = await api.get('/products?limit=100');
  return products.data.map((p: any) => ({ slug: p.slug }));
}

export default async function ProductPage({ params }: { params: { slug: string } }) {
  const product = await api.get(`/products/${params.slug}`);
  // ...
}
```

```typescript
// apps/web/app/(shop)/page.tsx (accueil)
export const revalidate = 60; // Revalide toutes les minutes

export default async function HomePage() {
  const [featured, categories] = await Promise.all([
    api.get('/products/featured?limit=8'),
    api.get('/categories'),
  ]);
  // ...
}
```

### CDN - Configuration Cloudflare

| Regle | Chemin | TTL | Cache Level |
|---|---|---|---|
| Assets statiques | `/_next/static/*` | 30 jours | Cache Everything |
| Images produits | `/uploads/products/*` | 7 jours | Cache Everything |
| Images categories | `/uploads/categories/*` | 7 jours | Cache Everything |
| API | `/api/*` | Bypass | Bypass |
| Pages HTML | `/*` | Bypass | Standard |

---

## 4. Application

### PM2 Cluster Mode

```javascript
// ecosystem.config.js (production HA)
module.exports = {
  apps: [
    {
      name: 'ecommerce-api',
      cwd: '/var/www/ecommerce/apps/api',
      script: 'dist/main.js',
      instances: 'max',             // Utilise tous les CPU
      exec_mode: 'cluster',
      max_memory_restart: '500M',
      env: {
        NODE_ENV: 'production',
        PORT: 3001,
      },

      // Graceful shutdown
      kill_timeout: 5000,           // 5s pour finir les requetes en cours
      listen_timeout: 10000,        // 10s max pour demarrer
      shutdown_with_message: true,

      // Logs
      error_file: '/var/log/pm2/api-error.log',
      out_file: '/var/log/pm2/api-out.log',
      merge_logs: true,
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',

      // Restart policy
      exp_backoff_restart_delay: 100,   // Backoff exponentiel
      max_restarts: 10,
      min_uptime: '10s',

      // Monitoring
      pmx: true,
    },
    {
      name: 'ecommerce-web',
      cwd: '/var/www/ecommerce/apps/web',
      script: 'node_modules/.bin/next',
      args: 'start -p 3000',
      instances: 2,                 // 2 instances minimum
      exec_mode: 'cluster',
      max_memory_restart: '500M',
      env: {
        NODE_ENV: 'production',
      },
      kill_timeout: 5000,
      listen_timeout: 15000,
      error_file: '/var/log/pm2/web-error.log',
      out_file: '/var/log/pm2/web-out.log',
      merge_logs: true,
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      exp_backoff_restart_delay: 100,
      max_restarts: 10,
      min_uptime: '10s',
    },
    {
      name: 'ecommerce-worker',
      cwd: '/var/www/ecommerce/apps/api',
      script: 'dist/worker.js',
      instances: 1,
      exec_mode: 'fork',
      max_memory_restart: '300M',
      env: {
        NODE_ENV: 'production',
      },
      error_file: '/var/log/pm2/worker-error.log',
      out_file: '/var/log/pm2/worker-out.log',
    },
  ],
};
```

### Graceful Shutdown

```typescript
// apps/api/src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Graceful shutdown
  app.enableShutdownHooks();

  // Ecouter les signaux d'arret
  process.on('SIGTERM', async () => {
    console.log('SIGTERM recu - arret gracieux...');
    // Arreter d'accepter de nouvelles connexions
    // Finir les requetes en cours
    await app.close();
    process.exit(0);
  });

  process.on('SIGINT', async () => {
    console.log('SIGINT recu - arret gracieux...');
    await app.close();
    process.exit(0);
  });

  await app.listen(process.env.PORT || 3001);
  console.log(`API lancee sur le port ${process.env.PORT || 3001}`);
}
bootstrap();
```

### Zero-Downtime Deploy

```bash
#!/bin/bash
# /usr/local/bin/deploy-zero-downtime.sh
set -euo pipefail

PROJECT_DIR="/var/www/ecommerce"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo "=== Deploiement zero-downtime E-Commerce ==="
echo "Date : $(date)"
echo "Commit : $(cd $PROJECT_DIR && git log --oneline -1)"

# 1. Tag de rollback
cd $PROJECT_DIR
git tag "deploy-$TIMESTAMP"

# 2. Pull du code
git pull origin main

# 3. Installation
npm ci --production

# 4. Migrations (si necessaire)
cd apps/api
npx prisma migrate deploy
npx prisma generate
cd ../..

# 5. Build
npm run build

# 6. Reload gracieux (zero-downtime)
pm2 reload ecosystem.config.js --update-env

# 7. Attendre que toutes les instances soient pretes
sleep 5

# 8. Health check
MAX_RETRIES=10
for i in $(seq 1 $MAX_RETRIES); do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3001/api/health)
    if [ "$STATUS" = "200" ]; then
        echo "Health check OK (tentative $i/$MAX_RETRIES)"
        break
    fi
    if [ "$i" = "$MAX_RETRIES" ]; then
        echo "ERREUR : Health check echoue apres $MAX_RETRIES tentatives"
        echo "Rollback en cours..."
        git checkout "deploy-$TIMESTAMP"
        npm ci --production
        npm run build
        pm2 reload ecosystem.config.js
        exit 1
    fi
    echo "Health check echoue (tentative $i/$MAX_RETRIES), retry dans 3s..."
    sleep 3
done

pm2 status
echo "=== Deploiement termine avec succes ==="
```

### Health Check Endpoints

```typescript
// apps/api/src/modules/health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { PrismaService } from '../../prisma/prisma.service';

@Controller('health')
export class HealthController {
  constructor(
    private prisma: PrismaService,
    @Inject('REDIS_CLIENT') private redis: Redis,
  ) {}

  @Get()
  async check() {
    const checks = {
      status: 'ok',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      memory: process.memoryUsage(),
      database: 'disconnected',
      redis: 'disconnected',
    };

    // Verifier PostgreSQL
    try {
      await this.prisma.$queryRaw`SELECT 1`;
      checks.database = 'connected';
    } catch (error) {
      checks.status = 'degraded';
      checks.database = 'error';
    }

    // Verifier Redis
    try {
      await this.redis.ping();
      checks.redis = 'connected';
    } catch (error) {
      checks.status = 'degraded';
      checks.redis = 'error';
    }

    return checks;
  }

  @Get('ready')
  async readiness() {
    // Readiness probe pour Kubernetes / Load Balancer
    try {
      await this.prisma.$queryRaw`SELECT 1`;
      await this.redis.ping();
      return { status: 'ready' };
    } catch (error) {
      throw new ServiceUnavailableException('Service not ready');
    }
  }

  @Get('live')
  liveness() {
    // Liveness probe - juste verifier que le process tourne
    return { status: 'alive', pid: process.pid };
  }
}
```

### Circuit Breaker (services externes)

```typescript
// apps/api/src/common/circuit-breaker/circuit-breaker.ts
export class CircuitBreaker {
  private failures = 0;
  private lastFailure: Date | null = null;
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';

  constructor(
    private readonly threshold: number = 5,      // Ouvrir apres 5 echecs
    private readonly timeout: number = 30000,     // 30s avant de retester
    private readonly name: string = 'default',
  ) {}

  async execute<T>(fn: () => Promise<T>, fallback?: () => T): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailure!.getTime() > this.timeout) {
        this.state = 'HALF_OPEN';
        console.log(`[CircuitBreaker:${this.name}] HALF_OPEN - tentative de recovery`);
      } else {
        console.warn(`[CircuitBreaker:${this.name}] OPEN - appel bloque`);
        if (fallback) return fallback();
        throw new Error(`Circuit breaker OPEN pour ${this.name}`);
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      if (fallback) return fallback();
      throw error;
    }
  }

  private onSuccess(): void {
    this.failures = 0;
    if (this.state === 'HALF_OPEN') {
      this.state = 'CLOSED';
      console.log(`[CircuitBreaker:${this.name}] CLOSED - service retabli`);
    }
  }

  private onFailure(): void {
    this.failures++;
    this.lastFailure = new Date();
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
      console.error(`[CircuitBreaker:${this.name}] OPEN - ${this.failures} echecs consecutifs`);
    }
  }

  getState() {
    return { name: this.name, state: this.state, failures: this.failures };
  }
}

// Utilisation dans le service de paiement
// const paymentCircuitBreaker = new CircuitBreaker(3, 60000, 'PaiementPro');
// const result = await paymentCircuitBreaker.execute(
//   () => this.callPaiementPro(data),
//   () => ({ error: 'Service de paiement temporairement indisponible' })
// );
```

### Queue System (BullMQ)

```typescript
// apps/api/src/common/queue/queue.module.ts
import { BullModule } from '@nestjs/bullmq';

@Module({
  imports: [
    BullModule.forRoot({
      connection: {
        host: process.env.REDIS_HOST || '127.0.0.1',
        port: parseInt(process.env.REDIS_PORT || '6379'),
        password: process.env.REDIS_PASSWORD,
      },
      defaultJobOptions: {
        attempts: 3,
        backoff: { type: 'exponential', delay: 1000 },
        removeOnComplete: 100,
        removeOnFail: 500,
      },
    }),
    BullModule.registerQueue(
      { name: 'email' },
      { name: 'payment-check' },
      { name: 'image-processing' },
      { name: 'analytics' },
    ),
  ],
})
export class QueueModule {}
```

**Jobs recommandes a mettre en queue** :

| Queue | Jobs | Description |
|---|---|---|
| `email` | Confirmation commande, changement statut, bienvenue | Emails transactionnels |
| `payment-check` | Verification paiement, relance | Verifier le statut des paiements en attente |
| `image-processing` | Resize, conversion WebP, thumbnail | Traitement des images uploadees |
| `analytics` | Calcul stats, mise a jour viewCount | Calculs lourds en arriere-plan |

---

## 5. Monitoring et Alerting

### Stack de monitoring recommandee

```
┌───────────────────────────────────────────────────────┐
│               Stack de Monitoring                      │
│                                                        │
│  ┌──────────────┐  ┌───────────────┐  ┌────────────┐  │
│  │   Netdata    │  │  UptimeRobot  │  │   Loki     │  │
│  │  (systeme)   │  │  (externe)    │  │   (logs)   │  │
│  │  CPU, RAM    │  │  Uptime check │  │  Requetes  │  │
│  │  Disque, IO  │  │  Latence      │  │  Erreurs   │  │
│  │  Reseau      │  │  SSL expiry   │  │  Acces     │  │
│  └──────┬───────┘  └──────┬────────┘  └─────┬──────┘  │
│         │                 │                  │          │
│  ┌──────▼─────────────────▼──────────────────▼──────┐  │
│  │              Grafana (Dashboard)                  │  │
│  │  Tableaux de bord unifies                        │  │
│  │  Alertes multi-canal                             │  │
│  └──────────────────┬───────────────────────────────┘  │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────────┐  │
│  │         Canaux d'alerte                           │  │
│  │  Email | Slack | Telegram | SMS (PagerDuty)       │  │
│  └──────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────┘
```

### Installation Netdata (monitoring systeme)

```bash
# Installation en une commande
bash <(curl -Ss https://my-netdata.io/kickstart.sh)

# Configuration des alertes
# /etc/netdata/health.d/ecommerce.conf
alarm: ecommerce_api_down
    on: apps.cpu
lookup: average -1m unaligned of ecommerce-api
  every: 10s
   warn: $this == 0
   crit: $this == 0
  delay: up 0 down 60 multiplier 1.5 max 3600
   info: L'API e-commerce ne consomme plus de CPU (probablement down)
     to: sysadmin

alarm: disk_space_critical
    on: disk.space
lookup: average -1m unaligned of /
  every: 30s
   crit: $this < 10
   info: Espace disque < 10%
     to: sysadmin
```

### Monitoring externe (UptimeRobot / BetterStack)

**Checks a configurer** :

| Check | URL | Intervalle | Alerte si |
|---|---|---|---|
| API Health | `https://domaine.com/api/health` | 1 min | Status != 200 ou timeout > 5s |
| Frontend | `https://domaine.com` | 1 min | Status != 200 |
| SSL Expiry | `https://domaine.com` | 12h | Expire dans < 14 jours |
| DNS | `domaine.com` | 5 min | Resolution echouee |
| Page produit | `https://domaine.com/products` | 5 min | Status != 200 |

### Log aggregation

**Configuration Loki + Promtail** :

```yaml
# /etc/promtail/config.yml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
  - job_name: pm2-logs
    static_configs:
      - targets: [localhost]
        labels:
          job: ecommerce
          __path__: /var/log/pm2/*.log

  - job_name: nginx-logs
    static_configs:
      - targets: [localhost]
        labels:
          job: nginx
          __path__: /var/log/nginx/*.log

  - job_name: postgresql-logs
    static_configs:
      - targets: [localhost]
        labels:
          job: postgresql
          __path__: /var/log/postgresql/*.log
```

### Metriques cles a surveiller

| Categorie | Metrique | Seuil Warning | Seuil Critique | Action |
|---|---|---|---|---|
| **Systeme** | CPU | > 70% pendant 5 min | > 90% pendant 2 min | Investiguer / scaler |
| **Systeme** | RAM | > 80% | > 95% | Investiguer les fuites memoire |
| **Systeme** | Disque | > 80% | > 90% | Nettoyer logs / augmenter |
| **API** | Temps de reponse p95 | > 500ms | > 2000ms | Optimiser les requetes |
| **API** | Taux d'erreur 5xx | > 1% | > 5% | Investiguer les erreurs |
| **API** | Requetes/seconde | Anomalie (-50%) | 0 req/s | Verifier le service |
| **DB** | Connexions actives | > 80% du max | > 95% du max | Augmenter pool / optimiser |
| **DB** | Replication lag | > 1s | > 10s | Verifier le replica |
| **DB** | Cache hit ratio | < 99% | < 95% | Augmenter shared_buffers |
| **Redis** | Memoire | > 80% du max | > 95% du max | Augmenter / purger |
| **Redis** | Connexions | > 80% du max | > 95% du max | Investiguer |

### Script d'alerte complet

```bash
#!/bin/bash
# /usr/local/bin/check-ecommerce.sh
# Execute toutes les 2 minutes par cron

DOMAIN="votredomaine.com"
SLACK_WEBHOOK="https://hooks.slack.com/services/xxx"
ALERT_EMAIL="admin@votredomaine.com"

send_alert() {
    local severity=$1
    local message=$2

    # Email
    echo "$message" | mail -s "[$severity] E-Commerce Alert" $ALERT_EMAIL

    # Slack
    curl -s -X POST "$SLACK_WEBHOOK" \
        -H 'Content-type: application/json' \
        -d "{\"text\": \":warning: *[$severity]* $message\"}" > /dev/null 2>&1
}

# 1. API Health
API_RESPONSE=$(curl -s -o /tmp/health.json -w "%{http_code}:%{time_total}" \
    --max-time 10 "http://localhost:3001/api/health")
API_CODE=$(echo $API_RESPONSE | cut -d: -f1)
API_TIME=$(echo $API_RESPONSE | cut -d: -f2)

if [ "$API_CODE" != "200" ]; then
    send_alert "CRITIQUE" "API down (HTTP $API_CODE) - auto-restart en cours"
    pm2 restart ecommerce-api
fi

# 2. Frontend
WEB_CODE=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 "http://localhost:3000")
if [ "$WEB_CODE" != "200" ]; then
    send_alert "CRITIQUE" "Frontend down (HTTP $WEB_CODE) - auto-restart en cours"
    pm2 restart ecommerce-web
fi

# 3. PostgreSQL
if ! pg_isready -q -h localhost -p 5432; then
    send_alert "CRITIQUE" "PostgreSQL est injoignable"
fi

# 4. Redis
if ! redis-cli -a "$REDIS_PASSWORD" ping > /dev/null 2>&1; then
    send_alert "ELEVE" "Redis est injoignable"
fi

# 5. Espace disque
DISK=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
if [ "$DISK" -gt 90 ]; then
    send_alert "CRITIQUE" "Espace disque critique : ${DISK}%"
elif [ "$DISK" -gt 80 ]; then
    send_alert "WARNING" "Espace disque bas : ${DISK}%"
fi

# 6. Memoire
MEM=$(free | awk '/Mem:/ {printf "%.0f", $3/$2 * 100}')
if [ "$MEM" -gt 95 ]; then
    send_alert "CRITIQUE" "Memoire critique : ${MEM}%"
fi

# 7. Temps de reponse API
API_MS=$(echo "$API_TIME * 1000" | bc 2>/dev/null | cut -d. -f1)
if [ "${API_MS:-0}" -gt 2000 ]; then
    send_alert "WARNING" "API lente : ${API_MS}ms"
fi
```

---

## 6. Disaster Recovery (Plan de reprise)

### Objectifs de reprise

| Metrique | Cible MVP | Cible Production | Cible Enterprise |
|---|---|---|---|
| **RTO** (temps de reprise) | < 4 heures | < 1 heure | < 15 minutes |
| **RPO** (perte de donnees max) | < 24 heures | < 1 heure | < 5 minutes |
| **Uptime cible** | 99% (87h d'arret/an) | 99.9% (8.7h/an) | 99.95% (4.4h/an) |

### Matrice des scenarios de panne

| Scenario | Probabilite | Impact | RTO cible | Procedure |
|---|---|---|---|---|
| Crash application (PM2) | Frequent | Faible | < 1 min | Auto-restart PM2 |
| Panne serveur unique | Moyen | Moyen | < 15 min | Failover LB vers serveur 2 |
| Corruption base de donnees | Rare | Critique | < 1 heure | Restore depuis backup |
| Panne datacenter | Tres rare | Critique | < 4 heures | Deployer sur datacenter backup |
| Attaque DDoS | Moyen | Moyen | < 30 min | Cloudflare Under Attack Mode |
| Compromission serveur | Rare | Critique | < 2 heures | Isolation, rebuild, restore |
| Erreur de deploiement | Moyen | Moyen | < 5 min | Rollback Git + PM2 reload |

### Runbook : Restauration complete

```bash
#!/bin/bash
# /usr/local/bin/disaster-recovery.sh
# Procedure de restauration complete sur un nouveau serveur

set -euo pipefail

echo "=== PROCEDURE DE DISASTER RECOVERY ==="
echo "Date : $(date)"
echo "ATTENTION : Cette procedure va restaurer la plateforme depuis un backup"
echo ""

# Variables
BACKUP_FILE=$1  # Chemin du fichier de backup (.dump)
DB_NAME="ecommerce"
DB_USER="ecommerce_user"
DB_PASS="CHANGE_ME"
PROJECT_DIR="/var/www/ecommerce"

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage : $0 <backup_file.dump>"
    exit 1
fi

# 1. Verifier les prerequisites
echo "[1/8] Verification des prerequis..."
command -v node >/dev/null || { echo "Node.js requis"; exit 1; }
command -v psql >/dev/null || { echo "PostgreSQL requis"; exit 1; }
command -v pm2 >/dev/null || { echo "PM2 requis"; exit 1; }
command -v nginx >/dev/null || { echo "Nginx requis"; exit 1; }

# 2. Restaurer la base de donnees
echo "[2/8] Restauration de la base de donnees..."
sudo -u postgres createuser --createdb $DB_USER 2>/dev/null || true
sudo -u postgres createdb -O $DB_USER $DB_NAME 2>/dev/null || true
pg_restore -U $DB_USER -d $DB_NAME -c --if-exists "$BACKUP_FILE"
echo "Base restauree avec succes"

# 3. Cloner le code
echo "[3/8] Clonage du code source..."
if [ ! -d "$PROJECT_DIR" ]; then
    git clone git@github.com:org/ecommerce.git $PROJECT_DIR
fi
cd $PROJECT_DIR
git checkout main
git pull origin main

# 4. Installation
echo "[4/8] Installation des dependances..."
npm ci --production

# 5. Regenerer Prisma
echo "[5/8] Generation du client Prisma..."
cd apps/api
npx prisma generate
cd ../..

# 6. Build
echo "[6/8] Build de l'application..."
npm run build

# 7. Demarrer PM2
echo "[7/8] Demarrage des services..."
pm2 start ecosystem.config.js
pm2 save
pm2 startup

# 8. Verification
echo "[8/8] Verification..."
sleep 5
curl -s http://localhost:3001/api/health | python3 -m json.tool
pm2 status

echo ""
echo "=== DISASTER RECOVERY TERMINE ==="
echo "Verifier : https://votredomaine.com"
echo "Verifier : https://votredomaine.com/api/health"
```

### Verification mensuelle des backups

```bash
#!/bin/bash
# /usr/local/bin/verify-backup.sh
# A executer mensuellement pour verifier que les backups sont restaurables

set -euo pipefail

BACKUP_DIR="/var/backups/postgresql"
TEST_DB="ecommerce_test_restore"
LATEST_BACKUP=$(ls -t ${BACKUP_DIR}/daily/*.dump 2>/dev/null | head -1)

if [ -z "$LATEST_BACKUP" ]; then
    echo "ERREUR : Aucun backup trouve" | mail -s "Backup Verification ECHEC" admin@votredomaine.com
    exit 1
fi

echo "Verification du backup : $LATEST_BACKUP"

# Creer une base temporaire
sudo -u postgres dropdb --if-exists $TEST_DB
sudo -u postgres createdb $TEST_DB

# Restaurer
if pg_restore -U postgres -d $TEST_DB "$LATEST_BACKUP" 2>/dev/null; then
    # Verifier les donnees
    USERS=$(psql -U postgres -d $TEST_DB -t -c "SELECT count(*) FROM \"User\";")
    PRODUCTS=$(psql -U postgres -d $TEST_DB -t -c "SELECT count(*) FROM \"Product\";")
    ORDERS=$(psql -U postgres -d $TEST_DB -t -c "SELECT count(*) FROM \"Order\";")

    echo "Verification OK :"
    echo "  - Utilisateurs : $USERS"
    echo "  - Produits : $PRODUCTS"
    echo "  - Commandes : $ORDERS"

    echo "Backup verification SUCCES - Users:$USERS Products:$PRODUCTS Orders:$ORDERS" | \
        mail -s "Backup Verification OK" admin@votredomaine.com
else
    echo "ERREUR : Restauration echouee" | mail -s "Backup Verification ECHEC" admin@votredomaine.com
fi

# Nettoyage
sudo -u postgres dropdb --if-exists $TEST_DB
```

### Backups off-site

| Fournisseur | Service | Cout estimatif | Avantages |
|---|---|---|---|
| Backblaze B2 | Stockage objet | ~$5/To/mois | Tres abordable, API S3-compatible |
| AWS S3 | S3 Glacier | ~$4/To/mois | Standard, tres fiable |
| OVH | Object Storage | ~$10/To/mois | Datacenters en France |
| Hetzner | Storage Box | ~$3.5/To/mois | Bon prix, EU |

---

## 7. Securite en production

### WAF (Web Application Firewall)

**Cloudflare WAF (recommande)** :

| Regle | Action | Description |
|---|---|---|
| OWASP Core Ruleset | Block | Protection contre les attaques courantes |
| SQL Injection | Block | Detection automatique d'injections SQL |
| XSS | Block | Detection de scripts malveillants |
| Bot Management | Challenge | Bloquer les bots malveillants |
| Geo-blocking | Block | Bloquer les pays a risque (optionnel) |
| Rate Limiting | Block/Challenge | Limiter les requetes abusives |

### DDoS Protection

```
┌─────────────────────────────────────────────────────┐
│          Protection DDoS multi-niveaux               │
│                                                      │
│  Niveau 1 : Cloudflare (ou AWS Shield)               │
│  ├── Absorption des attaques volumetriques           │
│  ├── Mode "Under Attack" (CAPTCHA global)            │
│  └── Anycast DNS (distribution geographique)         │
│                                                      │
│  Niveau 2 : Nginx rate limiting                      │
│  ├── limit_req_zone $binary_remote_addr              │
│  │   zone=api:10m rate=30r/s                         │
│  ├── limit_req_zone $binary_remote_addr              │
│  │   zone=login:10m rate=5r/m                        │
│  └── limit_conn_zone $binary_remote_addr             │
│      zone=addr:10m max=50                            │
│                                                      │
│  Niveau 3 : Application (NestJS Throttler)           │
│  ├── Global : 100 req/min par IP                     │
│  ├── Auth : 5 req/min par IP                         │
│  └── Payment : 10 req/min par utilisateur            │
└─────────────────────────────────────────────────────┘
```

**Configuration Nginx rate limiting** :

```nginx
# /etc/nginx/conf.d/rate-limiting.conf
limit_req_zone $binary_remote_addr zone=global:10m rate=30r/s;
limit_req_zone $binary_remote_addr zone=api:10m rate=20r/s;
limit_req_zone $binary_remote_addr zone=auth:10m rate=5r/m;
limit_req_zone $binary_remote_addr zone=payment:10m rate=10r/m;
limit_conn_zone $binary_remote_addr zone=addr:10m;

server {
    # ...

    # Global
    limit_conn addr 50;

    location /api/auth/ {
        limit_req zone=auth burst=3 nodelay;
        limit_req_status 429;
        proxy_pass http://api_backend/auth/;
    }

    location /api/payments/ {
        limit_req zone=payment burst=5 nodelay;
        proxy_pass http://api_backend/payments/;
    }

    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://api_backend/;
    }
}
```

### Secrets Management

**Niveaux de gestion des secrets** :

| Niveau | Methode | Quand l'utiliser |
|---|---|---|
| Basique | Fichier `.env` + permissions 600 | MVP, petit projet |
| Intermediaire | Variables d'environnement systeme + encrypted `.env` | Production standard |
| Avance | HashiCorp Vault / AWS Secrets Manager | Multi-serveurs, conformite |

**Configuration Vault (avance)** :

```bash
# Stocker les secrets
vault kv put secret/ecommerce/db \
    url="postgresql://user:pass@host:5432/db" \
    password="xxx"

vault kv put secret/ecommerce/jwt \
    secret="xxx" \
    refresh_secret="xxx"

vault kv put secret/ecommerce/payment \
    merchant_id="xxx" \
    api_key="xxx" \
    api_secret="xxx"
```

### Security Headers complets

```nginx
# /etc/nginx/conf.d/security-headers.conf
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
add_header Content-Security-Policy "
    default-src 'self';
    script-src 'self' 'unsafe-inline' 'unsafe-eval';
    style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
    font-src 'self' https://fonts.gstatic.com;
    img-src 'self' data: https: blob:;
    connect-src 'self' https://api.paiementpro.net;
    frame-src 'self' https://www.paiementpro.net;
    object-src 'none';
    base-uri 'self';
    form-action 'self';
" always;

# Masquer les informations serveur
server_tokens off;
more_clear_headers Server;
```

### Scan de vulnerabilites automatise

```bash
#!/bin/bash
# /usr/local/bin/security-scan.sh
# A executer hebdomadairement

echo "=== Scan de securite E-Commerce ==="

# 1. Audit npm
echo "--- Audit npm ---"
cd /var/www/ecommerce
npm audit --production 2>&1 | tail -5

# 2. Verifier les ports ouverts
echo "--- Ports ouverts ---"
ss -tlnp | grep LISTEN

# 3. Verifier les connexions PostgreSQL
echo "--- Connexions PostgreSQL ---"
sudo -u postgres psql -c "SELECT count(*), state FROM pg_stat_activity GROUP BY state;"

# 4. Verifier les certificats SSL
echo "--- Certificat SSL ---"
EXPIRY=$(echo | openssl s_client -servername votredomaine.com -connect votredomaine.com:443 2>/dev/null | \
    openssl x509 -noout -dates 2>/dev/null | grep notAfter | cut -d= -f2)
echo "Expiration : $EXPIRY"

# 5. Verifier les mises a jour de securite
echo "--- Mises a jour de securite ---"
apt list --upgradable 2>/dev/null | grep -i security | head -5

echo "=== Fin du scan ==="
```

---

## 8. Performance

### Load Testing (k6)

```javascript
// tests/load/homepage.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 20 },    // Montee progressive
    { duration: '3m', target: 50 },    // Charge soutenue
    { duration: '1m', target: 100 },   // Pic de charge
    { duration: '2m', target: 50 },    // Redescente
    { duration: '1m', target: 0 },     // Arret
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],
    http_req_failed: ['rate<0.01'],
  },
};

const BASE_URL = __ENV.BASE_URL || 'https://votredomaine.com';

export default function () {
  // 1. Page d'accueil
  let res = http.get(`${BASE_URL}/`);
  check(res, {
    'homepage status 200': (r) => r.status === 200,
    'homepage < 2s': (r) => r.timings.duration < 2000,
  });

  sleep(1);

  // 2. API Produits
  res = http.get(`${BASE_URL}/api/products?limit=20`);
  check(res, {
    'products status 200': (r) => r.status === 200,
    'products < 500ms': (r) => r.timings.duration < 500,
    'products has data': (r) => JSON.parse(r.body).data.length > 0,
  });

  sleep(1);

  // 3. API Categories
  res = http.get(`${BASE_URL}/api/categories`);
  check(res, {
    'categories status 200': (r) => r.status === 200,
    'categories < 200ms': (r) => r.timings.duration < 200,
  });

  sleep(2);

  // 4. Recherche
  res = http.get(`${BASE_URL}/api/products/search?q=chemise`);
  check(res, {
    'search status 200': (r) => r.status === 200,
    'search < 300ms': (r) => r.timings.duration < 300,
  });

  sleep(1);
}
```

```javascript
// tests/load/checkout-stress.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  scenarios: {
    concurrent_checkouts: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '30s', target: 10 },
        { duration: '2m', target: 30 },
        { duration: '30s', target: 0 },
      ],
    },
  },
  thresholds: {
    http_req_duration: ['p(95)<2000'],
    http_req_failed: ['rate<0.05'],
  },
};

export default function () {
  const BASE_URL = __ENV.BASE_URL || 'https://votredomaine.com';

  // 1. Login
  const loginRes = http.post(`${BASE_URL}/api/auth/login`, JSON.stringify({
    email: `testuser${__VU}@test.com`,
    password: 'TestPassword123!',
  }), { headers: { 'Content-Type': 'application/json' } });

  if (loginRes.status !== 200) return;
  const token = JSON.parse(loginRes.body).accessToken;
  const headers = {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`,
  };

  // 2. Ajouter au panier
  http.post(`${BASE_URL}/api/cart/items`, JSON.stringify({
    productId: 'test-product-id',
    quantity: 1,
  }), { headers });

  sleep(1);

  // 3. Creer la commande
  const orderRes = http.post(`${BASE_URL}/api/orders`, JSON.stringify({
    addressId: 'test-address-id',
    paymentMethod: 'ORANGE_MONEY',
  }), { headers });

  check(orderRes, {
    'order created': (r) => r.status === 201,
    'order < 3s': (r) => r.timings.duration < 3000,
  });

  sleep(2);
}
```

**Execution** :

```bash
# Test de charge basique
k6 run tests/load/homepage.js

# Test avec variable d'environnement
k6 run -e BASE_URL=https://staging.votredomaine.com tests/load/homepage.js

# Test de stress checkout
k6 run tests/load/checkout-stress.js
```

### Performance Budgets

| Metrique | Budget | Outil de mesure |
|---|---|---|
| **TTFB** (Time to First Byte) | < 200ms | WebPageTest, Lighthouse |
| **FCP** (First Contentful Paint) | < 1.8s | Lighthouse |
| **LCP** (Largest Contentful Paint) | < 2.5s | Lighthouse |
| **TBT** (Total Blocking Time) | < 200ms | Lighthouse |
| **CLS** (Cumulative Layout Shift) | < 0.1 | Lighthouse |
| **API p95** | < 500ms | k6 / monitoring |
| **API p99** | < 1000ms | k6 / monitoring |
| **Bundle JS** | < 200 Ko (gzip) | Webpack Analyzer |
| **Page HTML** | < 50 Ko | DevTools |
| **Taille totale page** | < 1.5 Mo | DevTools Network |
| **Score Lighthouse (mobile)** | > 80 | Lighthouse |
| **Score Lighthouse (desktop)** | > 90 | Lighthouse |

### Optimisation des requetes PostgreSQL

```sql
-- Index essentiels pour les performances e-commerce
CREATE INDEX CONCURRENTLY idx_products_category ON "Product"("categoryId") WHERE "status" = 'ACTIVE';
CREATE INDEX CONCURRENTLY idx_products_featured ON "Product"("isFeatured") WHERE "status" = 'ACTIVE' AND "isFeatured" = true;
CREATE INDEX CONCURRENTLY idx_products_search ON "Product" USING gin(to_tsvector('french', "name" || ' ' || COALESCE("description", '')));
CREATE INDEX CONCURRENTLY idx_products_slug ON "Product"("slug");
CREATE INDEX CONCURRENTLY idx_products_price ON "Product"("price") WHERE "status" = 'ACTIVE';

CREATE INDEX CONCURRENTLY idx_orders_user ON "Order"("userId");
CREATE INDEX CONCURRENTLY idx_orders_status ON "Order"("status");
CREATE INDEX CONCURRENTLY idx_orders_created ON "Order"("createdAt" DESC);
CREATE INDEX CONCURRENTLY idx_orders_number ON "Order"("orderNumber");

CREATE INDEX CONCURRENTLY idx_cart_user ON "Cart"("userId");
CREATE INDEX CONCURRENTLY idx_cart_items_product ON "CartItem"("productId");

CREATE INDEX CONCURRENTLY idx_users_email ON "User"("email");
CREATE INDEX CONCURRENTLY idx_users_role ON "User"("role") WHERE "isActive" = true;

-- Analyser une requete lente
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT p.*, c.name as category_name
FROM "Product" p
JOIN "Category" c ON c.id = p."categoryId"
WHERE p.status = 'ACTIVE'
  AND (p.name ILIKE '%chemise%' OR p.description ILIKE '%chemise%')
ORDER BY p."createdAt" DESC
LIMIT 20 OFFSET 0;
```

### Pipeline d'optimisation des images

```typescript
// apps/api/src/modules/upload/image-optimizer.service.ts
import sharp from 'sharp';
import { join } from 'path';

export class ImageOptimizerService {
  private readonly sizes = {
    thumbnail: { width: 150, height: 150 },
    card: { width: 400, height: 400 },
    detail: { width: 800, height: 800 },
    original: { width: 1200, height: 1200 },
  };

  async processProductImage(inputPath: string, filename: string): Promise<Record<string, string>> {
    const results: Record<string, string> = {};
    const baseName = filename.replace(/\.[^/.]+$/, '');

    for (const [size, dimensions] of Object.entries(this.sizes)) {
      const outputFilename = `${baseName}-${size}.webp`;
      const outputPath = join(process.env.UPLOAD_DIR!, 'products', outputFilename);

      await sharp(inputPath)
        .resize(dimensions.width, dimensions.height, {
          fit: 'cover',
          withoutEnlargement: true,
        })
        .webp({ quality: 80 })
        .toFile(outputPath);

      results[size] = `/uploads/products/${outputFilename}`;
    }

    return results;
  }
}
```

### Optimisation du bundle Next.js

```typescript
// apps/web/next.config.ts
import type { NextConfig } from 'next';

const config: NextConfig = {
  // Optimisation du build
  output: 'standalone',
  compress: true,

  // Optimisation des images
  images: {
    formats: ['image/webp', 'image/avif'],
    deviceSizes: [640, 750, 828, 1080, 1200],
    imageSizes: [16, 32, 48, 64, 96, 128, 256],
    minimumCacheTTL: 60 * 60 * 24 * 30, // 30 jours
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'votredomaine.com',
        pathname: '/uploads/**',
      },
    ],
  },

  // Optimisation du bundle
  experimental: {
    optimizePackageImports: ['lucide-react', '@radix-ui/react-icons'],
  },

  // Headers de cache
  async headers() {
    return [
      {
        source: '/:all*(svg|jpg|png|webp|avif)',
        headers: [
          { key: 'Cache-Control', value: 'public, max-age=2592000, immutable' },
        ],
      },
      {
        source: '/_next/static/:path*',
        headers: [
          { key: 'Cache-Control', value: 'public, max-age=31536000, immutable' },
        ],
      },
    ];
  },
};

export default config;
```

---

## 9. CI/CD Pipeline

### GitHub Actions - Workflow complet

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD E-Commerce

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  POSTGRES_USER: test_user
  POSTGRES_PASSWORD: test_password
  POSTGRES_DB: ecommerce_test

jobs:
  # ==========================================
  # JOB 1 : Lint + Type Check
  # ==========================================
  lint:
    name: Lint & Type Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci
      - run: npm run lint
      - run: npm run type-check

  # ==========================================
  # JOB 2 : Tests Backend
  # ==========================================
  test-api:
    name: Tests Backend
    runs-on: ubuntu-latest
    needs: lint

    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: ${{ env.POSTGRES_USER }}
          POSTGRES_PASSWORD: ${{ env.POSTGRES_PASSWORD }}
          POSTGRES_DB: ${{ env.POSTGRES_DB }}
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci

      - name: Setup database
        run: |
          cd apps/api
          npx prisma migrate deploy
          npx prisma generate
        env:
          DATABASE_URL: postgresql://${{ env.POSTGRES_USER }}:${{ env.POSTGRES_PASSWORD }}@localhost:5432/${{ env.POSTGRES_DB }}

      - name: Run tests
        run: npm run test --filter=api -- --coverage
        env:
          DATABASE_URL: postgresql://${{ env.POSTGRES_USER }}:${{ env.POSTGRES_PASSWORD }}@localhost:5432/${{ env.POSTGRES_DB }}
          JWT_SECRET: test-jwt-secret-for-ci
          JWT_REFRESH_SECRET: test-refresh-secret-for-ci
          REDIS_HOST: localhost

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: apps/api/coverage/lcov.info
          flags: backend

  # ==========================================
  # JOB 3 : Tests Frontend
  # ==========================================
  test-web:
    name: Tests Frontend
    runs-on: ubuntu-latest
    needs: lint

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci
      - run: npm run test --filter=web -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: apps/web/coverage/lcov.info
          flags: frontend

  # ==========================================
  # JOB 4 : Build
  # ==========================================
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [test-api, test-web]

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci
      - run: npm run build

      - name: Check bundle size
        run: |
          WEB_SIZE=$(du -sk apps/web/.next/static | cut -f1)
          echo "Bundle size: ${WEB_SIZE} Ko"
          if [ "$WEB_SIZE" -gt 500 ]; then
            echo "WARNING: Bundle size > 500 Ko"
          fi

  # ==========================================
  # JOB 5 : E2E Tests (sur PR uniquement)
  # ==========================================
  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name == 'pull_request'

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci
      - run: npx playwright install --with-deps

      - name: Run E2E tests
        run: npx playwright test
        env:
          BASE_URL: http://localhost:3000

      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  # ==========================================
  # JOB 6 : Deploy to Staging
  # ==========================================
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment: staging

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd /var/www/ecommerce-staging
            /usr/local/bin/deploy-zero-downtime.sh

  # ==========================================
  # JOB 7 : Deploy to Production
  # ==========================================
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment: production

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /var/www/ecommerce
            /usr/local/bin/deploy-zero-downtime.sh

      - name: Verify deployment
        run: |
          sleep 10
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://${{ secrets.PROD_DOMAIN }}/api/health)
          if [ "$STATUS" != "200" ]; then
            echo "Deployment verification failed! Status: $STATUS"
            exit 1
          fi
          echo "Deployment verified successfully"

      - name: Notify Slack
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: "Deploiement production E-Commerce : ${{ job.status }}"
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Strategie de deploiement Blue/Green

```
┌─────────────────────────────────────────────────────┐
│           Deploiement Blue/Green                     │
│                                                      │
│  Etat initial :                                      │
│  Load Balancer ──► [BLUE] (v1.2 - actif)            │
│                    [GREEN] (v1.2 - standby)          │
│                                                      │
│  Deploiement v1.3 :                                  │
│  1. Deployer v1.3 sur GREEN                          │
│  2. Tester GREEN (health check + smoke tests)        │
│  3. Basculer le LB : GREEN devient actif             │
│  Load Balancer ──► [GREEN] (v1.3 - actif)           │
│                    [BLUE] (v1.2 - rollback ready)    │
│                                                      │
│  En cas de probleme :                                │
│  Rebasculer le LB sur BLUE (< 1 minute)             │
└─────────────────────────────────────────────────────┘
```

### Rollback Strategy

```bash
#!/bin/bash
# /usr/local/bin/rollback-ecommerce.sh
set -euo pipefail

PROJECT_DIR="/var/www/ecommerce"
cd $PROJECT_DIR

echo "=== Rollback E-Commerce ==="

# Lister les tags de deploiement disponibles
echo "Tags disponibles :"
git tag -l "deploy-*" --sort=-creatordate | head -5

# Si un tag est specifie
if [ -n "${1:-}" ]; then
    TARGET_TAG=$1
else
    # Prendre le tag precedent (avant le dernier deploiement)
    TARGET_TAG=$(git tag -l "deploy-*" --sort=-creatordate | sed -n '2p')
fi

if [ -z "$TARGET_TAG" ]; then
    echo "Aucun tag de rollback trouve"
    exit 1
fi

echo "Rollback vers : $TARGET_TAG"
read -p "Confirmer ? (oui/non) " CONFIRM
if [ "$CONFIRM" != "oui" ]; then
    echo "Rollback annule"
    exit 0
fi

git checkout "$TARGET_TAG"
npm ci --production
cd apps/api && npx prisma generate && cd ../..
npm run build
pm2 reload ecosystem.config.js

sleep 5
curl -s http://localhost:3001/api/health | python3 -m json.tool

echo "=== Rollback termine ==="
```

---

## 10. Scalability Checklist

### Quand scaler horizontalement ?

| Indicateur | Seuil | Action |
|---|---|---|
| CPU moyen > 70% pendant 15 min | Warning | Preparer un nouveau serveur |
| CPU moyen > 85% pendant 5 min | Critique | Ajouter un serveur immediatement |
| RAM > 85% de maniere soutenue | Warning | Augmenter RAM ou ajouter serveur |
| Temps de reponse p95 > 1s | Warning | Optimiser ou scaler |
| Connexions DB > 80% du max | Warning | Augmenter pool ou ajouter replica |
| Requetes/seconde > 500 (par serveur) | Monitoring | Evaluer le scaling |
| File d'attente BullMQ > 1000 jobs | Warning | Ajouter des workers |

### Sharding de base de donnees (quand y penser)

Le sharding est rarement necessaire pour un e-commerce, mais voici les signaux :

| Signal | Seuil | Solution avant sharding |
|---|---|---|
| Table > 100 millions de lignes | Evaluer | Partitionnement PostgreSQL |
| Taille DB > 500 Go | Evaluer | Archiver les anciennes donnees |
| Writes > 10 000/seconde | Evaluer | Queue + batch inserts |
| Requetes cross-table lentes | Evaluer | Denormalisation, vues materialisees |

### Criteres d'extraction en microservices

Ne pas extraire trop tot. Envisager seulement si :

| Critere | Candidats potentiels |
|---|---|
| Volume de trafic tres different | Recherche (Elasticsearch), Notifications |
| Equipe dedicee necessaire | Paiements, Analytics |
| Technologie differente requise | ML/IA (recommandations), Processing images |
| Decouplage fort souhaite | Emails, SMS, Notifications push |

### Optimisation des couts

| Poste | Optimisation | Economie estimee |
|---|---|---|
| Serveurs | Right-sizing (pas de sur-provisionnement) | 20-40% |
| Base de donnees | Instances reservees (1 an) | 30-50% |
| CDN | Cloudflare gratuit vs CloudFront | 100% du CDN |
| Stockage images | Backblaze B2 vs AWS S3 | 70-80% |
| Monitoring | Netdata (gratuit) vs Datadog | 100% |
| CI/CD | GitHub Actions (gratuit < 2000 min/mois) | Variable |
| SSL | Let's Encrypt (gratuit) | 100% |
| Backups | Compression + retention intelligente | 50% de stockage |

---

## Resume : Checklist de production HA

### Essentiel (avant la mise en production)

- [ ] PM2 cluster mode (min 2 instances API)
- [ ] Health check endpoints (/health, /health/ready, /health/live)
- [ ] Backup automatique quotidien (pg_dump + cron)
- [ ] Monitoring basique (script de verification + alertes email)
- [ ] Graceful shutdown configure
- [ ] Logging structure (JSON en production)
- [ ] Rate limiting (Nginx + NestJS Throttler)
- [ ] Security headers (HSTS, CSP, X-Frame-Options, etc.)
- [ ] Cloudflare (CDN + DDoS protection gratuite)
- [ ] Script de deploiement avec rollback

### Recommande (30 premiers jours)

- [ ] 2 serveurs d'application + Load Balancer
- [ ] PostgreSQL read replica
- [ ] Redis pour cache et sessions
- [ ] PgBouncer (connection pooling)
- [ ] Monitoring externe (UptimeRobot / BetterStack)
- [ ] Netdata ou Prometheus + Grafana
- [ ] Log aggregation (Loki / ELK)
- [ ] Backups off-site (S3 / Backblaze B2)
- [ ] Verification mensuelle des backups
- [ ] CI/CD pipeline (GitHub Actions)
- [ ] Tests de charge (k6)
- [ ] Performance budgets definis

### Avance (fort trafic)

- [ ] PostgreSQL cluster Patroni (failover automatique)
- [ ] Redis Sentinel ou Cluster
- [ ] Blue/Green deployment
- [ ] Auto-scaling (si cloud)
- [ ] WAF avance (Cloudflare Pro)
- [ ] HashiCorp Vault pour les secrets
- [ ] Pipeline d'optimisation des images (sharp + WebP)
- [ ] Next.js ISR pour les pages produits
- [ ] BullMQ pour les taches asynchrones
- [ ] Circuit Breaker sur les services externes
