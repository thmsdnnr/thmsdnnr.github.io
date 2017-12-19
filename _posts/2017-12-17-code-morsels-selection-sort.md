---
layout: post
title:  "Selection Sort"
date:   2017-12-17 09:10:04 -0500
description: Selection sort algorithm, coded in JavaScript.
author: Thomas Danner
lang: en_US
categories: tutorials javascript codeMorsels algorithms
tags: selectionSort, sorting, algorithms, javascript, tutorial
---

[Selection sort](https://en.wikipedia.org/wiki/Selection_sort) is an algorithm used to sort an unordered array of items.

Given an unsorted array `arr`, selection sort builds up a sorted array element by element.

For each element in the original array, we:

1. assume element `arr[n]` is the smallest element in the array
2. check every element after this element (indices `n+1` through `arr.length-1`) to see if any element is smaller than `arr[n]`
* if so, we set that element equal to the smallest element in the array
3. If we found an element smaller than `arr[n]`, we swap positions of `arr[n]` with this element
* otherwise, we do nothing and continue to the next element of the array

This can be implemented using two nested for loops and a swap function (switch a & b by holding a in a temporary variable, setting a to b, and setting b to a) to swap out array elements.

The outer for loop performs the "for each element in the original array" in the algorithm statement, and the inner for loop perform the comparison of every other element in the array to this element, checking to see if there is a smaller element that should be swapped to the front.

Give the 'result' tab a click below and try entering some text to see it in action.

### Words < words ?!

Note that items are ordered [lexicographically](https://en.wikipedia.org/wiki/Lexicographical_order) in JS. You could write a custom `compare()` function to use instead of `<` and `>` to implement custom sorting behavior for different types of objects.

### What about sorting direction?

We could sort in descending order, rather than ascending, simply by changing step 1 to sasume that `arr[n]` is the *largest* element in the array, and then searching for any elements *larger*. It's a good exercise to try on your own: fork the CodePen and adjust the assumption (as well as the comparison step in the for loop) to turn this ascending selection sort into a descending sort.

### Edge cases

We throw in two lines to disallow sorting if the array provided is null or if the array length is empty or 1 (an array of length 1 is already 'sorted'):

```javascript
if (!arr) { return new Error('No array supplied to sort.'); }
else if (arr.length<=1) { return new Error('Array length is not sortable.'); }
```

This might seem like a minor step, but as your codebase grows larger, it's essential to have meaningful error messages & ensure functions are called as you expect them to be. Build good habits early.

### codepen

<p data-height="300" data-theme-id="32039" data-slug-hash="jYbmdj" data-default-tab="js" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Selection Sort" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/jYbmdj/">Selection Sort</a> by Thomas (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
