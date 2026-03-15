# Equipe Tests - Taches et Priorites

## Instructions pour l'agent IA

Ce document liste toutes les taches de tests de la plateforme e-commerce : tests unitaires backend et frontend, tests end-to-end (E2E), tests de charge et tests de securite. Chaque tache a un ID unique (TEST-XXX), une priorite, une complexite et des criteres d'acceptation detailles.

**Legende priorites** : P0 (bloquant/MVP), P1 (important), P2 (souhaitable), P3 (futur)
**Legende complexite** : S (< 2h), M (2-4h), L (4-8h), XL (> 8h)

**Objectifs de couverture** :
- Backend : 80%+ de couverture de code
- Frontend : 70%+ de couverture de code
- E2E : 100% des parcours critiques couverts

**Stack de tests** :
- Backend : Jest + Supertest + Prisma (mock)
- Frontend : Jest + React Testing Library + MSW (Mock Service Worker)
- E2E : Playwright
- Charge : k6
- Securite : OWASP ZAP + scripts personnalises

---

## Section 1 : Tests unitaires backend (Jest + Supertest)

### Module 1.1 : Setup et configuration

#### TEST-001 : Configuration de l'environnement de test backend
- **Priorite** : P0
- **Complexite** : M
- **Description** : Configurer Jest, la base de test PostgreSQL et les mocks Prisma pour les tests unitaires du backend NestJS
- **Criteres d'acceptation** :
  - [ ] `jest.config.ts` configure dans `apps/api/`
  - [ ] Base de donnees de test PostgreSQL isolee (DATABASE_URL de test)
  - [ ] Prisma mock configure (prisma-mock ou @quramy/jest-prisma)
  - [ ] Variables d'environnement de test dans `.env.test`
  - [ ] Script npm : `npm run test` lance les tests
  - [ ] Script npm : `npm run test:cov` genere le rapport de couverture
  - [ ] Rapport de couverture en HTML genere dans `coverage/`
  - [ ] Les tests sont isoles (pas d'effets de bord entre tests)

---

### Module 1.2 : Tests du service Auth

#### TEST-002 : Tests AuthService - Inscription
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester le service d'inscription des utilisateurs
- **Criteres d'acceptation** :
  - [ ] Test : inscription reussie avec donnees valides
  - [ ] Test : retourne un accessToken et un refreshToken
  - [ ] Test : le mot de passe est hache (bcrypt)
  - [ ] Test : erreur 400 si email deja utilise
  - [ ] Test : erreur 400 si mot de passe trop faible (< 8 chars)
  - [ ] Test : erreur 400 si mot de passe sans majuscule
  - [ ] Test : erreur 400 si mot de passe sans chiffre
  - [ ] Test : erreur 400 si email invalide
  - [ ] Test : erreur 400 si prenom manquant
  - [ ] Couverture : > 95% du service register

#### TEST-003 : Tests AuthService - Connexion
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester le service de connexion
- **Criteres d'acceptation** :
  - [ ] Test : connexion reussie avec identifiants valides
  - [ ] Test : retourne accessToken + refreshToken + user (sans password)
  - [ ] Test : met a jour lastLoginAt
  - [ ] Test : erreur 401 si email incorrect
  - [ ] Test : erreur 401 si mot de passe incorrect
  - [ ] Test : erreur 403 si compte desactive (isActive = false)
  - [ ] Test : le message d'erreur est generique (pas de fuite d'information)
  - [ ] Couverture : > 95% du service login

#### TEST-004 : Tests AuthService - Token Refresh
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester le mecanisme de rafraichissement des tokens
- **Criteres d'acceptation** :
  - [ ] Test : refresh reussi avec un refreshToken valide
  - [ ] Test : retourne un nouveau couple accessToken + refreshToken
  - [ ] Test : l'ancien refreshToken est invalide (rotation)
  - [ ] Test : erreur 401 si refreshToken expire
  - [ ] Test : erreur 401 si refreshToken invalide (signature)
  - [ ] Test : erreur 401 si refreshToken ne correspond pas a celui en base

#### TEST-005 : Tests AuthService - Logout et Change Password
- **Priorite** : P0
- **Complexite** : S
- **Description** : Tester la deconnexion et le changement de mot de passe
- **Criteres d'acceptation** :
  - [ ] Test : logout invalide le refreshToken en base
  - [ ] Test : change-password verifie l'ancien mot de passe
  - [ ] Test : change-password met a jour le hash
  - [ ] Test : change-password invalide tous les refreshTokens
  - [ ] Test : erreur 400 si ancien mot de passe incorrect
  - [ ] Test : erreur 400 si nouveau mot de passe trop faible

#### TEST-006 : Tests JWT Guards et Decorateurs
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester les guards d'authentification et d'autorisation
- **Criteres d'acceptation** :
  - [ ] Test : JwtAuthGuard accepte un token valide
  - [ ] Test : JwtAuthGuard rejette un token expire (401)
  - [ ] Test : JwtAuthGuard rejette un token invalide (401)
  - [ ] Test : RolesGuard accepte un utilisateur avec le bon role
  - [ ] Test : RolesGuard rejette un utilisateur sans le bon role (403)
  - [ ] Test : @CurrentUser() extrait correctement l'utilisateur du token
  - [ ] Test : @Public() bypass l'authentification

