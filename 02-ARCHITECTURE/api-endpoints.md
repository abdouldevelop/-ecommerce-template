# Endpoints API - E-Commerce

## Instructions pour l'agent IA

Ce document liste TOUS les endpoints de l'API REST. Chaque endpoint indique la méthode HTTP, le chemin, le niveau d'authentification requis, et le format de requête/réponse.

**Conventions** :
- Base URL : `/api` (via Nginx reverse proxy)
- Format : JSON
- Authentification : JWT Bearer token dans le header `Authorization: Bearer <token>`
- Pagination : `?page=1&limit=20` (défaut: page=1, limit=20, max limit=100)
- Tri : `?sortBy=createdAt&sortOrder=desc`
- Codes de réponse : 200 (OK), 201 (Created), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 500 (Server Error)

**Niveaux d'accès** :
- PUBLIC : Aucune authentification requise
- AUTH : Utilisateur connecté (CUSTOMER ou ADMIN)
- ADMIN : Utilisateur avec rôle ADMIN uniquement

---

## 1. Authentification (`/api/auth`)

### POST /api/auth/register
- **Accès** : PUBLIC
- **Description** : Inscription d'un nouveau client
- **Body** :
```json
{
  "email": "client@email.com",
  "password": "MotDePasse123!",
  "firstName": "Abdoul",
  "lastName": "Diallo",
  "phone": "+2250701234567"
}
```
- **Réponse 201** :
```json
{
  "user": {
    "id": "uuid",
    "email": "client@email.com",
    "firstName": "Abdoul",
    "lastName": "Diallo",
    "phone": "+2250701234567",
    "role": "CUSTOMER"
  },
  "accessToken": "eyJhbGciOi...",
  "refreshToken": "eyJhbGciOi..."
}
```
- **Erreurs** : 400 (email déjà utilisé, validation échouée)
- **Validation** :
  - email : format email valide, unique
  - password : min 8 caractères, 1 majuscule, 1 chiffre
  - firstName : min 2 caractères
  - lastName : min 2 caractères
  - phone : optionnel, format international

### POST /api/auth/login
- **Accès** : PUBLIC
- **Description** : Connexion d'un utilisateur
- **Body** :
```json
{
  "email": "client@email.com",
  "password": "MotDePasse123!"
}
```
- **Réponse 200** :
```json
{
  "user": {
    "id": "uuid",
    "email": "client@email.com",
    "firstName": "Abdoul",
    "lastName": "Diallo",
    "role": "CUSTOMER"
  },
  "accessToken": "eyJhbGciOi...",
  "refreshToken": "eyJhbGciOi..."
}
```
- **Erreurs** : 401 (identifiants incorrects), 403 (compte désactivé)

### POST /api/auth/refresh
- **Accès** : PUBLIC
- **Description** : Renouveler le token d'accès avec le refresh token
- **Body** :
```json
{
  "refreshToken": "eyJhbGciOi..."
}
```
- **Réponse 200** :
```json
{
  "accessToken": "eyJhbGciOi...",
  "refreshToken": "eyJhbGciOi..."
}
```
- **Erreurs** : 401 (refresh token invalide ou expiré)

### POST /api/auth/logout
- **Accès** : AUTH
- **Description** : Déconnexion (invalide le refresh token)
- **Réponse 200** :
```json
{
  "message": "Déconnexion réussie"
}
```

### POST /api/auth/change-password
- **Accès** : AUTH
- **Description** : Changer le mot de passe
- **Body** :
```json
{
  "currentPassword": "AncienMotDePasse123!",
  "newPassword": "NouveauMotDePasse456!"
}
```
- **Réponse 200** :
```json
{
  "message": "Mot de passe modifié avec succès"
}
```
- **Erreurs** : 400 (ancien mot de passe incorrect, nouveau mot de passe trop faible)

---

## 2. Utilisateurs / Profil (`/api/users`)

### GET /api/users/me
- **Accès** : AUTH
- **Description** : Récupérer le profil de l'utilisateur connecté
- **Réponse 200** :
```json
{
  "id": "uuid",
  "email": "client@email.com",
  "firstName": "Abdoul",
  "lastName": "Diallo",
  "phone": "+2250701234567",
  "role": "CUSTOMER",
  "createdAt": "2026-03-15T10:00:00Z"
}
```

