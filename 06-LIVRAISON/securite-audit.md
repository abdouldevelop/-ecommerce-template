# Audit de Sécurité - E-Commerce

## Instructions pour l'agent IA

Ce document est le template d'audit de sécurité complet à effectuer avant la mise en production et à re-effectuer périodiquement (recommandé : tous les 3 mois). Basé sur l'audit réel de la plateforme Atypick Groupe (69 points).

**Notation** :
- CRITIQUE (5 pts) : Exploitation immédiate possible
- ELEVE (3 pts) : Vulnérabilité sérieuse
- MOYEN (2 pts) : Risque modéré
- FAIBLE (1 pt) : Bonne pratique

**Score maximum** : 196 points
**Score minimum acceptable** : 150 points

---

## Informations de l'audit

| Paramètre | Valeur |
|---|---|
| Date de l'audit | [A REMPLIR] |
| Auditeur | [A REMPLIR] |
| Projet | [A REMPLIR] |
| URL | [A REMPLIR] |
| Version | [A REMPLIR] |
| Audit précédent | [A REMPLIR - Date ou "Premier audit"] |

---

## Section 1 : Authentification et sessions (12 points)

| ID | Point de contrôle | Sévérité | Statut | Remédiation |
|---|---|---|---|---|
| A01 | Mots de passe hachés avec bcrypt (>= 10 rounds) | CRITIQUE | [ ] | Vérifier le code du service auth : `bcrypt.hash(password, 10)` |
| A02 | Politique de mot de passe (min 8 chars, 1 maj, 1 chiffre) | ELEVE | [ ] | Vérifier le DTO de validation |
| A03 | Access token expire en 30 minutes max | ELEVE | [ ] | Vérifier JWT_EXPIRATION dans .env |
| A04 | Refresh token expire en 7 jours max | ELEVE | [ ] | Vérifier JWT_REFRESH_EXPIRATION dans .env |
| A05 | Rotation des refresh tokens (un nouveau à chaque refresh) | ELEVE | [ ] | Vérifier que l'ancien hash est remplacé en base |
| A06 | Invalidation du refresh token au logout | MOYEN | [ ] | Vérifier que refreshToken est mis à null en base |
| A07 | Invalidation de tous les tokens au changement de mot de passe | ELEVE | [ ] | Vérifier la suppression des tokens existants |
| A08 | Rate limiting sur /api/auth/login (max 5/min) | ELEVE | [ ] | Tester avec 6 requêtes rapides, vérifier le 429 |
| A09 | Messages d'erreur login génériques ("Identifiants incorrects") | MOYEN | [ ] | Tester avec email inexistant vs mauvais mot de passe |
| A10 | Secret JWT fort (>= 256 bits, aléatoire) | CRITIQUE | [ ] | Vérifier la longueur et l'aléatoire du JWT_SECRET |
| A11 | Secrets JWT différents pour access et refresh | ELEVE | [ ] | Comparer JWT_SECRET et JWT_REFRESH_SECRET |
| A12 | Comptes désactivés ne peuvent pas se connecter | ELEVE | [ ] | Tester avec isActive=false |

---

## Section 2 : Validation des entrées (10 points)

| ID | Point de contrôle | Sévérité | Statut | Remédiation |
|---|---|---|---|---|
| V01 | Toutes les entrées validées côté serveur (DTOs) | CRITIQUE | [ ] | Vérifier que chaque endpoint a un DTO avec class-validator |
| V02 | Pas d'injection SQL (requêtes paramétrées Prisma) | CRITIQUE | [ ] | Chercher `$executeRawUnsafe` dans le code |
| V03 | Protection XSS (échappement des sorties) | ELEVE | [ ] | Vérifier les usages de `dangerouslySetInnerHTML`, utiliser DOMPurify |
| V04 | Pas de vulnérabilité CSRF | MOYEN | [ ] | Vérifier que les tokens sont dans le header Authorization |
| V05 | Validation types et tailles sur chaque champ | ELEVE | [ ] | Vérifier les DTOs : @IsString, @MinLength, @MaxLength, @IsPositive |
| V06 | Fichiers uploadés vérifiés (type MIME + extension + taille) | CRITIQUE | [ ] | Vérifier la config Multer : fileFilter + limits |
| V07 | Noms de fichiers uploadés sanitisés | ELEVE | [ ] | Vérifier le renommage timestamp+uuid |
| V08 | UUIDs validés dans les paramètres | MOYEN | [ ] | Vérifier @IsUUID() dans les DTOs de paramètres |
| V09 | Pagination limitée (max 100) | MOYEN | [ ] | Vérifier Math.min(limit, 100) |
| V10 | Requêtes de recherche sanitisées | MOYEN | [ ] | Vérifier que Prisma échappe les caractères spéciaux |

