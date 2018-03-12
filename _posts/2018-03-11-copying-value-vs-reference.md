---
layout: post
title:  "Value vs Reference: Deep vs Shallow"
date:   2018-3-11 06:56:04 -0500
description: A short tutorial on object copying in JS and the distinction between a shallow and deep copies, and values and references.
author: Thomas Danner
lang: en_US
categories: fundamentals
tags: JS, fundamentals, object, copy, shallow, deep, clone, value, reference
comments: true
---

## Copying A Thing

It seems like the easiest thing in the world: copy the thing stored in one variable into another variable so that you can manipulate two instances of this variable.

Say you have an object you want a copy of. The naive way to copy would be to create a new variable and set it equal to the object you desire to copy.

At first, all appears well and good. But then you try to modify either of the two objects. Modifying the property of one modifies the property of the other:

```javascript
let catObj = { name: 'fluffy', age:5 }
let anotherCat = catObj;
anotherCat.age=10;
anotherCat.age==catObj.age; // true
delete catObj.name;
delete catObj.age;
console.log(anotherCat); // {}
```

This is not a bug. Why is this? The two properties *reference* the same value.

## Pass by Value vs Pass by Reference

The distinction between value and reference is the distinction between a unique copy of a value and a pointer to a single instance of a value. The two objects reference the same value. This means that changing the value of one of the references alters the single value.

If you don't need to change the value of a thing and it is shared across multiple object instances, it makes a sense to just store a pointer to that thing. After all, when you go to copy these instances, you only have to copy a single pointer rather than the entire object.

In Javascript, objects and arrays are mutable. Without explicitly copying them, any variables that point to them are *references* to them, meaning that making a local change will impact every other variable that points to that value.

```javascript
function giveCatHaircut(obj) {
  let trimmedHair=obj;
  trimmedHair.haircut=true;
}
let fluffy = {name: 'fluffy', haircut: false };
giveCatHaircut(fluffy);
let fluffy = {name: 'fluffy', haircut: true };
```
##### Object mutation inside a function

```javascript
function countSingles(arr) {
  let singles=0;
  while (arr.length) {
    let element = arr.shift();
    if (String(element).length==1) { singles++; }
  }
  return singles;
}

let mixedArr=[1,2,3,'cats', 'horses', 'dogs'];
countSingles(mixedArr); // 3, correct!
console.log(mixedArr); // [], oops.
```
##### Naive way to count the number of length===1 elements in an array

Pass-by-reference is usually a nice feature: indeed, it's a key mechanism for [prototypal inheritance](http://thmsdnnr.com/fundamentals/2018/03/05/prototypal-inheritance.html) to function. It's only an issue when the implicit nature of the reference causes you to unintentionally mutate state and create unexpected behavior. In languages like C, pointers and references are explicitly declared. In JS, it happens under the hood.

## Fixing the Two Examples

### Object copying

We can copy one-dimensional (non-nested) objects using [`Object.assign`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign), which copies all enumerable and own properties from specified sources (one or more objects) to a target object. If multiple source objects are defined, matching keys are that occur earlier overwritten by keys that occur later (assignment occurs from left-to-right).

```javascript
let a = { cat: "big" }
let b = { cat: "small" }
let c = { cat: "medium" }
let newCat = Object.assign({}, c, b, a);
newCat; // {cat: "big"}
let newCat1 = Object.assign({}, a, c, b);
newCat1; // {cat: "small"}
```

A selective cat barber.

```javascript
function giveCatHaircut(obj) {
  let trimmedHair=Object.assign({}, obj);
  trimmedHair.haircut=true;
  return trimmedHair;
}

let fluffy = {name: 'fluffy', haircut: false };
let haircut=giveCatHaircut(fluffy);
console.log(fluffy); // {name: 'fluffy', haircut: false };
console.log(haircut); // {name: 'fluffy', haircut: true };
```

### Array Copying

