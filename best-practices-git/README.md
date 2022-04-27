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


## Commitizen


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
