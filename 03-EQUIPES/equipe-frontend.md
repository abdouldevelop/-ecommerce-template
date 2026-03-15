# Equipe Frontend - Tâches et Priorités

## Instructions pour l'agent IA

Ce document liste toutes les tâches frontend avec leur priorité, complexité estimée et spécifications UX. Le frontend utilise Next.js 15 (App Router) avec Tailwind CSS et Zustand.

**Légende priorités** : P0 (bloquant/MVP), P1 (important), P2 (souhaitable), P3 (futur)
**Légende complexité** : S (< 2h), M (2-4h), L (4-8h), XL (> 8h)

**Principe directeur** : Mobile-first. Chaque composant doit être parfait sur mobile avant de penser au desktop.

---

## Module 1 : Setup projet (P0)

### FE-001 : Initialisation Next.js
- **Priorité** : P0
- **Complexité** : M
- **Description** : Initialiser le projet Next.js dans le monorepo
- **Critères d'acceptation** :
  - [ ] Next.js 15 avec App Router dans `apps/web/`
  - [ ] Tailwind CSS configuré avec le design system du client
  - [ ] Layout principal avec `<html lang="fr">`
  - [ ] Polices Google Fonts chargées (next/font)
  - [ ] Favicon configuré
  - [ ] Métadonnées SEO par défaut (title, description, og:image)
  - [ ] Client API (axios) configuré avec baseURL et intercepteurs
  - [ ] Types TypeScript partagés depuis `packages/shared/`

