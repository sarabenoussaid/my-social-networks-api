# Guide de lancement et de rendu

## 1. Installation (Node.js 22, MongoDB, OpenSSL)

Installer Node.js, MongoDB Community (démarrer le service MongoDB) et OpenSSL. Dans un terminal à la racine :

```bash
npm install
cp .env.example .env
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

Sous PowerShell, remplacer `cp` par `Copy-Item` si nécessaire. Copier le secret généré dans JWT_SECRET du fichier .env. Ne jamais publier ce fichier. Pour MongoDB Atlas, remplacer MONGODB_URI par son URI privée.

```bash
mkdir -p certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout certs/localhost-key.pem -out certs/localhost-cert.pem -subj '/CN=localhost' -addext 'subjectAltName=DNS:localhost,IP:127.0.0.1'
npm run dev
```

Sous Windows, utiliser Git Bash pour ces commandes OpenSSL. Le serveur écoute sur https://localhost:3000.

## 2. Postman

1. Importer `postman/My Social Networks.postman_collection.json`.
2. Pour le certificat autosigné local uniquement, désactiver SSL certificate verification dans Postman. Réactiver ensuite.
3. Dans Collection Runner, sélectionner uniquement le dossier **00 — Scénario de validation**, une itération. Les variables de collection enregistrent automatiquement les identifiants et le JWT. Le scénario crée des données fictives et vérifie quatre conflits 409. Les autres dossiers permettent de tester chaque route individuellement, après création des ressources nécessaires. Les routes de suppression et de départ ne sont pas toutes exécutées par le scénario.
4. Vérifier les résultats et corriger toute erreur avant rendu. Les exemples et assertions sont fournis, pas des résultats de tests réels.
5. Dans le menu de la collection, ouvrir View documentation, puis Publish (ou Publish documentation selon la version). Ne publier aucun token ni secret : vider les valeurs de token et identifiants de la collection, et utiliser des données fictives. Copier l'URL publique obtenue dans README.md.

## 3. GitHub ou GitLab

Créer un dépôt vide dans son compte, puis :

```bash
git init
git add .
git status
git commit -m "My Social Networks API - TP"
git branch -M main
git remote add origin URL_DU_DEPOT
git push -u origin main
```

Remplacer URL_DU_DEPOT par l'adresse réelle. Vérifier que .env et la clé TLS ne figurent pas dans git status. Remettre le lien du dépôt au professeur avec l'URL de documentation Postman publiée dans README.md.

## 4. Vérifications et limites

- Syntaxe JavaScript contrôlée avec node --check.
- Installation npm bloquée dans l'environnement de préparation par une erreur DNS ; MongoDB et les requêtes HTTP n'ont pas pu être exécutés ici.
- Le partage externe fournit des liens ; une page publique déployée doit être configurée pour que des personnes externes puissent consulter l'événement.
- Le sujet annonce trois étapes de création mais ne décrit que la première ; aucune interface graphique ni étapes supplémentaires inventées.
- Les fils sont créés/supprimés avec leurs groupes/événements ; les messages disposent des opérations de lecture, création, modification et suppression. Les billets achetés ne sont pas modifiables par l'acheteur ; les types de billets sont modifiables. La recherche utilise les listes et filtres disponibles dans le code d'origine.
- Le code source d'origine est conservé et complété ; la publication Postman et GitHub nécessite votre compte. Aucun lien fictif n'est fourni.
- Si les identifiants du fichier .env original étaient réels, les renouveler avant toute publication.
