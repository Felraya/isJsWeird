# isJsWeird

Présentation interactive (Jupyter + RISE) qui rassemble des bizarreries classiques de JavaScript, avec un petit quiz en direct
s
## Prérequis
- Python 3.x + [JupyterLab](https://jupyter.org/)
- [tslab](https://github.com/yunabe/tslab) — kernel JavaScript/TypeScript pour Jupyter
- [jupyterlab-rise](https://github.com/jupyterlab-contrib/rise) — extension de présentation (diaporama Reveal.js exécutable)

## Installation

```bash
pip install jupyterlab jupyterlab-rise
npm install -g tslab
tslab install
```

Vérifier que le kernel JavaScript est bien enregistré :

```bash
jupyter kernelspec list
```

## Lancer le projet

```bash
jupyter lab
```

Ouvrir [`main.ipynb`](main.ipynb) et sélectionner le kernel **TypeScript** (ou **JavaScript**) si ce n'est pas déjà fait.

## Faire la présentation

1. Dans chaque cellule, définir le **Slide Type** (Slide / Sub-Slide / Fragment / Skip) via le panneau de propriétés.
2. Cliquer sur l'icône **RISE** dans la barre d'outils pour entrer en mode présentation plein écran.
3. Naviguer avec les flèches, exécuter le code en direct avec `Shift+Entrée`.

## Format du quiz

Chaque bizarrerie suit le même schéma :
1. Une cellule **markdown** présente l'énigme (le bout de code à deviner).
2. Une cellule **code** (cachée ou en fragment) exécute l'expression pour révéler le résultat.
3. Une cellule **markdown** explique brièvement le pourquoi (coercition de types, hoisting, `this`, etc.).

## Structure

- [`main.ipynb`](main.ipynb) — notebook principal contenant l'ensemble des slides et du quiz.

