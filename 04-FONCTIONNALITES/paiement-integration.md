# Intégration Paiement

## Instructions pour l'agent IA

Ce document couvre l'intégration complète des prestataires de paiement pour le e-commerce en Afrique de l'Ouest. PaiementPro est le prestataire recommandé (testé en production). CinetPay est l'alternative.

**Important** : Ne JAMAIS stocker de données de carte bancaire sur le serveur. Tout passe par le prestataire.

---

## 1. PaiementPro - Intégration principale

### 1.1 Présentation

PaiementPro est une plateforme de paiement ivoirienne qui agrège :
- Cartes bancaires (Visa, Mastercard)
- Orange Money
- MTN Mobile Money
- Wave
- Moov Money / Flooz

**Site** : https://paiementpro.net
**Documentation** : https://paiementpro.net/docs/

### 1.2 Credentials nécessaires

| Variable | Description | Obtention |
|---|---|---|
| PAIEMENTPRO_MERCHANT_ID | Identifiant marchand | Fourni à l'inscription |
| PAIEMENTPRO_API_KEY | Clé API publique | Dashboard PaiementPro |
| PAIEMENTPRO_API_SECRET | Clé API secrète | Dashboard PaiementPro |

**Environnements** :
- **Sandbox** : pour les tests (transactions fictives)
- **Production** : pour les transactions réelles (nécessite validation du compte)

### 1.3 Flow de paiement complet

```
Client                    Notre API                  PaiementPro
  │                         │                           │
  │── [1] Clic "Payer" ───▶│                           │
  │                         │── [2] Init payment ──────▶│
  │                         │◀── [3] URL paiement ──────│
  │◀── [4] Redirect URL ───│                           │
  │                         │                           │
  │── [5] Redirigé ────────────────────────────────────▶│
  │      (page PaiementPro) │                           │
  │◀── [6] Paiement ───────────────────────────────────│
  │      (success/fail)     │                           │
  │                         │                           │
  │── [7] Retour ──────────▶│   ┌── [8] Callback ─────▶│
  │   (return URL)          │   │   (notification IPN)  │
  │                         │◀──┘                       │
  │                         │── [9] Vérification ──────▶│
  │                         │◀── [10] Confirmation ─────│
  │                         │                           │
  │◀── [11] Résultat ──────│                           │
  │   (succès ou échec)     │                           │
```

### 1.4 Etape 2 : Initiation du paiement

**Endpoint API** : POST /api/payments/init

**Requête vers PaiementPro** :

```typescript
async initPayment(orderId: string, channel: PaymentMethod, userId: string) {
  // 1. Récupérer la commande
  const order = await prisma.order.findUnique({
    where: { id: orderId },
    include: { payment: true, user: true },
  });

  // 2. Vérifications
  if (!order) throw new NotFoundException('Commande non trouvée');
  if (order.userId !== userId) throw new ForbiddenException();
  if (order.status !== 'PENDING') throw new BadRequestException('Commande déjà traitée');

  // 3. Générer un transactionId unique
  const transactionId = `TXN-${Date.now()}-${uuidv4().slice(0, 8)}`;

  // 4. Mapper le channel
  const channelMap = {
    CARD: 'credit_card',
    ORANGE_MONEY: 'orange_money_ci',
    MTN_MOMO: 'mtn_money_ci',
    WAVE: 'wave_ci',
    MOOV_FLOOZ: 'flooz_ci',
  };

  // 5. Appeler l'API PaiementPro
  const response = await axios.post(process.env.PAIEMENTPRO_BASE_URL, {
    merchantId: process.env.PAIEMENTPRO_MERCHANT_ID,
    amount: Number(order.total),
    currency: 'XOF',
    order_id: transactionId,
    description: `Commande ${order.orderNumber}`,
    channel: channelMap[channel],
    customer_email: order.user.email,
    customer_first_name: order.user.firstName,
    customer_last_name: order.user.lastName,
    customer_phone: order.user.phone || '',
    notify_url: process.env.PAIEMENTPRO_NOTIFY_URL,
    return_url: `${process.env.PAIEMENTPRO_RETURN_URL}?orderId=${order.id}`,
    cancel_url: `${process.env.PAIEMENTPRO_CANCEL_URL}&orderId=${order.id}`,
  }, {
    headers: {
      'Authorization': `Bearer ${process.env.PAIEMENTPRO_API_KEY}`,
      'Content-Type': 'application/json',
    },
  });

  // 6. Sauvegarder les infos de transaction
  await prisma.payment.update({
    where: { orderId: order.id },
    data: {
      transactionId,
      providerName: 'paiementpro',
      providerRef: response.data.ref || null,
      method: channel,
      status: 'PROCESSING',
    },
  });

  // 7. Retourner l'URL de paiement
  return {
    paymentUrl: response.data.payment_url,
    transactionId,
  };
}
```

