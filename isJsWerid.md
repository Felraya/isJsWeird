# Is JS Weird ? 🤪

> Présentation interactive sur les bizarreries de JavaScript, en deux parties :
>
> 1. **Les bases** : une révision rapide des notions nécessaires pour comprendre la suite.
> 2. **Les bizarreries** : chaque exemple renvoie à une notion vue en partie 1.
>
> Format des bizarreries : **question → vote du public → exécution live → explication**.

---

## Introduction

- Accroche : `[] + {}` vs `{} + []`… qui ose parier ? (on garde la réponse pour la fin)
- Pourquoi JS est comme ça : créé en 10 jours (1995, Brendan Eich), et la règle **"don't break the web"** : on ne peut pas corriger les erreurs de départ sans casser des sites existants.
- Déroulé : d'abord les bases, ensuite les bizarreries.

---

# Partie 1 : Les bases de JavaScript

## 1.1 Les types

JS a **7 types primitifs** et **un type objet** :

| Type        | Exemples                              |
| ----------- | ------------------------------------- |
| `number`    | `42`, `3.14`, `NaN`, `Infinity`       |
| `string`    | `"hello"`, `'a'`, `` `template` ``    |
| `boolean`   | `true`, `false`                       |
| `undefined` | valeur d'une variable non initialisée |
| `null`      | "absence volontaire de valeur"        |
| `bigint`    | `9007199254740993n`                   |
| `symbol`    | `Symbol("id")`                        |
| `object`    | `{}`, `[]`, fonctions, `Date`…        |

- Les tableaux et les fonctions **sont des objets**.
- `typeof x` renvoie le type sous forme de chaîne.
- JS est **dynamiquement typé** : une variable peut changer de type.

➡️ Utile pour : 2.5 (`typeof`)

## 1.2 Un seul type de nombre