### FE-002 : Stores Zustand
- **Priorité** : P0
- **Complexité** : M
- **Description** : Créer les stores de gestion d'état
- **Critères d'acceptation** :
  - [ ] `auth.store.ts` : user, token, isAuthenticated, login(), logout()
  - [ ] `cart.store.ts` : items, addItem(), removeItem(), updateQuantity(), total, persist localStorage
  - [ ] `search.store.ts` : query, results, selectedCategory, isOpen
  - [ ] Middleware persist pour le panier (survit au refresh)
  - [ ] Hydratation correcte (pas d'erreur SSR/client mismatch)

### FE-003 : Client API (axios)
- **Priorité** : P0
- **Complexité** : M
- **Description** : Configuration du client HTTP
- **Critères d'acceptation** :
  - [ ] Instance axios avec baseURL et timeout
  - [ ] Intercepteur request : ajoute le token Bearer si disponible
  - [ ] Intercepteur response 401 : tente un refresh token automatique
  - [ ] Si refresh échoue : déconnexion et redirection vers /login
  - [ ] Gestion des erreurs avec messages français
  - [ ] Fonctions helper : `api.get()`, `api.post()`, `api.patch()`, `api.delete()`

---

## Module 2 : Layout principal (P0)

### FE-004 : Header / Navbar
- **Priorité** : P0
- **Complexité** : L
- **Description** : Header fixe avec navigation
- **Spécifications UX** :
  - **Desktop** :
    - Logo à gauche
    - Barre de recherche au centre (avec autocomplete)
    - Icônes à droite : compte utilisateur, panier (avec badge quantité)
    - Fond gradient ou couleur primaire selon la charte
    - Sticky (reste visible au scroll)
  - **Mobile** :
    - Logo centré
    - Menu hamburger à gauche (ouvre un drawer)
    - Icône panier à droite (avec badge)
    - Barre de recherche en dessous (ou dans le menu)
    - Hauteur : 60-70px
- **Critères d'acceptation** :
  - [ ] Header responsive (mobile + desktop)
  - [ ] Logo cliquable → retour accueil
  - [ ] Badge panier avec le nombre d'articles (mis à jour en temps réel)
  - [ ] Menu mobile avec overlay sombre et animation slide
  - [ ] Liens : Accueil, Catégories (dynamiques), Compte, Panier
  - [ ] Affichage conditionnel : "Se connecter" si déconnecté, "Mon compte" si connecté

### FE-005 : Footer
- **Priorité** : P0
- **Complexité** : M
- **Description** : Pied de page
- **Spécifications UX** :
  - 3-4 colonnes sur desktop, empilées sur mobile
  - Colonnes : A propos, Liens utiles, Contact, Réseaux sociaux
  - Couleur de fond sombre (gris foncé ou couleur primaire sombre)
  - Copyright en bas
  - Bouton WhatsApp flottant (coin inférieur droit, au-dessus du footer)
- **Critères d'acceptation** :
  - [ ] Footer responsive
  - [ ] Liens vers : CGV, Politique de confidentialité, FAQ, Contact
  - [ ] Icônes réseaux sociaux (Facebook, Instagram, etc.)
  - [ ] Numéro de téléphone et email cliquables
  - [ ] Copyright avec année dynamique
  - [ ] Bouton WhatsApp flottant avec lien `https://wa.me/numero`

### FE-006 : Menu mobile (Drawer)
- **Priorité** : P0
- **Complexité** : M
- **Description** : Menu latéral pour mobile
- **Spécifications UX** :
  - Slide depuis la gauche
  - Overlay sombre semi-transparent
  - Logo en haut du drawer
  - Liens de navigation avec icônes
  - Catégories listées
  - Bouton "Se connecter" ou "Mon compte"
  - Fermeture : clic overlay, swipe gauche, bouton X
- **Critères d'acceptation** :
  - [ ] Animation fluide (300ms transition)
  - [ ] Empêche le scroll du body quand ouvert
  - [ ] Fermeture automatique lors de la navigation
  - [ ] Accessible (focus trap, aria-labels)

---

## Module 3 : Page d'accueil (P0)

### FE-007 : Slider / Carrousel
- **Priorité** : P0
- **Complexité** : M
- **Description** : Carrousel d'images en haut de page
- **Spécifications UX** :
  - Pleine largeur
  - 3-5 slides avec image, titre et CTA (bouton)
  - Auto-play (5 secondes par slide)
  - Indicateurs (dots) en bas
  - Swipe sur mobile
  - Hauteur : 400-500px desktop, 200-250px mobile
- **Critères d'acceptation** :
  - [ ] Composant carrousel responsive
  - [ ] Auto-play avec pause au hover/touch
  - [ ] Navigation par dots et swipe
  - [ ] Images optimisées (next/image avec priority pour le premier slide)
  - [ ] Texte overlay lisible (ombre ou fond semi-transparent)
  - [ ] CTA cliquable vers une catégorie ou une page

### FE-008 : Grille produits vedettes
- **Priorité** : P0
- **Complexité** : M
- **Description** : Section produits vedettes sur la page d'accueil
- **Spécifications UX** :
  - Titre de section ("Nos produits vedettes" ou personnalisé)
  - Grille de cartes produit
  - Desktop : 4 colonnes
  - Tablette : 2-3 colonnes
  - Mobile : 2 colonnes
  - Bouton "Voir tout" en bas → vers le catalogue
- **Critères d'acceptation** :
  - [ ] Charge les produits featured depuis l'API
  - [ ] Cartes produit avec : image, nom, prix, ancien prix (barré), badge promo
  - [ ] Skeleton loading pendant le chargement
  - [ ] Clic sur une carte → page détail produit
  - [ ] Responsive grid avec gap uniforme

### FE-009 : Section catégories
- **Priorité** : P1
- **Complexité** : M
- **Description** : Affichage des catégories sur la page d'accueil
- **Spécifications UX** :
  - Cartes ou cercles avec image et nom
  - Scroll horizontal sur mobile
  - Grille sur desktop
- **Critères d'acceptation** :
  - [ ] Charge les catégories depuis l'API
  - [ ] Clic → page catalogue filtré par catégorie
  - [ ] Images catégories ou icônes par défaut

### FE-010 : Bannière promotionnelle
- **Priorité** : P2
- **Complexité** : S
- **Description** : Bannière fixe entre les sections
- **Critères d'acceptation** :
  - [ ] Image pleine largeur avec texte et CTA
  - [ ] Responsive (image différente desktop/mobile si besoin)

---

## Module 4 : Catalogue et produits (P0)

### FE-011 : Page catalogue
- **Priorité** : P0
- **Complexité** : L
- **Description** : Liste des produits avec filtres
- **Spécifications UX** :
  - Barre de catégories en haut (scroll horizontal sur mobile)
  - Grille de produits
  - Pagination en bas
  - Option de tri (prix croissant/décroissant, nouveautés)
  - Affichage du nombre de résultats
- **Critères d'acceptation** :
  - [ ] Filtrage par catégorie (via les catégories ou URL query params)
  - [ ] Pagination fonctionnelle (pas de scroll infini pour simplifier)
  - [ ] Tri par prix et date
  - [ ] Skeleton loading pendant les requêtes
  - [ ] URL mise à jour avec les filtres (?category=t-shirts&page=2)
  - [ ] Message "Aucun produit trouvé" si résultat vide

### FE-012 : Carte produit (composant)
- **Priorité** : P0
- **Complexité** : M
- **Description** : Composant réutilisable de carte produit
- **Spécifications UX** :
  - Image avec ratio 1:1 ou 3:4
  - Nom du produit (2 lignes max, tronqué)
  - Prix actuel en gras
  - Ancien prix barré (si comparePrice)
  - Badge "Promo" ou pourcentage de réduction
  - Badge "Rupture" si stock = 0
  - Hover : légère élévation (shadow) sur desktop
- **Critères d'acceptation** :
  - [ ] Composant `<ProductCard product={product} />`
  - [ ] Image avec `next/image` et lazy loading
  - [ ] Formatage prix : "15 000 FCFA"
  - [ ] Calcul et affichage du pourcentage de réduction
  - [ ] Lien vers `/products/${slug}`
  - [ ] Skeleton variant pour le loading

### FE-013 : Page détail produit
- **Priorité** : P0
- **Complexité** : L
- **Description** : Page complète d'un produit
- **Spécifications UX** :
  - **Desktop** : Image(s) à gauche (60%), infos à droite (40%)
  - **Mobile** : Image en haut (pleine largeur), infos en dessous
  - Galerie d'images : miniatures cliquables, image principale grande
  - Nom, prix, ancien prix barré, badge promo
  - Description courte
  - Sélecteur de quantité (- / + avec min 1 et max stock)
  - Bouton "Ajouter au panier" (large, couleur primaire)
  - Indicateur de stock ("En stock", "Plus que 3", "Rupture de stock")
  - Description longue en dessous (HTML rendu)
  - Breadcrumb : Accueil > Catégorie > Nom du produit
- **Critères d'acceptation** :
  - [ ] Chargement par slug (URL SEO-friendly)
  - [ ] Galerie d'images fonctionnelle (clic miniature = change image principale)
  - [ ] Sélecteur de quantité avec validation stock
  - [ ] Bouton "Ajouter au panier" : ajoute au store Zustand + feedback visuel (toast)
  - [ ] Bouton désactivé si rupture de stock
  - [ ] SEO : title et meta description dynamiques
  - [ ] Partage : bouton partager (copier le lien ou WhatsApp)

---

## Module 5 : Panier (P0)

### FE-014 : Panier (drawer latéral)
- **Priorité** : P0
- **Complexité** : L
- **Description** : Mini-panier en drawer à droite
- **Spécifications UX** :
  - S'ouvre au clic sur l'icône panier du header
  - Slide depuis la droite
  - Liste des articles avec : image miniature, nom, prix, quantité (modifiable), bouton supprimer
  - Sous-total en bas
  - Bouton "Voir le panier" → page panier
  - Bouton "Commander" → checkout
  - Message si panier vide
- **Critères d'acceptation** :
  - [ ] S'ouvre automatiquement après ajout d'un article
  - [ ] Modification de quantité en temps réel
  - [ ] Suppression d'article avec confirmation
  - [ ] Mise à jour du badge header en temps réel
  - [ ] Animation smooth (300ms)
  - [ ] Fermeture : clic overlay, bouton X, navigation

### FE-015 : Page panier complète
- **Priorité** : P0
- **Complexité** : M
- **Description** : Page panier détaillée
- **Spécifications UX** :
  - Tableau/liste des articles (image, nom, prix unitaire, quantité, total ligne)
  - Colonne suppression
  - Récapitulatif : sous-total, frais de livraison, total
  - Bouton "Continuer les achats" → catalogue
  - Bouton "Commander" → checkout
  - Message de livraison gratuite si applicable ("Plus que X FCFA pour la livraison gratuite !")
- **Critères d'acceptation** :
  - [ ] Modification de quantité avec mise à jour immédiate
  - [ ] Recalcul automatique des totaux
  - [ ] Bouton "Vider le panier"
  - [ ] Responsive : sur mobile, layout en cartes empilées au lieu d'un tableau

---

## Module 6 : Checkout et paiement (P0)

### FE-016 : Page checkout
- **Priorité** : P0
- **Complexité** : XL
- **Description** : Page de finalisation de commande
- **Spécifications UX** :
  - Etapes : 1. Adresse → 2. Récapitulatif → 3. Paiement
  - Formulaire d'adresse (ou sélection d'une adresse existante)
  - Récapitulatif de commande (articles, frais, total)
  - Choix du moyen de paiement (icônes des opérateurs)
  - Bouton "Payer X FCFA"
