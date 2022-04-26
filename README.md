# Cours Node.js

Cours Node.js avec la classe A4 IWM M2

<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary><h2 style="display: inline-block">Table des matières</h2></summary>
  <ol>
    <li>
      <a href="#prerequis">Prérequis</a>
    </li>
    <li>
      <a href="#faire-son-serveur">Faire son serveur</a>
      <!-- <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul> -->
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

