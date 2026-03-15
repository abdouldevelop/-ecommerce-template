# Checklist de Mise en Production

## Instructions pour l'agent IA

Cette checklist contient 60+ vérifications à effectuer avant de mettre le site en production. Passer chaque point un par un. Ne pas mettre en production si des items CRITIQUE ou BLOQUANT ne sont pas cochés.

**Niveaux** :
- BLOQUANT : Le site ne doit pas être mis en ligne sans ce point
- IMPORTANT : Doit être fait dans les 24h après le lancement
- SOUHAITABLE : A planifier dans la première semaine

---

## 1. Code et fonctionnalités (15 items)

### BLOQUANT

- [ ] Tous les endpoints API fonctionnent correctement
- [ ] L'inscription et la connexion fonctionnent
- [ ] Le panier fonctionne (ajout, suppression, modification quantité)
- [ ] Le processus de commande fonctionne de bout en bout
- [ ] Le paiement fonctionne en mode production (pas sandbox)
- [ ] Le callback de paiement est reçu et traité correctement
- [ ] La page de résultat de paiement affiche le bon statut
- [ ] Les commandes apparaissent dans l'espace client
- [ ] Les commandes apparaissent dans le panel admin
- [ ] L'admin peut changer le statut des commandes
- [ ] Les images produits s'affichent correctement

### IMPORTANT

- [ ] La recherche de produits fonctionne
- [ ] Le filtrage par catégorie fonctionne
- [ ] La pagination fonctionne
- [ ] La page 404 s'affiche pour les URLs inexistantes

---

## 2. Base de données (8 items)

### BLOQUANT

- [ ] Migration de production exécutée avec succès
- [ ] Seed initial exécuté (admin + catégories + paramètres)
- [ ] Compte admin de production créé avec un mot de passe fort
- [ ] Les produits du client sont importés en base
- [ ] Les images des produits sont uploadées et liées

### IMPORTANT

- [ ] Backup automatique configuré (cron quotidien)
- [ ] Test de restauration effectué au moins une fois
- [ ] Connexion à la base restreinte à localhost uniquement

---

## 3. Sécurité (15 items)

### BLOQUANT

- [ ] HTTPS activé (certificat SSL valide)
- [ ] Redirection HTTP → HTTPS en place
- [ ] JWT_SECRET de production fort (32+ caractères aléatoires)
- [ ] JWT_REFRESH_SECRET de production fort et différent du JWT_SECRET
- [ ] Variables d'environnement en fichier .env (pas dans le code)
- [ ] .env en permissions 600 (lecture propriétaire uniquement)
- [ ] Pare-feu configuré (seuls ports 22, 80, 443 ouverts)
- [ ] SSH : authentification par clé uniquement, root login désactivé
- [ ] CORS configuré avec le bon domaine (pas `*`)
- [ ] Mot de passe admin de production changé (pas celui du seed)

### IMPORTANT

- [ ] Fail2ban installé et configuré
- [ ] Headers de sécurité HTTP en place (vérifier sur securityheaders.com)
- [ ] `npm audit` sans vulnérabilités critiques ou élevées
- [ ] Rate limiting en place sur /api/auth/login
- [ ] Les clés API de paiement sont celles de production (pas sandbox)

---

## 4. Performance (8 items)

### IMPORTANT