- Pas de distinction entier / décimal : tout est un **flottant 64 bits (IEEE 754)**.
- Valeurs spéciales : `NaN` (résultat d'une opération invalide), `Infinity`, `-Infinity`, `-0`.
- Entiers exacts seulement jusqu'à `Number.MAX_SAFE_INTEGER` (2⁵³ − 1).

➡️ Utile pour : 2.3 (les nombres)

## 1.3 Truthy / Falsy

En contexte booléen (`if`, `!`, `&&`, `||`), toute valeur est convertie en `true` ou `false`.

- **Falsy** (la liste complète) : `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`.
- **Tout le reste est truthy**, y compris `[]`, `{}`, `"0"`, `"false"`.

➡️ Utile pour : 2.2 (`[] == ![]`)

## 1.4 La conversion de type (coercition)

JS convertit les valeurs **automatiquement** quand un opérateur en a besoin.

- Conversion **explicite** : `Number("42")`, `String(42)`, `Boolean(0)`.
- Conversion **implicite** : faite par les opérateurs.
- Un objet converti en primitive appelle `valueOf()` puis `toString()` :
  - `[].toString()` → `""`
  - `[1, 2].toString()` → `"1,2"`
  - `({}).toString()` → `"[object Object]"`

➡️ Utile pour : 2.1 (coercition), 2.2 (égalité)

## 1.5 Les opérateurs

- `+` a **deux rôles** : addition **et** concaténation. Si un des côtés est une chaîne, il concatène.
- `-`, `*`, `/` ne font que des maths : ils convertissent toujours en nombre.
- `+x` (unaire) convertit `x` en nombre.
- `!x` convertit `x` en booléen puis l'inverse.

➡️ Utile pour : 2.1 (coercition)

## 1.6 Égalité : `==` vs `===`

- `===` (stricte) : même type **et** même valeur, aucune conversion.
- `==` (faible) : **convertit** les deux côtés avant de comparer, selon des règles compliquées.

➡️ Utile pour : 2.2 (l'égalité faible)

## 1.7 Les tableaux

- Un tableau est un objet dont les clés sont des index : `[0, 1, 2]`.
- `length` = plus grand index + 1 (pas le nombre d'éléments !).
- Méthodes de base : `push`, `map`, `filter`, `sort`.
- `map(fn)` appelle `fn(valeur, index, tableau)`.

➡️ Utile pour : 2.4 (les tableaux)

## 1.8 Variables, portée et fonctions

- `var` : portée **fonction**, "remontée" (hoisting) en haut de la fonction.
- `let` / `const` : portée **bloc** (`{ }`), pas utilisables avant leur déclaration.
- Fonctions classiques vs fonctions fléchées (`() => {}`).
- `this` : l'objet sur lequel la méthode est appelée (`obj.method()` → `this === obj`).

➡️ Utile pour : 2.6 (`this` et hoisting)

---

# Partie 2 : Les bizarreries

## 2.1 La coercition de type (le `+` et ses amis)

> Rappels : 1.4, 1.5

| Code                     | Résultat            | Pourquoi                                             |
| ------------------------ | ------------------- | ---------------------------------------------------- |
| `"22" + 2`               | `"222"`             | `+` avec une chaîne → concaténation                  |
| `"22" - 2`               | `20`                | `-` fait uniquement des maths → conversion en nombre |
| `[] + []`                | `""`                | `[].toString()` vaut `""`                            |
| `[] + {}`                | `"[object Object]"` | `({}).toString()`                                    |
| `true + true`            | `2`                 | `true` → `1`                                         |
| `"b" + "a" + +"a" + "a"` | `"baNaNa"`          | `+"a"` → `NaN` 🍌                                     |

## 2.2 L'égalité faible `==`

> Rappels : 1.3, 1.4, 1.6

| Code                | Résultat                                                             |
| ------------------- | -------------------------------------------------------------------- |
| `0 == "0"`          | `true`                                                               |
| `0 == ""`           | `true`                                                               |
| `"0" == ""`         | `false` 😱 (non transitif !)                                          |
| `null == undefined` | `true`                                                               |
| `null == 0`         | `false` … mais `null >= 0` → `true`                                  |
| `[] == ![]`         | `true` (`![]` → `false` car `[]` est truthy, puis `[]` → `""` → `0`) |

**Message** : toujours utiliser `===`.

## 2.3 Les nombres

> Rappel : 1.2

| Code                      | Résultat                                   | Pourquoi                              |
| ------------------------- | ------------------------------------------ | ------------------------------------- |
| `typeof NaN`              | `"number"`                                 | NaN est une valeur numérique spéciale |
| `NaN === NaN`             | `false`                                    | → utiliser `Number.isNaN()`           |
| `0.1 + 0.2 === 0.3`       | `false`                                    | IEEE 754 → `0.30000000000000004`      |
| `9999999999999999`        | `10000000000000000`                        | au-delà de `Number.MAX_SAFE_INTEGER`  |
| `Math.max()`              | `-Infinity`                                | valeur de départ du max               |
| `Math.min() > Math.max()` | `true`                                     | conséquence directe                   |
| `0 === -0`                | `true` … mais `Object.is(0, -0)` → `false` |                                       |

**Solutions** : `Number.EPSILON`, `BigInt`, `Object.is`.

## 2.4 Les tableaux

> Rappels : 1.4, 1.7

| Code                                 | Résultat        | Pourquoi                                                |
| ------------------------------------ | --------------- | ------------------------------------------------------- |
| `[10, 1, 3].sort()`                  | `[1, 10, 3]`    | tri **alphabétique** par défaut (conversion en chaînes) |
| `["1","2","3"].map(parseInt)`        | `[1, NaN, NaN]` | `map` passe l'index, que `parseInt` prend pour la base  |
| `[,,,].length`                       | `3`             | virgule finale ignorée, trous ("holes")                 |
| `const a = []; a[100] = 1; a.length` | `101`           | `length` = plus grand index + 1                         |
| `[1, 2, 3] + [4, 5, 6]`              | `"1,2,34,5,6"`  | conversion en chaînes puis concaténation                |

## 2.5 `typeof` et les types

> Rappel : 1.1

| Code                  | Résultat                                        |
| --------------------- | ----------------------------------------------- |
| `typeof null`         | `"object"` (bug historique de 1995)             |
| `typeof []`           | `"object"` → utiliser `Array.isArray()`         |
| `typeof function(){}` | `"function"` (alors que ce n'est pas un type !) |
| `typeof document.all` | `"undefined"` (bonus navigateur)                |

## 2.6 `this` et le hoisting

> Rappel : 1.8

- Perte de `this` : `const f = obj.method; f()` → `this` n'est plus `obj`.
- Les fonctions fléchées n'ont pas leur propre `this`.
- `var` utilisable avant sa déclaration (vaut `undefined`), `let` lève une erreur.
- Fonctions appelables avant leur déclaration.

## 2.7 Syntaxe piégeuse

- **ASI** (insertion automatique de `;`) :
  ```js
  function f() {
    return
    { ok: true }
  }
  f() // undefined
  ```
- Réponse à l'accroche : `{} + []` en début de ligne → `0` (le `{}` est lu comme un bloc vide !)
- Labels : `https://google.com` est du JS valide.
- `010` → `8` (octal legacy) mais `"010" * 1` → `10`.

## 2.8 Bonus : le chaos total

- **JSFuck** : écrire n'importe quel programme avec seulement `[]()!+`.
  - `(![] + [])[+[]]` → `"f"`
- Mois de `Date` indexés à partir de 0, jours à partir de 1.
- `new Date(2026, 0, 31)` puis `setMonth(1)` → 3 mars.

---

## Conclusion : comment s'en protéger ?

- `===` partout, `Number.isNaN`, `Array.isArray`, `Object.is`.
- `"use strict"` / modules ES.
- **TypeScript** (bloque déjà beaucoup de ces exemples — cf. les `// @ts-nocheck` de la démo 😉).
- Linters (ESLint : `eqeqeq`, `radix`…).
- JS n'est pas "cassé" : il est **spécifié** (ECMAScript), juste… créatif.

---

## Ressources

- [wtfjs](https://github.com/denysdovhan/wtfjs)
- Talk "Wat" — Gary Bernhardt (2012)
- [Spécification ECMAScript](https://tc39.es/ecma262/)
- [jsisweird](https://jsisweird.com/)