- **Critères d'acceptation** :
  - [ ] Redirection vers /login si non connecté
  - [ ] Formulaire d'adresse avec validation
  - [ ] Adresses enregistrées sélectionnables
  - [ ] Récapitulatif non modifiable (retour au panier pour modifier)
  - [ ] Choix du moyen de paiement avec icônes visuelles
  - [ ] Synchronisation panier local → serveur avant de créer la commande
  - [ ] Bouton de paiement avec loading state
  - [ ] Redirection vers la page du prestataire de paiement

### FE-017 : Page résultat de paiement
- **Priorité** : P0
- **Complexité** : M
- **Description** : Page après le retour du prestataire de paiement
- **Spécifications UX** :
  - **Succès** : Icône check vert, message de remerciement, numéro de commande, bouton "Voir ma commande"
  - **Echec** : Icône X rouge, message d'erreur, bouton "Réessayer", bouton "Contacter le support"
  - **En attente** : Spinner, message "Paiement en cours de vérification", polling du statut
- **Critères d'acceptation** :
  - [ ] Récupère le statut du paiement via l'API
  - [ ] Polling toutes les 3 secondes pendant 60 secondes max si status = PROCESSING
  - [ ] Affichage conditionnel selon le statut
  - [ ] Numéro de commande cliquable → détail commande
  - [ ] Bouton "Réessayer le paiement" si échec

