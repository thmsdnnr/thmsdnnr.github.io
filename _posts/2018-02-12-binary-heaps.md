---
layout: post
title:  "Binary Heaps: Min and Max"
date:   2018-2-12 06:56:04 -0500
description: A discussion of binary heaps (min and max heaps), a binary tree data structure that can be used to implement a priority queue.
author: Thomas Danner
lang: en_US
categories: tutorials compsci
tags: JS, binaryHeap, queue, big-O, complexity, dataStructures, binaryTree
comments: true
---

## Binary Heaps
> I owe much of my learning & implementation to [*Eloquent Javascript*](http://eloquentjavascript.net/1st_edition/appendix2.html) by Marijn Haverbeke.

[Binary heaps](https://en.wikipedia.org/wiki/Binary_heap) are partially ordered data structures in the form of binary trees. A binary heap orders elements such that parent nodes are either greater than/equal to their child nodes (max heap), or less than/equal to their child nodes (min heap).

A heap is a *full* binary tree (every node except leaves have two children) and *complete* binary tree (every level except possibly the last is filled; all nodes are as far left as they can be).

```
min-heap:
      1
    /   \
   2     3
 /  \   / \      
4   11

max-heap:
      11
    /    \
   4      3
 /  \    / \      
2    1   
```

Note that a binary heap is *not* a Binary Search Tree.

```
   1
 /   \
3     2
```
##### This is a valid min-heap, but an invalid binary search tree.
```
   1
 /   \
2     3
```
##### This is a valid min-heap that also happens to be a binary search tree.

In a Binary Search Tree, the left child is less than the parent and the right child is greater than the parent. In a Binary Heap, there is no ordering of the children relative to the parent other than children being strictly less or strictly greater than the parent.

## Big-O Breakdown

Binary Heaps have O(log n) complexity for insertion and deletion in the worst case, and O(1) complexity for removal (of the min or max element).

To insert an element, we add it to the very end of the tree and then swap its position with adjacent nodes until it satisfies the ordering property of the heap.

We can delete a specified element from the heap in O(log n) time by performing a binary search for the element and then removing it. However, min and max heaps are optimized for removal of the root element in O(1) time, rather than supporting arbitrary deletion.

## Implementing Heaps

Heaps are [often implemented using an array](https://en.wikipedia.org/wiki/Binary_heap#Heap_implementation). When implementing a heap as an array, we're creating what's called an [implicit data structure](https://en.wikipedia.org/wiki/Implicit_data_structure). Whereas in a traditional binary tree implementation

Given a tree array with index starting at 1, for any element with index idx, its parent is located at `Math.floor(idx/2)`, and its two children are located at `2*idx` and `2*idx+1`.

```
min-heap:

        1
      /   \
     2     3
   /  \   / \      
  4   11

1-indexed array:
values:  x  1  2  3  4  11
indices: 0  1  2  3  4   5

0-indexed array:
values:  1  2  3  4 11
indices: 0  1  2  3  4
```

We could start the array at index 0, but it would make the math just a little uglier.

If we did use a zero indexed array, the left and right children would be located at `idx*2+1` and `idx*2+2` respectively, and the parent would be located at `Math.floor((idx-1)/2)`.

## Simple BinaryHeap Object

Let's implement a Binary Heap.

We'll let the user supply a comparator function that will provide a "score" for each element that will be used to compare two elements. The heap will be a array with index starting at 1 to simplify subscripts for parents and children. The zero index, null, will never be accessed.

```javascript
var BinaryHeap = function(comparator) {
  this.heap=[null];
  this.comparator=comparator;
}
```

If we're ordering numbers or other elements with lexicographic ordering, the scoring function can just be the identity function: `function(x) { return x; }`. The scoring function allows us to define either a minimum or a maximum heap (by changing the sign of the returned element).

To start, we'll define a few helper methods. `score` will just call the specified comparator function on a given value of an element at index `idx`.

```javascript
BinaryHeap.prototype.score = function(idx) {
  return this.comparator.call(this, this.heap[idx]);
}
```

`maxIdx` returns the current heap `length-1`, which represents the largest array index we can match without going out of bounds.

```javascript
BinaryHeap.prototype.maxIdx = function() {
  return this.heap.length-1;
}
```

`swap` swaps two values in the heap at indices a and b.

```javascript
BinaryHeap.prototype.swap = function(a, b) {
  var temp=this.heap[b];
  this.heap[b]=this.heap[a];
  this.heap[a]=temp;
}
```

### Insert / Push

```javascript
BinaryHeap.prototype.push = function(value) {
  this.heap.push(value);
  this.bubbleUp(this.heap.length-1);
}
```

To insert values, we simply push them onto the end of the array and then call `bubbleUp` with the index of the newly inserted element. What does bubbleUp do?

`bubbleUp` moves the newly inserted element up in the tree until its score is greater than or equal to that of its parent.

```javascript
BinaryHeap.prototype.bubbleUp = function(idx) {
  while (idx>1) {
    var eleScore = this.score(idx);
    var parentIdx = Math.floor(idx/2);
    var parentScore = this.score(parentIdx);
    if (eleScore < parentScore) {
      this.swap(parentIdx, idx);
      idx = Math.floor(idx/2);
    } else { break; }
  }
}
```

If the score is less than that of the parent, we want to swap out the current index with the index of the parent. The parent index is always given by the floor of the current index divided by two, so long as `idx>1`. Remember, we are never accessing idx=0, and when idx=1, Math.floor(idx/2) => Math.floor(1/2) => 0.

### Delete / Pop

We must account for the fact that children are not necessarily ordered when we delete or "pop" from the heap. The algorithm for deletion is:

1. Remove the element at index 1. This is the value that will be returned.
2. Pop the last value from the heap. This is the value we'll use to replace index 1.
3. Make sure the index 1 element !== the last value from the heap.
* If it does, we're removing the final element from the heap: simply return it.
4. If the index 1 element isn't the last value of the heap, put the last value at index 1.
5. Call `siftDown` on the element at index 1.

`siftDown` is a method that propagates a newly added element down the tree until it is in the proper place given the heap's ordering.

```javascript
BinaryHeap.prototype.siftDown = function(idx) {
  const max = this.maxIdx();
  while (idx<max) {
    var eleScore = this.score(idx);
    var leftChild = idx*2;
    var rightChild = leftChild+1;
    var swap=null;
    if (leftChild <= max) {
      var leftChildScore=this.score(leftChild);
      if (leftChildScore < eleScore) { swap = leftChild; }
    }
    if (rightChild <= max) {
      if (this.score(rightChild) < (swap ? leftChildScore : eleScore)) {
        swap = rightChild;
      }
    }
    if (!swap) { break; }
    else {
      this.swap(idx, swap);
      idx=swap;
    }
  }
}
```

What we're doing is this:

1. Calculating the index for leftChild and rightChild
2. Ensuring the index is in bounds
3. If the index is in bounds, comparing the element score with the given child's score.
4. If the element has a score greater than one of the children's scores, we need to swap these two elements.

For each iteration of the loop in `siftDown`, assuming that a left and right child exist, the inserted element could either be:

1. In the correct place (`break` the loop)
2. Greater than the left child but less than the right child (swap the left child and the element)
3. Greater than both the left and the right child (swap the right child and the element)

After performing the comparisons, we check to see if we are due to swap elements. If so, we perform the swap between the element and either the left or the right child.

The most confusing part is the ternary expression `if (this.score(rightChild) < (swap ? leftChildScore : eleScore))` in the `if (rightChild <= max)` block.

This is required because it could be that the value of the inserted element is greater than the `leftChild` value but less than the `rightChild` value. In this case, we would not want to swap the element with `rightChild`.

Consider this tree that illustrates the case:

```
   4 (A)
 /     \
3 (B)   7 (C)
```

If we did swap the element with the right child, we know it would create an invalid heap: for three numbers A, B, C, if A > B < C, it follows that A < C, and C cannot become A's parent. B can become the parent of A and C, however.

Without the ternary expression, the tree above would become this (if we just compared the inserted element's score with each child individually):

```
  7
 / \
3   4
```

For each parent (A) we ask: is A greater than B? If not, we're done. If so, then we're going to swap it *at least* with B, and potentially with C as well. Now is C less than B? If not, swap A and B. Otherwise, swap A and C. Now consider the new tree generated by the movement to the left (swapping A with B) or right (swapping A with C).

Repeat until either A is less than both its left (B) and right (C) child or until we reach the end of the tree.

Let's visualize how this algorithm works for the following min-heap:

```
       4
    /     \
   9       14
 /   \    /  \
14   12  30  10
```

Note again this is a valid min-heap but an invalid binary search tree.

We remove the element 4.

4 does not equal the last element in the tree (10), so we know the tree is not empty.

We swap 10 with 4's position. The tree now looks like this:

```
       10
    /     \
   9       14
 /   \    /  \
14   12  30   
```

We then call `siftDown` on 8. Let's trace it through the tree. leftChild is 10, and rightChild is 8.

9 is less than 10, so swap is equal to 10.
14 is not less than 10, so we swap out 9 and 10.

```
       9
    /     \
  10       14
 /   \    /  \
14   12  30   
```

Now we continue comparing, since we are not yet at `idx==max`.

leftChild is 14. 10 is not greater than 14, so we continue.
rightChild is 12. swap is null, so we compare the rightChild score with the element score.

The right child is not less than the element, so we're done.

Notice this is a complete binary tree (but not a binary *search* tree): all levels are filled to the maximum amount, and unfilled levels are populated from left-to-right.

Consider another shorter case where a deletion happens to place the element in the correct position from the start (best-case insertion):

```
      4
   /     \
  10      7
 /  \    /  \
14  12  9    8
```

We remove the element 4.

4 does not equal the last element in the tree (8), so we know the tree is not empty.

We swap 8 with 4's position. The tree now looks like this:

```
      8
   /     \
  10      7
 /  \    /  \
14  12  9    
```

We then call `siftDown` on 8. Let's trace it through the tree. leftChild is 10, and rightChild is 8.

10 is *not* less than 8, so swap is equal to 10.
8 is not less than 8, so we stop.

## HeapSort

Organizing elements in a binary heap gives them the convenient property of being able to be sorted very quickly. Notice that each time we perform a `pop` operation on a min-heap, we retrieve the next ascending sorted element of the heap's values; similarly, when we `pop` a max-heap, we retrieve the next descending sorted element.

This can be used to implement a priority queue by high or low priority.

We can also use this ordered property of heaps to implement [HeapSort](https://en.wikipedia.org/wiki/Heapsort).

```javascript
BinaryHeap.prototype.heapSort = function() {
  const copy = this.heap.slice(0);
  let res=[];
  const max = this.maxIdx();
  for (var i=0; i<max; i++) {
    res.push(this.pop());
  }
  this.heap=copy;
  return res;
}
```

We don't want to destroy the heap when we return its elements in a sorted manner, so we want to somehow copy and restore the heap after the sorting operation. There are a few ways that we could go wrong in implementing this.

1. We might think that we could just say: `for (var i=0; i<this.maxIdx(); i++)`.
* This is a bad idea: `this.maxIdx()` changes with each invocation of `this.pop()` since removing an element shortens an array by one each time. We'll only get halfway through the array and then stop.

2. We might think that we can copy the heap like this: `const copy = this.heap`.
* This would be incorrect: `copy` would hold a *reference* to `this.heap`, but not the *value* of `this.heap`. Using [`array.slice()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) solves this problem, but with a caveat! Note from the documentation that `Array.slice` will copy strings, numbers, and booleans, *but not objects*.

If our heap contains an array of objects that we order by a given key, say `priority`, heapSort will wipe out each of these elements, because `copy` will contain an array of *references*, not a deep copy of the objects contained.

To implement this copying in a robust manner, we should really define a custom copy method for our array, so that we could simply say: `copy = this.copyHeap()`.

If the objects in the heap are large enough, it would make far more sense to simply create a map between heap indices and the key upon which they are sorted, sort the map array, and return it. This array, when iterated through from start to finish, will be the order in which we access elements of the heap to read them in ascending or descending order.

```javascript
BinaryHeap.prototype.orderMap = function(dir=1) { //1 ascending, -1 descending
  let priorities=this.heap
    .map((e,idx)=>(idx!==0) ? {key: this.comparator(e), idx: idx} : null)
    .filter(e=>e);
  return priorities.sort((a,b)=>a.key >= dir*b.key).map(e=>e.idx);
}
```

Since the elements are stored in an array for which we can look up the value of any cell in O(1) time, sorting is actually just an abstract operation of accessing cells in sorted *order*. Whether or not the cells are sorted when accessed in ascending or descending sequence of its index is incidental, and just an often convenient convention.

Indeed, it would perhaps make more sense to store a reference to each priority object in an array that is bound to a key on which we sort by and instead insert these priority->reference mappings into a heap. Making these decisions is implementation-specific, but it's good to think about the different ways you could achieve the same ends.

It is of course trivial to sort a heap that already exists. If we had an unsorted array, say, [3, 4, 70, 13, 11, -4, 7], we could easily sort it using heapSort like so:

```javascript
function heapSortArray(arr, dir=1) {
  let sortedArray = new BinaryHeap((x)=>dir*x);
  for (var i=0;i<arr.length;i++) { sortedArray.push(arr[i]); }
  return sortedArray.heapSort();
}
```

### Converting a min heap into a max heap

It's an interesting tidbit that we can convert a min heap into a max heap (or vice versa) by reversing the comparator function's direction and then calling `siftDown` on indices `[Math.floor(heapLength/2), ..., 1]`.

```javascript
let T = new BinaryHeap(function(x) { return x; });
T.push(4);
T.push(3);
T.push(5);
T.push(7);
T.push(13);
T.push(22);

console.log(T.heapSort());
// [3, 4, 5, 7, 13, 22]

T.comparator=function(x) { return -x; };
let len=T.maxIdx();
for (var i=Math.floor(len/2);i>0;i--) {
  T.siftDown(i);
}
console.log(T.heapSort());
// [22, 13, 7, 5, 4, 3]
```

This is an application of [Floyd's method of heap construction](https://en.wikipedia.org/wiki/Binary_heap#Building_a_heap) applied in-place to an existing heap structure. Kinda cool!

## Demo on CodePen

Play around with the demo to get a feel for things.

<p data-height="279" data-theme-id="32039" data-slug-hash="PQKpOQ" data-default-tab="js" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Implementation of Binary Heaps and Heapsort" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/PQKpOQ/">Implementation of Binary Heaps and Heapsort</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## Wrapping Up

Congratulations! You've mastered yet another simple data structure. Try reviewing the structures you've already learned and, most important, *coding them from scratch*, perhaps even with a pen and paper. It's one thing to read code and conceptually understand. It's far different to wrestle with off-by-one errors and indexing foibles.

Practice hones intuition, and sufficient intuition is indiscernible from mastery.