---

### Module 1.3 : Tests du service Products

#### TEST-007 : Tests ProductsService - Lecture
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester les endpoints de lecture des produits
- **Criteres d'acceptation** :
  - [ ] Test : GET /products retourne une liste paginee
  - [ ] Test : pagination correcte (page, limit, total, totalPages)
  - [ ] Test : filtre par categorie fonctionne
  - [ ] Test : filtre par prix min/max fonctionne
  - [ ] Test : filtre par featured fonctionne
  - [ ] Test : tri par prix (asc/desc) fonctionne
  - [ ] Test : tri par date (asc/desc) fonctionne
  - [ ] Test : recherche par nom (ILIKE) fonctionne
  - [ ] Test : seuls les produits ACTIVE sont retournes (public)
  - [ ] Test : GET /products/:slug retourne le produit avec images
  - [ ] Test : GET /products/:slug incremente viewCount
  - [ ] Test : erreur 404 si slug inexistant
  - [ ] Test : GET /products/featured retourne les produits vedettes
  - [ ] Test : GET /products/search retourne max 5-8 resultats

#### TEST-008 : Tests ProductsService - Ecriture (Admin)
- **Priorite** : P0
- **Complexite** : L
- **Description** : Tester les operations CRUD admin sur les produits
- **Criteres d'acceptation** :
  - [ ] Test : creation de produit avec donnees valides
  - [ ] Test : le slug est genere automatiquement depuis le nom
  - [ ] Test : slug unique (suffixe numerique si doublon)
  - [ ] Test : modification partielle (PATCH) fonctionne
  - [ ] Test : suppression fonctionne (ou archivage si commandes existantes)
  - [ ] Test : erreur 400 si donnees invalides (prix negatif, nom vide)
  - [ ] Test : erreur 403 si utilisateur non-admin
  - [ ] Test : GET /products/admin/all inclut DRAFT et ARCHIVED

---

### Module 1.4 : Tests du service Orders

#### TEST-009 : Tests OrdersService - Creation de commande
- **Priorite** : P0
- **Complexite** : XL
- **Description** : Tester le processus complet de creation de commande
- **Criteres d'acceptation** :
  - [ ] Test : creation de commande depuis un panier non vide
  - [ ] Test : le numero de commande suit le format CMD-YYYYMMDD-XXXX
  - [ ] Test : le stock est decremente pour chaque article
  - [ ] Test : les donnees produit sont copiees dans OrderItem (nom, prix, image)
  - [ ] Test : le panier est vide apres creation
  - [ ] Test : un paiement PENDING est cree
  - [ ] Test : le total est calcule correctement (subtotal + shippingFee)
  - [ ] Test : erreur si panier vide
  - [ ] Test : erreur si stock insuffisant pour un article
  - [ ] Test : rollback complet en cas d'erreur (transaction)
  - [ ] Test : le numero de commande est unique
  - [ ] Test : le compteur journalier se reinitialise chaque jour

#### TEST-010 : Tests OrdersService - Gestion des commandes
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester la consultation et la gestion des commandes
- **Criteres d'acceptation** :
  - [ ] Test : GET /orders retourne les commandes de l'utilisateur connecte
  - [ ] Test : un utilisateur ne peut pas voir les commandes d'un autre
  - [ ] Test : un admin peut voir toutes les commandes
  - [ ] Test : GET /orders/:id retourne le detail avec items, adresse, paiement
  - [ ] Test : changement de statut valide fonctionne (PENDING -> CONFIRMED)
  - [ ] Test : changement de statut invalide est rejete (DELIVERED -> PENDING)
  - [ ] Test : seul un admin peut changer le statut
  - [ ] Test : filtre par statut fonctionne
  - [ ] Test : pagination fonctionne

---

### Module 1.5 : Tests du service Payments

#### TEST-011 : Tests PaymentsService - Initiation
- **Priorite** : P0
- **Complexite** : L
- **Description** : Tester l'initiation de paiement avec le prestataire
- **Criteres d'acceptation** :
  - [ ] Test : initiation reussie retourne une URL de paiement
  - [ ] Test : un transactionId unique est genere
  - [ ] Test : erreur si la commande n'appartient pas a l'utilisateur
  - [ ] Test : erreur si la commande n'est pas en PENDING
  - [ ] Test : erreur si le montant est zero ou negatif
  - [ ] Test : les channels de paiement sont valides (CARD, ORANGE_MONEY, etc.)
  - [ ] Test : appel HTTP au prestataire avec les bons parametres (mock)
  - [ ] Test : gestion d'erreur si le prestataire est indisponible

#### TEST-012 : Tests PaymentsService - Callback
- **Priorite** : P0
- **Complexite** : L
- **Description** : Tester le traitement des notifications de paiement (IPN)
- **Criteres d'acceptation** :
  - [ ] Test : callback valide met a jour le paiement en COMPLETED
  - [ ] Test : callback valide passe la commande en CONFIRMED
  - [ ] Test : verification du montant (montant paye = montant attendu)
  - [ ] Test : anti-replay (callback ignore si paiement deja COMPLETED)
  - [ ] Test : callback avec montant incorrect est rejete
  - [ ] Test : callback avec transaction inconnue est rejete
  - [ ] Test : logging des transactions pour audit

