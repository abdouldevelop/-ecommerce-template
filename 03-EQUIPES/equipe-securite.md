# Equipe Sécurité - Audit et Checklist (69 points)

## Instructions pour l'agent IA

Ce document est une checklist de sécurité complète basée sur un audit réel d'une plateforme e-commerce. Chaque point a un niveau de sévérité et une remédiation. A passer **avant la mise en production** et à re-vérifier régulièrement.

**Niveaux de sévérité** :
- CRITIQUE : Exploitation immédiate possible, données en danger
- ELEVE : Vulnérabilité sérieuse, à corriger avant la mise en production
- MOYEN : Risque modéré, à planifier dans les 30 jours
- FAIBLE : Bonne pratique, à faire quand possible

---

## Section 1 : Authentification (12 points)

### SEC-001 : Hachage des mots de passe
- **Sévérité** : CRITIQUE
- **Vérification** : Les mots de passe sont hachés avec bcrypt (salt rounds >= 10)
- **Remédiation** : Ne jamais stocker de mots de passe en clair ou avec MD5/SHA1. Utiliser `bcrypt.hash(password, 10)`
- **Statut** : [ ]

### SEC-002 : Politique de mots de passe
- **Sévérité** : ELEVE
- **Vérification** : Minimum 8 caractères, 1 majuscule, 1 chiffre minimum
- **Remédiation** : Valider côté serveur avec class-validator ou regex
- **Statut** : [ ]

### SEC-003 : Expiration des tokens JWT
- **Sévérité** : ELEVE
- **Vérification** : Access token expire en 30 minutes, refresh token en 7 jours
- **Remédiation** : Ne jamais utiliser de tokens sans expiration
- **Statut** : [ ]

### SEC-004 : Rotation des refresh tokens
- **Sévérité** : ELEVE
- **Vérification** : Chaque utilisation du refresh token en génère un nouveau et invalide l'ancien
- **Remédiation** : Stocker le hash du refresh token en base, le mettre à jour à chaque refresh
- **Statut** : [ ]

### SEC-005 : Invalidation des tokens à la déconnexion
- **Sévérité** : MOYEN
- **Vérification** : Le logout invalide le refresh token en base
- **Remédiation** : Mettre refreshToken à null en base lors du logout
- **Statut** : [ ]

### SEC-006 : Invalidation après changement de mot de passe
- **Sévérité** : ELEVE
- **Vérification** : Tous les tokens sont invalidés quand le mot de passe change
- **Remédiation** : Supprimer tous les refresh tokens de l'utilisateur
- **Statut** : [ ]

### SEC-007 : Protection contre le brute force
- **Sévérité** : ELEVE
- **Vérification** : Rate limiting sur /api/auth/login (5 tentatives/minute max)
- **Remédiation** : Utiliser `@nestjs/throttler` ou rate limiting Nginx
- **Statut** : [ ]

### SEC-008 : Messages d'erreur génériques
- **Sévérité** : MOYEN
- **Vérification** : Le message d'erreur login ne révèle pas si c'est l'email ou le mot de passe qui est faux
- **Remédiation** : Utiliser "Identifiants incorrects" au lieu de "Email non trouvé" ou "Mot de passe incorrect"
- **Statut** : [ ]

### SEC-009 : Protection du header Authorization
- **Sévérité** : MOYEN
- **Vérification** : Le token JWT n'est pas logué dans les logs serveur
- **Remédiation** : Exclure le header Authorization des logs
- **Statut** : [ ]

### SEC-010 : Secret JWT fort
- **Sévérité** : CRITIQUE
- **Vérification** : Le secret JWT fait au moins 256 bits (32 caractères) et est aléatoire
- **Remédiation** : Générer avec `openssl rand -hex 32`. Ne jamais utiliser "secret" ou "jwt_secret"
- **Statut** : [ ]

### SEC-011 : Secrets différents pour access et refresh tokens
- **Sévérité** : ELEVE
- **Vérification** : JWT_SECRET et JWT_REFRESH_SECRET sont différents
- **Remédiation** : Utiliser deux secrets distincts
- **Statut** : [ ]

