# Types de Sites — Allons-y.ci

> Ce document definit les 8 types de sites supportes par la plateforme Allons-y.ci.
> Chaque type a son boilerplate, son pricing, et ses adaptations specifiques pour le pipeline BMAD.

---

## Vue d'ensemble

| # | Type | Prix (FCFA) | Boilerplate | Statut |
|---|------|-------------|-------------|--------|
| 1 | Site Vitrine | 350 000 | `vitrine-boilerplate` | A creer |
| 2 | E-Commerce | 650 000 | `ecommerce-boilerplate` | Existe |
| 3 | Restaurant | 450 000 | `restaurant-boilerplate` | A creer |
| 4 | Reservation/Booking | 550 000 | `booking-boilerplate` | A creer |
| 5 | Blog/Magazine | 300 000 | `blog-boilerplate` | A creer |
| 6 | Immobilier | 850 000 | `immo-boilerplate` | A creer |
| 7 | Portfolio | 250 000 | `portfolio-boilerplate` | A creer |
| 8 | Landing Page | 150 000 | `landing-boilerplate` | A creer |

---

## 1. Site Vitrine (`vitrine`)

**Prix** : 350 000 FCFA
**Boilerplate** : `git@github.com:abdouldevelop/vitrine-boilerplate.git` (a creer)
**Description** : Site de presentation d'entreprise classique avec pages statiques, formulaire de contact, et integration reseaux sociaux.

### Public cible
- PME, artisans, professions liberales, associations
- Entreprises qui veulent une presence en ligne simple et professionnelle

### Fonctionnalites
- Page d'accueil avec hero, services, temoignages, CTA
- Page "A propos" / "Qui sommes-nous"
- Page "Services" ou "Nos activites"
- Page "Contact" avec formulaire (email + WhatsApp)
- Page "Mentions legales"
- Responsive design (mobile-first)
- SEO de base (meta tags, sitemap, robots.txt)
- Integration Google Maps (localisation)
- Liens reseaux sociaux
- Bouton WhatsApp flottant
- Google Analytics / tracking

### Adaptations BMAD

**ANALYST (PRD)** :
- Identifier les pages necessaires (accueil, a propos, services, contact minimum)
- Definir le ton editorial (corporate, decontracte, luxe...)
- Lister les contenus a mettre en avant (services, equipe, realisations)
- Identifier si galerie photo est necessaire
- Verifier si un blog est demande (upgrade vers type Blog si oui)

**ARCHITECT** :
- Stack : Next.js 15 (SSG) + Tailwind CSS
- Pas de backend complexe — formulaire de contact via API route ou service externe (Formspree/EmailJS)
- Structure de pages statiques avec generateStaticParams
- Optimisation images avec next/image
- Configuration i18n si necessaire (francais par defaut)

**SCRUM MASTER (Stories)** :
- STORY-001 : Configuration projet + theming (couleurs, fonts, logo)
- STORY-002 : Page d'accueil (hero, services, temoignages)
- STORY-003 : Pages secondaires (a propos, services)
- STORY-004 : Page contact (formulaire + map)
- STORY-005 : SEO + performance + PWA
- STORY-006 : Responsive + animations

**DEVELOPER** :
- Privilegier le SSG (Static Site Generation) pour la performance
- Utiliser des composants reutilisables (Section, Card, Button)
- Animations subtiles avec Framer Motion
- Formulaire de contact avec validation cote client
- Pas de base de donnees necessaire

**QA Checklist** :
- [ ] Toutes les pages sont accessibles et responsive
- [ ] Formulaire de contact fonctionne (envoi email)
- [ ] Bouton WhatsApp redirige correctement
- [ ] SEO : meta title, description, og:image sur chaque page
- [ ] Temps de chargement < 3s sur mobile
- [ ] Score Lighthouse > 90 (Performance, SEO, Accessibility)
- [ ] Google Maps s'affiche correctement
- [ ] Liens reseaux sociaux fonctionnels

