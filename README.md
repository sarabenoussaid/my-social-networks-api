# My Social Networks API

API REST (Node.js, Express, MongoDB/Mongoose) pour le nouveau service d'Ã©vÃ©nements et de groupes de Facebook.

Elle reprend l'architecture et les protections du TP prÃ©cÃ©dent : classes contrÃ´leurs, modÃ¨les Mongoose, validation des corps de requÃªte (`src/validation.mjs` + rÃ¨gles Mongoose + `validator`), authentification JWT, HTTPS, Helmet, CORS et limitation du nombre de requÃªtes.

## DÃ©marrer l'API

Depuis Ubuntu, Ã  la racine du projet :

```bash
npm install
npm run dev
```

Le serveur Ã©coute en HTTPS sur le port `3000` : `https://localhost:3000`.

Le fichier `.env` (non versionnÃ©) doit contenir :

```dotenv
MONGODB_URI=mongodb://127.0.0.1:27017/my_social_networks
JWT_SECRET=<secret alÃ©atoire privÃ© d'au moins 32 octets>
```

Si les certificats locaux n'existent pas (aprÃ¨s un clonage), gÃ©nÃ¨re-les :

```bash
mkdir -p certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout certs/localhost-key.pem \
  -out certs/localhost-cert.pem \
  -subj "/CN=localhost" \
  -addext "subjectAltName=DNS:localhost,IP:127.0.0.1,IP:::1"
chmod 600 certs/localhost-key.pem
```

Dans Postman, dÃ©sactive **Settings â†’ General â†’ SSL certificate verification** pour ces essais locaux.

## Arborescence

```text
index.mjs
src/
  config.mjs            configuration (port, URI MongoDB lue dans .env)
  server.mjs            Express, middlewares de sÃ©curitÃ©, JWT, HTTPS
  errors.mjs            rÃ©ponses d'erreur communes (400, 403, 404, 409, 500)
  validation.mjs        validation des corps de requÃªte
  permissions.mjs       droits : organisateur, participant, administrateur, membre
  controllers/          une classe par ressource
  models/               un schÃ©ma Mongoose par collection
```

## Collections MongoDB

| Collection | Contenu |
|---|---|
| `users` | Utilisateurs (email unique, mot de passe hachÃ© avec bcrypt) |
| `groups` | Groupes : nom, description, icÃ´ne, couverture, type (`public`, `private`, `secret`), autorisations, administrateurs, membres |
| `events` | Ã‰vÃ©nements : nom, description, dates, lieu, couverture, visibilitÃ© (`public`, `private`), organisateurs, membres, groupe Ã©ventuel, billetterie activÃ©e ou non |
| `threads` | Fils de discussion, liÃ©s Ã  **1 groupe ou 1 Ã©vÃ©nement, jamais les deux** |
| `messages` | Messages d'un fil (`reply_to` permet de rÃ©pondre Ã  un message) |
| `albums` | Albums photo, chacun associÃ© Ã  1 Ã©vÃ©nement |
| `photos` | Photos d'un album, postÃ©es par un participant |
| `comments` | Commentaires des photos, Ã©crits par des participants |
| `polls` | Sondages d'un Ã©vÃ©nement : questions et rÃ©ponses possibles |
| `poll_responses` | RÃ©ponses d'un participant Ã  un sondage (une seule fois par sondage) |
| `ticket_types` | Types de billets : nom, montant, quantitÃ© limitÃ©e, nombre vendus |
| `tickets` | Billets achetÃ©s : type, nom, prÃ©nom, email, adresse complÃ¨te, date d'achat |
| `shopping_items` | Shopping list : nom, quantitÃ©, heure d'arrivÃ©e, utilisateur qui apporte |
| `carpools` | *(Bonus)* Covoiturages : lieu et heure de dÃ©part, prix, places, Ã©cart maximum, conducteur, passagers |

## RÃ¨gles fonctionnelles

