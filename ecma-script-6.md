# ECMAScript 6

## Abstract

The purpose of this guide is to present a curated selection of ECMAScript 6
features in a clear yet concise way.

Most of these features can already be tried out on almost any popular browser
(or [Node](https://nodejs.org/)), as long as you use a recent version.
Alternatively, you can use [Babel](https://babeljs.io/).

## Table of contents

* [Block scope](#block-scope)
* [Template strings](#template-strings)
* [Arrow functions](#arrow-functions)
* [Symbols](#symbols)
* [Iterables](#iterables)
* [Generators](#generators)
* [Arrow functions](#arrow-functions)
* [Classes](#classes)
* [Promises](#promises)
* [Object literal enhancements](#object-literal-enhancements)
* [Default parameters](#default-parameters)
* [Arrow functions](#arrow-functions)
* [Spread operator](#spread-operator)
* [Destructuring](#destructuring)
* [Sets and maps](#sets-and-maps)
* [Modules](#modules)

## Block scope

It's now possible to have block scoped variables by using `let` instead of
`var`:

```
if (true) {
  let x = 7;
  console.log(x); // 7
}
console.log(x); // Error
```

You can also define block scoped variables with `const` and prevent them from
multiple assignments:

```
const x = 7;
x = 8; // Error
```

`const` variables must be initialiazed:

```
const x; // Error
```

Remember that assigning an object to a `const` variable doesn't magically make
it immutable:

```
const marco = { name: 'Marco' };
marco.name = 'Mario'; // Oops
```

**Note:** `let` and `const` are immune to
[hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting=).

## Template strings

The character \` is a new delimiter for string literals:

```
const name = `Marco`;
```

This kind of string literal allows multi-line strings without appending `\n`:

```
const message = `I see trees of green, red roses too
I see them bloom for me and you
And I think to myself what a wonderful world`;
```

You can even interpolate values inside the string:

```
const name = 'Marco';
const age = 30;
const message = `Hi I'm ${name} and I'm ${age}`;
console.log(message); 'Hi I'm Marco and I'm 30'
```

## Arrow functions

It's a simplified syntax for anonymous functions that comes in many variants:

```
[ 1, 2, 3 ].map(x => 2 * x); // [ 2, 4, 6 ]
[ 1, 2, 3 ].map((x, i) => i + ': ' + x); // [ '0: 1', '1: 2', '2: 3' ]
[ 1, 2, 3 ].map((x, i) => {
  return i + ': ' + x;
}); // [ '0: 1', '1: 2', '2: 3' ]
...
```

In short:

* if you have more than one parameter, use round brackets
* if you have no parameters, use empty round brackets
* if you have multiple statements, use curly brackets
* if you have only statement, `return` is implied

Note that arrow functions capture `this`, treating it like any variable declared
with `var`. So, no neeed for tricks like `var self = this;`.

**Note:** The same goes for the special variable `arguments`.

## Symbols

It's a new global function that allows to create values that:

* are unique (`Symbol() !== Symbol()`)
* have type `symbol` (which is now a new JavaScript primitive type)
* can be used as keys for object properties
* can have a descriptive string (`Symbol('foo')`), but this doesn't affect
uniqueness (`Symbol('foo') !== Symbol('foo')`)

The goal is to avoid undesired overwriting of object properties:

```
const name = Symbol('name');
const marco = {
  [name]: 'Marco'
};
marco[name] = 'Mario'; // OK
marco.name = 'Mirko'; // OK (but it's not an overwrite)
console.log(marco[name]); // 'Mario'
const name2 = Symbol('name');
marco[name2] = 'Mark'; // OK (but it's not an overwrite)
console.log(marco[name]); // 'Mario'
```

## Iterables

You can now iterate over an object following a custom iteration logic.
Specifically:

* the object needs to have a property with a `Symbol.iterator` key, which is a
predefined global symbol
* the value of this property should be a function
* this function should return an object with a `next` method
* this method should return objects with a `value` and a `next` property in
order to provide the next value and signal iteration ending

When these conditions are met, you can use a brand new `for` syntax that the
JavaScript interpreter implements by silently calling the `next` method for you:

```
const randomNumbers = {
  [Symbol.iterator]: function () {
    return {
      next: function () {
        return {
          value: Math.random(),
          done: Math.random() > 0.9
        }
      }
    };
  }
};
for (let x of randomNumbers) {
  console.log(x);
}
```

Some terminology:

* objects that are iterable in this fashion are called "iterables"
* the object with the `next` method is called an "iterator"

## Generators

Generators (or generator functions) are functions that:

* have a `*` after the `function` keyword
* can temporarily suspend their execution by using the `yield` keyword
* when suspending, they can emit a value (using `yield` just like `return`)
* when they are invoked, they actually **don't get executed**, and an iterable
is returned instead
* this iterable provides access to the values that the generator function emits
* calls to the `next` method of the iterator start / resume the execution of the
function

Generator functions are the ideal way of implementing iterator factories:

```
const randomNumbers = {
  [Symbol.iterator]: function* () {
    while (Math.random() < 0.9) yield Math.random();
  }
};
for (let x of randomNumbers) {
  console.log(x);
}
```

## Classes

ECMAScript 6 introduces the `class` keyword which enables easier prototype
definition through a syntax that looks like class definitions in class-based
object-oriented languages:

```
class Person {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log(`Hi I'm ${this.name}`);
  }
}

const marco = new Person('Marco');
marco.sayHi(); // 'Hi I'm Marco'
console.log(marco instanceof Person); // true
Person.prototype.saySomething = () => console.log('Something!');
marco.saySomething(); // 'Something!'
```

### Getters and setters

While in other languages getters and setters provide controlled access to
otherwise private members (encapsulation), ECMAScript 6 getters and setters are
simply a way to customize property reads / writes:

```
class Person {
  get name() {
    console.log('Reading name');
    return this._name;
  }
  set name(n) {
    console.log('Writing name');
    this._name = n;
  }
}

const marco = new Person();
marco.name = 'Marco'; // 'Writing name'
console.log(marco.name); // 'Reading name', 'Marco'
```

**Note:** Prefixing variables with an underscore doesn't magically make them
private. It's just a convention to tell that they shouldn't be accessed
directly. Remember: JavaScript object properties are always public.
Encapsulation can't be guaranteed.

### Static methods

Example:

```
class PeopleHelper {
  static sameName(p1, p2) {
    return p1.name === p2.name;
  }
}

const marco = new Person('Marco');
const fabio = new Person('Fabio');
PeopleHelper.sameName(marco, fabio); // false
```

### Inheritance and overriding

Example:

```
class Person {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log(`Hi I'm ${this.name}`);
  }
}

// Inheritance
class Athlete extends Person {
  constructor(name, sport) {
    super(name);
    this.sport = sport;
  }
  // Overriding
  sayHi() {
    console.log(`Hi I'm ${this.name} and I play ${this.sport}`);
  }
}

const marco = new Athlete('Marco', 'soccer');
marco.sayHi(); // 'Hi I'm Marco and I play soccer'
console.log(marco instanceof Person); // true
console.log(marco instanceof Athlete); // true
```

## Promises

Promises are basically a convenient way of dealing with asynchronous tasks.

**Note:** I won't discuss promises here, since it's a topic that should be
treated individually.

ECMAScript 6 brings native support for promises through the `Promise`
constructor:

```
// Creating promises
function myAsyncTask() {
  // Will randomly fail / succeed 2 seconds from now
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (Math.random() > 0.5) resolve();
      else reject();
    }, 2000);
  });
}

// Consuming promises
myAsyncTask().then(() => {
  console.log('Task completed successfully');
}, () => {
  console.log('Failure');
});
```

## Object literal enhancements

You can now avoid to repeat the same identifier when using a variable as a value
for a property:
```
const name = 'Marco';
const marco = { name }; // instead of { name: name }
```

It's available a new syntax for methods:

```
const name = 'Marco';
const marco = {
  name,
  sayHi() {
    console.log(`Hi I'm ${this.name}`);
  }
};
```

Property names may be dynamically assigned:

```
const name = 'Marco';
const property = 'name';
const marco = {
  [property]: name
};
```

## Default parameters

```
function sayHi(name = 'John Doe') {
  console.log(`Hi I'm ${this.name}`);
}
sayHi(); 'Hi I'm John Doe';
sayHi('Marco'); 'Hi I'm Marco';
```

You can have multiple default parameters, and you can even mix them with regular
parameters:

```
function sumWithDefaultParameters(a, b = 1, c, d = 2) {
  console.log(a, b, c, d);
  return a + b + c + d;
}
sumWithDefaultParameters(1); // NaN (1 + 1 + undefined + 2)
sumWithDefaultParameters(1, 2); // NaN (1 + 2 + undefined + 2)
sumWithDefaultParameters(1, undefined, 1); // 5 (1 + 1 + 1 + 2)
sumWithDefaultParameters(1, undefined, 1, 10); // 13 (1 + 1 + 1 + 10)
```

## Spread operator

The spread operator `...` turns an array into a sequence of values:

```
let x = [ 3, 4 ];
let y = [ 7, 8 ];
console.log([ 1, 2, ...x, 5, 6, ...y ]); // [ 1, 2, 3, 4, 5, 6, 7, 8 ]
```

When applied to a string, the output is an array of individual characters:

```
console.log([ ...'hello' ]); // [ 'h', 'e', 'l', 'l', 'o' ]
```

Finally, when a function has N parameters and you apply the spread operator to
the last one, it will store all the arguments after the N-th one (included):

```
function example(first, second, ...etc) {
  console.log(etc);
}
example(1, 2, 3, 4); // [ 3, 4 ]
```

## Destructuring

It's a feature that allows to extract pieces of an object or array:

```
const { a, b } = { a: 'x', b: 'y', c: 'z' };
console.log(a); // 'x'
console.log(b); // 'y'

const [ , a, , , b, ] = [ 4, 8, 15, 16, 23, 42 ];
console.log(a); // 8
console.log(b); // 23
```

You can provide default values:

```
const [ a, b = 2 ] = [ 1 ];
console.log(a); // 1
console.log(b); // 2
```

It can be used in function signatures to automatically extract pieces of an
argument:

```
function sayHi({ name }) {
  console.log(`Hi I'm ${this.name}`);
}
sayHi({ name: 'Marco' }); // Hi I'm Marco
```

It's even possible to extract nested properties:

```
const { address: { city } } = {
  name: 'Marco',
  address: {
    city: 'Rome'
  }
};
console.log(city); // 'Rome'
```

...and you can choose a custom name:

```
const { address: { city: cityOfMarco } } = {
  name: 'Marco',
  address: {
    city: 'Rome'
  }
};
console.log(cityOfMarco); // 'Rome'
```

## Sets and maps

New constructors for sets and maps:

```
const stuff = new Set();
stuff.add('foo');
stuff.add('foo');
stuff.add('bar');
stuff.add('bar');
stuff.add('baz');
stuff.add('baz');
console.log(stuff.size); // 3
for (let thing of stuff) console.log(thing);

const presidents = new Map();
presidents.set('usa', 'Trump');
presidents.set('france', 'Macron');
for (let entry of presidents) {
  const country = entry[0];
  const president = entry[1];
  // ...
}
```

There are also `WeakSet` e `WeakMap` variants. Elements of a weak set can be
garbage collected even when still referenced, as long as the only references
are the ones from the set itself. The same applies to keys of a weak map.

## Modules

You can separate your code into different modules with private definitions that
must be shared explicitly with the `import` ed `export` keywords.

Export example:

```
// File 'my-module.js'
export const importantValue = 42;
```

Import example:

```
// File 'other-module.js' (same folder)
import { importantValue } from './my-module.js';

console.log(importantValue); // 42
```

Import example (alternative syntax):

```
// File 'other-module.js' (same folder)
import * as myModule from './my-module.js';

console.log(myModule.importantValue); // 42
```

**Note:** Use this syntax when you need to import a lot of definitions.

**Note:** There's a lot of additional syntax for modules but this is enough for
now ;)