---

## 2. E-Commerce (`ecommerce`)

**Prix** : 650 000 FCFA
**Boilerplate** : `git@github.com:abdouldevelop/ecommerce-boilerplate.git` (existe)
**Description** : Boutique en ligne complete avec catalogue produits, panier, paiement mobile money, et tableau de bord admin.

### Public cible
- Commercants, boutiques, marques locales
- Entreprises vendant des produits physiques ou numeriques

### Fonctionnalites
- Catalogue produits avec categories et filtres
- Fiches produits detaillees (images, prix, description, variantes)
- Panier d'achat persistant
- Tunnel de commande (checkout)
- Paiement mobile money (CinetPay : Wave, Orange Money, MTN, Moov)
- Gestion des commandes (statuts, historique)
- Espace client (profil, commandes, favoris)
- Tableau de bord admin (produits, commandes, statistiques)
- Gestion de stock
- Recherche et filtres avances
- Systeme de promotions / codes promo
- Notifications email (confirmation commande, expedition)
- SEO produits (structured data, sitemap)

### Adaptations BMAD

**ANALYST (PRD)** :
- Identifier les categories de produits
- Definir les methodes de livraison disponibles
- Lister les moyens de paiement souhaites
- Estimer le volume de produits (< 100, 100-1000, > 1000)
- Verifier si gestion multi-vendeurs est necessaire
- Identifier les promotions / soldes prevues

**ARCHITECT** :
- Stack : Next.js 15 (frontend) + Fastify/Express (API) + PostgreSQL + Prisma
- Architecture monorepo (apps/web + apps/api)
- Redis pour sessions et cache produits
- Stockage images : local + CDN
- Integration CinetPay pour paiements
- Queue pour emails transactionnels

**SCRUM MASTER (Stories)** :
- STORY-001 : Setup DB + modeles (Product, Category, Order, User)
- STORY-002 : CRUD produits + categories (admin)
- STORY-003 : Catalogue frontend (listing, filtres, recherche)
- STORY-004 : Fiche produit + variantes
- STORY-005 : Panier + checkout
- STORY-006 : Integration paiement CinetPay
- STORY-007 : Espace client (auth, profil, commandes)
- STORY-008 : Dashboard admin (stats, commandes)
- STORY-009 : SEO + performance
- STORY-010 : Tests E2E

**DEVELOPER** :
- Utiliser le boilerplate ecommerce-boilerplate existant
- Adapter les couleurs, logo, contenus du client
- Configurer CinetPay avec les credentials du client
- Seeder les produits si le client fournit un catalogue
- Optimiser les images produits (WebP, lazy loading)

**QA Checklist** :
- [ ] Catalogue : navigation, filtres, pagination fonctionnels
- [ ] Panier : ajout, modification quantite, suppression, persistance
- [ ] Checkout : formulaire valide, paiement CinetPay fonctionnel
- [ ] Paiement : webhook CinetPay recoit les notifications
- [ ] Commandes : creation, changement statut, email notification
- [ ] Admin : CRUD produits, gestion commandes, stats
- [ ] Responsive sur mobile (checkout inclus)
- [ ] Performance : < 3s chargement catalogue
- [ ] Securite : authentification, CSRF, validation inputs

---

## 3. Restaurant (`restaurant`)

**Prix** : 450 000 FCFA
**Boilerplate** : `git@github.com:abdouldevelop/restaurant-boilerplate.git` (a creer)
**Description** : Site pour restaurant avec menu interactif, commande en ligne (livraison/sur place), reservation de table, et galerie photo.

### Public cible
- Restaurants, fast-foods, traiteurs, bars, cafes
- Etablissements de restauration avec livraison

### Fonctionnalites
- Page d'accueil avec ambiance visuelle (hero plein ecran, galerie)
- Menu interactif avec categories (entrees, plats, desserts, boissons)
- Commande en ligne (livraison et/ou a emporter)
- Reservation de table (date, heure, nombre de couverts)
- Galerie photos (plats, ambiance, equipe)
- Horaires d'ouverture
- Localisation (Google Maps)
- Avis clients
- Integration WhatsApp pour commandes
- Page evenements / menus speciaux
- QR Code menu (generateur)