- Deux utilisateurs ne peuvent pas avoir le mÃªme email (`409`).
- Le crÃ©ateur d'un groupe en devient administrateur ; le crÃ©ateur d'un Ã©vÃ©nement en devient organisateur. Un groupe a toujours au moins 1 administrateur et un Ã©vÃ©nement au moins 1 organisateur.
- Un **groupe secret** n'est visible que par ses membres ; un **Ã©vÃ©nement privÃ©** n'est visible que par ses participants (organisateurs + membres). Pour les autres utilisateurs, ils rÃ©pondent `404`.
- Seuls les groupes publics et les Ã©vÃ©nements publics peuvent Ãªtre rejoints librement (`/join`). Dans les autres cas, ce sont les administrateurs/organisateurs qui gÃ¨rent les listes `admins`/`members` ou `organizers`/`members`.
- CrÃ©ation d'un Ã©vÃ©nement dans un groupe : autorisÃ©e aux administrateurs, et aux membres si `allow_members_to_create_events` est activÃ©. `invite_group_members: true` invite tous les membres du groupe en un clic.
- Ã€ sa crÃ©ation, chaque groupe et chaque Ã©vÃ©nement reÃ§oit automatiquement son fil de discussion.
- Fil d'un groupe : lecture par les membres ; publication par les administrateurs, et par les membres si `allow_members_to_post` est activÃ©. Fil d'un Ã©vÃ©nement : lecture et publication par les participants.
- Un message peut Ãªtre supprimÃ© par son auteur ou par un administrateur/organisateur.
- Les albums sont gÃ©rÃ©s par les organisateurs ; les photos sont postÃ©es et commentÃ©es par les participants.
- Les sondages sont crÃ©Ã©s par un organisateur ; chaque question possÃ¨de au moins 2 rÃ©ponses possibles ; un participant rÃ©pond une seule fois et choisit exactement 1 rÃ©ponse pour chaque question.
- La billetterie n'existe que pour les Ã©vÃ©nements **publics** avec `ticketing: true`. L'achat d'un billet est public (personne extÃ©rieure, sans compte). Une mÃªme personne (email) ne peut obtenir qu'**1 seul billet** par Ã©vÃ©nement, et un type de billet ne peut pas dÃ©passer sa quantitÃ©.
- Si `shopping_list` est activÃ©e sur l'Ã©vÃ©nement, chaque participant peut indiquer ce qu'il apporte. Chaque chose apportÃ©e est unique par Ã©vÃ©nement (Â« Chips Â» et Â« chips Â» sont considÃ©rÃ©s comme identiques â†’ `409`).
- *(Bonus)* Si `carpooling` est activÃ© sur l'Ã©vÃ©nement, un participant peut proposer un trajet. Les autres participants peuvent rÃ©server une place tant qu'il en reste (`409` si complet).
- La suppression d'un Ã©vÃ©nement supprime ses fils, messages, albums, photos, commentaires, sondages, rÃ©ponses, types de billets, billets, shopping list et covoiturages. La suppression d'un groupe supprime son fil et ses messages ; ses Ã©vÃ©nements sont conservÃ©s.

## Authentification

1. `POST /users` crÃ©e un compte (public).
2. `POST /auth/login` renvoie un `token` JWT valable 1 heure.
3. Les routes protÃ©gÃ©es attendent l'en-tÃªte `Authorization: Bearer <token>` (Postman : **Authorization â†’ Bearer Token**).

Routes publiques : `POST /users`, `POST /auth/login`, `GET /event/:idevent/ticket-types`, `POST /event/:idevent/ticket`. Toutes les autres demandent un token.

## Format des erreurs

```json
{ "code": 400, "message": "Validation failed", "errors": [{ "field": "name", "message": "This field is required" }] }
```

| Code | Signification |
|---|---|
| `400` | Corps invalide, champ inconnu, id invalide, JSON mal formÃ© |
| `401` | Token absent, invalide ou expirÃ© |
| `403` | Utilisateur connectÃ© mais sans le droit nÃ©cessaire |
| `404` | Ressource inexistante ou non visible |
| `409` | Conflit : email dÃ©jÃ  utilisÃ©, billet dÃ©jÃ  obtenu, sondage dÃ©jÃ  rÃ©pondu, billets Ã©puisÃ©s |
| `413` | Corps trop volumineux |
| `429` | Trop de requÃªtes (100 par heure et par IP) |
| `500` | Erreur interne (message gÃ©nÃ©rique + `requestId`) |

Les dates sont au format ISO 8601 (ex. `2026-12-31T20:00:00Z`). Les URL (photos, icÃ´nes, couvertures) doivent Ãªtre en HTTP ou HTTPS.

