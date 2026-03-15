# Stack Technique - E-Commerce

## Instructions pour l'agent IA

Ce document décrit la stack technique recommandée pour un projet e-commerce. Chaque choix est justifié. Les alternatives sont listées si le client a des contraintes spécifiques.

---

## Stack recommandée

### Vue d'ensemble

```
┌──────────────────────────────────────────────────┐
│                   Client (Browser)                │
│              Next.js 15 (App Router)              │
│           Tailwind CSS + Zustand + Axios          │
└───────────────────┬──────────────────────────────┘
                    │ HTTPS
┌───────────────────▼──────────────────────────────┐
│               Nginx (Reverse Proxy)               │
│            SSL + Compression + Cache              │
└───────┬───────────────────────┬──────────────────┘
        │ :3000                 │ :3001
┌───────▼──────┐        ┌──────▼───────────────────┐
│   Next.js    │        │       NestJS             │
│   (SSR/SSG)  │        │   REST API + JWT         │
│   Frontend   │        │   Backend                │
└──────────────┘        └──────┬───────────────────┘
                               │ Prisma ORM
                        ┌──────▼───────────────────┐
                        │     PostgreSQL 16         │
                        │     Base de données       │
                        └──────────────────────────┘
```

### Monorepo Turborepo

```
ecommerce/
├── apps/
│   ├── api/                    # NestJS backend
│   │   ├── src/
│   │   │   ├── modules/
│   │   │   │   ├── auth/       # Authentification JWT
│   │   │   │   ├── users/      # Gestion utilisateurs
│   │   │   │   ├── products/   # Produits CRUD
│   │   │   │   ├── categories/ # Catégories CRUD
│   │   │   │   ├── cart/       # Panier
│   │   │   │   ├── orders/     # Commandes
│   │   │   │   ├── payments/   # Intégration paiement
│   │   │   │   ├── upload/     # Upload images
│   │   │   │   └── admin/      # Endpoints admin
│   │   │   ├── common/
│   │   │   │   ├── guards/     # Auth guards
│   │   │   │   ├── filters/    # Exception filters
│   │   │   │   ├── pipes/      # Validation pipes
│   │   │   │   ├── decorators/ # Custom decorators
│   │   │   │   └── interceptors/
│   │   │   ├── prisma/
│   │   │   │   ├── schema.prisma
│   │   │   │   └── migrations/
│   │   │   ├── app.module.ts
│   │   │   └── main.ts
│   │   ├── uploads/            # Fichiers uploadés
│   │   ├── nest-cli.json
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   └── web/                    # Next.js frontend
│       ├── app/
│       │   ├── (shop)/         # Routes boutique
│       │   │   ├── page.tsx          # Accueil
│       │   │   ├── products/
│       │   │   │   ├── page.tsx      # Catalogue
│       │   │   │   └── [slug]/
│       │   │   │       └── page.tsx  # Détail produit
│       │   │   ├── cart/
│       │   │   │   └── page.tsx      # Panier
│       │   │   ├── checkout/
│       │   │   │   └── page.tsx      # Checkout
│       │   │   └── payment/
│       │   │       ├── page.tsx      # Page paiement
│       │   │       └── result/
│       │   │           └── page.tsx  # Résultat paiement
│       │   ├── (auth)/
│       │   │   ├── login/page.tsx
│       │   │   └── register/page.tsx
│       │   ├── account/              # Espace client
│       │   │   ├── page.tsx          # Dashboard
│       │   │   ├── orders/page.tsx
│       │   │   └── profile/page.tsx
│       │   ├── admin/                # Panel admin
│       │   │   ├── page.tsx          # Dashboard admin
│       │   │   ├── products/
│       │   │   ├── categories/
│       │   │   ├── orders/
│       │   │   └── clients/
│       │   └── layout.tsx
│       ├── components/
│       │   ├── ui/               # Composants réutilisables
│       │   ├── shop/             # Composants boutique
│       │   ├── admin/            # Composants admin
│       │   └── layout/           # Header, Footer, Sidebar
│       ├── lib/
│       │   ├── api.ts            # Client API (axios)
│       │   ├── auth.ts           # Helpers auth
│       │   └── utils.ts          # Utilitaires
│       ├── stores/
│       │   ├── cart.store.ts     # Zustand cart store
│       │   ├── auth.store.ts     # Zustand auth store
│       │   └── search.store.ts   # Zustand search store
│       ├── types/
│       │   └── index.ts          # Types TypeScript
│       ├── next.config.ts
│       ├── tailwind.config.ts
│       ├── tsconfig.json
│       └── package.json
│
├── packages/
│   └── shared/                   # Code partagé
│       ├── types/                # Types communs
│       ├── constants/            # Constantes
│       └── utils/                # Utilitaires communs
│
├── turbo.json                    # Config Turborepo
├── package.json                  # Root package.json
├── docker-compose.yml            # Docker pour le développement
├── .env                          # Variables d'environnement
└── .gitignore
```