### 1.5 Etape 8 : Callback (IPN - Instant Payment Notification)

**Endpoint** : POST /api/payments/callback

Le prestataire envoie une notification à cette URL quand le paiement est traité.

```typescript
async handleCallback(body: any) {
  // 1. Logger le callback complet (pour debug)
  console.log('Payment callback received:', JSON.stringify(body));

  // 2. Extraire les informations
  const { order_id, amount, status, transaction_id } = body;

  // 3. Retrouver le paiement par transactionId
  const payment = await prisma.payment.findUnique({
    where: { transactionId: order_id },
    include: { order: true },
  });

  if (!payment) {
    console.error(`Payment not found for transaction: ${order_id}`);
    return { status: 'error', message: 'Transaction not found' };
  }

  // 4. ANTI-REPLAY : Vérifier que le paiement est encore en PENDING/PROCESSING
  if (payment.status === 'COMPLETED') {
    console.warn(`Duplicate callback for transaction: ${order_id}`);
    return { status: 'ok', message: 'Already processed' };
  }

  // 5. VERIFICATION DU MONTANT : Le montant payé doit correspondre
  if (Number(amount) !== Number(payment.amount)) {
    console.error(`Amount mismatch: expected ${payment.amount}, got ${amount}`);
    await prisma.payment.update({
      where: { id: payment.id },
      data: {
        status: 'FAILED',
        failureReason: `Montant incorrect: attendu ${payment.amount}, reçu ${amount}`,
        failedAt: new Date(),
        providerData: body,
      },
    });
    return { status: 'error', message: 'Amount mismatch' };
  }

  // 6. Traiter selon le statut
  if (status === 'success' || status === 'approved') {
    // PAIEMENT REUSSI
    await prisma.$transaction([
      // Mettre à jour le paiement
      prisma.payment.update({
        where: { id: payment.id },
        data: {
          status: 'COMPLETED',
          providerRef: transaction_id,
          paidAt: new Date(),
          verifiedAt: new Date(),
          providerData: body,
        },
      }),
      // Mettre à jour la commande
      prisma.order.update({
        where: { id: payment.orderId },
        data: { status: 'CONFIRMED' },
      }),
    ]);

    return { status: 'ok' };
  } else {
    // PAIEMENT ECHOUE
    await prisma.payment.update({
      where: { id: payment.id },
      data: {
        status: 'FAILED',
        failureReason: body.error_message || 'Paiement refusé',
        failedAt: new Date(),
        providerData: body,
      },
    });

    return { status: 'ok' };
  }
}
```

### 1.6 Sécurité du callback

**Vérifications obligatoires** :

| Vérification | Description | Conséquence si échoue |
|---|---|---|
| Transaction existe | Le transactionId est en base | Ignorer le callback |
| Anti-replay | Le paiement n'est pas déjà COMPLETED | Ignorer (idempotent) |
| Montant correct | Le montant payé = montant attendu | Rejeter et marquer FAILED |
| Signature | Vérifier la signature du prestataire (si disponible) | Rejeter |

---

## 2. Moyens de paiement disponibles

### Channels PaiementPro

| Channel | Code API | Logo | Populaire en CI |
|---|---|---|---|
| Carte bancaire | credit_card | Visa/Mastercard | Moyen |
| Orange Money | orange_money_ci | Orange | Très populaire |
| MTN MoMo | mtn_money_ci | MTN | Populaire |
| Wave | wave_ci | Wave | Très populaire |
| Moov Flooz | flooz_ci | Moov | Modéré |

### Frais de transaction (indicatifs)

| Channel | Frais | Payé par |
|---|---|---|
| Carte bancaire | 2.5-3.5% | Marchand |
| Orange Money | 1-2% | Marchand |
| MTN MoMo | 1-2% | Marchand |
| Wave | 1% | Marchand |
| Moov Flooz | 1-2% | Marchand |

---

