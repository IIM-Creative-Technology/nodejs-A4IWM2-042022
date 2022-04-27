# Les bonnes pratiques Git

Cours Node.js avec la classe A4 IWM M2

<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary><h2 style="display: inline-block">Table des matières</h2></summary>
  <ol>
    <li>
      <a href="#git-flow">Git Flow</a>
    </li>
    <li>
      <a href="#commitizen">Commitizen</a>
    </li>
    <li>
      <a href="#template-de-pr">Template de PR</a>
    </li>
  </ol>
</details>

## Git Flow
Git Flow est une philosophie de gestion des branches dans un dépôt Git. [(Cf. tutoriel ici.)](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)

Le principe est le suivant :
* la branche 'main' est réservée à l'historique officiel des releases
* la branche 'develop' est réservée pour le développement (d'où le nom) de la prochaine version
* chaque feature doit être développée sur sa branche à part, en partant de 'develop'
* une fois qu'on a suffisamment de features, on crée une branche propre à une release, qui sera peaufinée, puis mergée avec 'main'
* les 'hotfix' sont des branches dédiées pour résoudre les bugs, accrochées à 'main' contrairement aux branches de features

Il existe un plugin git [(lien d'installation)](https://github.com/nvie/gitflow/wiki/Installation) pour s'en servir en ligne de commande.

Le graphique [ici](https://danielkummer.github.io/git-flow-cheatsheet/index.html) explique très bien les différentes commandes à utiliser.

## Commitizen
![](commit_names.png)

Evitons ça 👆

Commitizen aide à formatter ses messages Git avec un script interactif.

Installer commitizen et le package pour formatter :
`npm i -D commitizen cz-conventional-changelog`

Ajouter cette entrée au package.json :
```json5
// package.json
"config": {
    "commitizen": {
        "path": "cz-conventional-changelog"
    }
}
```

Ensuite, au lieu de `git commit`, on utilise `npx cz` pour commit en se laissant guider par le script :
![](cz.png)

Si on ne veut pas bouleverser ses habitudes, il est possible d'utiliser `git commit` avec quelques réglages supplémentaires :
 * aller dans le dossier caché de git `.git/`
 * aller dans le dossier de hooks `hooks/` : ce sont des scripts exécutés automatiquement lors de certains évènements de Git.
 * ajouter le fichier `prepare-commit-msg`
 * mettre ce contenu :
```shell
#!/bin/bash
exec < /dev/tty && node_modules/.bin/cz --hook || true
```

Ainsi commitizen sera appelé automatiquement à chaque fois qu'on fera `git commit` !

## Template de PR
Il n'est pas facile de relire la PR d'un autre, et ceci d'autant plus si elle manque d'une description suffisamment claire.

C'est pourquoi un template de PR peut grandement faciliter le travail d'une équipe.

Pour GitHub, il suffit d'ajouter le fichier `pull_request_template.md` à la racine du projet.

On peut mettre le template qu'on veut, mais voici un exemple :
```markdown
## Summary
| Q            | A                                                         |
|--------------|-----------------------------------------------------------|
| Branch?      | into <!-- PRs should be into 'develop' --> < from         |
| Bug fix?     | yes/no                                                    |
| New feature? | yes/no <!-- remember to update the README.md -->          |
| Task?        | closes #<!-- put the number of the related issue -->      |

## Changelog
### ✨ Features
* **[theme] Important change**
* [theme] Minor change
### 🐛 Fixes
### 🚀 Improvements
### 👻 Internal
### 🛠 Tools
```
