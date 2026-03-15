# Panel Administration

## Instructions pour l'agent IA

Ce document spécifie le panel d'administration complet : dashboard avec statistiques de revenus, gestion des produits, catégories, commandes et clients. Le panel admin utilise un thème sombre distinct de la boutique.

---

## 1. Layout admin (/admin)

### Structure

```
┌──────────────────────────────────────────────────────────┐
│ [Logo] E-Commerce Admin          Abdoul D. [Déconnexion] │
├────────────┬─────────────────────────────────────────────┤
│            │                                             │
│ Dashboard  │  CONTENU PRINCIPAL                          │
│ Produits   │                                             │
│ Catégories │  (varie selon la page)                      │
│ Commandes  │                                             │
│ Clients    │                                             │
│            │                                             │
│            │                                             │
│            │                                             │
│ ────────── │                                             │
│ Voir le    │                                             │
│ site →     │                                             │
│            │                                             │
└────────────┴─────────────────────────────────────────────┘
```

### Thème sombre

| Element | Couleur |
|---|---|
| Fond sidebar | #1a1a2e ou #0f172a (slate-900) |
| Fond contenu | #16213e ou #1e293b (slate-800) |
| Fond cartes | #1e293b ou #334155 (slate-700) |
| Texte principal | #f1f5f9 (slate-100) |
| Texte secondaire | #94a3b8 (slate-400) |
| Accent / liens | Couleur primaire du site |
| Bordures | #334155 (slate-700) |
| Hover sidebar | #334155 |

### Sidebar (desktop)

- Largeur fixe : 250px
- Logo en haut
- Liens de navigation avec icônes
- Lien actif : fond plus clair + bordure gauche couleur accent
- Lien "Voir le site" en bas → ouvre la boutique dans un nouvel onglet

### Mobile

- La sidebar devient un drawer (hamburger menu)
- Le header admin reste visible
- Les tableaux ont un scroll horizontal

### Protection

- Route protégée : rôle ADMIN requis
- Si CUSTOMER tente d'accéder → redirect vers /account
- Si non connecté → redirect vers /login

---

## 2. Dashboard (/admin)

### Cartes de statistiques (haut de page)

```
┌───────────────┐ ┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│ 💰 2 500 000  │ │ ✅ 1 800 000  │ │ ⏳ 700 000    │ │ 📦 85         │
│ CA Total      │ │ CA Payé       │ │ CA En attente │ │ Commandes     │
│ (FCFA)        │ │ (FCFA)        │ │ (FCFA)        │ │               │
│               │ │               │ │               │ │               │
│  ↑ 18.4%      │ │               │ │               │ │  15 ce mois   │
└───────────────┘ └───────────────┘ └───────────────┘ └───────────────┘

┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│ 👥 120        │ │ 🏷️ 45         │ │ ⚠️ 3          │
│ Clients       │ │ Produits      │ │ Stock bas     │
│               │ │ actifs        │ │               │
│  15 ce mois   │ │               │ │               │
└───────────────┘ └───────────────┘ └───────────────┘
```

### Tableau des clients (milieu de page)

**Titre** : "Statistiques clients"

| Client | Email | Commandé | Payé | En attente | Total payé |
|---|---|---|---|---|---|
| Abdoul Diallo | abdoul@email.com | 5 | 4 | 1 | 180 000 FCFA |
| Marie Kouassi | marie@email.com | 3 | 3 | 0 | 95 000 FCFA |
| Issa Camara | issa@email.com | 2 | 1 | 1 | 32 000 FCFA |

**Fonctionnalités** :
- Recherche par nom ou email
- Tri par colonnes (cliquable sur chaque en-tête)
- Pagination (20 par page)
- Formatage des montants : "180 000" avec séparateur de milliers

### Dernières commandes (bas de page)

**Titre** : "Dernières commandes"