---

### Module 1.6 : Tests des services secondaires

#### TEST-013 : Tests CategoriesService
- **Priorite** : P1
- **Complexite** : M
- **Description** : Tester le CRUD des categories
- **Criteres d'acceptation** :
  - [ ] Test : liste des categories avec productCount
  - [ ] Test : detail d'une categorie avec produits pagines
  - [ ] Test : creation avec slug automatique
  - [ ] Test : modification
  - [ ] Test : suppression impossible si produits associes
  - [ ] Test : support des sous-categories (parentId)
  - [ ] Test : tri par sortOrder

#### TEST-014 : Tests CartService
- **Priorite** : P1
- **Complexite** : M
- **Description** : Tester la gestion du panier
- **Criteres d'acceptation** :
  - [ ] Test : ajout d'un produit au panier
  - [ ] Test : ajout d'un produit deja present incremente la quantite
  - [ ] Test : modification de la quantite
  - [ ] Test : suppression d'un article
  - [ ] Test : vidange du panier
  - [ ] Test : calcul correct du subtotal et total
  - [ ] Test : erreur si stock insuffisant
  - [ ] Test : synchronisation du panier (POST /cart/sync)

#### TEST-015 : Tests AddressesService
- **Priorite** : P1
- **Complexite** : S
- **Description** : Tester le CRUD des adresses
- **Criteres d'acceptation** :
  - [ ] Test : CRUD complet
  - [ ] Test : un seul isDefault par utilisateur
  - [ ] Test : verification de propriete
  - [ ] Test : suppression impossible si commande active associee

#### TEST-016 : Tests UsersService
- **Priorite** : P1
- **Complexite** : S
- **Description** : Tester la gestion du profil et des utilisateurs
- **Criteres d'acceptation** :
  - [ ] Test : GET /users/me retourne le profil
  - [ ] Test : PATCH /users/me modifie les champs autorises
  - [ ] Test : impossible de modifier email ou role
  - [ ] Test : admin peut lister les utilisateurs
  - [ ] Test : admin peut activer/desactiver un utilisateur

#### TEST-017 : Tests SettingsService
- **Priorite** : P1
- **Complexite** : S
- **Description** : Tester les parametres du site
- **Criteres d'acceptation** :
  - [ ] Test : GET /settings/public retourne les parametres publics
  - [ ] Test : GET /settings (admin) retourne tous les parametres
  - [ ] Test : PATCH /settings met a jour en batch
  - [ ] Test : erreur 403 si non-admin

#### TEST-018 : Tests DashboardService
- **Priorite** : P1
- **Complexite** : M
- **Description** : Tester les statistiques du dashboard admin
- **Criteres d'acceptation** :
  - [ ] Test : revenus calcules correctement (total, paye, en attente)
  - [ ] Test : croissance mensuelle calculee correctement
  - [ ] Test : comptage des commandes par statut
  - [ ] Test : comptage des produits (actifs, drafts, archives, stock bas)
  - [ ] Test : top 5 produits par quantite vendue
  - [ ] Test : 5 dernieres commandes retournees
  - [ ] Test : erreur 403 si non-admin

---

## Section 2 : Tests unitaires frontend (Jest + React Testing Library)

### Module 2.1 : Setup et configuration

#### TEST-019 : Configuration de l'environnement de test frontend
- **Priorite** : P0
- **Complexite** : M
- **Description** : Configurer Jest, React Testing Library et MSW pour les tests frontend
- **Criteres d'acceptation** :
  - [ ] `jest.config.ts` configure dans `apps/web/`
  - [ ] React Testing Library installe et configure
  - [ ] MSW (Mock Service Worker) configure pour mocker l'API
  - [ ] Handlers MSW pour tous les endpoints utilises par le frontend
  - [ ] `@testing-library/jest-dom` pour les matchers DOM
  - [ ] Support des modules CSS / Tailwind dans les tests
  - [ ] Script npm : `npm run test` et `npm run test:cov`

---

### Module 2.2 : Tests des composants

#### TEST-020 : Tests des composants UI de base
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester les composants UI reutilisables
- **Criteres d'acceptation** :
  - [ ] Test : Toast s'affiche avec le bon type (success, error, info, warning)
  - [ ] Test : Toast disparait apres le delai
  - [ ] Test : Modale de confirmation s'ouvre et se ferme
  - [ ] Test : Modale appelle onConfirm au clic sur Confirmer
  - [ ] Test : Modale appelle onCancel au clic sur Annuler
  - [ ] Test : Modale se ferme avec Escape
  - [ ] Test : Skeleton loader s'affiche avec les bonnes dimensions

#### TEST-021 : Tests du composant ProductCard
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester l'affichage des cartes produit
- **Criteres d'acceptation** :
  - [ ] Test : affiche le nom du produit
  - [ ] Test : affiche le prix formate ("15 000 FCFA")
  - [ ] Test : affiche l'ancien prix barre si comparePrice
  - [ ] Test : affiche le badge "Promo" si reduction
  - [ ] Test : affiche le pourcentage de reduction
  - [ ] Test : affiche le badge "Rupture" si stock = 0
  - [ ] Test : le lien pointe vers /products/{slug}
  - [ ] Test : l'image utilise next/image avec lazy loading
  - [ ] Test : le composant Skeleton s'affiche en mode loading

