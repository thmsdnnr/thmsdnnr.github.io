---
layout: post
title:  "The Knapsack Problem"
date:   2018-3-15 06:56:04 -0500
description: Examining the Knapsack Problem and solutions using dynamic programming.
author: Thomas Danner
lang: en_US
categories: compsci
tags: JS, knapsack problem, computer science
comments: true
---

## The Knapsack Problem

You run an import-export company and are packing for a trip. You are allowed one bag of fixed size, and you have more items than can fit in the bag that you wish to bring back with you in order to sell. How can you maximize the value of the items that you fit in the bag? How can we maximize a given quantity subject to a constraint?

## Example Problem

```javascript
const bagSize = 8;
let items = [
  {name: 'cat dish', value: 4, size: 1},
  {name: 'laptop', value: 40, size: 4},
  {name: 'Boxed Set of Knuth', value: 30, size: 8},
  {name: 'assorted snacks', value: 6, size: 2},
  {name: 'catnip', value: 10, size: 1},
  {name: 'clothing', value: 7, size: 5},
  {name: 'diamonds', value: 70, size: 2}
];
let optimalBagContents = knapsackSolver(items, bagSize); // ??
```

Let's write our `knapsackSolver` function.

A naive approach would be to loop over the items and add them to the bag so long as they fit, without considering their values, until the bag is full:

```javascript
function knapsackSolver(items, size) {
  let res = [];
  let currentSz = 0;
  while (currentSz < size && items.length) {
    let newItem = items.shift();
    if (newItem.size+currentSz<=size) {
      currentSz+=newItem.size;
      res.push(newItem);
    }
  }
  return {contents: res, value: res.reduce((acc,b)=>acc+=b.value,0)};
}
```

This gives us
```
​​​​​​​​​​{ contents: ​​​​​
​​​​​   [ { name: 'cat dish', value: 4, size: 1 },​​​​​
​​​​​     { name: 'laptop', value: 40, size: 4 },​​​​​
​​​​​     { name: 'assorted snacks', value: 6, size: 2 },​​​​​
​​​​​     { name: 'catnip', value: 10, size: 1 } ],​​​​​
​​​​​  value: 60 }​​​​​
```

A value of 60 is not bad. But we can see just by looking at the items that we can clearly do better. Filling up the bag too early caused us to miss out on that small bag of diamonds worth 70. If we took diamonds + laptop + catnip + cat dish (70 + 40 + 10 + 4), we'd be much better off, at a value of 124.

So there's the question: how can we make our code figure out what, in this basic case, was easy for us to calculate by hand?

## Solving the Problem

A naive solution would be to generate all possible combinations of the given items, calculate each combination's value, and return the maximum.

To do this, we'd need to calculate the [Power Set](https://en.wikipedia.org/wiki/Power_set) of the items, which is the set of all subsets of the items. This makes sense: if we are naively grouping the items and checking all possible combinations, we're not excluding cases where the pack isn't full. So, we have one group with each item individually. One group with item one and item two, item one and item three, item one and item four, etc. etc.

The Power Set is on the order of 2^n, or an **exponential** runtime. This is completely prohibitive for solving any but the most trivial of problems. It's not uncommon to have a set of 100 items you need to maximize. Assuming those items were merely 1 byte each (numbers between 0-255), the operation would take 2^30 bytes, or just over a gigabyte of memory.

