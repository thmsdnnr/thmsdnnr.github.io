---
layout: post
title:  "Copying Objects: Deep vs Shallow"
date:   2018-3-10 06:56:04 -0500
description: A short tutorial on object copying in JS and the distinction between a shallow and deep copy.
author: Thomas Danner
lang: en_US
categories: fundamentals
tags: JS, fundamentals, object, copy
comments: true
---

## Copying Objects

It's often the case that you'll have an object you wish to make a copy of. The naive way to perform the copying would be to create a new variable and set it equal to the object you desire to copy.

At first, all appears well and good. But, then you try to modify either of the two objects. Notice that modifying the property of one modifies the property of the other:

```javascript
let catObj = { name: 'fluffy', age:5 }
let anotherCat = catObj;
anotherCat.age=10;
anotherCat.age==catObj.age; // true
delete catObj.name;
delete catObj.age;
console.log(anotherCat); // {}
```

Why is this? The two are linked by references.

## Shallow Copy

A shallow copy shares references to its property values with the original object it was copied from.

There's a `name` variable equal to "fluffy". When we copy the object into `anotherCat`, its name just points to the same name as catObj. This saves memory, because we have two different objects pointing to the same value in the same place. Indeed, we could have thousands more copies, and the only memory increase would be associated with the copy's name and its list of variables. All would point to this shared reference pool.

This shared reference is not the behavior we often desire, however.

How can we copy an object so that the copy has its *own* copy of each property?

## Object.assign

We can use [`Object.assign`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) to help solve this problem. Object.assign copies all enumerable and own properties from specified source object(s) to a target object.

We can use it like this:

```javascript
let catObj = { name: 'fluffy', age:5 }
let anotherCat = Object.assign({}, catObj);
anotherCat.age=10;
anotherCat.age==catObj.age; // false
delete catObj.name;
delete catObj.age;
console.log(anotherCat); // {name: 'fluffy', age:10}
```

And there we go, we're done! Well, almost.

## Oops: Nested Objects.

```javascript
let catObj = { name: 'fluffy', age:5, friends: ['larry', 'charlie', 'suzy']}
let anotherCat = Object.assign({}, catObj);
anotherCat.friends.push('ned');
console.log(anotherCat.friends); // ['larry', 'charlie', 'suzy', 'ned']
console.log(catObj.friends); // ['larry', 'charlie', 'suzy', 'ned']
```

What happened?! Object.assign copies property values, and if a property value is itself a *reference* to an object (as it is when we have a value of an array for instance, or another object), the copy is of a *reference* to this value. In order to make a deep copy of a nested object, we have to recurse over all the properties of the object and copy them to a new object.
