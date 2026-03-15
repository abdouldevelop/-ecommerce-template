# Schéma de Base de Données - E-Commerce

## Instructions pour l'agent IA

Ce schéma est au format Prisma et couvre tous les besoins d'un e-commerce complet. Adapter les champs selon les besoins spécifiques du client (variantes produits, options de livraison, etc.).

---

## Schéma Prisma complet

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ============================================================
// ENUMS
// ============================================================

enum Role {
  ADMIN
  CUSTOMER
}

enum ProductStatus {
  DRAFT      // Brouillon, non visible sur le site
  ACTIVE     // Publié et visible
  ARCHIVED   // Retiré de la vente mais conservé en base
}

enum OrderStatus {
  PENDING     // Commande créée, en attente de paiement
  CONFIRMED   // Paiement reçu, commande confirmée
  PROCESSING  // En cours de préparation
  SHIPPED     // Expédiée
  DELIVERED   // Livrée
  CANCELLED   // Annulée
  REFUNDED    // Remboursée
}

enum PaymentStatus {
  PENDING     // En attente de paiement
  PROCESSING  // Paiement en cours (redirection vers prestataire)
  COMPLETED   // Paiement réussi
  FAILED      // Paiement échoué
  CANCELLED   // Paiement annulé par l'utilisateur
  REFUNDED    // Remboursé
}

enum PaymentMethod {
  CARD          // Carte bancaire (Visa, Mastercard)
  ORANGE_MONEY  // Orange Money
  MTN_MOMO      // MTN Mobile Money
  WAVE          // Wave
  MOOV_FLOOZ    // Moov Money / Flooz
  CASH          // Paiement à la livraison
}

// ============================================================
// USER - Utilisateurs (clients et admins)
// ============================================================

model User {
  id            String    @id @default(uuid())
  email         String    @unique
  password      String    // Hash bcrypt
  firstName     String    @map("first_name")
  lastName      String    @map("last_name")
  phone         String?   // Numéro de téléphone (format international)
  role          Role      @default(CUSTOMER)
  isActive      Boolean   @default(true) @map("is_active")
  lastLoginAt   DateTime? @map("last_login_at")
  refreshToken  String?   @map("refresh_token") // Hash du refresh token actuel
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  // Relations
  addresses     Address[]
  orders        Order[]
  cart          Cart?
  sessions      Session[]

  @@map("users")
}

// ============================================================
// SESSION - Sessions utilisateur (pour tracking et sécurité)
// ============================================================

model Session {
  id            String    @id @default(uuid())
  userId        String    @map("user_id")
  token         String    @unique // Hash du JWT
  userAgent     String?   @map("user_agent")
  ipAddress     String?   @map("ip_address")
  expiresAt     DateTime  @map("expires_at")
  createdAt     DateTime  @default(now()) @map("created_at")

  // Relations
  user          User      @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([expiresAt])
  @@map("sessions")
}

// ============================================================
// ADDRESS - Adresses de livraison
// ============================================================

model Address {
  id            String    @id @default(uuid())
  userId        String    @map("user_id")
  label         String?   // "Maison", "Bureau", etc.
  firstName     String    @map("first_name")
  lastName      String    @map("last_name")
  phone         String
  street        String    // Adresse complète
  city          String    // Ville (ex: Abidjan)
  commune       String?   // Commune (ex: Cocody, Plateau)
  quarter       String?   // Quartier (ex: Angré, Riviera)
  country       String    @default("CI") // Code pays ISO 3166
  isDefault     Boolean   @default(false) @map("is_default")
  instructions  String?   // Instructions de livraison
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  // Relations
  user          User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  orders        Order[]

  @@index([userId])
  @@map("addresses")
}

// ============================================================
// CATEGORY - Catégories de produits
// ============================================================

model Category {
  id            String    @id @default(uuid())
  name          String    @unique
  slug          String    @unique // URL-friendly (ex: "t-shirts")
  description   String?
  image         String?   // URL de l'image de la catégorie
  parentId      String?   @map("parent_id") // Pour les sous-catégories
  sortOrder     Int       @default(0) @map("sort_order")
  isActive      Boolean   @default(true) @map("is_active")
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  // Relations
  parent        Category?  @relation("CategoryTree", fields: [parentId], references: [id])
  children      Category[] @relation("CategoryTree")
  products      Product[]

  @@index([slug])
  @@index([parentId])
  @@map("categories")
}

