---
layout: post
title:  "Quicksort"
date:   2017-12-20 10:40:04 -0500
description: Quicksort algorithm, coded in JavaScript.
author: Thomas Danner
lang: en_US
categories: tutorials javascript codeMorsels algorithms
tags: quicksort, sorting, algorithms, javascript, tutorial
---

[Quicksort](https://en.wikipedia.org/wiki/Quicksort) is an efficient divide-and-conquer sorting algorithm.

We pick a pivot from the array to be sorted. We can pick at random, or choose the beginning, midpoint, or endpoint. In this case, we choose the midpoint, `Math.floor((left+right)/2)`.

Now that we have a pivot, we sort all elements in the array into two groups: one group of elements less than the pivot, and one group of elements greater than the pivot.

We do this with a `partition` function that:

1. Swaps the pivot to the front of the array
2. Iterates from left-to-right (`i++`) and right-to-left (`j--`) through the array.
  * When arr[i] > pivot, we stop, saving position `i`.
  * When arr[j] < pivot, we stop, saving position `j`.
We swap positions `i` and `j`, since they are out of place with respect to the pivot.

When j>i, all of the elements are ordered with respect to the pivot (smaller to the left, larger to the right).

We can now swap the pivot back into the middle of the array and return the index of the pivot.

Quicksort uses this index to recursively sort the smaller & larger subarrays using `partition`.

We initially invoke `qSort` like this: `qSort(arr, 0, arr.length-1)`. As the array reaches its sorted state, the left and right indices grow closer together. When the left index is greater than or equal to the right index, we know the array is sorted, because the pivot is already put in its proper place, and the lengths of the unsorted subarrays on either side of the pivot are zero.

### codepen

<p data-height="300" data-theme-id="32039" data-slug-hash="ppbegj" data-default-tab="js" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Quicksort" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/ppbegj/">Quicksort</a> by Thomas (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
