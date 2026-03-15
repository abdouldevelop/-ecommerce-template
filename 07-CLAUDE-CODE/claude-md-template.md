# Template CLAUDE.md pour projet e-commerce

Copier ce fichier dans `.claude/CLAUDE.md` a la racine du projet genere. Remplacer les placeholders `[A REMPLIR]` par les valeurs du projet.

---

```markdown
# [NOM DU PROJET] - Plateforme E-Commerce

## Description
[A REMPLIR] - Plateforme e-commerce pour [client/marche]. Marche cible : [pays/region].

## Stack technique

| Composant | Technologie | Version |
|-----------|-------------|---------|
| Backend | NestJS | 11+ |
| Frontend | Next.js (App Router) | 15+ |
| ORM | Prisma | 6+ |
| Base de donnees | PostgreSQL | 16 |
| Style | Tailwind CSS | 4 |
| State | Zustand | 5+ |
| Tests | Jest + Playwright | Latest |
| Monorepo | Turborepo | Latest |
| Deploiement | Docker + PM2 + Nginx | - |

## Structure du projet

apps/
  api/          # Backend NestJS
  web/          # Frontend Next.js
packages/
  shared/       # Types et utilitaires partages
  config/       # Configurations partagees (eslint, tsconfig)
prisma/
  schema.prisma # Schema de base de donnees

## Commandes essentielles

### Build
- Build complet : `npm run build`
- Build backend : `cd apps/api && npm run build`
- Build frontend : `cd apps/web && npm run build`
- Generation Prisma : `cd apps/api && npx prisma generate`
- Migration Prisma : `cd apps/api && npx prisma migrate dev`

### Tests
- Tous les tests : `npm test`
- Tests backend : `cd apps/api && npm test`
- Tests frontend : `cd apps/web && npm test`
- Tests E2E : `npx playwright test`
- Coverage : `npm test -- --coverage`

### Dev
- Demarrer tout : `npm run dev`
- Backend seul : `cd apps/api && npm run start:dev`
- Frontend seul : `cd apps/web && npm run dev`

### Lint
- Lint : `npm run lint`
- Format : `npm run format`
- Type check : `npx tsc --noEmit`

## Conventions de code

### Nommage
- Fichiers : kebab-case (`product-category.service.ts`)
- Classes : PascalCase (`ProductCategoryService`)
- Variables/fonctions : camelCase (`getProductById`)
- Constantes : UPPER_SNAKE_CASE (`MAX_UPLOAD_SIZE`)
- Types/Interfaces : PascalCase avec prefixe I pour interfaces (`IProductResponse`)
- Enums : PascalCase (`OrderStatus`)
- Tables BDD : snake_case (`product_category`)

### Structure des fichiers backend (NestJS)

Chaque module suit cette structure :
modules/
  [nom]/
    [nom].module.ts       # Declaration du module
    [nom].controller.ts   # Routes et validation
    [nom].service.ts      # Logique metier
    [nom].repository.ts   # Acces base de donnees (Prisma)
    dto/
      create-[nom].dto.ts # DTO de creation
      update-[nom].dto.ts # DTO de mise a jour
    entities/
      [nom].entity.ts     # Types de reponse

### Structure des fichiers frontend (Next.js)

app/
  (public)/               # Pages publiques (catalogue, produit)
  (auth)/                 # Pages authentification
  (client)/               # Espace client (protege)
  (admin)/                # Panel admin (protege)
  layout.tsx              # Layout racine
components/
  ui/                     # Composants generiques (Button, Input, Modal)
  layout/                 # Header, Footer, Sidebar
  product/                # Composants specifiques produit
  cart/                   # Composants panier
  admin/                  # Composants admin
lib/
  api/                    # Client API (Axios)
  hooks/                  # Custom hooks
  stores/                 # Zustand stores
  utils/                  # Fonctions utilitaires
  types/                  # Types TypeScript

### Imports
- Toujours utiliser les alias de chemin (`@/` pour la racine de chaque app)
- Ordre des imports : 1) packages externes, 2) alias internes, 3) imports relatifs
- Pas d'imports circulaires entre modules NestJS

### API
- Toutes les reponses suivent le format : `{ success: boolean, data?: T, error?: string, message?: string }`
- Validation des entrees avec `class-validator` sur chaque DTO
- Pagination : `?page=1&limit=20` → `{ data: T[], meta: { total, page, limit, pages } }`
- Authentification : Bearer token dans le header Authorization
- Gestion d'erreurs : HttpException avec codes standards

## Regles de securite

### Obligatoire
- JAMAIS de secrets en dur dans le code (utiliser .env)
- JAMAIS exposer les stack traces en production
- TOUJOURS valider et sanitizer les entrees utilisateur
- TOUJOURS utiliser des requetes parametrees (Prisma le fait automatiquement)
- TOUJOURS verifier les permissions avant chaque action (guards NestJS)
- JWT avec expiration courte (30 min) + refresh token (7 jours)
- Rate limiting sur les endpoints sensibles (login, register, paiement)
- Helmet pour les headers de securite
- CORS configure pour le domaine autorise uniquement

### Fichiers interdits
- Ne JAMAIS modifier : `.env`, `.env.local`, `package-lock.json`
- Ne JAMAIS commiter : `node_modules/`, `dist/`, `.next/`, `coverage/`

### Mots de passe
- Hachage bcrypt avec salt rounds >= 12
- Validation : minimum 8 caracteres, 1 majuscule, 1 chiffre

## Taches courantes

### Ajouter un nouveau module backend
1. Creer le dossier `apps/api/src/modules/[nom]/`
2. Creer module, controller, service, repository
3. Creer les DTOs avec validations class-validator
4. Enregistrer le module dans `app.module.ts` imports[]
5. Ajouter les tests unitaires
6. Verifier : `npm run build && npm test`

### Ajouter une nouvelle page frontend
1. Creer le fichier dans `apps/web/app/[groupe]/[route]/page.tsx`
2. Ajouter les composants dans `components/[domaine]/`
3. Ajouter les appels API dans `lib/api/`
4. Ajouter le store Zustand si necessaire
5. Verifier : `npm run build && npm test`

### Modifier le schema de base de donnees
1. Editer `prisma/schema.prisma`
2. Lancer `npx prisma generate` pour regenerer les types
3. Lancer `npx prisma migrate dev --name [description]` pour creer la migration
4. Mettre a jour les DTOs et services concernes
5. Verifier : `npm run build && npm test`

## Deploiement

### Environnements
- Dev : `http://localhost:3000` (API) + `http://localhost:3001` (Web)
- Staging : `https://staging.[A REMPLIR]`
- Production : `https://[A REMPLIR]`

### Procedure
1. Verifier : `npm run build && npm test`
2. Construire les images Docker : `docker compose build`
3. Deployer : `docker compose up -d`
4. Lancer les migrations : `npx prisma migrate deploy`
5. Verifier les logs : `docker compose logs -f`

## Notes specifiques
- Ce projet cible le marche Afrique francophone
- Integrations paiement : PaiementPro / CinetPay (Orange Money, MTN MoMo, Wave)
- Optimiser pour connexions lentes (images compressees, lazy loading, SSR)
- Interface en francais par defaut
```

---

## Instructions d'utilisation

1. Copier le contenu entre les balises ` ``` ` dans un fichier `.claude/CLAUDE.md` a la racine du projet
2. Remplacer tous les `[A REMPLIR]` par les valeurs reelles du projet
3. Adapter la stack si differente (ex: si pas de Turborepo, retirer la ligne)
4. Ajouter des regles specifiques au projet dans `.claude/rules/`
5. Le fichier sera lu automatiquement par Claude Code a chaque session
