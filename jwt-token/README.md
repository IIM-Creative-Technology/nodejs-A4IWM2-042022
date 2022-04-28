# Comment implémenter l'authentification avec des JWT

## Un JWT c'est quoi ?
JWT (JSON Web Token) sert à transmettre en toute sécurité des informations sous la forme d'un objet JSON entre un client et un serveur par exemple. Il est le plus souvent utilisé lors d'appel à des API RESTful, afin d'authentifier le client voulant faire une requête. Il a une durée de vie limité et permet de ne pas avoir à se login à chaque fois.

### Comment est généré un jeton JWT ?

Pour générer un jeton JWT, on utilisera souvent une donnée de l'utilisateur en question (comme le mail ou le username, avec l'id), un algorithme de chiffrage (type HS256) et une durée de validité. Pour une question de sécurité, on peut ajouter d'autres options comme une private key.

### Comment une API recupère-t-elle un jeton JWT ?

Le token est souvent stocké dans le cache ou les cookies du client. Lors d'un appel API ce dernier sera récupéré dans le header ou cookie de la requête.

Il sera ensuite vérifié pour autoriser ou non l'accès, ainsi qu'orchestrer la récupération de l'utilisateur par exemple.

### Le refresh quoi ?

Un token JWT n'arrive souvent pas seul mais avec son ami le refresh token. Le refresh token est un second jeton JWT, avec une durée de vie plus longue. 

Lorsque le token principal a expiré, on stop la requête en cours et on check si le refresh token est toujours valide. Si c'est le cas on génère un nouveau token et refresh token, puis on relance la requête. (Un procédé invisible aux yeux du client)

## Node.js et JWT

Pour utiliser JWT dans Node.js avec express, nous aurons besoin de 2 packages.
- jsonwebtoken :
```npm i jsonwebtoken```
- cookie-parser :
```npm i cookie-parser```

Nous allons créer deux fichiers :
### Service ***(./services/authentication/authenticationService.js)***
```js
const jwt = require("jsonwebtoken")

class AuthenticationService {
	#jwtKey = process.env.JWTPRIVATEKEY;
	#jwtExpirySeconds = 86400;
	#refreshTokenExpirySeconds = 172800;

	createJwtToken = (res, username) => {
		// Create a new token with the username in the payload
		// and which expires 1 day after issue
		const token = jwt.sign({ username }, this.#jwtKey, {
			algorithm: "HS256",
			expiresIn: this.#jwtExpirySeconds,
		});

		const refreshToken = jwt.sign({ username }, this.#jwtKey, {
			algorithm: "HS256",
			expiresIn: this.#refreshTokenExpirySeconds,
		});

		// set the cookie as the token string, with a similar max age as the token
		// here, the max age is in milliseconds, so we multiply by 1000
		res.cookie("token", token, { maxAge: this.#jwtExpirySeconds * 1000 });
		res.cookie("refreshToken", refreshToken, { maxAge: this.#refreshTokenExpirySeconds * 1000 });
	}

	refreshJwtToken = async (req, res) => {
		// Get token by cookie
		const refreshToken = req.cookies.refreshToken;

		if (!refreshToken) {
			res.redirect('/login');
		}

		var payload
		try {
			payload = jwt.verify(refreshToken, this.#jwtKey);
			await createJwtToken(res, payload.username);
		} catch (e) {
			if (e instanceof jwt.JsonWebTokenError) {
				res.redirect('/login');
			}
			res.redirect('/login');
		}
	}

}

module.exports = AuthenticationService;
```


### Middleware ***(./middleware/authentication.js)***
```js
const jwt = require("jsonwebtoken");
const AuthenticationService = require("../services/authentication/authenticationToken");
const authService = new AuthenticationService();

class AuthenticationMiddleware{
    authentication = async (req, res, next) => {
        try {
            const token = req.cookies.token;
            if (!token) return res.redirect('/login');
    
            const decoded = jwt.verify(token, process.env.JWTPRIVATEKEY);
            req.user = decoded;
            return next();
        } catch (error) {
            await authService.refreshJwtToken(req, res);
        }
    };
}

module.exports = AuthenticationMiddleware;
```

### Serveur ***(app.js)***

Nous allons ensuite pouvoir initialiser notre serveur :
```js
const express = require("express");
const app = express();
const port = process.env.PORT || 3000;
const cookieParser = require("cookie-parser");
const http = require('http').Server(app);
const auth  = require('./src/middleware/authentication');
const authService  = require('./src/services/authentication/authenticationToken');

app.use(express.json());
app.use(cookieParser());

// Route need an auth user
app.get("/", new auth().authentication, (req, res) => {
  res.send("Hello world !");
});

// Route to log in a user
app.get("/login"), (req, res)=>{
    res.sendFile(__dirname + '/template/login.html');
});
app.post("/login", (req, res)=>{
    new authService().createJwtToken(res, req.body.username);
});

http.listen(port, async () => {
  try {
    console.log("Connection has been established successfully.");
    console.log(`Example app listening on port : http://127.0.0.1:${port}`);
  } catch (error) {
    console.error("Unable to connect:", error);
    server.close();
  }
});
```

## Comment ça fonctionne ?
Lors du login nous allons appeler notre service qui va créer un jeton JWT (ainsi que le refresh).

Notre middleware permet de bloquer certaines routes. Si nous allons sur notre page racine sans nous être log, nous serons automatiquement redirigé sur le login. Si nous sommes log, que nous avons nos tokens, le principal va être vérifié, puis notre requête va continuer à se dérouler.
Si notre token est expiré, alors nous appelons la méthode refreshToken de notre service qui va nous permettre de générer à nouveau nos tokens. (Si notre refreshToken est toujours valide et vérifié).

Ainsi nous limitons la tâche pénible de l'utilisateur de se connecter tout le temps.