We can copy an array with [`Array.slice()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice). We could also fix up our function by using `map` and `filter` to immutably calculate the desired value:

`return arr.map(e=>String(e)).filter(e=>e.length==1).length;`

Since map and filter maintain internal variables that track the state of their operations, they are making an implicit copy of the array and do not mutate the original in place.

## Multidimensional Copying

Object.assign fails to create copying behavior when the object is nested: that is, when its values are themselves objects. Instead, it assigns *references*.

```javascript
let catObj = { name: 'fluffy', age: 5, friends: ['larry', 'charlie', 'suzy']}
let anotherCat = Object.assign({}, catObj);
anotherCat.friends.push('ned');
console.log(anotherCat.friends); // ['larry', 'charlie', 'suzy', 'ned']
console.log(catObj.friends); // ['larry', 'charlie', 'suzy', 'ned']
```

`Object.assign` works fine at a given level of an object to copy values, but it fails when those values are not primitives or are themselves objects.

Similarly with arrays, `Array.slice()` does not create value copies of nested arrays.

```javascript
function notImmutable(arr) {
  let localCopy=arr.slice();
  localCopy[1][1] = 999;
  return localCopy;
}

let nestedArr=[1, [2, 3]]
console.log(notImmutable(nestedArr)); // [1, [2, 999]]
console.log(nestedArr); // [1, [2, 999]] doh!
```
##### Array.slice() only copies by value those elements on the level of the array on which .slice was called. Nested values are copied as references.

How can we make a deep copy of a nested array or object?

We need to recursively examine each level of the object in question. If a given level consists of only primitives, we assign the copy these values. If the given level consists of non-primitives, we copy the value according to the immutable copying method for that primitive (`Array.slice()`, `Date = new Date()`, etc.). We then continue until we reach a level of the object which does not contain any nested levels itself.

There are a few gotchas involved in writing such a method. One is circular references:

```javascript
anotherCat.self=anotherCat;
console.log(anotherCat); // self: [Circular]
anotherCat.self // another cat
anotherCat.self.self // another cat
```

`anotherCat.self` references anotherCat, and so on into infinity.

If we write a method that iterates over the object and copies its properties, we have to have a way of detecting whether there are circular references in the object. Otherwise, recursing on a given property of the object until it has no other children would lead to an infinite loop (imagine: recurse on anotherCat.self until anotherCat.self is a primitive).

Detecting a circular reference is more than just detecting an object that points to an object that is the *same* as itself -- note that we could have two distinct objects that share all the same values and have a link between them: the reference would not be circular, even though it would appear to be so, just as in the above example.

## Maybe Later.

Writing the code to deep copy an object while managing all the edge cases is outside the scope of this tutorial: indeed, entire libraries exist to solve this problem [like](https://github.com/mout/mout) [these](https://lodash.com/docs/4.17.5#cloneDeep). You should probably use one, if you really need to exhaustively copy an entire array or object. You should also consider: do you *really need* to exhaustively copy an entire array or object? Often using Immutable data structures or selectively slicing out elements depending on your use case is enough. It's also easier to read and maintain.

[ImmutableJS](https://facebook.github.io/immutable-js/) makes the intricacies of deep cloning one case for immutability: since the structure of objects cannot be changed directly, you can copy an object simply by copying a reference to itself. Note that all the problems we encountered above came about because we set a reference and then changed the value through that reference.

The key here is to understand the concepts and limitations of object and array copying methods, to understand the difference between pass-by-reference and pass-by-value, and to understand when you'd want to use one or the other: to modify in place vs to perform non-mutating operations to return a result. Keeping these concepts in mind is key to writing and understanding code with any level of complexity.

## TIL

Copying objects is complicated. It's rare that you will want to copy an entire object indiscriminately. The key takeaways:

* Objects and Arrays are passed as *references* and have to be explicitly copied unless you wish to mutate the original.
* Other primitives are passed by value.
* Individual levels of an array can be copies with `.slice()`, but `.slice()` returns *references* to nested arrays.
* Similarly with Objects, `Object.assign({}, source)` can be used, but nested objects will return references as well.
* Consider using a library or custom method for selective deep copying. It's often better to take intelligent slices rather than indiscriminate copies for memory and performance: and it's also a lot easier to debug and follow logically.
