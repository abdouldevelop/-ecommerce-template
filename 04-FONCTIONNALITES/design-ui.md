# Design System et UI

## Instructions pour l'agent IA

Ce document définit le design system complet : couleurs, typographie, composants, breakpoints, thèmes et spécifications visuelles pour l'ensemble du site e-commerce.

**Principe** : le design system doit être cohérent, accessible et performant. Utiliser Tailwind CSS pour l'implémentation.

---

## 1. Couleurs

### Palette de base (à adapter selon le client)

```
Couleur primaire    : #1a365d  (Bleu marine - confiance, sérieux)
Couleur secondaire  : #d4a843  (Or/doré - luxe, qualité)
Couleur accent      : #e53e3e  (Rouge - urgence, promotions)
```

### Palette complète Tailwind

```javascript
// tailwind.config.ts
{
  theme: {
    extend: {
      colors: {
        primary: {
          50:  '#eef2ff',
          100: '#dbe4ff',
          200: '#bac8ff',
          300: '#91a7ff',
          400: '#748ffc',
          500: '#1a365d',  // BASE
          600: '#162d4e',
          700: '#122440',
          800: '#0e1b31',
          900: '#0a1223',
        },
        secondary: {
          50:  '#fdf8ed',
          100: '#faefd2',
          200: '#f5dfa5',
          300: '#eecf78',
          400: '#e7bf4b',
          500: '#d4a843',  // BASE
          600: '#b88f3a',
          700: '#9c7631',
          800: '#805d28',
          900: '#64441f',
        },
        accent: {
          500: '#e53e3e',  // Rouge pour les promos
          600: '#c53030',
        },
        success: '#22c55e',
        warning: '#f59e0b',
        error:   '#ef4444',
        info:    '#3b82f6',
      },
    },
  },
}
```

### Couleurs fonctionnelles

| Utilisation | Couleur | Tailwind |
|---|---|---|
| Fond principal (boutique) | #ffffff | bg-white |
| Fond secondaire | #f8fafc | bg-slate-50 |
| Fond cartes | #ffffff | bg-white |
| Texte principal | #1e293b | text-slate-800 |
| Texte secondaire | #64748b | text-slate-500 |
| Texte désactivé | #cbd5e1 | text-slate-300 |
| Bordures | #e2e8f0 | border-slate-200 |
| Prix actuel | #1e293b | text-slate-800 font-bold |
| Prix barré | #94a3b8 | text-slate-400 line-through |
| Badge promo | #e53e3e / #fee2e2 | bg-red-100 text-red-700 |
| Badge stock | #22c55e / #dcfce7 | bg-green-100 text-green-700 |
| Badge rupture | #ef4444 / #fef2f2 | bg-red-100 text-red-700 |
| Bouton primaire | #1a365d | bg-primary-500 text-white |
| Bouton primaire hover | #162d4e | hover:bg-primary-600 |
| Liens | #1a365d | text-primary-500 |
| Liens hover | #122440 | hover:text-primary-700 |

### Thème admin (sombre)

| Utilisation | Couleur | Tailwind |
|---|---|---|
| Fond sidebar | #0f172a | bg-slate-900 |
| Fond contenu | #1e293b | bg-slate-800 |
| Fond cartes | #334155 | bg-slate-700 |
| Texte principal | #f1f5f9 | text-slate-100 |
| Texte secondaire | #94a3b8 | text-slate-400 |
| Bordures | #475569 | border-slate-600 |
| Hover sidebar | #334155 | hover:bg-slate-700 |

---

## 2. Typographie

### Polices recommandées

```javascript
// app/layout.tsx
import { Montserrat, Open_Sans } from 'next/font/google';

const montserrat = Montserrat({
  subsets: ['latin'],
  variable: '--font-heading',
  display: 'swap',
});

const openSans = Open_Sans({
  subsets: ['latin'],
  variable: '--font-body',
  display: 'swap',
});
```

### Echelle typographique

| Element | Police | Taille | Poids | Tailwind |
|---|---|---|---|---|
| H1 (page title) | Montserrat | 32px / 2rem | Bold (700) | text-3xl font-bold font-heading |
| H2 (section title) | Montserrat | 24px / 1.5rem | Semibold (600) | text-2xl font-semibold font-heading |
| H3 (card title) | Montserrat | 20px / 1.25rem | Semibold (600) | text-xl font-semibold font-heading |
| Body | Open Sans | 16px / 1rem | Regular (400) | text-base font-body |
| Body small | Open Sans | 14px / 0.875rem | Regular (400) | text-sm |
| Caption | Open Sans | 12px / 0.75rem | Regular (400) | text-xs |
| Prix | Montserrat | 18px / 1.125rem | Bold (700) | text-lg font-bold font-heading |
| Prix promo | Montserrat | 14px / 0.875rem | Regular (400) | text-sm line-through |
| Bouton | Montserrat | 16px / 1rem | Semibold (600) | text-base font-semibold |
| Bouton petit | Montserrat | 14px / 0.875rem | Medium (500) | text-sm font-medium |
| Navigation | Montserrat | 14px / 0.875rem | Medium (500) | text-sm font-medium |

