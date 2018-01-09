---
layout: post
title:  "Pixl: A Pixel Art Editor w/Canvas [part 1]"
date:   2018-1-7 14:30:04 -0500
description: Walkthrough/tutorial on creating a pixel art editor using JS and HTML5 Canvas.
author: Thomas Danner
lang: en_US
categories: tutorials javascript projects
tags: JS, events, canvas, pixelArt, editor
---

## Let's code a Pixel Editor!

<img src="/assets/pics/pixl.png" width="271" height="400" alt="Pixel editor finished product screenshot">

Requirements: It should be like a photo booth for pixel art. It should support multiple sizes of art with both transparent and solid backgrounds. The user should be able to save art created with the editor to resume work later on, as well as to copy or "fork" other users' work and make their own derivations.

### But first, we need a grid.

We're going to draw a square grid of squares on an HTML canvas. Each square will represent a pixel. We'll scale each pixel up by a factor of 30 (so each square has side length=30px). This makes the grid 480px by 480px (with 1px added so the canvas border is visible).

Let's set up some dimensions and then adjust the canvas size to fit.

```javascript
//Global reference to canvas and context
let ctx;
let canvas;
//-> WIDTH and HEIGHT have to be even multiple of NUM_ROWS & NUM_COLS
const WIDTH = 480; //px
const HEIGHT = 480; //px
// -> NUM_ROWS & NUM_COLS have to be even multiple of 16
const NUM_ROWS = 16;
const NUM_COLS = 16;
const BOX_SIDE_LENGTH = WIDTH/NUM_ROWS; //px

function getCanvasAndContext() {
  canvas = document.getElementById('editor');
  ctx = canvas.getContext('2d');
  ctx.imageSmoothingEnabled=false;
  canvas.width=NUM_COLS*BOX_SIDE_LENGTH+1; //+1 to display border
  canvas.height=NUM_ROWS*BOX_SIDE_LENGTH+1;
}
```

### A Place to Put our Data

We'll store the data for the canvas in an array with length `NUM_ROWS*NUM_COLS`.

```javascript
//Default color and canvas data array
let currentColor='#AAEEBB';
let DEFAULT_COLOR='#FFFFFF';
let canvasData = new Array(NUM_ROWS*NUM_COLS).fill(DEFAULT_COLOR);
```

Each array element holds a hex value (e.g., `#FFCCEE`) corresponding to its square's color.

### Let's Draw that Grid!

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

Since `drawGrid` will be called multiple times, we have to reset the drawing context's transformation on each function call. Canvas transformations are cumulative. We translate the canvas by 0.5px in the x and y directions in order to make the lines appear sharper: so that the lines appear *at* a pixel rather than between two pixels.

This is what it'd look like if you didn't reset the transform:

<img src="/assets/pics/cumulativeTransform.gif" alt="Canvas transformed into black point-five pixels at a time">
##### Granted, it's kind of neat —- but not what we're going for here.

Then we set up a loop and draw a horizontal and vertical line at each point for our rows and columns, depending on the dimensions that we defined.

### Gathering User Input

We have a grid, but there's no way to gather user input yet.

