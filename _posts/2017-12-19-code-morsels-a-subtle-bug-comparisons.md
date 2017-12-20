---
layout: post
title:  "A Subtle Bug Introduced by Comparisons"
date:   2017-12-19 10:55:04 -0500
description: Bubble Sort algorithm, coded in JavaScript.
author: Thomas Danner
lang: en_US
categories: tutorials javascript codeMorsels algorithms
tags: comparisons, bug, QA, testing, sorting, algorithms, javascript, tutorial
---

These demonstrations have all had a subtle bug in them that you might have caught if you tried to enter, say, `1, 2, 3, 11` in the CodePen box. Why not try it now.

(I've left the bug in the bubble sort algorithm for demonstration and implemented the "fix" described herein in the other sorting examples).

### codepen

<p data-height="300" data-theme-id="32039" data-slug-hash="PENEOo" data-default-tab="js" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Bubble Sort" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/PENEOo/">Bubble Sort</a> by Thomas (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

You get `[1, 11, 2, 3]`. What's going on?

We are importing the user input as the *string* value from the input and splitting that string on spaces. So instead of importing `[1,2,3,11]`, we are in fact inputting `['1','2','3','11']`. And in Javascript, `'11'<'2'===true`, even though `11<2===false`.

You might think the answer is just to cast each value to a number, e.g., `arr.map(e=>Number(e))`. But what if the array contains a mix of strings that evaluate to numbers and strings that do not, like `'cats'`? That map will create a lot of `NaN`s.

The answer to me seems to be this:

1. Break the input array into numbers and strings.
  ```javascript
    let numbers=arr.filter(e=>Number.isInteger(Number(e))).map(e=>Number(e));
    let strings=arr.filter(e=>Number.isInteger(Number(e)));
  ```
2. Sort the number and string arrays separately.
3. Concatenate together the sorted results.

Try fixing the CodePen above by filling in steps 2 and 3.

### Bug or Design Decision?

Note that the way in which you choose to compare strings, and whether you opt to sort arrays of mixed type at all, is an implementation decision. There is not a "right" or "wrong" way to do it, but there are confusing ways of doing it.

This is where a language like [TypeScript](https://www.typescriptlang.org/) that is [strongly typed](https://en.wikipedia.org/wiki/Strong_and_weak_typing) can prevent unforeseen bugs, by forcing you to handle type handling in the code itself, rather than making assumptions based on a best guess.