### SEC-012 : Protection des comptes désactivés
- **Sévérité** : ELEVE
- **Vérification** : Un compte avec isActive=false ne peut pas se connecter
- **Remédiation** : Vérifier isActive lors du login ET lors de la validation du token
- **Statut** : [ ]

---

## Section 2 : Validation des entrées (10 points)

### SEC-013 : Validation côté serveur
- **Sévérité** : CRITIQUE
- **Vérification** : Toutes les entrées utilisateur sont validées côté serveur (pas seulement côté client)
- **Remédiation** : Utiliser class-validator avec des DTOs NestJS pour chaque endpoint
- **Statut** : [ ]

### SEC-014 : Protection contre l'injection SQL
- **Sévérité** : CRITIQUE
- **Vérification** : Prisma utilise des requêtes paramétrées (pas de SQL brut avec concaténation)
- **Remédiation** : Ne jamais utiliser `$executeRawUnsafe` avec des entrées utilisateur. Prisma protège par défaut.
- **Statut** : [ ]

### SEC-015 : Protection contre le XSS
- **Sévérité** : ELEVE
- **Vérification** : Les entrées utilisateur sont échappées avant l'affichage
- **Remédiation** : React échappe par défaut. Attention aux `dangerouslySetInnerHTML` (descriptions produits HTML). Sanitiser avec DOMPurify.
- **Statut** : [ ]

### SEC-016 : Protection contre le CSRF
- **Sévérité** : MOYEN
- **Vérification** : Les tokens JWT sont envoyés dans le header Authorization (pas dans les cookies)
- **Remédiation** : Les API REST avec tokens Bearer ne sont pas vulnérables au CSRF. Si cookies utilisés, ajouter un token CSRF.
- **Statut** : [ ]

### SEC-017 : Validation des types et tailles
- **Sévérité** : ELEVE
- **Vérification** : Chaque champ a un type, une taille min/max et un format validés
- **Remédiation** : Exemples : email (format), nom (2-100 chars), prix (positif), quantité (1-999), description (max 5000 chars)
- **Statut** : [ ]

### SEC-018 : Validation des fichiers uploadés
- **Sévérité** : CRITIQUE
- **Vérification** : Les fichiers uploadés sont vérifiés (type MIME, extension, taille)
- **Remédiation** : Multer avec fileFilter (JPG, PNG, WebP uniquement), limits (5 Mo max). Vérifier le magic number du fichier, pas seulement l'extension.
- **Statut** : [ ]

### SEC-019 : Nom de fichier sécurisé
- **Sévérité** : ELEVE
- **Vérification** : Les noms de fichiers uploadés sont sanitisés (pas de path traversal)
- **Remédiation** : Renommer les fichiers avec un timestamp + uuid. Ne jamais utiliser le nom original directement.
- **Statut** : [ ]

### SEC-020 : Validation des slugs et identifiants
- **Sévérité** : MOYEN
- **Vérification** : Les slugs et UUIDs sont validés avant d'être utilisés en requête
- **Remédiation** : Valider le format UUID avec regex ou class-validator `@IsUUID()`
- **Statut** : [ ]

### SEC-021 : Limite de pagination
- **Sévérité** : MOYEN
- **Vérification** : Le paramètre `limit` est plafonné (max 100)
- **Remédiation** : `Math.min(parseInt(limit) || 20, 100)`
- **Statut** : [ ]

### SEC-022 : Sanitisation des requêtes de recherche
- **Sévérité** : MOYEN
- **Vérification** : Les termes de recherche sont sanitisés (pas de caractères spéciaux SQL/regex)
- **Remédiation** : Prisma échappe automatiquement. Pour les recherches LIKE, échapper les caractères spéciaux.
- **Statut** : [ ]

---

## Section 3 : Sécurité de l'API (12 points)