// ============================================================
// PRODUCT - Produits
// ============================================================

model Product {
  id            String        @id @default(uuid())
  name          String
  slug          String        @unique // URL-friendly
  description   String?       // Description courte (160 caractères pour SEO)
  longDescription String?     @map("long_description") // Description longue (HTML)
  price         Decimal       @db.Decimal(10, 0) // Prix en FCFA (pas de centimes)
  comparePrice  Decimal?      @map("compare_price") @db.Decimal(10, 0) // Ancien prix (barré)
  costPrice     Decimal?      @map("cost_price") @db.Decimal(10, 0) // Prix d'achat (usage interne)
  sku           String?       @unique // Référence produit
  stock         Int           @default(0) // Quantité en stock
  lowStockAlert Int           @default(5) @map("low_stock_alert") // Seuil alerte stock bas
  weight        Decimal?      @db.Decimal(8, 2) // Poids en kg (si livraison au poids)
  categoryId    String        @map("category_id")
  status        ProductStatus @default(DRAFT)
  isFeatured    Boolean       @default(false) @map("is_featured") // Produit vedette
  sortOrder     Int           @default(0) @map("sort_order")
  metaTitle     String?       @map("meta_title") // SEO
  metaDescription String?     @map("meta_description") // SEO
  viewCount     Int           @default(0) @map("view_count") // Nombre de vues
  createdAt     DateTime      @default(now()) @map("created_at")
  updatedAt     DateTime      @updatedAt @map("updated_at")

  // Relations
  category      Category      @relation(fields: [categoryId], references: [id])
  images        ProductImage[]
  cartItems     CartItem[]
  orderItems    OrderItem[]

  @@index([slug])
  @@index([categoryId])
  @@index([status])
  @@index([isFeatured])
  @@index([name]) // Pour la recherche
  @@map("products")
}

// ============================================================
// PRODUCT IMAGE - Images des produits
// ============================================================

model ProductImage {
  id            String    @id @default(uuid())
  productId     String    @map("product_id")
  url           String    // Chemin ou URL de l'image
  alt           String?   // Texte alternatif (accessibilité + SEO)
  isPrimary     Boolean   @default(false) @map("is_primary") // Image principale
  sortOrder     Int       @default(0) @map("sort_order")
  createdAt     DateTime  @default(now()) @map("created_at")

  // Relations
  product       Product   @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@index([productId])
  @@map("product_images")
}

// ============================================================
// CART - Panier
// ============================================================

model Cart {
  id            String    @id @default(uuid())
  userId        String    @unique @map("user_id") // Un panier par utilisateur
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  // Relations
  user          User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  items         CartItem[]

  @@map("carts")
}

// ============================================================
// CART ITEM - Articles du panier
// ============================================================

model CartItem {
  id            String    @id @default(uuid())
  cartId        String    @map("cart_id")
  productId     String    @map("product_id")
  quantity      Int       @default(1)
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  // Relations
  cart          Cart      @relation(fields: [cartId], references: [id], onDelete: Cascade)
  product       Product   @relation(fields: [productId], references: [id])

  @@unique([cartId, productId]) // Un seul CartItem par produit dans un panier
  @@index([cartId])
  @@index([productId])
  @@map("cart_items")
}

// ============================================================
// ORDER - Commandes
// ============================================================

model Order {
  id              String        @id @default(uuid())
  orderNumber     String        @unique @map("order_number") // Ex: CMD-20260315-0001
  userId          String        @map("user_id")
  addressId       String?       @map("address_id")
  status          OrderStatus   @default(PENDING)
  subtotal        Decimal       @db.Decimal(10, 0) // Sous-total avant frais
  shippingFee     Decimal       @default(0) @map("shipping_fee") @db.Decimal(10, 0)
  discount        Decimal       @default(0) @db.Decimal(10, 0) // Réduction appliquée
  total           Decimal       @db.Decimal(10, 0) // Total final
  currency        String        @default("XOF")
  notes           String?       // Notes du client
  adminNotes      String?       @map("admin_notes") // Notes internes admin
  shippedAt       DateTime?     @map("shipped_at")
  deliveredAt     DateTime?     @map("delivered_at")
  cancelledAt     DateTime?     @map("cancelled_at")
  cancellationReason String?    @map("cancellation_reason")
  createdAt       DateTime      @default(now()) @map("created_at")
  updatedAt       DateTime      @updatedAt @map("updated_at")

  // Relations
  user            User          @relation(fields: [userId], references: [id])
  address         Address?      @relation(fields: [addressId], references: [id])
  items           OrderItem[]
  payment         Payment?

  @@index([userId])
  @@index([orderNumber])
  @@index([status])
  @@index([createdAt])
  @@map("orders")
}

