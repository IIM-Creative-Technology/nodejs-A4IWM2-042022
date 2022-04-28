# Cours Node.js

Cours Node.js avec la classe A4 IWM M2

<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary><h2 style="display: inline-block">Table des matières</h2></summary>
  <ol>
    <li>
      <a href="#prérequis">Prérequis</a>
    </li>
    <li>
      <a href="#faire-son-serveur">Faire son serveur</a>
    </li>
    <li>
      <a href="#ajouter-un-package-via-npm">Ajouter un package via npm</a>
    </li>
    <li>
      <a href="#les-exports-en-javascript">Les exports en javascript</a>
    </li>
    <li>
      <a href="#upload-une-image-avec-multer">Upload une image avec Multer</a>
    </li>
    <li>
      <a href="#enlever-les-erreurs-cors">Enlever les erreurs CORS</a>
    </li>
    <li>
      <a href="#dockeriser-son-application">Dockeriser son application</a>
      <ul>
        <li><a href="#architecture-du-projet">Architecture du projet</a></li>
        <li><a href="#conteneur-base-de-données">Conteneur base de données</a></li>
        <li><a href="#conteneur-serveur">Conteneur serveur</a></li>        
        <li><a href="#fichier-env">Fichier .env</a></li>        
        <li><a href="#en-production">En production</a></li>       
        <li><a href="#lancer-les-conteneurs">Lancer les conteneurs</a></li>       
      </ul>
    </li>
  </ol>
</details>


<details>
  <summary><h2 style="display: inline-block">Autres documentations</h2></summary>
  <ol>
    <li>
      <a href="/course-package-npm">NPM et les packages</a>
    </li>
    <li>
      <a href="/best-practices-git">Bonnes pratiques git</a>
    </li>
    <li>
      <a href="#ce-que-nous-avons-vu">Ce que nous avons vu</a>
    </li>
  </ol>
</details>


# Prérequis

-   Un nordinateur
-   Wordpad
-   Node.js version 18