### Adaptations BMAD

**ANALYST (PRD)** :
- Obtenir le menu complet avec prix
- Identifier si livraison est proposee (zone, frais, delais)
- Verifier si reservation de table est necessaire
- Obtenir les horaires d'ouverture
- Photos des plats et du restaurant
- Identifier les evenements reguliers (brunch du dimanche, etc.)

**ARCHITECT** :
- Stack : Next.js 15 + API routes (pas de backend separe pour les petits restaus)
- Base de donnees : PostgreSQL (menu, commandes, reservations)
- Prisma pour l'ORM
- Systeme de commande simplifie (pas de paiement en ligne obligatoire — option WhatsApp)
- Generateur de QR code pour menu digital

**SCRUM MASTER (Stories)** :
- STORY-001 : Setup + theming restaurant (ambiance visuelle)
- STORY-002 : Menu interactif (categories, plats, prix, photos)
- STORY-003 : Systeme de commande (panier, choix livraison/emporter)
- STORY-004 : Reservation de table (formulaire, calendrier)
- STORY-005 : Galerie photo + page evenements
- STORY-006 : Page contact (horaires, map, WhatsApp)
- STORY-007 : Admin (gestion menu, commandes, reservations)
- STORY-008 : QR Code menu + SEO

**DEVELOPER** :
- Design "appetissant" : grandes images, couleurs chaudes
- Menu avec filtres (vegetarien, epice, populaire)
- Animations fluides sur le menu (hover, transitions)
- Formulaire de reservation avec validation horaires
- Notification WhatsApp/SMS pour nouvelles commandes
- Mode sombre optionnel pour ambiance restaurant

**QA Checklist** :
- [ ] Menu complet avec tous les plats, prix et photos
- [ ] Commande : selection plats, panier, validation commande
- [ ] Reservation : choix date/heure, nombre couverts, confirmation
- [ ] Horaires d'ouverture affiches correctement
- [ ] Google Maps affiche la bonne adresse
- [ ] Galerie photo responsive et performante
- [ ] Bouton WhatsApp commande fonctionnel
- [ ] QR Code genere et scanne correctement
- [ ] Admin : modification menu, gestion reservations

---

## 4. Reservation/Booking (`booking`)

**Prix** : 550 000 FCFA
**Boilerplate** : `git@github.com:abdouldevelop/booking-boilerplate.git` (a creer)
**Description** : Systeme de reservation en ligne pour hotels, salles, salons de beaute, consultations, ou tout service sur rendez-vous.

### Public cible
- Hotels, maisons d'hotes, residences meublees
- Salons de beaute, coiffure, spa
- Cabinets medicaux, consultants, coachs
- Salles de conference, espaces evenementiels

### Fonctionnalites
- Calendrier de disponibilites interactif
- Reservation avec choix de creneau (date + heure)
- Gestion des services/chambres/prestations
- Paiement en ligne (acompte ou total)
- Confirmation par email/SMS
- Espace client (mes reservations, historique)
- Tableau de bord admin (planning, reservations, revenus)
- Systeme d'annulation / modification
- Gestion des tarifs (haute/basse saison, week-end)
- Rappels automatiques (J-1, H-2)
- Avis clients apres la prestation

### Adaptations BMAD

**ANALYST (PRD)** :
- Identifier le type de reservation (hotel, rdv, salle, prestation)
- Definir les creneaux disponibles (horaires, duree)
- Identifier la politique d'annulation
- Definir les tarifs (fixe, saisonnier, par creneau)
- Verifier si paiement en ligne est requis ou optionnel
- Lister les services/prestations proposees

