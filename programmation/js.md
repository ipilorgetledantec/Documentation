# JavaScript — Fiche de référence complète

---

## Types de données

| Type | Exemple |
|---|---|
| `string` | `"bonjour"`, `'hello'`, `` `world` `` |
| `number` | `42`, `3.14`, `NaN`, `Infinity` |
| `bigint` | `9007199254740991n` |
| `boolean` | `true`, `false` |
| `undefined` | variable déclarée mais non initialisée |
| `null` | absence intentionnelle de valeur |
| `symbol` | `Symbol('id')` — identifiant unique |
| `object` | `{}`, `[]`, `null` (historiquement) |

---

## Déclaration de variables

```js
var x = 1;    // portée fonction, hissée (éviter)
let y = 2;    // portée bloc, modifiable
const z = 3;  // portée bloc, non réassignable
```

> **Règle :** toujours préférer `const`, puis `let`. Éviter `var`.

---

## Opérateurs

### Arithmétiques
```js
+  -  *  /  %  **   // addition, soustraction, *, division, modulo, puissance
```

### Comparaison
```js
===   !==   >   <   >=   <=
// === et !== comparent valeur ET type (toujours utiliser ===)
```

### Logiques
```js
&&   ||   !   ??   // ET, OU, NON, nullish coalescing
```

### Affectation
```js
=   +=   -=   *=   /=   **=   ??=   &&=   ||=
```

### Optionnel
```js
obj?.prop         // accès optionnel
obj?.method?.()   // appel optionnel
arr?.[0]          // index optionnel
```

---

## Structures de contrôle

### Conditions
```js
if (condition) { ... }
else if (autre) { ... }
else { ... }

// Ternaire
const val = condition ? 'oui' : 'non';

// Switch
switch (x) {
  case 1: ...; break;
  case 2: ...; break;
  default: ...;
}
```

### Boucles
```js
for (let i = 0; i < 10; i++) { ... }

while (condition) { ... }

do { ... } while (condition);

for (const val of tableau) { ... }   // itérable
for (const key in objet) { ... }     // clés d'objet

// Contrôle
break;    // sortir de la boucle
continue; // passer à l'itération suivante
```

---

## Fonctions

```js
// Déclaration (hissée)
function addition(a, b) { return a + b; }

// Expression
const add = function(a, b) { return a + b; };

// Fléchée
const add = (a, b) => a + b;
const double = x => x * 2;
const saluer = () => 'Bonjour';

// Paramètres par défaut
function saluer(nom = 'monde') { return `Bonjour ${nom}`; }

// Rest params
function somme(...nums) { return nums.reduce((a, b) => a + b, 0); }

// Spread
const arr = [1, 2, 3];
console.log(Math.max(...arr)); // 3
```

---

## Tableaux (Array)

```js
const arr = [1, 2, 3];

// Accès et modification
arr[0];               // 1
arr.length;           // 3
arr.push(4);          // ajoute en fin
arr.pop();            // supprime en fin
arr.unshift(0);       // ajoute en début
arr.shift();          // supprime en début

// Itération
arr.forEach(x => console.log(x));

// Transformation (retournent un nouveau tableau)
arr.map(x => x * 2);                    // [2, 4, 6]
arr.filter(x => x > 1);                 // [2, 3]
arr.reduce((acc, x) => acc + x, 0);     // 6
arr.find(x => x > 1);                   // 2
arr.findIndex(x => x > 1);              // 1
arr.some(x => x > 2);                   // true
arr.every(x => x > 0);                  // true
arr.flat();                              // aplatit d'un niveau
arr.flatMap(x => [x, x * 2]);           // map + flat

// Tri et recherche
arr.sort((a, b) => a - b);             // tri numérique croissant
arr.reverse();
arr.includes(2);                        // true
arr.indexOf(2);                         // 1

// Découpe
arr.slice(1, 3);                        // [2, 3] — non destructif
arr.splice(1, 1, 99);                   // modifie le tableau

// Destructuration
const [premier, deuxieme, ...reste] = arr;
```

---

## Objets

```js
const personne = {
  nom: 'Alice',
  age: 30,
  saluer() { return `Je suis ${this.nom}`; }
};

// Accès
personne.nom;
personne['nom'];

// Destructuration
const { nom, age } = personne;
const { nom: n, age: a = 25 } = personne; // alias + défaut

// Spread
const clone = { ...personne, ville: 'Paris' };

// Méthodes utiles
Object.keys(personne);     // ['nom', 'age', 'saluer']
Object.values(personne);   // ['Alice', 30, f]
Object.entries(personne);  // [['nom','Alice'], ...]
Object.assign({}, personne, { age: 31 });
Object.freeze(personne);   // rend immuable

// Raccourcis ES6
const nom = 'Bob';
const obj = { nom, age: 20 }; // { nom: 'Bob', age: 20 }
```

---

## Chaînes de caractères (String)

