---
layout: post
title:  "Pixl: A Pixel Art Editor w/Canvas (1 of 3)"
date:   2018-1-7 14:30:04 -0500
description: HTML5 Canvas, Pixel Art
author: Thomas Danner
lang: en_US
categories: tutorials javascript projects
tags: JS, events, canvas, pixelArt, editor
---

## Let's code a Pixel Editor!

<img src="/assets/pics/pixl.png" width="50%">

Requirements: It should be like a photo booth for pixel art. It should support multiple sizes of art with both transparent and solid backgrounds. The user should be able to save art created with the editor to resume work later on, as well as to copy or "fork" other users' work and make their own derivations.

## But first, we need a grid.

We're going to draw a 16x16 square grid on an HTML canvas. In order to make it easier to see, we'll scale each pixel by a factor of 30 (so each square is 30px). This makes the grid 480px by 480px (with 1px added so the canvas border is visible).

We'll store the data for the canvas in an array with length 256. The data structure is simply: index of square = index of array, and the array holds the hex string, e.g., `#FFCCEE` corresponding to that square's color.

When a user clicks a square, we look up which square was clicked based on the mouse coordinates and update the color of the square. If the color of the square is the *same* as the current color, we reset the square to the default background. This gives an "eraser" functionality without much overhead.

## Let's get started!

We'll assign a few constants and initialize our data array.

```javascript
let ctx=null;
let currentColor='#AAEEBB';
const DEFAULT_COLOR = '#EEEEED';
let canvasData = new Array(256).fill(DEFAULT_COLOR);
const WIDTH = 480; //px
const HEIGHT = 480; //px
const SQUARE_SIDE = 30;
```

We will be doing a lot of conversion from an index to a row + column, so we'll create a function that converts (X,Y) into an index `coordsToIdx` and a function that gets an index, given `coords`:

```javascript
function coordsToIdx(X, Y) {
  let col = Math.floor(X/30);
  let row = Math.floor(Y/30);
  return {row, col};
}

function boxIdxFromCoords(coords) {
  return coords.col+coords.row*16;
}
```

Now we need to create a function to set the data array when a box is clicked. We'll check to see if the current color matches the color we are placing in the square. If so, we reset to default.

If not, we set the color. For now, it's `currentColor`, which we hard-coded in. Later, we'll add color picker functionality that can modify this `currentColor` variable based on the user's selection.

```javascript
function setData(coords, color) {
  let idx=boxIdxFromCoords(coords);
  let currentColor=canvasData[idx];
  if (color!==currentColor) {
      canvasData[idx]=color;
  } else {
      canvasData[idx]=DEFAULT_COLOR;
  }
}
```

Here's how we draw the grid:

```javascript
function drawGrid() {
  ctx.lineWidth = 0.5;
  ctx.setTransform(1, 0, 0, 1, 0, 0); //reset transform
  ctx.translate(0.5, 0.5); //make lines sharp
  for (var i=0;i<=WIDTH;i+=SQUARE_SIDE) {
  //draw vertical line HEIGHT length, x=i
    ctx.beginPath();
    ctx.moveTo(i, 0);
    ctx.lineTo(i, HEIGHT);
    ctx.stroke();
    ctx.closePath();
  //draw horizontal line WIDTH length, y=i
    ctx.beginPath();
    ctx.moveTo(0, i)
    ctx.lineTo(WIDTH, i)
    ctx.stroke();
    ctx.closePath();
  }
  ctx.lineWidth = 0;
}
```

Since `drawGrid` will be called multiple times, we have to reset the drawing context's transformation before each drawing. Canvas transformations are cumulative. We translate the canvas by 0.5px in the x and y directions in order to make the lines appear sharper: so that the lines appear *at* a pixel rather than between two pixels.

Then we set up a loop and draw a horizontal and vertical line at each point for our rows and columns. We set `lineWidth` back to 0 when we are done, since don't want our arts' squares to have borders.

```javascript
function colorBox(box, color) {
  const { row, col } = box;
  ctx.fillStyle=color || currentColor;
  ctx.clearRect(col*30, row*30, 29.5, 29.5);
  ctx.beginPath();
  ctx.fillRect(col*30, row*30, 29.5, 29.5);
  ctx.closePath();
}

function drawData() {
  ctx.clearRect(0, 0, 500, 500);
  for (var i=0;i<canvasData.length;i++) {
    let row=Math.floor(i/16);
    let col=i%16;
    let color=canvasData[i];
    colorBox({row, col}, color);
  }
  drawGrid();
}
```

We define a click handler. If the click is outside the bounds of the canvas in the X or Y direction, we reject it. Otherwise, we call `setData` with the coordinates.

```javascript
function handleClick(e) {
  if ((e.offsetX>WIDTH||e.offsetX<0)||(e.offsetY>HEIGHT||e.offsetY<0)) { return false; }
  setData(coordsToIdx(e.offsetX, e.offsetY), currentColor);
  drawData();
}
```

### We're Off!

We set an event listener on our click event, adjust the canvas width and height, and draw the data!

```javascript
window.onload=function() {
  canvas = document.getElementById('editor');
  ctx = canvas.getContext('2d');
  ctx.imageSmoothingEnabled=false;
  canvas.width=16*30+1; //+1 to display border
  canvas.height=16*30+1;
  drawData();
  canvas.addEventListener('click', handleClick);
}
```

Stay tuned for Part Two.

### Codepen Demo for Part One:

<p data-height="550" data-theme-id="32039" data-slug-hash="WddbeW" data-default-tab="result" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Pixl: Part One" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/WddbeW/">Pixl: Part One</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