### FE-018 : Page paiement
- **Priorité** : P0
- **Complexité** : M
- **Description** : Page intermédiaire de choix de paiement (si la commande est déjà créée)
- **Spécifications UX** :
  - Récapitulatif de la commande (numéro, montant)
  - Grille de boutons pour chaque moyen de paiement
  - Chaque bouton affiche le logo de l'opérateur
  - Clic → initie le paiement et redirige
- **Critères d'acceptation** :
  - [ ] Accessible depuis l'espace client ("Finaliser le paiement" pour les commandes PENDING)
  - [ ] Affiche seulement les moyens de paiement activés
  - [ ] Loading state sur le bouton cliqué
  - [ ] Gestion d'erreur si l'initiation échoue

---

## Module 7 : Recherche (P1)

### FE-019 : Barre de recherche avec autocomplete
- **Priorité** : P1
- **Complexité** : L
- **Description** : Recherche en temps réel avec suggestions
- **Spécifications UX** :
  - Input avec icône loupe
  - Suggestions apparaissent après 1 caractère
  - Chaque suggestion : miniature image + nom + prix
  - Max 5 suggestions
  - Clic sur suggestion → page produit
  - "Entrée" → page catalogue avec recherche
  - Debounce 300ms pour éviter les requêtes excessives
- **Critères d'acceptation** :
  - [ ] Debounce 300ms
  - [ ] Appel API GET /api/products/search?q=xxx
  - [ ] Dropdown de suggestions sous l'input
  - [ ] Navigation au clavier (flèches haut/bas, Entrée)
  - [ ] Fermeture du dropdown au clic extérieur ou Escape
  - [ ] Affichage "Aucun résultat" si pas de match
  - [ ] Sur mobile : input en plein écran (overlay)

### FE-020 : Filtre par catégorie (barre)
- **Priorité** : P1
- **Complexité** : M
- **Description** : Barre de catégories horizontale
- **Spécifications UX** :
  - Barre horizontale sous le header (ou en haut du catalogue)
  - Boutons/pills pour chaque catégorie
  - "Tout" comme première option
  - Catégorie active visuellement distinguée
  - Scroll horizontal sur mobile
- **Critères d'acceptation** :
  - [ ] Charge les catégories depuis l'API
  - [ ] Clic → filtre le catalogue
  - [ ] Combinable avec la recherche
  - [ ] URL mise à jour (?category=slug)
  - [ ] Style pill/badge avec couleur active

