---
layout: post
title:  "Stacks, Queues, & A Simple Code Linter"
date:   2018-1-30 08:56:04 -0500
description: An introduction to the stack and queue data structures with examples.
author: Thomas Danner
lang: en_US
categories: tutorials javascript fundamentals
tags: JS, stack, queue, big-O, complexity, dataStructures
comments: true
---

<p data-height="243" data-theme-id="32039" data-slug-hash="bLNzbd" data-default-tab="js,result" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Code Linter: Mismatched Brackets" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/bLNzbd/">Code Linter: Mismatched Brackets</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
##### a cool thing we can build with stacks: a code linter!

## Stacks and Queues, Fundamental Data Structures

Today we're going to talk about stacks and queues, two common and incredibly useful data structures in computer science.

Data's the raw stuff of your computer. Types of data include a text file, an image, a string, an integer, etcetera. All of these pieces of data are ultimately boiled down to a sequence of zeroes and ones in memory. Imagine the task of trying to parse out of these zeroes and ones what the bits represent if we did not have way of classifying and organizing them? It would represent an impossible task.

Certain applications value different factors (lookup speed, addition speed, etc.). Hence, *data structures are choices*. We have to structure our data in meaningful ways, not only so that we can store it in memory, but also so that we can efficiently perform operations on the data that we find most important.

Let's look at some concrete examples.

### Stacks: Last In, First Out: LIFO.

Stacks. Stacks of books, stacks of plates. To add an item to the stack, you put it on top. To remove it, you take one off the top. You can't take one from the middle or the bottom -- the plates will fall and shatter! Stacks follow the Last In First Out discipline, which means just what it sounds like: the last thing you put on a stack is the first thing you get out of it.

> Stacks consist of a sequentially ordered list of elements of the same type. You can only add and remove elements from one end, and you can only remove the elements in the order of their addition (most recent first).

### Queues: First In, First Out: FIFO.

Lines at the grocery store. A row of cars at a stoplight. A four-way stop. In a queue, new items are added at the end of the list (an operation known as "enqueuing") and removed from the front of the list (an operation known as "dequeuing").

> Queues consist of a sequentially ordered list of elements of the same type. Operations take place on both ends, but you can only add items to the end of the queue (enqueue) and remove items from the beginning of the queue (dequeue).

## A four-way stop

Let's do a little thought experiment and implement a four-way stop as a stack and a queue.

If you aren't familiar with a four way stop, it is when all drivers who reach the intersection must stop and then yield to all other drivers who reached the intersection and stopped before them.

Let's implement the four-way stop as a **queue**. One driver rolls up, followed by another soon after. The first driver gets to leave first: he got there first. First in, first out.

If drivers A, B, C, and D arrive at different sides of the intersection, in this order, then B, C, D yield to A, C and D yield to B, D yields to C, and D goes. First in, first out.

Imagine if we implemented a four-way stop as a **stack**. One driver would roll up, then another, then another, until finally the last driver arrives. The last driver to arrive would be the first to leave, followed by the next-to-last, until finally we get back to the poor first driver, who's had to watch three other cars come and go before he can go.

Oops! We can see from these examples how a either a stack or a queue is the obvious chose for modeling real-world situations.

## Code linting

Let's get more abstract and write a quick code linter. [Code linters](https://en.wikipedia.org/wiki/Lint_(software)) are scripts that you can run on your code to make sure it follows a set of stylistic constraints (such as number of indentations, no unused variables, naming conventions, etc.).

We'll write a linter to ensure that opening and closing brackets are matched.

In programming, you can use parentheses and brackets (both curly and flat) to separate statements or blocks of code. There is often no restriction to the degree of nesting: `(((((2+2)))))` would be considered valid, for instance.

However, the thing you cannot do is have something like this: `(2+2]` or this: `((2+2)`, in which the opening and closing bracket types are mismatched, either in type or number.

We can use one of our new data structures to detect invalid use of brackets.

### Stack or Queue?

Which type of data structure do you think it would make sense to use? Consider what we're looking for: in a line of code, we take the *Last* bracket and compare it with the next bracket that we encounter in the line to see if the brackets match. If they do, we continue. If they don't, we've found a mismatch. The *Last* bracket that we encounter is the *First* one that we compare.

Too obvious? Well, a stack is a great structure to use to solve this problem. Here's the strategy:

