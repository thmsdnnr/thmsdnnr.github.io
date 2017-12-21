---
layout: post
title:  "Binary Insertion Sort"
date:   2017-12-19 09:50:04 -0500
description: Binary Insertion Sort algorithm, coded in JavaScript.
author: Thomas Danner
lang: en_US
categories: tutorials javascript codeMorsels algorithms
tags: binaryInsertionSort, sorting, binarySearch, algorithms, javascript, tutorial
---

[Binary Insertion Sort](https://en.wikipedia.org/wiki/Insertion_sort) is a more performant Insertion Sort implementation.

You put a Binary Search in your Insertion Sort so you can Binary Search where to insert.

Given an unsorted array `arr`, insertion sort builds up an array of sorted elements "in-place", checking *every* previous value in the sorted section of the array to determine where to place a new element.

Envision the partially-sorted array: `[1,3,4,0]`.

Imagine that we are placing `0`, the last element. Insertion sort would have us compare 0 with 4--nope, still bigger!--then with 3, and then with 1.

We can use a binary search technique with this sorted part of the array, `[1,3,4]` in order to divide-and-conquer to find the index of insertion so that we do not have to make as many comparisons.

### Tiny Modification to Binary Search Algorithm

The code is nearly identical. We rewrite our [binary search](http://thmsdnnr.com/tutorials/javascript/codemorsels/algorithms/2017/12/15/code-morsels-binary-search.html) algorithm slightly.

Previously, our search returned `false` if the element was missing from the array:
```javascript
if (L>R) { return false; }
```

Now, we return the `index` of *where the element would exist, if it did exist*:
```javascript
if (L>=R) { return (target > arr[L]) ? L+1 : L; }
```

### Tiny Modification to Insertion Sort Algorithm

We rewire our [insertion sort](http://thmsdnnr.com/tutorials/javascript/codemorsels/algorithms/2017/12/18/code-morsels-insertion-sort.html) to use the index returned by our new Binary Search function.

Previously, we checked every preceding value to find the index.
```javascript
while (j>=0 && arr[j]>key) {
  arr[j+1]=arr[j];
  j--;
}
arr[j+1]=key;
```

Now, we use the index provided by binary search:
```javascript
let idx=binarySearch(arr, key, 0, j)
while (j>=idx) {
  arr[j+1]=arr[j];
  j--;
}
arr[j+1]=key;
```

This might look like the same number of loops. While it is true that we still have to *shift each element forward* by one until we get to the index of insertion for `key`, we do not pay for the additional comparison of `arr[j]>key` in each while loop iteration.

This might seem trivial in a small case, but if you imagine sorting a very large array, or an array of items that are larger and more expensive to compare than numbers, it is worth the time savings.

### Let's walk through step by step.

`arr=[1,3,4,0]`. Where do we insert 0?

`target=0`. `L=0`, `R=2`.
Midpoint is 1 (`Math.floor((0+2)/2)`). `arr[1]===3`.

0 is less than 3, so `L=0` & `R=0`.

Now `L>R` is true. We evaluate the last statement:

`return (target > arr[L]) ? L+1 : L;`

`arr[0]` is 1, and `0>1` is false. We insert `0` at index `0`, *before 1*, instead of at index `1`.

We return the sorted array `[0,1,3,4]`.

#### This last line confuses me.

Me too, at first. `L>R` is true when the item does not exist in the binary search space. However, we know that if the element *were* in the search space, it would be *adjacent* to `arr[L]`.

**We need to know *on which side* of `arr[L]` to insert `target` so that the array is sorted.**

Consider if we were placing `2` in the array `[1,3,4,2]`.

`target=2` `L=0`, `R=2`.
Midpoint is 1 (`Math.floor((0+2)/2)`). `arr[1]===3`.

2 is less than 3, so L is 0 and R is 0.

Now `L>R` is true. We evaluate the last statement:

`return (target > arr[L]) ? L+1 : L;`

`arr[0]` is 1, and `2>1` is false. We insert `2` at index `1`, *after 1*, instead of at index `0`.

We return the sorted array `[1,2,3,4]`.

### codepen

<p data-height="300" data-theme-id="32039" data-slug-hash="wpMVQo" data-default-tab="js" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Binary Insertion Sort" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/wpMVQo/">Binary Insertion Sort</a> by Thomas (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