Clearly we need a way to generate combinations relative to the constraints *as* we construct the solution, not after the fact. [Dynamic Programming](https://en.wikipedia.org/wiki/Dynamic_programming) is a paradigm that allows us to do this: we can use the solution to previously calculated problems to dictate the space over which we search for the next steps in the solution.

How does this apply to the knapsack problem?

At a high level, how do we go about filling the knapsack? We iterate through the items and look at the current state of the knapsack. We ask ourself first:

1. Will the item fit? If not, skip it.
2. If so, does adding it add more value, *given the current size of the knapsack*? If so, add it. If not, skip it.

We are asking the question:

1. For a given size of the knapsack, iterating over each item, which item provides the most value relative to the size increase it creates? Then, of these items, which is permissible to add ()

## The Nuts and Bolts

We'll solve the problem by creating a grid with a width equal to the weight the knapsack can hold, and a height equal to the number of items we have to put in the sack.

When the knapsack is full and we've run through all of our items, the maximum value will be the last row and last column of the array.

We can then iterate through our "Did we pick the item up?" matrix to answer the questions of *which* items constitute this maximum value.

```javascript
const bagSize = 8;
let items = [
  {name: 'cat dish', value: 4, size: 1},
  {name: 'laptop', value: 40, size: 4},
  {name: 'Boxed Set of Knuth', value: 30, size: 8},
  {name: 'assorted snacks', value: 6, size: 2},
  {name: 'catnip', value: 10, size: 1},
  {name: 'clothing', value: 7, size: 5},
  {name: 'diamonds', value: 70, size: 2}
];

let values = [0, ...items.map(e=>e.value)];
let weights = [0, ...items.map(e=>e.size)];
const N = items.length;

const arrayGen = (m, n) => {
  let M = new Array(bagSize+1);
  M[0]=new Array(bagSize+1).fill(0);
  for (var i=1;i<=N;i++) { M[i] = new Array(bagSize); }  
  return M;
}

let M = arrayGen(bagSize, N);
let keepArr = arrayGen(bagSize, N);

for (var i=1; i<=N; i++) {
  for (var j=0; j<=bagSize; j++) {
    if (weights[i] > j) { // we can't add
      M[i][j] = M[i-1][j];
      keepArr[i][j]=0; // we can't keep this item
    }
    else {
      const valueIfLeft = M[i-1][j];
      const valueIfTaken = M[i-1][j-weights[i]] + values[i];
      if (valueIfTaken >= valueIfLeft) { // maximize the value
        M[i][j] = valueIfTaken;
        keepArr[i][j]=1; // we can keep this item
      } else {
        M[i][j] = valueIfLeft;
        keepArr[i][j]=0;
      }
    }
  }
}
let keptElements=[];
let eleKey = bagSize;
for (var i=N; i>0; i--) {
  console.log(i, eleKey);
  if (keepArr[i][eleKey]==1) {
    keptElements.push(i-1);
    eleKey -= weights[i];
  }
}
let optimalBagContents = knapsackSolver(items, bagSize); // ??
```

## Knapsack Problem Variants

We just solved the 0-1 knapsack problem: items in the collection exist either zero or one times. We don't have two cat dishes or two laptops.

There are two other variants:

* bounded knapsack problem: there can be more than one of a given item in the collection, but no more than some max quantity (bound)

`{name: 'cat dish', value: 4, size: 1, qty: 4}`

* unbounded knapsack problem: there can be any number of given items in the collection, so long as they are integral and non-negative

Allowed: `{name: 'cat dish', value: 4, size: 1, qty: 1e6}`


Not allowed: `{name: 'cat dish', value: 4, size: 1, qty: 2.3}` or `{name: 'cat dish', value: 4, size: 1, qty: -4}`

## Multidimensional Knapsacks

To imagine a multidimensional version of the problem, pretend you're UPS, and you're trying to distribute boxes to trucks. Each truck has a fixed volume and a fixed max weight capacity. Each package has a shipping cost that is not necessarily directly proportional to the volume and weight of a given box. Some small, lightweight boxes might have tons of insurance -- like a bag of diamonds. Some boxes might have cost more for expedited shipping. Other boxes might be very volume-heavy in proportion to the size and weight of their contents (like when you get a huge box from Amazon with big air cushions that only contains a single thing).

## TIL

Dynamic programming is a technique that uses [memoization](http://thmsdnnr.com/tutorials/javascript/fundamentals,/2018/01/24/fundamentals-memoizing.html), allowing us to solve problems much more quickly and efficiently than we might otherwise have been able.
