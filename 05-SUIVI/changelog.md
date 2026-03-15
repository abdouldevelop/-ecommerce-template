# Changelog - E-Commerce

## Instructions pour l'agent IA

Ce fichier documente toutes les modifications apportées au projet, organisées par version. Mettre à jour ce fichier à chaque livraison ou déploiement important.

**Format** : [Semantic Versioning](https://semver.org/lang/fr/) - MAJEUR.MINEUR.PATCH
- MAJEUR : Changements incompatibles (refonte majeure)
- MINEUR : Nouvelles fonctionnalités rétrocompatibles
- PATCH : Corrections de bugs

**Catégories** :
- **Added** : Nouvelles fonctionnalités
- **Changed** : Modifications de fonctionnalités existantes
- **Fixed** : Corrections de bugs
- **Security** : Corrections de sécurité
- **Removed** : Fonctionnalités supprimées
- **Deprecated** : Fonctionnalités qui seront supprimées dans une future version

---

## [Non publié]

### Added
- [A REMPLIR au fur et à mesure du développement]

### Changed
- [A REMPLIR]

### Fixed
- [A REMPLIR]

---

## [1.0.0] - [DATE DE LANCEMENT]

### Added
- Boutique en ligne complète (catalogue, produits, catégories)
- Système d'authentification (inscription, connexion, JWT + refresh token)
- Panier (ajout, suppression, modification quantité, persistence localStorage)
- Processus de commande complet (checkout, adresse, récapitulatif)
- Intégration PaiementPro (carte bancaire, Orange Money, MTN MoMo, Wave, Moov Flooz)
- Page de résultat de paiement (succès, échec, en attente)
- Espace client (dashboard, historique commandes, profil)
- Bouton "Finaliser le paiement" pour les commandes en attente
- Panel d'administration (dashboard, produits CRUD, catégories, commandes, clients)
- Recherche de produits avec autocomplete
- Filtrage par catégorie
- Slider / carrousel en page d'accueil
- Produits vedettes en page d'accueil
- Page A propos, CGV, Politique de confidentialité, Contact
- Bouton WhatsApp flottant
- Design responsive mobile-first
- Thème sombre pour le panel admin

### Security
- Authentification JWT avec rotation des refresh tokens
- Hachage bcrypt des mots de passe
- Validation de toutes les entrées utilisateur (côté serveur)
- CORS configuré
- HTTPS obligatoire
- Headers de sécurité HTTP (X-Frame-Options, HSTS, etc.)
- Protection des routes admin (vérification de rôle)
- Vérification de propriété des données (un client ne voit que ses données)
- Anti-replay et vérification de montant pour les callbacks de paiement

---

## [0.9.0] - [DATE BETA]

### Added
- [Fonctionnalités de la beta, si applicable]

### Known Issues
- [Bugs connus à corriger avant le lancement]

---

## Template pour les futures versions

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- Description de la nouvelle fonctionnalité

### Changed
- Description du changement

### Fixed
- Description du bug corrigé (#ID si issue tracker)

### Security
- Description de la correction de sécurité

### Removed
- Description de ce qui a été supprimé
```

---

## Historique des déploiements

| Version | Date | Environnement | Notes |
|---|---|---|---|
| 1.0.0 | [A REMPLIR] | Production | Lancement initial |
| 0.9.0 | [A REMPLIR] | Staging | Beta test |
| 0.1.0 | [A REMPLIR] | Développement | Premier prototype |
