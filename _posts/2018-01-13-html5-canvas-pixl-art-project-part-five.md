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

<img src="/assets/pics/pixl_thinview.png" alt="Pixl viewed on a small, narrow screen" width="300px">
##### small, narrow screen
<img src="/assets/pics/pixl_wideview.png" alt="Pixl viewed at normal screen width." width="300px">
##### normal, large screen

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
```

<style>
div#colorDemo {
  background-color: white;
}
@media all and (min-width: 500px) {
  div#colorDemo {
    background-color: gold;
  }
</style>

<div id="colorDemo">I change color to gold when I am wider than 500px.</div>

Note how we define rules for #colorDemo twice: once for the smaller screen, and once for the larger screen. In mobile-first design, we make our app look good on the smaller screen first, and then we set media queries to override this behavior on a larger screen: perhaps by widening the elements, or letting them take up additional columns.

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
  <div class="row">
    <div id="paletteControls">
     <button id="randomPalette">rando</button>
     <button id="reset">reset</button>
     <input class="jscolor" id="swatch">
     </div>
    <div id="palette"></div>
 </div>
<div class="row">
  <div id="savedImages"></div>
  <div id="canvas"><canvas id="editor"></canvas></div>
</div>
<div class="row">
  <div class="controls">
  <div class="row">
      <button id="saveImage">save image</button>
      <button id="loadImages">load images</button>
  </div>
  <div class="row">
      <div class="control"><button id="getImage">get image</button></div><br />
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
  </div>
  <div class="row"><div id="imageTray"></div></div>
</div>
```

We have a large container wrapper. The container on narrower screens:

```css
width: 95%;
min-width: 442px;
padding: 5px;
margin: 5px;
display: flex;
flex-flow: column nowrap;
justify-content: space-around
```

On wider screens:

```css
margin: 5px auto;
width: 680px;
```

We create a `row` class that is used to wrap individual rows of content. For narrow screens:

```css
display: flex;
flex-flow: row wrap;
align-items: center;
```

On wider screens:
```css
flex-flow: row nowrap;
```

See the difference? When the screen is small, it's fine for the rows' inside elements to wrap around. On a larger screen, we want the elements to stretch out.

div.controls and div#paletteControls are both flexboxes that contain flex children elements.

The #palette is a flexbox that contains `div.chip` flex child elements.

`#savedImages` is a flexbox that holds the images in a column with flex children of canvas elements that are saved in the user's localstorage.

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

Whenever the screen size changes, we redraw the canvas and auxiliary canvases at a fixed ratio based on the initial height and width (the `HEIGHT_BASIS` and `WIDTH_BASIS` variables). By providing a logical breakpoint between screen sizes, we enable the user to access the app on many different-sized devices.

### Codepen with refactored CSS [link here](https://codepen.io/thmsdnnr/full/JMBPzy/)
