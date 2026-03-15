# Panier et Commandes

## Instructions pour l'agent IA

Ce document spécifie le fonctionnement complet du panier (côté client et serveur) et du processus de commande, de la création jusqu'au suivi du statut.

---

## 1. Panier - Architecture

### Double stockage

Le panier fonctionne en double couche pour une expérience optimale :

| Couche | Technologie | Quand | Pourquoi |
|---|---|---|---|
| Client (local) | Zustand + localStorage | Toujours | Panier instantané, fonctionne sans connexion |
| Serveur (base) | PostgreSQL via API | Si connecté | Persiste entre les appareils, source de vérité |

### Flow selon l'état de connexion

**Utilisateur non connecté** :
1. Le panier est géré 100% côté client (Zustand avec persist localStorage)
2. Les prix sont ceux affichés sur la page (pas de vérification serveur)
3. A la connexion, le panier local est synchronisé avec le serveur

**Utilisateur connecté** :
1. Chaque modification du panier est envoyée à l'API
2. Le store Zustand est mis à jour avec la réponse serveur
3. Le localStorage est mis à jour en backup

**Synchronisation à la connexion** :
1. POST /api/cart/sync avec les items du localStorage
2. Le serveur fusionne : si un produit existe dans les deux, garder la plus grande quantité
3. Le serveur vérifie le stock de chaque item
4. Le store client est remplacé par la réponse serveur

---

## 2. Panier - Store Zustand

### Interface

```typescript
interface CartItem {
  productId: string;
  name: string;
  slug: string;
  price: number;
  comparePrice?: number;
  image: string;
  quantity: number;
  stock: number; // Pour validation côté client
}

interface CartStore {
  items: CartItem[];
  isOpen: boolean; // Drawer ouvert/fermé

  // Actions
  addItem: (product: Product, quantity?: number) => void;
  removeItem: (productId: string) => void;
  updateQuantity: (productId: string, quantity: number) => void;
  clearCart: () => void;
  openCart: () => void;
  closeCart: () => void;
  syncWithServer: (serverItems: CartItem[]) => void;

  // Computed
  itemsCount: number;    // Nombre total d'articles
  subtotal: number;      // Sous-total
  shippingFee: number;   // Frais de livraison
  total: number;         // Total final
}
```

### Logique d'ajout

```typescript
addItem: (product, quantity = 1) => {
  const existing = items.find(i => i.productId === product.id);

  if (existing) {
    // Produit déjà dans le panier → incrémenter
    const newQuantity = Math.min(existing.quantity + quantity, product.stock);
    updateQuantity(product.id, newQuantity);
  } else {
    // Nouveau produit
    items.push({
      productId: product.id,
      name: product.name,
      slug: product.slug,
      price: product.price,
      comparePrice: product.comparePrice,
      image: product.primaryImage?.url || '',
      quantity: Math.min(quantity, product.stock),
      stock: product.stock,
    });
  }

  openCart(); // Ouvrir le drawer
}
```

### Persist middleware

```typescript
// Le panier survit au refresh de la page
create(
  persist(
    (set, get) => ({ /* ... */ }),
    {
      name: 'cart-storage', // Clé localStorage
      partialize: (state) => ({ items: state.items }), // Ne persister que les items
    }
  )
)
```

---

## 3. Panier - Interface utilisateur

### 3.1 Drawer panier (mini-panier)

**Ouverture** :
- Clic sur l'icône panier dans le header
- Automatiquement après un ajout au panier

**Contenu** :
```
┌─── Panier (3 articles) ──── [X] ┐
│                                   │
│ ┌─────┬──────────────────┬───┐   │
│ │[IMG]│ T-Shirt Premium  │ X │   │
│ │     │ 15 000 FCFA      │   │   │
│ │     │ [- 2 +]          │   │   │
│ └─────┴──────────────────┴───┘   │
│                                   │
│ ┌─────┬──────────────────┬───┐   │
│ │[IMG]│ Pantalon Classic │ X │   │
│ │     │ 25 000 FCFA      │   │   │
│ │     │ [- 1 +]          │   │   │
│ └─────┴──────────────────┴───┘   │
│                                   │
│ ─────────────────────────────── │
│ Sous-total :        55 000 FCFA  │
│                                   │
│ [    VOIR LE PANIER    ]         │
│ [    COMMANDER          ]         │
└───────────────────────────────────┘
```

