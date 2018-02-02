---
layout: post
title:  "Intro to Linked Lists"
date:   2018-1-31 06:56:04 -0500
description: An introduction to the linked list data structure with code implementation, comparisons to arrays, and big-O analysis.
author: Thomas Danner
lang: en_US
categories: tutorials javascript fundamentals
tags: JS, stack, queue, big-O, complexity, dataStructures, linkedList, array
comments: true
---

## (Singly) Linked Lists

A [linked list](https://en.wikipedia.org/wiki/Linked_list) is a container that holds an ordered list of items called nodes. The nodes have the special characteristic that (unlike arrays) they do not have to be stored contiguously in memory.

### Nodes

Nodes minimally contain two properties.

1. `data` - the data of interest
2. `next` - a pointer to the next node in the list.

```javascript
var node = function(data, next) {
  this.data=data;
  this.next=next||null;
}
```

In order to compare nodes, we need to define a helper function that evaluates object equality, since in JavaScript `{data:'test', ct:1}=={data:'test', ct:1}` is false.

This is easy enough. Two nodes are equal if their data and next properties are equal, so we simply AND the results of the two comparisons and return the result:

```javascript
node.prototype.equals = function(node) {
  return this.data===node.data&&this.next===node.next;
}
```

### Linked List Container

A Linked List (in this case, a singly linked list) contains a reference to the beginning or head of the list. It also contains methods for operating on its nodes to perform tasks like insertion, deletion, and search.

```javascript
var LinkedList = function(head) {
  this.head=head||null;
  this.ct=head ? 1 : 0;
}
```

These operations include:

* getLength - return the number of elements in the array
* insertAfter - insert a given node after the target node in the array
* removeAfter - remove the node after the target node in the array
* insertBeginning - insert a node at the beginning of the list
* removeBeginning - remove and return the node at the beginning of the list
* findNode - find the index of a node in the given list (if the node exists)

We can also implement methods that allow us to use our LinkedList as a Stack or Queue.

### Stack fns
* push - insert an element at the end of the linked list
* pop - remove and return the element at the end of the linked list

### Queue fns
* shift - remove and return the first element of the linked list
* unshift - remove and return the first element of the linked list

Note that shift and unshift are equivalent to removeBeginning and insertBeginning.

## Let's implement the methods.

With all the methods that access the list, we'll want to do a sanity check and make sure the list has a valid length to perform the operation. We also want to validate the arguments when a node is passed in for insertion or removal. It's also crucial to accurately update the count of the number of items in the list in order to perform these bounds checks.

#### getLength()

```javascript
LinkedList.prototype.getLength = function() { return this.ct; }
```

Obtaining the length is simple enough: we just get the count of entries stored in the list. We must be sure to accurately increment and decrement this count in all our other list methods.

#### insertAfter()

```javascript
LinkedList.prototype.insertAfter = function(node, newNode) {
  if (!node&&newNode) { return false; }
  let currentNode=this.head;
  let found=false;
  while (currentNode!==null) {
    if (currentNode.equals(node)) { found=true; break; }
    currentNode=currentNode.next;
  }
  if (found) {
    let newNodeAfter=currentNode.next;
    currentNode.next=newNode;
    newNode.next=newNodeAfter;
    this.ct++;
    return true;
  }
  return false;
}
```

In order to insert a node we will perform a common operation: finding the node of interest. Since the nodes are not stored contiguously in memory, we cannot simply start at a memory address and increment by fixed intervals. Instead, we must **traverse** the list. This is the work of the `while` loop. We start at the head node and sequentially proceed through the list nodes by setting `currentNode=currentNode.next` (until `currentNode==null`).

We compare each element using our `.equals` method. There are two cases here.

1. The node we're looking for doesn't exist. The found flag will be false, and we return false.
2. The node we're looking for does exist. The found flag is true.

If we find the node to insert after, we:

1. Place the **current** next node in a temporary variable, `newNodeAfter`.
2. Set `currentNode.next` to the `newNode`. (Inserting the new node)
3. Set `newNode.next` to `newNodeAfter`. ("put back" the node we moved over for `newNode`)
4. Increase the list length by one.

#### removeAfter()

```javascript
LinkedList.prototype.removeAfter = function(node) {
  if (!node) { return false; }
  let found=false;
  let currentNode=this.head;
  while (currentNode!==null) {
    if (currentNode.equals(node)) { found=true; break; }
    currentNode=currentNode.next;
  }
  if (found) {
    if (!currentNode.next) { return false; }
    else if (currentNode.next.next) { currentNode.next=currentNode.next.next; }
    else { currentNode.next=null; }
    this.ct--;
    return true;
    }
  return false;
}
```

Removing a node is very similar to inserting a node. We perform the same operation to find the target node, returning false if it is not in the list.

All that differs is what we do when we reach the node. There are two things to consider when removing a node after a given node.

1. That node has no nodes after it (its `.next===null`)
2. That node has exactly one node after it (its `.next!==null` but its `.next.next===null`)
2. That node has two or more nodes following it (`.next.next!==null`)

In the first case, we return false. There are no nodes to remove.

In the second case, we simply set `currentNode.next=null`.

In the third case, we have to not only **remove** the next node, but also join up the current node with the rest of the list chain. We set the `currentNode.next` to `currentNode.next.next` (skipping over the element in between). We decrement the list length by one.

#### insertBeginning()

```javascript
LinkedList.prototype.insertBeginning = function(node) {
  if (!node) { return false; }
  this.ct++;
  if (!this.getLength()) { return this.head=node; }
  node.next=this.head;
  this.head=node;
}
```

To insert a node at the beginning of the list, there are two possible cases:

1. The list is empty. The node to insert becomes the head.
2. The list is not empty. The node to insert becomes the head, and we must attach the existing head to the new node's `.next`.

In either case, we increment the list length by one.

#### removeBeginning()

```javascript
LinkedList.prototype.removeBeginning = function() {
  if (!this.getLength()) { return false; }
  else {
    let removed=this.head;
    this.head=this.head.next; this.ct--;
    return removed;
  }
}
```

To remove a node at the beginning of the list, there are two possible cases:

1. The list is empty. We return false, as there's nothing to remove.
2. The list is not empty. We set the current list's head to the value of `head.next`, return the removed element, and decrement the list length by one.

#### findNode()

```javascript
LinkedList.prototype.findNode = function(searchNode) {
  if (!(this.getLength()&&searchNode)) { return false; }
  let currentNode = this.head;
  if (currentNode.equals(searchNode)) { return 0; }
  let idx=0;
  while (currentNode!==null) {
    if (currentNode.equals(searchNode)) { return idx; }
    currentNode=currentNode.next;
    idx++;
  }
  return false;
}
```

We've been finding nodes already in all the other methods: we just have been using them directly rather than tracking their position. The `findNode` method will simply track the index, by incrementing a counter by one each time we traverse to the next node. If the node we are searching for exists, we return its index.

If it does not exist, we return false.

### Two bonus methods

As mentioned, the Linked List can be used to implement a stack. For this, we need two functions: push and pop.

#### push()

```javascript
LinkedList.prototype.push = function(node) {
  if (!node) { return false; }
  this.ct++;
  if (!this.head) { return this.head=node; }
  let currentNode=this.head;
  while (currentNode.next!==null) {
    currentNode=currentNode.next;
  }
  currentNode.next=node;
}
```

This code is pretty simple. We find the last node in the list, and we set its `next` to the given node. We increment the list length by one.

#### pop()

```javascript
LinkedList.prototype.pop = function(node) {
  let currentNode=this.head;
  let length=this.getLength();
  let popped=null;
  if (!length) { return false; }
  if (length===1) {
    popped=this.head;
    this.head=null;
    this.ct--;
    return popped;
  } else {
    while (currentNode.next.next!==null) {
      currentNode=currentNode.next;
    }
    popped=currentNode.next;
    currentNode.next=null;
    this.ct--;
    return popped;
  }
}
```

Popping a value off the list is a little trickier. Once again, let's consider the cases.

1. The list is empty. We cannot pop the item off an empty list. We return false.
2. The list has one element. We most pop the head off the list and set the list's head to null, decrementing the length by one and returning the head.
3. The list has more than one element. We must remove the element and decrement the list length by one.

Note the termination condition here in the while loop: we are looking for `currentNode.next.next` equaling null. Why? Well, we want the second-to-last node to operate on, since we are removing the very last node from the list (that's what gets popped), and in order to do this, we set the second-to-last node's `next` to null.

## Big-O Complexity & Comparison with Arrays

First, a quick word about Big-O. [Big-O notation](https://en.wikipedia.org/wiki/Big_O_notation) is not a scary monster. In short, it's simply a tool for us to [asymptotically compare](https://en.wikipedia.org/wiki/Asymptotic_analysis) the performance of operations on sets of data. When you see `log` in Big-O, it's log base 2, written abbreviated as log.

Big-O gives us general comparisons, not strict results. We can determine whether an operation takes place in constant O(1) or O(log n), linear O(n), logarithmic O(n*log n), quadratic O(n^2), exponential O(2^n), or factorial O(n!) time.

(There's a Big-O equivalent for space complexity: it's just another dimension of comparison.)

What types of operations? When it comes to data structures it's things like insertion, deletion, access, or search. We compare in three dimensions: best case scenario, worst case scenario, and average case scenario. Practically though, we are most concerned about the worst-case scenario. The average case is hard to quantify. The best case scenario is rare. And if we're okay with the worst-case scenario, we'll always be pleasantly surprised (pessimistic, but true!).

In short: if your algorithm runs in quadratic or slower time, there's probably a better way to do it. The most efficient sorting algorithms (quicksort and mergesort) work in logarithmic time. There are ways to make searching even faster than `n*log n`, but the algorithms are not generalizable since they leverage fixed characteristics of the data to optimize further.

### Comparing Big-O of Linked Lists and Arrays

Let's take a look at the [Big-O Cheat Sheet](http://bigocheatsheet.com/) and do a quick comparison of Singly Linked Lists with Arrays. Linked lists are similar to arrays, but use case dictates which is most performant.

Looking at the worst case scenarios, Linked Lists give us constant time insertion and deletion, and linear time access and search.
Arrays give us constant time access, and linear time search, insertion, and deletion.

Let's think about why this is the case.

#### Search is the same speed

To search in our linked list or array, we start at the beginning of the array and go to the end. In the worst case scenario, the thing we're searching for is farthest away from our starting point (at the `n-1`th index). Thus, it'll take O(n) operations for us to pass through the data structure to find it.

#### Access: Arrays O(1), Linked Lists O(n)

A linked list takes O(n) in the worst case to access a given element, whereas arrays can access elements in constant time. This is because arrays are *indexed*. This means that given the order in the list, I can go directly to that list item. How does this work? Well, an array is ordered contiguously in memory. That means that I can skip to any item in the array immediately simply by jumping to this memory address:

`memory address of head` + `array element size` * `index offset`

With a linked list, we might know how many elements are in memory, but since the elements do not have to be stored contiguously, we have to traverse every node in the list until we find an item. This is the same performance as doing a search.

#### Insertion/Deletion: Arrays O(n), Linked Lists O(1)

Linked lists are much faster at insertion and deletion. Since arrays are indexed, inserting or deleting an element requires us to change, in the worst case, all the other elements of the array. Imagine we insert at the beginning of the array. We have to go through and increment each element by one. Deletion at the beginning of the array means decrementing all the offsets by one.

Insertion and deletion at the end of the array is O(1).

Insertion and deletion at the beginning of a linked list is O(1). In a linked list, we know the head, and to insert and element, we simply set the new element's `next` to the previous head and then update the linked list's head to point to the new element. For removal, we just replace the list's head with the current node at `head.next`. These operations are constant time.

Insertion and deletion at the **end** of a linked list is still O(1), but we have to first traverse the whole linked list to find the end. Then we insert or delete. That search process in the worst case is O(n).

I know what you're thinking: this is super confusing.

The key takeaways are that arrays shine when you need quick, random access to any element in the array due to indexing. Lookup of any element is O(1). They are also good when you only need to add or remove elements from the end (like a stack, hint hint).

Linked lists on the other hand shine when you need insert or remove a lot of elements, particularly toward the middle or beginning of the array since they do not require reindexing. They are less performant when you require random access to individual elements.

## Wrapping Up

This is a ton of material. Take a moment to pat yourself on the back. While there is certainly a bit more nuance to Big-O notation and the many ways in which we can implement a Linked List, this covers the basic groundwork. If you got this far, you understand the key differentiators between arrays and linked lists and have a a good grasp of which one will be most performant for your application.

Here's the full code on CodePen to play around with:

<p data-height="247" data-theme-id="32039" data-slug-hash="zRGmVJ" data-default-tab="js" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Singly-Linked List Implementation" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/zRGmVJ/">Singly-Linked List Implementation</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