---

## Module 8 : Authentification (P0)

### FE-021 : Page de connexion
- **Priorité** : P0
- **Complexité** : M
- **Description** : Page login
- **Spécifications UX** :
  - Formulaire centré : email + mot de passe
  - Bouton "Se connecter" avec loading state
  - Lien "Pas encore de compte ? Inscrivez-vous"
  - Messages d'erreur sous les champs
  - Logo du site en haut
- **Critères d'acceptation** :
  - [ ] Validation côté client avant envoi
  - [ ] Appel API POST /api/auth/login
  - [ ] Stockage du token dans le store Zustand (et localStorage)
  - [ ] Redirection vers la page précédente ou l'accueil
  - [ ] Synchronisation du panier local après connexion
  - [ ] Message d'erreur clair si identifiants incorrects

### FE-022 : Page d'inscription
- **Priorité** : P0
- **Complexité** : M
- **Description** : Page register
- **Spécifications UX** :
  - Formulaire : prénom, nom, email, téléphone, mot de passe, confirmation mot de passe
  - Indicateur de force du mot de passe
  - Bouton "Créer mon compte"
  - Lien "Déjà un compte ? Connectez-vous"
- **Critères d'acceptation** :
  - [ ] Validation : email format, password match, password strength
  - [ ] Appel API POST /api/auth/register
  - [ ] Connexion automatique après inscription
  - [ ] Redirection vers l'accueil

---

## Module 9 : Espace client (P1)

### FE-023 : Dashboard client
- **Priorité** : P1
- **Complexité** : M
- **Description** : Page d'accueil de l'espace client
- **Spécifications UX** :
  - Bienvenue "Bonjour, Prénom"
  - Cartes de statistiques : nombre de commandes, commandes payées, commandes en attente
  - Liste des commandes récentes (5 dernières)
  - Liens rapides : Mes commandes, Mon profil, Se déconnecter
- **Critères d'acceptation** :
  - [ ] Route protégée (redirection si non connecté)
  - [ ] Chargement des stats depuis l'API
  - [ ] Skeleton loading

### FE-024 : Page commandes client
- **Priorité** : P1
- **Complexité** : L
- **Description** : Historique des commandes
- **Spécifications UX** :
  - Liste de commandes avec : numéro, date, statut (badge couleur), montant
  - Clic sur une commande → détail expandable ou page dédiée
  - Détail : liste des articles, adresse, statut paiement
  - Bouton "Finaliser le paiement" si commande PENDING
- **Critères d'acceptation** :
  - [ ] Pagination
  - [ ] Badges de statut colorés (PENDING=jaune, CONFIRMED=bleu, SHIPPED=violet, DELIVERED=vert)
  - [ ] Détail expandable (accordéon) avec articles et prix
  - [ ] Bouton paiement pour les commandes PENDING

### FE-025 : Page profil client
- **Priorité** : P1
- **Complexité** : M
- **Description** : Modification du profil
- **Spécifications UX** :
  - Formulaire pré-rempli : prénom, nom, email (lecture seule), téléphone
  - Bouton "Enregistrer"
  - Section séparée : "Changer le mot de passe"
- **Critères d'acceptation** :
  - [ ] Pré-remplissage avec les données actuelles
  - [ ] Validation et envoi API
  - [ ] Feedback succès/erreur (toast)
  - [ ] Changement de mot de passe avec vérification ancien mot de passe

---

## Module 10 : Panel Admin (P1)

### FE-026 : Layout admin
- **Priorité** : P1
- **Complexité** : M
- **Description** : Layout spécifique pour le panel admin
- **Spécifications UX** :
  - Sidebar à gauche (fixe sur desktop, drawer sur mobile)
  - Liens : Dashboard, Produits, Catégories, Commandes, Clients
  - Header admin avec nom de l'admin et bouton déconnexion
  - Thème sombre (dark mode)
  - Breadcrumb sous le header
- **Critères d'acceptation** :
  - [ ] Route protégée (ADMIN uniquement)
  - [ ] Sidebar responsive (drawer sur mobile)
  - [ ] Thème sombre indépendant du thème de la boutique
  - [ ] Navigation active visuellement identifiée

