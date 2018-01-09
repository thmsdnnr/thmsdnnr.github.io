---
layout: post
title:  "Pixl: Scaling & Saving Art [part 3]"
date:   2018-1-9 09:30:04 -0500
description: Walkthrough/tutorial on creating a pixel art editor using JS and HTML5 Canvas.
author: Thomas Danner
lang: en_US
categories: tutorials javascript projects
tags: JS, events, canvas, pixelArt, editor, tinycolor, palette, UI
---

<img src="/assets/pics/pixl_3.png" alt="Image of pixel editor displaying a greeting" width="300px" height="auto">

## [codepen link to working demo](https://codepen.io/thmsdnnr/full/dJdRap/)

In [part one](http://thmsdnnr.com/tutorials/javascript/projects/2018/01/07/html5-canvas-pixl-art-project-part-one.html) of our tutorial, we got a pretty good start with a grid that refreshes gracefully based on a data array. The user can toggle a single colors on and off for each pixel.

In [part two](http://thmsdnnr.com/tutorials/javascript/projects/2018/01/08/html5-canvas-pixl-art-project-part-two.html) we added a color palette and mouse-dragging functionality to paint pixels more quickly.

Now we're going to enable the user to save their masterpiece. We'll offer the option to scale to 16px, 32px, 64px, or 128px and the choice of transparent vs solid color backgrounds. Finally, we'll display all the generated canvases in a tray at the bottom of the app.

### Let's get started.

First we'll create a div to hold the controls and the image tray of completed canvases.

```html
<div class="controls">
  <div class="control"><button id="getImage">get image</button></div>
  <div class="control">transparent<input id="transparent" type="checkbox" checked></div>
  <div class="control">bgcolor <input class="jscolor" id="bgColor"></div>
  <div class="control">width
     <select id="pxWidth">
         <option value="16">16px</option>
         <option value="32">32px</option>
         <option value="64">64px</option>
         <option value="128" selected>128px</option>
     </select>
  </div>
</div>
<div id="imageTray"></div>
```

Note the use of `checked` and `selected` in our `input` and `options` tags: this attribute specifies the default selected value. We're allowing the user to scale the drawing to some select multiples of 16. You could enable the user to input a number and scale to any size they'd like, but for simplicity, we're limiting the range to four options.

See [Hick's Law](https://www.interaction-design.org/literature/article/hick-s-law-making-the-choice-easier-for-users): the more choices you give the user, the longer it takes them to make a decision. **Corollary: if you give a user too many choices, they might make the decision to close the tab.**

Generating the canvases requires two steps.

1. Listen to the "get image" click event & get the parameters from the user
2. Scale the canvas based on these parameters and display it in the `imageTray`

Let's do the easy part first.

#### 1. Listen and Fetch Parameters

```javascript
document.getElementById('getImage').addEventListener('click', getUserImageParameters);
document.getElementById('bgColor').addEventListener('change', updateBackgroundColor);
```

We add a listener to the background color picker. By default, backgrounds are transparent, and we have the transparency checkbox ticked. When the user changes the background color, we simply untick the transparency box. If the user re-checks transparency, we will not display any background color, even if the user has already selected one.

Unchecking the box is one way to help out the user by anticipating their intention--choosing a background color for the image means they probably want one—-but providing a quick visual if they change their minds.

```javascript
function updateBackgroundColor(e) {
  let T=document.querySelector('input#transparent');
  T.checked=false;
}
```

Our `getUserImageParameters` handler grabs the values of the pixel width, transparency, and background color at the time the button is clicked and calls `grabCanvas` to display the user's image: the function that we're about to write!

```javascript
function getUserImageParameters(e) {
  e.preventDefault();
  let W=document.querySelector('select#pxWidth').value;
  let T=document.querySelector('input#transparent').checked;
  let BG='#'+document.querySelector('input#bgColor').value;
  grabCanvas(W, T, BG);
}
```

#### 2. Scale the canvas

Now that we know what size canvas the user wants, we need to scale it and set the background color (either to transparent or to the selected color).

We'll write a function `grabCanvas` that takes a width, transparent flag (true/false) an a bgColor from the user. It will output a canvas to the display that is scaled and has the specified background color.

In order to scale the main canvas, we're going to create a second, smaller canvas. We number it with a unique id that we increment on the page with the creation of each canvas, `cvsID`. We give the canvas a height & width based on the user selection. We add a class to the canvas so that we can style it with CSS, and then we use `prepend` to add it to the `imageTray` div.

We use [`prepend`](https://developer.mozilla.org/en-US/docs/Web/API/ParentNode/prepend) to add the canvases instead of [`appendChild`](https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild) because we want the newest canvases to appear on the leftmost side of the div. When more canvases are added than can be viewed, we'll automatically scroll to the far left of the div so the most recent canvas can be viewed.

```javascript
cvsID++;
let blit=document.createElement('canvas');
blit.height=width; //px
blit.width=width;
blit.classList.add('c-output');
blit.id='cvs_no'+cvsID;
let blitCtx=blit.getContext('2d');
const cellDim=Math.ceil(width/NUM_COLS);
let tray=document.getElementById('imageTray')
tray.prepend(blit);
tray.scrollLeft=0;
```

Now that we have the canvas on screen, we just have to draw the cells.

To do that I wrote a box-drawing function called `colorScaledBox`. It's very similar to `colorBox`. Indeed, we could have simply refactored `colorBox`, but there would be a bit of complication involved: we'd have to rework `colorBox` to take a context and a flag for whether a grid should be drawn.

```javascript
const cellDim=Math.ceil(width/NUM_COLS);
function colorScaledBox(box, color) {
  const { row, col } = box;
  if (!isValidCol(col)||!isValidRow(row)) { return false; }
  blitCtx.fillStyle=color || currentColor;
  blitCtx.clearRect(col*cellDim, row*cellDim, cellDim, cellDim);
  blitCtx.beginPath();
  blitCtx.fillRect(col*cellDim, row*cellDim, cellDim, cellDim);
  blitCtx.closePath();
}
```

Setting the background color is actually quite easy. We loop through the canvas data and use our new function `colorScaledBox`.

To determine which colors to change (either to a specified background color, or to a transparent square), we compare the color of each square to the `DEFAULT_COLOR`. If the square's color is the same as `DEFAULT_COLOR`, this means that the user never painted here. We either set the square to transparent or to the specified background color.

Otherwise, the user did paint here. We ignore the background color parameters since this square is not part of the background and we paint the user's color selection.

```javascript
for (var i=0;i<canvasData.length;i++) {
  let row=Math.floor(i/NUM_ROWS);
  let col=i%NUM_COLS;
  let color=canvasData[i];
  if (transparent) {
    if (color===DEFAULT_COLOR) {
      colorScaledBox({row, col}, 'rgba(255,255,255,0)');
    } else { colorScaledBox({row, col}, color); }
  } else {
    if (color===DEFAULT_COLOR) {
      colorScaledBox({row, col}, bgColor);
    } else { colorScaledBox({row, col}, color); }
  }
}
```

### Saving Off Images

The user can now just Right-click > Save Image on a canvas, and they get precisely-sized pixel art:

> <img src="/assets/pics/pixl_16.png" alt="Image saying hi generated with pixl" width="16px" height="16px">
<img src="/assets/pics/pixl_32.png" alt="Image saying hi generated with pixl" width="32px" height="32px">
<img src="/assets/pics/pixl_hi.png" alt="Image saying hi generated with pixl" width="64px" height="64px">
<img src="/assets/pics/pixl_128.png" alt="Image saying hi generated with pixl" width="128px" height="128px">
##### faaaaantastic

### Right click *save-as*?!

I hear ya. You're groaning at the desk. Is this 1995? I thought this was a web app! Never fear my friend. In the next part of the tutorial, we'll cover usability enhancements such as these:

* allowing the user to delete canvases from the `imageTray`
* maintaining a "gallery" of art (saving and deleting via localStorage for later editing)
* a more graceful way to save images from the app

### And what about the CSS?!

It's true that I have not covered styling for the app yet and am just throwing in some basic CSS to make things look decent as we move along. At the end of the tutorial series, I will cover styling extensively and show how we can make the app appear responsive at different screen sizes using mobile-first design and media queries. Stay tuned! I *do* have a day job :).

##### (but it's not coding, and I'd like it to be, so reach out if you've got a project.)