#### TEST-022 : Tests du composant Header
- **Priorite** : P1
- **Complexite** : M
- **Description** : Tester le header et la navigation
- **Criteres d'acceptation** :
  - [ ] Test : affiche le logo cliquable vers l'accueil
  - [ ] Test : affiche le badge panier avec le bon nombre d'articles
  - [ ] Test : affiche "Se connecter" si non connecte
  - [ ] Test : affiche "Mon compte" si connecte
  - [ ] Test : le menu mobile s'ouvre au clic sur le hamburger
  - [ ] Test : le menu mobile se ferme au clic sur l'overlay

#### TEST-023 : Tests des pages d'authentification
- **Priorite** : P1
- **Complexite** : M
- **Description** : Tester les pages de connexion et d'inscription
- **Criteres d'acceptation** :
  - [ ] Test : formulaire de connexion valide les champs
  - [ ] Test : connexion reussie redirige vers l'accueil
  - [ ] Test : connexion echouee affiche un message d'erreur
  - [ ] Test : formulaire d'inscription valide tous les champs
  - [ ] Test : indicateur de force du mot de passe fonctionne
  - [ ] Test : inscription reussie connecte automatiquement
  - [ ] Test : lien vers inscription depuis la page de connexion
  - [ ] Test : lien vers connexion depuis la page d'inscription

#### TEST-024 : Tests des pages produits
- **Priorite** : P1
- **Complexite** : L
- **Description** : Tester les pages catalogue et detail produit
- **Criteres d'acceptation** :
  - [ ] Test : le catalogue charge et affiche les produits
  - [ ] Test : la pagination fonctionne
  - [ ] Test : le filtre par categorie fonctionne
  - [ ] Test : le tri fonctionne
  - [ ] Test : le message "Aucun produit" s'affiche si resultat vide
  - [ ] Test : la page detail affiche toutes les informations du produit
  - [ ] Test : le selecteur de quantite respecte les limites de stock
  - [ ] Test : le bouton "Ajouter au panier" est desactive si rupture
  - [ ] Test : le bouton "Ajouter au panier" ajoute au store et affiche un toast

#### TEST-025 : Tests du panier (composants)
- **Priorite** : P1
- **Complexite** : M
- **Description** : Tester les composants du panier
- **Criteres d'acceptation** :
  - [ ] Test : le drawer s'ouvre avec les articles du store
  - [ ] Test : modification de quantite met a jour le store
  - [ ] Test : suppression d'article met a jour le store
  - [ ] Test : le sous-total et total sont calcules correctement
  - [ ] Test : le message "Panier vide" s'affiche si aucun article
  - [ ] Test : le bouton "Commander" redirige vers /checkout

---

### Module 2.3 : Tests des stores Zustand

#### TEST-026 : Tests du store Auth
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester le store de gestion d'authentification
- **Criteres d'acceptation** :
  - [ ] Test : etat initial (non connecte, pas de token)
  - [ ] Test : login() met a jour user, token, isAuthenticated
  - [ ] Test : logout() reinitialise l'etat
  - [ ] Test : le token est persiste dans localStorage
  - [ ] Test : hydratation depuis localStorage au chargement
  - [ ] Test : setUser() met a jour les donnees utilisateur

#### TEST-027 : Tests du store Cart
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester le store du panier
- **Criteres d'acceptation** :
  - [ ] Test : etat initial (panier vide)
  - [ ] Test : addItem() ajoute un article
  - [ ] Test : addItem() incremente la quantite si deja present
  - [ ] Test : removeItem() supprime un article
  - [ ] Test : updateQuantity() modifie la quantite
  - [ ] Test : clearCart() vide le panier
  - [ ] Test : total calcule correctement (somme des prix * quantite)
  - [ ] Test : itemsCount retourne le nombre total d'articles
  - [ ] Test : persistance dans localStorage (survit au refresh)

#### TEST-028 : Tests du store Search
- **Priorite** : P1
- **Complexite** : S
- **Description** : Tester le store de recherche
- **Criteres d'acceptation** :
  - [ ] Test : etat initial (query vide, results vide)
  - [ ] Test : setQuery() met a jour la query
  - [ ] Test : setResults() met a jour les resultats
  - [ ] Test : setCategory() met a jour la categorie selectionnee
  - [ ] Test : clearSearch() reinitialise l'etat

---

### Module 2.4 : Tests du client API

#### TEST-029 : Tests du client Axios
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester la configuration du client HTTP
- **Criteres d'acceptation** :
  - [ ] Test : l'intercepteur ajoute le header Authorization si token present
  - [ ] Test : l'intercepteur ne met pas de header si pas de token
  - [ ] Test : intercepteur 401 tente un refresh automatique
  - [ ] Test : si refresh reussit, la requete originale est rejouee
  - [ ] Test : si refresh echoue, l'utilisateur est deconnecte
  - [ ] Test : les erreurs reseau retournent un message en francais
  - [ ] Test : le timeout est respecte

