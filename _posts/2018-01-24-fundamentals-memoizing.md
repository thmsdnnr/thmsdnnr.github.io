---
layout: post
title:  "Memoization — Hey, I've Already Calculated That!"
date:   2018-1-24 08:56:04 -0500
description: An introduction to memoization in JS, with a factorial example and a breakdown of when and where to consider memoizing.
author: Thomas Danner
lang: en_US
categories: tutorials javascript fundamentals,
tags: JS, memoize, memoization, efficiency, complexity, optimization
comments: true
---

### Memoization: or (You) Call(ed) Me (Before), Maybe

[Memoization](https://en.wikipedia.org/wiki/Memoization) is a simple yet powerful technique to optimize the performance of your code.

Memoization gives a function a memory: if the function has already been called with a given set of inputs, it remembers its output from the previous call and returns that output directly, without recalculating a known result.

Think of it like a dictionary for your function. The combination of parameters are a word, and a given output is a definition. By building up a dictionary of terms, you save time having to scrounge around for the definition (calculating it from scratch).

Say you have an expensive function. What's expensive? Anything that takes longer time to calculate than it does to look up an answer for. Let's go with the Factorial function.

`N!` is defined as `N*(N-1)*(N-2)...*1`. So `4!` is `4*3*2*1`.

`0!` is defined as 1.

We can code factorial using recursion.

```javascript
function factorial(N) {
  if (N===0) { return 1; }
  else return N*factorial(N-1);
}
```

This works fine, but it's a lot of work for the computer. If we call factorial(200), we've got to push 200 things onto the stack, *every time we call the function*. What if we only had to do that once, and henceforth we could simply look up the answer in a dictionary?

### Memoize That!

```javascript
var memoizedFactorial = function(N) { this.dict={}; }

memoizedFactorial.prototype.calculate = function(N) {
  let key='k_'+N;
  return this.dict[key] ? this.dict[key] : this.dict[key]=factorial(N);
  function factorial(N) { return N===0 ? 1 : N*factorial(N-1); }
}

memoizedFactorial.prototype.showDict = function() { return this.dict; }
```

We prepend `k_` to the function parameter N, since JS does not allow integer dictionary keys (they get cast to Strings). Might as well avoid implicit type conversion and keep things clean.

We can use our new memoized factorial thusly:

```javascript
let Fact=new memoizedFactorial;
Fact.calculate(4); // 24
```

Okay, great...but I hear what you're thinking. Why bother with all of this? It's not that much faster! Well, imagine you have some loop that's going to need the value of 0 through 9 factorial many times over.

```javascript
for (var j=0;j<10000;j++) {
  for (var i=0;i<10;i++) {
    Fact.calculate(i);
  }
}
```

If you tried to perform this calculation without memoization, you'd be wasting a ton of extra CPU cycles. With memoization, we just look up the value. Another benefit of memoization with a recursive function is you're less likely to blow the stack. After calculating N factorial, for say N=10, we only have to calculate 10 more times to get 20! (we already have 10!, 9!, 8!...) memoized.

### Memoize All The Things!

So let's just write a wrapper to memoize everything!

Nope. There's two reasons why this is a bad idea.

#### Memoization requires *pure functions*.

What's a pure function? A function that returns the same output given the same input. It has no side effects, like getting the current time when called and multiplying that by one of its arguments.

If our function isn't pure, returning a previous answer is likely to be incorrect. Say we have a function `minuteMultiplier` that returns the result of a number multiplied by the current minute.

We *could* memoize the function, but we'd have to do it *not only* based on the arguments, but also on the current state of the function when called. In this case, memoization would likely be more trouble than it's worth, since you're already looking up the current time in order to determine *where* in your dictionary to search for a previous result. Instead of looking up the result, just return it.

Furthermore, lots of valuable functions have side effects (manipulating the DOM, querying the database, etc.). We do not want to memoize these results as they are dynamic, and any caching would be at best stale, at worst inaccurate and misleading.

#### Time vs Space Tradeoffs

There's always a tradeoff here. When we're storing numbers, no big deal. But imagine our function returns a very large object that is expensive to store in memory. In this case, memoization is a tradeoff between CPU cycles and storage space. This is often application-specific, but it's reason enough to not be cavalier in caching your data.

As always, a little bit of thought can save a lot of work spent on undoing poor design decisions.
