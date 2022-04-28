# Documentation d'utilisation de Sequelize

L'objectif est d'utiliser l'ORM Sequelize avec le Framework Express de NodeJS.

## Pré-requis 

Avoir NodeJS d'installé sur sa machine.
<br>
Vous pouvez tester l'installation et la version de votre NodeJS avec la commande :
```
node -v
```
Avoir un projet Express, par exemple un "hello world" suffit amplement.

## Installation de Sequelize CLI

Pour installer le "command line interface" de Sequelize sur votre machine, veuillez faire la commande suivante :
```
npm install -g sequelize-cli
```
## Installation de Sequelize dans votre projet Express

Veuillez utiliser la commande suivante :
```
npx sequelize-cli init 
```
Vous constaterez que 4 nouveaux dossiers sont désormais dans votre projet :

* `config`
* `models` 
* `migrations`
* `seeders`

## Création d'un model et d'une migration en 1 seule commande 

Pour créer un model et sa migration, nous allons prendre l'exemple d'un `USER` : 

Un utilisateur possède 5 attributs : 
1. <i>un email</i>
2. <i>un mot de passe</i>
3. <i>un rôle</i>
4. <i>un prénom</i>
5. <i>un nom de famille</i>

La commande à effectuer :
```
npx sequelize-cli model:generate --name User --attributes email:string,password:string,role:string,firstname:string,lastname:string
```
Un fichier `"user.js"` s'ajoute dans votre dossier `model` et un fichier `"000000-create-user.js"` s'ajoute dans le dossier `migration`

## Relié votre / vos base de données à votre projet Express

Lorsque vous avez effectué la commande : `npx sequelize-cli init`, un fichier `config.json` c'est créée dans le dossier `config`.

Voici le contenu du fichier :
```json
{
    "development": {
      "username": "", // Nom d'utilisateur
      "password": "", // Mot de passe d'accès
      "database": "node-demo-app", // Nom de la base de données
      "host": "127.0.0.1", // Adresse du serveur de base de données
      "dialect": "postgres" // MySQL / Postgres etc...
    },
    "staging": {
      "username": "",
      "password": "",
      "database": "node-demo-app",
      "host": "127.0.0.1",
      "dialect": "postgres"
    },
    "production": {
      "username": "",
      "password": "",
      "database": "",
      "host": "127.0.0.1",
      "dialect": "postgres"
    }
}
```

Vous devez compléter les informations de ce fichier avec VOS informations.

## Effectuer la migration de vos tables vers la base de données

Pour migrer vos tables vers votre serveur de base de données, il suffit d'effectuer la commande suivante :

```
npx sequelize-cli db:migrate
```

## Création d'une seed

Une seed est une donnée fictive, c'est très pratique pour effectuer des tests et toutes sortes de manipulation.

Dans un premier temps il faut créer un fichier, avec la commande suivante :

```
npx sequelize-cli seed:generate --name seed-user
```

Dans le dossier `seeders`, un nouveau fichier est disponible, dans mon cas `20220427101622-seed-user.js`.

Veuillez compléter le fichier avec le code suivant :

```js
'use strict'

module.exports = {
    async up (queryInterface, Sequelize) {
        return queryInterface.bulkInsert('Users', [{
            firstname : 'John',
            lastname : 'Doe',
            email : 'johnDoe@test.com',
            password: "password",
            createdAt : new Date(),
            updatedAt : new Date(),

        }], {})
    },

    async down (queryInterface, Sequelize) {
        /**
         * Add commands to revert seed here.
         *
         * Example:
         * await queryInterface.bulkDelete('People', null, {});
         */
    }
}
```

Et pour finir, vous pouvez "peupler" votre base de données avec la commande suivante :

```
npx sequelize-cli db:seed:all
```

## Annexes

Il existe plusieurs commandes pour "reverse" vos actions, dans cette documentation je présente la méthode pour FAIRE, pas pour DÉFAIRE.

Voici quelques commandes qui peuvent vous être utiles :

Défaire les migrations
```
npx sequelize-cli db:migrate:undo
```

Défaire les seeds
```
npx sequelize-cli db:seed:undo
```





