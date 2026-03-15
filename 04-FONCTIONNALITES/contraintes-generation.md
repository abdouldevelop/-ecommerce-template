# Contraintes pour la generation de code par IA

## Instructions pour l'agent IA

Ce document definit les regles obligatoires que Claude Code doit suivre lors de la generation automatique d'un projet e-commerce. Ces regles sont basees sur les problemes reels rencontres lors des premiers builds automatises.

---

## 1. Regles de configuration des fichiers

### Fichiers de configuration : TOUJOURS en `.js`

**REGLE ABSOLUE** : Ne JAMAIS creer de fichiers de configuration en `.ts`. Utiliser uniquement `.js` avec `module.exports`.

| Fichier | Format OBLIGATOIRE | Exemple |
|---|---|---|
| `next.config.js` | CommonJS `.js` | `module.exports = { reactStrictMode: true }` |
| `tailwind.config.js` | CommonJS `.js` | `module.exports = { content: [...], theme: {...} }` |
| `postcss.config.js` | CommonJS `.js` | `module.exports = { plugins: { tailwindcss: {}, autoprefixer: {} } }` |
| `nest-cli.json` | JSON | Standard NestJS config |
| `tsconfig.json` | JSON | Standard TypeScript config |

**Pourquoi** : Next.js 14 ne supporte pas `next.config.ts`. Le build echoue silencieusement. Tailwind et PostCSS ont le meme probleme dans certaines versions.

### Fichier `.env`

Le fichier `.env` doit etre cree a la racine du projet ET copie dans le dossier Prisma :

```bash
# Structure correcte
apps/api/.env              # Variables du backend
apps/api/prisma/.env       # COPIE pour que Prisma trouve DATABASE_URL
apps/web/.env.local        # Variables du frontend (NEXT_PUBLIC_*)
```

**Contenu minimum de `apps/api/.env`** :
```env
DATABASE_URL="postgresql://user:password@localhost:5432/dbname?schema=public"
JWT_SECRET="secret-256-bits-minimum"
JWT_REFRESH_SECRET="autre-secret-different"
JWT_EXPIRATION="30m"
JWT_REFRESH_EXPIRATION="7d"
PORT=3001
```

---

## 2. Ordre de generation obligatoire (methodologie incrementale)

### Phase 1 : Schema et base de donnees

```
1. Creer apps/api/prisma/schema.prisma
2. Creer apps/api/.env avec DATABASE_URL
3. Copier : cp apps/api/.env apps/api/prisma/.env
4. Executer : cd apps/api && npx prisma generate
5. VERIFIER : Le dossier node_modules/.prisma/client/ doit contenir index.js
6. Si erreur → corriger le schema et recommencer
```

**IMPORTANT** : Ne JAMAIS passer a la phase 2 si `npx prisma generate` echoue.

### Phase 2 : Backend NestJS (module par module)

```
Pour chaque module (auth, users, products, categories, cart, orders, payments, upload, admin) :
  1. Creer le module (*.module.ts, *.controller.ts, *.service.ts, *.dto.ts)
  2. Enregistrer le module dans app.module.ts
  3. Executer : npx nest build
  4. VERIFIER : dist/main.js existe ET a une taille > 0
  5. Si erreur → corriger et reconstruire AVANT de passer au module suivant
```

**Ordre recommande des modules** :
1. `prisma/` (PrismaModule + PrismaService)
2. `auth/` (JWT strategy, guards)
3. `users/` (CRUD utilisateurs)
4. `categories/` (CRUD categories)
5. `products/` (CRUD produits avec relations)
6. `upload/` (gestion images)
7. `cart/` (panier)
8. `orders/` (commandes)
9. `payments/` (integration paiement)
10. `admin/` (endpoints admin)

### Phase 3 : Frontend Next.js (page par page)

```
Pour chaque page/composant :
  1. Creer le fichier (page.tsx, composant)
  2. Executer : npm run build
  3. VERIFIER : Le dossier .next/ existe et contient des fichiers
  4. Si erreur → corriger et reconstruire
```

**Ordre recommande** :
1. Layout principal (layout.tsx, Header, Footer)
2. Page d'accueil
3. Catalogue produits
4. Detail produit
5. Panier
6. Authentification (login, register)
7. Checkout
8. Espace client
9. Panel admin

### Phase 4 : Integration et tests

```
1. Lancer le backend : cd apps/api && node dist/main.js
2. Verifier le health check : curl http://localhost:3001/health
3. Lancer le frontend : cd apps/web && npm run start
4. Verifier la page d'accueil : curl http://localhost:3000
5. Creer le fichier READY avec "OK" si tout fonctionne
```

---

## 3. Checklist de verification du build

Avant de declarer le projet termine, verifier CHAQUE point :

### Backend (apps/api/)

- [ ] `npx prisma generate` s'execute sans erreur
- [ ] `npx nest build` s'execute sans erreur
- [ ] `dist/main.js` existe et a une taille > 0
- [ ] `node dist/main.js` demarre sans crash (tester 5 secondes)
- [ ] `GET /health` repond `{ "status": "ok" }`

### Frontend (apps/web/)

- [ ] `next.config.js` est en `.js` (PAS `.ts`)
- [ ] `tailwind.config.js` est en `.js` (PAS `.ts`)
- [ ] `postcss.config.js` est en `.js` (PAS `.ts`)
- [ ] `npm run build` s'execute sans erreur
- [ ] Le dossier `.next/` contient des fichiers

### Configuration

- [ ] `.env` existe dans `apps/api/`
- [ ] `.env` est copie dans `apps/api/prisma/`
- [ ] `DATABASE_URL` est defini et valide
- [ ] Les ports sont corrects (API et Frontend)

