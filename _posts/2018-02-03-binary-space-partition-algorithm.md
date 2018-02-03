---
layout: post
title:  "Binary Tree Application: Binary Space Partitioning"
date:   2018-2-3 06:56:04 -0500
description: A discussion of binary space partitioning, an application of binary trees to partition a space for procedurally generated levels or art generation.
author: Thomas Danner
lang: en_US
categories: tutorials compsci
tags: JS, stack, queue, big-O, complexity, dataStructures, binaryTree, binarySpacePartitioning, canvas, art, proceduralGeneration
comments: true
---

<p data-height="514" data-theme-id="32039" data-slug-hash="yveddj" data-default-tab="result" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Binary Space Partitioning" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/yveddj/">Binary Space Partitioning</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
##### Procedural container generation using Binary Space Partitioning

## Binary Space Partitioning

Let's take a step back from all of this theory and look at a cool application of a binary tree: binary space partitioning. I used binary space partitioning in my [Roguelike Dungeoncrawler Game](http://thmsdnnr.com/roguelike/) to create procedurally generated worlds. This means that each level (and indeed, each game) never has the same size or configuration of rooms.

In the demo above, each room configuration is generated with a single function call: there are no predetermined dimensions other than the container's width and height constraints and a minimum child dimension. Pretty cool, right?

