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
| **Total tâches** | 89 |
| **P0 (MVP)** | 48 |
| **P1 (Important)** | 30 |
| **P2 (Souhaitable)** | 11 |
| **Estimation P0** | ~136h |
| **Estimation P0+P1** | ~200h |
| **BACKLOG** | 89 |
| **TODO** | 0 |
| **EN COURS** | 0 |
| **REVIEW** | 0 |
| **FAIT** | 0 |
| **Progression** | 0% |

---

## Velocity tracking (par sprint)

| Sprint | Dates | Tâches prévues | Tâches terminées | Heures estimées | Heures réelles |
|---|---|---|---|---|---|
| Sprint 1 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| Sprint 2 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| Sprint 3 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| Sprint 4 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| Sprint 5 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
