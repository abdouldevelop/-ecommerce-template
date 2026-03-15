# Template Projet E-Commerce

## Qu'est-ce que c'est ?

Ce template est un kit complet de gestion de projet pour la création d'une plateforme e-commerce. Il est basé sur l'expérience concrète de la construction de la plateforme **Atypick Groupe** (e-commerce Côte d'Ivoire) et couvre toutes les phases du projet, de la découverte client jusqu'à la mise en production.

Chaque fichier est conçu pour être **immédiatement utilisable** par un agent IA ou un développeur humain, avec des instructions claires, des checklists actionnables et des spécifications techniques détaillées.

## Stack technique recommandée

| Composant | Technologie |
|-----------|-------------|
| Backend | NestJS (Node.js) |
| Frontend | Next.js 15 (App Router) |
| Base de données | PostgreSQL 16 |
| ORM | Prisma 6+ |
| Style | Tailwind CSS 4 |
| Monorepo | Turborepo |
| Paiement | PaiementPro / CinetPay |
| Déploiement | Docker + PM2 + Nginx |

## Comment utiliser ce template (Workflow Agent IA)

### Etape 1 : Découverte client (01-DISCOVERY)

1. Ouvrir `questionnaire-client.md` et poser toutes les questions au client
2. Remplir `cahier-des-charges.md` avec les réponses obtenues
3. Valider chaque point de `checklist-validation.md` avant de démarrer le dev

### Etape 2 : Architecture (02-ARCHITECTURE)

1. **IMPORTANT** : Consulter `niveaux-disponibilite.md` pour choisir le bon niveau d'infrastructure (1 à 4) selon le budget et les besoins du client
2. Confirmer ou adapter la stack dans `stack-technique.md`
3. Adapter le schéma de base de données dans `schema-base-donnees.md`
4. Référencer les endpoints API dans `api-endpoints.md`
5. Si Niveau 3+, suivre le guide `haute-disponibilite.md`

### Etape 3 : Répartition des tâches (03-EQUIPES)

1. Assigner les tâches backend depuis `equipe-backend.md`
2. Assigner les tâches frontend depuis `equipe-frontend.md`
3. Planifier l'infrastructure avec `equipe-infra.md`
4. Intégrer les exigences de sécurité depuis `equipe-securite.md`

### Etape 4 : Développement (04-FONCTIONNALITES)

Chaque fichier de `04-FONCTIONNALITES/` est une spécification fonctionnelle complète. Les développer dans cet ordre de priorité :

1. `auth-utilisateurs.md` - Authentification (P0)
2. `catalogue-produits.md` - Produits et catégories (P0)
3. `panier-commande.md` - Panier et commandes (P0)
4. `paiement-integration.md` - Intégration paiement (P0)
5. `espace-client.md` - Espace client (P1)
6. `admin-dashboard.md` - Panel admin (P1)
7. `recherche-filtres.md` - Recherche et filtres (P1)
8. `design-ui.md` - Design system (P1)

### Etape 5 : Suivi (05-SUIVI)

1. Utiliser `kanban.md` pour suivre l'avancement
2. Logger les décisions dans `journal-decisions.md`
3. Mettre à jour `changelog.md` à chaque livraison

### Etape 6 : Livraison (06-LIVRAISON)

1. Passer la `checklist-production.md` point par point
2. Effectuer l'audit de sécurité avec `securite-audit.md`
3. Rédiger la documentation client avec `documentation-client.md`

### Etape 7 : Automatisation Claude Code (07-CLAUDE-CODE)

1. Configurer le `CLAUDE.md` du projet en partant de `claude-md-template.md`
2. Copier `settings-template.json` dans `.claude/settings.json` du projet cible
3. Suivre le `guide-automatisation.md` pour lancer la generation automatisee en 5 phases :
   - Phase 1 : Setup (CLAUDE.md + .env + structure)
   - Phase 2 : Generation backend avec builds incrementaux
   - Phase 3 : Generation frontend avec builds incrementaux
   - Phase 4 : Verification automatique + auto-fix + retry
   - Phase 5 : Deploiement

## Structure du répertoire

```
ecommerce-template/
│
├── README.md                              # Ce fichier
│
├── 01-DISCOVERY/                          # Phase découverte
│   ├── questionnaire-client.md            # 50+ questions à poser au client
│   ├── cahier-des-charges.md              # Template CDC à remplir
│   └── checklist-validation.md            # Checklist avant développement
│
├── 02-ARCHITECTURE/                       # Architecture technique
│   ├── stack-technique.md                 # Stack recommandée + justifications
│   ├── schema-base-donnees.md             # Modèles de données complets (Prisma)
│   └── api-endpoints.md                   # Tous les endpoints API
│
├── 03-EQUIPES/                            # Répartition des tâches
│   ├── equipe-backend.md                  # Tâches backend (P0→P3)
│   ├── equipe-frontend.md                 # Tâches frontend (P0→P3)
│   ├── equipe-infra.md                    # Tâches infrastructure
│   └── equipe-securite.md                 # Audit sécurité (69 points)
│
├── 04-FONCTIONNALITES/                    # Spécifications fonctionnelles
│   ├── auth-utilisateurs.md               # Auth JWT + refresh + rôles
│   ├── catalogue-produits.md              # Produits, catégories, images
│   ├── panier-commande.md                 # Panier, commandes, statuts
│   ├── paiement-integration.md            # PaiementPro / CinetPay
│   ├── espace-client.md                   # Dashboard client
│   ├── admin-dashboard.md                 # Panel administration
│   ├── recherche-filtres.md               # Recherche autocomplete + filtres
│   └── design-ui.md                       # Design system complet
│
├── 05-SUIVI/                              # Suivi de projet
│   ├── kanban.md                          # Board kanban pré-rempli
│   ├── journal-decisions.md               # Log des décisions
│   └── changelog.md                       # Historique des versions
│
├── 06-LIVRAISON/                          # Mise en production
│   ├── checklist-production.md            # 50+ vérifications avant go-live
│   ├── securite-audit.md                  # Audit sécurité complet
│   └── documentation-client.md            # Documentation de remise client
│
└── 07-CLAUDE-CODE/                        # Automatisation Claude Code
    ├── guide-automatisation.md            # Guide complet (CLI, hooks, patterns, tokens, erreurs)
    ├── claude-md-template.md              # Template CLAUDE.md pour projets générés
    └── settings-template.json             # Configuration .claude/settings.json
```

## Conventions

- **Priorités** : P0 (bloquant/MVP), P1 (important), P2 (souhaitable), P3 (futur)
- **Complexité** : S (< 2h), M (2-4h), L (4-8h), XL (> 8h)
- **Statuts tâche** : BACKLOG, TODO, EN COURS, REVIEW, FAIT
- **Statuts commande** : PENDING → CONFIRMED → PROCESSING → SHIPPED → DELIVERED
- **Placeholders** : `[A REMPLIR]` indique un champ à compléter avec les infos du client

## Notes importantes

- Ce template est optimisé pour le marché **Afrique francophone** (paiement mobile, livraison locale)
- Les intégrations de paiement couvrent : Carte bancaire, Orange Money, MTN MoMo, Wave, Moov Flooz
- Le template suppose un déploiement sur **VPS Linux** (pas de serverless)
- Les estimations de temps sont basées sur un développeur expérimenté travaillant seul