## 3. Interface utilisateur - Page de paiement

### 3.1 Choix du moyen de paiement

```
┌─────────────────────────────────────────────┐
│ PAIEMENT                                    │
│                                             │
│ Commande : CMD-20260315-0001                │
│ Montant  : 57 000 FCFA                      │
│                                             │
│ Choisissez votre moyen de paiement :        │
│                                             │
│ ┌───────────┐ ┌───────────┐ ┌───────────┐  │
│ │  [logo]   │ │  [logo]   │ │  [logo]   │  │
│ │   Carte   │ │  Orange   │ │   MTN     │  │
│ │ bancaire  │ │  Money    │ │  MoMo     │  │
│ └───────────┘ └───────────┘ └───────────┘  │
│                                             │
│ ┌───────────┐ ┌───────────┐                 │
│ │  [logo]   │ │  [logo]   │                 │
│ │   Wave    │ │   Moov    │                 │
│ │           │ │  Flooz    │                 │
│ └───────────┘ └───────────┘                 │
│                                             │
│      [PAYER 57 000 FCFA]                    │
│                                             │
│ 🔒 Paiement 100% sécurisé via PaiementPro │
└─────────────────────────────────────────────┘
```

**Comportement** :
1. Le client sélectionne un moyen de paiement (un seul à la fois)
2. Le bouton affiche "Payer [montant]"
3. Au clic : appel POST /api/payments/init avec le channel choisi
4. Loading spinner sur le bouton
5. Redirection vers la page PaiementPro
6. Après paiement : retour sur la page de résultat

### 3.2 Page de résultat (/payment/result)

**Paiement réussi** :
```
┌─────────────────────────────────────────────┐
│                                             │
│              ✓ (grand cercle vert)          │
│                                             │
│     Paiement réussi !                       │
│                                             │
│     Merci pour votre commande.              │
│     Votre commande CMD-20260315-0001        │
│     a été confirmée.                        │
│                                             │
│     Montant payé : 57 000 FCFA             │
│                                             │
│     [VOIR MA COMMANDE]                      │
│     [CONTINUER MES ACHATS]                  │
│                                             │
└─────────────────────────────────────────────┘
```

**Paiement échoué** :
```
┌─────────────────────────────────────────────┐
│                                             │
│              ✗ (grand cercle rouge)         │
│                                             │
│     Paiement non abouti                     │
│                                             │
│     Votre paiement n'a pas pu être          │
│     traité. Aucun montant n'a été débité.   │
│                                             │
│     [REESSAYER LE PAIEMENT]                 │
│     [CONTACTER LE SUPPORT]                  │
│                                             │
└─────────────────────────────────────────────┘
```

**En attente de confirmation** :
```
┌─────────────────────────────────────────────┐
│                                             │
│              ⟳ (spinner)                   │
│                                             │
│     Vérification du paiement...             │
│                                             │
│     Veuillez patienter pendant que nous     │
│     confirmons votre paiement.              │
│                                             │
└─────────────────────────────────────────────┘
```

**Logique de la page résultat** :
1. Récupérer le `orderId` depuis l'URL
2. Appeler GET /api/payments/:orderId/status
3. Si COMPLETED → afficher succès
4. Si FAILED → afficher échec
5. Si PROCESSING → afficher spinner, polling toutes les 3 secondes
6. Après 60 secondes de polling sans réponse → afficher "Paiement en cours de vérification, vous recevrez une confirmation"

---

## 4. CinetPay - Alternative

### 4.1 Présentation

CinetPay est une alternative à PaiementPro, présente dans plusieurs pays d'Afrique. L'intégration est similaire.

**Site** : https://cinetpay.com
**Documentation** : https://docs.cinetpay.com

### 4.2 Différences avec PaiementPro

| Aspect | PaiementPro | CinetPay |
|---|---|---|
| Pays couverts | Côte d'Ivoire principalement | 12+ pays africains |
| API | REST simple | REST + SDK disponible |
| Channels | Mobile Money + Carte | Mobile Money + Carte |
| Dashboard | Basique | Plus complet |
| Support | Réactif (WhatsApp/Email) | Email/Ticket |

### 4.3 Intégration CinetPay