### FE-027 : Dashboard admin
- **Priorité** : P1
- **Complexité** : L
- **Description** : Page d'accueil du panel admin
- **Spécifications UX** :
  - Cartes de stats : CA total, CA payé, CA en attente, nombre de commandes, nombre de clients
  - Graphique simple de revenus (optionnel)
  - Tableau des clients avec stats (nom, email, nb commandes payées/en attente, total)
  - Liste des 5 dernières commandes
- **Critères d'acceptation** :
  - [ ] Chargement des données depuis GET /api/admin/dashboard
  - [ ] Formatage des montants (séparateurs de milliers)
  - [ ] Skeleton loading
  - [ ] Tableau responsive (scroll horizontal sur mobile)

### FE-028 : Gestion des produits (admin)
- **Priorité** : P1
- **Complexité** : XL
- **Description** : CRUD complet des produits
- **Spécifications UX** :
  - **Liste** : tableau avec image miniature, nom, catégorie, prix, stock, statut, actions
  - **Création/édition** : formulaire avec tous les champs + upload d'images
  - **Suppression** : confirmation modale
  - Filtres : par statut, par catégorie, recherche par nom
- **Critères d'acceptation** :
  - [ ] Tableau paginé et filtrable
  - [ ] Formulaire de création/édition avec validation
  - [ ] Upload d'images avec preview avant envoi
  - [ ] Drag & drop pour réordonner les images
  - [ ] Toggle featured (étoile)
  - [ ] Changement de statut rapide (DRAFT/ACTIVE/ARCHIVED)
  - [ ] Confirmation avant suppression
  - [ ] Feedback succès/erreur pour chaque action

### FE-029 : Gestion des catégories (admin)
- **Priorité** : P1
- **Complexité** : M
- **Description** : CRUD catégories
- **Spécifications UX** :
  - Liste avec : nom, nombre de produits, ordre, actions
  - Formulaire de création/édition en modale
  - Suppression avec vérification (catégorie non vide)
- **Critères d'acceptation** :
  - [ ] Liste triée par sortOrder
  - [ ] Modale de création/édition
  - [ ] Avertissement si tentative de suppression avec des produits
  - [ ] Upload image catégorie

### FE-030 : Gestion des commandes (admin)
- **Priorité** : P1
- **Complexité** : L
- **Description** : Liste et gestion des commandes
- **Spécifications UX** :
  - Tableau : numéro, client, date, montant, statut (badge), paiement (badge), actions
  - Filtres : par statut, par date, recherche par numéro
  - Détail de commande : articles, adresse, paiement, historique des statuts
  - Bouton de changement de statut (dropdown)
- **Critères d'acceptation** :
  - [ ] Tableau paginé avec filtres
  - [ ] Badges de statut colorés
  - [ ] Détail commande dans une modale ou page dédiée
  - [ ] Changement de statut avec confirmation
  - [ ] Seulement les transitions valides proposées

### FE-031 : Liste des clients (admin)
- **Priorité** : P1
- **Complexité** : M
- **Description** : Liste des clients avec statistiques
- **Spécifications UX** :
  - Tableau : nom, email, téléphone, nb commandes, total payé, total en attente, date inscription
  - Recherche par nom ou email
  - Tri par colonnes
- **Critères d'acceptation** :
  - [ ] Tableau paginé et recherchable
  - [ ] Formatage des montants
  - [ ] Clic sur un client → voir ses commandes

---

## Module 11 : Pages statiques (P1)

### FE-032 : Page A propos
- **Priorité** : P1
- **Complexité** : S
- **Description** : Page de présentation de l'entreprise
- **Critères d'acceptation** :
  - [ ] Contenu fourni par le client
  - [ ] Image(s) si disponible(s)
  - [ ] SEO : title et meta description

### FE-033 : Pages légales (CGV, Confidentialité)
- **Priorité** : P1
- **Complexité** : S
- **Description** : Pages de texte légal
- **Critères d'acceptation** :
  - [ ] Contenu HTML rendu proprement
  - [ ] Table des matières si long
  - [ ] Imprimable

### FE-034 : Page contact
- **Priorité** : P1
- **Complexité** : S
- **Description** : Page de contact
- **Critères d'acceptation** :
  - [ ] Adresse, téléphone, email, horaires
  - [ ] Carte Google Maps (optionnel)
  - [ ] Formulaire de contact (optionnel, P2)
  - [ ] Bouton WhatsApp

