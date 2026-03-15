# Catalogue Produits et Catégories

## Instructions pour l'agent IA

Ce document spécifie tout le système de gestion des produits : affichage côté boutique, gestion admin, images, catégories et recherche. C'est le coeur fonctionnel du e-commerce.

---

## 1. Modèle de données produit

### Champs d'un produit

| Champ | Type | Obligatoire | Description |
|---|---|---|---|
| name | String | Oui | Nom du produit (ex: "T-Shirt Premium Coton") |
| slug | String | Auto-généré | URL-friendly (ex: "t-shirt-premium-coton") |
| description | String | Oui | Description courte (max 160 chars, pour SEO et aperçu) |
| longDescription | String | Non | Description longue en HTML (détail complet) |
| price | Decimal | Oui | Prix de vente en FCFA |
| comparePrice | Decimal | Non | Ancien prix (affiché barré si présent) |
| costPrice | Decimal | Non | Prix d'achat (usage interne uniquement, jamais affiché) |
| sku | String | Non | Référence produit unique |
| stock | Int | Oui | Quantité en stock |
| lowStockAlert | Int | Non | Seuil d'alerte stock bas (défaut: 5) |
| weight | Decimal | Non | Poids en kg (pour la livraison au poids) |
| categoryId | UUID | Oui | Catégorie du produit |
| status | Enum | Oui | DRAFT / ACTIVE / ARCHIVED |
| isFeatured | Boolean | Non | Produit vedette (affiché en page d'accueil) |
| sortOrder | Int | Non | Ordre d'affichage personnalisé |
| metaTitle | String | Non | Titre SEO (si différent du nom) |
| metaDescription | String | Non | Meta description SEO |
| viewCount | Int | Auto | Compteur de vues (incrémenté à chaque consultation) |
| images | Relation | Oui | Au moins 1 image |

### Statuts du produit

| Statut | Visible boutique | Visible admin | Achetable |
|---|---|---|---|
| DRAFT | Non | Oui | Non |
| ACTIVE | Oui | Oui | Oui (si stock > 0) |
| ARCHIVED | Non | Oui | Non |

---

## 2. Affichage boutique

### 2.1 Page catalogue (/products)

**Layout** :
- Barre de catégories en haut (scroll horizontal sur mobile)
- Compteur de résultats : "45 produits"
- Options de tri : "Trier par : Pertinence, Prix croissant, Prix décroissant, Nouveautés"
- Grille de cartes produit :
  - Desktop : 4 colonnes
  - Tablette : 3 colonnes
  - Mobile : 2 colonnes
- Pagination en bas : "< 1 2 3 ... 10 >"

**Paramètres d'URL** :
- `?category=t-shirts` : filtrer par catégorie (slug)
- `?search=chemise` : recherche textuelle
- `?page=2` : pagination
- `?sortBy=price&sortOrder=asc` : tri
- `?minPrice=5000&maxPrice=50000` : filtre prix (V2)
- Tous combinables : `?category=t-shirts&sortBy=price&sortOrder=asc&page=2`

**Comportement** :
- Les filtres modifient l'URL (navigation browser fonctionnelle)
- Chargement avec skeleton pendant la requête API
- Message "Aucun produit trouvé" avec bouton "Réinitialiser les filtres" si résultat vide
- 20 produits par page

### 2.2 Carte produit (composant réutilisable)

```
┌──────────────────┐
│                  │
│     [IMAGE]      │  ← Ratio 3:4 ou 1:1
│                  │
│   ┌─────────┐    │  ← Badge "Promo -30%" (si comparePrice)
│   │ -30%    │    │  ← Badge "Rupture" (si stock === 0)
│   └─────────┘    │
├──────────────────┤
│ Nom du produit   │  ← Max 2 lignes, text-overflow ellipsis
│                  │
│ 15 000 FCFA      │  ← Prix actuel en gras
│ ~~20 000 FCFA~~  │  ← Ancien prix barré, couleur grise
└──────────────────┘
```

**Calcul du pourcentage de réduction** :
```
reduction = Math.round(((comparePrice - price) / comparePrice) * 100)
```

**Formatage du prix** :
```
// 15000 → "15 000 FCFA"
price.toLocaleString('fr-FR') + ' FCFA'
```

### 2.3 Page détail produit (/products/[slug])

**Layout desktop** :
```
┌──────────────────────────────────────────────────┐
│ Accueil > Catégorie > Nom du produit (breadcrumb)│
├──────────────────────┬───────────────────────────┤
│                      │                           │
│   [IMAGE PRINCIPALE] │  Nom du produit           │
│                      │                           │
│                      │  15 000 FCFA              │
│                      │  ~~20 000 FCFA~~ -30%     │
│                      │                           │
│ [mini1] [mini2] [m3] │  En stock (25 disponibles)│
│                      │                           │
│                      │  Description courte...    │
│                      │                           │
│                      │  Quantité: [- 1 +]        │
│                      │                           │
│                      │  [AJOUTER AU PANIER]      │
│                      │                           │
│                      │  📱 Partager via WhatsApp │
├──────────────────────┴───────────────────────────┤
│                                                  │
│  Description détaillée                           │
│  (HTML rendu depuis longDescription)             │
│                                                  │
└──────────────────────────────────────────────────┘
```

**Layout mobile** : image en haut (pleine largeur), infos en dessous

**Galerie d'images** :
- Image principale grande
- Miniatures en dessous (scroll horizontal si > 4)
- Clic sur une miniature → affiche en grand
- Swipe sur mobile pour changer d'image (V2)
- Zoom au clic (V2)

**Sélecteur de quantité** :
- Boutons - et +
- Input numérique au centre
- Minimum : 1
- Maximum : stock disponible
- Bouton - désactivé si quantité = 1
- Bouton + désactivé si quantité = stock

**Indicateur de stock** :
| Stock | Affichage | Couleur |
|---|---|---|
| > 10 | "En stock" | Vert |
| 1-10 | "Plus que X en stock" | Orange |
| 0 | "Rupture de stock" | Rouge |

**Bouton "Ajouter au panier"** :
- Large, pleine largeur sur mobile
- Couleur primaire
- Au clic : ajoute au store Zustand, ouvre le drawer panier, affiche toast de confirmation
- Si rupture de stock : bouton grisé "Indisponible"
- Loading state pendant l'ajout

**SEO** :
- `<title>` : nom du produit + nom du site
- `<meta description>` : description courte du produit
- Open Graph : image principale du produit
- Structured data (JSON-LD) : Product schema

---

## 3. Gestion des images

### Upload
- Formats acceptés : JPG, PNG, WebP
- Taille max : 5 Mo par image
- Résolution recommandée : minimum 800x800px
- Nombre max par produit : 10

### Stockage
- Répertoire : `uploads/products/`
- Nom de fichier : `{timestamp}-{uuid}.{extension}`
- Jamais de nom original (sécurité)

### Affichage
- Image principale : `isPrimary = true`
- Ordre des images : trié par `sortOrder`
- Si pas d'image : afficher un placeholder gris avec le nom du produit

### Optimisation (Next.js)
```tsx
<Image
  src={product.primaryImage.url}
  alt={product.primaryImage.alt || product.name}
  width={400}
  height={400}
  quality={80}
  priority={isAboveFold} // true pour les premiers produits visibles
  className="object-cover"
/>
```

---

## 4. Catégories

### Modèle

| Champ | Type | Description |
|---|---|---|
| name | String | Nom de la catégorie (unique) |
| slug | String | URL-friendly (auto-généré) |
| description | String | Description (optionnelle) |
| image | String | Image de la catégorie |
| parentId | UUID | Parent (pour sous-catégories) |
| sortOrder | Int | Ordre d'affichage |
| isActive | Boolean | Active/inactive |

### Affichage

**Barre de catégories** (page catalogue) :
```
[Tout] [T-Shirts] [Pantalons] [Robes] [Accessoires] [Chaussures]
  ^active                                             ^scroll →
```
- Pill/badge pour chaque catégorie
- "Tout" = pas de filtre catégorie
- Catégorie active visuellement distinguée (fond coloré ou souligné)
- Scroll horizontal sur mobile avec indicateur de défilement

**Page d'accueil** (section catégories) :
- Cartes avec image de la catégorie + nom
- Lien vers le catalogue filtré

### Sous-catégories (si applicable)
- Affichage en menu déroulant sur le hover de la catégorie parent
- Sur mobile : expansion en accordéon
- URL : `?category=parent-slug&subcategory=child-slug`

---

## 5. Administration des produits

### 5.1 Liste des produits (admin)

**Tableau** :

| Image | Nom | Catégorie | Prix | Stock | Statut | Vedette | Actions |
|---|---|---|---|---|---|---|---|
| [mini] | T-Shirt Premium | T-Shirts | 15 000 | 25 | ACTIVE | ★ | Modifier / Supprimer |

**Fonctionnalités** :
- Filtres : par statut (DRAFT/ACTIVE/ARCHIVED), par catégorie, recherche par nom
- Tri par colonnes (nom, prix, stock, date de création)
- Pagination (20 par page)
- Actions rapides : toggle featured (clic sur l'étoile), changer le statut
- Badge stock : rouge si stock <= lowStockAlert

### 5.2 Formulaire création/édition

**Champs du formulaire** :

| Champ | Type input | Requis | Notes |
|---|---|---|---|
| Nom | text | Oui | Le slug est généré automatiquement |
| Description courte | textarea | Oui | Compteur de caractères (max 160) |
| Description longue | rich text / textarea | Non | HTML simple accepté |
| Prix | number | Oui | Minimum 0 |
| Ancien prix | number | Non | Doit être > prix si renseigné |
| Prix d'achat | number | Non | Usage interne |
| Référence (SKU) | text | Non | Unique si renseigné |
| Stock | number | Oui | Minimum 0 |
| Seuil alerte stock | number | Non | Défaut 5 |
| Catégorie | select | Oui | Dropdown des catégories |
| Statut | select | Oui | DRAFT / ACTIVE / ARCHIVED |
| Produit vedette | checkbox | Non | Défaut false |
| Images | file upload | Oui (min 1) | Multi-upload, drag & drop |

**Upload d'images dans le formulaire** :
- Zone de drag & drop ou bouton "Ajouter des images"
- Preview des images sélectionnées avant upload
- Possibilité de réordonner par drag & drop
- Marquer une image comme principale (clic droit ou bouton étoile)
- Supprimer une image (bouton X sur la preview)
- Progress bar pendant l'upload

### 5.3 Suppression

- Si le produit n'a aucune commande : suppression physique
- Si le produit a des commandes : passage en ARCHIVED (soft delete)
- Confirmation obligatoire via modale : "Etes-vous sûr de vouloir supprimer ce produit ?"

---

## 6. Recherche de produits

### Autocomplete (barre de recherche du header)

**Comportement** :
1. L'utilisateur tape dans l'input
2. Après 1 caractère : appel API GET /api/products/search?q=xxx (debounce 300ms)
3. Affichage des suggestions dans un dropdown
4. Chaque suggestion : miniature image + nom + prix
5. Max 5 suggestions
6. Clic sur une suggestion → page détail du produit
7. Appuyer sur Entrée → page catalogue avec ?search=xxx
8. Navigation au clavier (flèches, Entrée, Escape)

**Recherche full-text** :
```sql
-- Prisma query
WHERE name ILIKE '%terme%' AND status = 'ACTIVE'
ORDER BY
  CASE WHEN name ILIKE 'terme%' THEN 0 ELSE 1 END, -- Priorité au début du mot
  name ASC
LIMIT 5
```

### Recherche catalogue (page catalogue)

- Input de recherche dans le header
- Résultats affichés dans la grille du catalogue
- Combinable avec le filtre catégorie
- URL : `?search=chemise&category=t-shirts`
- Message "X résultats pour 'chemise'" en haut de la grille

---

## 7. Produits vedettes

### Sélection
- L'admin peut marquer un produit comme "vedette" (`isFeatured = true`)
- Maximum recommandé : 4-8 produits vedettes
- Pas de limite technique (mais le design prévoit 4-8 en page d'accueil)

### Affichage
- Endpoint dédié : GET /api/products/featured?limit=8
- Affichés en page d'accueil dans une section dédiée
- Tri par sortOrder puis par date de création (plus récent en premier)

---

## 8. Gestion du stock

### Règles
- Le stock est décrémenté à la création de commande (pas au paiement)
- Si une commande est annulée : le stock est ré-incrémenté
- Un produit avec stock = 0 est affiché avec le badge "Rupture de stock"
- Un produit avec stock = 0 ne peut pas être ajouté au panier
- L'alerte stock bas est visible uniquement dans l'admin

### Validation du stock
1. **Ajout au panier** : vérifier que stock >= quantité demandée
2. **Modification quantité panier** : vérifier que stock >= nouvelle quantité
3. **Création de commande** : re-vérifier le stock de chaque article (peut avoir changé)
4. **Si stock insuffisant à la commande** : erreur 400 avec le nom du produit et le stock disponible

---

## 9. Slug et URL

### Génération du slug
```typescript
function generateSlug(name: string): string {
  return name
    .toLowerCase()
    .normalize('NFD')
    .replace(/[\u0300-\u036f]/g, '') // Supprimer les accents
    .replace(/[^a-z0-9]+/g, '-')     // Remplacer les non-alphanum par des tirets
    .replace(/^-+|-+$/g, '');        // Supprimer les tirets en début/fin
}

// "T-Shirt Premium Coton" → "t-shirt-premium-coton"
// "Robe d'été élégante"   → "robe-d-ete-elegante"
```

### Unicité du slug
- Si le slug existe déjà, ajouter un suffixe numérique : `-1`, `-2`, etc.
- Exemple : "t-shirt-premium-coton", "t-shirt-premium-coton-1"
