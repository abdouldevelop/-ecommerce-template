# Questionnaire Client - Projet E-Commerce

## Instructions pour l'agent IA

Ce questionnaire doit être posé au client **avant** de commencer le développement. Chaque question est accompagnée d'un contexte expliquant pourquoi elle est importante et d'exemples de réponses possibles. Remplir ensuite le `cahier-des-charges.md` avec les réponses obtenues.

**Méthode recommandée** : Poser les questions section par section, pas toutes d'un coup. Laisser le client répondre à son rythme.

---

## Section 1 : Identité de l'entreprise (8 questions)

### Q1. Quel est le nom de votre entreprise/marque ?
- **Contexte** : Nécessaire pour le branding, le domaine, les mentions légales
- **Exemple** : "Atypick Groupe", "Ma Boutique CI"

### Q2. Quel est votre secteur d'activité principal ?
- **Contexte** : Influence le design, les fonctionnalités spécifiques et le vocabulaire du site
- **Exemple** : "Mode et vêtements", "Cosmétiques", "Alimentation", "Electronique", "Multi-catégories"

### Q3. Avez-vous déjà un logo ? Si oui, dans quels formats ?
- **Contexte** : Nécessaire en PNG transparent (haute résolution) minimum. SVG idéal pour le web.
- **Exemple** : "Oui, en PNG et AI", "Non, il faut le créer"

### Q4. Avez-vous une charte graphique existante (couleurs, polices) ?
- **Contexte** : Si oui, on la respecte. Sinon, on en propose une.
- **Exemple** : "Oui : bleu #1a365d, or #d4a843, police Montserrat", "Non, je vous fais confiance"

### Q5. Avez-vous un slogan ou une phrase d'accroche ?
- **Contexte** : Utilisé dans le header, les bannières, le SEO
- **Exemple** : "L'élégance accessible", "Votre style, notre passion"

### Q6. Avez-vous des comptes réseaux sociaux à lier au site ?
- **Contexte** : Liens dans le footer, partage produits, login social potentiel
- **Exemple** : "Facebook, Instagram, TikTok", "Pas encore"

### Q7. Avez-vous un site web existant ? Si oui, quelle est l'URL ?
- **Contexte** : Permet de voir le positionnement actuel, récupérer du contenu, planifier une migration
- **Exemple** : "Oui : www.maboutique.ci", "Non, c'est notre premier site"

### Q8. Avez-vous un numéro d'enregistrement commercial (RCCM, NIF) ?
- **Contexte** : Requis pour les mentions légales et les conditions générales de vente
- **Exemple** : "RCCM CI-ABJ-2024-B-12345"

---

## Section 2 : Catalogue produits (9 questions)

### Q9. Combien de produits avez-vous au lancement ?
- **Contexte** : Impact sur la pagination, le filtrage, la performance et le temps de saisie initial
- **Exemple** : "15 produits", "200+ produits", "Environ 50"

### Q10. Vos produits ont-ils des variantes (taille, couleur, etc.) ?
- **Contexte** : Si oui, il faut un système de variantes avec gestion de stock par variante
- **Exemple** : "Oui : taille (S/M/L/XL) et couleur", "Non, chaque produit est unique"

### Q11. Combien de catégories de produits prévoyez-vous ?
- **Contexte** : Influence la navigation, les filtres et la structure du menu
- **Exemple** : "5 catégories : Homme, Femme, Enfant, Accessoires, Chaussures"

### Q12. Avez-vous besoin de sous-catégories ?
- **Contexte** : Complexifie la navigation mais nécessaire pour les gros catalogues
- **Exemple** : "Oui : Homme > T-shirts, Pantalons, Vestes", "Non, des catégories simples suffisent"

### Q13. Les photos des produits sont-elles déjà prises ?
- **Contexte** : C'est souvent le goulot d'étranglement. Sans photos, pas de lancement.
- **Exemple** : "Oui, toutes prêtes en haute résolution", "Non, il faut les prendre"

