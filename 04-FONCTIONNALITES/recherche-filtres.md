# Recherche et Filtres

## Instructions pour l'agent IA

Ce document spécifie le système de recherche avec autocomplete et le filtrage par catégorie. La recherche est un élément clé de la conversion en e-commerce.

---

## 1. Recherche autocomplete

### 1.1 Comportement

| Paramètre | Valeur | Raison |
|---|---|---|
| Déclenchement | Après 1 caractère | Résultats dès la première lettre |
| Debounce | 300ms | Evite de surcharger l'API |
| Max résultats | 5 suggestions | UX optimale, pas trop de choix |
| Données par suggestion | Image, nom, prix | L'utilisateur identifie rapidement le produit |
| API | GET /api/products/search?q=xxx | Endpoint dédié optimisé |
| Temps de réponse | < 200ms | L'autocomplete doit être instantané |

### 1.2 Interface

**Desktop** :
```
┌─────────────────────────────────────────────┐
│ 🔍 [Rechercher un produit...              ] │
│                                              │
│ ┌────────────────────────────────────────┐   │
│ │ ┌────┐ Chemise classique blanche      │   │
│ │ │img │ 25 000 FCFA                    │   │
│ │ └────┘                                │   │
│ ├────────────────────────────────────────┤   │
│ │ ┌────┐ Chemise slim fit               │   │
│ │ │img │ 30 000 FCFA                    │   │
│ │ └────┘                                │   │
│ ├────────────────────────────────────────┤   │
│ │ ┌────┐ Chemisier femme                │   │
│ │ │img │ 22 000 FCFA                    │   │
│ │ └────┘                                │   │
│ ├────────────────────────────────────────┤   │
│ │ Voir tous les résultats pour "che" →  │   │
│ └────────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

**Mobile** :
```
┌─────────────────────────────────┐
│ ← 🔍 [Rechercher...        ] X │  ← Plein écran
│                                  │
│ ┌────┐ Chemise classique        │
│ │img │ 25 000 FCFA              │
│ └────┘                          │
│ ─────────────────────────────── │
│ ┌────┐ Chemise slim fit         │
│ │img │ 30 000 FCFA              │
│ └────┘                          │
│ ─────────────────────────────── │
│ ┌────┐ Chemisier femme          │
│ │img │ 22 000 FCFA              │
│ └────┘                          │
│                                  │
│ Voir tous les résultats →       │
│                                  │
└─────────────────────────────────┘
```

### 1.3 Interactions

| Action | Résultat |
|---|---|
| Taper dans l'input | Suggestions apparaissent (après debounce) |
| Clic sur une suggestion | Navigation vers la page produit |
| Appuyer Entrée | Navigation vers le catalogue avec ?search=terme |
| Flèche bas/haut | Navigation dans les suggestions |
| Entrée sur une suggestion | Navigation vers ce produit |
| Escape | Ferme les suggestions |
| Clic en dehors | Ferme les suggestions |
| Effacer l'input | Ferme les suggestions |
| Clic sur "Voir tous les résultats" | Navigation vers le catalogue avec ?search=terme |

### 1.4 Etats

| Etat | Affichage |
|---|---|
| Input vide | Rien (dropdown fermé) |
| En cours de frappe (< debounce) | Rien encore |
| Chargement API | Spinner dans l'input ou petit loader |
| Résultats trouvés | Liste de suggestions |
| Aucun résultat | "Aucun produit trouvé pour 'xxx'" |
| Erreur API | Rien (dégradation silencieuse) |

### 1.5 Implémentation Zustand

```typescript
interface SearchStore {
  query: string;
  results: SearchResult[];
  isOpen: boolean;
  isLoading: boolean;
  selectedIndex: number; // Pour la navigation clavier

  setQuery: (query: string) => void;
  setResults: (results: SearchResult[]) => void;
  open: () => void;
  close: () => void;
  reset: () => void;
  selectNext: () => void;
  selectPrevious: () => void;
}

interface SearchResult {
  id: string;
  name: string;
  slug: string;
  price: number;
  image: string;
}
```

### 1.6 Hook personnalisé

```typescript
function useSearch() {
  const { query, setQuery, results, setResults, isLoading } = useSearchStore();
  const debouncedQuery = useDebounce(query, 300);

  useEffect(() => {
    if (debouncedQuery.length < 1) {
      setResults([]);
      return;
    }

    const controller = new AbortController();

    async function search() {
      try {
        const { data } = await api.get(`/products/search?q=${debouncedQuery}&limit=5`, {
          signal: controller.signal,
        });
        setResults(data.data);
      } catch (error) {
        if (!axios.isCancel(error)) {
          setResults([]);
        }
      }
    }

    search();
    return () => controller.abort(); // Annuler les requêtes précédentes
  }, [debouncedQuery]);

  return { query, setQuery, results, isLoading };
}
```

### 1.7 Requête backend

```typescript
// ProductService.search()
async search(query: string, limit: number = 5) {
  return this.prisma.product.findMany({
    where: {
      status: 'ACTIVE',
      name: {
        contains: query,
        mode: 'insensitive', // Case-insensitive
      },
    },
    select: {
      id: true,
      name: true,
      slug: true,
      price: true,
      images: {
        where: { isPrimary: true },
        select: { url: true },
        take: 1,
      },
    },
    orderBy: [
      // Prioriser les noms qui commencent par le terme
      { name: 'asc' },
    ],
    take: limit,
  });
}
```

---

## 2. Filtrage par catégorie

### 2.1 Barre de catégories

**Position** : En haut de la page catalogue, sous le header

```
[Tout] [T-Shirts] [Pantalons] [Robes] [Accessoires] [Chaussures] →
  ^active                                             ^scroll indicator
