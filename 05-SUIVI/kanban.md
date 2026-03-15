# Board Kanban - Suivi du Projet E-Commerce

## Instructions pour l'agent IA

Ce kanban est pré-rempli avec toutes les tâches des fichiers `equipe-*.md`. Mettre à jour le statut de chaque tâche au fur et à mesure de l'avancement. Déplacer les tâches entre les colonnes.

**Colonnes** : BACKLOG → TODO → EN COURS → REVIEW → FAIT

---

## BACKLOG (pas encore planifié)

### Backend P2
| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| BE-019 | CinetPay (alternative) | P2 | L | - |
| BE-027 | Rate limiting | P2 | S | - |
| BE-028 | Logging structuré | P2 | M | - |
| BE-029 | Cache requêtes | P2 | M | - |
| BE-030 | Notifications email | P2 | L | - |

### Frontend P2
| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| FE-010 | Bannière promo | P2 | S | - |

### Infra P2
| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| INFRA-018 | CI GitHub Actions | P2 | L | - |
| INFRA-019 | CD automatique | P2 | L | - |

### Tests de charge P2
| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| TEST-045 | Charge - Pool DB | P2 | M | - |

---

## TODO (planifié, prêt à démarrer)

### Sprint 1 : Setup et Auth (P0)

| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| BE-001 | Setup NestJS | P0 | M | - |
| BE-002 | Schéma Prisma | P0 | L | - |
| BE-003 | Service Prisma | P0 | S | - |
| BE-004 | Auth Register | P0 | M | - |
| BE-005 | Auth Login | P0 | M | - |
| BE-006 | JWT Guard | P0 | M | - |
| BE-007 | Token Refresh | P0 | M | - |
| BE-008 | Logout + Change Password | P0 | S | - |
| FE-001 | Setup Next.js | P0 | M | - |
| FE-002 | Stores Zustand | P0 | M | - |
| FE-003 | Client API | P0 | M | - |
| FE-021 | Page connexion | P0 | M | - |
| FE-022 | Page inscription | P0 | M | - |
| FE-036 | Toast | P0 | S | - |
| FE-037 | Modale confirmation | P0 | S | - |
| FE-038 | Skeleton loaders | P0 | S | - |

### Sprint 2 : Produits et Layout (P0)

| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| BE-009 | CRUD Produits | P0 | L | - |
| BE-010 | Recherche autocomplete | P0 | M | - |
| BE-011 | Images produits | P0 | M | - |
| BE-012 | CRUD Catégories | P0 | M | - |
| FE-004 | Header/Navbar | P0 | L | - |
| FE-005 | Footer | P0 | M | - |
| FE-006 | Menu mobile | P0 | M | - |
| FE-007 | Slider | P0 | M | - |
| FE-008 | Grille produits vedettes | P0 | M | - |
| FE-011 | Page catalogue | P0 | L | - |
| FE-012 | Carte produit | P0 | M | - |
| FE-013 | Page détail produit | P0 | L | - |

### Sprint 3 : Panier, Commandes, Paiement (P0)

| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| BE-013 | Panier serveur | P0 | L | - |
| BE-015 | Création commande | P0 | XL | - |
| BE-016 | Gestion commandes | P0 | M | - |
| BE-017 | Numéro commande | P0 | S | - |
| BE-018 | PaiementPro | P0 | XL | - |
| FE-014 | Panier drawer | P0 | L | - |
| FE-015 | Page panier | P0 | M | - |
| FE-016 | Page checkout | P0 | XL | - |
| FE-017 | Page résultat paiement | P0 | M | - |
| FE-018 | Page paiement | P0 | M | - |

### Sprint 4 : Espace client et Admin (P1)

| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| BE-014 | Sync panier | P1 | M | - |
| BE-020 | Dashboard stats | P1 | L | - |
| BE-021 | Clients admin | P1 | M | - |
| BE-022 | Paramètres | P1 | S | - |
| BE-023 | Adresses | P1 | M | - |
| BE-024 | Profil utilisateur | P1 | S | - |
| BE-025 | Liste utilisateurs | P1 | S | - |
| BE-026 | Health check | P1 | S | - |
| FE-009 | Section catégories accueil | P1 | M | - |
| FE-019 | Recherche autocomplete | P1 | L | - |
| FE-020 | Filtre catégories | P1 | M | - |
| FE-023 | Dashboard client | P1 | M | - |
| FE-024 | Commandes client | P1 | L | - |
| FE-025 | Profil client | P1 | M | - |
| FE-026 | Layout admin | P1 | M | - |
| FE-027 | Dashboard admin | P1 | L | - |
| FE-028 | Produits admin | P1 | XL | - |
| FE-029 | Catégories admin | P1 | M | - |
| FE-030 | Commandes admin | P1 | L | - |
| FE-031 | Clients admin | P1 | M | - |
| FE-032 | Page A propos | P1 | S | - |
| FE-033 | Pages légales | P1 | S | - |
| FE-034 | Page contact | P1 | S | - |
| FE-035 | Page 404 | P1 | S | - |

### Sprint 6 : Tests unitaires et E2E (P0)

| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| TEST-001 | Setup tests backend | P0 | M | - |
| TEST-002 | Auth - Inscription | P0 | M | - |
| TEST-003 | Auth - Connexion | P0 | M | - |
| TEST-004 | Auth - Token Refresh | P0 | M | - |
| TEST-005 | Auth - Logout/Password | P0 | S | - |
| TEST-006 | JWT Guards | P0 | M | - |
| TEST-007 | Products - Lecture | P0 | M | - |
| TEST-008 | Products - Ecriture | P0 | L | - |
| TEST-009 | Orders - Creation | P0 | XL | - |
| TEST-010 | Orders - Gestion | P0 | M | - |
| TEST-011 | Payments - Initiation | P0 | L | - |
| TEST-012 | Payments - Callback | P0 | L | - |
| TEST-019 | Setup tests frontend | P0 | M | - |
| TEST-020 | Composants UI de base | P0 | M | - |
| TEST-021 | ProductCard | P0 | M | - |
| TEST-026 | Store Auth | P0 | M | - |
| TEST-027 | Store Cart | P0 | M | - |
| TEST-029 | Client Axios | P0 | M | - |
| TEST-031 | Setup Playwright | P0 | M | - |
| TEST-032 | E2E - Auth | P0 | M | - |
| TEST-033 | E2E - Produits | P0 | L | - |
| TEST-034 | E2E - Panier | P0 | M | - |
| TEST-035 | E2E - Checkout | P0 | XL | - |
| TEST-036 | E2E - Paiement | P0 | L | - |
| TEST-047 | Injection SQL | P0 | M | - |
| TEST-048 | XSS | P0 | M | - |
| TEST-049 | Auth bypass | P0 | L | - |

### Sprint 7 : Tests complementaires (P1)

| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| TEST-013 | Categories | P1 | M | - |
| TEST-014 | Cart | P1 | M | - |
| TEST-015 | Addresses | P1 | S | - |
| TEST-016 | Users | P1 | S | - |
| TEST-017 | Settings | P1 | S | - |
| TEST-018 | Dashboard | P1 | M | - |
| TEST-022 | Header | P1 | M | - |
| TEST-023 | Pages Auth | P1 | M | - |
| TEST-024 | Pages Produits | P1 | L | - |
| TEST-025 | Panier | P1 | M | - |
| TEST-028 | Store Search | P1 | S | - |
| TEST-030 | Fonctions utilitaires | P1 | S | - |
| TEST-037 | E2E - Admin Produits | P1 | XL | - |
| TEST-038 | E2E - Admin Commandes | P1 | L | - |
| TEST-039 | E2E - Admin Categories | P1 | M | - |
| TEST-040 | E2E - Mobile responsive | P1 | L | - |
| TEST-041 | Setup k6 | P1 | S | - |
| TEST-042 | Charge - Page d'accueil | P1 | M | - |
| TEST-043 | Charge - API | P1 | M | - |
| TEST-044 | Stress - Checkout | P1 | L | - |
| TEST-046 | Scan OWASP ZAP | P1 | L | - |
| TEST-050 | Rate limiting | P1 | M | - |
| TEST-051 | Upload securite | P1 | M | - |
| TEST-052 | HTTPS/Headers | P1 | S | - |

### Sprint 5 : Infrastructure (P0-P1)

| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| INFRA-001 | Docker Compose dev | P0 | M | - |
| INFRA-002 | Dockerfile API | P0 | M | - |
| INFRA-003 | Dockerfile Web | P0 | M | - |
| INFRA-004 | Config serveur | P0 | L | - |
| INFRA-005 | Dépendances serveur | P0 | M | - |
| INFRA-006 | Config PostgreSQL | P0 | M | - |
| INFRA-007 | Config Nginx | P0 | L | - |
| INFRA-008 | SSL Let's Encrypt | P0 | S | - |
| INFRA-009 | Config PM2 | P0 | M | - |
| INFRA-010 | Script déploiement | P0 | M | - |
| INFRA-011 | Config DNS | P0 | S | - |
| INFRA-012 | Backup BDD | P0 | M | - |
| INFRA-013 | Backup fichiers | P1 | S | - |
| INFRA-014 | Monitoring applicatif | P1 | M | - |
| INFRA-015 | Monitoring logs | P1 | S | - |
| INFRA-016 | Durcissement serveur | P0 | M | - |
| INFRA-017 | Headers sécurité | P0 | S | - |
| INFRA-020 | Opti Nginx | P1 | S | - |
| INFRA-021 | Opti Node.js | P1 | S | - |

### Sprint 8 : Automatisation Claude Code (P1)

| ID | Tâche | Priorité | Complexité | Assigné |
|---|---|---|---|---|
| AUTO-001 | Configurer CLAUDE.md du projet | P1 | S | - |
| AUTO-002 | Configurer .claude/settings.json (permissions, hooks) | P1 | S | - |
| AUTO-003 | Creer scripts d'automatisation Phase 1 (setup) | P1 | M | - |
| AUTO-004 | Creer scripts d'automatisation Phase 2 (backend) | P1 | L | - |
| AUTO-005 | Creer scripts d'automatisation Phase 3 (frontend) | P1 | L | - |
| AUTO-006 | Creer scripts d'automatisation Phase 4 (verify + auto-fix) | P1 | M | - |
| AUTO-007 | Creer scripts d'automatisation Phase 5 (deploy) | P1 | M | - |
| AUTO-008 | Configurer hooks PreToolUse (protection fichiers sensibles) | P1 | S | - |
| AUTO-009 | Configurer hooks PostToolUse (auto-format, prisma generate) | P1 | S | - |
| AUTO-010 | Configurer hook SessionStart (contexte post-compaction) | P1 | S | - |
| AUTO-011 | Tester le pipeline complet sur un projet test | P1 | XL | - |
| AUTO-012 | Documenter les couts reels par phase | P1 | S | - |

---

## EN COURS (en développement actif)

| ID | Tâche | Priorité | Démarré le | Assigné | Notes |
|---|---|---|---|---|---|
| - | - | - | - | - | - |

---

## REVIEW (en attente de validation/test)

| ID | Tâche | Priorité | Terminé le | Assigné | Notes |
|---|---|---|---|---|---|
| - | - | - | - | - | - |

---

## FAIT (terminé et validé)

| ID | Tâche | Priorité | Terminé le | Assigné | Notes |
|---|---|---|---|---|---|
| - | - | - | - | - | - |

---

## Statistiques

| Métrique | Valeur |
|---|---|
| **Total tâches** | 153 |
| **P0 (MVP)** | 75 |
| **P1 (Important)** | 66 |
| **P2 (Souhaitable)** | 12 |
| **Estimation P0** | ~200h |
| **Estimation P0+P1** | ~356h |
| **BACKLOG** | 153 |
| **TODO** | 0 |
| **EN COURS** | 0 |
| **REVIEW** | 0 |
| **FAIT** | 0 |
| **Progression** | 0% |

### Repartition par equipe

| Equipe | Tâches | Estimation |
|---|---|---|
| Backend | 30 | ~82h |
| Frontend | 38 | ~98h |
| Infrastructure | 21 | ~54h |
| Securite | 69 points d'audit | - |
| Tests | 52 | ~132h |
| **Automatisation Claude Code** | **12** | **~24h** |

---

## Velocity tracking (par sprint)

| Sprint | Dates | Tâches prévues | Tâches terminées | Heures estimées | Heures réelles |
|---|---|---|---|---|---|
| Sprint 1 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| Sprint 2 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| Sprint 3 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| Sprint 4 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| Sprint 5 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| Sprint 6 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| Sprint 7 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