1. Iterate through every character in the line of code. If the character is not an opening or closing bracket, continue. If the character is an opening bracket `(,[,{`, push it onto a stack.
2. If the character is a closing bracket `),],}`, pop the first bracket off the stack.
3. If the first bracket exists AND matches the closing bracket, continue parsing.
4. If the first bracket does NOT exist, or if it does not match the closing bracket, there is a mismatch.
5. If we have reached the end of the code and there are still items in the stack, there are too many opening brackets and not enough closing brackets. Record these instances.

World, I give thee: bracketLint()!

```javascript
function bracketLint(string) {
  const openersToClosers = {
      '[':']',
      '(':')',
      '{':'}'
    };  
    const closersToOpeners = {
      ']':'[',
      ')':'(',
      '}':'{'
    };  
    let lines=string.split(/\n/);
    let stack=[];
    let errors=[];
    lines.forEach((line,idx)=>{
      line=line.split("");
      for (var i=0;i<line.length;i++) {
        if (openersToClosers[line[i]]) { //opening bracket
          stack.push(line[i]);
        } else if (closersToOpeners[line[i]]) { //closing bracket
          //check to see if the closer matches the last opener
          let firstOut=stack.pop();
          if (!firstOut||openersToClosers[firstOut]!==line[i]) {
            errors.push(
              { line:idx+1,
                charIdx:i,
                missing:openersToClosers[firstOut]||closersToOpeners[line[i]]
              }
            );
          }
        }
      }
    });
  while (stack.length) {
    popElement=stack.pop();
    errors.push({line:lines.length, charIdx:0, missing: openersToClosers[popElement]});
  }
  return errors.length ? errors : true;
}
```

Let's walk through this code. First, we define a simple mapping from opening brackets to closing brackets and vice versa. This way, when we consider a bracket, we know its partner. We can also use the dictionaries to test the characters in the string that we parse for matching a bracket.

We take in the string we've been passed. We break the code up into an array with elements corresponding to lines. For each line, we break it into another array with elements at the character level. Then we do our parsing.

If we have an opening bracket, we push it on the stack! If we have a closing bracket, we [`.pop()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/pop) the last-encountered opening bracket off the stack. We look it up in our dictionary to see what closing bracket we expect, and then we compare our expectation with the actual bracket we encountered.

If they match, no problem! If not, there's an error. We push an error object into our errors array with the line (incremented by one, since line-numbers are not zero-based), character, and "missing closing bracket". We'll return this array at the end so the user knows where to go into the code to fix the error.

We finish running through all the lines, but we're not done quite yet. If the stack isn't empty, this means that there are not enough closing brackets (even if we've matched all our openers and closers thus far). Consider this: `[[]`. There's an extra `[` that never gets closed. To handle this case, we run a while loop over `stack.length` and pop each of the remaining items off the top.

We know that each one is an error because there's nothing else left in the code to close it out with, so we record each instance.

Similarly, we have to consider the case where the stack is empty, but we encounter a bracket. This means that there was not an opening bracket for the closing bracket. We record each instance, noting the missing partner of this extra closing bracket.

Once we've accounted for all the brackets, we return the array of errors, or an empty array if none exist.

We can invoke our bracketLinter with test strings like so:

```javascript
const testValidString=`function heyThere() {
  console.log('lots and lots of cats');
  return true;
}`;
const testInvalidString=`function heyThere()
  {
    console.log('lots and lots of cats'];
    return true;
  }`;

bracketLint(testValidString);
// []

bracketLint(testInvalidString);
// [{line: 3, charIdx: 39, missing: ")"}]
```

## Wrapping up

Stacks and queues are incredibly practical. Try to look around in the world and imagine some examples of stacks and queues you encounter every day. Consider how you can make your code more efficient by leveraging these structures. For a good, more in-depth article on stacks and queues, [go here](https://www.cs.cmu.edu/~adamchik/15-121/lectures/Stacks%20and%20Queues/Stacks%20and%20Queues.html).

### Abstract Data Types and Implementations

Note that we have specified these data structures but nothing about their implementations. Strictly speaking, stacks and queues as discussed are [Abstract Data Types](https://en.wikipedia.org/wiki/Abstract_data_type), or implementation-independent structures. We'll get more in depth on **how** you actually implement a stack and a queue in memory soon, but it gets a little complex. This article covers the basics, helps build a foundation, and motivates further study of the nitty gritty details of what's going on under the hood.