## Ce que nous avons vu
- [Node.js (version 18)](https://nodejs.org/en/)
- [Express](https://expressjs.com/fr/)
- [Socket.io](https://socket.io/fr/)
- [npm](https://www.npmjs.com) / [yarn](https://yarnpkg.com/) / node_modules
- Modules (import/require) / 
  - Code local / librairies internes / externes
  - export de modules
- Serveur Node.js
- Serveur express.js
- req, res, app
- Router
- CRUD
- Tests [(jest)](https://jestjs.io/fr/)
- [Librairie HTTP Node.js](https://nodejs.org/api/http.html)
- Hierarchisation du code
- Bonnes pratiques
  - Git [commitizen](https://github.com/commitizen/cz-cli)
  - [Git flow](https://www.atlassian.com/fr/git/tutorials/comparing-workflows/gitflow-workflow)
  - Pull Request
  - Review de code
- Présentation à l'oral
- Review (points blquants, objectifs atteints, tâches à faire)
- Travailler en équipe
- Liason BDD avec [PostgreSQL](https://www.postgresql.org/) ([Prisma](https://www.prisma.io/) / [Sequelize](https://sequelize.org/))
- Authentification
- Implémentation de [Docker](https://www.docker.com/)
- Mise en production ([Vercel](https://vercel.com/), [Heroku](https://www.heroku.com/))
- Changement de projet

# Faire son serveur

Définition des constantes globales de l'application.

On utilise la librairie native `http` de Node.js

```js
const http = require("http");

const hostname = "127.0.0.1";
const port = 3000;
```

On crée le serveur avec les paramètres nescessaires

```js
const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader("Content-Type", "text/plain");
    res.end("Hello, World!\n");
});
```

On lance le serveur sur le port et l'hôte défini.

La fonction de callback permet de nous assurer de la bonne exécution de notre code.

```js
server.listen(port, hostname, () => {
    console.log(`Server running at http://${hostname}:${port}/`);
});
```


# Dockeriser son application
On peut conteneuriser son application pour éviter d'avoir à installer tout sur son ordinateur, et optionnellement faciliter le déploiement.

Il y a pour l'instant 2 parties dans l'application, et donc 2 conteneurs. On peut utiliser un `docker-compose.yml` pour gérer ces conteneurs.

## Architecture du projet
Je propose ici une certaine architecture pour le projet, mais soyez libres de l'adapter à votre goût ! Il faudra juste changer les chemins correspondants dans `docker-compose.yml`.

![](architecture_du_projet.png)

## Conteneur base de données
Une simple image postgres importée de dockerhub est suffisante ici.

```yml
# docker/docker-compose.yml
services:
  db:
    image: postgres:14.2-alpine
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_DB: ${DB_DATABASE}
    ports:
      - ${DB_PORT}:${DB_PORT}
    volumes:
      - ../db:/var/lib/postgresql/data
    networks:
      - <nom_du_réseau_personnalisé> # pour que les conteneurs puissent discuter entre eux
```

## Conteneur serveur
Il y a besoin de commandes supplémentaires pour lancer le serveur : c'est pourquoi on va écrire un Dockerfile.

```yml
# docker/docker-compose.yml
services:
  server:
    depends_on:
      - db
    build:
      context: ../app/ # avec quels fichiers on initialise le conteneur
      dockerfile: ../docker/dev/Dockerfile # où se trouve le Dockerfile, à partir du contexte
    restart: unless-stopped
    environment:
      CONNECTION_STRING: "postgresql://${DB_USER}:${DB_PASSWORD}@db:${DB_PORT}/${DB_DATABASE}"
      SERVER_PORT: ${SERVER_PORT}
      NODE_ENV: dev
    ports:
      - "${SERVER_PORT}:${SERVER_PORT}"
    volumes:
      - ../app/src:/app/src # pas app/ en entier pour éviter d'écraser d'écraser node_modules dans le conteneur
    networks:
      - <nom_du_réseau_personnalisé>
```

```dockerfile
# docker/dev/Dockerfile
FROM node:18-alpine3.14

RUN mkdir /app
WORKDIR /app
# Install the dependencies
COPY package.json ./
RUN npm ci
# Copy the source files
COPY . .

# Start the server
EXPOSE ${SERVER_PORT}
CMD npm run dev
# ou npx nodemon <nom_du_fichier_principal> si vous n'avez pas de script npm pour lancer le serveur en dev
```

## Fichier env
Les variables d'environnement sont placées dans un fichier `.env` lu automatiquement par `docker-compose`.

```env
DB_USER=<nom_d'utilisateur>
DB_PASSWORD=<mot_de_passe>
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=<nom_de_la_bdd>
SERVER_PORT=3000
```

## En production
La commande pour lancer le serveur en production est différente de celle en développement, c'est pourquoi il y a un `Dockerfile` et un `docker-compose.yml` différents pour cet environnement.

```yml
# docker/production.yml
version: "3.4"

services:
  server:
    build:
      context: ../app/
      dockerfile: ../docker/prod/Dockerfile
    environment:
      NODE_ENV: production
```
```dockerfile
FROM node:18-alpine3.14

RUN mkdir /app
WORKDIR /app
# Install the dependencies
COPY package.json ./
RUN npm install --omit dev
# Copy the source files
COPY . .

# Start the server
EXPOSE ${SERVER_PORT}
CMD npm run start
```

## Lancer les conteneurs

### Développement
Pour toutes ces commandes, il faut se placer dans le dossier `/docker`.

En développement :
`docker-compose up -d`

On peut ensuite accéder au serveur depuis l'extérieur du conteneur à l'adresse : `localhost:3000`

Après avoir installé un module npm via `npm i`, reconstruire les conteneurs en ligne de commande avec :
`docker-compose up -d --build`

Ou supprimer le conteneur avec Docker Desktop :

![](docker_desktop_delete.png)

Puis relancer normalement avec `up`.

### Logs
Pour voir les logs, on peut soit utiliser la ligne de commande : `docker-compose logs -f <server ou db>`

Soit passer par Docker Desktop :
![](docker_desktop_logs.png)

### Production
En production, idem, mais on ajoute la configuration pour la prod :
`docker-compose -f docker-compose.yml -f production.yml up -d`


# Ajouter un package via npm
Dans un premier temps, on ajoute la 'notion' de package dans le projet.  
```bash
npm init
```

Ensuite on installe dans le projet le package souhaité (ici nodemon)
```bash
npm install nodemon --Dev
```

`nodemon` permet de faciliter le développement en relançant automatiquement le serveur lors de modifications des fichiers sources. On peut l'ajouter aux scripts npm:
```json
"scripts" : {
 "dev": "nodemon <nom_du_fichier_principal>"
}
```

Puis le lancer avec `npm run dev`

# Les exports en javascript
Dans un fichier appart créer une fonction
```js
function returnHelloWorld() {
    const helloWorldObject = {
        msg: 'Hello World !'
    }
    return JSON.stringify(helloWorldObject);
}
```

Dans ce meme fichier exporter la fonction que l'on vient de créer
```js
module.exports = {returnHelloWorld};
```

Importer dans le fichier souhaité l'ensemble des exports du fichier
```js
const functions = require('./functions');
```

Il est maintenant possible d'utiliser les fonctions exportés dans le fichier importé
```js
functions.returnHelloWorld()
```

# Upload une image avec Multer

Pour upload une image via un `<input type="file" />`, il faut installer le package [Multer](https://www.npmjs.com/package/multer) dans votre projet. Notons que package `express` doit déjà être installé au sein du projet pour le bon fonctionnement de Multer.
```
npm i multer
``` 

Dans votre rendu HTML, créer un formulaire 
```html
<form action="/upload" method="POST" enctype="multipart/form-data">
  <input type="file" name="myFile">
  <button type="submit">Submit</button>
</form>
```

Puis au sein de votre arborescence, créer un dossier où seront stockées les images
```
mkdir -p public/uploads/
```

Après avoir servi le dossier de destination statique dans votre fichier javascript principal, indiquer le path de stockage des images ainsi que leur nom
```js
app.use(express.static('./public'));

const storage = multer.diskStorage({
  destination: './public/uploads/',
  filename: function(req, file, cb){
    cb(null,file.fieldname + '-' + Date.now() + path.extname(file.originalname));
  }
});
```

Créer une méthode de validation du fichier, notamment pour l'extension de l'image
```js
function checkFileType(file, cb){
  const filetypes = /jpeg|jpg|png|gif/;
  const extname = filetypes.test(path.extname(file.originalname).toLowerCase());
  const mimetype = filetypes.test(file.mimetype);

  if(mimetype && extname){
    return cb(null,true);
  } else {
    cb('Error: Images Only!');
  }
}
```

Pour finir, créer une méthode d'upload et l'appeler au submit du formulaire 
```js
const upload = multer({
  storage: storage,
  limits:{fileSize: 1000000},
  fileFilter: function(req, file, cb){
    checkFileType(file, cb);
  }
}).single('myFile');

app.post('/upload', (req, res) => {
  upload(req, res)
});

# Enlever les erreurs CORS

## Qu'est-ce qu'une erreur CORS

Tout d'abord, CORS est un acronyme signifiant "Cross Origin Resource Sharing".

Cela consiste à ajouter des en-têtes HTTP afin d'accéder par exemple à une API située sur
un serveur distant.
Ainsi, le user agent procède à une requête HTTP "cross-origin" lorsqu'il souhaite accéder
à un domaine, un port ou un protocole différent de la page courante.

Il apparaît donc des erreurs CORS lorsque nous n'avons pas les droits d'accéder notamment à des URL distantes.

## Résoudre les erreurs CORS dans un projet NodeJS-Express

Il faut dans un premier temps installer un middleware que l'on retrouve dans un simple paquet NPM.
```bash
npm i cors
```
Ensuite, il suffit d'ajouter à la racine de votre projet les lignes suivantes :
```bash
var express = require('express');
var cors = require('cors');
var app = express();

app.use(cors());
```

# Déployer son projet node

## Heroku

### Création de vos environnements

Heroku est une plateforme permettant le déploiement d'applications web. Elle est connu pour être l'une des plateformes la plus simple d'utilisation pour déployer un projet web.

Dans cette partie nous allons voir le déploiement d'un projet node sur Heroku et nous allons nous baser sur la méthode de déploiement utilisant heroku-cli.

Dans un premier temps il vous faudra faire une pipeline sur votre dashboard Heroku. Vous n'avez pas besoin de lié votre pipeline à votre git si vous utilisez heroku-cli.

Vous allez par la suite devoir créer une application dans votre pipeline. Il s'agira de vos environnement. 

N'oubliez pas d'installer le buildpack node à votre application dans les Settings de cette-derniére. 

### Liaison du projet avec Heroku

Maintenant vous allez devoir installer heroku-cli :

MacOS :

```bash
w tap heroku/brew && brew install heroku
```

Windows :

<a href="https://cli-assets.heroku.com/heroku-x64.exe">https://cli-assets.heroku.com/heroku-x64.exe</a>

Linux (Ubuntu) :

```bash
curl https://cli-assets.heroku.com/install-ubuntu.sh | sh
```

Vous allez maintenant devoir vous connecter à votre compte Heroku et lier votre projet à votre application Heroku :

```bash
heroku login
heroku git:remote -a <nom_de_votre_app>
```

### Déployer votre projet

Créer un fichier Procfile à la racine de votre projet. Ce fichier va être le point d'entré de votre application. Il doit contenir la méthode de lancement de votre application. Exemple :

```
web: node app.js
```

Avant de déployer votre application veillez à bien avoir paramétré les variables d'environnement de votre application (Settings > Reveal Config Var)
Vous pouvez maintenant déployer votre application :

```bash
git push heroku main
```

Vous pouvez obtenir des logs en direct de l'état de votre application grâce aux logs heroku. Pour récupérer les logs il vous suffit de faire :

```bash
heroku logs --tail
```

