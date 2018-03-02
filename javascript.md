# JavaScript

## Abstract

If you're searching for a programming language to learn, JavaScript is probably
the best option: it's easy to get started (you just need a web browser) and is
a very useful language to learn since it allows to develop web apps, dekstop
apps ([Node.js](https://nodejs.org), [Electron](https://electronjs.org/)),
mobile apps ([Ionic](https://ionicframework.com/),
[React Native](https://facebook.github.io/react-native/)) **by using a single
programming language**.

This guide briefly explains JavaScript fundamentals with some examples. Prior
programming knowledge is assumed.

**Tip:** Use
[your browser's console](https://www.google.it/search?q=how+to+open+browser+console)
to try the examples in this guide.

## Table of contents

* [Variables](#variables)
* [Arrays](#arrays)
* [Conditionals](#conditionals)
* [Loops](#loops)
* [Functions](#functions)
* [Objects](#objects)
* [Prototypes](#prototypes)

## Variables

In JavaScript variables are declared with `var`. No need to specify a type:

```
var x;
```

Values, though, do have type. The simplest ones are numbers, strings and
booleans:

```
var n = 7;
var s = 'hello';
var b = true;

typeof n; // 'number'
typeof s; // 'string'
typeof b; // 'boolean'
```

**Note:** There's no distinction between integers and floats.

**Note:** Comments are inserted with `//` or `/*` and `*/`.

You can use `null` to represent non-valid values. There's also a special
`undefined` value which is automatically given to non-initialized variables:

```
var x;
console.log(x); // undefined
```

**Note:** `console.log` prints values to the console.

## Arrays

JavaScript has a very simple syntax for array literals:

```
var numeri = [ 1, 2, 3 ];
```

Unlike other languages, JavaScript allows arrays to contain values of different
types:

```
var chaos = [ 1, 'hello', false, null ];
```

This is how you access individual array elements:

```
var chars = [ 'a', 'b', 'c' ];
console.log(chars[0]); // 'a'
console.log(chars[1]); // 'b'
console.log(chars[2]); // 'c'
chars[0] = 'z';
console.log(chars); // [ 'z', 'b', 'c' ]
```

JavaScript arrays have type `object` and can have properties. The most useful
one is `length`:

```
[ 'a', 'b', 'c' ].length; // 3
```

## Conditionals

Same syntax as C and many other languages:

```
if (x > 5) {
  console.log('x is bigger than 5');
} else {
  console.log('x is lower than or equal to 5');
}
```

In boolean expressions remember that JavaScript has sloppy (`==`) and strict
(`===`) equal tests:

```
console.log('5' == 5); // true
console.log('5' === 5); // false
```

**Note:** There are also `!=` and `!==` operator with similar semantics.

**Note:** Use `==` and `!=` with caution. There's risk in propagating values
with unpredictable type.

In JavaScript `null`, `undefined` and empty strings are equivalent to `false`,
so conditionals are often written like this:

```
if (x) {
  console.log('x has no value');
} else {
  console.log('x is: ' + x);
}
```

## Loops

Same syntax as many other languages:

```
// Counting from 1 to 10
for (var i = 0; i < 10; i++) {
  console.log(i + 1);
}
```

## Functions

A JavaScript function is a parametrized sequence of instructions:

```
function sum(a, b) {
  return a + b;
}
var x = sum(7, 3);
console.log(x); // 10
```

**Note:** Just like variables, parameters have no type.

JavaScript functions are treated as values so you can assign them to variables:

```
var aFunction = function (x) {
  console.log(x);
};
aFunction('Hi'); // 'Hi'
```

This example also shows another JavaScript feature: anonymous functions.

Anonymous functions make sense when immediately assigned or invoked:

```
(function (x) {
  console.log(x);
})('Hi'); // 'Hi'
```

Immediately invoked functions may seem useless, for instance the previous
example may also be written like this:

```
var x = 'Hi';
console.log(x); // 'Hi'
```

In reality immediately invoked functions allow to preserve the current scope:

```
function example() {
  var n = 7;
  console.log(n);
}
esempio(); // 7
console.log(n); // Error (not existent here)
```

Scope will be treated in detail [here](#scope).

JavaScript functions may be invoked with an arbitrary number of arguments:
missing arguments will be `undefined`, while exceeding arguments will simply be
ignored.

```
function example(x1, x2) {
  if (x2 === undefined) {
    console.log(x1);
  } else {
    console.log([ x1, x2 ]);
  }
}
example(7); // 7
example(7, 42); // [ 7, 42 ]
example(7, 42, 666); // [ 7, 42 ]
```

You can also use the keyword `arguments` which represents current arguments as
an array:

```
function example() {
  console.log(arguments[0]);
  console.log(arguments[1]);
  console.log(arguments[2]);
  console.log(arguments.length);
}
example('a', 'b'); // 'a', 'b', undefined, 2
```

This makes it easy to implement function with a variable number of arguments.

## Scope

JavaScript variables have a scope that corresponds to the function in which are
declared and includes nested functions:

```
function externalFunction() {
  var n = 7;
  function aNestedFunction() {
    // n accessible here
    function anEvenDeeperFunction() {
      // n accessible here too
    }
  }
}
```

In other words, every function "captures" the execution context in which is
declared. In JavaScript we call "closure" a function with its execution context.

Closures allow to preserve an execution context even after a function returns:

```
function example(x) {
  return function () {
    return x;
  }
}
var f = example(7);
f(); // 7
```

Here `f` preserves `example`'s execution context even after `example` returns.

This behavior is very powerful, but keep in mind memory usage.

JavaScript variables are subjet to "hoisting": when code is executed it doesn't
matter where a declaration actually happens, declarations will always be
executed first. For instance this code:

```
function example() {
  console.log('hi');
  var n = 7;
}
```

Will be executed as if it was written like this:

```
function example() {
  var n;
  console.log('hi');
  n = 7;
}
```

This sometimes causes subtle bugs, so many JavaScript developers try to always
write declarations at the beginning of a function in order to be consistent with
actual execution.

## Objects

JavaScript objects are basically sets of name-value pairs called "properties":

```
var marco = {
  name: 'Marco',
  age: 30
};
console.log(marco.name); // 'Marco'
console.log(marco.age); // 30
```

Here we have an object with two properties called `name` and `age` with values
`'Marco'` and `30`. The `.` operator is used to access property values given
their names. Alternatively, you can use this syntax:

```
console.log(marco['name']); // 'Marco'
console.log(marco['age']); // 30
```

This syntax have 2 benefits. First, it allows dynamic access:

```
var whichProperty = 'name';
console.log(marco[whichProperty]); // 'Marco'
```

Second, it allows using names otherwise forbidden such as names with special
characters or language reserved identifiers:

```
var weirdObject = {
  'property with a weird name': 42
};
console.log(weirdObject['property with a weird name']); // 42
```

Since JavaScript functions are values, they can be assigned as property values
and used like methods:

```
var marco = {
  saySomething: function (what) {
    console.log(what);
  }
};
marco.saySomething('hi'); // 'hi'
```

The kayword `this` can be used within a method to access other properties of
the same object:

```
var marco = {
  name: 'Marco',
  sayHi: function () {
    console.log('Hi, my name is ' + this.name);
  }
};
marco.sayHi(); // 'Hi, my name is Marco'
```

Objects are always "editable": you can add, delete or change values of
properties, even when they are methods.

```
var marco = {}; // Let's start with an empty object
marco.name = 'Marco'; // We add a property
// We add a method
marco.sayHi = function () {
  console.log('Hi, my name is ' + this.name);
};
// We add, modify and finally delete a property
marco.age = 30;
marco.age = 31;
delete marco.age;
```

JavaScript object properties are easily iterable:

```
var marco = { name: 'Marco', age: 30 };
for (var propertyName in marco) {
  console.log(marco[propertyName]);
} // 'Marco', 30
```

## Prototypes

JavaScript functions may be used as object constructors:

```
function Person(name) {
  this.name = name;
  this.sayHi = function () {
    console.log('Hi, my name is ' + this.name);
  };
}
var marco = new Person('Marco');
marco.sayHi(); // 'Hi, my name is Marco'
```

This has some benefits. First, objects created with a constructor share a
common, extensible "prototype" from which they automatically inherit properties:

```
function Person(name) {
  this.name = name;
}
var marco = new Person('Marco');
var fabio = new Person('Fabio');
Person.prototype.sayHi = function () {
  console.log('Hi, my name is' + this.nome);
};
marco.sayHi(); // 'Hi, my name isMarco'
fabio.sayHi(); // 'Hi, my name isFabio'
```

Inherited properties may be overridden:

```
function Person(name) {
  this.name = name;
  this.sayHi = function () {
    console.log('Hi, my name is ' + this.name);
  };
}
var marco = new Person('Marco');
var fabio = new Person('Fabio');
fabio.sayHi = function () {
  console.log('Yo! I\'m ' + this.nome);
};
marco.sayHi(); // 'Hi, my name is Marco'
fabio.sayHi(); // 'Yo! I'm Fabio'
```

Another benefit of constructors is that they allow `instanceof` tests:

```
function Person(name) { ... }
var marco = new Person('Marco');
marco instanceof Persona; // true
var cloneOfMarco = { name: 'Marco' };
cloneOfMarco instanceof Persona; // false
```

This concludes our JavaScript overview. Other JavaScript guides may be added in
future, so stay tuned :D
