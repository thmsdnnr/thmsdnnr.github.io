---
layout: post
title:  "Scope & Closures: let vs var"
date:   2017-12-31 10:40:04 -0500
description: Closures are Everywhere. Watch Out.
author: Thomas Danner
lang: en_US
categories: tutorials javascript fundamentals
tags: JS, closures, fundamentals, let, var, scope
---

# Closures: or, these aren't the loop variables you're looking for.

We'd like to do something simple. Save & output the value of a counter variable after a short delay.

Have a look at this code. What do you think it will output?

```javascript
for (var i=0;i<5;i++) {
  setTimeout(function(){
    console.log(i);
  },1*i);
}
```

`0, 1, 2, 3, 4`, right?

Nope. `5, 5, 5, 5, 5`.

Okay, let's try to fix it.

```javascript
for (var i=0;i<5;i++) {
  setTimeout(function(){
    var loopVar=i;
    console.log(loopVar);
  },1*i);
}
```

Nope, still broken.

What exactly is happening here?

We throw five functions into the [Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop), a queue of callback functions that JavaScript will execute in order when it has time.

When the function executes, it looks for the variable `i`. It finds it, and it finds that `i` is equal to 5, because the for loop has already exited, running much faster than any of the functions within the loop.

What we need to do is to connect the value of `i` somehow to the function we pass to `setTimeout`. We can do that by wrapping `setTimeout` in an [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE), or an Immediately Invoked Function Expression.

```javascript
for (var i=0;i<5;i++) {
  (function(i){
    setTimeout(function() { console.log(i) }, 1*i);
  })(i);
}
```

`0, 1, 2, 3, 4`.

Okay. It works. But *why* does this work?

### Enter Closures

[Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) combine the context in which a function is called with the function itself. It allows us to execute a function outside of the context in which it was defined *and still access that original context*.

You might not have considered this, but the way in which you declare a variable affects its scope.

[`var`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var) is scoped to its "execution context": either the function that it is enclosed by or, if it is outside of a function, the global scope.

Consider the context in which setTimeout callback was declared. The only variable declaration for `i` was that of the for loop index, which equals 5 when the for loop exits. When the setTimeout callback is invoked and it looks up the value of `i` its execution context, `i` is 5.

When we wrap the function in an IIFE, we create a *new context for each callback* that we create. In these contexts, `i` is equal to the value that we pass the function *within the body of the for loop*. We bound the variable using closure by wrapping it in a function that we immediately execute.

This is closure at work. By defining a new context for each function call in which `i` equals the value of the loop variable *in each loop*, we are able to log those values individually at a later time, even when the context in which the function was defined (namely, the for loop with i=0,1,2,3,4) no longer exists.

### This Also Works (block-scoped variables)

```javascript
for (let i=0;i<5;i++) {
   setTimeout(function() { console.log(i); }, 1*i);
}
```

yields `0, 1, 2, 3, 4`. Why?

[`let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let) is block-scoped, which means that when you define a variable using `let`, it is scoped to the block in which it is contained (defined by the `{` and `}`). In this case, it is scoped to the block of the for loop.

[`const`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const) is also block-scoped. It differs from `let` in that its value cannot be changed once assigned (hence the name constant).

Using `let` as our loop variable means the `i` in our `setTimeout` is *automatically* bound to the value in *that iteration* of the loop. A new variable `i` is created every the loop runs, and this variable is bound to the block created by that iteration of the loop.

This block is the execution context of the `setTimeout` callback: hence, when the callback is invoked and looks for `i`, it finds this closure-bound value.

### You don't know JS

[There is an entire book on this topic](https://github.com/getify/You-Dont-Know-JS/tree/master/scope%20%26%20closures), and there are many areas in which scope and closure overlap, or a closure is created to maintain scope for a variable of interest. For instance, *You Don't Know JS* (previous link) suggests that an IIFE is not strictly a use of closure, since the function executes in the same context in which it is defined.

It's good to get familiar with the use of closure in code. It enables things like the Module pattern (we export a function that has a closure over the instance of the module that we import). For instance, [NPM](https://www.npmjs.com/). Get familiar with closure and look for ways you already using it in your code.
