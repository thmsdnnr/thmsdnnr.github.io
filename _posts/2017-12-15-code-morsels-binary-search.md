---
layout: post
title:  "Binary Search"
date:   2017-12-15 21:06:04 -0500
description: Binary search algorithm with a test runner implementation, coded in JavaScript.
author: Thomas Danner
lang: en_US
categories: tutorials javascript codeMorsels algorithms
tags: binarySearch, searching, algorithms, javascript, tutorial
---

[Binary search](https://en.wikipedia.org/wiki/Binary_search_algorithm) is an algorithm used to find the position of an item in a sorted array.

Given array `arr` and target `T`, we [divide and conquer](https://en.wikipedia.org/wiki/Divide_and_conquer_algorithm) to search for the index of a target value by iteratively applying the following operations (L and R are the lower and upper bounds of the search space, inclusive):

1. Find the `midpoint` of the subarray `arr.slice(L, R)`
* On the first run of the algorithm, `L=0` and `R=arr.length-1`
* On subsequent runs, we use previous values of L and R as the search space narrows
* Midpoint will be `Math.min((L+R)/2)`
2. Compare `arr[midpoint]` to `T`.
3. If `arr[midpoint]===T` we found the index. Return the index `midpoint`.
4. Else, `T` is either to the "right" (greater than) or the "left" (less than) of the current `midpoint`.
* *Since we assume the array is sorted*, we can determine which half `T` lies in.
  * If `arr[midpoint]>T`, then `T` lies in the lower half
  * Else, `T` lies in the upper half.
5. We adjust the search space accordingly.
* If `arr[midpoint]>T`
  * the new upper index is `midpoint-1`
  * re-use the current iteration's lower index
* If `arr[midpoint]<T`
  * the new lower index is `midpoint+1`
  * re-use the current iteration's upper index

Each loop, check iteration quit conditions:
* If L > R, the item does not exist in the array
* If loop count is equal to the number of elements in the array, the item does not exist in the array (or the input is not sorted)

Before running the loop, we also exclude a few cases in which we cannot reasonably run the search.

* If an array or target is not supplied, or if the array length is <=1
* If the `typeof target` does not equal the `typeof arr[0]`
  * (we already checked that `arr` exists so we know `arr[0]` is accessible)
* If the `typeof arr` is not an array

# aaaand a test runner!

We can also write a simple test runner to check the algorithm's output.

A test runner is basically code that runs code and tells you what happened when it ran. Usually we have expectations about what code will do when it is given certain inputs, so you can supply the runner with these expectations, or <b>expected results</b>. The runner will run the code you provide with these inputs, compare them to the expected results, and then give you a list of code that <b>passed</b> (met the expected result) or <b>failed</b> (did not meet the expected result).

This test runner is simple: we hard-code in the function call, and for each test, we use [`.apply()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) to apply the arguments array to the function.

We record the function outputs and compare them to the expected results for each test.

Then, we'll write a simple function to list out the description for each test, expected and actual results, and a *pass* or *fail* formatting based on whether the tests matched or failed to match expectations.

Click the `result` tab on the codepen below to see it in action.

# codepen

<p data-height="375" data-theme-id="32039" data-slug-hash="rparQr" data-default-tab="js" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Binary Search with Test Runner" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/rparQr/">Binary Search with Test Runner</a> by Thomas (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