**ARCHITECT** :
- Stack : Next.js 15 + Fastify API + PostgreSQL + Prisma
- Calendrier : react-big-calendar ou fullcalendar
- Gestion des conflits de reservation (verrous optimistes)
- Cron jobs pour rappels automatiques
- Integration CinetPay pour paiements
- API REST pour future appli mobile

**SCRUM MASTER (Stories)** :
- STORY-001 : Setup DB (Services, Slots, Bookings, Users)
- STORY-002 : Catalogue services/prestations
- STORY-003 : Calendrier de disponibilites (frontend)
- STORY-004 : Formulaire de reservation + validation
- STORY-005 : Paiement en ligne (acompte/total)
- STORY-006 : Espace client (mes reservations)
- STORY-007 : Dashboard admin (planning, stats)
- STORY-008 : Notifications (email, rappels)
- STORY-009 : Gestion annulations + politique tarifaire
- STORY-010 : SEO + tests

**DEVELOPER** :
- Calendrier interactif avec creneaux colores (libre/occupe/en attente)
- Gestion des fuseaux horaires (Afrique/Abidjan par defaut)
- Prevention des double-reservations (transaction DB)
- Interface admin type "planning" (vue jour/semaine/mois)
- Emails de confirmation avec recapitulatif et QR code

**QA Checklist** :
- [ ] Calendrier affiche les disponibilites en temps reel
- [ ] Reservation : selection creneau, formulaire, confirmation
- [ ] Pas de double-reservation possible (test concurrent)
- [ ] Paiement fonctionne (acompte et total)
- [ ] Email de confirmation recu apres reservation
- [ ] Annulation : politique respectee, remboursement si applicable
- [ ] Admin : vue planning, modification creneaux, stats
- [ ] Rappels automatiques envoyes (J-1)
- [ ] Responsive : reservation complete sur mobile

---

## 5. Blog/Magazine (`blog`)

**Prix** : 300 000 FCFA
**Boilerplate** : `git@github.com:abdouldevelop/blog-boilerplate.git` (a creer)
**Description** : Blog professionnel ou magazine en ligne avec systeme de publication, categories, commentaires, et newsletter.

### Public cible
- Medias en ligne, journaux, magazines
- Blogueurs professionnels, influenceurs
- Entreprises souhaitant un blog corporate
- Associations, ONG (actualites)

### Fonctionnalites
- Systeme de publication d'articles (editeur riche)
- Categories et tags
- Recherche d'articles
- Commentaires (moderes)
- Newsletter (inscription + envoi)
- Partage reseaux sociaux
- Articles a la une / slider
- Pagination infinie ou classique
- Auteurs multiples avec bio
- SEO avance (structured data Article, breadcrumbs)
- Flux RSS
- Mode lecture (clean reading)
- Statistiques de lecture

### Adaptations BMAD

**ANALYST (PRD)** :
- Identifier le type de contenu (actualites, tutoriels, opinions, magazine)
- Nombre d'auteurs/contributeurs
- Frequence de publication prevue
- Monetisation prevue (pub, sponsored, paywall)
- Newsletter existante ou a creer
- Verifier si commentaires sont souhaites

**ARCHITECT** :
- Stack : Next.js 15 (ISR pour articles) + API routes + PostgreSQL
- Editeur : TipTap ou MDX pour le contenu
- Prisma pour articles, categories, auteurs, commentaires
- RSS feed genere automatiquement
- Sitemap dynamique pour SEO
- Cache ISR (Incremental Static Regeneration) pour performance

**SCRUM MASTER (Stories)** :
- STORY-001 : Setup DB (Articles, Categories, Authors, Comments)
- STORY-002 : Editeur d'articles (admin) avec upload images
- STORY-003 : Page d'accueil (articles recents, a la une, categories)
- STORY-004 : Page article (contenu, auteur, partage, articles lies)
- STORY-005 : Categories + tags + recherche
- STORY-006 : Commentaires (ajout, moderation admin)
- STORY-007 : Newsletter (inscription, envoi)
- STORY-008 : SEO + RSS + sitemap
- STORY-009 : Dashboard admin (stats, moderation)