**Panier vide** :
```
┌─── Panier ──────────── [X] ┐
│                              │
│    🛒                       │
│    Votre panier est vide    │
│                              │
│    [VOIR NOS PRODUITS]      │
│                              │
└──────────────────────────────┘
```

### 3.2 Page panier (/cart)

**Desktop** :

| Image | Produit | Prix unitaire | Quantité | Total | |
|---|---|---|---|---|---|
| [mini] | T-Shirt Premium | 15 000 FCFA | [- 2 +] | 30 000 FCFA | [Supprimer] |
| [mini] | Pantalon Classic | 25 000 FCFA | [- 1 +] | 25 000 FCFA | [Supprimer] |

```
                        Sous-total :  55 000 FCFA
                        Livraison  :   2 000 FCFA
                        ─────────────────────────
                        Total      :  57 000 FCFA

         [Continuer les achats]  [Commander]
```

**Message livraison gratuite** :
- Si le seuil de livraison gratuite est défini et que le sous-total est en dessous :
- "Plus que X FCFA pour profiter de la livraison gratuite !"
- Barre de progression visuelle

**Mobile** :
- Chaque article en carte empilée (pas de tableau)
- Image à gauche, infos à droite
- Sélecteur de quantité et bouton supprimer sous les infos
- Récapitulatif sticky en bas

---

## 4. Processus de commande (Checkout)

### Flow complet

```
Panier → Checkout (adresse) → Récapitulatif → Paiement → Résultat
```

### Etape 1 : Adresse de livraison

**Si l'utilisateur a des adresses enregistrées** :
- Liste des adresses avec radio button
- Bouton "Utiliser une autre adresse" → formulaire
- Adresse par défaut pré-sélectionnée

**Si l'utilisateur n'a pas d'adresse** :
- Formulaire d'adresse directement

**Champs du formulaire** :

| Champ | Type | Requis | Placeholder |
|---|---|---|---|
| Prénom | text | Oui | "Prénom" |
| Nom | text | Oui | "Nom" |
| Téléphone | tel | Oui | "+225 07 XX XX XX XX" |
| Adresse | text | Oui | "Rue, numéro, lot" |
| Ville | text | Oui | "Abidjan" |
| Commune | text | Non | "Cocody" |
| Quartier | text | Non | "Angré 8ème tranche" |
| Instructions | textarea | Non | "Face à la pharmacie, portail bleu" |
| Sauvegarder | checkbox | Non | "Enregistrer cette adresse" |

### Etape 2 : Récapitulatif

**Affichage** :
```
┌─────────────────────────────────────────┐
│ RECAPITULATIF DE COMMANDE               │
│                                         │
│ Articles :                              │
│   T-Shirt Premium      x2   30 000 FCFA│
│   Pantalon Classic     x1   25 000 FCFA│
│                                         │
│ Adresse de livraison :                  │
│   Abdoul Diallo                         │
│   Angré 8ème tranche, Cocody, Abidjan   │
│   +225 07 01 23 45 67                   │
│                                         │
│ ────────────────────────────────────── │
│ Sous-total :                 55 000 FCFA│
│ Livraison :                   2 000 FCFA│
│ Total :                      57 000 FCFA│
│                                         │
│ [MODIFIER LE PANIER]   [PAYER 57 000 FCFA]│
└─────────────────────────────────────────┘
```

### Etape 3 : Choix du moyen de paiement et paiement

Voir `paiement-integration.md` pour les détails.

---

## 5. Création de commande (côté serveur)

### Processus transactionnel

Tout se passe dans une transaction Prisma (tout ou rien) :

