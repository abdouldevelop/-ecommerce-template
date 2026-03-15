# Checklist de Validation - Avant Développement

## Instructions pour l'agent IA

**Ne pas commencer le développement tant que tous les items P0 ne sont pas cochés.**

Les items P1 peuvent être obtenus en cours de développement mais doivent être prêts avant la mise en production. Les items P2 sont souhaitables mais non bloquants.

---

## 1. Documents et validations (P0)

- [ ] Cahier des charges rempli et validé par le client
- [ ] Budget validé et accord écrit/oral confirmé
- [ ] Planning accepté par le client
- [ ] Décideur(s) identifié(s) et joignable(s)
- [ ] Scope MVP clairement défini (ce qui est inclus / exclu)

## 2. Identité visuelle (P0)

- [ ] Logo reçu en haute résolution (PNG transparent minimum)
- [ ] Logo en format vectoriel (SVG ou AI) si disponible
- [ ] Couleurs principales définies (codes hexadécimaux)
- [ ] Police(s) de caractères choisie(s)
- [ ] Favicon fourni ou à créer à partir du logo
- [ ] Style général validé (minimaliste, luxe, coloré, etc.)

## 3. Contenu produits (P0)

- [ ] Liste complète des catégories validée
- [ ] Au moins 5 produits avec contenu complet pour tester
- [ ] Photos produits disponibles (minimum 1 par produit)
- [ ] Noms des produits finalisés
- [ ] Prix définis pour chaque produit
- [ ] Descriptions produits rédigées (au moins résumés)

## 4. Contenu rédactionnel (P1)

- [ ] Texte "A propos" rédigé
- [ ] Conditions Générales de Vente (CGV) rédigées
- [ ] Politique de confidentialité rédigée
- [ ] Politique de retour rédigée
- [ ] FAQ (si applicable) rédigée
- [ ] Texte de la page contact (adresse, téléphone, email, horaires)
- [ ] Slogan / phrase d'accroche validé(e)

## 5. Paiement (P0)

- [ ] Prestataire de paiement choisi (PaiementPro / CinetPay)
- [ ] Compte marchand créé chez le prestataire
- [ ] Clés API obtenues (mode test/sandbox)
- [ ] Clés API de production obtenues (ou en cours de validation)
- [ ] Moyens de paiement activés (carte, Orange Money, MTN MoMo, Wave, etc.)
- [ ] Devise confirmée (XOF / EUR / autre)
- [ ] TVA : applicable ou non, taux confirmé

## 6. Livraison (P1)

- [ ] Zones de livraison définies
- [ ] Grille tarifaire des frais de livraison validée
- [ ] Délais de livraison par zone définis
- [ ] Politique de retour validée
- [ ] Retrait en boutique : oui/non + adresse si oui
- [ ] Livraison gratuite : seuil défini ou non applicable

## 7. Hébergement et domaine (P0)

- [ ] Nom de domaine acheté et accessible
- [ ] Accès DNS disponible (pour configurer les enregistrements A/CNAME)
- [ ] Serveur/VPS provisionné (minimum 2 Go RAM, 2 vCPU)
- [ ] Accès SSH au serveur (clé SSH ou mot de passe)
- [ ] OS installé (Ubuntu 22.04 ou Debian 12 recommandé)
- [ ] IP du serveur connue

## 8. Accès et credentials (P0)

- [ ] Accès au compte hébergeur (panel de gestion)
- [ ] Accès au registrar domaine (pour DNS)
- [ ] Identifiants API paiement (clé marchande, clé secrète)
- [ ] Accès email du client (pour les notifications transactionnelles)
- [ ] Compte réseaux sociaux (si intégration prévue)
- [ ] Accès WhatsApp Business (si bouton WhatsApp prévu)

## 9. Ressources visuelles (P1)

- [ ] Photos produits en haute résolution (minimum 800x800px)
- [ ] Images pour le slider/carrousel (minimum 1200x500px)
- [ ] Photo(s) pour la page "A propos"
- [ ] Icônes pour les catégories (si design personnalisé)
- [ ] Images de fond ou patterns (si prévu dans le design)

## 10. Validation technique (P0)

- [ ] Stack technique validée (NestJS + Next.js + PostgreSQL)
- [ ] Structure de la base de données validée
- [ ] Endpoints API listés et validés
- [ ] Intégration paiement testée en sandbox
- [ ] Environnement de développement prêt

## 11. Communication (P2)

- [ ] Canal de communication défini (WhatsApp / Email / Slack)
- [ ] Fréquence des points d'avancement définie
- [ ] Processus de validation des livrables défini
- [ ] Personne de contact pour les questions techniques identifiée
- [ ] Personne de contact pour les questions métier identifiée

---

## Résumé de statut

| Section | Statut | Commentaire |
|---|---|---|
| Documents et validations | [ ] Prêt | |
| Identité visuelle | [ ] Prêt | |
| Contenu produits | [ ] Prêt | |
| Contenu rédactionnel | [ ] Prêt | |
| Paiement | [ ] Prêt | |
| Livraison | [ ] Prêt | |
| Hébergement et domaine | [ ] Prêt | |
| Accès et credentials | [ ] Prêt | |
| Ressources visuelles | [ ] Prêt | |
| Validation technique | [ ] Prêt | |
| Communication | [ ] Prêt | |

**Decision : GO / NO-GO**

Date de la décision : [A REMPLIR]
Items bloquants restants : [A REMPLIR]
Date de début de développement : [A REMPLIR]

---

## Notes

- Si le client n'a pas de CGV, utiliser un modèle standard adapté au pays
- Les photos peuvent être ajoutées progressivement, mais au moins 5 produits complets sont nécessaires pour commencer l'intégration
- L'intégration paiement doit être testée en sandbox AVANT la mise en production
- Le nom de domaine doit être acheté au moins 48h avant la mise en production (propagation DNS)
