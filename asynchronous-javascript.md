# Asynchronous JavaScript

## Abstract

Browsers and [Node.js](https://nodejs.org), which are the two most important
JavaScript platforms, provide input / output (I/O) operations through
asynchronous APIs. Many developers find that code using asynchronous APIs is
hard to write and understand, but it actually has some benefits and can be made
easier with specific patterns.

In this guide i will briefly introduce the motivations behind asynchronous APIs,
the classical ways in which they are used and some advanced "techniques".

## Indice

* [Background](#background)
* [Callbacks](#callbacks)
* [Promises](#promises)
* [Generators](#generators)
* [Async await](#async-await)

## Background

In order to use asynchronous APIs you need of course to understand what they are
and what they are used for.

Asynchronous APIs are basically a way to request an operation and specify how to
react when the operation will be completed, leaving the requesting thread free
to go on with other work while the requested operation is performed in parallel.

Asynchronous APIs may be implemented with auxiliary threads. In case of I/O
operations, more efficient implementations are possibile, but the OS should
support non blocking I/O.

So, why would browsers and Node.js need asynchronous APIs? In order to
understand it, you need first to understand that browsers and Node.js are
basically event handlers, with events being stuff like clicks on buttons
(browser) or incoming HTTP requests (Node.js). Events are handled by executing
JavaScript code specified by the page (browser) or program (Node.js). It's
crucial that you understand that **event are handled using a single thread, so
events are sequentially processed one after another**. Asynchronous APIs are
thus necessary in order to do I/O, which can be slow, and still process events
quickly. Without asynchronous APIs the browser can't keep up with user
interaction and Node.js can't handle concurrent requests.

In some ways, asynchronous APIs are a price we pay for using a single thread for
event processing. However, **using a single thread has some benefits**: handling
event one at a time means that **the code handling an event won't interfere with
the code handling another event**. Also, **Node.js gains a higher scalability
and a lower latency from the single approach**.

## Callback

So browsers and Node.js are event based. You can "subscribe" to an event and
pass a "callback" function in order to specify how an event should be handled.
For instance, here is how you react to a button press inside a web page:

```
button.addEventListener('click', function () {
  console.log('Button pressed!');
});
```

Since the completion of an asynchronous task is itself an event, i.e. browsers
and Node.js consider it an event and execute JavaScript in response to it,
asynchronous APIs are based on callback too:

```
fs.readFile('/home/marco/Desktop/example.txt', 'utf8', function () {
  console.log('Read completed!');
});
```

Here the `readFile` function from the `fs` module of Node.js is taking a
callback as last parameter (which is a common convention for JavaScript async
APIs).


Callbacks for async APIs usually have parameters representing errors and / or
data:

```
fs.readFile('/home/marco/Desktop/example.txt', 'utf8', function (errore, data) {
  if (errore) {
    console.log('Oops: an error occurred!');
  } else {
    console.log(data);
  }
});
```

**Note:** Errors are represented as the first parameter by convention.

Remember that events are handled one at a time, so **callbacks will always be
executed after the code handling the current event**. For instance, suppose that
the following is the code handling some event:

```
var error = null;
var data = null;
fs.readFile('/home/marco/Desktop/example.txt', 'utf8', function (e, d) {
  error = e;
  data = d;
});
if (error) {
  console.log('Oops: an error occurred!');
} else {
  console.log(data);
}
```

This code is not correct and will always show the error message, even if the
file will be correctly read. That's because the `if-else` will be executed
before the callback, even if it appears later in the code. What really matters
is that it appears inside the handler for the current event, while the callback
will handle a "future" event and thus will be executed later.

So you should always handle the outcome of an async task inside the
corresponding callback. This sometimes leads to nested callbacks:

```
taskOne(function () {
  taskTwo(function () {
    taskThree(function () {
      taskFour(function () {
        taskFive(function () {
          taskSix(function () {
            taskSeven(function () {
              // etc.
            });
          });
        });
      });
    });
  });
});
```

Such a structure is called "Pyramid of Doom" and it quickly becomes a mess in
terms of indentation and scope. Promises are a pattern that solves this problem
and simplify async code in general.

## Promises

With promises instead of passing a callback you get a... promise, i.e. an object
with a `then` method that can used to specify two distinct callbacks, one for
success and one for failure.

```
var aPromise = readPromise('/home/marco/Desktop/example.txt', 'utf8');
aPromise.then(function (data) {
  console.log(data);
}, function (error) {
  console.log('An error occurred!');
});
```

Two major benefits are evident:

* callbacks don't have to be provided immediately
* success and failure are handled independently

You can **chain** promises and solve the "Pyramid of Doom":

```
taskOne().then(function () {
  return taskTwo();
}).then(function () {
  return taskThree();
}).then(function () {
  return taskFour();
}).then(function () {
  return taskFive();
}).then(function () {
  return taskSix();
}).then(function () {
  return taskSeven();
}); // ecc.
```

If callbacks only return new promises, the code can be even shorter:

```
taskOne().
  then(taskTwo).
  then(taskThree).
  then(taskFour).
  then(taskFive).
  then(taskSix).
  then(taskSeven); // ecc.
```

A chain of promises is itself a promise and makes it possible to handle failure
in a single place:

```
taskOne().
  then(taskTwo).
  then(taskThree).
  then(function () {
    console.log('The three tasks succeded');
  }, function () {
    console.log('One of the three tasks failed');
  });
```

This should be enough to start using promises.
[Here](https://promisesaplus.com/) you can find a detailed specification that
can be read in order to fully understand the pattern, which is actually quite
complex, so you should use some library to help you. Conveniently, ECMAScript 6
implements promises out of the box. For instance, here is an example of how to
turn a regular async API into a promise-based API with ECMAScript 6:

```
function promiseBasedApi(x) {
  return new Promise(function (resolve, reject) {
    regularAsyncApi(x, function (error, y) {
      if (error) return reject(error);
      resolve(y);
    });
  };
}
```

## Generators

ECMAScript 6 also introduces generators, i.e. function that can be suspended and
resumed (even multiple times) before returning. As we'll see in the
[next section](#async-await), generators allow new async patterns. But let's
try to understand their basic usage first.

Again, generators can suspend. The `yield` keyword has been introduced for this
purpose:

```
function* exampleOfGenerator() {
  yield;
}
```

**Note:** Generators have an asterisk after `function`.

One way in which generators are different from regular functions is that calling
a generator is not enough to actually get it started. Instead, when you call a
generator you get an iterator object, i.e. an object with a `next` method that
finally lets you start the generator:

```
function* exampleOfGenerator() {
  console.log('Started');
  yield;
}
var iterator = exampleOfGenerator();
console.log('Still not started');
iterator.next(); // starts the generator
```

Execution will suspend when a `yield` is reached:

```
function* exampleOfGenerator() {
  console.log('This will be printed');
  yield;
  console.log('This will NOT be printed');
}
var iterator = exampleOfGenerator();
iterator.next();
```

Execution can then be resumed by calling `next` again:

```
function* exampleOfGenerator() {
  console.log('This will be printed');
  yield;
  console.log('This will be printed too');
}
var iterator = exampleOfGenerator();
iterator.next();
iterator.next();
```

So `next` and `yield` can be used to switch between the calling function and the
generator:

```
function* exampleOfGenerator() {
  console.log('b');
  yield;
  console.log('d');
  yield;
  console.log('f');
  yield;
  // etc.
}
var iterator = exampleOfGenerator();
console.log('a');
iterator.next();
console.log('c');
iterator.next();
console.log('e');
iterator.next();
console.log('g');
// etc.
```

Also, `next` and `yield` can be used to pass values to / get values from the
generator function. Specifically, `next` returns objects based on values that
the generator `yield`s:

```
function* exampleOfGenerator() {
  yield 1;
  yield 2;
  yield 3;
}
var iterator = exampleOfGenerator();
var n = iterator.next();
console.log(n); // { value: 1, done: false }
n = iterator.next();
console.log(n); // { value: 2, done: false }
n = iterator.next();
console.log(n); // { value: 3, done: false }
n = iterator.next();
console.log(n); // { value: undefined, done: true }
```

**Note:** The `done` properties is used to query generator termination.

In turn, `yield` gives access to values passed with `next`:

```
function* exampleOfGenerator() {
  var x;
  x = yield;
  console.log(x);
  x = yield;
  console.log(x);
  x = yield;
  console.log(x);
}
var iterator = exampleOfGenerator();
iterator.next();
iterator.next(1);
iterator.next(2);
iterator.next(3);
```

**Note:** The first `next` doesn't pass anything. In other words, the first
`yield` receives from the second `next`, the second `yield` from the third
`next`, and so on.

You can also trigger an error into a genarator by calling `throw` on its
iterator:

```
function* exampleOfGenerator() {
  try {
    yield;
  } catch (error) {
    console.log(error);
  }
}
var iterator = exampleOfGenerator()();
iterator.next();
iterator.throw('fake error');
```

## Async await

So what do generator have to do with async APIs?

First of all notice that while a regular JavaScript function is executed
entirely during the handling of a single event, the execution of a generator may
span (the handling of) different events. Basically, you just need to call `next`
on the same iterator from different callbacks. For instance, here is how you can
start a generator with a button on a web page and then resume it with another
button:

```
var iterator;
b1.addEventListener('click', function () {
  iterator = (function* () {
    console.log('started by b1...');
    yield;
    console.log('...resumed by b2');
  })();
  iterator.next();
});
b2.addEventListener('click', function () {
  iterator.next();
});
```

This makes generators suitable for "waiting" the result of an async task: you
start waiting by `yield`ing a promise and then you call `next` or `throw` inside
the callbacks:

```
function* example() {
  try {
    var result = yield makeMeAPromise();
    console.log(result);
  } catch (error) {
    console.log(error);
  }
}

var iterator = example();
var promise = iterator.next();
promise.then(function (result) {
  iterator.next(result);
}, function (error) {
  iterator.throw(error);
});
```

The code in the previous example may be refactored into this:

```
function magicFunction(aGenerator) {
  var iterator = aGenerator();
  var promise = iterator.next();
  promise.then(function (result) {
    iterator.next(result);
  }, function (error) {
    iterator.throw(error);
  });
}

magicFunction(function* () {
  try {
    var result = yield makeMeAPromise();
    console.log(result);
  } catch (error) {
    console.log(error);
  }
});
```

`magicFunction` can be extended in order to support waiting for multiple
promises:

```
magicFunction(function* () {
  try {
    var x = yield promiseOne();
    var y = yield promiseTwo();
    var z = yield promiseThree();
    console.log(x + y + z);
  } catch (error) {
    console.log(error);
  }
});
```

Now, if you look at the genarator's body you'll notice that it's starting to
look like plain old synchronous code. From the genarator's perspective there's
no difference between asynchronous values waited with `yield` and values
obtained through a synchronous function. In other words, you don't need anymore
callbacks or complex promise compositions: you just write "regular" code except
that you put `yield` before promises.

Of course this works only if you have a suitable `magicFunction`. You can find a
sample implementation [here](https://www.promisejs.org/generators/) and of
course you can make your own. In ECMAScript 2017 you won't need to do that: the
generator based pattern seen in this section is natively supported with
dedicated syntax. Specifically, you can declare `async` functions and use the
`await` keyword inside them:

```
async function example() {
  try {
    var x = await promiseOne();
    var y = await promiseTwo();
    var z = await promiseThree();
    console.log(x + y + z);
  } catch (error) {
    console.log(error);
  }
}
```

This will be internally translated into something like:

```
async(function* () {
  try {
    var x = yield promiseOne();
    var y = yield promiseTwo();
    var z = yield promiseThree();
    console.log(x + y + z);
  } catch (error) {
    console.log(error);
  }
});
```

Where `async` is a function similar to the `magicFunction` seen in the previous
examples.

This conclude our asynchronous JavaScript overview ;)