### FE-035 : Page 404
- **Priorité** : P1
- **Complexité** : S
- **Description** : Page d'erreur personnalisée
- **Critères d'acceptation** :
  - [ ] Message sympathique ("Oups, cette page n'existe pas")
  - [ ] Bouton retour à l'accueil
  - [ ] Cohérent avec le design du site

---

## Module 12 : Composants UI (P0)

### FE-036 : Toast / Notifications
- **Priorité** : P0
- **Complexité** : S
- **Description** : Système de notifications toast
- **Critères d'acceptation** :
  - [ ] Types : success (vert), error (rouge), info (bleu), warning (orange)
  - [ ] Position : en haut à droite
  - [ ] Auto-dismiss après 3-5 secondes
  - [ ] Empilables (max 3 visibles)

### FE-037 : Modale de confirmation
- **Priorité** : P0
- **Complexité** : S
- **Description** : Modale réutilisable pour les confirmations
- **Critères d'acceptation** :
  - [ ] Titre, message, boutons Confirmer/Annuler
  - [ ] Overlay sombre
  - [ ] Fermeture par Escape ou clic overlay
  - [ ] Variantes : danger (rouge) pour les suppressions

### FE-038 : Skeleton loaders
- **Priorité** : P0
- **Complexité** : S
- **Description** : Placeholders animés pendant le chargement
- **Critères d'acceptation** :
  - [ ] Skeleton pour : carte produit, liste, tableau, texte
  - [ ] Animation pulse (gris clair pulsant)
  - [ ] Même dimensions que le contenu réel

---

## Résumé des tâches

| ID | Tâche | Priorité | Complexité | Statut |
|---|---|---|---|---|
| FE-001 | Setup Next.js | P0 | M | TODO |
| FE-002 | Stores Zustand | P0 | M | TODO |
| FE-003 | Client API | P0 | M | TODO |
| FE-004 | Header/Navbar | P0 | L | TODO |
| FE-005 | Footer | P0 | M | TODO |
| FE-006 | Menu mobile | P0 | M | TODO |
| FE-007 | Slider | P0 | M | TODO |
| FE-008 | Grille produits vedettes | P0 | M | TODO |
| FE-009 | Section catégories | P1 | M | TODO |
| FE-010 | Bannière promo | P2 | S | TODO |
| FE-011 | Page catalogue | P0 | L | TODO |
| FE-012 | Carte produit | P0 | M | TODO |
| FE-013 | Page détail produit | P0 | L | TODO |
| FE-014 | Panier drawer | P0 | L | TODO |
| FE-015 | Page panier | P0 | M | TODO |
| FE-016 | Page checkout | P0 | XL | TODO |
| FE-017 | Page résultat paiement | P0 | M | TODO |
| FE-018 | Page paiement | P0 | M | TODO |
| FE-019 | Recherche autocomplete | P1 | L | TODO |
| FE-020 | Filtre catégories | P1 | M | TODO |
| FE-021 | Page connexion | P0 | M | TODO |
| FE-022 | Page inscription | P0 | M | TODO |
| FE-023 | Dashboard client | P1 | M | TODO |
| FE-024 | Commandes client | P1 | L | TODO |
| FE-025 | Profil client | P1 | M | TODO |
| FE-026 | Layout admin | P1 | M | TODO |
| FE-027 | Dashboard admin | P1 | L | TODO |
| FE-028 | Produits admin | P1 | XL | TODO |
| FE-029 | Catégories admin | P1 | M | TODO |
| FE-030 | Commandes admin | P1 | L | TODO |
| FE-031 | Clients admin | P1 | M | TODO |
| FE-032 | Page A propos | P1 | S | TODO |
| FE-033 | Pages légales | P1 | S | TODO |
| FE-034 | Page contact | P1 | S | TODO |
| FE-035 | Page 404 | P1 | S | TODO |
| FE-036 | Toast | P0 | S | TODO |
| FE-037 | Modale confirmation | P0 | S | TODO |
| FE-038 | Skeleton loaders | P0 | S | TODO |

**Total estimé P0** : ~56h
**Total estimé P1** : ~40h
**Total estimé P2** : ~2h