[Binary space partitioning](https://en.wikipedia.org/wiki/Binary_space_partitioning) is a technique for recursively dividing a space subject to some constraints. In this case, the constraint is the width and height of the container, and the fact that we do not want containers to overlap (since they represent separate "rooms"). By creating Leaves which hold Containers inside of them, we are able to generate non-overlapping rooms with a specified amount of space between them (namely, the padding of the leaves that offsets the boundary of the containers).

## Let's make some leaves and containers.

All that the leaves and containers know about is an x and y position (of their upper-left corner) and a width and height. Leaves hold containers. Containers are the shapes that you see drawn on the screen.

Leaves have an x and y position and a width and a height. They also have a container (the structure within them that you see drawn), as well as a left and a right child. Thus, a leaf is an *extension of the concept of a node in a binary tree* with a few additional properties for this application.

We pad each container by a random amount, setting its width and height to between 70 and 90% of its enclosing leaf. Then, we set the x and y coordinates of the container within the leaf by assigning half the padding to the left & right coordinate. This has the effect of centering the container within each leaf.

We can draw a container just by defining a rectangle with its x and y coordinates and stroking it on an HTML5 canvas. This is a [simple introduction](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Drawing_shapes) to drawing shapes with canvas if you need a refresher.

```javascript
const randomIn = (min,max) => Math.floor(Math.random()*(max-min))+min;

let Container = function(x1, y1, x2, y2) {
  this.x1=x1;
  this.y1=y1;
  this.x2=x2;
  this.y2=y2;
}

let Leaf = function(x, y, width, height) {
  this.x=x;
  this.y=y;
  this.width=width;
  this.height=height;
  this.left=null;
  this.right=null;
  this.container=null;
}

Leaf.prototype.addContainer = function() {
  let width=Math.floor(randomIn(0.7, 0.9)*this.width);
  let height=Math.floor(randomIn(0.7, 0.9)*this.height);
  let xPos=Math.floor(this.x+(this.width-width)/2);
  let yPos=Math.floor(this.y+(this.height-height)/2);
  this.container=new Container(xPos, yPos, xPos+width, yPos+height);
}

Leaf.prototype.drawSelf = function(ctx) {
  let C=this.container;
  ctx.strokeStyle='black';
  ctx.strokeWidth='2';
  ctx.stroke();
  ctx.strokeRect(C.x1, C.y1, C.x2-C.x1, C.y2-C.y1);
}
```

Now all we have to do is partition the space into non-overlapping leaves at the desired level of depth. We'll do this in two steps. First, we have to define a `splitLeaf` function for the leaves.

We choose whether to split the leaf vertically or horizontally. We do this based on a set proportion. If the width/height ratio is greater than or equal to 1.5 (so it's a rectangle with, e.g., width>= 15 for height 10), we choose to split the container vertically. Otherwise, the container is either a square or taller than it is wide. In this case, we choose to split the container horizontally.

You can toggle the desired width/height ratio to get more vertical containers, more horizontal containers, or more square-like containers.

Once we choose the direction of split, we calculate the available space to partition in the chosen direction. If we are splitting horizontally (dividing with a horizontal line, somewhere in the y-coordinates of the container), we calculate the total height of the container. If we are splitting vertically, (dividing with a vertical line, somewhere in the x-coordinates of the container) we calculate the total width of the container.

If the total width or height available is less than the minimum leaf size*2, we return false (it's not possible to create the container). Otherwise, we pick a random split point that is between the minimum leaf size and the maximum available space minus the minimum leaf size. This means that we ensure that each container has at a minimum `minLeafSize`, and at a maximum `max-minLeafSize`.

Once we've determined that we have the horizontal or vertical space available to make the split, we set the leaf's left and right children to the two new leaves.

If we split vertically, both new leaves share the same height, and their width is partitioned into `[0, splitLoc]` and `[splitLoc, this.width-splitLoc]`.

If we split horizontally, both new leaves share the same width, and their heights are partitioned into `[0, splitLoc]` and `[splitLoc, this.height-splitLoc]`.

```javascript
Leaf.prototype.splitLeaf = function() {
  let splitVertical;
  if (this.width/this.height>=1.5) { splitVertical=true; }
  else { splitVertical=false; }

  let max = (splitVertical ? this.width : this.height)-minLeafSize;
  if (max <= 2*minLeafSize) { return false; }
  let splitLoc=randomIn(minLeafSize, max-minLeafSize);

  if (splitVertical) {  
    this.left=new Leaf(this.x, this.y, splitLoc, this.height);
    this.right=new Leaf(this.x+splitLoc, this.y, this.width-splitLoc, this.height);
  }
  else {
    this.left=new Leaf(this.x, this.y, this.width, splitLoc);
    this.right=new Leaf(this.x, this.y+splitLoc, this.width, this.height-splitLoc);
  }
  return true;
}
```

Finally, we define a function that will recursively traverse all the leaves of a given leaf and return an array of their children. We'll use this later on the root leaf of our container to gather up all the leaves so that we can draw them.

```javascript
Leaf.prototype.getLeafs = function() {
  return (!this.left&&!this.right) ? [this] : [].concat(this.left.getLeafs(), this.right.getLeafs());
}
```

### Using the code we wired up

Now that we have a container class and a leaf class with a leaf splitting function, all that we have to do is call the function. We'll wrap it in a `redrawCanvas` method so that we can call it repeatedly with `setInterval` to randomly generate a new board.

Generating a board from the classes we've defined is pretty simple.

First we define some constants: the max width and height, a minimum leaf size, and the desired number of leaves.

Then, we define a root leaf that contains all the space available for partitioning.

Finally, we call `splitLeaf()` on the left and right child of every element in our leaf tree that has not yet been split (for which both `leaf.left` and `leaf.right` is null).

When the while loop exits, `root` will contain the maximum number of possible leaves given our constraints.

If you were to log out the length of the leaves in each iteration, you'll notice that it's in the mid-to-high 500s. This is almost half of our desired number of leaves. The reason this happens is because there are conditions in our `splitLeaf` method that return false if the leaf is not able to be split. This makes sense too if you consider the bounds of the board (700x420) and the minimum leaf size. It's actually impossible to have 1000 non-overlapping leaves of side dimension 10 on a board that's 700x420 (you'd need minimally 1000x1000 available).

Once we have a `root` element with all our leaves in it, we call `getLeaves` to populate an array with all the leaves. For each leaf, we add a container (the inner parts you see drawn on the screen, padded out by our specified percentages). We then draw the container.



```javascript
const gameWidth=700;
const gameHeight=420;
const minLeafSize=10;
const desiredNumberOfLeaves=1000;

function redrawCanvas() {
  ctx.setTransform(1, 0, 0, 1, 0, 0);
  ctx.clearRect(0, 0, gameWidth, gameHeight);
  ctx.translate(0.5, 0.5);

  //create leaves with containers
  let root = new Leaf(0, 0, gameWidth, gameHeight);
  let leafLitter=[root];
  let didSplit=true;
  while (didSplit) {
    didSplit=false;
    leafLitter.forEach((leaf)=>{
      if (leaf.left===null&&leaf.right===null) {
        if (leaf.splitLeaf()&&leafLitter.length<desiredNumberOfLeaves) {
          leafLitter.push(leaf.left, leaf.right);
          didSplit=true;
        }
      }
    });
  }

  // draw containers inside leaves
  let leaves=root.getLeafs();
  leaves.forEach(function(leaf) {
    leaf.addContainer();
    leaf.drawSelf(ctx);
  });
}
```

And this code just sets up our canvas when the window loads and draws a new, procedurally generated canvas every second to the screen for your viewing pleasure.

```javascript
window.onload = function() {
  const cvs=document.getElementById('board');
  const ctx=cvs.getContext('2d');
  redrawCanvas(ctx);
  setInterval(function() {
    redrawCanvas(ctx);
  }, 1000);
}
```

### Next steps

If you wanted to use this code to generate a game, you'd still have to connect each space with hallways and make the coordinates navigable. However, with just a few lines of code, we've solved a pretty complex problem: subdividing a space into arbitrarily small, non-overlapping spaces.

This is an application where trees make a lot of sense to use: each child element is smaller and consists of some subdivision of its parent's space. Imagine how hard it would be to keep track of all these elements if we didn't have some sort of hierarchy to store them in. Who's parents of who? How do we know how much space to allocate for each child?

Leveraging a binary tree structure allows us to easily achieve this task. Note that no collision detection was required to make sure the spaces are non-overlapping: with the binary subdivision algorithm, we get non-overlapping spaces for free.

Binary Space Partitioning can get even [more complicated](https://pdfs.semanticscholar.org/c496/61c65c1780053dcc1ccd71abec5f244af2c9.pdf) as it can be used to subdivide space in 3D to help render polygons in 3D graphics engines. Read more about them if you're interested: they are pretty awesome tools that can serve a variety of purposes.

Why not fork the Codepen? Try planting your own tree and watch it grow!