### PATCH /api/users/me
- **Accès** : AUTH
- **Description** : Modifier le profil de l'utilisateur connecté
- **Body** :
```json
{
  "firstName": "Abdoul",
  "lastName": "Diallo",
  "phone": "+2250707654321"
}
```
- **Réponse 200** : Profil mis à jour

### GET /api/users (Admin)
- **Accès** : ADMIN
- **Description** : Lister tous les utilisateurs
- **Query** : `?page=1&limit=20&role=CUSTOMER&search=abdoul`
- **Réponse 200** :
```json
{
  "data": [
    {
      "id": "uuid",
      "email": "client@email.com",
      "firstName": "Abdoul",
      "lastName": "Diallo",
      "role": "CUSTOMER",
      "isActive": true,
      "ordersCount": 5,
      "totalSpent": 150000,
      "createdAt": "2026-03-15T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "totalPages": 3
  }
}
```

### PATCH /api/users/:id/status (Admin)
- **Accès** : ADMIN
- **Description** : Activer/désactiver un utilisateur
- **Body** :
```json
{
  "isActive": false
}
```

---

## 3. Produits (`/api/products`)

### GET /api/products
- **Accès** : PUBLIC
- **Description** : Lister les produits (actifs uniquement pour le public)
- **Query** : `?page=1&limit=20&categoryId=uuid&search=chemise&sortBy=price&sortOrder=asc&featured=true&minPrice=5000&maxPrice=50000`
- **Réponse 200** :
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "T-Shirt Premium",
      "slug": "t-shirt-premium",
      "description": "T-shirt en coton égyptien",
      "price": 15000,
      "comparePrice": 20000,
      "stock": 25,
      "isFeatured": true,
      "status": "ACTIVE",
      "category": {
        "id": "uuid",
        "name": "T-Shirts",
        "slug": "t-shirts"
      },
      "primaryImage": {
        "url": "/uploads/products/tshirt-premium.jpg",
        "alt": "T-Shirt Premium"
      },
      "createdAt": "2026-03-15T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "totalPages": 3
  }
}
```

### GET /api/products/featured
- **Accès** : PUBLIC
- **Description** : Récupérer les produits vedettes (pour la page d'accueil)
- **Query** : `?limit=8`
- **Réponse 200** : Même format que GET /api/products

### GET /api/products/search
- **Accès** : PUBLIC
- **Description** : Recherche autocomplete
- **Query** : `?q=che&limit=5`
- **Réponse 200** :
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "Chemise classique",
      "slug": "chemise-classique",
      "price": 25000,
      "primaryImage": {
        "url": "/uploads/products/chemise.jpg"
      }
    }
  ]
}
```

### GET /api/products/:slug
- **Accès** : PUBLIC
- **Description** : Détail d'un produit par son slug
- **Réponse 200** :
```json
{
  "id": "uuid",
  "name": "T-Shirt Premium",
  "slug": "t-shirt-premium",
  "description": "T-shirt en coton égyptien",
  "longDescription": "<p>Description complète en HTML...</p>",
  "price": 15000,
  "comparePrice": 20000,
  "stock": 25,
  "isFeatured": true,
  "status": "ACTIVE",
  "category": {
    "id": "uuid",
    "name": "T-Shirts",
    "slug": "t-shirts"
  },
  "images": [
    { "id": "uuid", "url": "/uploads/products/tshirt-1.jpg", "alt": "Face", "isPrimary": true },
    { "id": "uuid", "url": "/uploads/products/tshirt-2.jpg", "alt": "Dos", "isPrimary": false },
    { "id": "uuid", "url": "/uploads/products/tshirt-3.jpg", "alt": "Détail", "isPrimary": false }
  ],
  "viewCount": 142,
  "createdAt": "2026-03-15T10:00:00Z"
}
```
- **Erreurs** : 404 (produit non trouvé ou non actif)

### POST /api/products (Admin)
- **Accès** : ADMIN
- **Description** : Créer un produit
- **Body** :
```json
{
  "name": "T-Shirt Premium",
  "description": "T-shirt en coton égyptien",
  "longDescription": "<p>Description complète...</p>",
  "price": 15000,
  "comparePrice": 20000,
  "stock": 25,
  "categoryId": "uuid",
  "status": "DRAFT",
  "isFeatured": false
}
```
- **Réponse 201** : Produit créé
- **Note** : Le slug est généré automatiquement à partir du nom

