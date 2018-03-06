---
layout: post
title:  "Prototypal Inheritance for Beginners"
date:   2018-3-5 06:56:04 -0500
description: An introductory discussion of prototypal inheritance in Javascript, covering the new keyword, Object.create, and ES5 class syntax.
author: Thomas Danner
lang: en_US
categories: fundamentals
tags: JS, fundamentals, inheritance, objects, classes, prototypes, inheritance, new
comments: true
---

## Properties From Thin Air?

When you define an object in Javascript, it has properties.

```javascript
const cat = {}
cat.speak = () => "Meow!";
cat.speak(); // Meow!
```

Here we have a cat object. We define and call a speak property. We get what we expect. Meow!

Now let's experiment. What do you think will happen when we run this code?

`cat.hasOwnProperty('speak');`

It returns true. But how? We didn't define `hasOwnProperty` for cat. Where did that function come from?

`console.log(cat.__proto__); // {}`

Can you guess what's happening here?

## Inheritance Through The Prototype Chain

What is inheritance? Receiving things from your parents you didn't create for yourself.

You might be able to override some of those things (cut your hair, wear glasses or contacts, spend your trust fund), but your parents set that initial state.

Things that we do not explicitly alter end up staying the same as when we inherited them.

Unless we override behaviors, we become our parents.

## Walking Up the Chain

When we attempt to access a property on an object, we first check that object to see if the property exists. If it does, we return the value.

If it does not, we access the object's parent (prototype) and check all of *its* properties to see if the property exists there. If it does, we return the value. If not, we repeat the process, until we finally reach a parent of null and return undefined.

In the case of `cat`, `hasOwnProperty` did not exist on the object we defined. However, `cat` is an Object, and so its prototype is the Object object. This makes sense: the blueprint for building a new object is an object! The [Object object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) has the `hasOwnProperty` method, and so the method is executed.

You might be scratching your head: okay, the cat did not have the property defined, so we looked at its prototype, the Object. We found the method there. But how come the method returned `true` when we called it?! Objects do not have a `speak` method!

`Object.speak()` // Uncaught TypeError: Object.speak is not a function

This (no pun intended) is a key element to prototypal inheritance. If a function does not exist as a property of a given object, when it is found on a parent object and executed, it is executed *in the context of the inheriting object*.