**DEVELOPER** :
- ISR pour les pages articles (revalidation toutes les 60s)
- Editeur riche avec upload d'images inline
- Systeme de slug automatique pour les URLs SEO-friendly
- Open Graph images generees automatiquement
- Temps de lecture estime affiche sur chaque article
- Lazy loading des images dans les articles

**QA Checklist** :
- [ ] Publication d'article : creation, edition, suppression
- [ ] Editeur : formatage texte, images, liens fonctionnels
- [ ] Categories et tags : navigation, filtrage
- [ ] Recherche : resultats pertinents, performance
- [ ] Commentaires : ajout, moderation, anti-spam
- [ ] Newsletter : inscription, desinscription
- [ ] RSS feed valide (W3C validator)
- [ ] SEO : structured data Article, meta tags, sitemap
- [ ] Performance : < 2s chargement article, Lighthouse > 90

---

## 6. Immobilier (`immo`)

**Prix** : 850 000 FCFA
**Boilerplate** : `git@github.com:abdouldevelop/immo-boilerplate.git` (a creer)
**Description** : Plateforme immobiliere avec annonces, recherche avancee, carte interactive, et gestion des biens pour agences ou promoteurs.

### Public cible
- Agences immobilieres
- Promoteurs immobiliers
- Courtiers et agents independants
- Plateformes de petites annonces immobilieres

### Fonctionnalites
- Catalogue de biens (vente et/ou location)
- Recherche avancee (type, prix, surface, localisation, nombre de pieces)
- Fiches bien detaillees (galerie, description, plan, caracteristiques)
- Carte interactive (Google Maps / Mapbox)
- Filtres geographiques (quartier, ville, zone)
- Systeme de favoris / alertes
- Formulaire de contact par bien
- Estimation de prix
- Visite virtuelle (integration 360)
- Espace agent (mes biens, statistiques)
- Dashboard admin (biens, agents, demandes)
- Comparateur de biens

### Adaptations BMAD

**ANALYST (PRD)** :
- Type d'agence (vente seule, location, les deux)
- Zone geographique couverte
- Volume de biens (< 50, 50-500, > 500)
- Nombre d'agents/commerciaux
- Types de biens (appartements, maisons, terrains, bureaux)
- Visite virtuelle souhaitee ou non
- Integration MLS ou import CSV

**ARCHITECT** :
- Stack : Next.js 15 + Fastify API + PostgreSQL + PostGIS (geospatial)
- Prisma + extension spatiale pour recherche geographique
- Mapbox ou Google Maps pour carte interactive
- Stockage images : S3 ou local + CDN
- Elasticsearch pour recherche avancee (optionnel, Prisma full-text sinon)
- API REST pour appli mobile future

**SCRUM MASTER (Stories)** :
- STORY-001 : Setup DB (Properties, PropertyTypes, Agents, Inquiries)
- STORY-002 : CRUD biens immobiliers (admin/agent)
- STORY-003 : Catalogue frontend (listing, filtres avances)
- STORY-004 : Fiche bien detaillee (galerie, carte, caracteristiques)
- STORY-005 : Carte interactive (markers, clusters, recherche zone)
- STORY-006 : Recherche avancee (prix, surface, type, localisation)
- STORY-007 : Contact par bien + favoris + alertes
- STORY-008 : Espace agent (mes biens, demandes recues)
- STORY-009 : Dashboard admin (stats, gestion agents)
- STORY-010 : SEO immobilier + performance
- STORY-011 : Comparateur de biens

**DEVELOPER** :
- Recherche geographique avec bounding box ou rayon
- Galerie d'images avec zoom et carrousel plein ecran
- Filtres avec mise a jour URL (partageables)
- Carte avec clusters pour gros volumes
- Fiches biens avec structured data (RealEstateListing)
- Formulaire de contact specifique par bien (pre-rempli)