```

**Style** :
- Boutons pill/badge avec bords arrondis
- Catégorie active : fond couleur primaire, texte blanc
- Catégorie inactive : fond gris clair, texte gris foncé
- Espacement entre les pills : 8-12px
- Hauteur du pill : 36-40px
- Police : 14px, medium weight

**Mobile** :
- Scroll horizontal natif (overflow-x: auto)
- Indicateur visuel qu'il y a plus de catégories (gradient de fondu à droite)
- Pas de scrollbar visible (-webkit-scrollbar: none)
- Touch/swipe natif

### 2.2 Comportement

| Action | Résultat |
|---|---|
| Clic "Tout" | Affiche tous les produits (supprime le filtre catégorie) |
| Clic sur une catégorie | Filtre les produits par cette catégorie |
| Clic sur la catégorie active | Désactive le filtre (retour à "Tout") |
| Changement de catégorie | Reset de la pagination à la page 1 |
| Changement de catégorie | URL mise à jour (?category=slug) |

### 2.3 Combinaison recherche + catégorie

La recherche et le filtre catégorie sont combinables :

| Recherche | Catégorie | Résultat |
|---|---|---|
| Vide | Aucune | Tous les produits |
| Vide | "T-Shirts" | Produits de la catégorie T-Shirts |
| "chemise" | Aucune | Produits contenant "chemise" (toutes catégories) |
| "chemise" | "Homme" | Produits contenant "chemise" dans la catégorie Homme |

**URL** : `?search=chemise&category=homme`

### 2.4 Implémentation

```typescript
// Page catalogue - récupération des produits
const searchParams = useSearchParams();
const category = searchParams.get('category');
const search = searchParams.get('search');
const page = parseInt(searchParams.get('page') || '1');
const sortBy = searchParams.get('sortBy') || 'createdAt';
const sortOrder = searchParams.get('sortOrder') || 'desc';

const { data, isLoading } = useQuery({
  queryKey: ['products', { category, search, page, sortBy, sortOrder }],
  queryFn: () => api.get('/products', {
    params: {
      categorySlug: category || undefined,
      search: search || undefined,
      page,
      limit: 20,
      sortBy,
      sortOrder,
    },
  }),
});
```

---

## 3. Tri des produits

### Options de tri

| Option affichée | sortBy | sortOrder |
|---|---|---|
| Pertinence | createdAt | desc | (par défaut) |
| Prix croissant | price | asc |
| Prix décroissant | price | desc |
| Nouveautés | createdAt | desc |
| Nom A-Z | name | asc |
| Plus populaires | viewCount | desc |

### Interface

**Desktop** : Dropdown en haut à droite du catalogue
```
Trier par : [Pertinence ▼]
```

**Mobile** : Bouton avec icône de tri, ouvre un bottom sheet ou dropdown

### Comportement
- Le tri modifie l'URL (?sortBy=price&sortOrder=asc)
- Le tri est conservé quand on change de catégorie
- Le tri est réinitialisé à "Pertinence" quand on fait une nouvelle recherche

---

## 4. Filtres avancés (V2)

### Filtre par prix (V2)

```
Prix : [5 000] ──●──────●── [100 000] FCFA
```

- Range slider double poignée
- Valeurs min et max basées sur les prix du catalogue
- Inputs numériques pour saisie directe
- URL : ?minPrice=5000&maxPrice=100000

### Filtre par disponibilité (V2)

```
[ ] En stock uniquement
```

### Filtre par état (V2)

```
[x] Nouveau  [ ] Promotion  [ ] Populaire
```

---

## 5. Page de résultats de recherche

Quand l'utilisateur appuie sur Entrée dans la barre de recherche ou clique sur "Voir tous les résultats" :

**URL** : /products?search=chemise

**Affichage** :
```
Résultats pour "chemise" (12 produits)
[Barre de catégories pour affiner]
[Grille de produits filtrés]
[Pagination]
```

**Si aucun résultat** :
```
┌─────────────────────────────────────────────────┐
│                                                  │
│  🔍                                            │
│  Aucun produit trouvé pour "xyzabc"             │
│                                                  │
│  Suggestions :                                   │
│  • Vérifiez l'orthographe                       │
│  • Essayez des termes plus généraux             │
│  • Parcourez nos catégories                      │
│                                                  │
│  [VOIR TOUS LES PRODUITS]                       │
│                                                  │
└─────────────────────────────────────────────────┘
```

---

## 6. Performance et optimisation

### Côté frontend

| Optimisation | Implémentation |
|---|---|
| Debounce | 300ms avant d'appeler l'API |
| Abort controller | Annule les requêtes précédentes si nouvelle saisie |
| Cache | React Query / SWR avec cache de 2 minutes |
| Skeleton | Afficher des placeholders pendant le chargement |
| Lazy loading images | next/image avec loading="lazy" pour la grille |

### Côté backend

| Optimisation | Implémentation |
|---|---|
| Index PostgreSQL | Index sur products.name pour la recherche |
| ILIKE optimisé | Utiliser pg_trgm extension pour les grandes bases |
| Limite stricte | Max 5 résultats pour l'autocomplete |
| Select minimal | Retourner seulement les champs nécessaires |
| Cache réponse | Cache en mémoire (2 min TTL) pour les requêtes fréquentes |

### Seuils de performance

| Métrique | Cible |
|---|---|
| Temps de réponse autocomplete | < 200ms |
| Temps d'affichage page catalogue | < 1 seconde |
| Debounce input | 300ms |
| Nombre max de suggestions | 5 |
| Cache TTL | 2 minutes |