### SEC-023 : CORS configuré
- **Sévérité** : ELEVE
- **Vérification** : CORS autorise uniquement l'origine du frontend (pas `*` en production)
- **Remédiation** : `app.enableCors({ origin: 'https://votredomaine.com' })`
- **Statut** : [ ]

### SEC-024 : Protection des routes admin
- **Sévérité** : CRITIQUE
- **Vérification** : Toutes les routes admin vérifient le rôle ADMIN
- **Remédiation** : Utiliser `@Roles('ADMIN')` + `RolesGuard` sur chaque route admin
- **Statut** : [ ]

### SEC-025 : Vérification de propriété
- **Sévérité** : CRITIQUE
- **Vérification** : Un utilisateur ne peut accéder qu'à SES données (commandes, panier, adresses)
- **Remédiation** : Vérifier `where: { userId: currentUser.id }` sur chaque requête
- **Statut** : [ ]

### SEC-026 : Pas de données sensibles dans les réponses
- **Sévérité** : ELEVE
- **Vérification** : Le champ `password` n'est jamais retourné dans les réponses API
- **Remédiation** : Exclure systématiquement : password, refreshToken, providerData (paiement)
- **Statut** : [ ]

### SEC-027 : Rate limiting global
- **Sévérité** : ELEVE
- **Vérification** : Rate limiting en place (100 requêtes/minute par IP)
- **Remédiation** : `@nestjs/throttler` ou Nginx `limit_req_zone`
- **Statut** : [ ]

### SEC-028 : Pas d'informations techniques dans les erreurs
- **Sévérité** : MOYEN
- **Vérification** : Les stack traces ne sont pas exposées en production
- **Remédiation** : Exception filter qui retourne des messages génériques en production et des détails en développement uniquement
- **Statut** : [ ]

### SEC-029 : Validation des transitions de statut
- **Sévérité** : ELEVE
- **Vérification** : Les changements de statut de commande suivent un flow valide (pas de retour en arrière arbitraire)
- **Remédiation** : Matrice de transitions autorisées vérifiée côté serveur
- **Statut** : [ ]

### SEC-030 : Protection des endpoints de callback paiement
- **Sévérité** : CRITIQUE
- **Vérification** : Le callback paiement vérifie l'authenticité de la notification
- **Remédiation** : Vérifier la signature/hash du prestataire, vérifier l'IP source si possible
- **Statut** : [ ]

### SEC-031 : Vérification du montant au callback
- **Sévérité** : CRITIQUE
- **Vérification** : Le montant payé est vérifié contre le montant attendu en base
- **Remédiation** : `if (callbackAmount !== order.total) { reject(); }`
- **Statut** : [ ]

### SEC-032 : Anti-replay pour les paiements
- **Sévérité** : CRITIQUE
- **Vérification** : Un callback de paiement ne peut pas être rejoué
- **Remédiation** : Vérifier que le paiement est encore en PENDING avant de le marquer comme COMPLETED
- **Statut** : [ ]

### SEC-033 : Transaction ID unique
- **Sévérité** : ELEVE
- **Vérification** : Chaque paiement a un transactionId unique et non prévisible
- **Remédiation** : Utiliser uuid v4 pour générer le transactionId
- **Statut** : [ ]

### SEC-034 : Logging des actions sensibles
- **Sévérité** : MOYEN
- **Vérification** : Les actions sensibles sont loguées (login, changement de mdp, paiement, changement de statut)
- **Remédiation** : Logger avec userId, action, timestamp, IP
- **Statut** : [ ]

---

## Section 4 : Sécurité des données (8 points)

### SEC-035 : HTTPS obligatoire
- **Sévérité** : CRITIQUE
- **Vérification** : Tout le trafic passe par HTTPS (HTTP redirige vers HTTPS)
- **Remédiation** : Nginx : `return 301 https://$server_name$request_uri;`
- **Statut** : [ ]

