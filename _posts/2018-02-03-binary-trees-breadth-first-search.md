---
layout: post
title:  "Binary Trees: Breadth First Search"
date:   2018-2-4 06:56:04 -0500
description: Examining the breadth-first traversal of binary trees.
author: Thomas Danner
lang: en_US
categories: tutorials compsci
tags: JS, stack, queue, big-O, complexity, dataStructures, binaryTree, treeTraversal, breadthFirst, d3, dataViz
comments: true
---

## Breadth First Search

[Breadth First Search](https://en.wikipedia.org/wiki/Breadth-first_search) is a method for traversing the nodes of a binary tree that visits all neighbor nodes in order before moving to the next level down of neighbors.

Unlike depth-first search, which imperatively uses a stack or uses recursion (which leverages the call stack), breath-first search uses a queue, which is First In First Out. This meshes well with the end goal, of displaying same-level neighbors before recursing down childrens' branches.

Note how we implement our queue with an array, but we define the end as the right side (where we `.push()` on new items) and the beginning as the left side, where we `.shift()` off existing items.

In order to determine the level at which we are in the tree, we use a technique where we push a null into the queue after we add the tree. Every time we encounter a null, we know that we've reached another level, because it means we've processed the current level of the queue that has been enqueued. When we reach another level, we push null into the queue again and check to see if there is a double null (null was shifted off the queue and then there is another null at the front). This means that the traversal has terminated and it's time to return the result.

With each iteration of the while loop, we push the current node into our result array.

```javascript
function breadthFirst(tree) {
  if (tree===null) { return; }
  let queue=[];
  let res=[];
  let level=0;
  queue.push(tree);
  queue.push(null);
  while (queue.length) {
    tree=queue.shift();
    if (tree===null) {
      level++;
      queue.push(null);
      if (queue[0]===null) { break; }
      else { continue; }
    }
    if (tree.left) { queue.push(tree.left); }
    if (tree.right) { queue.push(tree.right); }
    res.push(tree.data);
  }
  let levels={};
  res.forEach(obj=>{
   levels[obj.level] ? levels[obj.level].push(obj.data) : levels[obj.level]=[obj.data];
  });
  return res;
}
```

Compare the above to the depth-first approach below, where we push children deeper down onto the stack and process them one at a time as we pop them from the top.

```javascript
function depthFirst(tree) {
  let stack=[];
  let res=[];
  stack.push(tree);
  while (stack.length) {
    tree=stack.pop();
    res.push(tree);
    if (tree.left) {
      stack.push(tree.left);
    }
    if (tree.right) {
      stack.push(tree.right);
      tree=tree.left;
    }
  }
  return res;
}
```

### Which to use?

Breadth-first search is a useful technique for visualizing trees, whereas 