### Q14. Combien de photos par produit ?
- **Contexte** : Impact sur le stockage, le temps de chargement et le design de la galerie
- **Exemple** : "1 photo principale", "3-5 photos (face, dos, détail, porté)"

### Q15. Avez-vous des descriptions produits rédigées ?
- **Contexte** : Les descriptions sont essentielles pour le SEO et la conversion
- **Exemple** : "Oui, en français", "Non, il faudra les rédiger"

### Q16. Gérez-vous les stocks ?
- **Contexte** : Si oui, il faut un système de suivi de stock avec alertes. Si non, on peut simplifier.
- **Exemple** : "Oui, stock limité", "Non, c'est du sur-commande", "Oui avec alerte quand stock < 5"

### Q17. Voulez-vous mettre en avant certains produits (vedettes/featured) ?
- **Contexte** : Affichage prioritaire en page d'accueil, badges spéciaux
- **Exemple** : "Oui, les 4 meilleures ventes en haut de page", "Non"

---

## Section 3 : Paiement (8 questions)

### Q18. Quels moyens de paiement souhaitez-vous accepter ?
- **Contexte** : En Afrique de l'Ouest, le mobile money est souvent plus utilisé que la carte bancaire
- **Exemple** : "Carte bancaire + Orange Money + Wave", "Tous les moyens disponibles"

### Q19. Avez-vous déjà un compte marchand chez un prestataire de paiement ?
- **Contexte** : PaiementPro et CinetPay sont les plus utilisés en Côte d'Ivoire
- **Exemple** : "Oui, chez PaiementPro", "Non, il faut en créer un"

### Q20. Si non, quel prestataire préférez-vous ?
- **Contexte** : PaiementPro (recommandé, bon support), CinetPay (alternative fiable)
- **Exemple** : "PaiementPro", "Je ne sais pas, conseillez-moi"

### Q21. Dans quelle devise vendez-vous ?
- **Contexte** : Impact sur l'affichage des prix, les calculs et le prestataire de paiement
- **Exemple** : "FCFA (XOF)", "Euros", "Multi-devises"

### Q22. La TVA est-elle applicable sur vos produits ?
- **Contexte** : Si oui, il faut la calculer et l'afficher (18% en Côte d'Ivoire)
- **Exemple** : "Oui, 18% TVA incluse dans le prix", "Non, pas de TVA"

### Q23. Proposez-vous des codes promo ou des réductions ?
- **Contexte** : Si oui, il faut un système de coupons avec validation et calcul de réduction
- **Exemple** : "Oui, des codes promo en pourcentage", "Pas pour le lancement"

### Q24. Acceptez-vous le paiement à la livraison ?
- **Contexte** : Très demandé en Afrique, mais complique la gestion (commandes non payées)
- **Exemple** : "Oui", "Non, paiement en ligne uniquement"

### Q25. Y a-t-il un montant minimum de commande ?
- **Contexte** : Pour rentabiliser les frais de livraison
- **Exemple** : "Oui, 5000 FCFA minimum", "Non"

---

## Section 4 : Livraison (7 questions)

### Q26. Quelle zone géographique couvrez-vous pour la livraison ?
- **Contexte** : Impact sur les frais de livraison, les délais et les partenaires logistiques
- **Exemple** : "Abidjan uniquement", "Toute la Côte d'Ivoire", "Afrique de l'Ouest"

### Q27. Comment calculez-vous les frais de livraison ?
- **Contexte** : Forfait fixe (plus simple), par zone, par poids, ou gratuit au-delà d'un seuil
- **Exemple** : "2000 FCFA fixe pour Abidjan, 5000 FCFA hors Abidjan", "Gratuit à partir de 25000 FCFA"

### Q28. Livrez-vous vous-même ou utilisez-vous un service de livraison ?
- **Contexte** : Si service externe, possibilité d'intégration API pour le suivi
- **Exemple** : "Livraison interne avec nos propres livreurs", "Via Jumia Logistics", "Les deux"