```typescript
async createOrder(userId: string, dto: CreateOrderDto) {
  return prisma.$transaction(async (tx) => {
    // 1. Récupérer le panier avec les produits
    const cart = await tx.cart.findUnique({
      where: { userId },
      include: { items: { include: { product: true } } },
    });

    if (!cart || cart.items.length === 0) {
      throw new BadRequestException('Votre panier est vide');
    }

    // 2. Vérifier le stock de chaque article
    for (const item of cart.items) {
      if (item.product.stock < item.quantity) {
        throw new BadRequestException(
          `Stock insuffisant pour "${item.product.name}" (${item.product.stock} disponibles)`
        );
      }
      if (item.product.status !== 'ACTIVE') {
        throw new BadRequestException(
          `Le produit "${item.product.name}" n'est plus disponible`
        );
      }
    }

    // 3. Calculer les montants
    const subtotal = cart.items.reduce(
      (sum, item) => sum + Number(item.product.price) * item.quantity, 0
    );
    const shippingFee = calculateShippingFee(subtotal);
    const total = subtotal + shippingFee;

    // 4. Générer le numéro de commande
    const orderNumber = await generateOrderNumber(tx);

    // 5. Créer la commande
    const order = await tx.order.create({
      data: {
        orderNumber,
        userId,
        addressId: dto.addressId,
        status: 'PENDING',
        subtotal,
        shippingFee,
        total,
        currency: 'XOF',
        notes: dto.notes,
        items: {
          create: cart.items.map(item => ({
            productId: item.productId,
            productName: item.product.name,
            productImage: item.product.images?.[0]?.url || null,
            price: item.product.price,
            quantity: item.quantity,
            total: Number(item.product.price) * item.quantity,
          })),
        },
      },
    });

    // 6. Créer le paiement en PENDING
    await tx.payment.create({
      data: {
        orderId: order.id,
        amount: total,
        currency: 'XOF',
        method: dto.paymentMethod,
        status: 'PENDING',
      },
    });

    // 7. Décrémenter le stock
    for (const item of cart.items) {
      await tx.product.update({
        where: { id: item.productId },
        data: { stock: { decrement: item.quantity } },
      });
    }

    // 8. Vider le panier
    await tx.cartItem.deleteMany({ where: { cartId: cart.id } });

    return order;
  });
}
```

### Numéro de commande

**Format** : `CMD-YYYYMMDD-XXXX`

**Exemples** :
- CMD-20260315-0001 (1ère commande du 15 mars 2026)
- CMD-20260315-0002 (2ème commande du 15 mars 2026)
- CMD-20260316-0001 (1ère commande du 16 mars 2026)

**Génération** :
```typescript
async function generateOrderNumber(tx: PrismaTransaction): Promise<string> {
  const today = new Date();
  const dateStr = today.toISOString().slice(0, 10).replace(/-/g, '');

  // Compter les commandes du jour
  const count = await tx.order.count({
    where: {
      createdAt: {
        gte: new Date(today.getFullYear(), today.getMonth(), today.getDate()),
        lt: new Date(today.getFullYear(), today.getMonth(), today.getDate() + 1),
      },
    },
  });

  const sequence = String(count + 1).padStart(4, '0');
  return `CMD-${dateStr}-${sequence}`;
}
```

### Calcul des frais de livraison

```typescript
function calculateShippingFee(subtotal: number): number {
  const freeShippingThreshold = getSetting('free_shipping_threshold'); // ex: 25000
  const defaultShippingFee = getSetting('shipping_fee_default');       // ex: 2000

  if (freeShippingThreshold && subtotal >= freeShippingThreshold) {
    return 0; // Livraison gratuite
  }
  return defaultShippingFee;
}
```

---

## 6. Statuts de commande

### Machine à états

```
                    ┌──────────┐
              ┌─────│ PENDING  │────────────────────┐
              │     └────┬─────┘                    │
              │          │ paiement réussi           │ annulation
              │     ┌────▼──────┐                    │
              │     │ CONFIRMED │                    │
              │     └────┬──────┘                    │
              │          │ préparation               │
              │     ┌────▼───────┐              ┌────▼──────┐
              │     │ PROCESSING │              │ CANCELLED │
              │     └────┬───────┘              └───────────┘
              │          │ expédition
              │     ┌────▼──────┐
              │     │  SHIPPED  │
              │     └────┬──────┘
              │          │ réception
              │     ┌────▼──────┐
              │     │ DELIVERED │
              │     └───────────┘
              │
         (annulation possible depuis PENDING ou CONFIRMED uniquement)