## Endpoints

### Utilisateurs et authentification

| MÃ©thode | Route | Description |
|---|---|---|
| POST | `/users` | CrÃ©er un compte : `firstname`, `lastname`, `age`, `email`, `password` |
| POST | `/auth/login` | Se connecter : `email`, `password` â†’ `token` |
| GET | `/users` | Lister les utilisateurs |
| GET | `/users/:id` | DÃ©tail d'un utilisateur |
| PUT | `/users/:id` | Modifier `firstname`, `lastname`, `age` |
| DELETE | `/users/:id` | Supprimer un utilisateur |

### Groupes

| MÃ©thode | Route | Droit | Description |
|---|---|---|---|
| GET | `/groups?name=` | connectÃ© | Groupes publics/privÃ©s + groupes secrets dont on est membre |
| GET | `/group/:id` | membre si secret | DÃ©tail avec administrateurs et membres |
| POST | `/group` | connectÃ© | CrÃ©er un groupe |
| PUT | `/group/:id` | administrateur | Modifier un groupe (tous les champs, `name` obligatoire) |
| DELETE | `/group/:id` | administrateur | Supprimer un groupe |
| POST | `/group/:id/join` | connectÃ© | Rejoindre un groupe public |
| POST | `/group/:id/leave` | membre | Quitter le groupe (sauf dernier administrateur) |

Exemple `POST /group` :

```json
{
  "name": "Randonneurs de Paris",
  "description": "Sorties nature le week-end",
  "icon": "https://example.com/icon.png",
  "cover_photo": "https://example.com/cover.jpg",
  "type": "private",
  "allow_members_to_post": true,
  "allow_members_to_create_events": false,
  "admins": [],
  "members": ["<userId>", "<userId>"]
}
```

### Ã‰vÃ©nements

| MÃ©thode | Route | Droit | Description |
|---|---|---|---|
| GET | `/events?name=&group=` | connectÃ© | Ã‰vÃ©nements publics + Ã©vÃ©nements dont on est participant |
| GET | `/event/:id` | participant si privÃ© | DÃ©tail avec organisateurs, membres et groupe |
| POST | `/event` | connectÃ© (droits du groupe si `group`) | CrÃ©er un Ã©vÃ©nement |
| PUT | `/event/:id` | organisateur | Modifier un Ã©vÃ©nement |
| DELETE | `/event/:id` | organisateur | Supprimer un Ã©vÃ©nement et ses donnÃ©es |
| POST | `/event/:id/join` | connectÃ© | Participer Ã  un Ã©vÃ©nement public |
| POST | `/event/:id/leave` | participant | Quitter l'Ã©vÃ©nement (sauf dernier organisateur) |

Exemple `POST /event` :

```json
{
  "name": "SoirÃ©e du Nouvel An",
  "description": "On fÃªte 2027 ensemble",
  "start_date": "2026-12-31T20:00:00Z",
  "end_date": "2027-01-01T04:00:00Z",
  "location": "Paris",
  "cover_photo": "https://example.com/cover.jpg",
  "visibility": "public",
  "organizers": ["<userId>"],
  "members": ["<userId>"],
  "ticketing": true,
  "shopping_list": true,
  "carpooling": true,
  "group": "<groupId>",
  "invite_group_members": true
}
```

`group` et `invite_group_members` ne sont acceptÃ©s qu'Ã  la crÃ©ation. `ticketing: true` n'est acceptÃ© que si `visibility` vaut `public`.

### Fils de discussion

| MÃ©thode | Route | Droit | Description |
|---|---|---|---|
| GET | `/group/:id/thread` | membre du groupe | Fil du groupe |
| GET | `/event/:id/thread` | participant | Fil de l'Ã©vÃ©nement |
| GET | `/thread/:idthread/messages` | lecteur du fil | Messages du fil (ordre chronologique) |
| POST | `/thread/:idthread/message` | voir rÃ¨gles | Poster `{ "content": "...", "reply_to": "<messageId>" }` (`reply_to` facultatif) |
| DELETE | `/thread/:idthread/message/:idmessage` | auteur ou admin/organisateur | Supprimer un message |

### Albums, photos et commentaires

