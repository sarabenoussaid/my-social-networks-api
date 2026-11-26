# My Social Networks API

## Présentation du projet

Ce projet a été réalisé dans le cadre d'un TP de développement d'API REST. L'objectif est de développer un service permettant de gérer les groupes et les événements d'un réseau social, ainsi que les interactions entre les utilisateurs.

## Fonctionnalités

- Création et gestion des comptes utilisateurs
- Authentification et gestion des droits d'accès
- Création et gestion des groupes et des événements
- Gestion des participants et des organisateurs
- Fils de discussion et messages
- Albums photos et commentaires
- Création de sondages et gestion des réponses
- Gestion de la billetterie
- Gestion des listes de courses
- Organisation du covoiturage (bonus)

## Technologies utilisées

| Technologie | Utilisation |
|---|---|
| Node.js | Développement du serveur |
| Express.js | Création des routes de l'API |
| MongoDB | Stockage des données |
| Mongoose | Gestion des modèles de données |
| JWT | Authentification |
| bcrypt | Protection des mots de passe |
| Postman | Tests et documentation |
| GitHub | Gestion du code source |

## Installation et lancement

Le projet a été développé et testé sous Windows.

Installation des dépendances :

```bash
npm install
```

Création du fichier de configuration sous PowerShell :

```powershell
Copy-Item .env.example .env
```

Configuration de MongoDB dans le fichier `.env` :

```env
MONGODB_URI=mongodb://127.0.0.1:27017/my_social_networks
JWT_SECRET=REMPLACER_PAR_UN_SECRET_PRIVE
```

Le secret JWT doit être généré aléatoirement.

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

MongoDB doit être démarré avant le lancement du serveur.

Génération des certificats HTTPS locaux sous PowerShell :

```powershell
New-Item -ItemType Directory -Force certs
node -e "const fs=require('fs');const p=require('selfsigned').generate([{name:'commonName',value:'localhost'}],{days:365,keySize:2048,algorithm:'sha256'});fs.writeFileSync('certs/localhost-key.pem',p.private);fs.writeFileSync('certs/localhost-cert.pem',p.cert)"
```

Lancement du serveur :

```bash
npm run dev
```

L'API est accessible à l'adresse suivante :

https://localhost:3000

## Tests

Les tests ont été réalisés avec Postman afin de vérifier le fonctionnement des principales fonctionnalités de l'API et la gestion des erreurs.

| Résultat | Nombre |
|---|---:|
| Requêtes exécutées | 25 |
| Assertions réussies | 46 |
| Assertions échouées | 0 |
| Erreurs d'exécution | 0 |

La collection Postman est disponible dans le dossier `postman` du projet.

Pour les tests locaux avec le certificat autosigné, la vérification SSL peut être désactivée temporairement dans Postman.

## Documentation de l'API

La documentation présente les endpoints, les paramètres et les requêtes associés aux différentes fonctionnalités du projet.

[Consulter la documentation Postman](https://sara-s-team.docs.buildwithfern.com/my-social-networks-api-tp/introduction)
