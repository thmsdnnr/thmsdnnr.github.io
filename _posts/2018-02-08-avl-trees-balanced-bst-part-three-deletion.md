---
layout: post
title:  "AVL Trees Part 3: Deletions"
date:   2018-2-8 06:56:04 -0500
description: A discussion of AVL trees, a balanced binary search tree with better performance. Specifically, insert and delete methods that use node rebalancing.
author: Thomas Danner
lang: en_US
categories: tutorials compsci
tags: JS, stack, queue, big-O, complexity, dataStructures, binaryTree, AVL, balanacedBinarySearch, insert, delete
comments: true
---

## Motivating Demo on CodePen

<p data-height="516" data-theme-id="32039" data-slug-hash="QQGMyN" data-default-tab="result" data-user="thmsdnnr" data-embed-version="2" data-pen-title="AVL Tree w/ Insert + Delete in D3" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/QQGMyN/">AVL Tree w/ Insert + Delete in D3</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
##### We're almost there! We're so close!

## Deletion From AVL Trees

To delete nodes from our tree, we perform the same operations as we did in [binary search tree deletion](http://thmsdnnr.com/tutorials/compsci/2018/02/02/binary-trees-part-two.html).

For a short revise, there are three cases in deletion:

1. The node is a leaf node. It has 0 children. Simply remove the node from the tree.
2. The node has one child. We splice it out, connecting its parent to its child.
3. The node has three children. We follow the following algorithm:
* Find the successor of the node.
* Remove the successor and splice the parent with its right child.
* Save the successor we spliced out and swap it with the node to be deleted.

After we finish these steps, we then perform any necessary rebalancing caused by the change in branch length due to node removal.

There's a good summary of which paths to rebalance, plus some helpful diagrams, [on page 4 of these lecture notes](https://courses.cs.washington.edu/courses/cse332/10sp/lectures/lecture8.pdf).

Case 1 & 2: Rebalance from deleted node to root.

Case 3: Rebalance from deleted successor node to root.

Here's what the code looks like:

```javascript
function deleteNode(tree, value) {
  let T=tree.root;
  let parentNode=null;
  while (T!==null) { // find the value in question
    if (T.data==value) { break; }
    if (value<T.data) { parentNode=T; T = T.left; }
    else if (value>T.data) { parentNode=T; T = T.right; }
  }
  if (!T) { return false; } // the value does not exist
  if (parentNode==null) { parentNode=tree.root; }
  let childCt=countChildren(T);
  if (childCt===0) {
    parentNode.left==T ? parentNode.left=null : parentNode.right=null;
    let pToV=pathToValue(tree, parentNode.data);
    return unwindAndBalance(pToV, tree);
  }
  else if (childCt===1) {
    let nodeToSplice = T.left ? T.left : T.right;
    parentNode.left===T ? parentNode.left=nodeToSplice : parentNode.right=nodeToSplice;
    let pToV=pathToValue(tree, parentNode.data);
    return unwindAndBalance(pToV, tree);
  } else {
    let succ=findSuccessor(T);
    const node = succ.node;
    const path = succ.pathTo;
    let parent=findParent(T, node);
    parent.left===node ? parent.left=node.right : parent.right=node.right;
    let leafPath = parent.left===node ? longestPathToLeaf(T.right) : longestPathToLeaf(T.left);
    T.swapValue(node);
    leafPath=leafPath.filter(e=>Math.abs(e.getBF())>0);
    unwindAndBalance(leafPath, tree);
  }
}
```

It looks gnarly, but quite honestly it's not. Take a look again at the [deletion diagrams](http://thmsdnnr.com/tutorials/compsci/2018/02/02/binary-trees-part-two.html) in part two of the deletion tutorial for binary trees. We're covering these three cases. The only thing we're doing differently is adding an operation at the end: calculate the path to the deleted node, pass it to our stack unwinder, and call balance on all the nodes from leaf to root that might have been imbalanced by our deletion.

## Next steps

In a production implementation, we would likely optimize the calculation of balance factors by storing them locally in individual nodes and modifying the nodes' heights with inserts and deletes. For simplicity, since we're just learning the concepts, I omitted these details. Also, there is a method of deletion known as "lazy" deletion: you leave the node in the tree, but simply set a property `deleted: true` on the node.

This has advantages and disadvantages.

Good
* you only have to find the node in ~`n * log n` time to delete and save rebalancing cost.
* you can easily support undo/redo by toggling deleted flag

Bad
* leaving the deleted node in the tree requires extra space
* leaving the deleted node in the tree makes balancing tricky: if you are to insert another node at the deleted node's spot, do you overwrite the previous? This makes the potential benefit of caching dubious, as it might be unreliably persisted.

You now understand the basics of AVL trees. There are certainly some optimizations that would take place to make this code production ready, but if you can follow this code, you understand the concepts. Nice work!
