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
      <a href="#utiliser-le-body-parser">Utiliser le body-parser</a>
    </li>
    <li>
      <a href="#déployer-son-projet-node">Déployer son projet node</a>
      <ul>
        <li><a href="#heroku">Heroku</a></li>
            <ul>
                <li><a href="#création-de-vos-environnements">Création de vos environnements</a></li>
                <li><a href="#liaison-du-projet-avec-heroku">Liaison du projet avec Heroku</a></li>        
                <li><a href="#déployer-votre-projet">Déployer votre projet</a></li>   
            </ul>
      </ul>
    </li>
    <li>
      <a href="#intégrer-typescript">Intégrer TypeScript</a>
    </li>
    <li>
      <a href="#tester-avec-jest-et-supertest">Tester avec Jest et Supertest</a>
      <ul>
        <li><a href="#jest-c-est-quoi">Jest c'est quoi ?</a></li>
        <li><a href="#installation-de-jest">Installation de Jest</a></li>
        <li><a href="#mon-premier-test">Mon premier test</a></li>        
        <li><a href="#utilisation-de-supertest">Utilisation de Supertest</a></li>             
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
      <a href="#authentification-jwt">Authentification JWT</a>
    </li>
  </ol>
</details>


# Prérequis

-   Un nordinateur
-   Wordpad
-   Node.js version 18

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
```

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

# Authentification JWT

Il convient très souvent d'avoir un système permettant d'authentifier les appels à son API. Les token JWT sont devenus une meilleur pratique que les sessions classiques.

Un token  JWT c'est ça : eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InRlc3R1cGRhdGU0QGdtYWlsLmNvbSIsImlkIjo0LCJpYXQiOjE2NTExNDY0NjIsImV4cCI6MTY1MTE1MDA2Mn0.Fd_GSMptgZNL9BbeASs3mIhL0gvPCDWGkJyXw60sW9s

Il contient des informations encodées qui peuvent être décodées mais une autre partie ne peut l'être qu'avec la clé de chiffrement qui a permis sa génération.

C'est avec cette clé stockées sur le serveur qu'on pourra valider un token.

## Installation de Jsonwebtoken et dotenv

Librairie permettant de générer des tokens JWT.
dotenv sert à lire les variable d'environnement.

```bash
npm i jsonwebtoken dotenv
```

Ensuite, on ajoute une constante qui nous renverra la clé de chiffrement dans l'application.

```js
//config/auth.config.js

require('dotenv').config();

module.exports = {
    jwtSecret: process.env.JWT_SECRET,
    jwtExpiration: 3600
};

```

Ne pas oublier d'ajouter le secret dans le .env ! (à placer à la racine du projet) 


Au niveau de l'application, on ajoute une route qui va nous permettre de générer un token JWT.
L'implementation de cette route diffère selon vos cas d'usages mais peut être comme ceci:

```js
// routes/index.js

router.post("/login", async (req, res) => {
    try {
        // on récupère les données de connexion envoyées 
        const {email, password} = req.body;

        // on va chercher dans la bdd un utilisateur avec cet email
        const user = await userService.findUserByEmail(email);

        // on compare les mots de passes (qui sont hashés dans ce cas)
        if (user && await bcrypt.compare(password, user.password)) {
            // on génère un token JWT et on l'envoie au client
            // on ajoute d'autres informations basiques sur
            // dans la réponse

            jwt.sign({email: user.email, id: user.id}, jwtSecret, {algorithm: 'HS256', expiresIn: jwtExpiration }, (err, token) => {
                res.send({
                    token,
                    id: user.id,
                    email: user.email
                });
            })
        } else {
            res.status(401);
            res.send({message: 'Email or password incorrect'});
        }
    } catch (error) {
        res.status(500).send({message: 'An error occured while signin you in'})
    }
});
```

Maintenant qu'on a notre route pour se connecter, voyons comment on va utiliser le token JWT pour authentifier nos requêtes.

! Attention a bien ajouter le préfixe `Bearer ` au token JWT avant de l'envoyer dans les headers de la requête.


## Création d'un middleware d'authentification

```js
// middlewares/auth.middleware.js

const jwt = require('jsonwebtoken');
const {jwtSecret} = require("../../config/auth.config");
const UserService = require("../services/userService");
const userService = new UserService();

const auth = (req, res, next) => {
    try {
        // on récupère le token dans le header
        const authHeader = req.headers.authorization
        if(!authHeader){
            res.status(401).send({message:'You are not logged in'});
            return;
        }
        // on enlève le préfixe "Bearer "
        const token = authHeader.split(' ')[1];
        // on décode le token JWT de manière asynchrone
        jwt.verify(token, jwtSecret, async (err, decoded) => {
            if(!decoded?.id){
                res.status(401).send({message: 'Unable to verify token'})
            }
            const user = await userService.findUserById(decoded.id);
            if (!user) {
                res.status(404).send({message: "User not found"});
            } else {
                // si le token est validé on passe l'utilisateur courant à la requête pour le réutiliser dans nos routes pour l'autorisation d'accès aux ressources
                req.user = user;
                next();
            }
        });
    } catch(error) {
        res.status(401).send(error);
    }
};