We need to add a click [Event Listener](https://developer.mozilla.org/en-US/docs/Web/API/EventListener) to our canvas and then write a function to handle click events.

```javascript
function handleClick(e) { /*do something with our MouseEvent */ }
canvas.addEventListener('click', handleClick);
```

Our `MouseEvent` contains many different coordinates. There's `clientX/Y`, `pageX/Y`, `screenX/Y`, and `offsetX/Y`.

Without going into too much detail:

* [pageX/Y](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/pageX) gives coordinates of the mouse *relative to the entire page, including scrolling*. This means that if your page is 2000px long and you are scrolled to the very bottom, clicking at the very top of your browser window will return a y coordinate of `2000px` - `browserWindowHeight`
* [clientX/Y](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/clientX): gives coordinates of the mouse relative to the browser window area. Clicking on the upper-left corner of your browser window gives (0, 0).
* [screenX/Y](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/screenX): gives coordinates of the mouse relative to the screen. Clicking on the upper-left corner of your browser window gives you the X and Y coordinates of the upper-left point of your browser window on your screen.
* [offsetX/Y](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/offsetX): gives coordinates of the mouse *relative to the offset of the target node*.

We're using `offsetX` and `offsetY` because we put our event listener on the canvas element itself. This means that when we click the upper-left corner of the canvas, we'll get (0,0), and when we click on the bottom right, we'll get (CANVAS_WIDTH, CANVAS_HEIGHT). We'll get (0,0) when we click here regardless of browser window size, scroll status, or browser window position.

This is what we need in order to translate click coordinates into box indices.

We will be doing a lot of conversion from an index to a row + column, so we'll create two functions: one that converts an index into a row and column number (`idxToRowCol`), and one that converts mouse coordinates into an index (`coordsToIdx`).

```javascript
function idxToRowCol(idx) {
  let row=Math.floor(idx/NUM_ROWS);
  let col=idx%NUM_COLS;
  return {row, col};
}

function coordsToIdx(X, Y) {
  let col = Math.floor(X/BOX_SIDE_LENGTH);
  let row = Math.floor(Y/BOX_SIDE_LENGTH);
  return row*NUM_ROWS+col;
}
```

When the user clicks a square, we look up which square was clicked and set the data array. We bounds check the coordinates in our `handleClick` to make sure the click is in bounds and is not right on the margin. If we didn't do this, users could click the rightmost or leftmost boundary and have the row calculation "bump over" into the next/previous row at the opposite column. Consider a click at `480px`. 480/30 is 16, not 15 (rows are indexed 0-15).

```javascript
function handleClick(e) {
  let X=e.offsetX;
  let Y=e.offsetY;
  if ((X>=WIDTH||X<=0)||(Y>=HEIGHT||Y<=0)) { return false; }
  setData(coordsToIdx(X, Y), currentColor);
}
```

### Now we just need to write setData...

Once we find out *which* square was clicked, we decide *how* to fill the square.

We want to allow the user to erase colors that have been painted. We'll do this by checking to see if `currentColor` is the same as the color of the square being clicked. If so, we turn the square back to the default color. If not, we set the square equal to `currentColor`. This gives an "eraser" functionality without much overhead.

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

### Displaying the Contents of our Data Array

We'll write a function `colorBox` that colors in `box` with `color`, along with two helper functions to make sure that the row and column are in bounds.

```javascript
const isValidCol = (col) => col>=0&&col<=NUM_COLS-1;
const isValidRow = (row) => row>=0&&row<=NUM_ROWS-1;

function colorBox(box, color) {
  const { row, col } = box;
  if (!isValidCol(col)||!isValidRow(row)) { return false; }
  ctx.fillStyle=color || currentColor;
  ctx.clearRect(col*BOX_SIDE_LENGTH, row*BOX_SIDE_LENGTH, BOX_SIDE_LENGTH, BOX_SIDE_LENGTH);
  ctx.beginPath();
  ctx.fillRect(col*BOX_SIDE_LENGTH, row*BOX_SIDE_LENGTH, BOX_SIDE_LENGTH, BOX_SIDE_LENGTH);
  ctx.closePath();
}
```

We have the data in an array, and we have a way to color in a box. Now all we have to do is write a for loop and color away at each box, right?

```javascript
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

## NO NO NO NO NO.

Although this method of drawing data *will work*, it is not a fantastic idea. For several reasons.

First, we are redrawing *every single box*, regardless of whether that box was changed. There's no need to redraw boxes that haven't changed.

Secondly, we want to put our `drawData` in some sort of loop.

Here's the thing: **the entire program is a loop.** Event listeners for user input. Change squares' values. Redraw changed squares. User selects another color: update UI, update current color. Update Canvas.

We want to write all of our rendering functions in an efficient way so that we can call them over and over, and they will *only act on changed elements*.

We'll refactor our `setData` function slightly. When we set a value of the data array, we will push the index into an array called `dirtyIndices`.

Each time we call `drawData`, we will iterate through `dirtyIndices` instead of `canvasData`. We'll draw the `dirtyIndices`, and then we'll reset the `dirtyIndices` list to zero.

In `setData`, we'll also refuse to change our data array for an index that is dirty. This means that the data cannot be modified until the user sees the result of the last modification on the canvas.

This is what it's going to look like:

```javascript
let dirtyIndices=[];

function setData(idx, color) {
  if (dirtyIndices.includes(idx)) { return false; }
  let currentColor=canvasData[idx];
  if (color!==currentColor) {
    canvasData[idx]=color;
  } else {
    canvasData[idx]=DEFAULT_COLOR;
  }
  dirtyIndices.push(idx);
}

function drawData() {
  for (var i=0;i<dirtyIndices.length;i++) {
    let color=canvasData[dirtyIndices[i]];
    colorBox(idxToRowCol(dirtyIndices[i]), color);
  }
  dirtyIndices=[];
  drawGrid();
  requestAnimationFrame(drawData);
}
```

### requestAnimationFrame?

[requestAnimationFrame](http://creativejs.com/resources/requestanimationframe/index.html) is much kinder to the browser than `setInterval` when you're calling animation code. The timing of `setInterval` is not guaranteed, and it is not sync'd up with the browser's native refresh rate (around 60 Hz). When you put your animation code in `setInerval`, you're often asking the browser to redraw when it's not ready.

This hogs CPU and GPU, not to mention making things look strange.

`requestAnimationFrame` tells the browser "hey, when you're ready to update next, call this function to get the next frame of my animation." Less CPU usage. Things appear smoother. This is going to be especially important in Part Two of the tutorial when we have more things going on in each frame: mouse drag events, changing UI, etc.

### Further Refactoring

Y'know what...we can even refactor our grid drawing code. From `colorBox`, we'll call `reOutline` with the row and column. `reOutline` will do just that: re-outline the given square.

```javascript
function reOutline(row, col) {
  ctx.lineWidth = 0.5;
  ctx.setTransform(1, 0, 0, 1, 0, 0);
  ctx.translate(0.5, 0.5);
  ctx.beginPath();
  ctx.moveTo(col*BOX_SIDE_LENGTH, row*BOX_SIDE_LENGTH);
  ctx.lineTo(col*BOX_SIDE_LENGTH, (row+1)*BOX_SIDE_LENGTH);
  ctx.stroke();
  ctx.lineTo((col+1)*BOX_SIDE_LENGTH, (row+1)*BOX_SIDE_LENGTH);
  ctx.stroke();
  ctx.lineTo((col+1)*BOX_SIDE_LENGTH, row*BOX_SIDE_LENGTH);
  ctx.stroke();
  ctx.lineTo(col*BOX_SIDE_LENGTH, row*BOX_SIDE_LENGTH);
  ctx.stroke();
  ctx.closePath();
}
```

Finally, we can write a `resetCanvas` function that will redraw the entire canvas with the grid. `resetCanvas` is just a special case of `drawData()`, where *all the cells* are dirty.

```javascript
/* Reset canvas: mark all as dirty */
function resetCanvas() {
  for (var i=0;i<canvasData.length;i++) {
    canvasData[i]=DEFAULT_COLOR;
    dirtyIndices.push(i);
  }
}
```

Ahh, so much better.

### We're Off!

We'll write a function to initialize the editor and one to set Event Listeners. It might seem like overkill now, but we'll build them out later on to finish off the app.

```javascript
function addListeners() {
  canvas.addEventListener('click', handleClick);
}
function initEditor() {
  getCanvasAndContext();
  resetCanvas();
  addListeners();
}
```

All we need to start the app now is two simple calls. The loop runs, updates only when it needs to, and the user is happy!

```javascript
window.onload = function() {
  initEditor();
  drawData();
}
```

Stay tuned for Parts Two and Three, where we will continue to build out the UI, allow the user to choose different colors, add mouse drag support, and more.

### [Codepen Demo for Part One](https://codepen.io/thmsdnnr/full/WddbeW/)

<p data-height="333" data-theme-id="32039" data-slug-hash="WddbeW" data-default-tab="js" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Pixl: Part One" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/WddbeW/">Pixl: Part One</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

##### Click the URL. Event listeners are funky in CodePen's embed.