#### TEST-030 : Tests des fonctions utilitaires
- **Priorite** : P1
- **Complexite** : S
- **Description** : Tester les fonctions utilitaires du frontend
- **Criteres d'acceptation** :
  - [ ] Test : formatPrice() formate correctement (15000 -> "15 000 FCFA")
  - [ ] Test : formatDate() formate en francais
  - [ ] Test : calculateDiscount() retourne le bon pourcentage
  - [ ] Test : truncateText() tronque avec "..."
  - [ ] Test : validateEmail() valide les formats d'email
  - [ ] Test : validatePassword() verifie la force du mot de passe
  - [ ] Test : slugify() genere un slug valide

---

## Section 3 : Tests End-to-End (Playwright)

### Module 3.1 : Setup

#### TEST-031 : Configuration Playwright
- **Priorite** : P0
- **Complexite** : M
- **Description** : Configurer Playwright pour les tests E2E
- **Criteres d'acceptation** :
  - [ ] `playwright.config.ts` configure
  - [ ] Navigateurs : Chromium + Firefox + WebKit (mobile Safari)
  - [ ] Viewports : Desktop (1280x720) + Mobile (375x667)
  - [ ] Base URL configurable via variable d'environnement
  - [ ] Screenshots et videos en cas d'echec
  - [ ] Serveur de dev demarre automatiquement avant les tests
  - [ ] Base de donnees de test seedee avant chaque suite
  - [ ] Script : `npm run test:e2e`

---

### Module 3.2 : Parcours utilisateur

#### TEST-032 : E2E - Inscription et connexion
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester le parcours complet d'inscription et de connexion
- **Criteres d'acceptation** :
  - [ ] Test : inscription avec donnees valides
  - [ ] Test : redirection vers l'accueil apres inscription
  - [ ] Test : deconnexion
  - [ ] Test : connexion avec les identifiants crees
  - [ ] Test : le header affiche "Mon compte" apres connexion
  - [ ] Test : inscription avec email deja utilise affiche une erreur
  - [ ] Test : connexion avec mot de passe incorrect affiche une erreur
  - [ ] Test : acces a /account redirige vers /login si non connecte

#### TEST-033 : E2E - Navigation et recherche de produits
- **Priorite** : P0
- **Complexite** : L
- **Description** : Tester la navigation dans le catalogue et la recherche
- **Criteres d'acceptation** :
  - [ ] Test : la page d'accueil charge et affiche les produits vedettes
  - [ ] Test : clic sur "Voir tout" redirige vers le catalogue
  - [ ] Test : clic sur une carte produit ouvre la page detail
  - [ ] Test : la page detail affiche toutes les informations
  - [ ] Test : navigation par categorie filtre les produits
  - [ ] Test : le tri par prix fonctionne visuellement
  - [ ] Test : la pagination fonctionne (page suivante / precedente)
  - [ ] Test : la recherche retourne des suggestions (autocomplete)
  - [ ] Test : clic sur une suggestion ouvre le produit
  - [ ] Test : recherche avec "Entree" redirige vers le catalogue filtre

#### TEST-034 : E2E - Panier et ajout d'articles
- **Priorite** : P0
- **Complexite** : M
- **Description** : Tester le parcours d'ajout au panier
- **Criteres d'acceptation** :
  - [ ] Test : ajout d'un produit au panier depuis la page detail
  - [ ] Test : le toast de confirmation s'affiche
  - [ ] Test : le badge du panier dans le header est mis a jour
  - [ ] Test : le drawer du panier s'ouvre avec l'article
  - [ ] Test : modification de la quantite dans le drawer
  - [ ] Test : suppression d'un article dans le drawer
  - [ ] Test : ajout de plusieurs produits differents
  - [ ] Test : la page panier affiche tous les articles et les totaux
  - [ ] Test : le bouton "Vider le panier" fonctionne

#### TEST-035 : E2E - Checkout complet
- **Priorite** : P0
- **Complexite** : XL
- **Description** : Tester le parcours de commande complet
- **Criteres d'acceptation** :
  - [ ] Test : redirection vers /login si non connecte
  - [ ] Test : apres connexion, retour au checkout
  - [ ] Test : saisie d'une adresse de livraison
  - [ ] Test : recapitulatif de commande correct (articles, frais, total)
  - [ ] Test : selection du moyen de paiement
  - [ ] Test : le bouton "Payer" affiche le montant correct
  - [ ] Test : clic sur "Payer" cree la commande et redirige
  - [ ] Test : la page de resultat affiche le numero de commande (sandbox)
  - [ ] Test : la commande apparait dans "Mes commandes"
  - [ ] Test : le panier est vide apres la commande

#### TEST-036 : E2E - Paiement (sandbox)
- **Priorite** : P0
- **Complexite** : L
- **Description** : Tester le flux de paiement en mode sandbox
- **Criteres d'acceptation** :
  - [ ] Test : initiation du paiement retourne une URL
  - [ ] Test : page de resultat avec statut SUCCESS (mock)
  - [ ] Test : page de resultat avec statut FAILED (mock)
  - [ ] Test : page de resultat avec statut PENDING (polling)
  - [ ] Test : bouton "Reessayer" sur la page d'echec
  - [ ] Test : le statut de la commande est mis a jour correctement

---

### Module 3.3 : Parcours admin

