---
layout: post
title:  "Closures: let vs var"
date:   2017-12-31 10:40:04 -0500
description: Closures are Everywhere. Watch Out.
author: Thomas Danner
lang: en_US
categories: tutorials javascript fundamentals
tags: JS, closures, fundamentals, let, var, scope
---

# Closures: or, these aren't the loop variables you're looking for.

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

What we need to do in order to display the value of `i` in each function call is to pass the value of `i` into the `setTimeout` function. We can do that by wrapping `setTimeout` in an [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE), or an Immediately Invoked Function Expression.

```javascript
for (var i=0;i<5;i++) {
  (function(i){
    setTimeout(function() { console.log(i) }, 1*i);
  })(i);
}
```

`0, 1, 2, 3, 4`.

Okay, but *why* does this work?

### Enter Closures

[Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) combines the context in which a function is called with the function itself. This allows us to access variables declared within the function locally. It allows you to have two functions with variables of the same name. If closures didn't exist, the variables would interfere with one another.

Consider the context in which setTimeout was declared at first. The only variable declaration for `i` was that of the for loop index, which equals 5 when the for loop exits. When we look up the value of `i` in the setTimeout function, it equals 5.

When we wrap the function in an IIFE, we create a new context in which setTimeout will execute. In this context, `i` is equal to the value that we pass the function *within the body of the for loop*. We bound the variable using closure by wrapping it in a function that we immediately execute.

This is closure at work. By defining a new context for each function call in which `i` equals the value of the loop variable *in each loop*, we are able to log those values individually at a later time.

### This Also Works.

```javascript
for (let i=0;i<5;i++) {
   setTimeout(function() { console.log(i); console.log(this); }, 1*i);
}
```

yields `0, 1, 2, 3, 4`. Why?

You might not have considered this, but the way in which you declare a variable affects its scope.

[`var`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var) is scoped to its "execution context": either the function that it is enclosed by or, if it is outside of a function, the global scope.

[`let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let) is block-scoped, which means that when you define a variable using `let`, it is scoped to the block in which it is contained (defined by the `{` and `}`). In this case, it is scoped to the block of the for loop.

[`const`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const) is also block-scoped. It differs from `let` in that its value cannot be changed once assigned (hence the name constant).

Using `let` as our loop variable means the `i` in our `setTimeout` is *automatically* bound to the value in *that iteration* of the loop. A new variable is created every the loop runs, since let gets bound to that loop block's iteration. This is again closure at work.