| N° Commande | Client | Date | Montant | Statut | Paiement |
|---|---|---|---|---|---|
| CMD-20260315-0001 | Abdoul D. | 15/03 14:30 | 57 000 | ● En attente | ● En attente |
| CMD-20260314-0003 | Marie K. | 14/03 10:15 | 32 000 | ● Livrée | ● Payée |

**Fonctionnalités** :
- 5-10 dernières commandes
- Clic sur une ligne → détail de la commande
- Lien "Voir toutes les commandes →"

---

## 3. Gestion des produits (/admin/products)

### 3.1 Liste des produits

**Barre d'actions** :
```
[Rechercher...        ] [Statut ▼] [Catégorie ▼]  [+ Nouveau produit]
```

**Tableau** :

| | Produit | Catégorie | Prix | Stock | Statut | Vedette | Actions |
|---|---|---|---|---|---|---|---|
| [img] | T-Shirt Premium | T-Shirts | 15 000 | 25 | ● ACTIVE | ★ | [Modifier] [Supprimer] |
| [img] | Pantalon Classic | Pantalons | 25 000 | 3 ⚠️ | ● ACTIVE | ☆ | [Modifier] [Supprimer] |
| [img] | Robe d'été | Robes | 35 000 | 0 | ● DRAFT | ☆ | [Modifier] [Supprimer] |

**Fonctionnalités** :
- Filtre par statut (DRAFT/ACTIVE/ARCHIVED) avec compteur
- Filtre par catégorie (dropdown)
- Recherche par nom
- Tri par colonnes
- Pagination (20 par page)
- Badge stock bas : icône warning si stock <= lowStockAlert
- Toggle vedette : clic sur l'étoile → toggle isFeatured
- Changement de statut rapide : clic sur le badge statut → dropdown (DRAFT/ACTIVE/ARCHIVED)

### 3.2 Formulaire produit (création/édition)

**Page** : /admin/products/new ou /admin/products/[id]/edit