- [ ] `NODE_ENV=production` configuré
- [ ] Build de production effectué (`npm run build`)
- [ ] PM2 configuré en mode cluster (2+ instances pour l'API)
- [ ] Compression gzip activée dans Nginx
- [ ] Cache des assets statiques configuré (images, CSS, JS : 30 jours)
- [ ] Images des produits optimisées (poids raisonnable < 500 Ko chacune)
- [ ] Score Google PageSpeed Mobile > 60
- [ ] Score Google PageSpeed Desktop > 80

---

## 5. SEO (10 items)

### IMPORTANT

- [ ] `<title>` unique et pertinent sur chaque page
- [ ] `<meta description>` sur chaque page
- [ ] Open Graph tags (og:title, og:description, og:image) sur les pages produits
- [ ] Balises H1 uniques sur chaque page
- [ ] URLs propres et lisibles (slugs pour les produits et catégories)
- [ ] Favicon configuré (ICO + PNG + Apple Touch Icon)
- [ ] `robots.txt` en place (autoriser le crawl)
- [ ] `sitemap.xml` généré (optionnel mais recommandé)

### SOUHAITABLE

- [ ] Structured data (JSON-LD) pour les produits (schema.org/Product)
- [ ] Balise `lang="fr"` sur la balise `<html>`

---

## 6. Contenu (10 items)

### BLOQUANT

- [ ] Tous les produits sont créés avec nom, description, prix, images
- [ ] Les catégories sont créées et les produits assignés
- [ ] Les prix sont corrects et en bonne devise
- [ ] Les images sont en bonne qualité et bien cadrées
- [ ] Les produits vedettes sont sélectionnés

### IMPORTANT

- [ ] Page "A propos" rédigée avec le contenu du client
- [ ] CGV (Conditions Générales de Vente) en place
- [ ] Politique de confidentialité en place
- [ ] Informations de contact correctes (téléphone, email, adresse)
- [ ] Slider de la page d'accueil configuré avec les bonnes images et textes

---

## 7. Infrastructure (10 items)

### BLOQUANT

- [ ] DNS configuré : domaine pointe vers l'IP du serveur
- [ ] Nginx configuré comme reverse proxy
- [ ] PM2 démarre automatiquement au boot du serveur (`pm2 startup`)
- [ ] Health check accessible : `https://domaine.com/api/health`
- [ ] Le site est accessible via HTTPS sur le nom de domaine

### IMPORTANT

- [ ] Script de déploiement testé et fonctionnel
- [ ] Logs PM2 configurés avec rotation
- [ ] Monitoring en place (vérification toutes les 5 minutes)
- [ ] Renouvellement automatique du certificat SSL (certbot renew)
- [ ] Espace disque vérifié (au moins 5 Go libres)

---

## 8. Tests finaux (10 items)

### BLOQUANT

- [ ] Test complet du parcours client : navigation → ajout panier → checkout → paiement → résultat
- [ ] Test sur mobile (iPhone Safari et Android Chrome)
- [ ] Test sur desktop (Chrome et Firefox)
- [ ] Test du paiement en production avec un vrai montant (puis remboursement)
- [ ] Test de la réception du callback de paiement

### IMPORTANT

- [ ] Test du refresh token (attendre 30 minutes et vérifier que la session persiste)
- [ ] Test du panel admin : création/modification/suppression de produit
- [ ] Test du changement de statut de commande
- [ ] Test de l'inscription d'un nouveau client
- [ ] Test de la recherche et du filtrage

---

## 9. Communication et formation (5 items)

### IMPORTANT

- [ ] Documentation client remise (voir `documentation-client.md`)
- [ ] Identifiants admin communiqués au client de manière sécurisée
- [ ] Formation à l'utilisation du panel admin effectuée (ou tutoriel vidéo)
- [ ] Contact support communiqué au client
- [ ] Présentation du site au client pour validation finale

---

## Résumé

| Section | BLOQUANT | IMPORTANT | SOUHAITABLE | Total |
|---|---|---|---|---|
| Code et fonctionnalités | 11 | 4 | 0 | 15 |
| Base de données | 5 | 3 | 0 | 8 |
| Sécurité | 10 | 5 | 0 | 15 |
| Performance | 0 | 8 | 0 | 8 |
| SEO | 0 | 8 | 2 | 10 |
| Contenu | 5 | 5 | 0 | 10 |
| Infrastructure | 5 | 5 | 0 | 10 |
| Tests finaux | 5 | 5 | 0 | 10 |
| Communication | 0 | 5 | 0 | 5 |
| **Total** | **41** | **48** | **2** | **91** |

---

## Go / No-Go

| Critère | Statut |
|---|---|
| Items BLOQUANT cochés | [ ] Tous cochés (41/41) |
| Items IMPORTANT cochés | [ ] Au moins 80% (38/48+) |
| Test paiement réel effectué | [ ] Oui |
| Client a validé le site | [ ] Oui |

**Décision** : [ ] GO / [ ] NO-GO

**Date de mise en production** : [A REMPLIR]
**Responsable** : [A REMPLIR]
**URL de production** : [A REMPLIR]
