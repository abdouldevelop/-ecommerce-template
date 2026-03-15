# Cahier des Charges - Projet E-Commerce

## Instructions pour l'agent IA

Ce document doit être rempli après le questionnaire client (`questionnaire-client.md`). Remplacer tous les champs `[A REMPLIR]` par les informations du client. Ce document sert de référence unique pour tout le projet.

---

## 1. Contexte du projet

### 1.1 Présentation du client

| Information | Détail |
|---|---|
| Nom de l'entreprise | [A REMPLIR] |
| Secteur d'activité | [A REMPLIR] |
| Localisation | [A REMPLIR] |
| Site web existant | [A REMPLIR - URL ou "Aucun"] |
| Contact principal | [A REMPLIR - Nom, téléphone, email] |
| Décideur(s) | [A REMPLIR] |

### 1.2 Contexte et motivation

[A REMPLIR - Pourquoi le client veut un site e-commerce ? Quel problème résout-il ?]

### 1.3 Cible utilisateurs

| Critère | Description |
|---|---|
| Tranche d'âge | [A REMPLIR] |
| Localisation | [A REMPLIR] |
| Appareils principaux | [A REMPLIR - Mobile / Desktop / Les deux] |
| Niveau technique | [A REMPLIR - Habitué aux achats en ligne ou non] |

---

## 2. Objectifs du projet

### 2.1 Objectifs business

- [ ] Vendre des produits en ligne
- [ ] Augmenter la visibilité de la marque
- [ ] Toucher une clientèle plus large géographiquement
- [ ] Réduire les coûts de distribution
- [ ] Autre : [A REMPLIR]

### 2.2 Objectifs techniques

- [ ] Site performant (chargement < 3 secondes)
- [ ] Responsive mobile-first
- [ ] Paiement sécurisé en ligne
- [ ] Panel d'administration autonome
- [ ] Autre : [A REMPLIR]

### 2.3 KPIs de succès

| Indicateur | Objectif |
|---|---|
| Nombre de visites/mois | [A REMPLIR] |
| Taux de conversion | [A REMPLIR - Standard e-commerce : 1-3%] |
| Panier moyen | [A REMPLIR] |
| Nombre de commandes/mois | [A REMPLIR] |

---

## 3. Périmètre fonctionnel

### 3.1 MVP (Version 1 - Lancement)

#### Boutique en ligne
- [ ] Page d'accueil avec slider et produits vedettes
- [ ] Catalogue produits avec pagination
- [ ] Page détail produit avec galerie d'images
- [ ] Filtrage par catégorie
- [ ] Recherche de produits : [A REMPLIR - Simple / Avec autocomplete]

#### Panier et commande
- [ ] Ajout/suppression/modification quantité dans le panier
- [ ] Panier persistant (localStorage + serveur si connecté)
- [ ] Processus de commande (checkout)
- [ ] Récapitulatif avant paiement

#### Paiement
- [ ] Prestataire : [A REMPLIR - PaiementPro / CinetPay]
- [ ] Moyens de paiement : [A REMPLIR - Carte, Orange Money, MTN MoMo, Wave, etc.]
- [ ] Page de résultat (succès/échec)
- [ ] Paiement à la livraison : [A REMPLIR - Oui / Non]

#### Espace client
- [ ] Inscription / connexion
- [ ] Historique de commandes
- [ ] Suivi de statut de commande
- [ ] Modification du profil

#### Administration
- [ ] Dashboard avec statistiques de ventes
- [ ] Gestion des produits (CRUD + images)
- [ ] Gestion des catégories
- [ ] Gestion des commandes (voir, changer statut)
- [ ] Liste des clients

#### Pages statiques
- [ ] A propos
- [ ] Conditions générales de vente (CGV)
- [ ] Politique de confidentialité
- [ ] Contact
- [ ] FAQ : [A REMPLIR - Oui / Non]

### 3.2 Version 2 (Post-lancement)