```
┌─────────────────────────────────────────────────────┐
│ [← Retour] NOUVEAU PRODUIT                          │
│                                                     │
│ ┌─────────────────────────────────────┐             │
│ │ Informations générales              │             │
│ │                                     │             │
│ │ Nom *          [T-Shirt Premium   ] │             │
│ │ Slug           t-shirt-premium      │ (auto)      │
│ │                                     │             │
│ │ Description *  [T-shirt en coton  ] │             │
│ │ courte         [égyptien 100%...  ] │ 45/160      │
│ │                                     │             │
│ │ Description    [Editeur rich text ] │             │
│ │ longue         [ou textarea HTML  ] │             │
│ │                                     │             │
│ │ Catégorie *    [T-Shirts        ▼] │             │
│ └─────────────────────────────────────┘             │
│                                                     │
│ ┌─────────────────────────────────────┐             │
│ │ Prix et stock                       │             │
│ │                                     │             │
│ │ Prix *         [15000      ] FCFA   │             │
│ │ Ancien prix    [20000      ] FCFA   │             │
│ │ Prix d'achat   [8000       ] FCFA   │ (interne)   │
│ │ Référence      [TSH-001    ]        │             │
│ │ Stock *        [25         ]        │             │
│ │ Alerte stock   [5          ]        │             │
│ └─────────────────────────────────────┘             │
│                                                     │
│ ┌─────────────────────────────────────┐             │
│ │ Images                              │             │
│ │                                     │             │
│ │ ┌──────┐ ┌──────┐ ┌──────┐ ┌────┐  │             │
│ │ │[img1]│ │[img2]│ │[img3]│ │ +  │  │             │
│ │ │ ★ X  │ │   X  │ │   X  │ │    │  │             │
│ │ └──────┘ └──────┘ └──────┘ └────┘  │             │
│ │                                     │             │
│ │ ★ = image principale               │             │
│ │ Glisser-déposer pour réordonner    │             │
│ └─────────────────────────────────────┘             │
│                                                     │
│ ┌─────────────────────────────────────┐             │
│ │ Publication                         │             │
│ │                                     │             │
│ │ Statut *       [DRAFT           ▼] │             │
│ │ Vedette        [☐] Produit vedette │             │
│ └─────────────────────────────────────┘             │
│                                                     │
│ [Annuler]                       [Enregistrer]       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Validation** :
- Nom : requis, 2-200 caractères
- Description courte : requis, max 160 caractères (compteur visible)
- Prix : requis, nombre positif
- Ancien prix : si renseigné, doit être supérieur au prix
- Stock : requis, nombre >= 0
- Catégorie : requis (dropdown)
- Images : au moins 1 image requise pour publier en ACTIVE

**Upload d'images** :
- Zone de drop ou bouton "+"
- Preview immédiate après sélection
- Progress bar pendant l'upload
- L'image marquée ★ est l'image principale (clic pour changer)
- Bouton X pour supprimer
- Drag & drop pour réordonner (change le sortOrder)
- Max 10 images par produit
- Formats : JPG, PNG, WebP
- Taille max : 5 Mo par image

### 3.3 Suppression de produit

**Modale de confirmation** :
```
┌────────────────────────────────────────┐
│ Supprimer le produit ?                 │
│                                        │
│ Êtes-vous sûr de vouloir supprimer    │
│ "T-Shirt Premium" ?                    │
│                                        │
│ Cette action est irréversible.         │
│                                        │
│ [Annuler]          [Supprimer]         │
└────────────────────────────────────────┘
```

- Si le produit a des commandes : archivage au lieu de suppression
- Toast de confirmation après la suppression

---

## 4. Gestion des catégories (/admin/categories)

### Liste

```
┌─────────────────────────────────────────────────────┐
│ CATEGORIES                      [+ Nouvelle catégorie]│
│                                                      │
│ | Catégorie     | Produits | Ordre | Actions       | │
│ |───────────────|──────────|───────|───────────────| │
│ | T-Shirts      | 12       | 1     | [✏️] [🗑️]   | │
│ | Pantalons     | 8        | 2     | [✏️] [🗑️]   | │
│ | Robes         | 15       | 3     | [✏️] [🗑️]   | │
│ | Accessoires   | 6        | 4     | [✏️] [🗑️]   | │
│ | Chaussures    | 4        | 5     | [✏️] [🗑️]   | │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Modale création/édition

```
┌────────────────────────────────────────┐
│ Nouvelle catégorie                     │
│                                        │
│ Nom *       [                    ]     │
│ Description [                    ]     │
│ Image       [Choisir un fichier]       │
│             [preview si uploadée]      │
│ Ordre       [1                  ]      │
│                                        │
│ [Annuler]              [Enregistrer]   │
└────────────────────────────────────────┘
```

### Suppression

- Impossible si la catégorie contient des produits
- Message : "Impossible de supprimer cette catégorie car elle contient X produits. Déplacez-les d'abord."

---

## 5. Gestion des commandes (/admin/orders)

### Barre de filtres

```
[Rechercher N° ou client...] [Statut ▼] [Paiement ▼] [Date du ▼] [au ▼]
```

### Tableau

| N° Commande | Client | Date | Articles | Montant | Statut | Paiement | Actions |
|---|---|---|---|---|---|---|---|
| CMD-20260315-0001 | Abdoul Diallo | 15/03/26 14:30 | 2 | 57 000 | ● En attente | ● En attente | [Voir] [Statut ▼] |
| CMD-20260314-0003 | Marie Kouassi | 14/03/26 10:15 | 1 | 32 000 | ● Livrée | ● Payée | [Voir] |

### Détail d'une commande (modale ou page)

