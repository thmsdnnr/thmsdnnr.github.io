---
layout: post
title:  "Bubble Sort"
date:   2017-12-19 10:40:04 -0500
description: Bubble Sort algorithm, coded in JavaScript.
author: Thomas Danner
lang: en_US
categories: tutorials javascript codeMorsels algorithms
tags: bubbleSort, sorting, algorithms, javascript, tutorial
---

[Bubble Sort](https://en.wikipedia.org/wiki/Bubble_sort) is a naive sorting implementation.

We might thing that to sort an array, we could just run through it element by element and, when a subsequent element is less than the previous element, flip the two values.

Set a flag for `swapped`. Each time we flip two values, set the flag to true. After we iterate through the array, if the flag is true, we recursively call the function again & set the value to false.

When we are able to run through the entire array without flipping, we return the sorted array.

This is not an efficient algorithm. Also, [Obama said not to use it](https://www.youtube.com/watch?v=koMpGeZpu4Q).

Consider the worst-case scenario of a completely unsorted array:

`[10, 9, 8, 7, 6, 5, 4, 3, 2, 1]`.

It will take 10 loops to sort the array. Also note a subtle bug in the results below.

Can you figure out why `0 2 11 3 4` sorts to `0,11,2,3,4`? [Here's the run-down](http://thmsdnnr.com/tutorials/javascript/codemorsels/algorithms/2017/12/19/code-morsels-a-subtle-bug-comparisons.html).

### codepen

<p data-height="300" data-theme-id="32039" data-slug-hash="PENEOo" data-default-tab="js" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Bubble Sort" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/PENEOo/">Bubble Sort</a> by Thomas (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