#### TEST-037 : E2E - Admin : Gestion des produits
- **Priorite** : P1
- **Complexite** : XL
- **Description** : Tester les fonctionnalites admin de gestion des produits
- **Criteres d'acceptation** :
  - [ ] Test : connexion en tant qu'admin
  - [ ] Test : le dashboard affiche les statistiques
  - [ ] Test : navigation vers la page produits
  - [ ] Test : la liste des produits est affichee avec pagination
  - [ ] Test : creation d'un nouveau produit (formulaire complet)
  - [ ] Test : upload d'une image pour le produit
  - [ ] Test : modification d'un produit existant
  - [ ] Test : changement de statut (DRAFT -> ACTIVE)
  - [ ] Test : toggle featured (etoile)
  - [ ] Test : suppression d'un produit avec confirmation
  - [ ] Test : filtres et recherche dans la liste

#### TEST-038 : E2E - Admin : Gestion des commandes
- **Priorite** : P1
- **Complexite** : L
- **Description** : Tester la gestion des commandes par l'admin
- **Criteres d'acceptation** :
  - [ ] Test : liste des commandes avec filtres par statut
  - [ ] Test : detail d'une commande (articles, client, adresse)
  - [ ] Test : changement de statut (PENDING -> CONFIRMED -> SHIPPED -> DELIVERED)
  - [ ] Test : seules les transitions valides sont proposees
  - [ ] Test : recherche par numero de commande

#### TEST-039 : E2E - Admin : Gestion des categories
- **Priorite** : P1
- **Complexite** : M
- **Description** : Tester le CRUD des categories par l'admin
- **Criteres d'acceptation** :
  - [ ] Test : liste des categories
  - [ ] Test : creation d'une nouvelle categorie
  - [ ] Test : modification d'une categorie
  - [ ] Test : suppression refusee si produits associes
  - [ ] Test : suppression d'une categorie vide

---

### Module 3.4 : Tests responsive

#### TEST-040 : E2E - Tests mobile responsive
- **Priorite** : P1
- **Complexite** : L
- **Description** : Tester l'experience mobile sur les ecrans 375px et 768px
- **Criteres d'acceptation** :
  - [ ] Test : page d'accueil lisible sur 375px (iPhone SE)
  - [ ] Test : menu hamburger fonctionne sur mobile
  - [ ] Test : le catalogue affiche 2 colonnes sur mobile
  - [ ] Test : la page detail produit est scrollable sur mobile
  - [ ] Test : le checkout fonctionne sur mobile
  - [ ] Test : le panier drawer fonctionne sur mobile
  - [ ] Test : le panel admin est navigable sur tablette (768px)
  - [ ] Test : pas de scroll horizontal non desire
  - [ ] Test : les boutons sont assez grands pour le touch (min 44x44px)
  - [ ] Test : les formulaires sont utilisables sur mobile (clavier)

---

## Section 4 : Tests de charge (k6)