```

### Transitions autorisées

| De | Vers | Qui | Condition |
|---|---|---|---|
| PENDING | CONFIRMED | Système | Paiement réussi (callback) |
| PENDING | CANCELLED | Admin/Client | Annulation |
| CONFIRMED | PROCESSING | Admin | Début de préparation |
| CONFIRMED | CANCELLED | Admin | Annulation avant préparation |
| PROCESSING | SHIPPED | Admin | Expédition |
| SHIPPED | DELIVERED | Admin | Confirmation de livraison |

### Transitions interdites
- Pas de retour en arrière (DELIVERED → SHIPPED)
- CANCELLED est un état final
- Un client ne peut annuler que si PENDING
- Un admin peut annuler si PENDING ou CONFIRMED

### Actions associées aux transitions

| Transition | Action |
|---|---|
| → CONFIRMED | Enregistrer la date de confirmation |
| → SHIPPED | Enregistrer la date d'expédition, envoyer notification client |
| → DELIVERED | Enregistrer la date de livraison, envoyer notification client |
| → CANCELLED | Ré-incrémenter le stock, enregistrer la raison et la date |

---

## 7. Historique des commandes (espace client)

### Liste des commandes

```
┌───────────────────────────────────────────────────┐
│ MES COMMANDES                                      │
│                                                    │
│ ┌──────────────────────────────────────────────┐  │
│ │ CMD-20260315-0001          15 mars 2026      │  │
│ │ 57 000 FCFA                                  │  │
│ │ Statut: ● DELIVERED  Paiement: ● COMPLETED   │  │
│ │                                    [Détails ▼]│  │
│ ├──────────────────────────────────────────────┤  │
│ │  T-Shirt Premium      x2       30 000 FCFA  │  │
│ │  Pantalon Classic     x1       25 000 FCFA  │  │
│ │  Livraison                      2 000 FCFA  │  │
│ │  Total                         57 000 FCFA  │  │
│ └──────────────────────────────────────────────┘  │
│                                                    │
│ ┌──────────────────────────────────────────────┐  │
│ │ CMD-20260310-0003          10 mars 2026      │  │
│ │ 32 000 FCFA                                  │  │
│ │ Statut: ● PENDING    Paiement: ● PENDING     │  │
│ │               [Finaliser le paiement]         │  │
│ └──────────────────────────────────────────────┘  │
│                                                    │
│                    [< 1 2 3 >]                     │
└───────────────────────────────────────────────────┘
```

### Badges de statut

| Statut | Couleur | Texte affiché |
|---|---|---|
| PENDING | Jaune/Orange | "En attente" |
| CONFIRMED | Bleu | "Confirmée" |
| PROCESSING | Violet | "En préparation" |
| SHIPPED | Indigo | "Expédiée" |
| DELIVERED | Vert | "Livrée" |
| CANCELLED | Rouge | "Annulée" |

### Détail expandable
- Clic sur "Détails" → accordéon qui s'ouvre avec :
  - Liste des articles (image, nom, quantité, prix)
  - Adresse de livraison
  - Date de commande, date de livraison (si livré)
  - Statut du paiement

### Bouton "Finaliser le paiement"
- Affiché uniquement si : statut commande = PENDING ET statut paiement = PENDING ou FAILED
- Redirige vers la page de paiement avec les options de moyen de paiement
- Permet au client de réessayer le paiement sans recréer la commande

---

## 8. Gestion des commandes (admin)

### Tableau

| N° Commande | Client | Date | Montant | Statut | Paiement | Actions |
|---|---|---|---|---|---|---|
| CMD-20260315-0001 | Abdoul D. | 15/03/2026 | 57 000 | ● DELIVERED | ● COMPLETED | [Voir] [Statut ▼] |
| CMD-20260315-0002 | Marie K. | 15/03/2026 | 32 000 | ● PENDING | ● PENDING | [Voir] [Statut ▼] |

### Filtres
- Par statut (dropdown multi-select)
- Par statut de paiement
- Par date (date picker range)
- Recherche par numéro de commande ou nom client

### Changement de statut
- Dropdown avec les transitions valides uniquement
- Confirmation via modale : "Confirmer le changement de statut vers [NOUVEAU STATUT] ?"
- Champ optionnel pour ajouter une note admin
