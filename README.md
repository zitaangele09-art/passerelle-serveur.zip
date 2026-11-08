# Passerelle – serveur de taux en direct

## Lancer en local
1. Installer Node.js 18 ou plus.
2. Dans ce dossier : `npm start`
3. Ouvrir http://localhost:3000

## Ce que fait le serveur
- `GET /api/rates` renvoie les taux pour 1 EUR : parités fixes (FCFA, KMF, CVE, STN, BAM), puis BCE, puis marché (open.er-api.com) pour les autres devises.
- Les taux sont gardés 10 minutes en mémoire. Si les sources tombent, les derniers taux connus sont servis avec `"stale": true`.
- Il sert aussi l'application (`public/index.html`), qui lit `/api/rates` toutes les minutes.

## Mettre en ligne
Render, Railway, Fly.io ou n'importe quel hébergeur Node : commande de démarrage `npm start`, aucune variable à définir (le port est lu dans `PORT`).

## À faire avant un usage réel
- Les taux BCE sont indicatifs : un taux de transfert réel vient de votre partenaire de paiement (étape 2).
- Vérifier les conditions d'utilisation de chaque source de taux pour un usage commercial.

## Installer l'application sur le téléphone
L'application est une PWA : une fois le serveur en ligne **en HTTPS**, elle s'installe comme une application.
- **Android (Chrome)** : menu ⋮ puis « Installer l'application » (ou « Ajouter à l'écran d'accueil »).
- **iPhone (Safari)** : bouton Partager puis « Sur l'écran d'accueil ».
En local (`http://localhost:3000`) l'installation fonctionne sur l'ordinateur, mais pas depuis un téléphone : il faut le HTTPS, que Render, Railway ou Fly.io fournissent automatiquement.

## Envois et prestataires (étape 2)
Le serveur expose :
- `GET /api/quote?from=XOF&to=EUR&amount=100000` : frais, taux et montant reçu (le serveur fait foi).
- `POST /api/transfers` : crée un envoi (`from`, `to`, `amount`, `country`, `method`, `name`, `account`, `key`). Une même `key` ne crée jamais deux envois.
- `GET /api/transfers/:ref` : statut (`processing`, `paid`, `failed`).

Par défaut c'est le prestataire **sandbox** (aucun argent ne bouge ; un compte finissant par 0000 est refusé pour tester l'échec).
Pour un vrai partenaire : copiez `providers/template.js`, branchez l'API du partenaire, déclarez-le dans `providers/index.js`, puis lancez avec `PROVIDER=monpartenaire npm start`.

Avant tout usage réel, il manque notamment : une base de données (les envois sont en mémoire), la connexion des utilisateurs, la vérification d'identité (KYC), le contrôle anti-blanchiment, des plafonds par client, et l'agrément ou le partenaire agréé.

### Dépôts et retraits
- `POST /api/deposits` et `POST /api/withdrawals` : `currency`, `amount`, `method`, `account` (si le mode l'exige), `key`.
- Le prestataire doit implémenter `cashIn(t)` et `cashOut(t)` en plus de `send(t)` et `status(t)` (voir `providers/template.js`).
- Pour les cartes bancaires, n'enregistrez jamais de numéro de carte vous-même : utilisez la page de paiement hébergée du partenaire.
- Le solde du client n'est pas géré par ce serveur de démonstration. Un vrai produit doit tenir un registre des soldes (base de données), avec contrôle avant chaque retrait.