```js
const str = 'Bonjour monde';

str.length;
str.toUpperCase();
str.toLowerCase();
str.trim();
str.trimStart();
str.trimEnd();
str.includes('jour');        // true
str.startsWith('Bon');       // true
str.endsWith('nde');         // true
str.indexOf('o');            // 1
str.slice(0, 7);             // 'Bonjour'
str.split(' ');              // ['Bonjour', 'monde']
str.replace('monde', 'JS'); 
str.replaceAll('o', '0');
str.repeat(3);
str.padStart(20, '-');
str.padEnd(20, '-');
str.charAt(0);               // 'B'
str.at(-1);                  // 'e' (dernier caractère)

// Template literals
const msg = `Bonjour ${nom}, tu as ${age} ans`;
```

---

## Asynchrone

### Callbacks
```js
setTimeout(() => console.log('après 1s'), 1000);
```

### Promises
```js
const p = new Promise((resolve, reject) => {
  if (ok) resolve('succès');
  else reject(new Error('échec'));
});

p.then(val => console.log(val))
 .catch(err => console.error(err))
 .finally(() => console.log('terminé'));

// Combinateurs
Promise.all([p1, p2]);          // attend toutes (échoue si une échoue)
Promise.allSettled([p1, p2]);   // attend toutes (résultats même en cas d'échec)
Promise.race([p1, p2]);         // première à se résoudre
Promise.any([p1, p2]);          // première réussie
```

### Async / Await
```js
async function charger() {
  try {
    const reponse = await fetch('https://api.example.com/data');
    const data = await reponse.json();
    return data;
  } catch (err) {
    console.error(err);
  }
}
```

---

## Gestion des erreurs

```js
try {
  risque();
} catch (err) {
  console.error(err.message);
  console.error(err.stack);
} finally {
  nettoyage();
}

// Créer une erreur custom
class ErreurValidation extends Error {
  constructor(message) {
    super(message);
    this.name = 'ErreurValidation';
  }
}

throw new ErreurValidation('Valeur invalide');
```

---

## Classes

```js
class Animal {
  #nom; // propriété privée

  constructor(nom) {
    this.#nom = nom;
  }

  parler() {
    return `${this.#nom} fait un bruit`;
  }

  get nom() { return this.#nom; }
  set nom(val) { this.#nom = val; }

  static creer(nom) { return new Animal(nom); }
}

class Chien extends Animal {
  constructor(nom) {
    super(nom);
  }

  parler() {
    return `${super.parler()} — Ouaf!`;
  }
}

const rex = new Chien('Rex');
rex instanceof Chien; // true
rex instanceof Animal; // true
```

---

## Modules (ES Modules)

```js
// export nommé
export const PI = 3.14;
export function addition(a, b) { return a + b; }

// export par défaut
export default class MaClasse { ... }

// import
import MaClasse from './ma-classe.js';
import { PI, addition } from './math.js';
import { addition as add } from './math.js';
import * as Math from './math.js';

// import dynamique
const module = await import('./heavy-module.js');
```

---

## Méthodes utiles globales

```js
parseInt('42px');         // 42
parseFloat('3.14abc');    // 3.14
Number('42');             // 42
String(42);               // '42'
Boolean(0);               // false
isNaN(NaN);               // true
isFinite(Infinity);       // false
typeof 'hello';           // 'string'
typeof null;              // 'object' (bug historique)
Array.isArray([]);        // true
JSON.stringify(obj);
JSON.parse(json);
```

---

## Symboles et itérables

```js
// Symbol
const id = Symbol('id');
const obj = { [id]: 123 };

// Itérable custom
const plage = {
  [Symbol.iterator]() {
    let i = 0;
    return {
      next() {
        return i < 5
          ? { value: i++, done: false }
          : { done: true };
      }
    };
  }
};

for (const n of plage) console.log(n); // 0 1 2 3 4
```

---

## Trucs et pièges courants

```js
// Coercition de type
'5' + 3;      // '53' (string)
'5' - 3;      // 2 (number)
null == undefined;   // true
null === undefined;  // false

// Valeurs falsy
false, 0, '', null, undefined, NaN

// Nullish vs falsy
const a = val ?? 'défaut';  // seulement si null ou undefined
const b = val || 'défaut';  // si falsy (0, '' inclus)

// Copie superficielle vs profonde
const copie = { ...obj };           // superficielle
const profonde = structuredClone(obj); // profonde (moderne)

// This dans les arrow functions
// Les arrow functions n'ont pas leur propre `this`
const obj = {
  val: 42,
  fléchée: () => this.val,    // undefined (this = global)
  normale() { return this.val; } // 42
};
```

---

## Nouveautés récentes (ES2020+)

```js
// Nullish coalescing (??)
const val = user?.address?.city ?? 'Inconnue';

// Optional chaining (?.)
user?.profile?.avatar?.url;

// BigInt
const grand = 9007199254740991n + 1n;

// Promise.allSettled
// structuredClone (deep copy)
// Array.at(-1) — dernier élément
// Object.hasOwn(obj, 'prop')
// Array.from({ length: 5 }, (_, i) => i)
// String replaceAll
// Logical assignment (&&=, ||=, ??=)
```