- [ ] Système d'avis produits
- [ ] Liste de souhaits (wishlist)
- [ ] Newsletter
- [ ] Blog
- [ ] Programme de fidélité
- [ ] Codes promo/coupons
- [ ] Multi-langues
- [ ] Application mobile
- [ ] Autre : [A REMPLIR]

---

## 4. Spécifications techniques

### 4.1 Stack technique

| Composant | Technologie |
|---|---|
| Backend | NestJS (Node.js) |
| Frontend | Next.js 15 (App Router) |
| Base de données | PostgreSQL 16 |
| ORM | Prisma |
| Style | Tailwind CSS |
| Monorepo | Turborepo |
| Paiement | [A REMPLIR - PaiementPro / CinetPay] |
| Serveur | [A REMPLIR - VPS OVH / Contabo / etc.] |
| Reverse proxy | Nginx |
| Process manager | PM2 |

### 4.2 Hébergement

| Paramètre | Valeur |
|---|---|
| Fournisseur | [A REMPLIR] |
| Nom de domaine | [A REMPLIR] |
| SSL | Let's Encrypt (gratuit) |
| RAM minimum | 2 Go |
| CPU minimum | 2 vCPU |
| Stockage | [A REMPLIR - Minimum 20 Go SSD] |
| OS | Ubuntu 22.04 / Debian 12 |

### 4.3 Performance

| Critère | Cible |
|---|---|
| Temps de chargement page | < 3 secondes |
| Temps de réponse API | < 500ms |
| Uptime | 99.5% |
| Utilisateurs simultanés | [A REMPLIR] |

---

## 5. Design et identité visuelle

### 5.1 Charte graphique