### SEC-036 : Variables d'environnement sécurisées
- **Sévérité** : CRITIQUE
- **Vérification** : Les secrets (JWT, DB, API paiement) sont dans des variables d'environnement, pas dans le code
- **Remédiation** : Fichier `.env` avec permissions 600, `.env` dans `.gitignore`
- **Statut** : [ ]

### SEC-037 : Pas de secrets dans le code source
- **Sévérité** : CRITIQUE
- **Vérification** : Aucun mot de passe, clé API ou secret codé en dur dans le code
- **Remédiation** : Grep dans le code pour "password", "secret", "api_key". Tout doit venir de process.env.
- **Statut** : [ ]

### SEC-038 : .gitignore complet
- **Sévérité** : ELEVE
- **Vérification** : `.env`, `node_modules`, `uploads/`, `.next/`, `dist/` sont dans `.gitignore`
- **Remédiation** : Vérifier que `.env` n'est pas tracké avec `git log --all --full-history -- .env`
- **Statut** : [ ]

### SEC-039 : Données personnelles protégées
- **Sévérité** : ELEVE
- **Vérification** : Les emails, téléphones et adresses ne sont pas exposés publiquement
- **Remédiation** : Les endpoints publics (produits, catégories) ne retournent jamais de données personnelles
- **Statut** : [ ]

### SEC-040 : Backup de la base de données
- **Sévérité** : ELEVE
- **Vérification** : Backups automatiques quotidiens, rétention 30 jours, test de restauration effectué
- **Remédiation** : Script pg_dump + cron + test restore
- **Statut** : [ ]

### SEC-041 : Données de paiement
- **Sévérité** : CRITIQUE
- **Vérification** : Aucune donnée de carte bancaire n'est stockée sur le serveur
- **Remédiation** : Les paiements sont gérés par le prestataire (PaiementPro/CinetPay). Stocker uniquement la référence de transaction.
- **Statut** : [ ]

### SEC-042 : Suppression sécurisée
- **Sévérité** : MOYEN
- **Vérification** : Les suppressions de comptes suppriment toutes les données associées
- **Remédiation** : Cascade delete dans Prisma ou soft-delete avec anonymisation des données
- **Statut** : [ ]

---

## Section 5 : Infrastructure (12 points)

### SEC-043 : Pare-feu configuré
- **Sévérité** : CRITIQUE
- **Vérification** : UFW activé, seuls les ports 22, 80, 443 sont ouverts
- **Remédiation** : `ufw default deny incoming && ufw allow 22 && ufw allow 80 && ufw allow 443 && ufw enable`
- **Statut** : [ ]

### SEC-044 : SSH sécurisé
- **Sévérité** : CRITIQUE
- **Vérification** : Authentification par clé publique, root login désactivé, mot de passe désactivé
- **Remédiation** : Dans `/etc/ssh/sshd_config` : `PasswordAuthentication no`, `PermitRootLogin no`
- **Statut** : [ ]

### SEC-045 : Fail2ban actif
- **Sévérité** : ELEVE
- **Vérification** : Fail2ban protège SSH et Nginx
- **Remédiation** : `apt install fail2ban`, configurer les jails sshd et nginx-http-auth
- **Statut** : [ ]

### SEC-046 : Mises à jour automatiques de sécurité
- **Sévérité** : ELEVE
- **Vérification** : Les patchs de sécurité du système sont installés automatiquement
- **Remédiation** : `apt install unattended-upgrades && dpkg-reconfigure unattended-upgrades`
- **Statut** : [ ]

### SEC-047 : PostgreSQL non accessible de l'extérieur
- **Sévérité** : CRITIQUE
- **Vérification** : PostgreSQL écoute uniquement sur localhost (port 5432 non exposé)
- **Remédiation** : `listen_addresses = 'localhost'` dans postgresql.conf
- **Statut** : [ ]

### SEC-048 : Utilisateur PostgreSQL dédié
- **Sévérité** : ELEVE
- **Vérification** : L'application utilise un utilisateur dédié (pas postgres/root)
- **Remédiation** : `CREATE USER ecommerce_user WITH PASSWORD '...' ; GRANT ALL ON DATABASE ecommerce TO ecommerce_user;`
- **Statut** : [ ]

