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
    </li>
    <li>
      <a href="#ajouter-un-package-via-npm">Ajouter un package via npm</a>
    </li>
    <li>
      <a href="#les-exports-en-javascript">Les exports en javascript</a>
    </li>
    <li>
      <a href="#enlever-les-erreurs-cors">Enlever les erreurs CORS</a>
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