// ============================================================
// ORDER ITEM - Articles d'une commande
// ============================================================

model OrderItem {
  id            String    @id @default(uuid())
  orderId       String    @map("order_id")
  productId     String    @map("product_id")
  productName   String    @map("product_name")   // Copie du nom (si produit supprimé)
  productImage  String?   @map("product_image")  // Copie de l'image principale
  price         Decimal   @db.Decimal(10, 0)     // Prix au moment de la commande
  quantity      Int
  total         Decimal   @db.Decimal(10, 0)     // price * quantity

  // Relations
  order         Order     @relation(fields: [orderId], references: [id], onDelete: Cascade)
  product       Product   @relation(fields: [productId], references: [id])

  @@index([orderId])
  @@map("order_items")
}

// ============================================================
// PAYMENT - Paiements
// ============================================================

model Payment {
  id              String        @id @default(uuid())
  orderId         String        @unique @map("order_id") // Un paiement par commande
  amount          Decimal       @db.Decimal(10, 0) // Montant payé
  currency        String        @default("XOF")
  method          PaymentMethod
  status          PaymentStatus @default(PENDING)

  // Données du prestataire de paiement
  providerRef     String?       @map("provider_ref") // Référence transaction prestataire
  providerName    String?       @map("provider_name") // "paiementpro" ou "cinetpay"
  providerData    Json?         @map("provider_data") // Réponse complète du prestataire

  // Anti-fraude
  transactionId   String?       @unique @map("transaction_id") // Notre ID unique de transaction
  customerIp      String?       @map("customer_ip")
  verifiedAt      DateTime?     @map("verified_at") // Date de vérification callback

  paidAt          DateTime?     @map("paid_at")
  failedAt        DateTime?     @map("failed_at")
  failureReason   String?       @map("failure_reason")
  refundedAt      DateTime?     @map("refunded_at")
  createdAt       DateTime      @default(now()) @map("created_at")
  updatedAt       DateTime      @updatedAt @map("updated_at")

  // Relations
  order           Order         @relation(fields: [orderId], references: [id])

  @@index([transactionId])
  @@index([providerRef])
  @@index([status])
  @@map("payments")
}

// ============================================================
// SETTINGS - Paramètres du site (clé/valeur)
// ============================================================

model Setting {
  id            String    @id @default(uuid())
  key           String    @unique // Ex: "site_name", "shipping_fee_abidjan"
  value         String    // Valeur en string (à parser selon le type)
  type          String    @default("string") // "string", "number", "boolean", "json"
  description   String?   // Description pour l'admin
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  @@map("settings")
}
```

---

## Diagramme des relations

```
User 1──* Address
User 1──1 Cart
User 1──* Order
User 1──* Session

Cart 1──* CartItem
CartItem *──1 Product

Category 1──* Product
Category 1──* Category (self-relation: parent/children)

Product 1──* ProductImage
Product 1──* CartItem
Product 1──* OrderItem

Order 1──* OrderItem
Order 1──1 Payment
Order *──1 Address
Order *──1 User

OrderItem *──1 Product
```

---

## Index et performance

Les index suivants sont définis pour optimiser les requêtes les plus fréquentes :

| Table | Index | Utilisation |
|---|---|---|
| products | slug | Affichage page produit |
| products | categoryId | Filtrage par catégorie |
| products | status | Liste des produits actifs |
| products | isFeatured | Produits vedettes en accueil |
| products | name | Recherche par nom |
| orders | userId | Historique commandes client |
| orders | orderNumber | Recherche par numéro de commande |
| orders | status | Filtrage admin par statut |
| orders | createdAt | Tri chronologique |
| payments | transactionId | Vérification callback paiement |
| payments | providerRef | Réconciliation avec le prestataire |
| categories | slug | Navigation par catégorie |
| sessions | expiresAt | Nettoyage des sessions expirées |

---

## Données de seed (initiales)

```typescript
// prisma/seed.ts

import { PrismaClient, Role, ProductStatus } from '@prisma/client';
import * as bcrypt from 'bcrypt';