---

## 3. Espacement

### Echelle de spacing

Utiliser l'échelle Tailwind par défaut (multiples de 4px) :

| Valeur | Pixels | Utilisation |
|---|---|---|
| 1 | 4px | Micro-espacement (entre icône et texte) |
| 2 | 8px | Padding interne des badges |
| 3 | 12px | Gap entre éléments dans une ligne |
| 4 | 16px | Padding des cartes (mobile) |
| 5 | 20px | Gap entre les cartes |
| 6 | 24px | Padding des cartes (desktop) |
| 8 | 32px | Marges entre les sections |
| 10 | 40px | Padding du contenu principal |
| 12 | 48px | Espacement entre les grandes sections |
| 16 | 64px | Marge top/bottom des sections de page |
| 20 | 80px | Espacement entre les sections majeures (homepage) |

### Layout

| Element | Valeur |
|---|---|
| Max-width du contenu | 1280px (max-w-screen-xl) |
| Padding horizontal (desktop) | 32px (px-8) |
| Padding horizontal (mobile) | 16px (px-4) |
| Gap grille produits | 20px (gap-5) |
| Hauteur header | 64-72px |
| Largeur sidebar admin | 250px |

---

## 4. Ombres et élévation

```javascript
// tailwind.config.ts
{
  theme: {
    extend: {
      boxShadow: {
        'card': '0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px -1px rgba(0, 0, 0, 0.1)',
        'card-hover': '0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1)',
        'dropdown': '0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1)',
        'modal': '0 25px 50px -12px rgba(0, 0, 0, 0.25)',
      },
    },
  },
}
```

| Niveau | Shadow | Utilisation |
|---|---|---|
| 0 | Aucune | Etat normal des éléments plats |
| 1 | shadow-card | Cartes produit, cartes stats |
| 2 | shadow-card-hover | Hover des cartes |
| 3 | shadow-dropdown | Dropdown, autocomplete, menu |
| 4 | shadow-modal | Modales, drawers |

---

## 5. Breakpoints (responsive)

### Points de rupture

| Nom | Taille | Tailwind | Colonnes grille |
|---|---|---|---|
| Mobile | < 640px | (default) | 2 colonnes produits |
| Tablette | >= 640px | sm: | 2-3 colonnes |
| Desktop petit | >= 768px | md: | 3 colonnes |
| Desktop | >= 1024px | lg: | 4 colonnes |
| Grand écran | >= 1280px | xl: | 4 colonnes |

### Grille produits responsive

```html
<div class="grid grid-cols-2 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4 md:gap-5">
  <ProductCard />
  <ProductCard />
  ...
</div>
```

### Mobile-first

Toujours développer pour le mobile d'abord, puis ajouter des styles pour les écrans plus grands :

```css
/* Mobile (default) */
.element { padding: 16px; }

/* Tablette et plus */
@media (min-width: 640px) { .element { padding: 24px; } }

/* Desktop et plus */
@media (min-width: 1024px) { .element { padding: 32px; } }
```

---

## 6. Composants

### 6.1 Boutons

**Variantes** :

```
Primary:    [████████████████] bg-primary-500, text-white, hover:bg-primary-600
Secondary:  [████████████████] bg-white, border-primary-500, text-primary-500, hover:bg-primary-50
Danger:     [████████████████] bg-red-500, text-white, hover:bg-red-600
Ghost:      [████████████████] bg-transparent, text-primary-500, hover:bg-primary-50
```

**Tailles** :

| Taille | Padding | Font size | Classe |
|---|---|---|---|
| Small | px-3 py-1.5 | text-sm | btn-sm |
| Medium | px-4 py-2 | text-base | btn-md (default) |
| Large | px-6 py-3 | text-lg | btn-lg |
| Full width | w-full py-3 | text-base | btn-full |

**Etats** :

| Etat | Style |
|---|---|
| Default | Couleur de base |
| Hover | Couleur plus foncée + cursor pointer |
| Active | Couleur encore plus foncée + scale(0.98) |
| Disabled | opacity-50 + cursor-not-allowed |
| Loading | Spinner à gauche du texte + disabled |

### 6.2 Inputs

```
┌────────────────────────────────────┐
│ Label                              │
│ ┌──────────────────────────────┐   │
│ │ Placeholder text...          │   │ border-slate-300, rounded-lg
│ └──────────────────────────────┘   │ focus:border-primary-500, focus:ring-2
│ Message d'aide ou d'erreur         │ text-sm text-slate-500 ou text-red-500
└────────────────────────────────────┘
```