| MÃ©thode | Route | Droit | Description |
|---|---|---|---|
| GET | `/albums?event=&title=` | connectÃ© | Albums des Ã©vÃ©nements visibles |
| GET | `/album/:id` | visibilitÃ© de l'Ã©vÃ©nement | DÃ©tail avec photos |
| POST | `/album` | organisateur | `{ "title": "...", "description": "...", "event": "<eventId>" }` |
| PUT | `/album/:id` | organisateur | `title`, `description` |
| DELETE | `/album/:id` | organisateur | Supprime l'album, ses photos et leurs commentaires |
| GET | `/album/:idalbum/photos` | visibilitÃ© de l'Ã©vÃ©nement | Photos de l'album |
| GET | `/album/:idalbum/photo/:idphoto` | visibilitÃ© de l'Ã©vÃ©nement | DÃ©tail d'une photo |
| POST | `/album/:idalbum/photo` | participant | `{ "title": "...", "url": "https://...", "description": "..." }` |
| PUT | `/album/:idalbum/photo/:idphoto` | auteur | Modifier une photo |
| DELETE | `/album/:idalbum/photo/:idphoto` | auteur ou organisateur | Supprimer une photo |
| GET | `/album/:idalbum/photo/:idphoto/comments` | visibilitÃ© de l'Ã©vÃ©nement | Commentaires |
| POST | `/album/:idalbum/photo/:idphoto/comment` | participant | `{ "content": "..." }` |
| DELETE | `/album/:idalbum/photo/:idphoto/comment/:idcomment` | auteur ou organisateur | Supprimer un commentaire |

### Sondages

| MÃ©thode | Route | Droit | Description |
|---|---|---|---|
| GET | `/event/:idevent/polls` | visibilitÃ© de l'Ã©vÃ©nement | Sondages de l'Ã©vÃ©nement |
| GET | `/poll/:id` | visibilitÃ© de l'Ã©vÃ©nement | DÃ©tail (ids des questions et des rÃ©ponses) |
| POST | `/event/:idevent/poll` | organisateur | CrÃ©er un sondage |
| DELETE | `/poll/:id` | organisateur | Supprimer un sondage et ses rÃ©ponses |
| POST | `/poll/:id/response` | participant | RÃ©pondre (une seule fois) |
| GET | `/poll/:id/results` | visibilitÃ© de l'Ã©vÃ©nement | Nombre de votes par rÃ©ponse |

Exemple `POST /event/:idevent/poll` :

```json
{
  "title": "Organisation",
  "questions": [
    { "label": "Quel menu ?", "answers": ["Pizza", "Sushi", "Raclette"] },
    { "label": "Quelle musique ?", "answers": ["Rock", "Ã‰lectro"] }
  ]
}
```

Exemple `POST /poll/:id/response` (une rÃ©ponse pour chaque question) :

```json
{
  "answers": [
    { "question": "<questionId>", "answer": "<answerId>" },
    { "question": "<questionId>", "answer": "<answerId>" }
  ]
}
```

### Billetterie

| MÃ©thode | Route | Droit | Description |
|---|---|---|---|
| GET | `/event/:idevent/ticket-types` | public | Types de billets avec places restantes (`remaining`) |
| POST | `/event/:idevent/ticket-type` | organisateur | `{ "name": "Standard", "amount": 15, "quantity": 100 }` |
| PUT | `/event/:idevent/ticket-type/:id` | organisateur | Modifier (quantitÃ© â‰¥ billets dÃ©jÃ  vendus) |
| DELETE | `/event/:idevent/ticket-type/:id` | organisateur | Supprimer un type sans billet vendu |
| POST | `/event/:idevent/ticket` | public | Acheter un billet |
| GET | `/event/:idevent/tickets` | organisateur | Liste des billets achetÃ©s |

Exemple `POST /event/:idevent/ticket` (sans token) :

```json
{
  "ticket_type": "<ticketTypeId>",
  "firstname": "Jean",
  "lastname": "Dupont",
  "email": "jean.dupont@example.com",
  "address": {
    "street": "10 rue de la Paix",
    "zip_code": "75002",
    "city": "Paris",
    "country": "France"
  }
}
```

La date d'achat (`purchased_at`) est ajoutÃ©e automatiquement.

### Shopping list (obligatoire)

Accessible aux participants, uniquement si l'Ã©vÃ©nement a `"shopping_list": true` (sinon `404`).