### Q29. Quels sont vos délais de livraison ?
- **Contexte** : Affiché sur la page produit et la confirmation de commande
- **Exemple** : "24h Abidjan, 48-72h hors Abidjan", "3-5 jours ouvrés"

### Q30. Proposez-vous le retrait en boutique ?
- **Contexte** : Option populaire qui évite les frais de livraison
- **Exemple** : "Oui, à notre boutique de Cocody", "Non"

### Q31. Avez-vous une politique de retour ?
- **Contexte** : Obligatoire légalement dans certains pays, influence la confiance client
- **Exemple** : "Retour sous 7 jours si article non porté", "Pas de retour"

### Q32. Souhaitez-vous un suivi de livraison en temps réel ?
- **Contexte** : Complexe à implémenter, nécessite un partenaire logistique avec API
- **Exemple** : "Oui, indispensable", "Non, un simple statut (expédié/livré) suffit"

---

## Section 5 : Design et UX (7 questions)

### Q33. Avez-vous des sites e-commerce que vous admirez (références visuelles) ?
- **Contexte** : Essentiel pour comprendre les attentes esthétiques du client
- **Exemple** : "J'aime le design de Zara et de Sephora", "Je veux quelque chose de simple comme Apple"

### Q34. Préférez-vous un design minimaliste ou riche en visuels ?
- **Contexte** : Influence l'ensemble de la direction artistique
- **Exemple** : "Minimaliste, beaucoup de blanc", "Riche, avec des bannières et des images partout"

### Q35. Quelles couleurs principales souhaitez-vous ?
- **Contexte** : Si pas de charte graphique, proposer 2-3 couleurs maximum
- **Exemple** : "Noir et or (luxe)", "Bleu et blanc (confiance)", "Je veux quelque chose de moderne"

### Q36. Souhaitez-vous un slider/carrousel en page d'accueil ?
- **Contexte** : Très demandé par les clients, permet de mettre en avant promotions et nouveautés
- **Exemple** : "Oui, avec 3-4 slides", "Non, une image fixe suffit"

### Q37. Quelle importance accordez-vous à la version mobile ?
- **Contexte** : En Afrique, 80%+ du trafic est mobile. Le mobile-first est obligatoire.
- **Exemple** : "Priorité absolue", "Important mais le desktop aussi"

### Q38. Voulez-vous un mode sombre (dark mode) ?
- **Contexte** : Populaire mais double le travail CSS. Souvent limité au panel admin.
- **Exemple** : "Oui pour tout le site", "Seulement pour le panel admin", "Non"

### Q39. Avez-vous besoin de pages statiques (A propos, CGV, FAQ) ?
- **Contexte** : CGV et mentions légales sont obligatoires. FAQ réduit les demandes support.
- **Exemple** : "Oui : A propos, CGV, FAQ, Contact", "Le minimum légal"

---

## Section 6 : Fonctionnalités spécifiques (8 questions)

### Q40. Voulez-vous un système de recherche avancée ?
- **Contexte** : Autocomplete, recherche par catégorie, suggestions. Essentiel si > 50 produits.
- **Exemple** : "Oui, avec suggestions en temps réel", "Une recherche simple suffit"

### Q41. Souhaitez-vous un système de liste de souhaits (wishlist) ?
- **Contexte** : Fonctionnalité populaire mais P2 (pas critique pour le MVP)
- **Exemple** : "Oui, pour la V2", "Non"

### Q42. Voulez-vous des avis/commentaires sur les produits ?
- **Contexte** : Améliore la confiance mais nécessite une modération
- **Exemple** : "Oui, avec modération admin", "Non, pas pour le lancement"

### Q43. Avez-vous besoin d'un système de newsletter ?
- **Contexte** : Capture d'emails pour le marketing. Simple à ajouter.
- **Exemple** : "Oui, avec Mailchimp", "Juste la capture d'email"

### Q44. Souhaitez-vous des notifications (email, SMS, push) ?
- **Contexte** : Confirmation de commande, changement de statut, promotions
- **Exemple** : "Email pour les confirmations de commande", "Email + SMS"