```
┌─────────────────────────────────────────────────────┐
│ COMMANDE CMD-20260315-0001                           │
│ Créée le 15 mars 2026 à 14:30                       │
│                                                      │
│ Client : Abdoul Diallo (abdoul@email.com)            │
│ Téléphone : +225 07 01 23 45 67                     │
│                                                      │
│ ─── Articles ───                                     │
│ T-Shirt Premium       x2     30 000 FCFA            │
│ Pantalon Classic      x1     25 000 FCFA            │
│                                                      │
│ Sous-total :                  55 000 FCFA            │
│ Livraison :                    2 000 FCFA            │
│ Total :                       57 000 FCFA            │
│                                                      │
│ ─── Livraison ───                                    │
│ Abdoul Diallo                                        │
│ Angré 8ème tranche, Cocody, Abidjan                 │
│ Face à la pharmacie, portail bleu                   │
│                                                      │
│ ─── Paiement ───                                     │
│ Méthode : Orange Money                               │
│ Statut : ● En attente                               │
│ Référence : TXN-20260315-abc123                     │
│                                                      │
│ ─── Statut ───                                       │
│ Statut actuel : ● EN ATTENTE                        │
│ Changer vers : [CONFIRMED ▼] [Appliquer]            │
│                                                      │
│ Notes admin : [                              ]       │
│               [                              ]       │
│                                                      │
│ ─── Historique ───                                   │
│ 15/03 14:30 - Commande créée                        │
│ 15/03 14:35 - Paiement initié (Orange Money)        │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Changement de statut

**Dropdown avec transitions valides uniquement** :

| Statut actuel | Transitions proposées |
|---|---|
| PENDING | CONFIRMED, CANCELLED |
| CONFIRMED | PROCESSING, CANCELLED |
| PROCESSING | SHIPPED |
| SHIPPED | DELIVERED |
| DELIVERED | (aucune, état final) |
| CANCELLED | (aucune, état final) |

**Confirmation** :
- Modale de confirmation avant chaque changement
- Champ de notes optionnel (ex: "Envoyée via Chronopost")
- Toast de confirmation après le changement

---

## 6. Liste des clients (/admin/clients)

### Tableau

```
[Rechercher nom, email ou tél...        ]

| Client | Email | Tél | Commandes | Payé | En attente | Inscrit le |
|--------|-------|-----|-----------|------|------------|------------|
| Abdoul Diallo | abdoul@email.com | +225... | 5 | 180 000 | 32 000 | 10/01/26 |
| Marie Kouassi | marie@email.com | +225... | 3 | 95 000 | 0 | 15/01/26 |
```

**Fonctionnalités** :
- Recherche par nom, email ou téléphone
- Tri par colonnes
- Pagination
- Clic sur un client → voir ses commandes (filtre les commandes par ce client)
- Formatage des montants
- Badge spécial pour les meilleurs clients (> X commandes)

---

## 7. Données API pour le dashboard

### Endpoint GET /api/admin/dashboard

Requêtes SQL optimisées pour le dashboard :

```sql
-- Revenue total par statut de paiement
SELECT
  SUM(CASE WHEN p.status = 'COMPLETED' THEN p.amount ELSE 0 END) as paid,
  SUM(CASE WHEN p.status = 'PENDING' THEN p.amount ELSE 0 END) as pending,
  SUM(p.amount) as total
FROM payments p
JOIN orders o ON p.order_id = o.id
WHERE o.status != 'CANCELLED';

-- Revenue ce mois vs mois dernier
SELECT
  SUM(CASE WHEN p.paid_at >= DATE_TRUNC('month', NOW()) THEN p.amount ELSE 0 END) as this_month,
  SUM(CASE WHEN p.paid_at >= DATE_TRUNC('month', NOW() - INTERVAL '1 month')
    AND p.paid_at < DATE_TRUNC('month', NOW()) THEN p.amount ELSE 0 END) as last_month
FROM payments p
WHERE p.status = 'COMPLETED';

-- Commandes par statut
SELECT status, COUNT(*) as count
FROM orders
GROUP BY status;

-- Produits avec stock bas
SELECT COUNT(*)
FROM products
WHERE status = 'ACTIVE' AND stock <= low_stock_alert;
```

**Temps de réponse cible** : < 500ms
**Stratégie** : exécuter les requêtes en parallèle avec `Promise.all()`
