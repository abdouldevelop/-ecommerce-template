# Documentation Client - Remise de Projet

## Instructions pour l'agent IA

Ce document est le template de documentation à remettre au client lors de la livraison du site. Remplir tous les champs `[A REMPLIR]` avec les informations spécifiques au projet. Ce document doit être compréhensible par une personne non technique.

---

## 1. Informations du projet

| Paramètre | Valeur |
|---|---|
| Nom du site | [A REMPLIR] |
| URL du site | [A REMPLIR - ex: https://maboutique.ci] |
| Date de livraison | [A REMPLIR] |
| Développeur / Agence | [A REMPLIR] |
| Contact support | [A REMPLIR - téléphone, email, WhatsApp] |

---

## 2. Accès au site

### 2.1 Panel d'administration

| Paramètre | Valeur |
|---|---|
| URL | [A REMPLIR - ex: https://maboutique.ci/admin] |
| Email admin | [A REMPLIR] |
| Mot de passe | [A REMPLIR - changer immédiatement après la première connexion] |

**Important** : Changez le mot de passe dès votre première connexion. Utilisez un mot de passe fort (minimum 8 caractères, avec des majuscules et des chiffres).

### 2.2 Accès au serveur (pour le mainteneur technique)

| Paramètre | Valeur |
|---|---|
| Hébergeur | [A REMPLIR - ex: OVH, Contabo] |
| Adresse IP du serveur | [A REMPLIR] |
| Accès SSH | [A REMPLIR - utilisateur + méthode] |
| Panel hébergeur | [A REMPLIR - URL + identifiants] |

### 2.3 Nom de domaine

| Paramètre | Valeur |
|---|---|
| Registrar | [A REMPLIR - ex: OVH, Namecheap] |
| URL panel registrar | [A REMPLIR] |
| Date d'expiration | [A REMPLIR - penser au renouvellement !] |

### 2.4 Paiement en ligne

| Paramètre | Valeur |
|---|---|
| Prestataire | [A REMPLIR - PaiementPro / CinetPay] |
| URL dashboard | [A REMPLIR] |
| Identifiant | [A REMPLIR] |
| Moyens de paiement activés | [A REMPLIR - Carte, Orange Money, MTN, Wave, Moov] |

---

## 3. Guide d'utilisation du panel admin

### 3.1 Dashboard

Le dashboard est la page d'accueil de l'administration. Il affiche :

- **Chiffre d'affaires total** : le montant total de toutes les commandes
- **CA payé** : montant des commandes effectivement payées
- **CA en attente** : montant des commandes dont le paiement n'a pas encore abouti
- **Nombre de commandes** : total des commandes sur le site
- **Nombre de clients** : total des clients inscrits
- **Produits avec stock bas** : nombre de produits dont le stock est inférieur au seuil d'alerte

Le tableau des clients en bas du dashboard montre pour chaque client :
- Le nombre de commandes passées
- Le nombre de commandes payées vs en attente
- Le montant total payé

### 3.2 Gérer les produits

#### Ajouter un nouveau produit

1. Cliquez sur **Produits** dans le menu latéral
2. Cliquez sur le bouton **+ Nouveau produit**
3. Remplissez les informations :
   - **Nom** : le nom du produit tel qu'il apparaîtra sur le site
   - **Description courte** : une phrase résumant le produit (max 160 caractères, important pour Google)
   - **Description longue** : description détaillée du produit
   - **Prix** : le prix de vente en FCFA
   - **Ancien prix** : si le produit est en promotion, indiquez l'ancien prix (il sera affiché barré)
   - **Stock** : la quantité disponible
   - **Catégorie** : sélectionnez la catégorie du produit
4. Ajoutez au moins **une image** :
   - Cliquez sur la zone d'upload ou glissez-déposez une image
   - L'image doit faire maximum 5 Mo (JPG, PNG ou WebP)
   - Recommandation : au moins 800x800 pixels
   - La première image sera l'image principale (celle affichée dans la liste)
5. Choisissez le **statut** :
   - **Brouillon** : le produit ne sera pas visible sur le site (vous pouvez le préparer tranquillement)
   - **Actif** : le produit sera visible et achetable sur le site
   - **Archivé** : le produit n'est plus en vente mais reste dans la base
6. Cochez **Produit vedette** si vous souhaitez que ce produit apparaisse en page d'accueil
7. Cliquez sur **Enregistrer**

#### Modifier un produit

1. Dans la liste des produits, cliquez sur **Modifier** à côté du produit
2. Modifiez les champs souhaités
3. Cliquez sur **Enregistrer**

#### Supprimer un produit

1. Dans la liste des produits, cliquez sur **Supprimer** à côté du produit
2. Confirmez la suppression dans la fenêtre qui apparaît
3. Si le produit a des commandes, il sera archivé (pas supprimé définitivement)

#### Gérer le stock

- Le stock est décrémenté automatiquement quand un client passe une commande
- Quand le stock atteint 0, le produit affiche "Rupture de stock" et ne peut plus être acheté
- Pour réapprovisionner : modifiez le produit et changez la quantité en stock

### 3.3 Gérer les catégories

#### Ajouter une catégorie

1. Cliquez sur **Catégories** dans le menu
2. Cliquez sur **+ Nouvelle catégorie**
3. Remplissez le nom de la catégorie
4. Ajoutez une description et une image (optionnel)
5. Cliquez sur **Enregistrer**

#### Supprimer une catégorie

- Une catégorie ne peut être supprimée que si elle ne contient aucun produit
- Déplacez d'abord les produits vers une autre catégorie

### 3.4 Gérer les commandes

#### Voir les commandes

1. Cliquez sur **Commandes** dans le menu
2. Vous verrez la liste de toutes les commandes avec :
   - Le numéro de commande (ex: CMD-20260315-0001)
   - Le nom du client
   - La date
   - Le montant
   - Le statut de la commande
   - Le statut du paiement

#### Comprendre les statuts

| Statut | Signification | Ce que vous devez faire |
|---|---|---|
| En attente | Le client a passé commande mais n'a pas encore payé | Attendre le paiement |
| Confirmée | Le paiement a été reçu | Préparer la commande |
| En préparation | Vous préparez la commande | Préparer les articles |
| Expédiée | La commande a été envoyée | Le client est notifié |
| Livrée | Le client a reçu sa commande | Commande terminée |
| Annulée | La commande a été annulée | Le stock est restauré |

#### Changer le statut d'une commande

1. Cliquez sur le bouton de statut à côté de la commande
2. Choisissez le nouveau statut dans la liste
3. Ajoutez une note si nécessaire (ex: "Envoyée avec le livreur Moussa")
4. Confirmez le changement

**Recommandation de workflow** :
1. Le client passe commande → statut **En attente**
2. Le paiement est reçu → statut automatiquement **Confirmée**
3. Vous préparez la commande → passez en **En préparation**
4. Vous envoyez la commande → passez en **Expédiée**
5. Le client confirme la réception → passez en **Livrée**

### 3.5 Voir les clients

- La page **Clients** affiche la liste de tous les clients inscrits
- Pour chaque client, vous voyez : nom, email, téléphone, nombre de commandes, montant total
- Vous pouvez rechercher un client par son nom ou son email

---

## 4. Fonctionnement du site pour les clients

### 4.1 Parcours client type

1. Le client arrive sur le site et navigue dans les produits
2. Il ajoute des produits à son panier
3. Quand il est prêt, il clique sur "Commander"
4. Il se connecte ou crée un compte (s'il n'en a pas)
5. Il saisit son adresse de livraison
6. Il voit le récapitulatif de sa commande
7. Il choisit son moyen de paiement (Orange Money, Wave, carte, etc.)
8. Il est redirigé vers la page de paiement du prestataire
9. Il effectue le paiement
10. Il revient sur le site et voit le résultat (succès ou échec)
11. Il peut suivre sa commande dans son espace client

### 4.2 Paiement

- Les paiements sont gérés par **[A REMPLIR - PaiementPro / CinetPay]**
- Les fonds sont collectés sur votre compte marchand
- Vous pouvez consulter les transactions dans le dashboard du prestataire
- Les frais de transaction sont : [A REMPLIR - ex: 2% par transaction]
- Les virements vers votre compte bancaire se font : [A REMPLIR - ex: automatiquement chaque semaine]

### 4.3 Commandes en attente de paiement

- Si un client ne finalise pas son paiement, la commande reste en "En attente"
- Le client peut revenir dans son espace client et cliquer sur "Finaliser le paiement"
- Les commandes en attente n'affectent pas votre stock (le stock a déjà été réservé)

---

## 5. Maintenance et support

### 5.1 Ce qui est inclus

| Service | Inclus | Durée |
|---|---|---|
| Corrections de bugs | [A REMPLIR - ex: Oui, 30 jours] | [A REMPLIR] |
| Support par email/WhatsApp | [A REMPLIR - ex: Oui, 30 jours] | [A REMPLIR] |
| Mises à jour de sécurité | [A REMPLIR] | [A REMPLIR] |
| Ajout de nouvelles fonctionnalités | [A REMPLIR - ex: Non, sur devis] | - |
| Formation complémentaire | [A REMPLIR] | - |

### 5.2 Ce que vous pouvez faire vous-même

- Ajouter, modifier ou supprimer des produits
- Créer ou modifier des catégories
- Changer le statut des commandes
- Consulter les statistiques et les clients
- Modifier les textes du slider de la page d'accueil (si configurable)

### 5.3 Ce qui nécessite un développeur

- Modifier le design du site (couleurs, disposition, polices)
- Ajouter de nouvelles fonctionnalités (wishlist, codes promo, blog, etc.)
- Modifier les pages statiques (A propos, CGV)
- Résoudre les problèmes techniques (erreurs serveur, problèmes de paiement)
- Mettre à jour les logiciels du serveur

### 5.4 Sauvegardes

- La base de données est sauvegardée automatiquement **tous les jours** à 3h du matin
- Les sauvegardes sont conservées pendant **30 jours**
- Les images des produits sont sauvegardées [A REMPLIR - quotidiennement / hebdomadairement]
- En cas de problème, nous pouvons restaurer les données

### 5.5 Renouvellements à prévoir

| Service | Fréquence | Prix estimé | Date de renouvellement |
|---|---|---|---|
| Nom de domaine | Annuel | [A REMPLIR] | [A REMPLIR] |
| Hébergement (serveur) | Mensuel/Annuel | [A REMPLIR] | [A REMPLIR] |
| Certificat SSL | Automatique (gratuit) | Gratuit (Let's Encrypt) | Auto-renouvelé |
| Compte prestataire paiement | - | Frais par transaction | - |

---

## 6. FAQ - Questions fréquentes

### "Le site est lent / ne s'affiche pas"

1. Vérifiez que le serveur est en ligne (contactez le support)
2. Videz le cache de votre navigateur
3. Essayez depuis un autre navigateur ou un autre appareil
4. Si le problème persiste, contactez le support technique

### "Un client dit avoir payé mais la commande est en attente"

1. Connectez-vous au dashboard du prestataire de paiement
2. Recherchez la transaction par le numéro de commande
3. Si le paiement est confirmé dans le dashboard, contactez le support pour synchroniser
4. Si le paiement n'apparaît pas, demandez au client de vérifier avec son opérateur mobile

### "Je veux ajouter un nouveau moyen de paiement"

Contactez le support technique. L'ajout d'un nouveau moyen de paiement nécessite une configuration chez le prestataire et dans le code du site.

### "Je veux changer le prix d'un produit"

1. Allez dans **Produits** > **Modifier** le produit concerné
2. Changez le prix
3. Si vous voulez afficher l'ancien prix barré, remplissez le champ "Ancien prix"
4. Cliquez sur **Enregistrer**
5. Le changement est immédiat sur le site

### "Je veux mettre un produit en promotion"

1. Modifiez le produit
2. Mettez le nouveau prix dans le champ **Prix**
3. Mettez l'ancien prix dans le champ **Ancien prix**
4. Le site affichera automatiquement l'ancien prix barré et le badge de réduction

### "Le certificat SSL a expiré"

Le certificat est renouvelé automatiquement. Si vous voyez une erreur de sécurité dans le navigateur, contactez immédiatement le support technique.

### "Je veux modifier la page A propos / CGV"

Contactez le support technique avec le nouveau texte. La modification sera faite dans les 24h.

### "J'ai oublié mon mot de passe admin"

Contactez le support technique. Le mot de passe sera réinitialisé manuellement.

---

## 7. Contacts utiles

| Service | Contact | Notes |
|---|---|---|
| Support technique (développeur) | [A REMPLIR] | [A REMPLIR - horaires, délai de réponse] |
| Hébergeur | [A REMPLIR] | [A REMPLIR] |
| Prestataire paiement | [A REMPLIR] | [A REMPLIR] |
| Registrar (nom de domaine) | [A REMPLIR] | [A REMPLIR] |

---

## 8. Signatures de remise

| Rôle | Nom | Date | Signature |
|---|---|---|---|
| Développeur / Agence | [A REMPLIR] | [A REMPLIR] | __________ |
| Client (réceptionnaire) | [A REMPLIR] | [A REMPLIR] | __________ |

**Le client reconnaît avoir reçu :**
- [ ] Les accès au panel d'administration
- [ ] Les accès au serveur (si contrat de maintenance)
- [ ] Les accès au prestataire de paiement
- [ ] La présente documentation
- [ ] Une formation à l'utilisation du panel admin

**Date de fin de garantie (corrections de bugs)** : [A REMPLIR]
