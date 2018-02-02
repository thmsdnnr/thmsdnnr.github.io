---
layout: post
title:  "Map, Filter, and Reduce"
date:   2018-1-3 10:04:04 -0500
description: Map, Filter, and Reduce basics with Standard Deviation and Applications
author: Thomas Danner
lang: en_US
categories: tutorials javascript fundamentals
tags: JS, map, filter, reduce, standardDeviation, functionChaining, functionalProgramming
comments: true
---

Map, filter, and reduce are useful tools that facilitate a coding ethos known as Functional Programming.

> [Functional programming](https://en.wikipedia.org/wiki/Functional_programming) is a paradigm in which we create programs using expressions and declarations rather than statements.

```javascript
const cats=[ //expression
  { name: 'fluffy', age:10 },
  { name: 'charles', age:14 },
  { name: 'earl', age:1 },
  { name: 'antoinette', age:3 }
];
//declaration
let catsAfterFiveYears = cats.map(cat=>Object.assign({}, cat, {age:cat.age+5}));
```
Writing this code using a statement would iterate through with a for loop like this:
```javascript
//statement
function insteadOfUsingMap() {
  var nextCats=[];
  for (var i=0;i<cats.length;i++) {
    let thisCat=cats[i];
    thisCat.age+=5;
    nextCats.push(thisCat);
  }
  return nextCats;
}

let catsAfterFiveYears = insteadOfUsingMap();
```

Both pieces of code work. But imagine what would happen if we wanted to tweak the code slightly. The FP approach is more maintainable, and it's easy to see at a glance what is going on. Also, it is not prone to typos (what if you accidentally mutate your for loop variable, for instance).

### So what does it look like?

`map`, `filter`, and `reduce` enable programming in an FP style. A good way to understand what a function is doing is to see how it is invoked and then try to mimic its behavior by coding it yourself. That's what we'll do, as well as provide a few examples of how the built-in functions can be used.

Map and filter both take a callback function that is passed these parameters:

* the current element (required)
* the index of the element in the array (optional)
* the array that is being mapped over (optional)
* a value to use as `this` when executing the function (optional)

## Map

[Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) applies a function to every element of an array and returns an array containing the results of these applications. In FP, it is often called "apply-to-all".

### Here are some examples.

```javascript
[1,2,3,4,5].map(num=>num*2);
//[2, 4, 6, 8, 10]
```
##### *multiply every element by two*
```javascript
[1,2,3,4,5].map((num,idx)=>idx%2!==0 ? num*2 : num);
//[1, 4, 3, 8, 5]
```
##### *multiply every odd-indexed element by two*
```javascript
[1,2,3,4,5].map((num,idx,arr)=>(idx!==0&&idx<arr.length-1) ? num*2 : num);
//[1, 4, 6, 8, 5]
```
##### *multiply every element (except the first and last) by two*

We can also apply the array's `map` to other types, like strings.

`'fluffykins'.map(e=>e)` gives us a TypeError, because strings do not have a `map` function. But we can apply array map's function to strings, like this:

```javascript
Array.prototype.map.call('fluffykins', (letter)=>{
  return String.fromCharCode(letter.charCodeAt()+1);
}).join("");
//"gmvggzljot"
```
##### *shift every letter of a string one to the "right" in the alphabet*

Map can also be chained: for example, to operate on nested arrays.

```javascript
[[1,2],[3,4],[5]].map(subArray=>{
  return subArray.map((e, idx, arr)=>arr.length>1 ? e*2 : e)
});
//[[2,4],[6,8],[5]]
```
##### *multiply each element of every nested array by two if it contains more than one element*

### Let's write our own map.

```javascript
function myOwnMap(array, callback, thisArg) {
    thisArg = thisArg || null;
    let results=[];
    for (var idx=0;idx<array.length;idx++) {
      results.push(callback.apply(thisArg,[array[idx],idx,array]));
    }
    return results;
}
```

Note what map's doing: it's taking this iteration over the elements and wrapping it up into a function that we only have to write once. We can then re-use that function with more complex logic (checking indices, comparing elements) or more complex data types (arrays of objects with many interrelated keys) without worrying about the low-level implementation.

```javascript
myOwnMap([1,2,3,4,5], (num,idx,arr)=>(idx!==0&&idx<arr.length-1) ? num*2 : num);
//[1, 4, 6, 8, 5]
```

Nice!

## Filter

[Filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) tests every element of an array against a callback function, returning all elements that pass the test. In FP, it is often called "remove-if".

Filter passes its callback functions the same parameter as map. The only difference is that instead of returning a result array with the callback's return *value*, it returns a result array with the elements for which the callback's return value is *true*.

### Here are some examples.

```javascript
const firstFibs=[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233];
const fibsUnder100 = firstFibs.filter(fNum=>fNum<100);
//[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
```
##### *Fibonacci numbers under 100 from a larger list*

```javascript
const numbers=[1,2,3,4,5,6,7,8,9,10];
const middle8 = numbers.filter((num,idx,arr)=>(idx!==0&&idx<arr.length-1));
//[2, 3, 4, 5, 6, 7, 8, 9]
```
##### *filter out the first and last numbers from an array*

```javascript
const mixedTypes=[1,'3',{obj:'a'},77,'cats','cute cats','misc text'];
const onlyStrings = mixedTypes.filter(e=>typeof e==='string')
//["3", "cats", "cute cats", "misc text"]
```
##### *extract strings from an array*

Filter can also be chained.

```javascript
const mixedTypes=[1,'3',{obj:'a'},77,'cats','cute cats','misc text'];
const onlySingleWordStrings = mixedTypes
.filter(e=>typeof e==='string')
.filter(e=>e.split(" ").length===1)
//["3", "cats"]
```
##### *extract strings from an array that only have a single word (no spaces)*

### Let's write our own filter.

```javascript
function myOwnFilter(array, callback, thisArg) {
    thisArg = thisArg || null;
    let results=[];
    for (var idx=0;idx<array.length;idx++) {
      let result = callback.apply(thisArg,[array[idx],idx,array]);
      if (result) { // if element meets conditions
        results.push(array[idx]);
      }
    }
    return results;
}
```

It works!

```
myOwnFilter([1,2,3,4,5], e=>e>2)
//[3,4,5]
```

## Reduce

[Reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) takes an array of elements and `reduce`s them to a single element, following the rule given by the callback function supplied. Reduce also takes an optional `initialValue`, the value of the accumulator before any operations are performed. It is often used to sum/multiply/subtract/divide.

In FP, reduce is known as `fold`.

Reduce passes its callback different parameters than `map` and `filter`.

* accumulator -> holds result of all previous reduce calls, or `initialValue`, if provided (required)
* currentValue -> the current value being processed (required)
* currentIndex -> the index of currentValue (optional)
* the array that reduce is being called upon (optional)

It is also passed the optional parameter `initialValue`.

### Here are some examples.

```javascript
[1,2,3,4,5].reduce((acc,ele)=>acc+=ele, 0);
//15
```
##### *sum all the numbers in an array*

Note the use of the `initialValue`. This is important. Imagine we try to reduce an empty array.

`[].reduce((acc,ele)=>acc+=ele);`

If we do not supply an initial value, we'll get this error:

```
VM70:1 Uncaught TypeError: Reduce of empty array with no initial value
    at Array.reduce (<anonymous>)
    at <anonymous>:1:4
```

Reduce is trying to add nothing to nothing.

* `acc` is undefined, since at the beginning, it is equal to `intialValue`
* `ele` is undefined, since the array contains no elements. What's undefined+undefined?

```javascript
[1,2,3,4,5].reduce((acc,ele,idx)=>idx%2!==0 ? acc+=ele : acc, 0);
// 6
```
##### *sum every odd-indexed element*

```javascript
[1,2,3,4,5].reduce((acc,ele,idx,arr)=>(idx!==0&&idx<arr.length-1) ? acc+=ele : acc,0)
// 9
```
##### *sum all but the first and last element*

Unlike `map` and `filter`, there is not an application for chaining `reduce`, since its end product is a single value from many values: a mapping from many-to-one.

### Let's write our own reduce.

```javascript
function myOwnReduce(array, callback, initialValue) {
  let acc=initialValue;
  for (var idx=0;idx<array.length;idx++) {
    acc=callback.apply(null,[acc, array[idx], idx, array]);
  }
  return acc;
}
```

It works the same way.

```javascript
myOwnReduce([1,2,3,4,5],(acc,ele,idx)=>idx%2!==0 ? acc+=ele : acc, 0);
// 6
```

## What about dot notation?

Our methods can be chained by [functional composition](https://en.wikipedia.org/wiki/Function_composition), but not with dot notation.

If you wanted to write your own dot-chainable methods, [see this](http://xahlee.info/js/js_function_chaining.html). The reason that we are able to chain the native `.map` is because it is a [prototype method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype) of the Array object.

You can **but should not** ["monkey patch"](https://en.wikipedia.org/wiki/Monkey_patch#Pitfalls) built-in JS methods in this way:

```JavaScript
Array.prototype.myOwnMap = function(callback, thisArg) {
    thisArg = thisArg || null;
    let results=[];
    for (var idx=0;idx<this.length;idx++) {
      results.push(callback.apply(thisArg,[this[idx],idx,this]));
    }
    return results;
}
```

Note that this is identical to our `myOwnMap` function, except instead of taking an `array` argument, it uses `this` to refer to the array it is operating on (itself).

Then we could do this:

```javascript
[1,2,3,4,5].myOwnMap(e=>e*10).myOwnMap(e=>e/5)
//[2,4,6,8,10]
```

However, monkey-patching is almost never a good idea, nor is it a good idea to define a "special" `map` behavior for your own objects. If you need a special behavior that *uses* map, write a function that describes the behavior and use map *inside* of it. It would be an especially bad idea to modify `Array.prototype.map` directly, since *every other instance* in your code that uses `map` will be effected, almost certainly for the worse.

These are tenets of functional programming: create functions that do not create side-effects. Create small and maintainable functions that can be composed to create larger functions.

## Bring it all together: Calculating Standard Deviation

[Standard deviation](https://en.wikipedia.org/wiki/Standard_deviation) is a common tool used to measure variance from a mean, assuming a normally distributed set of data. We'll write a function using `map`, `reduce`, and `filter` to calculate the standard deviation of a set of data, excluding data that lies outside the bounds passed to the function.

```javascript
function standardDeviation(data, lowerBound, upperBound) {
    lowerBound = lowerBound || Math.min(...data);
    upperBound = upperBound || Math.max(...data);
    let filteredData=data.filter(e=>(e>=lowerBound&&e<=upperBound));
    let average=filteredData.reduce((acc,ele)=>acc+=ele,0)/filteredData.length;
    let numerator=filteredData.map(e=>{
      return Math.pow((e-average),2);
    }).reduce((acc,ele)=>acc+=ele,0);
    //don't divide by zero
    let denominator=filteredData.length-1;
    if (denominator<=0) { return 0; }
    let division=numerator/denominator;
    return Math.sqrt(division);
}
```

We can use the ES6 [spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator) with `Math.min()` and `Math.max()` to cleanly set default lower and upper bounds to the extent of the dataset, if no bounds are provided. Then, we just compose our previous functions to calculate the sample standard deviation using this formula:

> <img src="https://i.imgur.com/rbZj7lP.png" height="70px" alt="Formula for sample standard deviation from Wikipedia">

## TIL

Today we learned about `map`, `filter`, and `reduce`, [higher-order functions](https://en.wikipedia.org/wiki/Higher-order_function) that facilitate FP.

One goal of functional programming is to create code that can be reused. This means coding small, functional units that can be composed in different ways to produce different results. This makes code easier to reason about, test, and debug.

Try to identify places in your code where you can utilize a functional approach. If you're interested more, try playing with [Haskell](https://www.haskell.org/)...see [Learn You a Haskell For Great Good](http://learnyouahaskell.com/) for a fun starting point.