### PATCH /api/products/:id (Admin)
- **Accès** : ADMIN
- **Description** : Modifier un produit
- **Body** : Champs à modifier (partiel)
- **Réponse 200** : Produit mis à jour

### DELETE /api/products/:id (Admin)
- **Accès** : ADMIN
- **Description** : Supprimer un produit (ou archiver si commandes existantes)
- **Réponse 200** :
```json
{
  "message": "Produit supprimé"
}
```
- **Logique** : Si le produit a des commandes, le passer en ARCHIVED au lieu de supprimer

### GET /api/products/admin/all (Admin)
- **Accès** : ADMIN
- **Description** : Lister tous les produits (incluant DRAFT et ARCHIVED)
- **Query** : `?page=1&limit=20&status=DRAFT&categoryId=uuid&search=chemise`

---

## 4. Catégories (`/api/categories`)

### GET /api/categories
- **Accès** : PUBLIC
- **Description** : Lister toutes les catégories actives
- **Réponse 200** :
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "T-Shirts",
      "slug": "t-shirts",
      "description": "Nos t-shirts",
      "image": "/uploads/categories/tshirts.jpg",
      "productCount": 12,
      "sortOrder": 1,
      "children": []
    }
  ]
}
```

### GET /api/categories/:slug
- **Accès** : PUBLIC
- **Description** : Détail d'une catégorie avec ses produits
- **Query** : `?page=1&limit=20`

### POST /api/categories (Admin)
- **Accès** : ADMIN
- **Description** : Créer une catégorie
- **Body** :
```json
{
  "name": "Chaussures",
  "description": "Toutes nos chaussures",
  "parentId": null,
  "sortOrder": 5
}
```
- **Réponse 201** : Catégorie créée

### PATCH /api/categories/:id (Admin)
- **Accès** : ADMIN
- **Description** : Modifier une catégorie
- **Body** : Champs à modifier (partiel)

### DELETE /api/categories/:id (Admin)
- **Accès** : ADMIN
- **Description** : Supprimer une catégorie (seulement si aucun produit associé)
- **Erreurs** : 400 (catégorie contient des produits)

---

## 5. Panier (`/api/cart`)

### GET /api/cart
- **Accès** : AUTH
- **Description** : Récupérer le panier de l'utilisateur connecté
- **Réponse 200** :
```json
{
  "id": "uuid",
  "items": [
    {
      "id": "uuid",
      "product": {
        "id": "uuid",
        "name": "T-Shirt Premium",
        "slug": "t-shirt-premium",
        "price": 15000,
        "stock": 25,
        "primaryImage": {
          "url": "/uploads/products/tshirt-1.jpg"
        }
      },
      "quantity": 2,
      "subtotal": 30000
    }
  ],
  "itemsCount": 2,
  "subtotal": 30000,
  "shippingFee": 2000,
  "total": 32000
}
```

### POST /api/cart/items
- **Accès** : AUTH
- **Description** : Ajouter un produit au panier
- **Body** :
```json
{
  "productId": "uuid",
  "quantity": 1
}
```
- **Réponse 201** : Article ajouté
- **Logique** : Si le produit est déjà dans le panier, augmenter la quantité
- **Erreurs** : 400 (stock insuffisant), 404 (produit non trouvé)

### PATCH /api/cart/items/:itemId
- **Accès** : AUTH
- **Description** : Modifier la quantité d'un article
- **Body** :
```json
{
  "quantity": 3
}
```
- **Réponse 200** : Quantité mise à jour
- **Erreurs** : 400 (quantity < 1 ou > stock)

### DELETE /api/cart/items/:itemId
- **Accès** : AUTH
- **Description** : Retirer un article du panier
- **Réponse 200** :
```json
{
  "message": "Article retiré du panier"
}
```

### DELETE /api/cart
- **Accès** : AUTH
- **Description** : Vider le panier
- **Réponse 200** :
```json
{
  "message": "Panier vidé"
}
```

### POST /api/cart/sync
- **Accès** : AUTH
- **Description** : Synchroniser le panier local (localStorage) avec le serveur
- **Body** :
```json
{
  "items": [
    { "productId": "uuid", "quantity": 2 },
    { "productId": "uuid", "quantity": 1 }
  ]
}
```
- **Réponse 200** : Panier synchronisé (fusion avec le panier existant côté serveur)

---

## 6. Commandes (`/api/orders`)

### POST /api/orders
- **Accès** : AUTH
- **Description** : Créer une commande à partir du panier
- **Body** :
```json
{
  "addressId": "uuid",
  "notes": "Livrer entre 14h et 18h",
  "paymentMethod": "ORANGE_MONEY"
}
```
- **Réponse 201** :
```json
{
  "id": "uuid",
  "orderNumber": "CMD-20260315-0001",
  "status": "PENDING",
  "subtotal": 30000,
  "shippingFee": 2000,
  "total": 32000,
  "items": [...],
  "createdAt": "2026-03-15T14:30:00Z"
}
```
- **Logique** :
  1. Valider le stock de chaque article
  2. Créer la commande avec les articles (copier prix et nom)
  3. Décrémenter le stock
  4. Vider le panier
  5. Créer le paiement en PENDING
  6. Générer le numéro de commande : `CMD-YYYYMMDD-XXXX`

### GET /api/orders
- **Accès** : AUTH
- **Description** : Historique des commandes de l'utilisateur connecté
- **Query** : `?page=1&limit=10&status=DELIVERED`
- **Réponse 200** :
```json
{
  "data": [
    {
      "id": "uuid",
      "orderNumber": "CMD-20260315-0001",
      "status": "DELIVERED",
      "total": 32000,
      "itemsCount": 2,
      "payment": {
        "status": "COMPLETED",
        "method": "ORANGE_MONEY"
      },
      "createdAt": "2026-03-15T14:30:00Z",
      "deliveredAt": "2026-03-17T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 5,
    "totalPages": 1
  }
}
```

### GET /api/orders/:id
- **Accès** : AUTH (propriétaire) ou ADMIN
- **Description** : Détail d'une commande
- **Réponse 200** : Commande complète avec items, adresse, paiement

### GET /api/orders/admin/all (Admin)
- **Accès** : ADMIN
- **Description** : Lister toutes les commandes
- **Query** : `?page=1&limit=20&status=PENDING&search=CMD-20260315`

### PATCH /api/orders/:id/status (Admin)
- **Accès** : ADMIN
- **Description** : Changer le statut d'une commande
- **Body** :
```json
{
  "status": "SHIPPED",
  "adminNotes": "Envoyée via transporteur Express"
}
```
- **Réponse 200** : Commande mise à jour
- **Logique** : Vérifier que la transition de statut est valide (pas de retour en arrière sauf annulation)

---

## 7. Paiements (`/api/payments`)

### POST /api/payments/init
- **Accès** : AUTH
- **Description** : Initier un paiement auprès du prestataire
- **Body** :
```json
{
  "orderId": "uuid",
  "channel": "ORANGE_MONEY"
}
```
- **Réponse 200** :
```json
{
  "paymentUrl": "https://www.paiementpro.net/webservice/onlinepayment/processing/pay.php?ref=xxx",
  "transactionId": "TXN-20260315-abc123",
  "expiresAt": "2026-03-15T15:00:00Z"
}
```
- **Logique** :
  1. Vérifier que la commande appartient à l'utilisateur
  2. Vérifier que la commande est en PENDING
  3. Appeler l'API du prestataire pour initier le paiement
  4. Retourner l'URL de redirection

### POST /api/payments/callback
- **Accès** : PUBLIC (appelé par le prestataire)
- **Description** : Callback de notification de paiement (IPN)
- **Body** : Format spécifique au prestataire
- **Logique** :
  1. Vérifier l'authenticité de la notification (anti-replay)
  2. Vérifier que le montant correspond
  3. Mettre à jour le statut du paiement
  4. Si paiement réussi : passer la commande en CONFIRMED
  5. Répondre 200 au prestataire

### GET /api/payments/:orderId/status
- **Accès** : AUTH
- **Description** : Vérifier le statut du paiement d'une commande
- **Réponse 200** :
```json
{
  "orderId": "uuid",
  "orderNumber": "CMD-20260315-0001",
  "paymentStatus": "COMPLETED",
  "orderStatus": "CONFIRMED",
  "method": "ORANGE_MONEY",
  "amount": 32000,
  "paidAt": "2026-03-15T14:35:00Z"
}
```

---

## 8. Adresses (`/api/addresses`)

### GET /api/addresses
- **Accès** : AUTH
- **Description** : Lister les adresses de l'utilisateur

### POST /api/addresses
- **Accès** : AUTH
- **Description** : Ajouter une adresse
- **Body** :
```json
{
  "label": "Maison",
  "firstName": "Abdoul",
  "lastName": "Diallo",
  "phone": "+2250701234567",
  "street": "Lot 125, Rue des jardins",
  "city": "Abidjan",
  "commune": "Cocody",
  "quarter": "Angré 8ème tranche",
  "instructions": "Face à la pharmacie, portail bleu",
  "isDefault": true
}
```

### PATCH /api/addresses/:id
- **Accès** : AUTH (propriétaire)
- **Description** : Modifier une adresse

### DELETE /api/addresses/:id
- **Accès** : AUTH (propriétaire)
- **Description** : Supprimer une adresse

---

## 9. Upload (`/api/upload`)

### POST /api/upload/product-image
- **Accès** : ADMIN
- **Description** : Uploader une image produit
- **Body** : `multipart/form-data` avec champ `image`
- **Contraintes** : Max 5 Mo, formats acceptés : JPG, PNG, WebP
- **Réponse 201** :
```json
{
  "url": "/uploads/products/1710504600-tshirt.jpg",
  "filename": "1710504600-tshirt.jpg"
}
```

### POST /api/upload/category-image
- **Accès** : ADMIN
- **Description** : Uploader une image de catégorie
- **Body** : `multipart/form-data` avec champ `image`
- **Réponse 201** : Même format

### DELETE /api/upload/:filename
- **Accès** : ADMIN
- **Description** : Supprimer un fichier uploadé

---

## 10. Dashboard Admin (`/api/admin`)

### GET /api/admin/dashboard
- **Accès** : ADMIN
- **Description** : Statistiques pour le dashboard admin
- **Réponse 200** :
```json
{
  "revenue": {
    "total": 2500000,
    "paid": 1800000,
    "pending": 700000,
    "thisMonth": 450000,
    "lastMonth": 380000,
    "growth": 18.4
  },
  "orders": {
    "total": 85,
    "pending": 5,
    "confirmed": 3,
    "processing": 2,
    "shipped": 4,
    "delivered": 68,
    "cancelled": 3
  },
  "products": {
    "total": 45,
    "active": 38,
    "draft": 5,
    "archived": 2,
    "lowStock": 3
  },
  "customers": {
    "total": 120,
    "thisMonth": 15,
    "active": 95
  },
  "recentOrders": [
    {
      "id": "uuid",
      "orderNumber": "CMD-20260315-0001",
      "customer": "Abdoul Diallo",
      "total": 32000,
      "status": "PENDING",
      "createdAt": "2026-03-15T14:30:00Z"
    }
  ],
  "topProducts": [
    {
      "id": "uuid",
      "name": "T-Shirt Premium",
      "totalSold": 42,
      "revenue": 630000
    }
  ]
}
```

### GET /api/admin/clients
- **Accès** : ADMIN
- **Description** : Liste des clients avec statistiques de commandes
- **Query** : `?page=1&limit=20&search=abdoul`
- **Réponse 200** :
```json
{
  "data": [
    {
      "id": "uuid",
      "firstName": "Abdoul",
      "lastName": "Diallo",
      "email": "client@email.com",
      "phone": "+2250701234567",
      "ordersCount": 5,
      "paidOrdersCount": 4,
      "pendingOrdersCount": 1,
      "totalPaid": 120000,
      "totalPending": 32000,
      "lastOrderAt": "2026-03-15T14:30:00Z",
      "createdAt": "2026-01-10T08:00:00Z"
    }
  ],
  "pagination": { "page": 1, "limit": 20, "total": 120, "totalPages": 6 }
}
```

---

## 11. Paramètres (`/api/settings`)

### GET /api/settings/public
- **Accès** : PUBLIC
- **Description** : Récupérer les paramètres publics du site (nom, devise, frais livraison)

### GET /api/settings (Admin)
- **Accès** : ADMIN
- **Description** : Récupérer tous les paramètres

### PATCH /api/settings (Admin)
- **Accès** : ADMIN
- **Description** : Modifier un ou plusieurs paramètres
- **Body** :
```json
{
  "settings": [
    { "key": "shipping_fee_default", "value": "3000" },
    { "key": "free_shipping_threshold", "value": "30000" }
  ]
}
```

---

## 12. Santé (`/api/health`)

### GET /api/health
- **Accès** : PUBLIC
- **Description** : Vérifier que l'API est opérationnelle
- **Réponse 200** :
```json
{
  "status": "ok",
  "timestamp": "2026-03-15T14:30:00Z",
  "uptime": 86400,
  "database": "connected"
}
```

---

## Résumé des endpoints

| Méthode | Chemin | Accès | Description |
|---|---|---|---|
| POST | /api/auth/register | PUBLIC | Inscription |
| POST | /api/auth/login | PUBLIC | Connexion |
| POST | /api/auth/refresh | PUBLIC | Rafraîchir le token |
| POST | /api/auth/logout | AUTH | Déconnexion |
| POST | /api/auth/change-password | AUTH | Changer le mot de passe |
| GET | /api/users/me | AUTH | Mon profil |
| PATCH | /api/users/me | AUTH | Modifier mon profil |
| GET | /api/users | ADMIN | Liste des utilisateurs |
| PATCH | /api/users/:id/status | ADMIN | Activer/désactiver un utilisateur |
| GET | /api/products | PUBLIC | Liste des produits |
| GET | /api/products/featured | PUBLIC | Produits vedettes |
| GET | /api/products/search | PUBLIC | Recherche autocomplete |
| GET | /api/products/:slug | PUBLIC | Détail produit |
| POST | /api/products | ADMIN | Créer un produit |
| PATCH | /api/products/:id | ADMIN | Modifier un produit |
| DELETE | /api/products/:id | ADMIN | Supprimer un produit |
| GET | /api/products/admin/all | ADMIN | Liste admin tous produits |
| GET | /api/categories | PUBLIC | Liste des catégories |
| GET | /api/categories/:slug | PUBLIC | Détail catégorie |
| POST | /api/categories | ADMIN | Créer une catégorie |
| PATCH | /api/categories/:id | ADMIN | Modifier une catégorie |
| DELETE | /api/categories/:id | ADMIN | Supprimer une catégorie |
| GET | /api/cart | AUTH | Mon panier |
| POST | /api/cart/items | AUTH | Ajouter au panier |
| PATCH | /api/cart/items/:itemId | AUTH | Modifier quantité |
| DELETE | /api/cart/items/:itemId | AUTH | Retirer du panier |
| DELETE | /api/cart | AUTH | Vider le panier |
| POST | /api/cart/sync | AUTH | Synchroniser le panier |
| POST | /api/orders | AUTH | Créer une commande |
| GET | /api/orders | AUTH | Mes commandes |
| GET | /api/orders/:id | AUTH/ADMIN | Détail commande |
| GET | /api/orders/admin/all | ADMIN | Toutes les commandes |
| PATCH | /api/orders/:id/status | ADMIN | Changer statut commande |
| POST | /api/payments/init | AUTH | Initier un paiement |
| POST | /api/payments/callback | PUBLIC | Callback paiement (IPN) |
| GET | /api/payments/:orderId/status | AUTH | Statut paiement |
| GET | /api/addresses | AUTH | Mes adresses |
| POST | /api/addresses | AUTH | Ajouter une adresse |
| PATCH | /api/addresses/:id | AUTH | Modifier une adresse |
| DELETE | /api/addresses/:id | AUTH | Supprimer une adresse |
| POST | /api/upload/product-image | ADMIN | Upload image produit |
| POST | /api/upload/category-image | ADMIN | Upload image catégorie |
| DELETE | /api/upload/:filename | ADMIN | Supprimer un fichier |
| GET | /api/admin/dashboard | ADMIN | Dashboard stats |
| GET | /api/admin/clients | ADMIN | Liste clients + stats |
| GET | /api/settings/public | PUBLIC | Paramètres publics |
| GET | /api/settings | ADMIN | Tous les paramètres |
| PATCH | /api/settings | ADMIN | Modifier paramètres |
| GET | /api/health | PUBLIC | Santé API |

**Total : 42 endpoints**