---

## 4. Pieges courants et solutions

### Piege 1 : Prisma client non genere

**Symptome** : ~56 erreurs TypeScript `Cannot find module '@prisma/client'`
**Cause** : `npx prisma generate` n'a pas ete execute apres `npm install`
**Solution** :
```bash
cd apps/api && npx prisma generate
```

### Piege 2 : next.config.ts au lieu de .js

**Symptome** : Build Next.js echoue ou le fichier de config est ignore
**Cause** : Next.js 14 ne supporte pas les fichiers de config TypeScript
**Solution** :
```bash
mv apps/web/next.config.ts apps/web/next.config.js
# Remplacer export default par module.exports =
```

### Piege 3 : DATABASE_URL introuvable par Prisma

**Symptome** : `Error: Environment variable not found: DATABASE_URL`
**Cause** : Le `.env` n'est pas dans le repertoire de `schema.prisma`
**Solution** :
```bash
cp apps/api/.env apps/api/prisma/.env
```

### Piege 4 : nest build produit un dist/ vide

**Symptome** : `dist/` existe mais `main.js` est absent
**Cause** : Erreurs TypeScript silencieuses ou mauvaise config tsconfig
**Solution** :
```bash
# Verifier la config
cat apps/api/tsconfig.build.json
# Doit contenir : "outDir": "./dist"
# Forcer le build avec logs
npx nest build --webpack 2>&1 | tee build.log
```

### Piege 5 : PM2 demarre avant la fin du build

**Symptome** : L'application crash immediatement avec `MODULE_NOT_FOUND`
**Cause** : Le script de deploiement lance PM2 sans verifier que le build est termine
**Solution** : Toujours verifier `dist/main.js` avant de lancer PM2

### Piege 6 : Import circulaire dans NestJS

**Symptome** : `Error: A circular dependency has been detected`
**Cause** : Deux modules s'importent mutuellement
**Solution** : Utiliser `forwardRef(() => ModuleName)` dans les imports

### Piege 7 : Tailwind ne genere pas de styles

**Symptome** : Le site s'affiche sans aucun style CSS
**Cause** : `content` dans `tailwind.config.js` ne couvre pas les bons fichiers
**Solution** :
```javascript
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  // ...
}
```

---

## 5. Structure de fichiers requise

```
project/
├── apps/
│   ├── api/
│   │   ├── src/
│   │   │   ├── modules/          # Un dossier par module
│   │   │   ├── common/           # Guards, filters, pipes, decorators
│   │   │   ├── prisma/           # PrismaModule + PrismaService
│   │   │   ├── app.module.ts     # Module racine
│   │   │   └── main.ts           # Point d'entree
│   │   ├── prisma/
│   │   │   ├── schema.prisma
│   │   │   ├── seed.ts
│   │   │   └── .env              # COPIE du .env principal
│   │   ├── uploads/              # Dossier images
│   │   ├── .env                  # Variables d'environnement
│   │   ├── nest-cli.json
│   │   ├── tsconfig.json
│   │   ├── tsconfig.build.json
│   │   └── package.json
│   │
│   └── web/
│       ├── app/                  # App Router Next.js
│       ├── components/           # Composants React
│       ├── lib/                  # Utilitaires, API client
│       ├── stores/               # Zustand stores
│       ├── types/                # Types TypeScript
│       ├── public/               # Assets statiques
│       ├── next.config.js        # OBLIGATOIREMENT .js
│       ├── tailwind.config.js    # OBLIGATOIREMENT .js
│       ├── postcss.config.js     # OBLIGATOIREMENT .js
│       ├── tsconfig.json
│       └── package.json
│
├── packages/
│   └── shared/                   # Types et utilitaires partages
│
├── turbo.json
├── package.json
└── .gitignore
```

---

## 6. Conventions pour les variables d'environnement

| Variable | Ou | Obligatoire | Description |
|---|---|---|---|
| `DATABASE_URL` | `apps/api/.env` | Oui | URL PostgreSQL |
| `JWT_SECRET` | `apps/api/.env` | Oui | Secret JWT (min 32 caracteres) |
| `JWT_REFRESH_SECRET` | `apps/api/.env` | Oui | Secret refresh token (different de JWT_SECRET) |
| `JWT_EXPIRATION` | `apps/api/.env` | Oui | Duree access token (ex: `30m`) |
| `JWT_REFRESH_EXPIRATION` | `apps/api/.env` | Oui | Duree refresh token (ex: `7d`) |
| `PORT` | `apps/api/.env` | Oui | Port du backend |
| `UPLOAD_DIR` | `apps/api/.env` | Non | Dossier uploads (defaut: `./uploads`) |
| `NEXT_PUBLIC_API_URL` | `apps/web/.env.local` | Oui | URL de l'API pour le frontend |
| `NEXT_PUBLIC_SITE_URL` | `apps/web/.env.local` | Oui | URL du site |
| `NEXT_PUBLIC_SITE_NAME` | `apps/web/.env.local` | Oui | Nom affiche du site |

---

## 7. Endpoint de health check (obligatoire)

Chaque projet doit exposer un endpoint de health check pour la verification automatisee :

### Backend (NestJS)

```typescript
// src/health/health.controller.ts
@Controller('health')
export class HealthController {
  @Get()
  check() {
    return {
      status: 'ok',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
    };
  }
}
```

### Frontend (Next.js)

```typescript
// app/api/health/route.ts
export async function GET() {
  return Response.json({
    status: 'ok',
    timestamp: new Date().toISOString(),
  });
}
```

Ces endpoints sont utilises par le script de deploiement pour verifier que l'application est operationnelle apres le demarrage PM2.
