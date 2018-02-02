---
layout: post
title:  "Intro to Binary (Search) Trees"
date:   2018-2-1 06:56:04 -0500
description: An introduction to the binary tree data structure with code implementation of insertion, traversal, and search methods.
author: Thomas Danner
lang: en_US
categories: tutorials javascript fundamentals
tags: JS, stack, queue, big-O, complexity, dataStructures, binaryTree, treeTraversal
comments: true
---

## Binary Trees

A [binary tree](https://en.wikipedia.org/wiki/Binary_tree) is a tree data structure composed of nodes that contain two branches, a `left` and a `right`. The tree consists of a top-level or root node. Each branch from the root can hold nodes that themselves have arbitrarily many children.

By convention, the root of a binary tree is drawn at the top, with branches proceeding downward.

In a subset of the binary tree structure, the *Full binary tree*, every node has either 0 or 2 children. Nodes with zero children (where `left` and `right` are null) are called leaf nodes.

Binary trees in and of themselves are often not used in computer science, but the tree structure can be used to create a data structure called a *Binary Search Tree*, which is much more useful.

## Binary Search Trees

In a [binary search tree](https://en.wikipedia.org/wiki/Binary_search_tree), an "ordered" or "sorted" binary tree, the child nodes are kept in an order relative to their parents. We create an insert method to insert data in this ordered fashion.

For inserting item A in tree T, we proceed down the tree. For each node we encounter, we compare the value of A with the value of that node. If A is less than the node, we go left. If A is greater than the node, we go right.

We repeat this process until we reach leaf nodes (a node where one or both children are null). Then we compare the value of A to this final node. If A is less than the leaf node value, we set A as the new left child of this leaf node. If A is greater than the leaf node value, we set A as the new right child of this leaf node.

Let's look at the node structure and then consider a more concrete example for search and insertion.

### Nodes

Nodes contain four properties.

1. `data` - the data of interest
2. `left` - a pointer to the next left child node
3. `right` - a pointer to the next right child node
4. `iteration` - the number of times this node has been visited

```javascript
var node = function(data, left, right) {
  this.data=data || null;
  this.left=left || null;
  this.right=right || null;
  this.iteration=0;
}
```

Using this node object, we can create a simple tree with B as the root, A as the left child, and C as the right child like so:

`let tree = new node('B', new node('A'), new node('C'))`

gives us:

```
     B
   /   \
  A     C
/  \   / \
˚  ˚  ˚   ˚
```
##### tiny circles ˚ correspond to null

## Insertion

Let's start by considering the tree above and what the insertion behavior should look like with some examples. Then we'll write the code.

Pretend that we want to insert `node('D')` into the above tree. We start at the root. 'D'>'B', so we go to the right. `C.left` and `C.right` are both null, so we compare 'D' with 'C' to decide which side to insert it on. 'D'>'C', so we insert to the right. The tree looks like this after insertion:

```
    B
  /   \
 A     C
/ \   /  \
˚ ˚   ˚   D
```

Now let's insert `node('Apple')` into the tree (remember, Javascript uses lexicographical ordering, so it compares letter by letter.) 'Apple'<'B', so we go left. 'Apple'>'A', so we insert 'Apple' to the right. The tree then looks like this:


```
       B
   /       \
  A         C
 /  \      / \
˚ 'Apple' ˚   D
```

What happens if we have a duplicate value: say we insert `node('D')` into the above tree again? We want to add nodes as "leftmost" as is possible (while maintaining ordering). So we'll get down to node `D`, and then left and right are null. `D` is neither greater than nor less than `D`, but we have to insert the node somewhere. By convention, we insert it to the left. So calling `insert('D')` on this tree above would give us this result:

```
       B
   /       \
  A         C
 /  \      / \
˚ 'Apple' ˚   D
             /  \
            D    ˚
```

Let's code up this. We can code insert imperatively or using recursion.

### Recursive insert

Consider the base case (the node we reach is null and we want to insert the new value here). In this case, we return a new node with the value.

If we haven't reached the base case, we need to make our way down the tree until we reach a null node. We compare the `value` we wish to insert with current node's value `tree.data`. If it is less than or equal to the current node, we recurse on the left child. If its is greater, we recurse on the right child.

```javascript
function recursiveInsert(tree, value) {
  if (tree==null) { return new node(number); }
  if (value<=tree.data) { tree.left=insert(tree.left, value); }
  else { tree.right=insert(tree.right, value); }
  return tree;
}
```

### Imperative Insert

We set a pointer `navi` equal to tree. We navigate down using the pointer until we reach the point where `navi`'s children are null (so that `navi` is now a leaf node). When we reach the leaf node, we break out of the traversal loop and decide on which side to insert the data, create a new node with the value, and return the tree with the inserted item.

```javascript
function imperativeInsert(tree, number) {
  if (tree==null) { tree = new node(number); }
  let navi=tree;
  while (navi!==null) {
    if (number<=navi.data)
    {
      if (navi.left!==null) { navi=navi.left; }
      else { break; }
    }
    else {
      if (navi.right!==null) { navi=navi.right; }
      else { break; }
    }
  }
  if (number<=navi.data) { navi.left=new node(number); }
  else { navi.right=new node(number); }
  return tree;
}
```

## Search

You might wonder why bother with all of this complicated insertion. Well, the way that we've inserted the nodes means that we can search the tree incredibly quickly. The tree is already set up to perform a binary search: at each branch of the tree, we can eliminate half of the possible items to consider, since we know that all the children on one side will be greater than the value of the current node.

Consider this tree:

```
       5
    /     \
   2       7
  / \     / \
 1   4   6   9
```

We're going to search for the value 6. If we had this tree as an ordered array, [1, 2, 4, 5, 6, 7, 9], we'd have to iterate through the elements one by one until we found 6. We'd iterate through 1, 2, 4, and 5 before we found the answer. That's 5 operations.

In the binary tree, we perform only 3 operations. Is 6>5 ? Yes. Go right. Is 6>7 ? No. Go left. We're at 6!

If you consider the way that we inserted our data, it becomes clear how to implement a search. The only thing that's different is that we might find the node does not exist (the node that we have reached is null), in which case, we return false. Otherwise, we check each node to see if:

1. It's the value. If so, return it.
2. It's less than the value. If so, go right.
3. It's more than the value. If so, go left.

It looks like this:

```javascript
function binarySearch(tree, value) {
  if (!value) { return false; }
  let res=[];
  let stack=[];
  let node=tree.root;
  while (node!==null) {
    if (node.data==value) { return true; }
    if (value<node.data) { node=node.left; }
    else { node=node.right; }
  }
  return false;
}
```

Easy enough! NB: The search we are talking about here is [depth-first](https://en.wikipedia.org/wiki/Depth-first_search). There is another style of traversing, [breadth-first](https://en.wikipedia.org/wiki/Breadth-first_search), which we will discuss in a later tutorial.

## Traversal

What good is having a tree if you can't climb it?

There are three common types of traversal:

1. pre-order: the root is printed *before* the subtree values
2. in-order: the root is printed *in between* the left and right subtree values
3. post-order: the root is printed *after* the subtree values are traversed
4. (bonus): reverse in-order the right subtree values are printed *before* the root and then the left subtree values.

Let's try to perform what's called an in-order traversal of a binary search tree, which will print out all of the items of the tree in their order.

Can you think about how to do this?

```
       5
    /     \
   2       7
  / \     / \
 1   4   6   9
```

Given this tree, here's what our three traversal types will output.

1. pre-order: `[5, 2, 1, 4, 7, 6, 9]`
2. in-order: `[1, 2, 4, 5, 6, 7, 9]`
3. post-order: `[1, 4, 2, 6, 9, 7, 5]`
4. reverse in-order: `[9, 7, 6, 5, 4, 2, 1]`

This is a case that's handled quite nicely by recursion.

```javascript
function printNodesInOrder(tree) {
  let res=[];
  function traverse(tree) {
    if (tree==null) { return; }
    traverse(tree.left);
    res.push(tree.data);
    traverse(tree.right);
  }
  traverse(tree);
  return res;
}
```

Let's think about what the function is doing step by step. We exhaustively traverse the nodes of a tree until the point at which that node equals null. This means that we reached a leaf node's child, so we stop traversing. Otherwise, for each leaf node, we push the value of the left leaf, the value of the root, and the value of the right leaf.

We perform this operation on all of the nodes of the tree from left-to-right. Since values are inserted in order in a binary search tree and we perform the traversal in a left-to-right (or ascending) order, the numbers come out in order, as we'd expect.

As a neat side-note, we can turn the print node function into pre-order and post-order simply by changing the order of `res.push` in our recursive `traverse` call. Similarly, we can get a reverse in-order result by swapping the  `traverse(tree.right)` and `traverse(tree.left)` calls.

#### pre-order:

```javascript
res.push(tree.data);
traverse(tree.left);
traverse(tree.right);
```

#### post-order:

```javascript
traverse(tree.left);
traverse(tree.right);
res.push(tree.data);
```

#### reverse in-order:

```javascript
traverse(tree.right);
res.push(tree.data);
traverse(tree.left);
```

While the use cases of pre and post-order traversal on a binary search tree are limited, it's good to know about these traversal types, since they're frequently used in other tree traversal algorithms.

## Wrapping Up

Whew! Again, pat yourself on the back. This stuff isn't easy! In the next tutorial, we'll dig into deletion methods for binary search trees and explore a few more neat applications.