---

## Section 3 : Sécurité de l'API (12 points)

| ID | Point de contrôle | Sévérité | Statut | Remédiation |
|---|---|---|---|---|
| P01 | CORS configuré (pas `origin: '*'` en production) | ELEVE | [ ] | `curl -H "Origin: https://evil.com" https://domaine.com/api/health` |
| P02 | Toutes les routes admin protégées par @Roles('ADMIN') | CRITIQUE | [ ] | Lister toutes les routes admin, tester avec un token CUSTOMER |
| P03 | Vérification de propriété des données | CRITIQUE | [ ] | Tester : un client essaie de voir la commande d'un autre |
| P04 | Champ password exclu de toutes les réponses | ELEVE | [ ] | Chercher les select/include qui pourraient exposer le password |
| P05 | Rate limiting global (100 req/min) | ELEVE | [ ] | Tester avec un script de flood |
| P06 | Pas de stack traces dans les erreurs en production | MOYEN | [ ] | Tester avec une requête invalide, vérifier la réponse |
| P07 | Transitions de statut commande validées | ELEVE | [ ] | Tester : passer DELIVERED → PENDING |
| P08 | Callback paiement vérifie l'authenticité | CRITIQUE | [ ] | Vérifier la signature/vérification dans le handler |
| P09 | Montant vérifié au callback (comparé avec la base) | CRITIQUE | [ ] | Vérifier la comparaison callbackAmount vs order.total |
| P10 | Anti-replay paiement (pas de double traitement) | CRITIQUE | [ ] | Envoyer le même callback deux fois, vérifier idempotence |
| P11 | Transaction ID unique et non prévisible | ELEVE | [ ] | Vérifier l'utilisation de uuid v4 |
| P12 | Actions sensibles loguées | MOYEN | [ ] | Vérifier les logs : login, changement mdp, paiement |

---

## Section 4 : Protection des données (8 points)

| ID | Point de contrôle | Sévérité | Statut | Remédiation |
|---|---|---|---|---|
| D01 | HTTPS obligatoire (HTTP redirige) | CRITIQUE | [ ] | `curl -I http://domaine.com` → doit retourner 301 vers HTTPS |
| D02 | Secrets dans .env (pas dans le code) | CRITIQUE | [ ] | `grep -r "password\|secret\|api_key" --include="*.ts" apps/` |
| D03 | .env dans .gitignore | ELEVE | [ ] | `git log --all --full-history -- .env` ne doit rien retourner |
| D04 | Données personnelles non exposées publiquement | ELEVE | [ ] | Vérifier que les endpoints publics ne retournent pas d'emails/téléphones |
| D05 | Backup quotidien de la base | ELEVE | [ ] | `crontab -l | grep backup` |
| D06 | Test de restauration de backup effectué | ELEVE | [ ] | Restaurer le dernier backup dans une base de test |
| D07 | Aucune donnée de carte bancaire stockée | CRITIQUE | [ ] | Vérifier qu'on stocke uniquement la référence de transaction |
| D08 | .env en permissions 600 | MOYEN | [ ] | `ls -la .env` → doit afficher `-rw-------` |

---

## Section 5 : Infrastructure serveur (12 points)

| ID | Point de contrôle | Sévérité | Statut | Remédiation |
|---|---|---|---|---|
| I01 | Pare-feu UFW activé (22, 80, 443 seulement) | CRITIQUE | [ ] | `ufw status` |
| I02 | SSH par clé uniquement, pas de root login | CRITIQUE | [ ] | Vérifier `/etc/ssh/sshd_config` |
| I03 | Fail2ban actif | ELEVE | [ ] | `systemctl status fail2ban` |
| I04 | Mises à jour auto de sécurité | ELEVE | [ ] | `dpkg -l unattended-upgrades` |
| I05 | PostgreSQL écoute localhost uniquement | CRITIQUE | [ ] | `grep listen_addresses /etc/postgresql/*/main/postgresql.conf` |
| I06 | Utilisateur PostgreSQL dédié (pas postgres) | ELEVE | [ ] | Vérifier DATABASE_URL dans .env |
| I07 | Processus Node.js non-root | ELEVE | [ ] | `ps aux | grep node` → vérifier le user |
| I08 | Permissions fichiers correctes (code 644, dirs 755, .env 600) | MOYEN | [ ] | `stat -c '%a %n' .env uploads/ apps/` |
| I09 | Uploads ne peuvent pas être exécutés | ELEVE | [ ] | Vérifier la config Nginx pour /uploads/ |
| I10 | Version Nginx masquée | FAIBLE | [ ] | `curl -I https://domaine.com` → pas de version Nginx |
| I11 | Header X-Powered-By supprimé | FAIBLE | [ ] | `curl -I https://domaine.com/api/health` → pas de X-Powered-By |
| I12 | Certificat SSL valide, TLS 1.2+ | CRITIQUE | [ ] | Tester sur ssllabs.com → note A minimum |

