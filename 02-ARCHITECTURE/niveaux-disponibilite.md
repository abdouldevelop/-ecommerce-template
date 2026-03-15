# Guide des Niveaux de Disponibilite — E-Commerce

## Instructions pour l'agent IA

Apres le questionnaire client (section 8), utilise ce guide pour choisir le bon niveau d'infrastructure. **Ne jamais deployer un e-commerce avec paiement en dessous du Niveau 2.**

---

## Tableau comparatif rapide

| | Niveau 1 | Niveau 2 | Niveau 3 | Niveau 4 |
|---|---|---|---|---|
| **Nom** | Essentiel | Professionnel | Haute disponibilite | Mission critique |
| **Serveurs** | 1 VPS | 1 VPS optimise | 2+ serveurs | Cloud auto-scale |
| **Uptime** | ~99% | ~99.5% | ~99.9% | ~99.99% |
| **Downtime/an** | ~3.6 jours | ~1.8 jours | ~8 heures | ~52 minutes |
| **Visiteurs/jour** | < 500 | 500 - 5 000 | 5 000 - 50 000 | 50 000+ |
| **Budget infra/mois** | 5-15K FCFA | 15-40K FCFA | 80-200K FCFA | 200K+ FCFA |
| **Paiement en ligne** | Non recommande | Oui | Oui | Oui |
| **Cas d'usage** | Portfolio, vitrine | PME, boutique | E-commerce serieux | Marketplace, SaaS |

---

## Niveau 1 — Essentiel (1 VPS basique)

### Quand choisir ce niveau
- Site vitrine avec formulaire de contact
- Boutique sans paiement en ligne (commande par WhatsApp)
- Prototype / MVP pour tester le marche
- Budget tres limite

### Architecture
```
[Client] → [Apache/Nginx + SSL] → [Node.js (1 instance)] → [PostgreSQL]
                                         |
                                    [1 seul VPS]
```

### Ce qui est inclus
- 1 VPS (2 Go RAM, 2 vCPU minimum)
- Node.js en mode fork (1 processus)
- PostgreSQL sur le meme serveur
- Apache/Nginx reverse proxy + SSL Let's Encrypt
- Backup manuel ou basique

### Ce qui manque
- Pas de redondance (si le serveur tombe, le site tombe)
- Pas de monitoring
- Pas de backup automatique
- Pas de rate limiting
- Pas de swap

### Risques acceptes
- Downtime en cas de maintenance
- Perte de donnees possible (pas de backup auto)
- Vulnerable aux pics de trafic

---

## Niveau 2 — Professionnel (1 VPS optimise)

### Quand choisir ce niveau
- E-commerce avec paiement en ligne
- PME avec clientele reguliere
- Site qui doit etre fiable mais budget limite
- **C'est le minimum pour du paiement en ligne**

### Architecture
```
[Client] → [Apache + SSL + HSTS + CSP] → [PM2 Cluster (2+ instances)]
                                                    |
                                          [PostgreSQL (tune)]
                                                    |
                                          [Redis (sessions/cache)]
                                                    |
                                    [Backups auto + WAL archiving]
                                                    |
                                         [1 VPS optimise]
```

### Ce qui est inclus

**Application :**
- PM2 cluster mode (minimum 2 instances)
- Backoff restart exponentiel
- Limite memoire par processus
- Health check endpoints
- Graceful shutdown

**Base de donnees :**
- PostgreSQL tune (shared_buffers, work_mem, effective_cache_size)
- WAL archiving active (point-in-time recovery)
- max_connections adapte au nombre d'apps

**Securite :**
- Helmet (headers securite API)
- Rate limiting (@nestjs/throttler)
- JWT secrets forts (256-bit aleatoires)
- CORS configure
- Swagger desactive en production
- Validation des entrees (DTOs)
- Firewall (UFW) configure
- Ports sensibles fermes (DB, Redis, MinIO)

**Backups :**
- Backup quotidien automatique (cron)
- Rotation 7 jours
- Verification des dumps (pg_restore --list)
- Fichiers en permission 600

**Monitoring :**
- PM2+ ou script de monitoring custom
- Auto-restart des processus crashes
- Swap configure (4-8 Go)

**Infrastructure :**
- Swap 4+ Go (anti OOM-killer)
- Logrotate configure
- SSL auto-renouvele (certbot)
- Plan de reprise documente (DISASTER_RECOVERY_PLAN.md)

### Taches d'implementation (pour l'agent)
```bash
# 1. Swap
fallocate -l 4G /swapfile && chmod 600 /swapfile && mkswap /swapfile && swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab

# 2. PM2 cluster
pm2 start app.js -i 2 --max-memory-restart 300M --exp-backoff-restart-delay 100

# 3. PostgreSQL tuning (adapter selon RAM)
# Pour 4 Go RAM :
shared_buffers = 1GB
effective_cache_size = 3GB
work_mem = 16MB

# Pour 8 Go RAM :
shared_buffers = 2GB
effective_cache_size = 6GB
work_mem = 32MB

# Pour 16+ Go RAM :
shared_buffers = 4GB
effective_cache_size = 12GB
work_mem = 32MB
maintenance_work_mem = 512MB

# 4. WAL archiving
archive_mode = on
archive_command = 'cp %p /var/backups/postgresql/wal/%f'

# 5. Backup auto
# Voir le script pg-backup-all.sh dans 03-EQUIPES/equipe-infra.md

# 6. Firewall
ufw default deny incoming
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable

# 7. Helmet + throttler
npm install helmet @nestjs/throttler
```

