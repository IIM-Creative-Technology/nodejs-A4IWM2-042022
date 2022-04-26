# Les bonnes pratiques du package-lock.json

## Y’a quoi dans le package-lock ?
Le package-lock est un **snapshot** de l’arbre de dépendances d’un projet
    
    Il est auto-généré.

Pour chaque dépendance **et sous-dépendance** (de façon recursive):

- Version
- URL de téléchargement
- Hash d’intégrité
- D’autres trucs pas très importants pour cette présentation

L’intérêt du lock: avoir une représentation déterministe de `node_modules` sans avoir à comiter `node_modules`

# Comment npm fait-il pour gérer à la fois package.json et package-lock.json ?

Pour savoir quelle version installer lors d’un `npm install`, **npm** lit à la fois le `package.json` **et** le `package-lock.json` puis applique un comportement qui varie selon les cas.

<aside>
💡 Rappel syntaxique:

`1.2.3` → uniquement la version **1.2.3**
`~1.2.3` → toutes les versions **1.2**.X ≥ 1.2.3. Exemples: `1.2.6`, `1.2.99`
`^1.2.3` → toutes les versions **1**.X.Y ≥ 1.2.3, Exemples: `1.2.8`, `1.8.99` ,`1.33.520`

</aside>

### Cas n°1: Les versions matchent

```jsx
 //  package.json
 {
   "dependencies": {
     "prettier": "^2.2.1"
   }
 }
   
 // package-lock.json
 {
   "requires": true,
   "lockfileVersion": 1,
   "dependencies": {
     "prettier": {
       "version": "2.5.0",
       "resolved": "https://registry.npmjs.org/prettier/-/prettier-2.5.0.tgz",
       "integrity": "sha512-FM/zAKgWTxj40rH03VxzIPdXmj39SwSjwG0heUcNFwI+EMZJnY93yAiKXM3dObIKAM5TA88werc8T/EwhB45eg=="
     }
   }
 }
 ```

- Que fait **npm install** ?
    
**npm** installe la `2.5.0` , le package-lock est respecté à la lettre.


### Cas n°2: Les versions ne matchent pas (ou package absent du lock)

```jsx
   //  package.json
   {
     "dependencies": {
       "prettier": "^2.1.0"
     }
   }
   
   // package-lock.json
   {
     "requires": true,
     "lockfileVersion": 1,
     "dependencies": {
       "prettier": {
         "version": "1.19.1",
         "resolved": "https://registry.npmjs.org/prettier/-/prettier-1.19.1.tgz",
         "integrity": "sha512-s7PoyDv/II1ObgQunCbB9PdLmUcBZcnWOcxDh7O0N/UwDEsHyqkW+Qh28jW+mVuCdx7gLB0BotYI1Y6uI9iyew=="
       }
     }
   }
   ```

- Que fait **npm install** ?
    
    **npm** installe la `2.6.2` (la matching version la plus récente), puis remplace le `package.lock` par:
    
    ```jsx
    {
      "requires": true,
      "lockfileVersion": 1,
      "dependencies": {
        "prettier": {
          "version": "2.6.2",
          "resolved": "https://registry.npmjs.org/prettier/-/prettier-2.6.2.tgz",
          "integrity": "sha512-PkUpF+qoXTqhOeWL9fu7As8LXsIUZ1WYaJiY/a7McAQzxjk82OF0tibkFXVCDImZtWxbvojFjerkiLb0/q8mew=="
        }
      }
    }
    ```
    
    l’information contenue dans le package-lock **est complètement ignorée**.
    
    La commande lancée par npm en interne est en réalité un `npm install prettier@^2.1.0`
    

## Le cas du npm ci

<aside>
💡 Fun fact, `npm ci` est une commande destinée aux scripts d’intégration continue, mais en fait CI c’est un diminutif pour `npm clean-install`

</aside>

`npm ci` ressemble à `npm i` à quelques exceptions près:

- Si les version du `package.json` et du `package-lock` ne matchent pas (ou version manquante): le script échoue.
- Ne modifie jamais le package-lock
- Vide le dossier `node_modules` avant l’exécution

# Gestion de problématiques liées au lock

### Mise en situation #1

> Oh mince y’a écrit dans le `package.json` qu’on utilise `awesome-package` `^2.3.4` mais y’a un gros bug dans cette version il nous faut au moins la `2.5.0` !
> 

<aside>
⛔ Pas de soucis, j’écris `^2.5.0` à la place puis je lance `npm install`, ça va mettre le module à jour et régler le problème

</aside>

<aside>
✨ En regardant dans le package-lock, j’aurais pu voir que j’avais déjà `awesome-package@2.6.0`

D’ailleurs, en faisant `npm install`, mon lock n’a pas changé, la preuve que ma modification n’a absolument rien changé dans mes dépendances.

Pro tip: `npm ls <nom-du-package>` permet de voir la version installée pour aller encore plus vite.

</aside>

### Mise en situation #2

> J’ai masse de conflits sur mon package-lock depuis que j’ai mis des dépendances à jour 😢
OK pas de panique j’ai une idée pour repartir d’une base propre !
> 

<aside>
⛔ Je supprime le `package-lock` et je relance `npm install`. Problème réglé ! *#yolo*

</aside>

<aside>
✨ En faisant un `npm install` sans lock, ça met à jour **toutes les dépendances** du monorepo, en sautant potentiellement plusieurs versions mineures.
Problems coming soon !

A la place, je peux juste repartir d’un package-lock potentiellement un peu vieux mais qui reflète vraiment les packages qu’on avait à un instant *t*

Par exemple, si je suis en train de faire une PR sur master, je peux récupérer le dernier package-lock de master:
`git checkout origin/master package-lock.json`

Solution alternative pour être sûr de pas faire des conneries:

Ensuite, je peux lancer un `npm install` sereinement. Seuls les nouveaux packages vont être rajoutés au package-lock.

</aside>

## Bonnes pratiques générales

1. Ne **jamais** faire un `npm install` sans `package-lock`2. Toujours se questionner si une PR comporte un diff de `package-lock`:
2. Si `package.json` n’a pas changé, ça pue dans 99% des cas (le 1% restant correspond à un `npm audit fix`
3. Si une dépendance a été rajoutée ou mise à jour, le diff doit faire **quelques dizaines de lignes** maximum !