| Elément | Valeur |
|---|---|
| Couleur primaire | [A REMPLIR - ex: #1a365d] |
| Couleur secondaire | [A REMPLIR - ex: #d4a843] |
| Couleur d'accentuation | [A REMPLIR - ex: #e53e3e] |
| Couleur de fond | [A REMPLIR - ex: #ffffff] |
| Couleur de texte | [A REMPLIR - ex: #1a202c] |
| Police principale | [A REMPLIR - ex: Montserrat] |
| Police secondaire | [A REMPLIR - ex: Open Sans] |
| Style général | [A REMPLIR - Minimaliste / Luxe / Coloré / Corporate] |

### 5.2 Logo

| Paramètre | Valeur |
|---|---|
| Logo fourni | [A REMPLIR - Oui / Non] |
| Format(s) | [A REMPLIR - PNG, SVG, AI] |
| Déclinaisons | [A REMPLIR - Couleur, Blanc, Noir] |

### 5.3 Références visuelles

Sites appréciés par le client :
1. [A REMPLIR - URL + ce qui plaît]
2. [A REMPLIR - URL + ce qui plaît]
3. [A REMPLIR - URL + ce qui plaît]

---

## 6. Catalogue produits

### 6.1 Structure

| Paramètre | Valeur |
|---|---|
| Nombre de produits au lancement | [A REMPLIR] |
| Nombre de catégories | [A REMPLIR] |
| Sous-catégories | [A REMPLIR - Oui / Non] |
| Variantes (taille, couleur) | [A REMPLIR - Oui / Non] |
| Produits vedettes | [A REMPLIR - Oui / Non, combien] |

### 6.2 Liste des catégories

| N | Catégorie | Sous-catégories | Nb produits estimé |
|---|---|---|---|
| 1 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| 2 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| 3 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| 4 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| 5 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |

### 6.3 Contenu produit

Chaque produit nécessite :
- [ ] Nom du produit
- [ ] Description courte (max 160 caractères)
- [ ] Description longue (HTML)
- [ ] Prix en [A REMPLIR - FCFA / EUR]
- [ ] Prix barré (ancien prix, optionnel)
- [ ] Photos (minimum [A REMPLIR] par produit)
- [ ] Catégorie
- [ ] Stock initial
- [ ] Poids (si livraison au poids) : [A REMPLIR - Oui / Non]

### 6.4 Etat de préparation du contenu

- [ ] Photos produits prêtes
- [ ] Descriptions produits rédigées
- [ ] Prix définis
- [ ] Stocks comptés
- [ ] Catégories validées

---

## 7. Livraison

### 7.1 Configuration

| Paramètre | Valeur |
|---|---|
| Zone de livraison | [A REMPLIR] |
| Mode de calcul frais | [A REMPLIR - Forfait / Par zone / Par poids] |
| Délai moyen | [A REMPLIR] |
| Livraison gratuite à partir de | [A REMPLIR - Montant ou "Non"] |
| Retrait en boutique | [A REMPLIR - Oui / Non + adresse] |

### 7.2 Grille tarifaire

| Zone | Frais | Délai |
|---|---|---|
| [A REMPLIR - ex: Abidjan] | [A REMPLIR - ex: 2000 FCFA] | [A REMPLIR - ex: 24h] |
| [A REMPLIR - ex: Hors Abidjan] | [A REMPLIR - ex: 5000 FCFA] | [A REMPLIR - ex: 48-72h] |

### 7.3 Politique de retour

[A REMPLIR - Conditions de retour, délai, remboursement ou échange]

---

## 8. Contraintes

### 8.1 Contraintes techniques
- [A REMPLIR - ex: Doit fonctionner sur connexion 3G]
- [A REMPLIR - ex: Compatible avec les navigateurs UCBrowser et Opera Mini]

### 8.2 Contraintes légales
- [ ] Mentions légales obligatoires
- [ ] CGV conformes à la législation [A REMPLIR - pays]
- [ ] RGPD / Protection des données : [A REMPLIR - Applicable / Non applicable]
- [ ] Autre : [A REMPLIR]

### 8.3 Contraintes organisationnelles
- [A REMPLIR - ex: Le client ne parle pas anglais, tout doit être en français]
- [A REMPLIR - ex: Pas d'équipe technique interne]

---

## 9. Planning

### 9.1 Jalons

| Phase | Durée estimée | Date début | Date fin |
|---|---|---|---|
| Découverte et validation CDC | 2-3 jours | [A REMPLIR] | [A REMPLIR] |
| Setup projet et architecture | 1-2 jours | [A REMPLIR] | [A REMPLIR] |
| Backend (Auth + Produits + Panier) | 3-5 jours | [A REMPLIR] | [A REMPLIR] |
| Frontend (Boutique + Checkout) | 4-6 jours | [A REMPLIR] | [A REMPLIR] |
| Intégration paiement | 2-3 jours | [A REMPLIR] | [A REMPLIR] |
| Panel admin | 3-4 jours | [A REMPLIR] | [A REMPLIR] |
| Tests et corrections | 2-3 jours | [A REMPLIR] | [A REMPLIR] |
| Mise en production | 1 jour | [A REMPLIR] | [A REMPLIR] |
| **Total estimé** | **16-27 jours** | | |

### 9.2 Budget

| Poste | Montant |
|---|---|
| Développement | [A REMPLIR] |
| Hébergement (annuel) | [A REMPLIR] |
| Nom de domaine (annuel) | [A REMPLIR] |
| Prestataire paiement (frais) | [A REMPLIR - Généralement 2-3% par transaction] |
| Maintenance (mensuel) | [A REMPLIR] |
| **Total** | **[A REMPLIR]** |

---

## 10. Validation

### Signatures

| Rôle | Nom | Date | Signature |
|---|---|---|---|
| Client | [A REMPLIR] | [A REMPLIR] | __________ |
| Développeur | [A REMPLIR] | [A REMPLIR] | __________ |

### Historique des modifications

| Version | Date | Auteur | Modifications |
|---|---|---|---|
| 1.0 | [A REMPLIR] | [A REMPLIR] | Version initiale |
