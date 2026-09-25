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
- Deuxième accroche : que vaut ceci ? (réponse expliquée en conclusion)
  ```js
  (![] + [])[+[]] +
    (![] + [])[+!+[]] +
    ([![]] + [][[]])[+!+[] + [+[]]] +
    (![] + [])[!+[] + !+[]];
  ```
- **Comment et pourquoi JS est né :**
  - **1995, Netscape** veut un langage **simple** pour rendre les pages web interactives.
  - **Brendan Eich** le crée en **10 jours**, avec une consigne : **ressembler à Java**.
  - Le nom **JavaScript** est un choix **marketing** : il n'a presque rien à voir avec Java.
  - **1997** : standardisé sous le nom **ECMAScript**.

- **La règle "don't break the web" :**
  - Un navigateur doit pouvoir afficher **tous les sites existants**, même ceux écrits en 1996 et jamais mis à jour.
  - Donc une nouvelle version de JS ne peut **jamais** changer le comportement d'un code qui marche déjà.
  - **Impacts :**
    - Les erreurs des **premières années** (1995-1999, jusqu'à ECMAScript 3) sont **gravées dans le marbre**. En 2006, une proposition de corriger `typeof null === "object"` a été rejetée : trop de sites en dépendaient.
    - On ne corrige pas, on **ajoute** : `===` à côté de `==`, `let`/`const` à côté de `var`, `Number.isNaN` à côté de `isNaN`, `"use strict"` en opt-in.
    - Conséquence : le langage **ne fait que grossir**. Les nouveautés s'empilent, les pièges restent.
- Et TypeScript dans tout ça ? Essayez l'accroche en TS :
  ```ts
  [] + {}
  // ❌ error TS2365: Operator '+' cannot be applied to types 'never[]' and '{}'.
  ```
  TypeScript **refuse de compiler** beaucoup des exemples de cette présentation. Il comble certains manques de JavaScript en détectant ces pièges **avant l'exécution**.
  - ⚠️ Mais TS n'est **qu'une couche de vérification** : le code compilé reste du JavaScript, avec exactement les mêmes règles à l'exécution. Par exemple, `{} + []` en début de ligne passe la compilation TS et donne toujours `0` (voir 2.7).
  - Pour la démo, on désactive la vérification avec `// @ts-nocheck` afin de voir le vrai comportement de JS.
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
- **Le `+` unaire** : placé **devant une seule valeur** (sans rien à sa gauche), `+x` convertit `x` en nombre. C'est un raccourci de `Number(x)`.

  | Code         | Résultat | Pourquoi                                                                          |
  | ------------ | -------- | --------------------------------------------------------------------------------- |
  | `+"42"`      | `42`     | chaîne numérique → nombre                                                         |
  | `+""`        | `0`      | une chaîne vide vaut `0`                                                          |
  | `+"12px"`    | `NaN`    | la chaîne entière doit être un nombre (contrairement à `parseInt("12px")` → `12`) |
  | `+true`      | `1`      | `true` → `1`, `false` → `0`                                                       |
  | `+null`      | `0`      | `null` → `0`…                                                                     |
  | `+undefined` | `NaN`    | … mais `undefined` → `NaN`                                                        |
  | `+[]`        | `0`      | objet → chaîne (1.4) : `[]` → `""` → `0`                                          |
  | `+{}`        | `NaN`    | `{}` → `"[object Object]"` → `NaN`                                                |

  - Comment le reconnaître : dans `a + b`, le `+` est **binaire** (deux valeurs). Dans `+b`, ou dans `a + +b`, le second `+` est **unaire** (une seule valeur, à sa droite).
  - Le `-` unaire fait la même conversion, puis change le signe : `-"5"` → `-5`.
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

| Code                     | Résultat            | Pourquoi                                                                                                                                                                                                        |
| ------------------------ | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `"22" + 2`               | `"222"`             | Un des côtés est une chaîne, donc `+` **concatène** : `2` devient `"2"`                                                                                                                                         |
| `"22" - 2`               | `20`                | `-` ne sait faire que des maths, donc `"22"` est converti en nombre                                                                                                                                             |
| `[] + []`                | `""`                | `+` ne sait pas additionner des objets : il les convertit en chaînes. `[].toString()` vaut `""`, et `"" + ""` → `""`                                                                                            |
| `[] + {}`                | `"[object Object]"` | Même mécanisme : `""` + `({}).toString()` → `"" + "[object Object]"`                                                                                                                                            |
| `{} + []`                | `0` 😱               | On inverse juste l'ordre, et le résultat change ! En début de ligne, `{}` n'est pas un objet mais un **bloc de code vide**. Il reste `+[]` : le `+` unaire convertit `[]` en `""`, puis en `0` (détails en 2.7) |
| `console.log({} + [])`   | `"[object Object]"` | Encore différent ! Ici `{}` est un **argument** de fonction, donc JS attend une valeur : `{}` redevient un objet, et on retombe sur `"[object Object]" + ""`                                                    |
| `true + true`            | `2`                 | Aucune chaîne en jeu, donc `+` additionne : les booléens deviennent des nombres (`true` → `1`)                                                                                                                  |
| `"b" + "a" + +"a" + "a"` | `"baNaNa"`          | Le `+` collé à `"a"` est le `+` **unaire** (1.5) : `+"a"` tente de convertir `"a"` en nombre → `NaN`. Ensuite : `"ba" + NaN` → `"baNaN"`, puis `+ "a"` 🍌                                                        |

## 2.2 L'égalité faible `==`

> Rappels : 1.3, 1.4, 1.6

| Code                                    | Résultat                            | Pourquoi                                                                                                                                                                                     |
| --------------------------------------- | ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0 == "0"`                              | `true`                              | Nombre contre chaîne : la chaîne est convertie en nombre → `0 == 0`                                                                                                                          |
| `0 == ""`                               | `true`                              | Même règle : `""` converti en nombre vaut `0`                                                                                                                                                |
| `"0" == ""`                             | `false` 😱                           | Deux chaînes : **aucune conversion**, on compare le texte, et `"0"` ≠ `""`. Donc `==` n'est pas transitif                                                                                    |
| `[] == 0`                               | `true`                              | Objet contre nombre : le tableau est converti (1.4) : `[]` → `""` → `0`                                                                                                                      |
| `[0] == 0`                              | `true`                              | Même chose : `[0]` → `"0"` → `0`                                                                                                                                                             |
| `[0] == []`                             | `false` 😱                           | Donc `[0] == []`… non ! Deux objets : **aucune conversion**, `==` compare les **références** (même objet en mémoire ?). Ce sont deux tableaux différents. Encore un `==` non transitif       |
| `true == "true"`                        | `false` 😱                           | `==` ne compare pas le texte : il convertit **les deux côtés en nombre**. `true` → `1`, `"true"` → `NaN`, et `1 == NaN` est faux. À l'inverse, `true == "1"` → `true` (`1 == 1`)             |
| `null == true`                          | `false`                             | Rien d'étonnant : `null` n'est pas `true`…                                                                                                                                                   |
| `null == false`                         | `false` 😱                           | … mais il n'est pas `false` non plus ! `==` convertit le booléen en nombre (`false` → `0`), puis applique sa règle spéciale : `null` n'est égal qu'à `null` et `undefined` (voir ci-dessous) |
| `null ? "c'est truthy" : "c'est falsy"` | `"c'est falsy"`                     | Et pourtant `null` est bien **falsy** (1.3) ! Un `if` ou un `? :` utilise la conversion en booléen, alors que `==` suit ses propres règles. Être falsy ≠ être `== false`                     |
| `null == undefined`                     | `true`                              | Règle spéciale de `==` : `null` et `undefined` sont égaux **entre eux**, et à rien d'autre                                                                                                   |
| `null == 0`                             | `false` … mais `null >= 0` → `true` | `==` applique la règle spéciale ci-dessus : pas de conversion. Mais `>=` est un opérateur de **comparaison**, qui convertit en nombre : `null` → `0`, et `0 >= 0`                            |
| `[] == ![]`                             | `true`                              | Étape par étape : `[]` est truthy (1.3), donc `![]` → `false`. Puis `[] == false` : le booléen devient `0`, le tableau devient `""` puis `0`. Donc `0 == 0`                                  |

**Message** : toujours utiliser `===`.

## 2.3 Les nombres

> Rappel : 1.2

| Code                                     | Résultat                                   | Pourquoi                                                                                                                                                                                                                                                                                                                                  |
| ---------------------------------------- | ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `typeof NaN`                             | `"number"`                                 | La norme IEEE 754 définit `NaN` comme une **valeur numérique** qui signifie "résultat invalide" (`0 / 0`, `Math.sqrt(-1)`)                                                                                                                                                                                                                |
| `NaN === NaN`                            | `false`                                    | IEEE 754 impose que `NaN` soit différent de **tout**, y compris de lui-même : deux calculs ratés n'ont aucune raison d'être "égaux". → utiliser `Number.isNaN()`                                                                                                                                                                          |
| `0.1 + 0.2 === 0.3`                      | `false`                                    | En binaire, `0.1` s'écrit avec une infinité de chiffres (comme `1/3` = `0.333…` en décimal). Il est donc arrondi, et les arrondis s'additionnent → `0.30000000000000004`                                                                                                                                                                  |
| `9999999999999999 === 10000000000000000` | `true` 😱                                   | Un flottant n'a que 53 bits pour les chiffres. Au-delà de `Number.MAX_SAFE_INTEGER` (2⁵³ − 1), tous les entiers ne sont plus représentables : JS arrondit `9999999999999999` au plus proche, `10000000000000000`. Les deux nombres sont donc **la même valeur**, même avec `===`                                                          |
| `10n === 10`                             | `false` 😱                                  | 🪤 Piège ! Le suffixe `n` crée un **`bigint`**, un type à part (1.1) : `typeof 10n` → `"bigint"`. `===` compare aussi le type, donc c'est faux. Ironie : `10n == 10` → `true`, ici c'est `==` qui donne la réponse intuitive. Et avec des `bigint`, le problème précédent disparaît : `9999999999999999n === 10000000000000000n` → `false` |
| `Math.max()`                             | `-Infinity`                                | `Math.max` part de `-Infinity` et garde toute valeur plus grande. Sans argument, il n'a rien comparé : il renvoie sa valeur de départ                                                                                                                                                                                                     |
| `Math.min() > Math.max()`                | `true`                                     | À l'inverse, `Math.min()` part de `+Infinity`. Donc `Infinity > -Infinity`                                                                                                                                                                                                                                                                |
| `0 === -0`                               | `true` … mais `Object.is(0, -0)` → `false` | IEEE 754 stocke un **bit de signe**, donc `-0` existe (`Math.round(-0.4)` → `-0`). `===` les considère égaux par convention, `Object.is` les distingue. La différence se voit avec `1 / -0` → `-Infinity`                                                                                                                                 |

💡 Ces bizarreries ne sont **pas propres à JS** : Python, Java ou C donnent les mêmes résultats, car ils utilisent tous IEEE 754.

**Solutions** : `Number.EPSILON`, `BigInt`, `Object.is`.

## 2.4 Les tableaux

> Rappels : 1.4, 1.7

| Code                                 | Résultat              | Pourquoi                                                                                                                                                                                                                                                                         |
| ------------------------------------ | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `[10, 1, 3].sort()`                  | `[1, 10, 3]`          | Sans fonction de comparaison, `sort` convertit tout en **chaînes** et trie comme un dictionnaire : `"10"` < `"3"` car `"1"` < `"3"`. Raison : un tableau peut mélanger les types, et la chaîne est la seule conversion qui marche pour tout. → `sort((a, b) => a - b)`           |
| `["1","2","3"].map(parseInt)`        | `[1, NaN, NaN]`       | `map` appelle `fn(valeur, index)` (1.7), et `parseInt` a un 2ᵉ paramètre : la **base**. Donc `parseInt("1", 0)` → base auto → `1`, `parseInt("2", 1)` → base 1 invalide → `NaN`, `parseInt("3", 2)` → `3` n'existe pas en binaire → `NaN`                                        |
| `["10","0","2","2"].map(parseInt)`   | `[10, NaN, NaN, 2]` 😱 | Le même `"2"` donne deux résultats différents ! Tout dépend de sa **position** : `parseInt("10", 0)` → base auto → `10`, `parseInt("0", 1)` → base 1 invalide → `NaN`, `parseInt("2", 2)` → `2` n'existe pas en binaire → `NaN`, `parseInt("2", 3)` → `2` existe en base 3 → `2` |
| `[,,,].length`                       | `3`                   | La dernière virgule est une virgule finale autorisée, donc ignorée : il reste **3 cases vides** ("holes"). Ces cases ne contiennent même pas `undefined`, elles n'existent pas : `0 in [,,,]` → `false`                                                                          |
| `const a = []; a[100] = 1; a.length` | `101`                 | `length` ne compte pas les éléments : c'est le **plus grand index + 1** (1.7). Les index 0 à 99 sont des cases vides                                                                                                                                                             |
| `[1, 2, 3] + [4, 5, 6]`              | `"1,2,34,5,6"`        | Comme en 2.1 : `+` convertit les tableaux en chaînes → `"1,2,3" + "4,5,6"`                                                                                                                                                                                                       |

## 2.5 `typeof` et les types

> Rappel : 1.1

| Code                                                  | Résultat      | Pourquoi                                                                                                                                                                                                                                                            |
| ----------------------------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `typeof null`                                         | `"object"`    | Bug de la première implémentation (1995) : le type était stocké dans les premiers bits de chaque valeur, et `0` signifiait "objet". Or `null` était représenté par `0`… Jamais corrigé (don't break the web)                                                        |
| `typeof []`                                           | `"object"`    | Un tableau **est** un objet (1.1), et `typeof` ne fait pas la différence. → utiliser `Array.isArray()`                                                                                                                                                              |
| `typeof function(){}`                                 | `"function"`  | "function" n'est pas un type officiel : une fonction est un objet **appelable**. `typeof` fait un cas particulier pour pouvoir tester simplement si on peut appeler une valeur (très utile pour les callbacks). Idem : `typeof class {}` → `"function"`             |
| `const document2 = { all: [] }; typeof document2.all` | `"object"`    | Rien d'anormal : `all` est un tableau, donc un objet (1.1). Cet exemple sert de **point de comparaison** pour le suivant…                                                                                                                                           |
| `typeof document.all`                                 | `"undefined"` | (Bonus navigateur) Les vieux sites testaient `if (document.all)` pour détecter Internet Explorer. Les autres navigateurs voulaient supporter `document.all` **sans** être pris pour IE : ils l'ont rendu falsy et "undefined". C'est le seul objet falsy du langage |
| `typeof document.anchors`                             | `"object"`    | Pour comparer : `document.anchors` renvoie lui aussi une liste d'éléments de la page (une `HTMLCollection`, comme `document.all`), et il est tout aussi ancien. Mais il se comporte normalement : l'exception ne concerne **que** `document.all`                    |

## 2.6 `this` et le hoisting

> Rappel : 1.8

- **Perte de `this`** : `const f = obj.method; f()` → `this` n'est plus `obj`.
  - Pourquoi : `this` n'est pas fixé à la création de la fonction, mais **au moment de l'appel**. `this` est l'objet écrit avant le point. `obj.method()` → `obj`. `f()` → pas de point, donc `this` vaut `undefined` (ou l'objet global hors mode strict).
- **Les fonctions fléchées n'ont pas leur propre `this`**.
  - Pourquoi : elles ont été ajoutées (ES6, 2015) justement pour régler le problème ci-dessus. Elles reprennent le `this` de l'endroit où elles sont écrites, ce qui est pratique dans les callbacks.
- **`var` utilisable avant sa déclaration** (vaut `undefined`), alors que `let` lève une erreur.
  - Pourquoi : avant d'exécuter le code, JS repère toutes les déclarations (**hoisting**). Une `var` est créée et initialisée à `undefined` dès le début. Un `let` est aussi repéré, mais reste inutilisable jusqu'à sa ligne (la "Temporal Dead Zone"), pour éviter ce piège.
- **Fonctions appelables avant leur déclaration**.
  - Pourquoi : une déclaration `function f() {}` est remontée **avec son contenu**, contrairement à une `var` qui ne remonte que son nom.

## 2.7 Syntaxe piégeuse

- **ASI** (insertion automatique de `;`) :
  ```js
  function f() {
    return
    { ok: true }
  }
  f() // undefined
  ```
  - Pourquoi : JS ajoute un `;` automatiquement quand une ligne se termine sur `return`. Le code devient `return;`, et le `{ ok: true }` en dessous n'est jamais atteint (il est lu comme un bloc contenant un label `ok:`, voir plus bas). → toujours ouvrir l'accolade sur la même ligne que `return`.
- **Réponse à l'accroche** : `{} + []` en début de ligne → `0`.
  - Pourquoi : en début d'instruction, `{` ouvre un **bloc de code** (comme après un `if`), pas un objet. Il reste donc `+[]` : le `+` unaire convertit `[]` en `""`, puis en `0`. Dans `const x = {} + []`, le `{}` est bien un objet et on retrouve `"[object Object]"`.
- **Labels** : `https://google.com` est du JS valide.
  - Pourquoi : `https:` est un **label** (un nom suivi de `:`, qui sert à nommer une boucle pour `break`/`continue`), et `//google.com` est un **commentaire**.
- **`010` → `8`** mais `"010" * 1` → `10`.
  - Pourquoi : héritage du C, un nombre qui commence par `0` est lu en **octal** (base 8). Mais la conversion d'une chaîne en nombre se fait toujours en base 10. Pire : `019` → `19`, car `9` n'existe pas en octal, donc JS repasse en décimal. Interdit en mode strict ; la syntaxe moderne est `0o10`.

## 2.8 Bonus : le chaos total

- **JSFuck** : écrire n'importe quel programme avec seulement `[]()!+`.
  - `(![] + [])[+[]]` → `"f"`
  - Pourquoi : `![]` → `false` ; `false + []` → `"false"` (conversion en chaîne) ; `+[]` → `0`. Donc `"false"[0]` → `"f"`. On peut obtenir chaque lettre de cette façon.
- **Mois de `Date` indexés à partir de 0**, jours à partir de 1 : `new Date(2026, 0, 1)` est le 1ᵉʳ janvier.
  - Pourquoi : `Date` a été copié de Java (`java.util.Date`), qui l'avait copié du C. Là-bas, le mois servait d'index dans un tableau de noms (`["Janvier", "Février", …]`), donc il commençait à 0.
- **`new Date(2026, 0, 31)` puis `setMonth(1)` → 3 mars**.
  - Pourquoi : on demande le 31 février, qui n'existe pas. `Date` ne lève pas d'erreur : il **déborde** sur le mois suivant. 31 février = 28 février + 3 jours → 3 mars.

---

## Conclusion : comment s'en protéger ?

- **`===` au lieu de `==`** : aucune conversion cachée, des types différents donnent `false`.
- **Conversions explicites** (`Number(x)`, `String(x)`, `parseInt(x, 10)`) : on voit la conversion dans le code, JS ne devine plus à notre place.
- **Les fonctions "corrigées"** (`Number.isNaN`, `Array.isArray`, `Object.is`, elles ont été ajoutées à côté des anciennes, sans leurs pièges.
- **`const` / `let` au lieu de `var`** : portée de bloc, et une erreur si on utilise la variable trop tôt.
- **Mode strict** (`"use strict"` ou modules ES) : les erreurs silencieuses deviennent des erreurs visibles (`010`, variables non déclarées, `this` perdu).
- **TypeScript** : il détecte les incohérences de types **avant l'exécution** (`"22" - 2`, `0 == "0"`, `10n === 10`…). Mais il ne voit pas tout (`sort()`, `map(parseInt)`, `0.1 + 0.2`).
- **ESLint** : il signale les pratiques risquées dans l'éditeur (`eqeqeq`, `radix`, `no-var`, `use-isnan`…).

⚠️ `===` ne règle pas tout : `NaN === NaN` et `[] === []` restent `false`.

JS n'est pas "cassé" : tout est **spécifié** (ECMAScript) et s'explique. Une fois les règles connues, les bizarreries deviennent… presque logiques.

### Réponse à la deuxième accroche : `"fail"` 🎉

Avec tout ce qu'on a vu, on peut maintenant le décoder. Trois briques suffisent :

| Brique     | Valeur    | Pourquoi                                                         |
| ---------- | --------- | ---------------------------------------------------------------- |
| `+[]`      | `0`       | `+` unaire : `[]` → `""` → `0` (1.5)                             |
| `![]`      | `false`   | `[]` est truthy, donc `![]` → `false` (1.3)                      |
| `![] + []` | `"false"` | `+` avec un objet → conversion en chaînes → `"false" + ""` (2.1) |

On en déduit les nombres : `+!+[]` → `+!0` → `+true` → `1`, et `!+[] + !+[]` → `true + true` → `2`.

Lettre par lettre :

| Morceau                           | Calcul                                                                                                                                                                                                                     | Lettre |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| `(![] + [])[+[]]`                 | `"false"[0]`                                                                                                                                                                                                               | `"f"`  |
| `(![] + [])[+!+[]]`               | `"false"[1]`                                                                                                                                                                                                               | `"a"`  |
| `([![]] + [][[]])[+!+[] + [+[]]]` | `[![]]` → `[false]` ; `[][[]]` → `[][""]` → `undefined` (propriété inexistante) ; `[false] + undefined` → `"falseundefined"`. L'index : `1 + [0]` → `"1" + "0"` → `"10"` (concaténation, 2.1). Donc `"falseundefined"[10]` | `"i"`  |
| `(![] + [])[!+[] + !+[]]`         | `"false"[2]`                                                                                                                                                                                                               | `"l"`  |

`"f" + "a" + "i" + "l"` → **`"fail"`**.


Voir https://jsfuck.com/ :

Avec seulement `[]()!+`, on peut écrire n'importe quel programme.

```js
// Exec de fin
[][(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+([]+[])[(![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]()[+!+[]+[!+[]+!+[]]]+((!![]+[])[+[]]+[+!+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+([][[]]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+([][[]]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]])[(![]+[])[!+[]+!+[]+!+[]]+(+(!+[]+!+[]+[+!+[]]+[+!+[]]))[(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([]+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]][([][[]]+[])[+!+[]]+(![]+[])[+!+[]]+((+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]]](!+[]+!+[]+!+[]+[+!+[]])[+!+[]]+(![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]]((!![]+[])[+[]])[([][(!![]+[])[!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([![]]+[][[]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]](([][(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+(![]+[+[]])[([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()[+!+[]+[+[]]]+![]+(![]+[+[]])[([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()[+!+[]+[+[]]])()[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[+[]])[([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()[+!+[]+[+[]]])+[])[+!+[]])+([]+[])[(![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]()[+!+[]+[!+[]+!+[]]])()
```

---

## Ressources

- [wtfjs](https://github.com/denysdovhan/wtfjs)
- Talk "Wat" — Gary Bernhardt (2012)
- [Spécification ECMAScript](https://tc39.es/ecma262/)
- [jsisweird](https://jsisweird.com/)