| MÃ©thode | Route | Droit | Description |
|---|---|---|---|
| GET | `/event/:idevent/shopping-items` | participant | Liste des choses apportÃ©es (triÃ©e par heure d'arrivÃ©e) |
| POST | `/event/:idevent/shopping-item` | participant | Ajouter ce qu'on apporte |
| PUT | `/event/:idevent/shopping-item/:id` | auteur | Modifier un Ã©lÃ©ment |
| DELETE | `/event/:idevent/shopping-item/:id` | auteur ou organisateur | Supprimer un Ã©lÃ©ment |

Exemple `POST /event/:idevent/shopping-item` :

```json
{ "name": "Chips", "quantity": 3, "arrival_time": "2026-12-31T19:30:00Z" }
```

### Covoiturage (Bonus)

Accessible aux participants, uniquement si l'Ã©vÃ©nement a `"carpooling": true` (sinon `404`).

| MÃ©thode | Route | Droit | Description |
|---|---|---|---|
| GET | `/event/:idevent/carpools` | participant | Trajets proposÃ©s avec places restantes (`remaining_seats`) |
| POST | `/event/:idevent/carpool` | participant | Proposer un trajet (on en devient le conducteur) |
| PUT | `/event/:idevent/carpool/:id` | conducteur | Modifier un trajet (places â‰¥ passagers dÃ©jÃ  inscrits) |
| DELETE | `/event/:idevent/carpool/:id` | conducteur ou organisateur | Supprimer un trajet |
| POST | `/event/:idevent/carpool/:id/join` | participant | RÃ©server une place |
| POST | `/event/:idevent/carpool/:id/leave` | passager | Annuler sa place |

Exemple `POST /event/:idevent/carpool` (`max_detour` est en minutes : sur un trajet de 2h30, `30` donne un trajet de 3h maximum) :

```json
{
  "departure_location": "Gare de Lyon, Paris",
  "departure_time": "2026-12-31T17:00:00Z",
  "price": 10,
  "seats": 3,
  "max_detour": 30
}
```

## ScÃ©nario de test rapide (Postman)

1. `POST /users` deux fois (utilisateurs A et B), puis `POST /auth/login` avec A.
2. `POST /group` avec A, puis `POST /group/:id/join` avec B (groupe public).
3. `POST /event` avec A en donnant `group` et `invite_group_members: true` : B est membre de l'Ã©vÃ©nement.
4. `GET /event/:id/thread`, puis `POST /thread/:idthread/message` avec A et une rÃ©ponse avec B (`reply_to`).
5. `POST /album` (A), `POST /album/:idalbum/photo` (B), `POST /album/:idalbum/photo/:idphoto/comment` (A).
6. `POST /event/:idevent/poll` (A), `GET /poll/:id` pour rÃ©cupÃ©rer les ids, `POST /poll/:id/response` (B), `GET /poll/:id/results`.
7. `POST /event/:idevent/ticket-type` (A, Ã©vÃ©nement public avec `ticketing: true`), puis `POST /event/:idevent/ticket` sans token ; un deuxiÃ¨me achat avec le mÃªme email renvoie `409`.
8. CrÃ©er l'Ã©vÃ©nement avec `"shopping_list": true` et `"carpooling": true`. `POST /event/:idevent/shopping-item` avec A, puis le mÃªme `name` avec B â†’ `409`. `POST /event/:idevent/carpool` avec A (`seats: 1`), `POST /event/:idevent/carpool/:id/join` avec B â†’ `200`, puis Ã  nouveau avec B â†’ `409`.

## ComplÃ©ments

- `PUT /poll/:id` : modification avant le premier vote.
- `PUT /thread/:idthread/message/:idmessage` : modification par lâ€™auteur.
- `GET /event/:id/share` : liens de partage Facebook et LinkedIn, rÃ©servÃ©s aux organisateurs dâ€™Ã©vÃ©nements publics dans un groupe public. Aucun envoi automatique. Configurer PUBLIC_EVENT_BASE_URL avec une vraie page publique pour le partage externe.

## Rendu Postman et GitHub

Voir `GUIDE_RENDU.md`. URL de documentation Postman : Ã  renseigner aprÃ¨s publication rÃ©elle.

## Documentation API
https://sara-s-team.docs.buildwithfern.com/my-social-networks-api-tp/introduction

