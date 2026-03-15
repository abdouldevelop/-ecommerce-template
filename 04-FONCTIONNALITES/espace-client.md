# Espace Client

## Instructions pour l'agent IA

Ce document spécifie l'espace client complet : dashboard avec statistiques, historique des commandes avec détails expandables, gestion du profil et pages de connexion/inscription.

---

## 1. Dashboard client (/account)

### Layout

```
┌─────────────────────────────────────────────────────┐
│ HEADER (boutique)                                   │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Bonjour, Abdoul 👋                                │
│                                                     │
│  ┌───────────────┐ ┌───────────────┐ ┌────────────┐│
│  │ 📦 5          │ │ ✅ 4          │ │ ⏳ 1       ││
│  │ Commandes     │ │ Payées        │ │ En attente ││
│  │ totales       │ │               │ │            ││
│  └───────────────┘ └───────────────┘ └────────────┘│
│                                                     │
│  ─── Mes dernières commandes ───                    │
│                                                     │
│  [Liste des 5 dernières commandes]                  │
│                                                     │
│  [Voir toutes mes commandes →]                      │
│                                                     │
│  ─── Liens rapides ───                              │
│                                                     │
│  [📦 Mes commandes] [👤 Mon profil] [🔓 Déconnexion]│
│                                                     │
├─────────────────────────────────────────────────────┤
│ FOOTER                                              │
└─────────────────────────────────────────────────────┘
```

### Cartes de statistiques

| Carte | Données | Couleur | Icone |
|---|---|---|---|
| Commandes totales | Nombre total de commandes du client | Bleu | Package |
| Commandes payées | Commandes avec paiement COMPLETED | Vert | Check circle |
| Commandes en attente | Commandes avec paiement PENDING | Orange | Clock |

### Données nécessaires (API)

```json
// GET /api/orders?summary=true (ou un endpoint dédié)
{
  "summary": {
    "totalOrders": 5,
    "paidOrders": 4,
    "pendingOrders": 1,
    "totalPaid": 180000,
    "totalPending": 32000
  },
  "recentOrders": [
    {
      "id": "uuid",
      "orderNumber": "CMD-20260315-0001",
      "total": 57000,
      "status": "DELIVERED",
      "paymentStatus": "COMPLETED",
      "createdAt": "2026-03-15T14:30:00Z"
    }
  ]
}
```

### Protection de la route

- Route protégée : redirection vers /login si non connecté
- Middleware Next.js ou vérification dans le composant
- Si le token est expiré : tenter un refresh automatique avant de rediriger

---

## 2. Historique des commandes (/account/orders)

### Liste des commandes

Chaque commande est affichée en tant que carte avec possibilité d'expansion :

**Carte fermée** :
```
┌─────────────────────────────────────────────────┐
│ CMD-20260315-0001              15 mars 2026      │
│ 57 000 FCFA                    2 articles        │
│                                                   │
│ ● En attente          ● Paiement en attente      │
│                                                   │
│ [Finaliser le paiement]              [Détails ▼] │
└─────────────────────────────────────────────────┘
```

**Carte ouverte (expandée)** :
```
┌─────────────────────────────────────────────────┐
│ CMD-20260315-0001              15 mars 2026      │
│ 57 000 FCFA                    2 articles        │
│                                                   │
│ ● En attente          ● Paiement en attente      │
│                                                   │
│ [Finaliser le paiement]              [Détails ▲] │
├─────────────────────────────────────────────────┤
│                                                   │
│ Articles :                                        │
│ ┌──────┬────────────────┬──────┬──────────────┐  │
│ │[img] │ T-Shirt Premium│  x2  │ 30 000 FCFA │  │
│ │[img] │ Pantalon Class.│  x1  │ 25 000 FCFA │  │
│ └──────┴────────────────┴──────┴──────────────┘  │
│                                                   │
│ Sous-total :                      55 000 FCFA    │
│ Livraison :                        2 000 FCFA    │
│ Total :                           57 000 FCFA    │
│                                                   │
│ Adresse de livraison :                           │
│ Abdoul Diallo                                     │
│ Angré 8ème tranche, Cocody, Abidjan              │
│ +225 07 01 23 45 67                              │
│                                                   │
│ Moyen de paiement : Orange Money                 │
│                                                   │
└─────────────────────────────────────────────────┘
```

### Badges de statut commande