#### TEST-041 : Configuration k6
- **Priorite** : P1
- **Complexite** : S
- **Description** : Configurer k6 et les scenarios de charge
- **Criteres d'acceptation** :
  - [ ] k6 installe
  - [ ] Scripts dans `tests/load/`
  - [ ] Variables d'environnement configurables (BASE_URL)
  - [ ] Thresholds definis (p95, taux d'erreur)
  - [ ] Rapport JSON genere apres chaque run

#### TEST-042 : Test de charge - Page d'accueil
- **Priorite** : P1
- **Complexite** : M
- **Description** : Tester la charge sur la page d'accueil et les endpoints publics
- **Criteres d'acceptation** :
  - [ ] Scenario : montee progressive de 0 a 100 utilisateurs virtuels
  - [ ] Test : GET / (page d'accueil)
  - [ ] Test : GET /api/products/featured
  - [ ] Test : GET /api/categories
  - [ ] Seuil : p95 < 500ms
  - [ ] Seuil : taux d'erreur < 1%
  - [ ] Seuil : 0 erreur 5xx
  - [ ] Duree : 5 minutes de charge soutenue

#### TEST-043 : Test de charge - API endpoints
- **Priorite** : P1
- **Complexite** : M
- **Description** : Tester la charge sur les endpoints API principaux
- **Criteres d'acceptation** :
  - [ ] Test : GET /api/products?limit=20 (pagination)
  - [ ] Test : GET /api/products/search?q=xxx (recherche)
  - [ ] Test : GET /api/products/:slug (detail)
  - [ ] Test : POST /api/auth/login (authentification)
  - [ ] Seuil : p95 < 500ms pour les lectures
  - [ ] Seuil : p95 < 1000ms pour les ecritures
  - [ ] Seuil : taux d'erreur < 1%

#### TEST-044 : Test de stress - Checkout concurrent
- **Priorite** : P1
- **Complexite** : L
- **Description** : Tester les achats simultanes et la gestion du stock
- **Criteres d'acceptation** :
  - [ ] Scenario : 30 utilisateurs tentent d'acheter le meme produit simultanement
  - [ ] Test : pas de survente (stock final >= 0)
  - [ ] Test : chaque commande recoit un numero unique
  - [ ] Test : les transactions sont atomiques (pas de commande partielle)
  - [ ] Seuil : p95 < 3000ms pour la creation de commande
  - [ ] Seuil : taux d'erreur < 5% (erreurs de stock attendues)

#### TEST-045 : Test de charge - Pool de connexions DB
- **Priorite** : P2
- **Complexite** : M
- **Description** : Verifier le comportement sous charge de la base de donnees
- **Criteres d'acceptation** :
  - [ ] Scenario : 200 utilisateurs simultanees pendant 5 minutes
  - [ ] Test : pas d'erreur "too many connections"
  - [ ] Test : le pool PgBouncer gere les connexions correctement
  - [ ] Test : les requetes ne depassent pas 5s de timeout
  - [ ] Monitoring : nombre de connexions actives pendant le test

---

## Section 5 : Tests de securite

#### TEST-046 : Scan OWASP ZAP
- **Priorite** : P1
- **Complexite** : L
- **Description** : Effectuer un scan automatise OWASP ZAP sur l'application
- **Criteres d'acceptation** :
  - [ ] Scan actif de toute l'application (frontend + API)
  - [ ] 0 alerte de severite HIGH
  - [ ] 0 alerte de severite MEDIUM non justifiee
  - [ ] Rapport HTML genere et archive
  - [ ] Toutes les alertes LOW documentees avec justification ou plan de correction

#### TEST-047 : Tests d'injection SQL
- **Priorite** : P0
- **Complexite** : M
- **Description** : Verifier la resistance aux injections SQL
- **Criteres d'acceptation** :
  - [ ] Test : injection dans les parametres de recherche (?q='; DROP TABLE--)
  - [ ] Test : injection dans les champs de formulaire (login, register)
  - [ ] Test : injection dans les parametres d'URL (:id, :slug)
  - [ ] Test : injection dans les headers (Authorization)
  - [ ] Test : verification que Prisma utilise des requetes parametrees
  - [ ] Test : aucune requete $executeRawUnsafe avec entree utilisateur
  - [ ] Resultat attendu : aucune injection possible

#### TEST-048 : Tests XSS (Cross-Site Scripting)
- **Priorite** : P0
- **Complexite** : M
- **Description** : Verifier la resistance aux attaques XSS
- **Criteres d'acceptation** :
  - [ ] Test : XSS reflechi via les parametres de recherche
  - [ ] Test : XSS stocke via les champs de formulaire (nom, description)
  - [ ] Test : XSS via les noms de fichiers uploades
  - [ ] Test : XSS dans les descriptions de produit (HTML sanitise)
  - [ ] Test : verification du header Content-Security-Policy
  - [ ] Test : verification du header X-XSS-Protection
  - [ ] Test : pas de dangerouslySetInnerHTML sans DOMPurify
  - [ ] Resultat attendu : aucun script execute

#### TEST-049 : Tests d'authentification bypass
- **Priorite** : P0
- **Complexite** : L
- **Description** : Verifier qu'aucune route protegee n'est accessible sans authentification
- **Criteres d'acceptation** :
  - [ ] Test : acces aux routes admin sans token -> 401
  - [ ] Test : acces aux routes admin avec token CUSTOMER -> 403
  - [ ] Test : acces aux routes auth sans token -> 401
  - [ ] Test : acces a la commande d'un autre utilisateur -> 403
  - [ ] Test : acces au panier d'un autre utilisateur -> 403
  - [ ] Test : acces aux adresses d'un autre utilisateur -> 403
  - [ ] Test : token expire -> 401
  - [ ] Test : token avec signature invalide -> 401
  - [ ] Test : token avec role modifie manuellement -> 401/403
  - [ ] Resultat attendu : aucun bypass possible

#### TEST-050 : Tests de rate limiting
- **Priorite** : P1
- **Complexite** : M
- **Description** : Verifier que le rate limiting fonctionne correctement
- **Criteres d'acceptation** :
  - [ ] Test : login > 5 requetes/minute -> 429 Too Many Requests
  - [ ] Test : API globale > 100 requetes/minute par IP -> 429
  - [ ] Test : callback paiement > 10 requetes/minute -> 429
  - [ ] Test : le header X-RateLimit-Remaining est present
  - [ ] Test : le header Retry-After est present en cas de 429
  - [ ] Test : les requetes sont comptees par IP (pas par user-agent)

#### TEST-051 : Tests de securite des fichiers uploades
- **Priorite** : P1
- **Complexite** : M
- **Description** : Verifier la securite des uploads de fichiers
- **Criteres d'acceptation** :
  - [ ] Test : upload d'un fichier PHP deguise en JPG -> rejete
  - [ ] Test : upload d'un fichier > 5 Mo -> rejete
  - [ ] Test : upload d'un fichier .exe -> rejete
  - [ ] Test : path traversal dans le nom de fichier (../../etc/passwd) -> rejete
  - [ ] Test : verification du magic number (pas seulement l'extension)
  - [ ] Test : les fichiers uploades ne sont pas executables

#### TEST-052 : Tests HTTPS et headers de securite
- **Priorite** : P1
- **Complexite** : S
- **Description** : Verifier la configuration HTTPS et les headers de securite
- **Criteres d'acceptation** :
  - [ ] Test : HTTP redirige vers HTTPS (301)
  - [ ] Test : HSTS est present avec max-age >= 31536000
  - [ ] Test : X-Frame-Options est SAMEORIGIN
  - [ ] Test : X-Content-Type-Options est nosniff
  - [ ] Test : Content-Security-Policy est present
  - [ ] Test : Referrer-Policy est present
  - [ ] Test : le header Server ne revele pas la version
  - [ ] Test : X-Powered-By est absent
  - [ ] Test : note SSL Labs >= A

---

## Resume des taches

| ID | Tache | Priorite | Complexite | Statut |
|---|---|---|---|---|
| **Tests unitaires backend** | | | | |
| TEST-001 | Setup tests backend | P0 | M | TODO |
| TEST-002 | Auth - Inscription | P0 | M | TODO |
| TEST-003 | Auth - Connexion | P0 | M | TODO |
| TEST-004 | Auth - Token Refresh | P0 | M | TODO |
| TEST-005 | Auth - Logout/Password | P0 | S | TODO |
| TEST-006 | JWT Guards | P0 | M | TODO |
| TEST-007 | Products - Lecture | P0 | M | TODO |
| TEST-008 | Products - Ecriture | P0 | L | TODO |
| TEST-009 | Orders - Creation | P0 | XL | TODO |
| TEST-010 | Orders - Gestion | P0 | M | TODO |
| TEST-011 | Payments - Initiation | P0 | L | TODO |
| TEST-012 | Payments - Callback | P0 | L | TODO |
| TEST-013 | Categories | P1 | M | TODO |
| TEST-014 | Cart | P1 | M | TODO |
| TEST-015 | Addresses | P1 | S | TODO |
| TEST-016 | Users | P1 | S | TODO |
| TEST-017 | Settings | P1 | S | TODO |
| TEST-018 | Dashboard | P1 | M | TODO |
| **Tests unitaires frontend** | | | | |
| TEST-019 | Setup tests frontend | P0 | M | TODO |
| TEST-020 | Composants UI de base | P0 | M | TODO |
| TEST-021 | ProductCard | P0 | M | TODO |
| TEST-022 | Header | P1 | M | TODO |
| TEST-023 | Pages Auth | P1 | M | TODO |
| TEST-024 | Pages Produits | P1 | L | TODO |
| TEST-025 | Panier | P1 | M | TODO |
| TEST-026 | Store Auth | P0 | M | TODO |
| TEST-027 | Store Cart | P0 | M | TODO |
| TEST-028 | Store Search | P1 | S | TODO |
| TEST-029 | Client Axios | P0 | M | TODO |
| TEST-030 | Fonctions utilitaires | P1 | S | TODO |
| **Tests E2E** | | | | |
| TEST-031 | Setup Playwright | P0 | M | TODO |
| TEST-032 | E2E - Auth | P0 | M | TODO |
| TEST-033 | E2E - Produits | P0 | L | TODO |
| TEST-034 | E2E - Panier | P0 | M | TODO |
| TEST-035 | E2E - Checkout | P0 | XL | TODO |
| TEST-036 | E2E - Paiement | P0 | L | TODO |
| TEST-037 | E2E - Admin Produits | P1 | XL | TODO |
| TEST-038 | E2E - Admin Commandes | P1 | L | TODO |
| TEST-039 | E2E - Admin Categories | P1 | M | TODO |
| TEST-040 | E2E - Mobile responsive | P1 | L | TODO |
| **Tests de charge** | | | | |
| TEST-041 | Setup k6 | P1 | S | TODO |
| TEST-042 | Charge - Page d'accueil | P1 | M | TODO |
| TEST-043 | Charge - API | P1 | M | TODO |
| TEST-044 | Stress - Checkout | P1 | L | TODO |
| TEST-045 | Charge - Pool DB | P2 | M | TODO |
| **Tests de securite** | | | | |
| TEST-046 | Scan OWASP ZAP | P1 | L | TODO |
| TEST-047 | Injection SQL | P0 | M | TODO |
| TEST-048 | XSS | P0 | M | TODO |
| TEST-049 | Auth bypass | P0 | L | TODO |
| TEST-050 | Rate limiting | P1 | M | TODO |
| TEST-051 | Upload securite | P1 | M | TODO |
| TEST-052 | HTTPS/Headers | P1 | S | TODO |

---

## Statistiques

| Categorie | Nombre | P0 | P1 | P2 | Estimation |
|---|---|---|---|---|---|
| Tests unitaires backend | 18 | 12 | 6 | 0 | ~44h |
| Tests unitaires frontend | 12 | 6 | 6 | 0 | ~24h |
| Tests E2E | 10 | 6 | 4 | 0 | ~36h |
| Tests de charge | 5 | 0 | 4 | 1 | ~12h |
| Tests de securite | 7 | 3 | 4 | 0 | ~16h |
| **Total** | **52** | **27** | **24** | **1** | **~132h** |

---

## Objectifs de couverture par module

| Module | Couverture cible | Priorite |
|---|---|---|
| Auth (backend) | 95%+ | P0 |
| Orders (backend) | 90%+ | P0 |
| Payments (backend) | 90%+ | P0 |
| Products (backend) | 85%+ | P0 |
| Cart (backend) | 85%+ | P1 |
| Categories (backend) | 80%+ | P1 |
| Stores Zustand (frontend) | 90%+ | P0 |
| Client API (frontend) | 85%+ | P0 |
| Composants UI (frontend) | 70%+ | P1 |
| Pages (frontend) | 60%+ | P1 |
| **Moyenne backend** | **80%+** | - |
| **Moyenne frontend** | **70%+** | - |