**QA Checklist** :
- [ ] Catalogue : listing, filtres, pagination, tri
- [ ] Recherche : par prix, surface, type, localisation, nombre pieces
- [ ] Carte : affichage markers, clusters, clic → fiche bien
- [ ] Fiche bien : galerie, description, caracteristiques, carte
- [ ] Contact par bien : formulaire envoie email a l'agent
- [ ] Favoris : ajout, suppression, persistance
- [ ] Admin/Agent : CRUD biens, upload images multiple
- [ ] Performance : carte fluide avec 100+ markers
- [ ] SEO : structured data, meta tags par bien
- [ ] Responsive : navigation carte sur mobile

---

## 7. Portfolio (`portfolio`)

**Prix** : 250 000 FCFA
**Boilerplate** : `git@github.com:abdouldevelop/portfolio-boilerplate.git` (a creer)
**Description** : Portfolio creatif pour presentez vos realisations, competences, et services. Ideal pour freelances, artistes, et createurs.

### Public cible
- Freelances (developpeurs, designers, photographes)
- Artistes, createurs, artisans
- Architectes, decorateurs d'interieur
- Photographes, videographes

### Fonctionnalites
- Page d'accueil avec effet "wow" (animations, hero creatif)
- Galerie de projets/realisations avec filtres
- Fiche projet detaillee (images, description, technologies, lien)
- Section "A propos" / CV interactif
- Section competences avec barres ou graphiques
- Temoignages clients
- Formulaire de contact
- Telechargement CV (PDF)
- Blog optionnel (articles techniques)
- Mode sombre/clair
- Animations avancees (scroll, parallax, transitions)

### Adaptations BMAD

**ANALYST (PRD)** :
- Type de portfolio (tech, design, photo, art, architecture)
- Nombre de projets a presenter
- Style souhaite (minimaliste, creatif, corporate)
- CV a integrer ou non
- Blog souhaite ou non
- Reseaux sociaux a mettre en avant

**ARCHITECT** :
- Stack : Next.js 15 (SSG) + Tailwind CSS + Framer Motion
- Pas de backend — contenu en MDX ou JSON
- Galerie avec lightbox
- Animations avec Framer Motion ou GSAP
- Hosting optimise statique (Vercel-like)

**SCRUM MASTER (Stories)** :
- STORY-001 : Setup + design system (theme, fonts, couleurs)
- STORY-002 : Page d'accueil (hero anime, navigation)
- STORY-003 : Galerie projets (filtres, grille, animations)
- STORY-004 : Fiche projet detaillee
- STORY-005 : Section competences + a propos + CV
- STORY-006 : Contact + temoignages
- STORY-007 : Animations avancees + mode sombre
- STORY-008 : SEO + performance + deploy

**DEVELOPER** :
- Animations de haute qualite (scroll-triggered, page transitions)
- Galerie avec effet masonry ou grille isotope
- Cursor personalise (optionnel selon le style)
- Preloader anime
- Smooth scrolling
- Images optimisees avec blur placeholder
- Microinteractions sur les boutons et liens

**QA Checklist** :
- [ ] Animations fluides (pas de saccade, 60fps)
- [ ] Galerie : filtres, lightbox, responsive
- [ ] Projets : fiches completes avec images et description
- [ ] Contact : formulaire fonctionnel
- [ ] CV : telechargement PDF fonctionne
- [ ] Mode sombre/clair : transition fluide, persistance
- [ ] Performance : Lighthouse > 90 malgre les animations
- [ ] SEO : meta tags, structured data Person
- [ ] Responsive : animations adaptees au mobile (simplifiees)

---

## 8. Landing Page (`landing`)

**Prix** : 150 000 FCFA
**Boilerplate** : `git@github.com:abdouldevelop/landing-boilerplate.git` (a creer)
**Description** : Page unique de promotion pour un produit, service, evenement, ou campagne marketing. Optimisee pour la conversion.

