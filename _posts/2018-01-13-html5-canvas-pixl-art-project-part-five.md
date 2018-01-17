---
layout: post
title:  "Pixl: A Sea of CSS [part 5]"
date:   2018-1-12 09:30:04 -0500
description: Walkthrough/tutorial on creating a pixel art editor using JS and HTML5 Canvas.
author: Thomas Danner
lang: en_US
categories: tutorials javascript projects
tags: JS, events, canvas, pixelArt, editor, tinycolor, palette, UI, CSS
---

## [codepen link to working demo](https://codepen.io/thmsdnnr/full/JMBPzy/)

In [part one](http://thmsdnnr.com/tutorials/javascript/projects/2018/01/07/html5-canvas-pixl-art-project-part-one.html) of our tutorial, we got a pretty good start with a grid that refreshes gracefully based on a data array. The user can toggle a single colors on and off for each pixel.

In [part two](http://thmsdnnr.com/tutorials/javascript/projects/2018/01/08/html5-canvas-pixl-art-project-part-two.html) we added a color palette and mouse-dragging functionality to paint pixels more quickly.

In [part three](http://thmsdnnr.com/tutorials/javascript/projects/2018/01/09/html5-canvas-pixl-art-project-part-three.html) we created an image tray at the bottom of the screen where users could grab different-sized canvases of their pixel art to save on their PC.

In [part four](http://thmsdnnr.com/tutorials/javascript/projects/2018/01/12/html5-canvas-pixl-art-project-part-four.html) we questioned whether we ought to be using React after all, and we added support for saving canvas images in localStorage.

### What's happening today?

Now we're going to push the project to the limit: we'll add CSS styling to make the app responsive on multiple screen sizes using media queries and flexbox.

##### small, narrow screen

<img src="/assets/pics/pixl_five_one.png" alt="Pixl viewed at normal screen width." width="300px">

<img src="/assets/pics/pixl_five_three.png" alt="Pixl viewed on a small, narrow screen" width="300px">

<img src="/assets/pics/pixl_five_two.png" alt="Pixl viewed on a small, narrow screen" width="300px">

### Media queries

A [media query](https://en.wikipedia.org/wiki/Media_queries) allows us to define rules for styling elements differently depending on *media type* (e.g., print, screen, TV, braille) and a list of *media features* that include `color`, `width`, and `device-aspect-ratio`. Media queries allow us to define a an element once in our HTML, and then make it appear differently depending on context.

Here's a simple example.

```css
div#colorDemo {
  background-color: white;
}
@media all and (min-width: 500px) {
  div#colorDemo {
    background-color: gold;
  }
}
```

<style>
div#colorDemo {
  background-color: white;
}
@media all and (min-width: 500px) {
  div#colorDemo {
    background-color: gold;
  }
}
</style>

<div id="colorDemo">I change color to gold when I am wider than 500px.</div>

Note how we define rules for #colorDemo twice: once for the smaller screen, and once for the larger screen. In mobile-first design, we make our app look good on smaller screens first, and then we set media queries to override this behavior on a larger screen: perhaps by widening the elements, or letting them take up additional columns.

This is done since styling for smaller screens is usually simpler. Most things end up being full-width and margins and padding do not come into play so much.

We can leverage media queries in JavaScript too to change our code dynamically. It looks quite similar to the CSS. The `window` object has a `matchMedia` method that allows us to add listeners to a given CSS query of interest.

```javascript
var mediaQueryList = window.matchMedia("(min-width: 500px)"); // Create the query list.
  function handleOrientationChange(mql) {
    isLargeScreen = mql.matches ? true : false;
    if (!isLargeScreen) { //transition to small screen
      redrawAtScale(0.8);  
    } else { //transition to large screen
      redrawAtScale(1);
    }
  }
mediaQueryList.addListener(handleOrientationChange); // Add the callback function as a listener to the query list.
handleOrientationChange(mediaQueryList); // Run the orientation change handler once.
```

We'll add an event listener to our window width and use it to dynamically change the size of the grid and the size of the preview thumbnails.

We'll also add some CSS to make the page look nice whether the screen is small or large.

### First the HTML

```html
<div id="container">
  <div id="header"><h1>pixl</h1> // <a href="http://github.com/thmsdnnr/pixl">repo</a></div>
  <div class="row" id="row_0">
    <div id="paletteControls">
       <div class="control"><button id="randomPalette">rando</button></div>
       <div class="control"><button id="reset">reset</button></div>
       <div class="control"><input class="jscolor" id="swatch"></div>
    </div>
   <div id="palette"></div>
 </div>
  <div class="row" id="row_1">
    <div id="savedImages"></div>
    <div id="canvas"><canvas id="editor"></canvas></div>
  </div>
  <div class="row" id="row_2">
    <div class="controls">
      <div class="row">
        <button id="saveImage">save image</button>
        <button id="loadImages">load images</button>
      </div>
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
  </div>
  <div class="row" id="row_3"><div id="imageTray"></div></div></div>
</div>
```

## Styling this HTML

There are a few global styles.

```css
html { box-sizing: border-box; }
*, *:before, *:after { box-sizing: inherit; }

body {
  background-color: #FFF;
  cursor: crosshair;
  font-family: 'Lekton', sans-serif;
}

body, h1, input, button, select {
  font-family: 'Lekton', sans-serif;
}
```

### side note — box-sizing: border-box

Setting `box-sizing: border-box` allows us to "unquirk" the [typical CSS box sizing behavior](https://css-tricks.com/box-sizing/).

Without this setting, the width and height of elements will not be what you specify. Rather, elements will take the specified width and height as a base, adding on any padding and border that you specify (or, if you do not do a CSS reset, that a browser stylesheet has by default).

This can result in some strange behavior. Consider this box. Without `border-box` set, it will display as 106x106 px instead of 100x100.

```css
div#box {
  width: 100px;
  height: 100px;
  padding: 5px;
  border: 1px solid black;
}
```

Now imagine that you have a div to hold a row of boxes, and you'd like to display six per line. Well, you can't make it 600px, because it's going to have to hold 600 + 6*6 = 636px of width total.

You could just resize the container div. But now imagine you want to tweak the padding slightly on your elements. You'll have to go back and resize the container div.

### Row by row

We have a large container wrapper that holds all of our rows. Here's what the container looks like on narrower screens:

```css
div#container {
  display: flex;
  flex-flow: column wrap;
  width: 99%;
  max-width: 680px;
  border: 1px solid black;
  margin: 0px auto;
  padding: 10px 4px;
}
```
We set a maximum width in pixels so that the percentage width does not become too large on large screens.

We have a line of content for each "row" on the screen.

We create a `row` class that is used to wrap individual rows of content. It looks the same on any type of screen size, and it scales automatically given that it's a flexbox.

```css
div.row {
  display: flex;
  flex-flow: row wrap;
  align-items: center;
}
```

The rows are:

1. Header
2. Color Palette & Palette Controls
3. localStorage Saved Images & Drawing Canvas
4. Canvas Controls
5. User-generated images to save to computer

Let's describe row by row what the style will look like for large and small screens.

### Header

When the screen is 450px or wider, we display the header.
```css
div#header {
  display: block;
  font-size: 0.7em;
  width: 90px;
  border-bottom: 1px solid tomato;
  user-select: none;
}
```

Otherwise, the header is hidden.

`div#header { display: none; }`

### Color Palette & Palette Controls

The palette is a div container filled with color "chips".

```css
div#palette {
  flex: 6;
  padding: 8px;
  flex-flow: row nowrap;
  height: 32px;
  display: flex;
  align-items: center;
  box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
  user-select: none;
}

div.chip {
  flex: 2;
  width: auto;
  height: 100%;
  padding: 2px;
  border-radius: 6px;
}
```

Chips can have a class `activeColor` when they are selected. `activeColor` gives a visual indication that the color is selected by spinning the chip around, raising it slightly, and giving it a box-shadow.

```css
div.activeColor {
  transform: rotate(-90deg) translateX(11px);
  transition: 200ms all ease-in-out;
  box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
}
```

The controls are a div container filled with control divs.

```css
div#paletteControls {
  display: flex;
  flex: 1;
  padding: 8px;
  flex-flow: row nowrap;
  height: 32px;
  justify-content: center;
  box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
  user-select: none;
}

div.control {
  flex: 0 1 auto;
  padding: 2px;
  align-self: center;
}
```

### localStorage Saved Images & Drawing Canvas

The saved images are those that are stored in localStorage and can be retrieved for future editing. On small screens, they'll appear as a single row across the top of the page. We set `flex-flow` to row nowrap and `overflow-x` to cause the container to scroll when there are too many images inside to appear at once.

```css
div#savedImages {
  flex: 0 1 auto;
  display: none;
  justify-content: flex-start;
  align-items: center;
  flex-flow: row nowrap;
  overflow-x: scroll;
  width: 90%;
  margin: 4px auto;
}
```

For screens wider than 700px, we want the saved images to appear as a vertical column to the left of the canvas. Here, we set the `flex-flow` to column and set a height on the div:

```css
div#savedImages {
  flex: 0 1 140px;
  flex-flow: column;
  height: 430px;
 }
```

The canvas is where it all (well, all the drawing at least) happens.

```css
div#canvas {
  flex: 0 1 auto;
  margin: 4px auto;
  padding: 5px;
  height: auto;
  box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
}

canvas#editor {
  margin: 4px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
}
```

### Canvas Controls

The controls (buttons like save/load/get image and background settings) will all appear as a single row below the canvas.

```css
div.controls {
  flex: 0 1 auto;
  margin: auto;
  text-align: center;
  font-size: 0.9em;
  display: flex;
  flex-flow: row nowrap;
  justify-content: center;
  align-items: center;
  user-select: none;
}
```

### User-generated images to save to computer

```css
div#imageTray {
  display: flex;
  margin: auto;
  width: 90%;
  white-space: nowrap;
  overflow-x: scroll;
  padding: 5px;
  transition: 200ms all ease-in;
  user-select: none;
}
```

## We set a few more styles everywhere the same:

```css
input.jscolor { cursor: grab; }

button {
  outline: none;
  font-size: 0.9em;
  max-width: 110px;
  flex: 0 1 auto;
}

h1 {
  display: inline;
  font-size: 2em;
}

input#swatch {
  max-width: 52px;
  padding: 0px;
  outline: none;
  user-select: none;
  flex: 0 1 auto;
}

input#bgColor {
  width: 50px;
  padding: 0px;
  outline: none;
}

select#pxWidth {
  width: 64px;
  font-family: 'Lekton', sans-serif;
  font-size: 0.9em;
}
```

Canvas preview containers hold canvases saved in localStorage, or canvases that the user has generated with the "get image" button. These containers look the same everywhere. A little box-shadow makes them pop from the page, and a border on hover gives the user a hint that they are clickable and have associated actions.

```css
div.cvs-preview {
  position: relative;
  flex: none;
  margin: 5px;
  border: 1px solid black;
  box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
  user-select: none;
}

div.cvs-preview:hover {
  border: 1px solid gold;
}
```

#### Dynamically controlling row orientation

On largest screens, I want to have the controls displayed above the preview images. But on smaller screens, I want to. move the controls to the very bottom and display preview images closer to the canvas.

Flexbox allows you to dynamically switch the order of elements using the `order` property. I number each row with an ID, `row_0` through `row_3`.

I can then swap the position of the last two rows on smaller screens like this:

```css
#row_3 { order: 0; }
#row_2 { order: 1; }
```

On larger screens, I can flip the order back to normal.
```css
#row_3 { order: 1; }
#row_2 { order: 0; }
```

Nice!

By using [CSS flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) judiciously, we're able to define a simple, row-based layout that collapses to a single column of elements on smaller screens.

### Making the canvas smaller/larger

We are able to toggle the canvas size and appearance using the JavaScript media query listener we defined earlier. Remember the `handleOrientationChange` bit we wrote earlier?

We also wrote the method `redrawAtScale`. This function does the following:

```javascript
let WIDTH = 400;
let WIDTH_BASIS = 400; //px
let HEIGHT_BASIS = 400;
let HEIGHT = 400; //px

WIDTH=n*WIDTH_BASIS;
HEIGHT=n*HEIGHT_BASIS;
PREVIEW_IMAGE_SCALE=n;
getCanvasAndContext();
forceRedraw();
```

Whenever the screen size changes, we redraw the canvas and auxiliary canvases at a fixed ratio based on the initial height and width (the `HEIGHT_BASIS` and `WIDTH_BASIS` variables). By providing a breakpoint between screen sizes, we enable the user to access the app on different-sized devices.

There you have it. TIL about responsive design, flexbox, and mobile-first CSS!

### Codepen with refactored CSS [link here](https://codepen.io/thmsdnnr/full/JMBPzy/)
