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

Ne pas oublier d'ajouter le secret dans le .env !


Ensuite, il suffit d'ajouter à la racine de votre projet les lignes suivantes :
```js
var express = require('express');
var cors = require('cors');
var app = express();

app.use(cors());
```

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