### Public cible
- Lancement de produit / service
- Campagnes marketing / pub
- Evenements (conferences, formations, concerts)
- Applications mobiles (page de telechargement)
- Pre-lancement / collecte d'emails

### Fonctionnalites
- Section hero avec CTA principal
- Section benefices / avantages
- Temoignages / preuves sociales
- Section pricing (si applicable)
- FAQ accordion
- Formulaire de capture (email, inscription)
- Compteur (compte a rebours pour evenement)
- Video embed (YouTube/Vimeo)
- Boutons CTA multiples
- Social proof (logos clients, chiffres cles)
- Footer minimaliste
- Tracking conversions (Facebook Pixel, Google Analytics)

### Adaptations BMAD

**ANALYST (PRD)** :
- Objectif de la landing page (vente, inscription, telechargement, evenement)
- CTA principal (texte du bouton, action)
- Contenu de chaque section
- Urgence / scarcite a communiquer
- Temoignages disponibles
- Video a integrer ou non

**ARCHITECT** :
- Stack : Next.js 15 (SSG, page unique) + Tailwind CSS
- Zero backend — formulaire via Formspree, Mailchimp, ou API route simple
- Page unique avec sections ancrages
- Optimisation extreme : < 1s LCP
- A/B testing ready (variantes de CTA)

**SCRUM MASTER (Stories)** :
- STORY-001 : Setup + theming (couleurs, fonts, branding)
- STORY-002 : Hero + CTA principal
- STORY-003 : Sections benefices + social proof
- STORY-004 : Temoignages + FAQ
- STORY-005 : Formulaire de capture + integration email
- STORY-006 : Animations + compte a rebours (si evenement)
- STORY-007 : SEO + tracking + performance

**DEVELOPER** :
- Page unique, scroll fluide entre sections
- Animations au scroll (intersection observer)
- CTA visible en permanence (sticky ou repete)
- Formulaire avec validation immediate
- Compte a rebours dynamique (si date evenement)
- Optimisation extreme des images (WebP, lazy, blur)
- Score Lighthouse 100 vise

**QA Checklist** :
- [ ] CTA principal visible et fonctionnel
- [ ] Formulaire : soumission, validation, message succes
- [ ] Compte a rebours correct (si applicable)
- [ ] Video se charge et se lit correctement
- [ ] FAQ : accordion ouvre/ferme
- [ ] Scroll fluide entre sections
- [ ] Tracking : events Google Analytics / Pixel fonctionnels
- [ ] Performance : LCP < 1s, Lighthouse 95+
- [ ] Responsive : CTA accessible sur mobile
- [ ] A/B : variantes fonctionnent si configurees

---

## Matrice de correspondance Boilerplate ↔ Type

Quand un boilerplate n'existe pas encore, l'orchestrateur clone le template generique
et laisse Claude Code generer le site from scratch en suivant les instructions BMAD ci-dessus.

```
site_type       → boilerplate_repo                    → fallback
────────────────────────────────────────────────────────────────
ecommerce       → ecommerce-boilerplate.git           → (existe)
vitrine         → vitrine-boilerplate.git              → template
restaurant      → restaurant-boilerplate.git           → template
booking         → booking-boilerplate.git              → template
blog            → blog-boilerplate.git                 → template
immo            → immo-boilerplate.git                 → template
portfolio       → portfolio-boilerplate.git            → template
landing         → landing-boilerplate.git              → template
```

---

## Notes d'implementation

1. Le `site_type` est stocke dans la table `projects` (colonne `site_type VARCHAR(20) DEFAULT 'vitrine'`)
2. L'orchestrateur lit `site_type` depuis le JSON du projet et choisit le bon boilerplate
3. Si le boilerplate n'existe pas, on clone le template et on genere from scratch
4. Les prix sont stockes dans la table `settings` avec les cles `price_vitrine`, `price_ecommerce`, etc.
5. Le prix par defaut affiche au client depend du `site_type` selectionne lors de l'inscription