```typescript
// Initiation du paiement CinetPay
const response = await axios.post('https://api-checkout.cinetpay.com/v2/payment', {
  apikey: process.env.CINETPAY_API_KEY,
  site_id: process.env.CINETPAY_SITE_ID,
  transaction_id: transactionId,
  amount: Number(order.total),
  currency: 'XOF',
  description: `Commande ${order.orderNumber}`,
  customer_name: `${order.user.firstName} ${order.user.lastName}`,
  customer_email: order.user.email,
  customer_phone_number: order.user.phone,
  notify_url: process.env.CINETPAY_NOTIFY_URL,
  return_url: process.env.CINETPAY_RETURN_URL,
  channels: 'ALL', // ou 'MOBILE_MONEY', 'CREDIT_CARD'
});

// L'URL de paiement est dans response.data.data.payment_url
```

### 4.4 Architecture interchangeable

Pour supporter les deux prestataires, utiliser une interface commune :

```typescript
interface PaymentProvider {
  initPayment(params: InitPaymentParams): Promise<InitPaymentResult>;
  handleCallback(body: any): Promise<CallbackResult>;
  verifyPayment(transactionId: string): Promise<PaymentStatus>;
}

class PaiementProProvider implements PaymentProvider { /* ... */ }
class CinetPayProvider implements PaymentProvider { /* ... */ }

// Factory
function getPaymentProvider(): PaymentProvider {
  const provider = process.env.PAYMENT_PROVIDER || 'paiementpro';
  switch (provider) {
    case 'paiementpro': return new PaiementProProvider();
    case 'cinetpay': return new CinetPayProvider();
    default: throw new Error(`Unknown payment provider: ${provider}`);
  }
}
```

---

## 5. Paiement à la livraison (optionnel)

Si le client souhaite proposer le paiement à la livraison :

### Flow
1. Le client choisit "Paiement à la livraison"
2. Pas de redirection vers un prestataire
3. La commande est créée avec paymentMethod = CASH et paymentStatus = PENDING
4. La commande passe directement en CONFIRMED (pas de paiement en ligne)
5. Le livreur collecte le paiement
6. L'admin passe le paiement en COMPLETED manuellement

### Risques
- Commandes non honorées (le client ne paie pas à la réception)
- Pas de garantie de paiement
- Gestion manuelle nécessaire

### Recommandation
- Si activé, exiger un numéro de téléphone vérifié
- Limiter le montant (ex: max 50 000 FCFA en paiement à la livraison)
- Historique client : bloquer si annulations répétées

---

## 6. Gestion des erreurs de paiement

### Messages d'erreur (en français)

| Situation | Message affiché |
|---|---|
| Paiement réussi | "Paiement effectué avec succès ! Merci pour votre commande." |
| Paiement refusé | "Le paiement a été refusé. Veuillez vérifier vos informations et réessayer." |
| Solde insuffisant | "Solde insuffisant. Veuillez recharger votre compte et réessayer." |
| Timeout | "Le paiement a expiré. Veuillez réessayer." |
| Annulé par l'utilisateur | "Paiement annulé. Votre commande est en attente de paiement." |
| Erreur technique | "Une erreur technique est survenue. Veuillez réessayer dans quelques minutes." |
| Commande déjà payée | "Cette commande a déjà été payée." |

### Mécanisme de retry
- Si le paiement échoue, la commande reste en PENDING
- Le client peut réessayer depuis son espace client (bouton "Finaliser le paiement")
- Possibilité de choisir un moyen de paiement différent au retry
- La commande PENDING expire après 48h (optionnel, à configurer)

---

## 7. Réconciliation et suivi

### Logs de transaction

Chaque transaction est loguée avec :
- transactionId (notre référence)
- providerRef (référence du prestataire)
- amount, currency
- channel (moyen de paiement)
- status (PENDING/PROCESSING/COMPLETED/FAILED)
- providerData (réponse complète du prestataire, en JSON)
- timestamps (createdAt, paidAt, failedAt)

### Vérification manuelle

Si le client affirme avoir payé mais le statut est PENDING :
1. Vérifier dans le dashboard PaiementPro/CinetPay avec la référence
2. Si le paiement est trouvé, mettre à jour manuellement en base
3. Contacter le support du prestataire si nécessaire

### Remboursements

Les remboursements se font via le dashboard du prestataire, pas via l'API :
1. L'admin passe la commande en CANCELLED ou REFUNDED
2. L'admin initie le remboursement dans le dashboard PaiementPro
3. L'admin met à jour le statut du paiement en REFUNDED