---

## Section 6 : Headers HTTP de sécurité (8 points)

| ID | Point de contrôle | Sévérité | Statut | Remédiation |
|---|---|---|---|---|
| H01 | X-Frame-Options: SAMEORIGIN | MOYEN | [ ] | Vérifier dans les headers de réponse |
| H02 | X-Content-Type-Options: nosniff | MOYEN | [ ] | Empêche le MIME sniffing |
| H03 | X-XSS-Protection: 1; mode=block | MOYEN | [ ] | Protection XSS navigateur |
| H04 | Referrer-Policy: strict-origin-when-cross-origin | MOYEN | [ ] | Contrôle l'envoi du referer |
| H05 | Strict-Transport-Security (HSTS) | ELEVE | [ ] | max-age=31536000; includeSubDomains |
| H06 | Content-Security-Policy configuré | MOYEN | [ ] | Adapter selon les besoins du site |
| H07 | Permissions-Policy configuré | FAIBLE | [ ] | Désactiver geolocation, microphone, camera si non utilisés |
| H08 | Score securityheaders.com >= B | MOYEN | [ ] | Tester sur securityheaders.com |

---

## Section 7 : Dépendances (4 points)

| ID | Point de contrôle | Sévérité | Statut | Remédiation |
|---|---|---|---|---|
| N01 | npm audit sans vulnérabilités critiques | ELEVE | [ ] | `npm audit` dans chaque app |
| N02 | Node.js version LTS (20.x) | ELEVE | [ ] | `node -v` |
| N03 | package-lock.json commité | MOYEN | [ ] | `git ls-files package-lock.json` |
| N04 | Helmet middleware actif | MOYEN | [ ] | Vérifier app.use(helmet()) |

---

## Section 8 : Frontend (3 points)

| ID | Point de contrôle | Sévérité | Statut | Remédiation |
|---|---|---|---|---|
| F01 | Pas de secrets dans le code frontend | CRITIQUE | [ ] | Vérifier NEXT_PUBLIC_* dans .env |
| F02 | Routes protégées côté client (redirect si non auth) | MOYEN | [ ] | Tester l'accès à /admin et /account en déconnecté |
| F03 | Pas de console.log sensibles en production | FAIBLE | [ ] | Chercher console.log dans le code frontend |

---

## Résultat de l'audit

### Calcul du score

| Sévérité | Points par item | Items conformes | Items non conformes | Score |
|---|---|---|---|---|
| CRITIQUE (5 pts) | 5 | __/16 | __/16 | __/80 |
| ELEVE (3 pts) | 3 | __/24 | __/24 | __/72 |
| MOYEN (2 pts) | 2 | __/19 | __/19 | __/38 |
| FAIBLE (1 pt) | 1 | __/6 | __/6 | __/6 |
| **Total** | | | | **__/196** |

### Evaluation

| Score | Evaluation | Action |
|---|---|---|
| 180-196 | Excellent | Production ready |
| 150-179 | Bon | Corriger les points ELEVE restants |
| 120-149 | Acceptable | Corriger les points CRITIQUE et ELEVE |
| 90-119 | Insuffisant | Corrections urgentes, ne pas mettre en production |
| < 90 | Critique | Audit complet et remédiation nécessaire |

### Points critiques non conformes

| ID | Description | Date de remédiation prévue | Responsable |
|---|---|---|---|
| | | | |

### Points élevés non conformes

| ID | Description | Date de remédiation prévue | Responsable |
|---|---|---|---|
| | | | |

---

## Recommandations

### Priorité 1 (à corriger immédiatement)

1. [A REMPLIR]
2. [A REMPLIR]

### Priorité 2 (à corriger dans les 30 jours)

1. [A REMPLIR]
2. [A REMPLIR]

### Priorité 3 (à planifier)

1. [A REMPLIR]
2. [A REMPLIR]

---

## Validation

| Rôle | Nom | Date | Signature |
|---|---|---|---|
| Auditeur | [A REMPLIR] | [A REMPLIR] | __________ |
| Responsable technique | [A REMPLIR] | [A REMPLIR] | __________ |

**Prochain audit prévu** : [A REMPLIR - dans 3 mois recommandé]