module.exports = auth;
```

On exporte le middleware dans le fichier principal

```js
const authJwt = require("./auth.middleware");
module.exports = {
    authJwt,
};
```

Et voilà, on a notre middleware d'authentification.

## Ajouter le middleware dans les routes

Il faut maintenant appliquer ce middleware. Pour cela, on ajoute le middleware dans le second paramètre de la route.

TIP : Si vous utilisez plusieurs middelwares, il faut les ajouter dans un tableau.

Dans le cas ci-dessous, on veut récupérer les informations d'un utilisateur avec un id. On ne peut le faire qu'en étant cet utilisateur.

On va donc grâce au middleware pouvoir comparer les id utilisateurs de la base de donées et de celui connecté et donc valider l'autorisation d'accès à la ressource !
```js

router.get("/:id", authJwt, async (req, res) => {
    try{

        const user = await userService.findUserById(req.params.id);
        
        if(user){
            if(user.id !== req.user.id && !user.isAdmin){
                res.status(403).send({message:'Not authorized'})
                return
            }

            res.json(user);

        }else{
            res.status(404).send({message: 'User not found'})
        }
    }catch (error){
        res.status(500).send({message: 'Internal server error'})
    }

});

```
# Implémenter des sockets

## A quoi servent les sockets ?

Les sockets permettent à un client et un serveur de communiquer de façon instantannée sans rechargement de page. Elles peuvent par exemple être utile pour créer un chat en ligne ou un jeu multijoueurs.

## Comment implémenter les sockets ?

Il faut d'abord installer le package socket.io: 
```bash
npm i socket.io
```

On peut ensuite initialiser les sockets dans notre fichier server :
```js
const express = require('express');
const { createServer } = require('http');
const { Server } = require('socket.io');

// Create server
const app = express();
const httpsServer = createServer(app);
const io = new Server(httpsServer);

// Launch server
httpsServer.listen(3000, () => {
  console.log(`Example app listening on port 3000`);
});

// Socket IO
io.on("connection", (socket) => {
  // TODO - Socket listeners
});
```

Avec cette initialisation, Socket.io crée automatiquement une route '/socket.io/socket.io.js'. On peut donc y connecter notre vue et initialiser notre client :
```html
  <script src="/socket.io/socket.io.js"></script>
  <script>
    const socket = io.connect('http://localhost:3000/');
    // TODO - My code
  </script>
```

On peut ensuite envoyer notre premier événement du client vers le serveur en continuant dans notre balise ```<script>``` :
```js
  const socket = io.connect('http://localhost:3000/');
  socket.emit('msgToServer', 'Salut serveur !');
```
Ici, on envoie un événement appelé "msgToServer" avec un argument contenant notre message (on peut ajouter autant d'arguments qu'on le souhaite).

Côté serveur, on va attendre cet événement et répondre au client en lui émettant un autre message.
```js
  // Socket IO
  io.on("connection", (socket) => {
    // Listeners
    socket.on('msgToServer', (message) => {
      console.log(message);
      socket.emit('msgToClient', 'Salut client !');
    });
  });
```

Enfin, on va attendre ce nouvel événement chez notre client, toujours dans notre balise ```script``` :
```js
  socket.on('msgToClient', (message, message) => {
    console.log(message);
  });
# Utiliser le body parser

## L'erreur "req.body is undefined"

Lorsqu'il vous arrive d'envoyer de la donnée via une méthode POST, le serveur reçoit le contenu de celle-ci via le paramètre ```(req)```.
Précision : ```req``` fait ici référence à la requête envoyée par le client.

```js
app.post('/user', (req, res) => {
    console.log("Reponse : ", req.body)
```

Pour que le serveur puisse lire le contenu de de la requête, nous devons accèder à son body via ```req.body```.

Cependant, sur un serveur express, il se peut que votre donnée soit "undefined" lorsque vous essayez de ```console.log()``` celle-ci.

## Résoudre cette erreur

Cette erreur peut être résolu en utilisant le middleware ```body-parser```.
Celui-ci va parser notre réponse. En outre, le ```body-parser``` va extraire le body de la requête reçue et l'exposer sur le ```req.body```

## Installation

```
npm install body-parser --save
```

## Déclaration

```js
const bodyParser = require('body-parser');
```
Mettez ces deux ligne au début de votre code :
```js
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
```

## Mise à jour

Cette méthode est surtout utile si vous utilisez une version d'Express.js inférieure à Express 4.

Depuis Express 4, il est possible de fonctionner comme ceci :