| Statut | Badge | Couleur fond | Couleur texte |
|---|---|---|---|
| PENDING | "En attente" | bg-yellow-100 | text-yellow-800 |
| CONFIRMED | "Confirmée" | bg-blue-100 | text-blue-800 |
| PROCESSING | "En préparation" | bg-purple-100 | text-purple-800 |
| SHIPPED | "Expédiée" | bg-indigo-100 | text-indigo-800 |
| DELIVERED | "Livrée" | bg-green-100 | text-green-800 |
| CANCELLED | "Annulée" | bg-red-100 | text-red-800 |

### Badges de statut paiement

| Statut | Badge | Couleur |
|---|---|---|
| PENDING | "Paiement en attente" | Orange |
| PROCESSING | "Paiement en cours" | Bleu |
| COMPLETED | "Payée" | Vert |
| FAILED | "Paiement échoué" | Rouge |
| CANCELLED | "Paiement annulé" | Gris |

### Bouton "Finaliser le paiement"

**Conditions d'affichage** :
- Statut commande = PENDING
- Statut paiement = PENDING ou FAILED

**Comportement** :
1. Clic → redirige vers /payment?orderId=xxx
2. La page de paiement affiche le récapitulatif et les moyens de paiement
3. Le client peut choisir un moyen de paiement (y compris différent du premier essai)
4. Redirection vers le prestataire puis retour sur la page résultat

### Pagination et tri

- 10 commandes par page
- Tri par défaut : plus récente en premier
- Filtre optionnel par statut (dropdown)
- Bouton "Charger plus" ou pagination numérique

---

## 3. Profil (/account/profile)

### Formulaire de profil

```
┌─────────────────────────────────────────────┐
│ MON PROFIL                                   │
│                                              │
│ Prénom :     [Abdoul            ]            │
│ Nom :        [Diallo            ]            │
│ Email :      client@email.com (non modifiable)│
│ Téléphone :  [+225 07 01 23 45 67]           │
│                                              │
│              [Enregistrer les modifications]  │
│                                              │
│ ─── Changer le mot de passe ───              │
│                                              │
│ Mot de passe actuel : [••••••••]             │
│ Nouveau mot de passe : [••••••••]            │
│ Confirmer :            [••••••••]            │
│                                              │
│              [Changer le mot de passe]        │
│                                              │
│ ─── Compte ───                               │
│                                              │
│ Membre depuis : 10 janvier 2026              │
│ Dernière connexion : 15 mars 2026            │
│                                              │
└─────────────────────────────────────────────┘
```

### Règles de modification

| Champ | Modifiable | Validation |
|---|---|---|
| Prénom | Oui | 2-50 caractères |
| Nom | Oui | 2-50 caractères |
| Email | Non | Lecture seule (grisé) |
| Téléphone | Oui | Format international ou vide |

### Changement de mot de passe

| Champ | Validation |
|---|---|
| Mot de passe actuel | Requis, vérifié côté serveur |
| Nouveau mot de passe | Min 8 chars, 1 majuscule, 1 chiffre |
| Confirmation | Identique au nouveau mot de passe |

**Flow** :
1. L'utilisateur remplit les 3 champs
2. Validation côté client
3. Envoi POST /api/auth/change-password
4. Si ancien mot de passe incorrect → message d'erreur
5. Si succès → toast "Mot de passe modifié avec succès"
6. L'utilisateur reste connecté (nouveau token généré)

---

## 4. Page de connexion (/login)

### Layout

```
┌─────────────────────────────────────────────┐
│                                              │
│              [LOGO]                          │
│                                              │
│         Bienvenue !                          │
│         Connectez-vous à votre compte        │
│                                              │
│ Email :          [                    ]       │
│ Mot de passe :   [••••••••           ]       │
│                                              │
│              [SE CONNECTER]                  │
│                                              │
│ Pas encore de compte ?                       │
│ Créer un compte →                            │
│                                              │
└─────────────────────────────────────────────┘
```

### Spécifications

- Centré verticalement et horizontalement
- Max-width : 400px
- Logo du site en haut
- Input email avec type="email" et autocomplete="email"
- Input password avec type="password" et autocomplete="current-password"
- Toggle visibilité du mot de passe (icône oeil)
- Bouton "Se connecter" : pleine largeur, couleur primaire
- Loading state sur le bouton pendant l'appel API
- Messages d'erreur sous les champs concernés
- Lien "Créer un compte" vers /register

### Comportement après connexion

1. Stocker les tokens (store + localStorage)
2. Stocker les infos user dans le store
3. Synchroniser le panier local avec le serveur
4. Rediriger vers :
   - La page précédente si l'utilisateur a été redirigé depuis un checkout
   - L'accueil si connexion directe
   - Le dashboard admin si l'utilisateur est ADMIN

---

## 5. Page d'inscription (/register)