---

## Détail de chaque choix technique

### 1. Backend : NestJS

**Version** : NestJS 10+
**Justification** :
- Architecture modulaire proche de Angular (modules, controllers, services)
- Support natif de TypeScript
- Injection de dépendances intégrée
- Guards pour l'authentification, Pipes pour la validation
- Swagger auto-généré pour la documentation API
- Excellente intégration Prisma
- Large communauté et documentation exhaustive

**Alternatives** :

| Alternative | Quand l'utiliser | Inconvénients |
|---|---|---|
| Fastify (standalone) | Projet léger, performance critique | Pas de structure imposée, plus de boilerplate |
| Express.js | Equipe habituée à Express | Pas de TypeScript natif, pas de structure |
| Strapi | Client veut un CMS headless | Moins flexible, overhead pour du e-commerce |
| AdonisJS | Fans de Laravel | Communauté plus petite |

### 2. Frontend : Next.js 15 (App Router)

**Version** : Next.js 15+ avec App Router
**Justification** :
- SSR (Server-Side Rendering) pour le SEO des pages produits
- App Router avec React Server Components pour la performance
- Optimisation automatique des images (`next/image`)
- API Routes intégrées (utile pour les callbacks de paiement)
- Excellent support TypeScript
- Déploiement simple avec PM2 ou Docker

**Alternatives** :

| Alternative | Quand l'utiliser | Inconvénients |
|---|---|---|
| Nuxt.js 3 | Equipe Vue.js | Ecosystème plus petit que React |
| Remix | UX très poussée | Courbe d'apprentissage, communauté plus petite |
| SvelteKit | Performance maximale | Ecosystème limité, peu de développeurs |
| React (SPA) | Panel admin uniquement | Pas de SSR, mauvais SEO |

### 3. Base de données : PostgreSQL 16

**Version** : PostgreSQL 16
**Justification** :
- Robuste et éprouvé en production
- Support JSON natif (utile pour les métadonnées produits)
- Recherche full-text intégrée (recherche produits)
- Excellent avec Prisma
- Gratuit et open source
- Performance excellente pour les requêtes complexes (stats admin)

**Alternatives** :

| Alternative | Quand l'utiliser | Inconvénients |
|---|---|---|
| MySQL 8 | Infrastructure existante MySQL | Moins de fonctionnalités que PostgreSQL |
| MongoDB | Catalogue très hétérogène | Pas de relations, transactions limitées |
| SQLite | Prototype/MVP rapide | Pas de connexions concurrentes |

### 4. ORM : Prisma

**Version** : Prisma 6+
**Justification** :
- Schema déclaratif lisible (`schema.prisma`)
- Migrations automatiques et reproductibles
- Client TypeScript auto-généré (typage parfait)
- Requêtes type-safe (pas de SQL en string)
- Seeding intégré pour les données initiales
- Prisma Studio pour explorer la base visuellement

**Alternatives** :

| Alternative | Quand l'utiliser | Inconvénients |
|---|---|---|
| TypeORM | Projet Angular/NestJS existant | Plus verbose, types moins fiables |
| Drizzle ORM | Performance maximale | Plus récent, moins de documentation |
| Knex.js | Requêtes SQL complexes | Pas de génération de types |

### 5. Style : Tailwind CSS

**Version** : Tailwind CSS 4
**Justification** :
- Utility-first : développement rapide
- Pas de CSS personnalisé à maintenir (ou très peu)
- Design system intégré (spacing, colors, typography)
- Responsive natif avec les breakpoints `sm:`, `md:`, `lg:`
- Dark mode intégré avec `dark:`
- Tree-shaking automatique (CSS minimal en production)

**Alternatives** :

| Alternative | Quand l'utiliser | Inconvénients |
|---|---|---|
| Shadcn/ui | Composants pré-faits nécessaires | Dépendance supplémentaire |
| Chakra UI | Prototypage rapide | Bundle plus lourd |
| CSS Modules | Isolation CSS stricte | Plus lent à développer |
| Styled Components | Equipe habituée | Performance SSR, bundle size |

### 6. Gestion d'état : Zustand

