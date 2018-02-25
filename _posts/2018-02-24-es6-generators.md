---
layout: post
title:  "Generator Functions"
date:   2018-2-24 06:56:04 -0500
description: A discussion of generator functions, an ES6 technique using yield to pause the execution of functions until they are called later.
author: Thomas Danner
lang: en_US
categories: tutorials fundamentals
tags: JS, generator
comments: true
---

## Generator Functions

A [generator function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) is a function that can exit and re-enter execution at a later time. When the function executes later on, it retains its memory of its local variables. In this sense, it acts like a [closure](http://localhost:4000/tutorials/javascript/fundamentals/2017/12/31/code-morsels-closures-let-vs-var.html#enter-closures).

The best way to understand generators is with some simple examples.

### Defining Generation

The syntax for defining a generator function is to use a `*` before the function name and then have `yield` statements in the function body.

```javascript
function *generate() {
  yield "this";
  yield "is";
  yield "a";
  yield "sentence";
}
```

We instantiate the generator like so:

```JavaScript
let G = generate();
G.next();
```

Calling G.next() returns an object with a `value` and a `done` property. `value` corresponds to the current value yielded by the generator function. `done` is a boolean signifying whether the generator function is exhausted for not.

Calling `G.next()` sequentially in our example function yields:

```javascript
{value: "this", done: false}
{value: "is", done: false}
{value: "a", done: false}
{value: "sentence", done: false}
{value: undefined, done: true}
```

Note that `done` is *not* true until we call the function and there are no more yields left.

### Generators With Arguments

Generators can also take arguments.
```javascript
function *AddOneMore(a) {
  yield (a+1);
  yield (a+2);
  yield (a+3);
}

let v=AddOneMore(1);
v.next(); // {value: 2, done: false}
v.next(); // {value: 3, done: false}
v.next(); // {value: 4, done: false}
```

### Generators With Returns

If a return statement is included in a generator, the generator will be exhausted when the return statement is reached. Unlike a generator that only contains yields, when the return statement is reached, the object will have `done` true and the value returned.

```javascript
function *returnedG(arg) {
  yield 1;
  return arg;
}

let v=returnedG(2);
v.next(); // {value: 1, done: false}
v.next(); // {value: 2, done: true}
v.next(); // {value: undefined, done: true}
```

### Infinite Generators

An infinite loop in a generator function can be used to return the results of an infinite series. For example, we can create an infinite list of IDs and return the next ID each time the generator function is called.

```javascript
function *idGenerator() {
  let id=0;
  while(true) { yield ++id; }
}
```
##### Generate unique numeric IDs (until integer max of course)

```javascript
function *zeroToTen() {
  let id=0;
  while(true) { yield id>=10 ? id=0 : ++id; }
}
```
##### Generate recurring numbers 0-10

```javascript
function *fibbo() {
  let a = 0;
  let b = 1;
  while (true) {
    let current = a;
    a = b;
    b = current + a;
    yield current;
  }
}
```
##### Generator for the Fibonacci sequence

### Generators Yielding To Other Generators

Generators can even yield execution to other generators.

```javascript
function *a() {
  yield 1;
  yield *b();
  yield "a is now in control";
}

function *b() {
  yield "b is now in control";
}

let backAndForth=a();
backAndForth.next(); //1
backAndForth.next(); //"b is now in control"
backAndForth.next(); //"a is now in control"
backAndForth.next(); //{value: undefined, done: true}
```

## But, why?

Generators are more than just syntactic sugar. They are powerful tools for making infinite sequences or tasks manageable by implementing [lazy evaluation](https://en.wikipedia.org/wiki/Lazy_evaluation), the paradigm of only producing results when they are requested. Lazy Evaluation is why you can have a `while (true)` loop in the function that doesn't hang infinitely: `yield` breaks out of the while loop until the next time the function is called.

Look for ways that you can implement generators to make your code more readable. Consider how you might use it with asynchronous code like fetch or use it to make more readable functions that perform similar tasks.
