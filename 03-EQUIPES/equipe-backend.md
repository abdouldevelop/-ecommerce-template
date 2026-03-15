# Equipe Backend - Tâches et Priorités

## Instructions pour l'agent IA

Ce document liste toutes les tâches backend avec leur priorité, complexité estimée et critères d'acceptation. Développer dans l'ordre des priorités (P0 d'abord, puis P1, etc.).

**Légende priorités** : P0 (bloquant/MVP), P1 (important), P2 (souhaitable), P3 (futur)
**Légende complexité** : S (< 2h), M (2-4h), L (4-8h), XL (> 8h)

---

## Module 1 : Setup projet (P0)

### BE-001 : Initialisation du projet NestJS
- **Priorité** : P0
- **Complexité** : M
- **Description** : Initialiser le projet NestJS dans le monorepo Turborepo
- **Critères d'acceptation** :
  - [ ] Projet NestJS créé dans `apps/api/`
  - [ ] TypeScript configuré (strict mode)
  - [ ] Prisma installé et configuré avec PostgreSQL
  - [ ] Variables d'environnement gérées avec `@nestjs/config`
  - [ ] Validation globale avec `class-validator` et `class-transformer`
  - [ ] CORS configuré
  - [ ] Exception filter global pour les erreurs uniformes
  - [ ] `npm run dev` lance le serveur sur le port 3001

### BE-002 : Schéma Prisma et migrations
- **Priorité** : P0
- **Complexité** : L
- **Description** : Créer le schéma complet de la base de données
- **Critères d'acceptation** :
  - [ ] Tous les modèles définis (User, Product, Category, Cart, Order, Payment, etc.)
  - [ ] Enums définis (Role, OrderStatus, PaymentStatus, etc.)
  - [ ] Index définis pour les requêtes fréquentes
  - [ ] Migration initiale exécutée avec succès
  - [ ] Seed script fonctionnel (admin + catégories + paramètres)
  - [ ] `npx prisma studio` affiche les tables

### BE-003 : Service Prisma
- **Priorité** : P0
- **Complexité** : S
- **Description** : Créer le service Prisma injectable dans NestJS
- **Critères d'acceptation** :
  - [ ] PrismaService créé avec `onModuleInit` et `onModuleDestroy`
  - [ ] PrismaModule exportable et importable dans tous les modules
  - [ ] Logging des requêtes en mode développement

---

## Module 2 : Authentification (P0)

### BE-004 : Module Auth - Register
- **Priorité** : P0
- **Complexité** : M
- **Description** : Endpoint d'inscription
- **Critères d'acceptation** :
  - [ ] POST /api/auth/register fonctionnel
  - [ ] Validation : email unique, password min 8 chars avec 1 majuscule et 1 chiffre
  - [ ] Password hashé avec bcrypt (salt rounds = 10)
  - [ ] Retourne accessToken + refreshToken + user (sans password)
  - [ ] Erreur 400 si email déjà utilisé

### BE-005 : Module Auth - Login
- **Priorité** : P0
- **Complexité** : M
- **Description** : Endpoint de connexion
- **Critères d'acceptation** :
  - [ ] POST /api/auth/login fonctionnel
  - [ ] Comparaison bcrypt du password
  - [ ] Retourne accessToken (30min) + refreshToken (7 jours)
  - [ ] Met à jour `lastLoginAt`
  - [ ] Erreur 401 si identifiants incorrects
  - [ ] Erreur 403 si compte désactivé (`isActive = false`)

### BE-006 : JWT Strategy et Guard
- **Priorité** : P0
- **Complexité** : M
- **Description** : Protection des routes avec JWT
- **Critères d'acceptation** :
  - [ ] JwtStrategy configurée avec `@nestjs/passport`
  - [ ] JwtAuthGuard créé et utilisable avec `@UseGuards(JwtAuthGuard)`
  - [ ] Décorateur `@CurrentUser()` pour extraire l'utilisateur du token
  - [ ] RolesGuard pour vérifier le rôle (ADMIN/CUSTOMER)
  - [ ] Décorateur `@Roles('ADMIN')` pour protéger les routes admin
  - [ ] Décorateur `@Public()` pour les routes publiques

### BE-007 : Token Refresh
- **Priorité** : P0
- **Complexité** : M
- **Description** : Renouvellement du token d'accès
- **Critères d'acceptation** :
  - [ ] POST /api/auth/refresh fonctionnel
  - [ ] Vérifie le refreshToken (signature + expiration)
  - [ ] Vérifie que le hash du refreshToken correspond en base
  - [ ] Retourne un nouveau couple accessToken + refreshToken
  - [ ] Invalide l'ancien refreshToken (rotation)

### BE-008 : Logout et Change Password
- **Priorité** : P0
- **Complexité** : S
- **Description** : Déconnexion et changement de mot de passe
- **Critères d'acceptation** :
  - [ ] POST /api/auth/logout invalide le refreshToken en base
  - [ ] POST /api/auth/change-password vérifie l'ancien mot de passe
  - [ ] Nouveau mot de passe validé (même règles que register)
  - [ ] Invalide tous les refreshTokens après changement de password

---

## Module 3 : Produits (P0)

### BE-009 : CRUD Produits
- **Priorité** : P0
- **Complexité** : L
- **Description** : Gestion complète des produits
- **Critères d'acceptation** :
  - [ ] GET /api/products : liste paginée, filtrable (catégorie, recherche, prix, featured)
  - [ ] GET /api/products/:slug : détail avec images et catégorie
  - [ ] GET /api/products/featured : produits vedettes (limit configurable)
  - [ ] POST /api/products (ADMIN) : création avec génération de slug automatique
  - [ ] PATCH /api/products/:id (ADMIN) : modification partielle
  - [ ] DELETE /api/products/:id (ADMIN) : suppression ou archivage
  - [ ] Slug unique : si doublon, ajouter un suffixe numérique
  - [ ] Tri par : prix, date, nom, popularité (viewCount)
  - [ ] Incrémenter viewCount sur GET /:slug

### BE-010 : Recherche produits (autocomplete)
- **Priorité** : P0
- **Complexité** : M
- **Description** : Recherche rapide pour l'autocomplete
- **Critères d'acceptation** :
  - [ ] GET /api/products/search?q=xxx : recherche par nom (ILIKE)
  - [ ] Retourne max 5-8 résultats avec : id, name, slug, price, primaryImage
  - [ ] Recherche à partir de 1 caractère
  - [ ] Temps de réponse < 200ms
  - [ ] Seulement les produits ACTIVE

### BE-011 : Gestion des images produits
- **Priorité** : P0
- **Complexité** : M
- **Description** : Upload et gestion des images
- **Critères d'acceptation** :
  - [ ] POST /api/upload/product-image : upload avec Multer
  - [ ] Max 5 Mo, formats : JPG, PNG, WebP
  - [ ] Nom de fichier : `{timestamp}-{originalname}`
  - [ ] Stockage dans `uploads/products/`
  - [ ] Association image-produit avec isPrimary et sortOrder
  - [ ] DELETE pour supprimer une image (fichier + base)

---

## Module 4 : Catégories (P0)

### BE-012 : CRUD Catégories
- **Priorité** : P0
- **Complexité** : M
- **Description** : Gestion des catégories
- **Critères d'acceptation** :
  - [ ] GET /api/categories : liste avec productCount
  - [ ] GET /api/categories/:slug : détail avec produits paginés
  - [ ] POST /api/categories (ADMIN) : création avec slug auto
  - [ ] PATCH /api/categories/:id (ADMIN) : modification
  - [ ] DELETE /api/categories/:id (ADMIN) : seulement si aucun produit associé
  - [ ] Support des sous-catégories (parentId)
  - [ ] Tri par sortOrder

---

## Module 5 : Panier (P0)

### BE-013 : Gestion du panier
- **Priorité** : P0
- **Complexité** : L
- **Description** : Panier côté serveur
- **Critères d'acceptation** :
  - [ ] GET /api/cart : panier avec items, subtotal, shippingFee, total
  - [ ] POST /api/cart/items : ajout (si déjà présent, incrémenter la quantité)
  - [ ] PATCH /api/cart/items/:id : modifier la quantité
  - [ ] DELETE /api/cart/items/:id : retirer un article
  - [ ] DELETE /api/cart : vider le panier
  - [ ] Validation du stock à chaque opération
  - [ ] Calcul automatique du subtotal et total
  - [ ] Frais de livraison calculés selon les paramètres

### BE-014 : Synchronisation panier
- **Priorité** : P1
- **Complexité** : M
- **Description** : Fusionner le panier local avec le panier serveur lors de la connexion
- **Critères d'acceptation** :
  - [ ] POST /api/cart/sync : reçoit les items du localStorage
  - [ ] Fusion : si produit déjà en base, garder la plus grande quantité
  - [ ] Validation du stock pour chaque item
  - [ ] Retourne le panier fusionné complet

---

## Module 6 : Commandes (P0)

### BE-015 : Création de commande
- **Priorité** : P0
- **Complexité** : XL
- **Description** : Processus de création de commande
- **Critères d'acceptation** :
  - [ ] POST /api/orders : crée la commande à partir du panier
  - [ ] Vérifie que le panier n'est pas vide
  - [ ] Vérifie le stock de chaque article
  - [ ] Copie les données produit (nom, image, prix) dans OrderItem
  - [ ] Décrémente le stock de chaque produit
  - [ ] Calcule subtotal, shippingFee, total
  - [ ] Génère le numéro de commande (CMD-YYYYMMDD-XXXX)
  - [ ] Crée le Payment en PENDING
  - [ ] Vide le panier après création
  - [ ] Transaction Prisma pour atomicité (tout ou rien)

### BE-016 : Gestion des commandes
- **Priorité** : P0
- **Complexité** : M
- **Description** : Consultation et gestion des commandes
- **Critères d'acceptation** :
  - [ ] GET /api/orders : commandes de l'utilisateur connecté (paginé)
  - [ ] GET /api/orders/:id : détail avec items, adresse, paiement
  - [ ] Vérification que la commande appartient à l'utilisateur (ou ADMIN)
  - [ ] GET /api/orders/admin/all (ADMIN) : toutes les commandes avec filtres
  - [ ] PATCH /api/orders/:id/status (ADMIN) : changement de statut
  - [ ] Validation des transitions de statut (pas de retour en arrière)

### BE-017 : Numéro de commande séquentiel
- **Priorité** : P0
- **Complexité** : S
- **Description** : Générer des numéros de commande uniques et séquentiels
- **Critères d'acceptation** :
  - [ ] Format : CMD-YYYYMMDD-XXXX (ex: CMD-20260315-0001)
  - [ ] Compteur journalier remis à 0001 chaque jour
  - [ ] Pas de doublon possible (unique constraint en base)

---

## Module 7 : Paiements (P0)

### BE-018 : Intégration PaiementPro
- **Priorité** : P0
- **Complexité** : XL
- **Description** : Intégration complète avec PaiementPro
- **Critères d'acceptation** :
  - [ ] POST /api/payments/init : appelle l'API PaiementPro pour initier le paiement
  - [ ] Génère un transactionId unique pour le tracking
  - [ ] Retourne l'URL de paiement pour redirection
  - [ ] Support des channels : CARD, ORANGE_MONEY, MTN_MOMO, WAVE, MOOV_FLOOZ
  - [ ] POST /api/payments/callback : traite la notification IPN
  - [ ] Vérifie l'authenticité du callback (anti-replay)
  - [ ] Vérifie que le montant payé = montant attendu
  - [ ] Met à jour Payment et Order en cas de succès
  - [ ] GET /api/payments/:orderId/status : statut actuel du paiement
  - [ ] Logs détaillés de chaque transaction (pour debug)

### BE-019 : Intégration CinetPay (alternative)
- **Priorité** : P2
- **Complexité** : L
- **Description** : Intégration avec CinetPay comme alternative
- **Critères d'acceptation** :
  - [ ] Mêmes fonctionnalités que PaiementPro
  - [ ] Configurable via variable d'environnement (choix du prestataire)
  - [ ] Interface commune (PaymentProvider) pour interchanger facilement

---

## Module 8 : Administration (P1)

### BE-020 : Dashboard statistiques
- **Priorité** : P1
- **Complexité** : L
- **Description** : Endpoint dashboard avec toutes les stats
- **Critères d'acceptation** :
  - [ ] GET /api/admin/dashboard retourne :
  - [ ] Revenue : total, payé, en attente, ce mois, mois dernier, croissance %
  - [ ] Commandes : total par statut
  - [ ] Produits : total, actifs, brouillons, archivés, stock bas
  - [ ] Clients : total, ce mois, actifs
  - [ ] 5 dernières commandes
  - [ ] 5 meilleurs produits (par quantité vendue)
  - [ ] Temps de réponse < 500ms (requêtes optimisées)

### BE-021 : Gestion des clients (admin)
- **Priorité** : P1
- **Complexité** : M
- **Description** : Liste et gestion des clients par l'admin
- **Critères d'acceptation** :
  - [ ] GET /api/admin/clients : liste avec stats (nb commandes, total payé/pending)
  - [ ] Recherche par nom, email, téléphone
  - [ ] Pagination
  - [ ] Tri par date d'inscription, nombre de commandes, total dépensé

### BE-022 : Paramètres du site
- **Priorité** : P1
- **Complexité** : S
- **Description** : CRUD paramètres clé/valeur
- **Critères d'acceptation** :
  - [ ] GET /api/settings/public : paramètres publics (nom site, devise, frais livraison)
  - [ ] GET /api/settings (ADMIN) : tous les paramètres
  - [ ] PATCH /api/settings (ADMIN) : modification en batch

---

## Module 9 : Adresses (P1)

### BE-023 : CRUD Adresses
- **Priorité** : P1
- **Complexité** : M
- **Description** : Gestion des adresses de livraison
- **Critères d'acceptation** :
  - [ ] CRUD complet avec vérification de propriété
  - [ ] Un seul `isDefault` par utilisateur
  - [ ] Champs : label, firstName, lastName, phone, street, city, commune, quarter, instructions
  - [ ] Suppression impossible si adresse liée à une commande active

---

## Module 10 : Utilisateurs (P1)

### BE-024 : Profil utilisateur
- **Priorité** : P1
- **Complexité** : S
- **Description** : Gestion du profil
- **Critères d'acceptation** :
  - [ ] GET /api/users/me : profil complet
  - [ ] PATCH /api/users/me : modification (firstName, lastName, phone)
  - [ ] Ne pas permettre la modification de l'email et du rôle

### BE-025 : Liste utilisateurs (admin)
- **Priorité** : P1
- **Complexité** : S
- **Description** : Liste et gestion des utilisateurs par l'admin
- **Critères d'acceptation** :
  - [ ] GET /api/users : liste paginée avec filtres (rôle, recherche)
  - [ ] PATCH /api/users/:id/status : activer/désactiver

---

## Module 11 : Santé et monitoring (P1)

### BE-026 : Health check
- **Priorité** : P1
- **Complexité** : S
- **Description** : Endpoint de santé
- **Critères d'acceptation** :
  - [ ] GET /api/health : statut API, uptime, connexion DB
  - [ ] Temps de réponse < 50ms

---

## Module 12 : Optimisations (P2)

### BE-027 : Rate limiting
- **Priorité** : P2
- **Complexité** : S
- **Description** : Protection contre les abus
- **Critères d'acceptation** :
  - [ ] Limiter à 100 requêtes/minute par IP
  - [ ] Limiter login à 5 tentatives/minute
  - [ ] Header `X-RateLimit-Remaining` dans les réponses

### BE-028 : Logging structuré
- **Priorité** : P2
- **Complexité** : M
- **Description** : Logs structurés pour le monitoring
- **Critères d'acceptation** :
  - [ ] Format JSON pour les logs en production
  - [ ] Niveaux : error, warn, info, debug
  - [ ] Contexte : userId, requestId, duration
  - [ ] Rotation des fichiers de logs

### BE-029 : Cache des requêtes fréquentes
- **Priorité** : P2
- **Complexité** : M
- **Description** : Cache en mémoire pour les requêtes lourdes
- **Critères d'acceptation** :
  - [ ] Cache des catégories (TTL 5 min)
  - [ ] Cache des paramètres (TTL 5 min)
  - [ ] Cache des produits featured (TTL 2 min)
  - [ ] Invalidation du cache lors de modifications

### BE-030 : Notifications email
- **Priorité** : P2
- **Complexité** : L
- **Description** : Envoi d'emails transactionnels
- **Critères d'acceptation** :
  - [ ] Email de confirmation de commande
  - [ ] Email de changement de statut (expédié, livré)
  - [ ] Email de confirmation de paiement
  - [ ] Templates HTML responsives

---

## Résumé des tâches

| ID | Tâche | Priorité | Complexité | Statut |
|---|---|---|---|---|
| BE-001 | Setup NestJS | P0 | M | TODO |
| BE-002 | Schéma Prisma | P0 | L | TODO |
| BE-003 | Service Prisma | P0 | S | TODO |
| BE-004 | Auth Register | P0 | M | TODO |
| BE-005 | Auth Login | P0 | M | TODO |
| BE-006 | JWT Guard | P0 | M | TODO |
| BE-007 | Token Refresh | P0 | M | TODO |
| BE-008 | Logout + Change Password | P0 | S | TODO |
| BE-009 | CRUD Produits | P0 | L | TODO |
| BE-010 | Recherche autocomplete | P0 | M | TODO |
| BE-011 | Images produits | P0 | M | TODO |
| BE-012 | CRUD Catégories | P0 | M | TODO |
| BE-013 | Panier | P0 | L | TODO |
| BE-014 | Sync panier | P1 | M | TODO |
| BE-015 | Création commande | P0 | XL | TODO |
| BE-016 | Gestion commandes | P0 | M | TODO |
| BE-017 | Numéro commande | P0 | S | TODO |
| BE-018 | PaiementPro | P0 | XL | TODO |
| BE-019 | CinetPay | P2 | L | TODO |
| BE-020 | Dashboard stats | P1 | L | TODO |
| BE-021 | Clients admin | P1 | M | TODO |
| BE-022 | Paramètres | P1 | S | TODO |
| BE-023 | Adresses | P1 | M | TODO |
| BE-024 | Profil utilisateur | P1 | S | TODO |
| BE-025 | Liste utilisateurs | P1 | S | TODO |
| BE-026 | Health check | P1 | S | TODO |
| BE-027 | Rate limiting | P2 | S | TODO |
| BE-028 | Logging | P2 | M | TODO |
| BE-029 | Cache | P2 | M | TODO |
| BE-030 | Notifications email | P2 | L | TODO |

**Total estimé P0** : ~50h
**Total estimé P1** : ~16h
**Total estimé P2** : ~16h
