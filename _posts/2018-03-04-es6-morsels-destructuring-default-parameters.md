---
layout: post
title:  "ES6 — Destructuring + Default Parameters"
date:   2018-3-4 06:56:04 -0500
description: A discussion of object destructuring in ES6, a technique to easily extract selected properties of nested objects.
author: Thomas Danner
lang: en_US
categories: ES6
tags: JS, ES6-morsel, destructuring
comments: true
---

## ES6 Morsels?

In this series, I'm going to cover one or two new ES6 features per post with a deep dive into code examples and consider when it makes sense to use new functionality.

## What's ES6?

Javascript, like any language, evolves. The [TC-39](http://www.ecma-international.org/memento/TC39.htm) meets to discuss changes to the language. The most recent version [is here](http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf), at a cool 885 pages long.

Let's reduce this complexity a little bit. The language specification is implemented to varying extents in different browsers. [This compatibility table](http://kangax.github.io/compat-table/es6/) is a great way to learn what you can and cannot implement natively.

If a browser does not support a new feature yet, there's a tool called [Babel](https://babeljs.io/) that lets you "use next generation Javascript, today". It is a transpiler: it compiles JS written with new features into JS that browsers that do not implement these features can still understand.

Babel can be hooked into your development toolbox so that any code you write gets transpiled down in your final product.

## Isn't this needlessly complex?

Maybe. The biggest draw toward new features is future compatibility. Where backwards compatibility seeks to make new code jive with older browsers (the hole that Babel fills), future compatibility aims to make code conversant in the "languages of tomorrow". Whether this ad copy is needless headache, revelatory development, or fair to middling is a question for your project team. Even if you don't plan to use many of the new features, it's still good to know that they exist.

> With all of these new features, as with most anything in life, it's important to critically ask: am I using this because it's new and other people are adopting it, or because it truly makes sense for my life and will enrich it?

## Default Parameters

**[Default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)** are exactly what they sound like: the values that things default to when no other values are provided.

In ES6, they are syntactic sugar for checking whether a function is passed an undefined argument and, if so, setting a given default value. Take a look:

```javascript
function treeFell(observer="me") { return observer ? true : false; }
```
##### ES6

```javascript
"use strict";

function treeFell() {
  var observer = arguments.length > 0 && arguments[0] !== undefined
    ? arguments[0] : "me";
  return observer ? true : false;
}
```
###### Babel-transpiled

Hey, this function always returns true, thanks to the default value. Maybe [essence is not perception](https://en.wikipedia.org/wiki/If_a_tree_falls_in_a_forest) after all!

Default arguments can refer to default arguments that *precede* them.

```javascript
function pluralize(singular="cat", plural=singular+"s") {
  return {one: singular, many: plural};
}
console.log(pluralize('house')); // {one: 'house', many: 'houses'}
```

Note the importance of order here. `pluralize(plural=singular+"s", singular="cat") {` will *not* work: instead, it returns `{one: 'cat', many: 'house'}`.

Finally, the default argument does not persist once the function returns. If you need to maintain the state, you should use a closure or store the value somewhere else.

```javascript
function newEachTime(valToPush, array=[]) {
  array.push(valToPush);
  return array;
}
newEachTime(1); // [1]
newEachTime(4); // [4], NOT [1, 4]
```

Default function parameters can be handy. However, they can also be dangerous. You should guard against relying upon them too heavily. Their use can both limit the generality of a function as well as obfuscate what happens when a function is called. This can make code more difficult to maintain.

## Destructuring

**[Destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)** enables us to selectively extract properties from an object or entries from an array. Attempting to access properties that do not exist or indices out of range returns a value of undefined.

#### With Arrays

```javascript
let test=[1,2,3,4,5];
let [, two, three, , five] = test;
console.log(two+three); // 5
console.log(five); // 5
```
##### ES6

```javascript
"use strict";
var test = [1, 2, 3, 4, 5];
var two = test[1],
    three = test[2],
    five = test[4];
```
##### Babel-transpiled

You can perform nested extraction from arrays as well.

```javascript
let test=[1,[2, 2.1, 2.2, 2.3, 2.4, 2.5],3,4,5];
let [, [,,,,,twoPointFive], three, , five, six] = test;
console.log(twoPointFive); // 2.5
```
##### ES6

```javascript
"use strict";

var _slicedToArray = function () {
  function sliceIterator(arr, i) {
    var _arr = []; var _n = true; var _d = false; var _e = undefined;
    try { for (var _i = arr[Symbol.iterator](), _s; !(_n = (_s = _i.next()).done); _n = true)
      { _arr.push(_s.value); if (i && _arr.length === i) break; }
    }
    catch (err) {
      _d = true; _e = err;
    }
    finally {
      try {
        if (!_n && _i["return"]) _i["return"]();
      }
      finally {
        if (_d) throw _e;
      }
    }
    return _arr;
  }
  return function (arr, i) {
    if (Array.isArray(arr)) { return arr; }
    else if (Symbol.iterator in Object(arr)) {
      return sliceIterator(arr, i);
    }
    else {
      throw new TypeError("Invalid attempt to destructure non-iterable instance");
    }
  };
}();

var test = [1, [2, 2.1, 2.2, 2.3, 2.4, 2.5], 3, 4, 5];

var _test$ = _slicedToArray(test[1], 6),
    twoPointFive = _test$[5],
    three = test[2],
    five = test[4],
    six = test[5];
```
##### Babel-transpiled

Whether this is worth it or not is questionable. You can write five commas or `test[1][5]`. I prefer the latter.

#### With Objects

```javascript
let cuteAnimalRatings = {
  cats: [{name: "Fluffykins", rating: 10}, {name: "Hugo", rating: 6}],
  dogs: [{name: "Errol", rating: 3}, {name:"Hedwig", rating:11}]
};

const { cats, dogs, iDoNotExist } = cuteAnimalRatings;
console.log(cats); // [{name: "Fluffykins", rating: 10}, {name: "Hugo", rating: 6}],
console.log(dogs); // [{name: "Errol", rating: 3}, {name:"Hedwig", rating:11}]
console.log(iDoNotExist); // undefined
```
##### ES6

```javascript
var cuteAnimalRatings = {
  cats: [{ name: "Fluffykins", rating: 10 }, { name: "Hugo", rating: 6 }],
  dogs: [{ name: "Errol", rating: 3 }, { name: "Hedwig", rating: 11 }]
};

var cats = cuteAnimalRatings.cats,
    dogs = cuteAnimalRatings.dogs,
    iDoNotExist = cuteAnimalRatings.iDoNotExist;
```
##### Babel-transpiled

But what if the object is nested? Well, it works, but it gets pretty ugly.

```javascript
let cuteAnimalRatings = {
  cats: {data:
    {
      nestedWhy: [{name: "Fluffykins", rating: 10}, {name: "Hugo", rating: 6}]
    }
  },
  dogs: [{name: "Errol", rating: 3}, {name:"Hedwig", rating:11}]
};
const { cats: {data: {nestedWhy}={}}={}, dogs, iDoNotExist } = cuteAnimalRatings;
console.log(nestedWhy); //[{name: "Fluffykins", rating: 10}, {name: "Hugo", rating: 6}]
```
##### ES6

```javascript
var _cuteAnimalRatings$ca = cuteAnimalRatings.cats.data;
_cuteAnimalRatings$ca = _cuteAnimalRatings$ca === undefined ? {} : _cuteAnimalRatings$ca;
var _cuteAnimalRatings$ca2 = _cuteAnimalRatings$ca.nestedWhy,
    nestedWhy = _cuteAnimalRatings$ca2 === undefined ? {} : _cuteAnimalRatings$ca2,
    dogs = cuteAnimalRatings.dogs,
    iDoNotExist = cuteAnimalRatings.iDoNotExist;
```
##### Babel-transpiled

Wait, wait. Why these strange extra equals signs and brackets? They are providing default values in the event that a given property does not exist. We are guarding against the case where `cats` in `cuteAnimalRatings` is missing a `nestedWhy` or a `data` property.

Good question: why do we need a default value to prevent against undefined values in this nested case, whereas we did not in the case of `iDoNotExist`? Notice that one is nested at just one level, while the other properties are more deeply nested.

Why does this matter? Long answer: [prototypal inheritance](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain) at work.

The short answer is that `undefined` is not an object and does not have any properties. Actually, since it's not an object, it does not even have the *concept* of having properties.

When we call `cuteAnimalRatings.cats`, we are asking whether the `cats` property exists in `cuteAnimalRatings`. We search in `cuteAnimalRatings` for the property and return it, if it exists. Otherwise, we return undefined. Under the hood, the code might look something like this:

```javascript
const hasOwnProperty = (prop) => Object.keys(this).filter(e=>e===prop) ?
  this.prop : undefined;
```

`Object.keys(undefined)` gives us a TypeError though. An empty object `{}` is an object, and any properties that do not exist on that object will be undefined. So it's safe to check properties on it, even though it has none (`Object.keys({}).length` => 0). By inserting it as a default value in our lookup chain, it's a safe placeholder for null.

If this seems like an awful lot of work, you're right.

#### What problem are we trying to solve?

The pattern here is: how can we access nested properties of an object when we're not sure whether those properties are undefined? Wouldn't it be nice if objects had a property accessor that would [short-circuit evaluate](https://en.wikipedia.org/wiki/Short-circuit_evaluation) at the first instance of undefined and would automatically check for property existence under the hood for us so we don't have to?

Gratuitous segue to [the proposal for optional chaining](https://github.com/tc39/proposal-optional-chaining).

There's a proposal for syntax like this: `cuteAnimalRatings?.cats?.data?.nestedWhy`. You can read it as: check for existence of the previous value — if it exists, check for the property following the period.

1. Does cuteAnimalRatings exist? Yes=> check cats property. No=> return undefined.
2. Does the cats property exist? Yes=> check data property. No=> return undefined.
3. Does the data property exist? Yes=> check for nestedWhy. No=> return undefined.
4. Does nestedWhy exist? Yes=> return it. No=> return undefined.

```javascript
let cuteAnimalRatings = {
  cats: {data:
    {
      nestedWhy: [{name: "Fluffykins", rating: 10}, {name: "Hugo", rating: 6}]
    }
  },
  dogs: [{name: "Errol", rating: 3}, {name:"Hedwig", rating:11}]
};
```

It is a fun exercise to write your own option-chaining code.

It looks something like this.

```javascript
function parseObject(propString, obj) {
  propString=propString.split("?").map(e=>e.replace(/^\./g,''));
  if (obj==undefined) { return undefined; }
  for (var i=1;i<propString.length;i++) {
    if (!propString[i].length>0) {
     throw new Error('Property names must have a length of at least one!');
   }
    if (obj[propString[i]]) { obj=obj[propString[i]]; }
    else { return undefined; }
  }
  return obj;
}
console.log(parseObject('cuteAnimalRatings?.cats?.data?.nestedWhy', cuteAnimalRatings));
// [{name: "Fluffykins", rating: 10}, {name: "Hugo", rating: 6}]
```

Coming soon to a language near you!

## TIL

Functions can have default parameters: syntactic sugar to check whether the passed parameter is undefined and, if so, setting the parameter to the given default.

Arrays and objects can have their indices and properties extracted using special syntax called destructuring.

* Arrays use commas `,` as placeholders for destructuring to skip over given indices on assignment.
* Objects do not require placeholders, since they act as associative arrays.

When destructuring nested objects, provide default values to handle nested nulls or undefined properties. Alternatively, create your own polyfill for option-chaining so that properties of unknown status can be handled safely without unhandled exceptions.

And yes, in case you were wondering, you can use destructuring in default parameters.

```javascript
const funcD = ({cats, dogs} = obj) => [cats, dogs];
console.log(funcD({cats:10, dogs:7})); // [10, 7]
```

But again: favor readability over novelty.
