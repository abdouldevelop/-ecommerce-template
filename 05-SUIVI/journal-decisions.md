# Journal des Décisions

## Instructions pour l'agent IA

Ce journal documente toutes les décisions architecturales et de design prises pendant le projet. Chaque décision est datée et justifiée. En cas de question ou de changement futur, ce journal permet de comprendre le "pourquoi" de chaque choix.

**Format** : Une entrée par décision importante. Les décisions mineures (choix de nommage, style CSS) n'ont pas besoin d'être documentées.

---

## Template d'entrée

```
### DEC-XXX : [Titre court de la décision]
- **Date** : JJ/MM/AAAA
- **Contexte** : [Quel problème ou question a mené à cette décision ?]
- **Décision** : [Qu'est-ce qui a été décidé ?]
- **Alternatives considérées** :
  1. [Alternative 1] - [Raison du rejet]
  2. [Alternative 2] - [Raison du rejet]
- **Conséquences** : [Impact de cette décision sur le projet]
- **Statut** : ACCEPTEE / REVISEE / ANNULEE
```

---

## Décisions architecturales

### DEC-001 : Stack technique NestJS + Next.js + PostgreSQL
- **Date** : [A REMPLIR]
- **Contexte** : Choix de la stack technique pour le projet e-commerce. Le client n'a pas de préférence technique.
- **Décision** : Utiliser NestJS pour le backend, Next.js 15 pour le frontend, PostgreSQL 16 pour la base de données, Prisma comme ORM.
- **Alternatives considérées** :
  1. Express.js + React SPA - Rejeté car pas de SSR (mauvais SEO) et pas de structure imposée
  2. Laravel + Vue.js - Rejeté car l'équipe est plus forte en TypeScript/Node.js
  3. Strapi + Next.js - Rejeté car Strapi impose trop de contraintes pour un e-commerce custom
- **Conséquences** : Typage complet TypeScript de bout en bout. SSR pour le SEO des pages produits. Architecture modulaire facilement extensible.
- **Statut** : ACCEPTEE

### DEC-002 : Monorepo Turborepo
- **Date** : [A REMPLIR]
- **Contexte** : Organisation du code pour le frontend et le backend.
- **Décision** : Utiliser un monorepo Turborepo avec `apps/api` et `apps/web`.
- **Alternatives considérées** :
  1. Deux repos séparés - Rejeté car plus complexe à synchroniser les types
  2. NX - Rejeté car trop complexe pour un projet de cette taille
- **Conséquences** : Un seul `git clone`, partage facile des types TypeScript, builds parallèles avec cache.
- **Statut** : ACCEPTEE

### DEC-003 : JWT pour l'authentification
- **Date** : [A REMPLIR]
- **Contexte** : Choix du mécanisme d'authentification.
- **Décision** : JWT avec access token (30 min) + refresh token (7 jours) avec rotation.
- **Alternatives considérées** :
  1. Sessions serveur avec cookies - Rejeté car scaling horizontal plus complexe
  2. JWT dans les cookies HttpOnly - Considéré mais Bearer token plus standard pour les API REST
- **Conséquences** : Stateless, fonctionne avec les futurs clients mobiles. Nécessite un intercepteur axios pour le refresh automatique.
- **Statut** : ACCEPTEE

### DEC-004 : PaiementPro comme prestataire de paiement
- **Date** : [A REMPLIR]
- **Contexte** : Choix du prestataire de paiement pour le marché ivoirien.
- **Décision** : PaiementPro en principal, CinetPay en alternative.
- **Alternatives considérées** :
  1. Stripe - Non disponible en Côte d'Ivoire
  2. PayPal - Pas de mobile money, pas adapté au marché local
  3. CinetPay seul - Possible mais PaiementPro a un meilleur support local
- **Conséquences** : Support Orange Money, MTN MoMo, Wave, Moov Flooz, carte bancaire. Interface PaymentProvider pour faciliter le changement futur.
- **Statut** : ACCEPTEE

### DEC-005 : Zustand pour la gestion d'état frontend
- **Date** : [A REMPLIR]
- **Contexte** : Gestion du panier, de l'état d'authentification et de la recherche côté client.
- **Décision** : Zustand avec middleware persist pour le panier.
- **Alternatives considérées** :
  1. Redux Toolkit - Rejeté car trop de boilerplate pour ce projet
  2. Context API React - Rejeté car re-renders excessifs et pas de persist
  3. Jotai/Recoil - Considéré mais Zustand est plus simple et suffisant
- **Conséquences** : Store léger (< 1 KB), API simple, persist automatique du panier dans localStorage.
- **Statut** : ACCEPTEE

---

## Décisions de design

### DEC-006 : [Titre de la décision design]
- **Date** : [A REMPLIR]
- **Contexte** : [A REMPLIR]
- **Décision** : [A REMPLIR]
- **Alternatives considérées** : [A REMPLIR]
- **Conséquences** : [A REMPLIR]
- **Statut** : [A REMPLIR]

---

## Décisions fonctionnelles

### DEC-007 : [Titre de la décision fonctionnelle]
- **Date** : [A REMPLIR]
- **Contexte** : [A REMPLIR]
- **Décision** : [A REMPLIR]
- **Alternatives considérées** : [A REMPLIR]
- **Conséquences** : [A REMPLIR]
- **Statut** : [A REMPLIR]

---

## Décisions d'infrastructure

### DEC-008 : [Titre de la décision infra]
- **Date** : [A REMPLIR]
- **Contexte** : [A REMPLIR]
- **Décision** : [A REMPLIR]
- **Alternatives considérées** : [A REMPLIR]
- **Conséquences** : [A REMPLIR]
- **Statut** : [A REMPLIR]

---

## Décisions révisées

_Les décisions révisées restent documentées avec leur historique. La nouvelle décision fait référence à l'ancienne._

---

## Index rapide

| ID | Décision | Date | Statut |
|---|---|---|---|
| DEC-001 | Stack NestJS + Next.js + PostgreSQL | [A REMPLIR] | ACCEPTEE |
| DEC-002 | Monorepo Turborepo | [A REMPLIR] | ACCEPTEE |
| DEC-003 | JWT pour l'authentification | [A REMPLIR] | ACCEPTEE |
| DEC-004 | PaiementPro | [A REMPLIR] | ACCEPTEE |
| DEC-005 | Zustand pour l'état frontend | [A REMPLIR] | ACCEPTEE |
| DEC-006 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| DEC-007 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
| DEC-008 | [A REMPLIR] | [A REMPLIR] | [A REMPLIR] |