```js
app.use(express.json());
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


## Intégrer TypeScript
[TypeScript](https://typescriptlang.org/) est un sur-langage de JavaScript dont le but est de permettre de typer les éléments JS. Il est extrêmement répandu dans l'écosystème JavaScript de nos jours. 

### Typer son code
Prenons un exemple simple : soit variable `id` étant un nombre hexadécimal, et, pour récupérer l'id suivant, on l'incrémente.

```ts
const id = "a25f0";
const getNextId = (id) => id + 1;
```

Pas de souci, n'est-ce pas ?

Seulement, si on regarde le résultat... 
```ts
const id = "a25f0";
getNextId(id) // "a25f01" au lieu de "a25f1" ! 👎👎
```

L'id a été traité comme un string, et donc `id + 1` a été traité comme une concaténation au lieu d'une addition 🤨

Il aurait fallu pouvoir s'assurer que `getNextId` prend bien un nombre en paramètre, et c'est là que TypeScript se rend utile.

Il suffit d'ajouter une petite annotation pour indiquer le type.
```ts
const getNextId = (id: number) => id + 1; // le paramètre "id" est de type "number"  
```

On peut même aller plus loin et indiquer le type du retour de la fonction !
```ts
const getNextId = (id: number): number => id + 1; // la fonction renvoie un "number"
```

Retournons à notre id : 
```ts
const id: number = "a25f0"; // Error: Type 'string' is not assignable to type 'number'. ts(2322) 
```
On voit directement le problème ici : le contenu de la variable id n'est pas du bon type !

On peut donc corriger le problème en le convertissant en nombre et continuer à coder en toute sérénité :)
```ts
const id: number = parseInt("a25f0", 16); // no problemo !
const getNextId = (id: number): number => id + 1;

getNextId(id); // a25f1 👍👍
```

[Exemple du code sur Codesandbox.io](https://codesandbox.io/s/cours-typescript-9433qz?file=/src/index.ts)

### Compiler TypeScript
On ne peut pas exécuter directement du TypeScript : il faut le compiler en JavaScript avec de le faire exécuter.

Il existe de nombreux outils (Webpack + Babel, `tsc` puis `node`...), mais dans le cadre de Node, le plus simple est [ts-node](https://www.npmjs.com/package/ts-node).

Il se charge de compiler puis d'exécuter un fichier `.ts`, de la même manière que node exécute un `.js`.

Pour développer, il existe une version intégrant `nodemon` : [ts-node-dev](https://www.npmjs.com/package/ts-node).

```json5
// package.json
{
  "scripts": {
    "dev": "tsnd index.ts",
    "prod": "ts-node index.ts"
  },
  "devDependencies": {
    "typescript": "^4.6.2",
    // ...
  }
}
```

### Les types des librairies importées
Bien plus que le code de notre équipe, il est intéressant d'avoir des types sur les librairies qu'on installe depuis npm 😉

Certaines sont écrites en TS, et donc les types sont déjà intégrés tels quels. Super !
![](typescript/integrated_ts.png)

Pour d'autres, il est nécessaire de les récupérer via un autre package (typiquement avec `npm i -D @types/<nom_de_la_lib>`).
![](typescript/external_ts.png)

Pour info, @types vient du dépôt [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) qui est open source !

# Tester avec Jest et Supertest

## Jest c'est quoi ?

Jest est un framework permettant de tester son code JavaScript afin de garantir un code de qualité avec la maximum de simplicité.

Il est possible d'utiliser Jest avec n'importe quel projet utilisant : Babel, TypeScript, Node, React, Angular, Vue, etc...

## Installation de Jest

Dans un premier temps, vous devez installer Jest (NPM ou Yarn) :

Avec NPM :
```bash
npm install --save-dev jest
```
Avec Yarn :
```bash
yarn add --dev jest
```

## Mon premier test

Avant de tester une route, nous allons tester le retour d'un simple calcul pour découvrir la syntaxe de Jest.

Dans un premier temps créer un dossier tests avec un fichier main.test.js.

```bash
test('Additionner 5 + 7', () => {
  expect(5 + 7).toBe(12);
});
```

Ensuite, ouvrez votre package.json et ajouter le script suivant :

```bash
{
  "scripts": {
    "test": "jest"
  }
}
```

Lancer `npm test` ou `yarn test`

Résultat :
```bash
 PASS  test/main.test.js
  ✓ Additionner 5 + 7 (2 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
```

## Utilisation de Supertest

Supertest vous permettra d'aller tester des routes préalablement configurées. Vous pourrez passer les paramètres souhaités afin de tester n'importe quelle route.

Installons Supertest :

Avec NPM :
```bash
npm install supertest --save-dev
```
Avec Yarn :
```bash
yarn add --dev supertest
```

Ensuite il faut ajouter Supertest au fichier de test préalablement crée, ainsi que vos routers déjà configurés.
```bash
const express = require("express");
const request = require("supertest");

const app = express();

const router = require("../routes/main");
```

Nous allons tester notre page d'accueil, celle-ci renvoi "Hello World!"
Pour que le test fonctionne, le test doit renvoyer le statut `200` ainsi que le contenu `Hello World!`

```bash
app.use("/", router);

test("Test la route '/', doit renvoyer Hello World!", async () => {
  const res = await request(app).get("/");
  expect(res.statusCode).toBe(200);
  expect(res.text).toBe("Hello World!");
});
```

Résultat :
```bash
 PASS  test/main.test.js
  ✓ test if get request on '/' respond Hello World! (32 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
```
