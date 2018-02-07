---
layout: post
title:  "AVL Trees: Insertions and Deletions"
date:   2018-2-5 06:56:04 -0500
description: A discussion of AVL trees, a balanced binary search tree with better performance. Specifically, insert and delete methods that use node rebalancing.
author: Thomas Danner
lang: en_US
categories: tutorials compsci
tags: JS, stack, queue, big-O, complexity, dataStructures, binaryTree, binarySpacePartitioning, AVL, balanacedBinarySearch, insert, depete
comments: true
---

It's key that you read the [previous tutorial on AVL trees](http://thmsdnnr.com/tutorials/compsci/2018/02/05/avl-trees-balanced-bst.html) for this tutorial to make sense. We've covered the concepts of node balance factors, keeping node height differential to within `[-1, 1]`, and the use of rotations to correct unbalanced nodes.

Today we're going to incorporate this balancing behavior into our tree structure so that the tree will automatically rebalance itself and update node height upon insertion and deletion.

## Generating Unbalance

Before we code an insertion and deletion routine, let's first consider how trees can go out of balance. We will correct any imbalance when it is -2 or 2, so the tree will never get farther out of balance than this at a node.

There are four cases that can cause unbalance. In each case, we are starting at the grandparent node (the node that is imbalanced) and considering the configuration of its two children (child and grandchild).

### Single rotations required

[1] Right side, right child. Solution: rotate left.

<img src="/assets/pics/btree/r_1_before.png" height="200px" alt="Unbalanced AVL before left rotation.">

We can add new child on the right that causes the root to become right-heavy like this:

We can fix this by rotating the tree left about 1.

<img src="/assets/pics/btree/r_1_after.png" width="300px" alt="AVL balanced post left rotation.">

Now the tree is balanced.

[2] Left side, left child.

<img src="/assets/pics/btree/r_2_before.png" height="200px" alt="Long chain of nodes in binary search tree, resembling a linked list.">

We can fix this by rotating the tree right about 3.

<img src="/assets/pics/btree/r_1_after.png" width="300px" alt="AVL balanced post left rotation.">

Note that when the node that caused the unbalance is on the same side of its parent as *its* parent is of its parent, we perform a rotation that is *opposite* the direction of this side about the grandparent to correct the behavior.

With numbers for the two cases:

* [1] 2 is on the right of its parent 1. 3 is also on the right of its parent 2. We fix the imbalance by rotating left (opposite of right) about the grandparent, 1.
* [2] is on the left side of its parent 3. 1 is also on the left side of its parent 2. We fix the imbalance by rotating right (opposite of left) about the grandparent, 3.

### Double Rotations Required

The other two cases require us to rotate twice. The first rotation reduces the problem to one of the two problems above.

Notably, the first in a double rotation is about the *first child* of the unbalanced node, not the second.

[3] Right side, left child.

<img src="/assets/pics/btree/r_3_before.png" height="200px" alt="Long chain of nodes in binary search tree, resembling a linked list.">

First we have to rotate right about three, and then left about one.

Rotating right about three gives us:

```
1
 \
  2
   \
    3
```

after which it is easy to create the balanced tree, applying rotation [1] from above.

[4] Left side, right child.

<img src="/assets/pics/btree/r_4_before.png" height="200px" alt="Long chain of nodes in binary search tree, resembling a linked list.">

First we have to rotate left about one, and then right about three.

Rotating left about one gives us:

```
    3
   /
  2
 /
1
```

after which it is easy to create the balanced tree by applying rotation [2] from above.

## Coding Strategy

When we insert an element, we'll record the nodes that we traversed to get to the insertion spot in a stack. Then we'll pop items off the stack one by one, retracing our steps to the tree root. At each node, we'll calculate its recalculate its balance. If the node is unbalanced, we'll apply the appropriate balancing rotations.

How can we identify which rotations to apply in this process when we encounter an unbalanced node?

We can use the sign of the balance to determine this.

Remember that Balance will be Left Branch Length - Right Branch Length. So, if balance is -2, we know the node is right-heavy. If the balance is +2, we know the node is left-heavy.

If we encounter balance -2, we are in either a `[1] Right side, right child` or a `[3] Right side, left child`. We can determine which by looking at the balance of the right child (we know right, since the parent is right-heavy) of this unbalanced parent.

The child will have balance of either 1 or -1.

If the balance is 1, we know that we are in case `[3] Right side, left child` and require a double rotation (right-left).

If the balance is -1, we are in case `[1] Right side, right child` case and only require a single rotation.

We can create this table so that we know exactly what to do when we encounter a balance factor of 2.

| BF Parent  |   | BF Child       | | Rotations to Perform  |
|--:|--:|--:|-----:|
| 2 | | -1 | |Child left, Parent right |
| -2 | | 1 | |Child right, Parent left|
| 2 | | 1 | |Parent right |
| -2 | | -1 | |Parent left |

## Code Implementation

We'll perform a depth-first search to find the spot at which to insert or delete the node.

For inserts, we can observe that since we recurse down the tree until we reach a null (a leaf node), we don't have to worry about the inserted node itself throwing the tree out of balance. We only have to check its parent node and all other nodes on that path to the top of the tree.

To facilitate checking, we'll store the nodes we traverse in a stack, since we are going from root to child to find the node, and we want to trace back from child to root to perform balancing. As we push items onto the stack, we'll descend, and as we pop items from the top of the stack, we'll ascend.

When we pop nodes off the stack, we check each one to see if it is balanced. We'll write a simple method for the node, `getBF` that fetches the balance factor (the difference in the max height of the left subtree and the max height of the right subtree). If we encounter nodes whose difference is -2 or 2, we perform the appropriate rebalancing procedure.

```javascript
node.prototype.getBF = function() {
  return maxHeight(this.left)-maxHeight(this.right);
}
function maxHeight(tree) {
  if (tree==null) { return 0; }
  else {
    let leftD=maxDepth(tree.left);
    let rightD=maxDepth(tree.right);
    return leftD > rightD ? leftD+1 : rightD+1;
  }
}
```

We'll use our previous node search method, adding a stack so that if we find the insert point, we can retrace our path up the tree and correct any imbalances.

We don't need to

## Next steps