### SEC-049 : Process Node.js non-root
- **Sévérité** : ELEVE
- **Vérification** : PM2 ne lance pas les applications en tant que root
- **Remédiation** : Créer un utilisateur dédié `ecommerce` et configurer PM2 avec cet utilisateur
- **Statut** : [ ]

### SEC-050 : Permissions des fichiers
- **Sévérité** : MOYEN
- **Vérification** : Les fichiers de l'application ont les permissions minimales nécessaires
- **Remédiation** : Code : 644, répertoires : 755, .env : 600, uploads : 755
- **Statut** : [ ]

### SEC-051 : Stockage des uploads sécurisé
- **Sévérité** : ELEVE
- **Vérification** : Les fichiers uploadés ne peuvent pas être exécutés
- **Remédiation** : Nginx : `location /uploads/ { add_header X-Content-Type-Options nosniff; }`, pas de handler PHP/Node pour ce répertoire
- **Statut** : [ ]

### SEC-052 : Nginx : masquer la version
- **Sévérité** : FAIBLE
- **Vérification** : Le header `Server` ne révèle pas la version de Nginx
- **Remédiation** : `server_tokens off;` dans nginx.conf
- **Statut** : [ ]

### SEC-053 : Node.js : masquer le framework
- **Sévérité** : FAIBLE
- **Vérification** : Le header `X-Powered-By` n'est pas présent
- **Remédiation** : NestJS : `app.disable('x-powered-by')` ou via helmet
- **Statut** : [ ]

### SEC-054 : Certificat SSL valide et fort
- **Sévérité** : CRITIQUE
- **Vérification** : SSL Labs note A ou A+, TLS 1.2+ uniquement
- **Remédiation** : Certbot avec les paramètres par défaut donne un A. Désactiver TLS 1.0 et 1.1.
- **Statut** : [ ]

---

## Section 6 : Dépendances (8 points)

### SEC-055 : Audit des dépendances npm
- **Sévérité** : ELEVE
- **Vérification** : `npm audit` ne retourne aucune vulnérabilité critique ou élevée
- **Remédiation** : `npm audit fix` ou mise à jour manuelle des packages vulnérables
- **Statut** : [ ]

### SEC-056 : Dépendances à jour
- **Sévérité** : MOYEN
- **Vérification** : Les dépendances majeures sont dans des versions supportées
- **Remédiation** : `npx npm-check-updates` pour vérifier, mettre à jour progressivement
- **Statut** : [ ]

### SEC-057 : Node.js LTS
- **Sévérité** : ELEVE
- **Vérification** : Version Node.js LTS en production (20.x actuellement)
- **Remédiation** : Ne pas utiliser de versions impaires (19, 21) en production
- **Statut** : [ ]

### SEC-058 : Lock file commité
- **Sévérité** : MOYEN
- **Vérification** : `package-lock.json` est dans le dépôt Git
- **Remédiation** : Commiter le lock file pour des builds reproductibles
- **Statut** : [ ]

### SEC-059 : Pas de dépendances inutiles
- **Sévérité** : FAIBLE
- **Vérification** : Toutes les dépendances en `dependencies` sont effectivement utilisées
- **Remédiation** : `npx depcheck` pour identifier les dépendances inutilisées
- **Statut** : [ ]

### SEC-060 : Helmet installé
- **Sévérité** : MOYEN
- **Vérification** : Le middleware `helmet` est actif pour les headers de sécurité HTTP
- **Remédiation** : `npm install helmet`, `app.use(helmet())`
- **Statut** : [ ]

### SEC-061 : Compression activée
- **Sévérité** : FAIBLE
- **Vérification** : La compression gzip/brotli est gérée (Nginx ou middleware)
- **Remédiation** : Préférer la compression Nginx plutôt que Node.js
- **Statut** : [ ]

