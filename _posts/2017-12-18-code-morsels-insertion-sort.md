---
layout: post
title:  "Insertion Sort"
date:   2017-12-18 20:10:04 -0500
description: Insertion sort algorithm, coded in JavaScript.
author: Thomas Danner
lang: en_US
categories: tutorials javascript codeMorsels algorithms
---

[Insertion sort](https://en.wikipedia.org/wiki/Insertion_sort) is an algorithm used to sort an unordered array of items.

Given an unsorted array `arr`, insertion sort builds up an array of sorted elements "in-place".

Visualize sorting an unsorted `deck` of cards. Take off the first card and put it into a `sorted` pile. Then take the next card `key` off the deck.

Compare `key` with each card in the `sorted` pile from greatest to least, until you find the card which `key` is greater than. Insert `key` there. Continue until `deck` is empty.

Or, in array-speak, for each element `key` in the original array, excluding the first and the last, we:

Test `key` against all preceding elements from indices `(i...0]`
* is `key` smaller than the element?
  * shift `arr[j]` forward by one to `j+1`
    * this duplicates j in the array. the duplicate will be overwritten in the next loop, either with the adjacent smaller element or, if we find the position for `key`, with `key` itself
    * decrease `j` by one
* is `key` larger than `arr[j]` (or are we at the beginning of the array, `j===0`)?
  * we have found the position for `key`! insert `key` at `j+1`

The code looks like this:

```javascript
function insertionSort(arr) {
  for (var i=1;i<arr.length;i++) {
    let key=arr[i];
    console.log(i, key)
    j=i-1;
    while (j>=0 && arr[j]>key) {
      arr[j+1]=arr[j];
      j--;
    }
    arr[j+1]=key;
  }
  return arr;
}
```
### What the...?

It's tough to visualize. Let's take it step by step in sorting `var arr=[3,2,1]` with length 3.

#### i is 1, key is `arr[i] => 2`

`j===0`. `arr[0]===3`.
* `3>2 && j>=0` is true, so we execute the while loop body.
* `arr[0+1]=arr[0]`
* `j--`

The array now looks like this: `[3,3,1]`

`j===-1`. `arr[0]===4`.
* `-1>=0` is false. We do not execute the while loop body.
* Instead, we replace `arr[j+1]` (`arr[0]`) with `key`.

The array now looks like this: `[2,3,1]`

* i++

#### i is 2, key is `arr[2] => 1`

`j===1`. `arr[1]===3`.
* `3>1&&j>=0` is true, so we execute the while loop body.
* `arr[1+1]=arr[1]`
* `j--`

The array now looks like this: `[2,3,3]`

`j===0`. `arr[0]===2`.
* `2>1 && j>=0` is true, so we execute the while loop body.
* `arr[1+1]=arr[1]`
* `j--`

The array now looks like this: `[2,2,3]`

`j===-1`. `arr[0]===4`.
* `-1>=0` is false. We do not execute the while loop body.
* Instead, we replace `arr[j+1]` (`arr[0]`) with `key`.

The array now looks like this: `[1,2,3]`

* i++

#### i is 3. `3<3` is false

We exit the for loop and return the sorted array, `[1,2,3]`.

### codepen

<p data-height="300" data-theme-id="32039" data-slug-hash="aEdGWj" data-default-tab="js" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Insertion Sort" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/aEdGWj/">Insertion Sort</a> by Thomas (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
