---
layout: post
title:  "Pixl: Scaling & Saving Art [part 4]"
date:   2018-1-12 09:30:04 -0500
description: Walkthrough/tutorial on creating a pixel art editor using JS and HTML5 Canvas.
author: Thomas Danner
lang: en_US
categories: tutorials javascript projects
tags: JS, events, canvas, pixelArt, editor, tinycolor, palette, UI
---

### A note to the reader.

To be frank, this project was an experiment to see how far one could go with VanillaJS without using any outside frameworks like React, Redux, Angular, or Vue. I think it has been an interesting exercise. You can get quite far! It also becomes clear that after trying to juggle events and showing/hiding divs that it'd be nice to have components for UI that updated automatically when they received a change in their slice of state.

Perhaps these exercises can, in addition to showing you some techniques of JS and Canvas, also illustrate the problems that these view frameworks were designed to solve. A potential topic for a future tutorial would be a rewrite of Pixl in React+Redux, showing how the UI is simplified and we can more easily add features to the app.

## Now on to part 4!

<img src="/assets/pics/pixl_4.png" alt="Image of pixel editor displaying a greeting with saved image pane on the left" width="400px" height="auto">

## [codepen link to working demo](https://codepen.io/thmsdnnr/full/MrXemo/)

In [part one](http://thmsdnnr.com/tutorials/javascript/projects/2018/01/07/html5-canvas-pixl-art-project-part-one.html) of our tutorial, we got a pretty good start with a grid that refreshes gracefully based on a data array. The user can toggle a single colors on and off for each pixel.

In [part two](http://thmsdnnr.com/tutorials/javascript/projects/2018/01/08/html5-canvas-pixl-art-project-part-two.html) we added a color palette and mouse-dragging functionality to paint pixels more quickly.

In [part three](http://thmsdnnr.com/tutorials/javascript/projects/2018/01/09/html5-canvas-pixl-art-project-part-three.html) we created an image tray at the bottom of the screen where users could grab different-sized canvases of their pixel art to save on their PC.

### What's happening today?

Now we're going to enable the user to save off their masterpiece in localStorage and restore these saved images at a later date for further editing. In addition to loading images, we'll also support the deletion of these images in case the user does not want them any longer.

### Let's get started.

First we'll create a div to hold the saved image pane on the left-hand side of the canvas. We'll put it right between the color palette and the canvas.

```html
<div id="palette"><button id="randomPalette">rando</button>/<button id="reset">reset</button><input class="jscolor" id="swatch"></div>
<div id="savedImages"></div>
<div id="canvas"><canvas id="editor"></canvas></div>
```

We'll also add two buttons to the toolbar. One will save the current image to the list of images in localStorage. The other will toggle open the saved image pane for the user to select from.

```html
<button id="saveImage">save image</button>
<button id="loadImages">load images</button>
```

In order to make these new buttons work, we have to:

1. Listen to the two new buttons we created.
2. When the user clicks "Save Image", we must take the current `canvasData` array and save it off to an array of images in localStorage.
3. When the user clicks "Load Images", we must grab the canvasData from localStorage (if it exists), and display each image in the preview pane.

In the preview pane, we allow users to:

1. Select an image for editing
2. Delete an image from storage

#### 1. Listen and Fetch Parameters

```javascript
document.getElementById('saveImage').addEventListener('click', saveImage);
document.getElementById('loadImages').addEventListener('click', loadImages);
```

Save image is our handler that fires when the user decides to save the current canvas image.

If you aren't familiar with localStorage, check out [my tutorial on it](http://thmsdnnr.com/tutorials/javascript/fundamentals/2018/01/02/code-morsels-localstorage.html) *cough shameless plug*.

localStorage in one line: a key-value store persistent across browser sessions that our app can use to save data on the user's machine (until they clear out their cookies+other saved data of course).

#### Save the image

```javascript
function saveImage(e) {
  let saved=localStorage.getItem('pixl');
  if (!saved) { //initialize save object
      let saveObj=JSON.stringify({
          images: [canvasData]
      });
      localStorage.setItem('pixl', saveObj);
  } else { //retrieve image array and place new image at the front
      saved=JSON.parse(saved);
      let newImages=saved.images.slice();
      newImages.push(canvasData);
      let saveObj=Object.assign({}, saved, {images:newImages});
      localStorage.setItem('pixl', JSON.stringify(saveObj));
  }
  loadImages();
}
```

We store the images as an object with a key `images`, mapping to an array of `canvasData` arrays.

When the user clicks `saveImage`, we first check to see if an array of saved images exists. If it does not, we initialize it with the `images` array containing only this current canvas. Then we use `setItem` to set our stringified object under the `pixl` key in storage.

If the array already exists, we make a copy of `saved.images`, push our image onto the copy, and re-stringify the object with our new image, assigning it to localStorage.

You might be asking yourself: "Hey, what's all this [`Object.assign()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) mess?"

`Object.assign` is a shortcut to copy values from source objects to a target object.

What if we decide to save other things in the `pixl` key in storage: say, the time the user last visited, or editor settings like the active color? I'm defining my image storage as existing under the `images` key.

Here, we copy everything from the saved object into an empty object `{}`. Then we copy everything from the `images` key we just modified into this new object. This means that if we have other keys in `saved`, they'll be preserved. We only save off the `images` "slice" of the program's state.

#### Immutable State

I'm enforcing [immutability](https://en.wikipedia.org/wiki/Immutable_object) here: the idea that once an object is created, it cannot be modified. Instead of mutating the original value I have stored in localStorage, I create a new object, explicitly assigning to one key of that object.

If you use [Redux](https://redux.js.org/) or [Immutable](https://facebook.github.io/immutable-js/), this pattern will already be familiar to you. If not, it's worth reading about. Having an immutable state as a single source of truth can save a lot of hair-pulling when you are debugging your code.

#### Load the images

Loading images is a bit more complicated, since once we load them, we will have to display them in the pane on the left for the viewer to see. First, we'll grab them from localStorage, making sure that the data exists before we call `JSON.parse()` on it! If the data doesn't exist, we return false. We don't want to show an empty pane.

```javascript
function loadImages(e) {
  let saved=localStorage.getItem('pixl');
  if (!saved) { return false; }
  loadedImages=JSON.parse(saved);
}
```

We create a `loadedImages` global variable to hold the currently loaded images.

Okay, we've got the data. Now how do we display it?

You might remember the  [`grabCanvas`](http://thmsdnnr.com/tutorials/javascript/projects/2018/01/09/html5-canvas-pixl-art-project-part-three.html#2-scale-the-canvas) method from the last tutorial. It's what we used to grab `canvasData` and "blit" it onto a new canvas at the bottom of the screen at a user-specified size, background color, and transparency.

Previously, it took three arguments: width, transparency (true/false), and background color. I've added a fourth argument, `data`, so that grabCanvas can now make us a canvas with an arbitrary data array. We have data arrays corresponding to saved images stored in localStorage...I think you see where we're going with this.

We can draw arbitrary data by changing just two lines.

`function grabCanvas(width, transparent, bgColor) {`

becomes

`function grabCanvas(width, transparent, bgColor, data) {`,

and

`for (var i=0;i<canvasData.length;i++) {`

becomes

`for (var i=0;i<data.length;i++) {`

That's not quite enough though. I also modularized `grabCanvas`. Previously, I had the function not only grab the current canvas state and create a new canvas: I also had it *append that canvas to the DOM in the image pane at the bottom*.

This is a **side-effect**. A function is said to have [side effects](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) if it "modifies state outside its scope" or does something like a database call or a DOM manipulation.

Side effects are not bad. This is good, because they are inevitable. The key is to make them predictable and modular enough so that they do not impact how the rest of your code is structured.

In this case the side effect is detrimental. Can you see why? We want to have a separate pane to display saved images, but the pane where `grabCanvas` writes to has been hard-coded in.

Previously, we had this:

```javascript
let blit=document.createElement('canvas');
blit.height=width; //px
blit.width=width;
blit.classList.add('c-output');
blit.id='cvs_no'+cvsID;
let blitCtx=blit.getContext('2d');
const cellDim=Math.ceil(width/NUM_COLS);
let tray=document.getElementById('imageTray') // we hard-coded in imageTray and
tray.prepend(blit); // added our new canvas to it directly. SIDE EFFECT!
tray.scrollLeft=0;
```

Why not just **return** blit (scaled canvas) and let the calling code decide where (or if!) to place it?

```javascript
let blit=document.createElement('canvas');
blit.height=width; //px
blit.width=width;
blit.classList.add('c-preview');
blit.id='cvs_'+cvsID;
let blitCtx=blit.getContext('2d');
const cellDim=Math.ceil(width/NUM_COLS);
// ... //
return blit;
```

We can make use of our new function to draw saved images in the pane we added on the left side of the page.

Let's finish up `loadImages`. For each image, we:

1. Create a container div
2. Give that div a unique id (which we'll use to enable click-and-select + delete saved)
3. Add a close box div (a little circular "click to delete" div)
4. Append the close box to the container
5. Grab the scaled-down canvas from `grabCanvas`, passing in `img` (the imageData for this image)
6. Append the canvas to the container from Step 1
7. Append the container div to the saved image pane to the left of the canvas.

```javascript
let sI=document.getElementById('savedImages');
sI.innerHTML=''; //clear out contents of div, if it already has been populated
loadedImages.images.map((img,idx)=>{
  let blitContainer=document.createElement('div');
  blitContainer.id='container_close_'+idx;
  blitContainer.classList.add('blit-container');
  let closeBox=document.createElement('div');
  closeBox.id='close_'+idx;
  closeBox.classList.add('close');
  blitContainer.appendChild(closeBox);
  let canvas=grabCanvas(128, true, null, img);
  canvas.id='close_'+idx;
  blitContainer.appendChild(canvas);
  sI.prepend(blitContainer);
});
sI.addEventListener('click', handleSavedPaneClick);
sI.style.display='flex';
}
```

We make sure to clear out the saved images div prior to writing these images (say it's already been populated!).

We also add an event listener to handle clicks on the saved images pane and set the images pane display style to `'flex'` (previously it did not appear, because its CSS display is `'none'`).

We refactor our old function to use our newly refactored `grabCanvas` method. Instead of just calling `grabCanvas`, it calls `grabCanvas` and then mounts the returned canvas in its right place.

```javascript
function getUserImageParameters(e) {
  e.preventDefault();
  let W=document.querySelector('select#pxWidth').value;
  let T=document.querySelector('input#transparent').checked;
  let BG='#'+document.querySelector('input#bgColor').value;
  let tray=document.getElementById('imageTray');
  tray.prepend(grabCanvas(W, T, BG, canvasData));
  tray.scrollLeft=0;
}
```

Phew, things are less messy and interrelated now.

#### Loading saved images on user click

Now our images will display in the pane on the left, but that's rather boring. Wouldn't it be nice if the user could scroll through the saved images, click on one, and have it appear back in the canvas for editing?

```javascript
function forceRedraw() { dirtyIndices=canvasData.map((e,idx)=>idx); }

function selectImage(e) {
  let saved=localStorage.getItem('pixl');
  if (!saved) { return false; }
  saved=JSON.parse(saved);
  let savedIdx=e.target.id.split("_")[1];
  canvasData=saved.images[savedIdx];
  forceRedraw();
  //close saved images pane
  document.getElementById('savedImages').style.display='none';
}
```

We've assigned each preview image a unique ID which we can access by splitting the ID on `_` and grabbing the second part of that split, the ID number. If we want to load up the image, we set `canvasData` equal to that entry in our `savedImages` array and call `forceRedraw`. `forceRedraw` marks all the canvas indices as dirty. The next `requestAnimationFrame`, all the canvas squares will be redrawn, displaying our newly-loaded image on the canvas.

When we load the image, we close the preview pane, setting the display style back to `'none'`.

We write the first part of our `handleSavedPaneClick` handler to invoke `selectImage`:

```javascript
function handleSavedPaneClick(e) {
  if (e.target.classList.contains('c-preview')) { selectImage(e); }
}
```

#### Deleting saved images on user click

First we need a function to delete an image from localStorage. Just as we did with loading images, we pick out this image based on its index, which we can get from the click event.

To delete the image, we simply call `filter` on the `loadedImages.images` array to return an array with everything but the image of the index we want to remove. Then we write this filtered array to localStorage.

To make the UI reflect the changed state, we also must remove the preview canvas from the pane.

Here's the function to delete the image, given its index, and save the updated array back to localStorage:

```javascript
function deleteImageByIdx(imgIdx) {    
  //remove image from localStorage
  let saved=localStorage.getItem('pixl');
  if (!saved) { return false; }
  saved=JSON.parse(saved);
  let filtered=saved.images.filter((e,idx)=>idx!==imgIdx);
  let saveObj=Object.assign({}, filtered, {images:filtered});
  localStorage.setItem('pixl', JSON.stringify(saveObj));
}
```

And here's the second part of this logic in our `handleSavedPaneClick` method. We only want to delete the image if the user clicked delete (if the user clicked the image itself, we want to **load** the image!). SO we only listen to click events with the class `close`. If this click happens, we grab the index based on the `id` of the click event and invoke `deleteImageByIdx` with the id.

Then, we select the `div` that holds the saved images, select the div that holds the image we wish to delete, and call `parent.removeChild(child)` to remove the deleted div from the DOM.

```javascript
if (e.target.classList.contains('close')) {
  let imgIdx=Number(e.target.id.split("_")[1]);
  deleteImageByIdx(imgIdx);
  let savedPane=document.getElementById('savedImages');
  let closing=document.getElementById('container_close_'+imgIdx);
  savedPane.removeChild(closing);
}
```

### Stay tuned for CSS

We've added all the features to this project for now. It's been a good run. In the last installment, we'll cover project styling and optimize the CSS.
