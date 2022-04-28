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
    <li>
      <a href="#intégrer-typescript">Intégrer TypeScript</a>
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