### SEC-062 : Pas de dépendances en pre/post scripts suspects
- **Sévérité** : MOYEN
- **Vérification** : Les scripts npm ne contiennent pas de commandes suspectes
- **Remédiation** : Vérifier `package.json` scripts avant chaque `npm install`
- **Statut** : [ ]

---

## Section 7 : Sécurité frontend (7 points)

### SEC-063 : Pas de secrets dans le code frontend
- **Sévérité** : CRITIQUE
- **Vérification** : Aucun secret (JWT, clé API privée) dans le code client ou `NEXT_PUBLIC_`
- **Remédiation** : Seules les variables NEXT_PUBLIC_ non sensibles (URL API, nom site) côté client
- **Statut** : [ ]

### SEC-064 : Validation côté client
- **Sévérité** : MOYEN
- **Vérification** : Les formulaires ont une validation côté client (UX) en plus du serveur
- **Remédiation** : Valider avant d'envoyer (email format, champs requis, etc.)
- **Statut** : [ ]

### SEC-065 : Tokens stockés de manière sécurisée
- **Sévérité** : ELEVE
- **Vérification** : Les tokens JWT sont dans le store Zustand (mémoire) et/ou localStorage
- **Remédiation** : Ne pas stocker dans les cookies sans flag HttpOnly/Secure/SameSite. localStorage est acceptable pour les SPA.
- **Statut** : [ ]

### SEC-066 : Protection des routes privées côté client
- **Sévérité** : MOYEN
- **Vérification** : Les pages admin et compte redirigent si non connecté
- **Remédiation** : Middleware Next.js ou composant `withAuth` qui vérifie l'état auth
- **Statut** : [ ]

### SEC-067 : Pas de console.log en production
- **Sévérité** : FAIBLE
- **Vérification** : Pas de données sensibles dans les console.log en production
- **Remédiation** : Supprimer ou conditionner les console.log à NODE_ENV === 'development'
- **Statut** : [ ]

### SEC-068 : Content Security Policy
- **Sévérité** : MOYEN
- **Vérification** : CSP configuré pour empêcher l'injection de scripts tiers
- **Remédiation** : Header CSP via Nginx ou Next.js `headers()` dans next.config.ts
- **Statut** : [ ]

### SEC-069 : Intégrité des sous-ressources (SRI)
- **Sévérité** : FAIBLE
- **Vérification** : Les scripts et styles externes ont un attribut `integrity`
- **Remédiation** : Ajouter l'attribut SRI pour les CDN externes (s'il y en a)
- **Statut** : [ ]

---

## Résumé par sévérité

| Sévérité | Nombre | Points |
|---|---|---|
| CRITIQUE | 16 | SEC-001, 010, 013, 014, 018, 024, 025, 030, 031, 032, 035, 036, 037, 041, 043, 044, 047, 054, 063 |
| ELEVE | 24 | SEC-002, 003, 004, 006, 007, 011, 012, 015, 017, 019, 023, 026, 027, 029, 033, 038, 039, 040, 045, 046, 048, 049, 051, 055, 057, 065 |
| MOYEN | 19 | SEC-005, 008, 009, 016, 020, 021, 022, 028, 034, 042, 050, 056, 058, 060, 062, 064, 066, 068 |
| FAIBLE | 6 | SEC-052, 053, 059, 061, 067, 069 |

---

## Score d'audit

Calculer le score en cochant chaque point :
- CRITIQUE : 5 points par item
- ELEVE : 3 points par item
- MOYEN : 2 points par item
- FAIBLE : 1 point par item

**Score maximum** : 16x5 + 24x3 + 19x2 + 6x1 = 80 + 72 + 38 + 6 = **196 points**

| Score | Evaluation |
|---|---|
| 180-196 | Excellent - Production ready |
| 150-179 | Bon - Quelques améliorations nécessaires |
| 120-149 | Acceptable - Des corrections à planifier |
| 90-119 | Insuffisant - Corrections urgentes requises |
| < 90 | Critique - Ne pas mettre en production |