### Q45. Voulez-vous un chat en direct ou un bouton WhatsApp ?
- **Contexte** : WhatsApp est le canal préféré en Afrique, très simple à intégrer
- **Exemple** : "Oui, bouton WhatsApp flottant", "Non"

### Q46. Avez-vous besoin d'un blog intégré ?
- **Contexte** : Bon pour le SEO mais complexifie le projet. Peut être ajouté en V2.
- **Exemple** : "Oui, pour partager des conseils mode", "Non, pas pour le moment"

### Q47. Souhaitez-vous des rapports de ventes/statistiques ?
- **Contexte** : Dashboard admin avec chiffre d'affaires, nombre de commandes, meilleurs produits
- **Exemple** : "Oui, un dashboard complet", "Les basiques suffisent"

---

## Section 7 : Hébergement et technique (5 questions)

### Q48. Avez-vous déjà un nom de domaine ?
- **Contexte** : Si non, le réserver immédiatement (peut prendre 24-48h pour la propagation DNS)
- **Exemple** : "Oui : maboutique.ci", "Non, il faut l'acheter"

### Q49. Avez-vous un hébergement/serveur ?
- **Contexte** : VPS recommandé (minimum 2 Go RAM, 2 vCPU). Pas de mutualisé pour du Node.js.
- **Exemple** : "Oui, un VPS chez OVH", "Non, il faut en prendre un"

### Q50. Avez-vous besoin d'un certificat SSL ?
- **Contexte** : Obligatoire pour le e-commerce (https). Let's Encrypt = gratuit.
- **Exemple** : "Oui", "C'est quoi ?" (réponse : c'est obligatoire, on s'en occupe)

### Q51. Qui va gérer le site après la livraison ?
- **Contexte** : Impact sur la documentation, la formation et la complexité du panel admin
- **Exemple** : "Moi-même", "Mon assistante", "On aura besoin de formation"

### Q52. Avez-vous besoin d'une maintenance après livraison ?
- **Contexte** : Mises à jour de sécurité, backups, support technique
- **Exemple** : "Oui, un contrat mensuel", "Non, juste le développement"

---

## Section 8 : Budget et planning (5 questions)

### Q53. Quel est votre budget pour ce projet ?
- **Contexte** : Permet de calibrer les fonctionnalités et de prioriser le MVP
- **Exemple** : "500 000 FCFA", "1 000 000 FCFA", "Flexible selon les fonctionnalités"

### Q54. Quelle est votre date de lancement souhaitée ?
- **Contexte** : Permet de planifier les sprints et de définir le MVP vs les phases suivantes
- **Exemple** : "Dans 3 semaines", "Pour la rentrée", "Pas de deadline stricte"

### Q55. Préférez-vous un lancement en une fois ou par phases ?
- **Contexte** : Le lancement par phases réduit les risques et permet un retour rapide
- **Exemple** : "Par phases : MVP d'abord, puis les extras", "Tout d'un coup"

### Q56. Qui sont les décideurs pour la validation du projet ?
- **Contexte** : Evite les allers-retours avec des personnes non identifiées
- **Exemple** : "Moi seul", "Moi et mon associé", "Le directeur marketing doit valider"

### Q57. Avez-vous d'autres projets ou besoins connexes ?
- **Contexte** : Application mobile, intégration ERP, site vitrine séparé
- **Exemple** : "Une application mobile plus tard", "Non, juste le site"

---

## Résumé post-questionnaire

Après avoir obtenu toutes les réponses, rédiger un résumé de 10 lignes maximum avec :

1. **Nom du projet** et secteur
2. **Nombre de produits** et catégories
3. **Moyens de paiement** choisis
4. **Zone de livraison** et frais
5. **Fonctionnalités MVP** (ce qui est inclus dans la V1)
6. **Fonctionnalités V2** (ce qui est reporté)
7. **Budget** et **deadline**
8. **Points d'attention** (risques identifiés)

Ce résumé sera la base du `cahier-des-charges.md`.