> When an inherited function is executed, the value of this points to the inheriting object, not to the prototype object where the function is an own property. [MDN: Inheriting Methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain#Inherting_methods)

Here's a function we can use to explore the methods on a given prototype chain.

```javascript
function logProtoChain(thing) {
  console.log('the things properties: ', Object.getOwnPropertyNames(thing));
  let thisProto=Object.getPrototypeOf(thing);
  while (thisProto!==null) {
    console.log('prototype properties:', Object.getOwnPropertyNames(thisProto));
    thisProto=Object.getPrototypeOf(thisProto);
  }
  console.log(thisProto);
}
```

#### logProtoChain([])
prototype chain: Array -> Object -> null

`["length"]`
##### The thing's properties

`["length", "constructor", "concat", "pop", "push", "shift", "unshift", "slice", "splice", "includes", "indexOf", "keys", "entries", "forEach", "filter", "map", "every", "some", "reduce", "reduceRight", "toString", "toLocaleString", "join", "reverse", "sort", "lastIndexOf", "copyWithin", "find", "findIndex", "fill"]`
##### Array methods

`["constructor", "__defineGetter__", "__defineSetter__", "hasOwnProperty", "__lookupGetter__", "__lookupSetter__", "isPrototypeOf", "propertyIsEnumerable", "toString", "valueOf", "__proto__", "toLocaleString"]`
##### Object methods

and then null.

#### logProtoChain("fluffykins")
prototype chain: String -> Object -> null

`["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "length"]`
##### The thing's properties

`["length", "constructor", "anchor", "big", "blink", "bold", "charAt", "charCodeAt", "codePointAt", "concat", "endsWith", "fontcolor", "fontsize", "fixed", "includes", "indexOf", "italics", "lastIndexOf", "link", "localeCompare", "match", "normalize", "padEnd", "padStart", "repeat", "replace", "search", "slice", "small", "split", "strike", "sub", "substr", "substring", "sup", "startsWith", "toString", "trim", "trimLeft", "trimRight", "toLowerCase", "toUpperCase", "valueOf", "toLocaleLowerCase", "toLocaleUpperCase"]`
##### String methods

`["constructor", "__defineGetter__", "__defineSetter__", "hasOwnProperty", "__lookupGetter__", "__lookupSetter__", "isPrototypeOf", "propertyIsEnumerable", "toString", "valueOf", "__proto__", "toLocaleString"]`
##### Object methods

and then null.

Try calling `logProtoChain` on a function. Yes, functions *are objects*.

prototype chain: Function -> Object -> null

`["length", "name", "arguments", "caller", "constructor", "apply", "bind", "call", "toString"]`
##### Function methods

## Property Shadowing / Method Overriding

Continuing the above example, what do you think will happen if we do this:

```javascript
cat.hasOwnProperty = () => false;
cat.hasOwnProperty('hasOwnProperty'); // false
```

Since we no longer have to look up the prototype chain from cat to find `hasOwnProperty`, we return the `hasOwnProperty` defined on the object itself. For comic relief, it's defined in a manner that is both logically true and semantically false.

## Assigning Prototypes

We've seen how when we create an array or a string, the language automatically assigns a prototype that provides the object with handy methods.

The real power comes from defining our own prototypes to create object hierarchies that can inherit from each other in meaningful ways.

How can we define our own prototypes for a given object?

There are three ways:

1. [New operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new)
2. [`Object.create()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
3. [Class syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

Onward!

### 1. New

```javascript
function Animal(name, age, greeting) {
  this.name=name || "anon";
  this.age=age || "4";
  this.greeting=greeting || "hello good sir";
}
Animal.prototype.greet = function() { return this.greeting; }
```
##### Our Animal constructor

We can create a new object by using the new keyword. New is nothing magical.

Let's write our *own* new function, then, so that we can understand it. New does a few things:

1. Creates a new object.
2. Sets the object's prototype to that of the constructor function's prototype.
3. Calls the constructor function with the given arguments, setting `this` to the context of the newly-created object. See the MDN docs for [`.apply()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply).
4. Returns the new object or, if the constructor function returned a non-null object, this returned object.

```javascript
function makeNewObject(baseObj, ...args) {
  let newObj = {};
  newObj.__proto__=baseObj.prototype;
  let construct=newObj.constructor.apply(newObj, args);
  return construct ? construct : newObj;
}
```

```javascript
let cat = new Animal('kevin', 33, 'meow');
cat.greet(); // meow
let ourNewCat = makeNewObject(Animal, 'kevin', 33, 'meow');
ourNewCat.greet(); // meow
```

These two are equivalent.

Note that if we change the cat's greeting, the greeting function is updated accordingly. As discussed before, `this` is bound to the object instance.

```javascript
cat.greeting = 'ROAR I AM A LION!!';
cat.greet(); // ROAR I AM A LION!!
```

### 2. Object.create

[`Object.create()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create) takes a prototype object and returns an object with this prototype.

```javascript
const cat = {
  greeting: () => "meow",
  whereDoYouSeeYourself: function (years=5) {
    return 'In '+years+' years, I will be sitting on a windowsill watching squirrels.';
  }
};

const fluffykins = Object.create(cat);
fluffykins.whereDoYouSeeYourself();
// In 5 years, I will be sitting on a windowsill watching squirrels.
```

### 3. Class Syntax

ES5 introduced [class syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) that is an overlay of prototypal inheritance.

```javascript
class Point {
  constructor(pt) {
    if (!pt) { return; }
    this.x = pt[0];
    this.y = pt[1];
  }
  static computeQuadrant(x, y) {
    if (x==0||y==0) { return null; } // quadrant boundary
    else {
      /* 1st q: x>0 y>0, 4th q: x>0, y<0
         2nd q: x<0 y>0, 3rd q: x<0 y<0 */
      if (x>0) { return y>0 ? 1 : 4; }
      else { return y>0 ? 2 : 3; }
    }
  }
  get quadrant() { return Point.computeQuadrant(this.x, this.y);  }
}

class Line extends Point {
  constructor(startPt, endPt) {
    super();
    this.a = new Point(startPt);
    this.b = new Point(endPt);
  }
  get length() { // Pythagoras
    return Math.sqrt(Math.pow((this.b.x-this.a.x),2)+Math.pow(this.b.y-this.a.y,2));
  }
  get startQuadrant() { return this.a.quadrant; }
  get endQuadrant() { return this.b.quadrant; }
}

let Q = new Line([-1,-1], [5, -1]);
Q.length; // 6
Q.startQuadrant; // 3
Q.endQuadrant; // 4
```

[`static` methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static) are functions that can be called only on the class for which they are defined, not by any of the individual instances of the class.

```javascript
let P = new Point([3, 3]);
P.computeQuadrant(); // Error: P.computeQuadrant is not a function
P.quadrant; // 1
```

It is worth taking a look at the MDN docs for classes, since there's a bit more nuance here.

## Updating Prototypes

What do you think will happen if we add to the Animal prototype after we've already created objects based off of it?

```javascript
let cat = new Animal('kevin', 33, 'meow');
cat.greet(); // meow
Animal.prototype.sayGoodbye = function() { 'Farewell, human.'; }
cat.sayGoodbye(); // ?
```

Well, think about what happens when a property doesn't exist on an object. We check the prototype. Did we define `sayGoodbye` on the prototype? Yes! So even objects that are already created will have the `sayGoodbye` method. Since we are looking up a method rather than recreating it with each new instance of an object, it is far more memory-efficient.

If necessary, objects can override parent behavior on an instance-level:

```javascript
cat.sayGoodbye = () => "I don't know why you say goodbye; I say hello!";
cat.sayGoodbye(); // Beatles lyrics
```

## Performance

Prototype chain lookup can be expensive: the farther up you have to look, the longer the code will take to run. This can be a factor with long chains or a large number of objects. Also, whenever you attempt to look up undefined values, the entire chain is traversed (we have to look *everywhere* until we reach a null prototype).

## Monkey-patching (never ever)

It is a bad idea to extend existing prototypes. For example, don't redefine `Object.prototype.hasOwnProperty=()=>false`. If you require special behavior, override existing methods in a class that inherits from the parent class. Otherwise, you break encapsulation and defeat the purpose of inheritance altogether (not to mention incur the wrath of your fellow programmers who expect built-in language contracts to obtain).

## TIL

Prototypal inheritance is fundamentally pretty simple. Key takeaways:

1. Blueprints within blueprints. A chain of lookups, stopping at the first value we find, or null, whichever comes first.
2. If we do not explicitly set a prototype, the language will do it for us based on variable type (string, array, function, etc.).
3. Almost every object has Object object as its highest-level prototype.
4. Whenever we find a method in an object lookup, any `this` references in that method refer to the *inheriting object*, not the object on which the method is found.
5. Three ways to harness: new, Object.create, and class syntax.

Gaining an understanding of the prototype chain is key to leveling up your understanding of Javascript. There is a ton more nuance than discussed here, but if you understand this content, you're well on your way to more advanced applications.
