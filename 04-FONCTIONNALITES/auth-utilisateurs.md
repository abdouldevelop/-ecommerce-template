# Authentification et Gestion des Utilisateurs

## Instructions pour l'agent IA

Ce document spécifie le système d'authentification complet. C'est le premier module à développer car il est requis par tous les autres (panier, commandes, admin, etc.).

---

## 1. Inscription (Register)

### Flow
1. L'utilisateur remplit le formulaire : prénom, nom, email, téléphone (optionnel), mot de passe, confirmation mot de passe
2. Validation côté client (UX immédiate)
3. Envoi POST /api/auth/register
4. Validation côté serveur
5. Hashage du mot de passe (bcrypt, 10 rounds)
6. Création de l'utilisateur en base (rôle CUSTOMER)
7. Génération d'un access token (JWT, 30 min) et d'un refresh token (JWT, 7 jours)
8. Stockage du hash du refresh token en base
9. Retour : user + accessToken + refreshToken
10. Frontend : stockage du token, redirection vers l'accueil, synchronisation du panier

### Règles de validation

| Champ | Règles | Message d'erreur |
|---|---|---|
| email | Format email valide, unique en base | "Adresse email invalide" / "Cette adresse email est déjà utilisée" |
| password | Min 8 caractères, au moins 1 majuscule, au moins 1 chiffre | "Le mot de passe doit contenir au moins 8 caractères, une majuscule et un chiffre" |
| confirmPassword | Identique à password | "Les mots de passe ne correspondent pas" |
| firstName | Min 2 caractères, max 50, pas de chiffres | "Le prénom doit contenir entre 2 et 50 caractères" |
| lastName | Min 2 caractères, max 50, pas de chiffres | "Le nom doit contenir entre 2 et 50 caractères" |
| phone | Format international optionnel (+225XXXXXXXXXX) | "Numéro de téléphone invalide" |

### Edge cases
- Email avec des espaces en début/fin : trimmer automatiquement
- Email en majuscules : convertir en minuscules
- Email déjà utilisé : retourner 400 avec message clair (pas 500)
- Mot de passe avec des espaces : autoriser (certains utilisent des passphrases)
- Injection HTML dans le nom : échapper les caractères spéciaux

---

## 2. Connexion (Login)

### Flow
1. L'utilisateur saisit email + mot de passe
2. Validation côté client
3. Envoi POST /api/auth/login
4. Recherche de l'utilisateur par email
5. Si non trouvé : erreur 401 "Identifiants incorrects" (message générique)
6. Si isActive === false : erreur 403 "Votre compte a été désactivé"
7. Comparaison bcrypt du mot de passe
8. Si incorrect : erreur 401 "Identifiants incorrects"
9. Mise à jour de lastLoginAt
10. Génération access token + refresh token
11. Retour : user + tokens
12. Frontend : stockage, redirection, sync panier

### Edge cases
- Email non trouvé : même message que mot de passe incorrect (sécurité)
- Plusieurs tentatives échouées : rate limiting (5/minute)
- Connexion depuis un nouvel appareil : pas de notification (V1), à ajouter en V2
- Token expiré en cours de navigation : intercepteur axios refresh automatique

---

## 3. JWT et Refresh Token

### Configuration des tokens

| Token | Durée | Secret | Payload |
|---|---|---|---|
| Access Token | 30 minutes | JWT_SECRET | { sub: userId, email, role, iat, exp } |
| Refresh Token | 7 jours | JWT_REFRESH_SECRET | { sub: userId, iat, exp } |

### Mécanisme de refresh

```
Client                    Serveur
  │                         │
  │── Requête API ─────────▶│
  │◀── 401 Unauthorized ────│
  │                         │
  │── POST /auth/refresh ──▶│
  │   (refreshToken)        │
  │                         │── Vérifie refresh token
  │                         │── Génère nouveau access + refresh
  │                         │── Invalide ancien refresh token
  │◀── Nouveau tokens ──────│
  │                         │
  │── Requête API (retry) ─▶│ (avec nouveau access token)
  │◀── 200 OK ──────────────│
```

### Implémentation côté frontend (intercepteur axios)

```typescript
// Pseudocode de l'intercepteur
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401 && !error.config._retry) {
      error.config._retry = true;
      try {
        const { accessToken, refreshToken } = await refreshTokenAPI();
        updateTokens(accessToken, refreshToken);
        error.config.headers.Authorization = `Bearer ${accessToken}`;
        return api(error.config); // Retry la requête originale
      } catch {
        logout(); // Refresh échoué → déconnexion
        redirect('/login');
      }
    }
    return Promise.reject(error);
  }
);
```

### Stockage des tokens côté frontend