### Tests de charge attendus
| Metrique | Cible |
|----------|-------|
| Utilisateurs simultanes | 200+ sans degradation |
| Requetes/seconde | 300+ |
| Temps de reponse moyen | < 500ms |
| Temps de reponse P95 | < 1s |
| Zero erreur | Jusqu'a 500 simultanes |

---

## Niveau 3 — Haute disponibilite (multi-serveurs)

### Quand choisir ce niveau
- E-commerce avec volume de ventes important
- Le site ne peut pas se permettre d'etre down
- Plusieurs milliers de visiteurs par jour
- Exigences SLA client

### Architecture
```
                        [CDN - Cloudflare]
                              |
                     [Load Balancer (HAProxy/Nginx)]
                        /              \
              [App Server 1]      [App Server 2]
              [PM2 Cluster]       [PM2 Cluster]
                   |                    |
                   +--------+-----------+
                            |
                    [PgBouncer (pool)]
                        /         \
              [PG Primary]    [PG Replica]
                   |                |
              [Redis Sentinel - 3 noeuds]
                            |
                    [S3 / MinIO Cluster]
                            |
                [Monitoring: Prometheus + Grafana]
                            |
                  [Alertes: Slack / Email / SMS]
```

### Ce qui est inclus (en plus du Niveau 2)

**Load balancing :**
- HAProxy ou Nginx upstream
- Health checks automatiques
- Sticky sessions si necessaire
- SSL termination au load balancer

**Base de donnees :**
- PostgreSQL streaming replication (1 primary + 1 replica)
- PgBouncer pour le connection pooling
- Failover automatique (Patroni ou repmgr)
- Read replica pour les requetes SELECT lourdes (stats, export)

**Cache :**
- Redis Sentinel (3 noeuds minimum)
- Cache des sessions, du panier, du rate limiting
- Cache des reponses API (produits, categories)
- CDN pour les assets statiques et images

**Monitoring avance :**
- Prometheus + Grafana (dashboards)
- Alertes automatiques (CPU > 80%, RAM > 90%, disk > 85%, error rate > 1%)
- Uptime monitoring externe (UptimeRobot, BetterStack)
- Log aggregation (Loki ou ELK)

**Backups :**
- Backup quotidien + WAL archiving
- Backup hors-site (S3, Backblaze B2)
- Test de restauration mensuel
- Retention 30 jours

**CI/CD :**
- Pipeline GitHub Actions
- Tests automatises sur chaque PR
- Deploiement blue/green (zero downtime)
- Rollback automatique si health check echoue

### Tests de charge attendus
| Metrique | Cible |
|----------|-------|
| Utilisateurs simultanes | 2 000+ |
| Requetes/seconde | 1 000+ |
| Temps de reponse moyen | < 200ms |
| Temps de reponse P95 | < 500ms |
| Disponibilite | 99.9% |

---

## Niveau 4 — Mission critique (cloud manage)

### Quand choisir ce niveau
- Marketplace avec des milliers de vendeurs
- SaaS e-commerce multi-tenant
- Conformite reglementaire stricte (PCI-DSS, RGPD)
- Budget infrastructure consequent

### Architecture
```
[WAF + DDoS Protection - Cloudflare Pro]
              |
    [Cloud Load Balancer (ALB/NLB)]
              |
    [Kubernetes Cluster (EKS/GKE)]
       /      |       \
  [Pod 1]  [Pod 2]  [Pod N]  ← Auto-scaling
              |
    [Managed PostgreSQL (RDS/Cloud SQL)]
       Primary + Multi-AZ Replica
              |
    [ElastiCache Redis - Multi-AZ]
              |
    [S3 + CloudFront CDN]
              |
    [CloudWatch / Datadog / New Relic]
              |
    [PagerDuty / OpsGenie - Astreinte 24/7]
```

### Ce qui est inclus (en plus du Niveau 3)
- Kubernetes avec auto-scaling horizontal
- Base de donnees managee multi-AZ
- WAF (Web Application Firewall)
- DDoS protection
- Secrets management (AWS Secrets Manager, HashiCorp Vault)
- Multi-region si necessaire
- Audit logs complets
- Conformite PCI-DSS
- Astreinte 24/7

### Tests de charge attendus
| Metrique | Cible |
|----------|-------|
| Utilisateurs simultanes | 10 000+ |
| Requetes/seconde | 5 000+ |
| Temps de reponse moyen | < 100ms |
| Disponibilite | 99.99% |

---

## Decision pour l'agent IA

### Algorithme de decision

```
SI budget < 200K FCFA ET pas de paiement en ligne:
  → Niveau 1

SI budget 200K-1M FCFA OU paiement en ligne:
  → Niveau 2 (MINIMUM pour tout e-commerce avec paiement)

SI budget > 1M FCFA ET visiteurs > 5000/jour ET SLA exige:
  → Niveau 3

SI marketplace OU multi-tenant OU conformite reglementaire:
  → Niveau 4
```

### Regles strictes
1. **Paiement en ligne = Niveau 2 minimum** (pas de negociation)
2. **Toujours configurer les backups** meme au Niveau 1
3. **Toujours mettre un swap** meme au Niveau 1
4. **Documenter le plan de reprise** a tous les niveaux
5. **Tester la charge** avant la mise en production

### Ce qu'il faut dire au client

> "Votre boutique traite des paiements en ligne. Pour la securite de vos clients et de votre activite, nous deploierons au minimum un serveur optimise avec backups automatiques, monitoring et securite renforcee (Niveau 2). Si vous prevoyez plus de 5 000 visiteurs par jour, nous recommandons une architecture multi-serveurs (Niveau 3)."