const prisma = new PrismaClient();

async function main() {
  // 1. Admin par défaut
  const adminPassword = await bcrypt.hash('Admin@2026!', 10);
  const admin = await prisma.user.upsert({
    where: { email: 'admin@votredomaine.com' },
    update: {},
    create: {
      email: 'admin@votredomaine.com',
      password: adminPassword,
      firstName: 'Admin',
      lastName: 'Principal',
      phone: '+2250700000000',
      role: Role.ADMIN,
    },
  });

  // 2. Catégories
  const categories = await Promise.all([
    prisma.category.upsert({
      where: { slug: 'nouveautes' },
      update: {},
      create: { name: 'Nouveautés', slug: 'nouveautes', sortOrder: 1 },
    }),
    prisma.category.upsert({
      where: { slug: 'populaires' },
      update: {},
      create: { name: 'Populaires', slug: 'populaires', sortOrder: 2 },
    }),
    prisma.category.upsert({
      where: { slug: 'promotions' },
      update: {},
      create: { name: 'Promotions', slug: 'promotions', sortOrder: 3 },
    }),
  ]);

  // 3. Paramètres par défaut
  const settings = [
    { key: 'site_name', value: 'Ma Boutique', type: 'string', description: 'Nom du site' },
    { key: 'site_description', value: 'Votre boutique en ligne', type: 'string', description: 'Description du site' },
    { key: 'currency', value: 'XOF', type: 'string', description: 'Devise' },
    { key: 'shipping_fee_default', value: '2000', type: 'number', description: 'Frais de livraison par défaut (FCFA)' },
    { key: 'free_shipping_threshold', value: '25000', type: 'number', description: 'Seuil livraison gratuite (FCFA)' },
    { key: 'tax_rate', value: '0', type: 'number', description: 'Taux de TVA (ex: 18 pour 18%)' },
    { key: 'whatsapp_number', value: '+2250700000000', type: 'string', description: 'Numéro WhatsApp' },
  ];

  for (const setting of settings) {
    await prisma.setting.upsert({
      where: { key: setting.key },
      update: {},
      create: setting,
    });
  }

  console.log('Seed terminé avec succès');
  console.log(`Admin créé : admin@votredomaine.com / Admin@2026!`);
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

---

## Notes importantes

### Conventions de nommage
- **Tables** : snake_case au pluriel (`users`, `products`, `order_items`)
- **Colonnes** : snake_case (`first_name`, `created_at`)
- **Modèles Prisma** : PascalCase singulier (`User`, `Product`, `OrderItem`)
- **Champs Prisma** : camelCase (`firstName`, `createdAt`)
- Le mapping `@@map()` et `@map()` assure la conversion automatique

### Prix et montants
- Tous les montants sont en `Decimal(10, 0)` car le FCFA n'a pas de centimes
- Le type `Decimal` évite les erreurs d'arrondi des `Float`
- Si la devise a des centimes (EUR), utiliser `Decimal(10, 2)`

### Données dénormalisées dans OrderItem
- `productName` et `productImage` sont copiés au moment de la commande
- Cela permet de conserver l'historique même si le produit est modifié/supprimé

### Extension possible : Variantes produits
Si le client a besoin de variantes (taille, couleur), ajouter :

```prisma
model ProductVariant {
  id        String  @id @default(uuid())
  productId String  @map("product_id")
  name      String  // "Taille", "Couleur"
  value     String  // "M", "Rouge"
  price     Decimal? @db.Decimal(10, 0) // Prix spécifique si différent
  stock     Int     @default(0)
  sku       String? @unique

  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@index([productId])
  @@map("product_variants")
}
```

### Extension possible : Codes promo

```prisma
model Coupon {
  id              String    @id @default(uuid())
  code            String    @unique
  discountType    String    @map("discount_type") // "percentage" ou "fixed"
  discountValue   Decimal   @map("discount_value") @db.Decimal(10, 0)
  minOrderAmount  Decimal?  @map("min_order_amount") @db.Decimal(10, 0)
  maxUses         Int?      @map("max_uses")
  usedCount       Int       @default(0) @map("used_count")
  isActive        Boolean   @default(true) @map("is_active")
  expiresAt       DateTime? @map("expires_at")
  createdAt       DateTime  @default(now()) @map("created_at")

  @@map("coupons")
}
```