### Layout

```
┌─────────────────────────────────────────────┐
│                                              │
│              [LOGO]                          │
│                                              │
│         Créer votre compte                   │
│                                              │
│ Prénom :         [                    ]       │
│ Nom :            [                    ]       │
│ Email :          [                    ]       │
│ Téléphone :      [                    ]       │
│ Mot de passe :   [••••••••           ]       │
│                  [■■■■□□□□ Moyen]            │
│ Confirmer :      [••••••••           ]       │
│                                              │
│              [CREER MON COMPTE]              │
│                                              │
│ Déjà un compte ?                             │
│ Se connecter →                               │
│                                              │
└─────────────────────────────────────────────┘
```

### Indicateur de force du mot de passe

| Force | Conditions | Couleur | Label |
|---|---|---|---|
| Faible | < 8 caractères | Rouge | "Faible" |
| Moyen | 8+ chars, 1 critère (majuscule OU chiffre) | Orange | "Moyen" |
| Fort | 8+ chars, majuscule ET chiffre | Vert | "Fort" |
| Très fort | 12+ chars, majuscule ET chiffre ET caractère spécial | Vert foncé | "Très fort" |

### Validation en temps réel

- Chaque champ est validé au blur (quand l'utilisateur quitte le champ)
- Le bouton "Créer mon compte" est désactivé tant que le formulaire n'est pas valide
- Messages d'erreur sous chaque champ en rouge
- Le champ "Confirmer" vérifie la correspondance en temps réel

### Comportement après inscription

1. Appel POST /api/auth/register
2. Si succès : connexion automatique (tokens reçus dans la réponse)
3. Redirection vers l'accueil
4. Toast : "Compte créé avec succès ! Bienvenue."
5. Synchronisation du panier local

---

## 6. Gestion des adresses (/account/addresses) - V2

### Note

La gestion des adresses comme page dédiée est une fonctionnalité V2. En V1, les adresses sont gérées directement dans le checkout.

### Layout (V2)

```
┌─────────────────────────────────────────────┐
│ MES ADRESSES                                 │
│                                              │
│ ┌────────────────────────────────────────┐   │
│ │ ★ Maison (par défaut)                  │   │
│ │ Abdoul Diallo                          │   │
│ │ Angré 8ème tranche, Cocody, Abidjan    │   │
│ │ +225 07 01 23 45 67                    │   │
│ │            [Modifier] [Supprimer]      │   │
│ └────────────────────────────────────────┘   │
│                                              │
│ ┌────────────────────────────────────────┐   │
│ │ Bureau                                 │   │
│ │ Abdoul Diallo                          │   │
│ │ Plateau, Immeuble CCIA, 5ème étage    │   │
│ │ +225 07 01 23 45 67                    │   │
│ │    [Par défaut] [Modifier] [Supprimer] │   │
│ └────────────────────────────────────────┘   │
│                                              │
│ [+ Ajouter une adresse]                     │
│                                              │
└─────────────────────────────────────────────┘
```

---

## 7. Navigation de l'espace client

### Sidebar (desktop) ou Menu (mobile)

```
Mon compte
├── Dashboard
├── Mes commandes
├── Mon profil
├── Mes adresses (V2)
└── Se déconnecter
```

### Responsive

- **Desktop** : sidebar à gauche, contenu à droite
- **Mobile** : liens en haut de page (scroll horizontal) ou dans le header de la section
- Le header de la boutique reste visible (l'utilisateur peut continuer à naviguer)

---

## 8. Protection et redirections

### Middleware de protection

```typescript
// Middleware Next.js ou composant wrapper
function withAuth(Component) {
  return function AuthenticatedComponent(props) {
    const { isAuthenticated, isLoading } = useAuthStore();
    const router = useRouter();

    useEffect(() => {
      if (!isLoading && !isAuthenticated) {
        router.push(`/login?redirect=${encodeURIComponent(router.asPath)}`);
      }
    }, [isAuthenticated, isLoading]);

    if (isLoading) return <LoadingSpinner />;
    if (!isAuthenticated) return null;

    return <Component {...props} />;
  };
}
```

### Redirections

| Situation | Action |
|---|---|
| Non connecté → page protégée | Redirect vers /login?redirect=URL_ORIGINALE |
| Login réussi avec redirect | Redirect vers l'URL sauvegardée |
| Login réussi sans redirect | Redirect vers /account (CUSTOMER) ou /admin (ADMIN) |
| Token expiré | Refresh automatique, si échoue → /login |
| CUSTOMER tente /admin | Redirect vers /account avec message "Accès non autorisé" |
