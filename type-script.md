Error# TypeScript

## Abstract

TypeScript is a language designed by Microsoft in order to provide JavaScript
developers with some automatic error checking (and code suggestions) at
implementation time, without the need for running the code (static analysis).
These features are built upon class definitions and type declaration for
variables. TypeScript also emulates modern JavaScript features such as those
defined in ECMAScript 6.

This guide will briefly present TypeScript (version 2), skipping some basic
concepts (interfaces, encapsulation, inheritance, polymorphism, overriding,
static members) that are familiar to every developer who has class-based
object-oriented  programming knowledge. Also, ECMAScript 6 features won't be
discussed, as they are treated [in another
guide](http://marcoliceti.xyz/en/guides/ecma-script-6.html).

TypeScript compiles (more precisely...
["transpiles"](https://en.wikipedia.org/wiki/Source-to-source_compiler)) into
regular JavaScript and that's why you should follow this guide after installing
[the official compiler](https://www.typescriptlang.org). Alternatively, you can
use an [online tool](https://www.typescriptlang.org/play/index.html). When you
will need to get more serious, try also [Visual Studio
Code](https://code.visualstudio.com/) or another IDE.

## Table of contents

* [Basic types and typed variable declarations](#basic-types-and-typed-variable-declarations)
* [Array and tuples](#array-and-tuples)
* [Functions](#functions)
* [Interfaces](#interfaces)
* [Type assertions](#type-assertions)
* [Classes](#classes)
* [Implementing interfaces](#implementing-interfaces)
* [Encapsulation](#encapsulation)
* [Static members](#static-members)
* [Inheritance and polymorphism](#inheritance-and-polymorphism)
* [Overriding](#overriding)
* [Abstract classes](#abstract-classes)
* [Generics](#generics)

## Basic types and typed variable declarations

* boolean
* number
* string
* void (can only be `null` e `undefined`)
* any (can be any value)

Examples:

```
let n: number = 7; // OK
let s: string = 'hello'; // OK
let b: boolean = true; // OK
let x: void = null; // OK
let y: any = 'thing'; // OK

n = '7'; // Error(string assigned to number)
s = 42; // Error (number assigned to string)
b = 'true'; // Error (string assigned to boolean)
x = 'foo'; // Error (not null, not undefined)
```

You can omit types and rely on type inference:

```
let n = 7; // (type number inferred)
n = 'hello'; // Error (string assegnata a number)
```

**Nota:** Uninitialized variables with no type declaration (eg. `let x;`) are
considered `any` variables.

## Array and tuples

Arrays can be declared like this:

```
let n: number[];
```

or:

```
let n: Array<number>;
```

TypeScript arrays can only store values of the declared type:

```
let n: number[];
n = [ 1, 2, 3 ]; // OK
n = [ '1', '2', '3' ]; // Error
```

If you need to store different types in the same array (just like in JavaScript)
you can use tuples:

```
let rating: [ string, number ];
rating = [ 'Matrix', 5 ]; // OK
rating = [ 'Sharknado, '1' ]; // Error
```

The type inference systems supports arrays too:

```
let n = [ 1, 2, 3 ]; // number[]
```

## Functions

You can specify parameter types and return type:

```
function sum(a: number, b: number): number {
  return a + b;
}
sum(2, 2); // 4
sum('2', '2'); // Error
```

It's usually safe to omit the return type and rely on type inference:

```
function sum(a: number, b: number) {
  return a + b;
}
let s: string = sum(2, 2); // Error (number assigned to string)
```

## Interfaces

Interfaces allow to defined object types in terms of expected properties:

```
interface Person {
  name: string;
  age: number;
}
```

Remember that properties can also be methods:

```
interface Calculator {
  sum(a: number, b: number): number;
  multiply(a: number, b: number): number;
  ...
}
```

Declared properties are implicitly required:

```
interface Person {
  name: string;
  age: number;
}
const marco: Person = { name: 'Marco', age: 30 }; // OK
const johnDoe: Person = { age: 33 }; // Error (name missing)
```

...but you can mark them as optional:

```
interface Person {
  name?: string;
  age: number;
}
const marco: Person = { name: 'Marco', age: 30 }; // OK
const johnDoe: Person = { age: 33 }; // OK
```

You can't use unexpected properties:

```
interface Person {
  name?: string;
  age: number;
}
let marco: Person = { name: 'Marco', age: 30 }; // OK
marco = { name: 'Marco', age: 30, job: 'developer' }; // Error
```

## Type assertions

You can temporarily treat a variable as if it had a different type:

```
interface Person {
  name: string;
}

interface Athlete {
  name: string;
  sport: string;
}

const marco: Person = { name: 'Marco' };
marco.sport = 'soccer'; // Error
(marco as Athlete).sport = 'soccer'; // OK
```

There's also an alternative syntax:

```
(<Athlete>marco).sport = 'soccer';
```

## Classes

You can define classes and declare their expected istance variables:

```
class Person {
  name: string;
}
const marco = new Person();
marco.name = 'Marco'; // OK
marco.job = 'Developer'; // Error
```

You can define a custom constructor:

```
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}
const marco = new Person('Marco');
console.log(marco.name); // 'Marco'
```

You can define one or more instance methods:

```
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  sayHi(): string {
    return `Hi I'm ${this.name}`;
  }
}

const marco = new Person('Marco');
marco.sayHi(); // 'Hi I'm Marco'
```

TypeScript classes compile into JavaScript constructor functions. For instance,
this code:

```
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}
const marco = new Person('Marco');
```

...would be translated into something like:

```
function Person(name) {
  this.name = name;
}
var marco = new Person('Marco');
```

## Implementing interfaces

The implementing class must contain every property declared inside the
interface:

```
interface HasName {
  name: string;
}

interface CanTalk {
  saySomething(what: string);
}

class Person implements HasName, CanTalk {
  name: string;

  saySomething(what: string) {
    console.log(what);
  }
}
```

## Encapsulation

```
class Person {
  private name: string;

  constructor(name: string) {
    this.name = name;
  }
}

const marco = new Person('Marco');
console.log(marco.name); // Error (private variable)
```

You can also declare `protected` instance variables, which makes them directlty
accessible inside [subclasses](#inheritance).

You can encapsulate methods too:

```
class Example {
  private aPrivateMethod() {
    console.log('OK');
  }

  aPublicMethod() {
    this.aPrivateMethod();
  }
}

const example = new Example();
example.aPrivateMethod(); // Error (private method)
example.aPublicMethod(); // 'OK'
```

You can also declare `readonly` instance variables, which can be assigned only
inside constructors:

```
class Person {
  readonly name: string;

  constructor(name: string) {
    this.name = name;
  }

  changeName(newName: string) {
    this.name = newName; // Error
  }
}

const marco = new Person('Marco');
console.log(marco.name); // 'Marco';
marco.name = 'Mario'; // Error
```

## Static members

Static members are attributes or methods of a class. When a TypeScript class is
compiled, static members become properties of the resulting JavaScript
constructor function. For instance, this code:

```
class Counter {
  static n: number;

  static initialize() {
    Counter.n = 0;
  }
}
```

...translates into something like this:

```
function Counter() {}
Counter.initialize = function () {
  Counter.n = 0;
};
```

## Inheritance and polymorphism

```
class Person {
  protected name: string;

  constructor(name: string) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

class Athlete extends Person {
  protected sport: string;

  constructor(name: string, sport: string) {
    super(name);
    this.sport = sport;
  }

  getNameAndSport() {
    return `${this.name} ${this.sport}`;
  }
}

const marco: Person = new Athlete('Marco', 'Soccer');
marco.getName(); // 'Marco'
marco.getNameAndSport(); // Errore
(marco as Athlete).getNameAndSport(); // 'Marco Soccer'
```

## Overriding

```
class Person {
  protected name: string;

  constructor(name: string) {
    this.name = name;
  }

  sayHi() {
    return `Hi I'm ${this.name}`;
  }
}

class Athlete extends Person {
  protected sport: string;

  constructor(name: string, sport: string) {
    super(name);
    this.sport = sport;
  }

  sayHi() {
    return `Hi I'm ${this.name} and I play ${this.sport}`;
  }
}

const marcoPerson = new Person('Marco');
const marcoAthlete = new Athlete('Marco', 'soccer');
marcoPerson.sayHi(); // 'Hi I'm Marco'
marcoAthlete.sayHi(); // 'Hi I'm Marco and I play soccer'
```

## Abstract classes

```
abstract class Shape {
  abstract area(): number;
}

class Square extends Shape {
  side: number;

  constructor(side: number) {
    super();
    this.side = side;
  }

  area() {
    return this.side * this.side;
  }
}

const square: Shape = new Square(8);
square.area(); // 64
```

## Generics

You can define generic functions:

```
function identity<T>(x: T): T {
  return x;
}

identity<number>(7); // 7
identity<number>('7'); // Error
identity<string>('7'); // '7'
identity('7'); // '7' (type string inferred)
```

...or even generic classes:

```
class IdentityCalculator<T> {
  apply(x: T): T {
    return x;
  }
}

const numberIdentity = new IdentityCalculator<number>();
numberIdentity.apply(7); // 7
numberIdentity.apply('7'); // Error

const stringIdentity = new IdentityCalculator<string>();
stringIdentity.apply(7); // Error
stringIdentity.apply('7'); // '7'
```