| Donnée | Stockage | Raison |
|---|---|---|
| accessToken | Zustand store (mémoire) | Perdu au refresh de page, c'est ok car le refresh token restaure |
| refreshToken | localStorage | Persiste au refresh de page |
| user (profil) | Zustand store + localStorage | Affichage immédiat, revalidé avec l'API |

### Rotation des refresh tokens
- A chaque utilisation du refresh token, un NOUVEAU refresh token est généré
- L'ancien est invalidé (le hash en base est remplacé)
- Cela empêche la réutilisation d'un refresh token volé

---

## 4. Changement de mot de passe

### Flow
1. L'utilisateur est connecté
2. Il saisit : ancien mot de passe, nouveau mot de passe, confirmation
3. Envoi POST /api/auth/change-password
4. Vérification de l'ancien mot de passe (bcrypt.compare)
5. Si incorrect : erreur 400 "Mot de passe actuel incorrect"
6. Validation du nouveau mot de passe (mêmes règles que l'inscription)
7. Hashage et mise à jour en base
8. Invalidation de TOUS les refresh tokens (force la reconnexion partout)
9. Retour : succès
10. Frontend : toast de succès

### Edge cases
- Nouveau mot de passe identique à l'ancien : autoriser (pas de restriction)
- Utilisateur connecté sur plusieurs appareils : tous déconnectés après le changement

---

## 5. Déconnexion (Logout)

### Flow
1. L'utilisateur clique sur "Se déconnecter"
2. Envoi POST /api/auth/logout (avec le token actuel)
3. Serveur : met refreshToken à null en base
4. Frontend : vide le store Zustand, vide le localStorage, redirige vers l'accueil

### Edge cases
- Requête de logout avec un token expiré : accepter quand même (le token est dans le header)
- Double clic sur le bouton : ignorer si déjà en cours

---

## 6. Rôles et permissions

### Matrice d'accès

| Action | PUBLIC | CUSTOMER | ADMIN |
|---|---|---|---|
| Voir les produits | OUI | OUI | OUI |
| Voir les catégories | OUI | OUI | OUI |
| Rechercher des produits | OUI | OUI | OUI |
| S'inscrire | OUI | - | - |
| Se connecter | OUI | - | - |
| Gérer son panier | - | OUI | OUI |
| Passer une commande | - | OUI | OUI |
| Voir ses commandes | - | OUI | OUI |
| Modifier son profil | - | OUI | OUI |
| Gérer ses adresses | - | OUI | OUI |
| Créer/modifier des produits | - | - | OUI |
| Gérer les catégories | - | - | OUI |
| Voir toutes les commandes | - | - | OUI |
| Changer le statut des commandes | - | - | OUI |
| Voir les statistiques | - | - | OUI |
| Gérer les utilisateurs | - | - | OUI |

### Implémentation NestJS

```typescript
// Décorateurs personnalisés

// @Public() - Route accessible sans authentification
@SetMetadata('isPublic', true)

// @Roles('ADMIN') - Route réservée aux admins
@SetMetadata('roles', ['ADMIN'])

// @CurrentUser() - Extraire l'utilisateur du token
createParamDecorator((data, ctx) => {
  const request = ctx.switchToHttp().getRequest();
  return request.user;
})
```

---

## 7. Gestion de session (côté frontend)

### Au chargement de l'application (app mount)
1. Vérifier si un refreshToken existe dans localStorage
2. Si oui : appeler POST /api/auth/refresh pour obtenir un nouveau accessToken
3. Si le refresh réussit : l'utilisateur est connecté (restauration de session)
4. Si le refresh échoue : nettoyer le localStorage (session expirée)
5. Afficher l'application une fois l'état auth déterminé

### Synchronisation du panier à la connexion
1. Après un login ou register réussi
2. Récupérer les items du panier localStorage (Zustand persist)
3. Envoyer POST /api/cart/sync avec ces items
4. Remplacer le panier local par la réponse du serveur (panier fusionné)

---

## 8. Messages d'erreur (en français)

| Code HTTP | Situation | Message affiché |
|---|---|---|
| 400 | Email déjà utilisé | "Cette adresse email est déjà utilisée" |
| 400 | Validation échouée | Messages spécifiques par champ |
| 400 | Ancien mot de passe incorrect | "Le mot de passe actuel est incorrect" |
| 401 | Login échoué | "Identifiants incorrects" |
| 401 | Token expiré | (Géré silencieusement par le refresh) |
| 403 | Compte désactivé | "Votre compte a été désactivé. Contactez le support." |
| 403 | Accès non autorisé | "Vous n'avez pas les droits pour effectuer cette action" |
| 429 | Trop de tentatives | "Trop de tentatives. Réessayez dans quelques minutes." |
| 500 | Erreur serveur | "Une erreur est survenue. Veuillez réessayer." |