**Variantes** :

| Etat | Border | Texte aide |
|---|---|---|
| Default | border-slate-300 | text-slate-500 |
| Focus | border-primary-500 + ring | - |
| Error | border-red-500 + ring-red | text-red-500 |
| Disabled | bg-slate-100, opacity-70 | - |

### 6.3 Badges / Pills

```
[En stock]     bg-green-100 text-green-700 px-2.5 py-0.5 rounded-full text-xs font-medium
[En attente]   bg-yellow-100 text-yellow-700
[Rupture]      bg-red-100 text-red-700
[Promo -30%]   bg-red-500 text-white
[DRAFT]        bg-slate-100 text-slate-700
[ACTIVE]       bg-green-100 text-green-700
[ARCHIVED]     bg-slate-100 text-slate-500
```

### 6.4 Cartes

```
┌──────────────────────────────────┐
│                                  │  bg-white
│  Contenu de la carte             │  rounded-xl (border-radius: 12px)
│                                  │  shadow-card
│                                  │  hover:shadow-card-hover
│                                  │  transition-shadow duration-200
│                                  │  overflow-hidden (pour les images)
└──────────────────────────────────┘
```

### 6.5 Modales

```
Overlay : bg-black/50 (fond noir semi-transparent)
┌────────────────────────────────┐
│ Titre                    [X]   │  bg-white rounded-xl
├────────────────────────────────┤  shadow-modal
│                                │  max-w-md mx-auto
│ Contenu de la modale           │  p-6
│                                │  animate-scale-in
├────────────────────────────────┤
│         [Annuler] [Confirmer]  │
└────────────────────────────────┘
```

### 6.6 Skeleton loaders

```
┌────────────────────────────┐
│ ████████████████████████   │  bg-slate-200
│ ████████████████████████   │  animate-pulse
│ ████████████████████████   │  rounded
│                            │
│ ████████████               │
│ ████████                   │
└────────────────────────────┘
```

### 6.7 Toast notifications

```
Position : fixed top-4 right-4 z-50

┌──────────────────────────────────────┐
│ ✓ Produit ajouté au panier          │  bg-green-500 text-white
│                                [X]   │  rounded-lg shadow-lg
└──────────────────────────────────────┘  animate-slide-in-right
                                          auto-dismiss: 3-5 secondes
```

---

## 7. Header

### Structure desktop

```
┌──────────────────────────────────────────────────────────┐
│ [Logo]        [🔍 Rechercher...              ] [👤] [🛒3]│
└──────────────────────────────────────────────────────────┘
```