**Justification** :
- Ultra-léger (< 1 KB)
- API simple et intuitive
- Pas de boilerplate (contrairement à Redux)
- Persist middleware pour localStorage (panier)
- Fonctionne avec React Server Components
- DevTools disponibles

**Utilisation dans le projet** :
- `cart.store.ts` : Gestion du panier (ajout, suppression, quantité, total)
- `auth.store.ts` : Etat de connexion, token, user
- `search.store.ts` : Terme de recherche, suggestions, catégorie active

### 7. Monorepo : Turborepo

**Justification** :
- Un seul dépôt Git pour frontend + backend
- Builds parallèles et cache intelligent
- Partage de types TypeScript entre api et web
- Scripts unifiés (`turbo dev`, `turbo build`)
- Configuration minimale

**turbo.json** exemple :
```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {}
  }
}
```

### 8. Authentification : JWT + Refresh Token

**Justification** :
- Stateless (pas de session serveur)
- Access token court (30 minutes)
- Refresh token long (7 jours)
- Fonctionne avec mobile et web
- Guards NestJS natifs pour la protection des routes

### 9. Upload d'images : Multer + stockage local

**Justification** :
- Simple à mettre en place
- Pas de dépendance externe (pas de S3/Cloudinary)
- Suffisant pour un e-commerce de taille moyenne (< 10 000 produits)
- Sauvegarde avec le backup du serveur

**Alternative pour scale** : AWS S3 ou Cloudinary si > 10 000 images

### 10. Déploiement : Docker + PM2 + Nginx

**Justification** :
- Docker : environnement reproductible
- PM2 : process manager avec restart automatique et clustering
- Nginx : reverse proxy, SSL, compression gzip, cache statique
- Let's Encrypt : SSL gratuit avec renouvellement automatique (certbot)

---

## Variables d'environnement

```env
# Backend (apps/api/.env)
DATABASE_URL="postgresql://user:password@localhost:5432/ecommerce?schema=public"
JWT_SECRET="votre-secret-jwt-256-bits-minimum"
JWT_REFRESH_SECRET="votre-secret-refresh-different"
JWT_EXPIRATION="30m"
JWT_REFRESH_EXPIRATION="7d"
PORT=3001
UPLOAD_DIR="./uploads"
MAX_FILE_SIZE=5242880

# Paiement (PaiementPro)
PAIEMENTPRO_MERCHANT_ID="votre-merchant-id"
PAIEMENTPRO_API_KEY="votre-api-key"
PAIEMENTPRO_API_SECRET="votre-api-secret"
PAIEMENTPRO_BASE_URL="https://www.paiementpro.net/webservice/onlinepayment/init/initiate-payment.php"
PAIEMENTPRO_NOTIFY_URL="https://votredomaine.com/api/payments/callback"
PAIEMENTPRO_RETURN_URL="https://votredomaine.com/payment/result"
PAIEMENTPRO_CANCEL_URL="https://votredomaine.com/payment/result?status=cancelled"

# Frontend (apps/web/.env)
NEXT_PUBLIC_API_URL="http://localhost:3001"
NEXT_PUBLIC_SITE_URL="http://localhost:3000"
NEXT_PUBLIC_SITE_NAME="Ma Boutique"
NEXT_PUBLIC_WHATSAPP_NUMBER="+2250700000000"
```

---

## Commandes de développement

```bash
# Installation
npm install                # Installe toutes les dépendances (monorepo)

# Développement
npm run dev                # Lance api + web en parallèle
npm run dev --filter=api   # Lance seulement le backend
npm run dev --filter=web   # Lance seulement le frontend

# Base de données
cd apps/api
npx prisma migrate dev     # Applique les migrations
npx prisma generate        # Génère le client Prisma
npx prisma studio          # Interface visuelle de la base
npx prisma db seed         # Peuple la base avec des données initiales

# Build production
npm run build              # Build api + web
npm run start              # Lance en mode production

# Lint et format
npm run lint               # Vérifie le code
npm run format             # Formate le code
```

---

## Configuration Nginx (production)

```nginx
server {
    listen 80;
    server_name votredomaine.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name votredomaine.com;

    ssl_certificate /etc/letsencrypt/live/votredomaine.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/votredomaine.com/privkey.pem;

    # Frontend
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API
    location /api/ {
        proxy_pass http://127.0.0.1:3001/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 120s;
        proxy_send_timeout 120s;
        proxy_read_timeout 120s;
        client_max_body_size 10M;
    }

    # Images uploadées
    location /uploads/ {
        alias /var/www/ecommerce/apps/api/uploads/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml;
    gzip_min_length 1000;
}
```
