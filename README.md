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
      <a href="#upload-une-image-avec-multer">Upload une image avec Multer</a>
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
"scripts": {
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