**Style** :
- Position : fixed (sticky au scroll)
- Fond : blanc ou gradient de la couleur primaire
- Hauteur : 64-72px
- Z-index : 50
- Ombre : shadow-sm au scroll (pas d'ombre en haut de page)
- Transition de fond au scroll (optionnel)

### Structure mobile

```
┌──────────────────────────────────┐
│ [☰]       [Logo]          [🛒3] │
├──────────────────────────────────┤
│ [🔍 Rechercher...              ] │
└──────────────────────────────────┘
```

**Style** :
- 2 lignes : navigation + recherche
- Hauteur totale : 110-120px
- La barre de recherche peut se cacher au scroll down et réapparaître au scroll up

---

## 8. Footer

### Structure

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  [Logo]              Liens utiles    Contact             │
│  Description         • Accueil       • Adresse           │
│  courte du site      • Produits      • Téléphone         │
│                      • A propos      • Email             │
│  Réseaux sociaux     • CGV           • WhatsApp          │
│  [FB] [IG] [TK]      • FAQ                               │
│                                                          │
│ ──────────────────────────────────────────────────────── │
│  © 2026 NomDuSite. Tous droits réservés.                │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Style** :
- Fond : couleur primaire sombre ou slate-900
- Texte : slate-300
- Liens : slate-300, hover: white
- Padding : py-16 sur desktop, py-8 sur mobile
- 3-4 colonnes sur desktop, 1 colonne sur mobile (empilées)

### Bouton WhatsApp flottant

```
Position : fixed bottom-6 right-6 z-40

    ┌────────┐
    │ [WhatsApp│  bg-green-500 text-white
    │  icon]  │  rounded-full w-14 h-14
    └────────┘  shadow-lg hover:bg-green-600
                animate-bounce (subtle, optionnel)
```

- Lien : `https://wa.me/NUMERO?text=Bonjour,%20je%20viens%20de%20votre%20site`
- Visible sur toutes les pages (sauf admin)
- Au-dessus du footer quand on scrolle en bas

---

## 9. Slider / Carrousel (Page d'accueil)

### Spécifications

| Paramètre | Valeur |
|---|---|
| Hauteur (desktop) | 400-500px |
| Hauteur (mobile) | 200-250px |
| Nombre de slides | 3-5 |
| Auto-play | 5 secondes par slide |
| Transition | Fade ou slide (300-500ms) |
| Indicateurs | Dots en bas (centrés) |
| Navigation | Flèches gauche/droite (desktop uniquement) |
| Swipe | Oui (mobile et desktop) |

### Structure d'un slide

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  [Image de fond, pleine largeur]                         │
│                                                          │
│    ┌──────────────────────────┐                          │
│    │ Titre accrocheur         │  Texte blanc avec ombre  │
│    │ Sous-titre / description │  ou fond semi-transparent│
│    │                          │                          │
│    │ [DECOUVRIR →]            │  Bouton CTA              │
│    └──────────────────────────┘                          │
│                                                          │
│                    ● ○ ○ ○                               │
└──────────────────────────────────────────────────────────┘
```

### Accessibilité

- Pause au hover (desktop) et au toucher (mobile)
- `aria-label` sur les contrôles de navigation
- `alt` sur chaque image de slide
- Réduire la vitesse si `prefers-reduced-motion`

---

## 10. Animations et transitions

### Transitions globales

```css
/* Transition par défaut pour les éléments interactifs */
transition: all 200ms ease-in-out;

/* Transitions spécifiques */
transition-colors: 150ms;   /* Hover des boutons */
transition-shadow: 200ms;   /* Hover des cartes */
transition-transform: 200ms; /* Scale des boutons */
transition-opacity: 300ms;  /* Fade in/out des modales */
```

### Animations

| Animation | Durée | Utilisation |
|---|---|---|
| fade-in | 300ms | Apparition de modales, overlays |
| slide-in-right | 300ms | Drawer panier, menu mobile |
| slide-in-left | 300ms | Menu mobile |
| scale-in | 200ms | Modales, popups |
| slide-up | 300ms | Notifications toast |
| pulse | 2s (repeat) | Skeleton loading |
| spin | 1s (repeat) | Loading spinner |

### Accessibilité des animations

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 11. Icônes

### Bibliothèque recommandée

**Lucide React** (anciennement Feather Icons) :
- Légère (tree-shakeable)
- Style cohérent et moderne
- 1000+ icônes

```bash
npm install lucide-react
```

### Icônes principales utilisées

| Utilisation | Icône | Nom Lucide |
|---|---|---|
| Recherche | Loupe | Search |
| Panier | Chariot | ShoppingCart |
| Utilisateur | Personne | User |
| Menu | Hamburger | Menu |
| Fermer | Croix | X |
| Flèche droite | Chevron | ChevronRight |
| Coeur | Coeur | Heart |
| Etoile | Etoile | Star |
| Modifier | Crayon | Pencil |
| Supprimer | Poubelle | Trash2 |
| Plus | Plus | Plus |
| Moins | Moins | Minus |
| Check | Coche | Check |
| Alerte | Triangle | AlertTriangle |
| Dashboard | Graphique | BarChart3 |
| Produits | Boîte | Package |
| Commandes | Facture | Receipt |
| Clients | Personnes | Users |
| Paramètres | Engrenage | Settings |
| Déconnexion | Porte | LogOut |
| WhatsApp | Message | MessageCircle |
| Partager | Partage | Share2 |
| Oeil | Oeil | Eye / EyeOff |
| Upload | Upload | Upload |
| Image | Image | ImageIcon |

### Tailles standard

| Contexte | Taille | Classe |
|---|---|---|
| Dans le texte | 16px | w-4 h-4 |
| Boutons | 20px | w-5 h-5 |
| Header | 24px | w-6 h-6 |
| Illustrations vides | 48px | w-12 h-12 |
| Hero/CTA | 64px | w-16 h-16 |

---

## 12. Checklist design

Avant de considérer le design comme terminé :

- [ ] Toutes les couleurs respectent la charte graphique du client
- [ ] Les polices sont chargées via next/font (pas de FOUT)
- [ ] Le site est lisible et utilisable sur mobile 360px
- [ ] Le contraste texte/fond respecte les standards WCAG AA (ratio 4.5:1)
- [ ] Les boutons ont des états hover, active, disabled et loading
- [ ] Les formulaires ont des messages d'erreur visibles
- [ ] Les images ont des attributs alt
- [ ] Les skeleton loaders sont en place pour tous les chargements
- [ ] Le thème admin est sombre et distinct de la boutique
- [ ] Le footer est cohérent sur toutes les pages
- [ ] Le header est sticky et fonctionne au scroll
- [ ] Les animations respectent prefers-reduced-motion
- [ ] Le bouton WhatsApp est visible et fonctionnel
- [ ] Les prix sont formatés avec séparateurs de milliers
- [ ] Les badges de statut sont colorés et lisibles
