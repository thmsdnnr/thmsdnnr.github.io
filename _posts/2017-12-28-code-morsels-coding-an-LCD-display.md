---
layout: post
title:  "Coding an LCD Display with SVG"
date:   2017-12-28 09:15:04 -0500
description: Coding an LCD display
author: Thomas Danner
lang: en_US
categories: tutorials javascript projects SVG
tags: SVG, CSS, LCD, javascript, tutorial, generativeSVG
---

### let's code an LCD

<p data-height="376" data-theme-id="32039" data-slug-hash="JMWPOM" data-default-tab="result" data-user="thmsdnnr" data-embed-version="2" data-pen-title="SVG LCD Display" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/JMWPOM/">SVG LCD Display</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

### wait, but why?

It's cool.

### how?

A basic LCD digit consists of [7 segments](https://en.wikipedia.org/wiki/Seven-segment_display). Each segment can be toggled on or off to display a total of 128 (2^7) states. 16 of these states correspond to a visual display of the numbers 0-9 and A-F. We'll also include H, E, and L so we can spell 'HELL0').

Some displays have [more segments](https://en.wikipedia.org/wiki/Fourteen-segment_display). Diagonal segments can be included to display a greater range of letters. Some also contain a decimal point, so that floating-point numbers can be displayed.

To keep things simple, we'll just stick to 0-9, A-F. But after you read this tutorial, try extending the display yourself!

How should we make these digits? We require segments that can easily be scaled to a large range of sizes while maintaining their position. We require segments that are individually manipulatable to generate patterns corresponding to letters. We want this to all happen without messy computation.

We can describe the LCD as a vector. Let's use [Scalable Vector Graphics](https://en.wikipedia.org/wiki/Scalable_Vector_Graphics).

### SVG?

[SVG](https://developer.mozilla.org/en-US/docs/XML_introduction) is a an XML-derived markup language that tells browsers how to draw 2-D graphics.

There are some [SVG "primitives"](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Basic_Shapes) that can be used to draw pictures. These are `<rect>`, `<circle>`, `<ellipse>`, `<line>`, `<polyline>`, `<polygon>`, and `<path>`.

```xml
<svg xmlns="http://www.w3.org/2000/svg" version="1.1"
     viewBox="0 0 32 32" height="32" width="32" style="border: 1px solid black">
<g><circle cx="16" cy="16" r="11"
    fill-color="black" stroke="gold" stroke-width="5"/>
    <circle cx="48" cy="48" r="11"
        fill-color="black" stroke="gold" stroke-width="5"/></g>
</svg>
```
##### This code draws a black circle (fill-color) with center (16px, 16px), radius 11px, and a 5px (stroke-width) outline (stroke). It draws another circle at (48px, 48px) that you can't see yet.

<svg xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 32 32" height="32" width="32" style="border: 1px solid black">
<g><circle cx="16" cy="16" r="11" fill-color="black" stroke="gold" stroke-width="5" /><circle cx="48" cy="48" r="11"
    fill-color="black" stroke="gold" stroke-width="5"/>
    <circle cx="48" cy="48" r="11"
        fill-color="black" stroke="gold" stroke-width="5"/></g>
</svg>

Cool, but wait: why do I only see one circle when there are two circles defined? `viewBox`.

The [viewBox](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/viewBox) SVG tag specifies four values: `min-x, min-y, width, height`.

`viewBox` maps a rectangular area in user-space to the bounds of the SVG element.

This means that if the viewport width and height are equal to the first circle's width and height, that circle fills the entire space. The second circle we drew at center (48px, 48px) exists, but it is hidden.

If we change width and height to 64px each, the first circle will take up the upper-left quadrant of the square, and the second circle will come into view in the bottom-left quadrant:

```xml
<svg xmlns="http://www.w3.org/2000/svg" version="1.1"
     viewBox="0 0 64 64" height="32" width="32" style="border: 1px solid black">
<g><circle cx="16" cy="16" r="11"
    fill-color="black" stroke="gold" stroke-width="5"/>
    <circle cx="48" cy="48" r="11"
        fill-color="black" stroke="gold" stroke-width="5"/></g>
</svg>
```

<svg xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 64 64" height="32" width="32" style="border: 1px solid black">
<g><circle cx="16" cy="16" r="11" fill-color="black" stroke="gold" stroke-width="5" />
<circle cx="48" cy="48" r="11" fill-color="black" stroke="gold" stroke-width="5"/></g>
</svg>

If you think through it, this makes sense. We're mapping a rectangle 2x larger to our drawing, which has the effect of "zooming out" by a factor of 4 (we decreased width by a factor of 2 and area by 2^2).

Takeaway: we can set the viewBox to the bounds of our SVG element. When we change the height and width of the SVG, the element will be **Scaled** according to its **Vector Graphic** definition.

```xml
<svg xmlns="http://www.w3.org/2000/svg" version="1.1"
     viewBox="0 0 32 32" height="128" width="128" style="border: 1px solid black">
<g><circle cx="16" cy="16" r="11"
    fill-color="black" stroke="gold" stroke-width="5"/></g>
</svg>
```
##### Four times as big; just as crisp: no pixelation!

<svg xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 32 32" height="128" width="128" style="border: 1px solid black">
<g><circle cx="16" cy="16" r="11" fill-color="black" stroke="gold" stroke-width="5" /><circle cx="48" cy="48" r="11"
    fill-color="black" stroke="gold" stroke-width="5"/>
    <circle cx="48" cy="48" r="11"
        fill-color="black" stroke="gold" stroke-width="5"/></g>
</svg>

### okay great, circles, but what about the LCD?

To draw the LCD, we can describe each digit as a collection of seven paths in space. We'll compose seven [path elements](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths) per digit. I'll refer to these paths as grains, since they look a bit like grains of rice on the screen.

Path elements are the most powerful elements of SVG. They can be used to create complex paths like lines, curves, arcs, and LCD screens.

A path has a `d` attribute. It contains a single string of browser instructions of how to draw the path.

The string is a combination of numeric coordinates and letter commands. `M` means Move and is followed by an X and Y coordinate. `L` means line-to: draw a line from the current position to the X and Y specified. `h` means draw a horizontal line from the current X position to the given X position (the y-coordinate stays the same). `z` means 'close path', or go back to the start.

So for this grain:

```xml
<path d="M2.1 2L.2 0h31.6l-1.9 2L28 4H4z" fill="tomato"/>
```

1. Move to (2.1, 2).
2. Draw a line to (0.2, 0).
3. Draw a horizontal line to x=31.6.
4. Draw a line to (1.9, 2).
5. Draw a line to (28, 4).
6. Draw a horizontal line to x=4.
7. Return to the original point, (2.1, 2).
8. Fill the enclosed space with tomato.

Profit:
<svg xmlns="http://www.w3.org/2000/svg" height="32" width="32">
  <g fill="#0f0">
    <path d="M2.1 2L.2 0h31.6l-1.9 2L28 4H4z" fill="tomato"/>
  </g>
</svg>
##### look at the picture above and go through the eight steps. Hint: it starts at the bottom-left.

Every path has its own fill color and fill-opacity. We can set the fill color to be whatever color we'd like the digits to appear as, and then we can toggle the fill-opacity to turn a path "on" or "off".

Here's what the entire digit looks like:

```xml
<svg xmlns="http://www.w3.org/2000/svg" height="64" width="32">
  <g fill="#0f0">
    <path d="M2.1 2L.2 0h31.6l-1.9 2L28 4H4z" fill="tomato"/>
    <path d="M0 16V1.2L2 3l1.9 2V29l-2 1-1.8.9z" fill-opacity="0.1"/>
    <path d="M30 30l-2-1V5l2-1.9 2-2V31l-2-1z" fill-opacity="0.3"/>
    <path d="M16 33.9H3.9l-1.9-1L.2 32l.2-.1 1.8-1 1.7-.8H28l1.9 1 1.8 1-1.9.9-1.9 1z" fill-opacity="0.5"/>
    <path d="M0 48L.2 33l2 1 1.8 1v24l-2 1.8-1.8 2z" fill-opacity="0.6"/>
    <path d="M30 60.9l-2-2V35l2-.9 2-1V62.9z" fill-opacity="0.8"/>
    <path d="M1.5 62.6l1.9-2 .6-.5h24l1.4 1.4 1.9 1.9.5.5H.2z" fill-opacity="1.0"/>
  </g>
</svg>
```

<svg xmlns="http://www.w3.org/2000/svg" height="64" width="32">
  <g fill="#0f0">
    <path d="M2.1 2L.2 0h31.6l-1.9 2L28 4H4z" fill="tomato"/>
    <path d="M0 16V1.2L2 3l1.9 2V29l-2 1-1.8.9z" fill-opacity="0.1"/>
    <path d="M30 30l-2-1V5l2-1.9 2-2V31l-2-1z" fill-opacity="0.3"/>
    <path d="M16 33.9H3.9l-1.9-1L.2 32l.2-.1 1.8-1 1.7-.8H28l1.9 1 1.8 1-1.9.9-1.9 1z" fill-opacity="0.5"/>
    <path d="M0 48L.2 33l2 1 1.8 1v24l-2 1.8-1.8 2z" fill-opacity="0.6"/>
    <path d="M30 60.9l-2-2V35l2-.9 2-1V62.9z" fill-opacity="0.8"/>
    <path d="M1.5 62.6l1.9-2 .6-.5h24l1.4 1.4 1.9 1.9.5.5H.2z" fill-opacity="1.0"/>
  </g>
</svg>

See how we can change color and opacity? See how each path is uniquely described by a single `d` attribute that tells the browser how to draw the path? Are you getting excited? We're almost there!

### Code the grains!

It is possible to code SVG without using a GUI, just like we did above with the circles. However, working out the details for drawing each path yourself is tedious and error-prone. An open-source tool like [Inkscape](https://inkscape.org/en/) makes the task much more pleasant. It's how I generated the paths you see above.

This is not an in-depth Inkscape tutorial. But I will give a brief overview of how I drew the digit.

##### File > New From Template > choose "default px".

You'll be presented with a lovely blank canvas. Let's resize it to fit the shape we wish to draw.

##### File > Document Properties > Page tab.

Change the units from "mm" to "px" and set the width and height to 32px and 64px.

##### > Grid tab

Click "New" to generate a rectangular grid. This will give us something for our paths' lines to snap to. Close out of the Properties pane.

##### Edit > Create Guides Around the Page.

Now zoom in and you should have a nice grid on which to begin drawing your crystal.

I made each of my crystal elements 5px wide, with a slight gap between them of about a pixel. I drew the left side and then used Edit > Duplicate (Ctrl+D) and Object > Flip Horizontal (H) to mirror on the right side what I drew on the left to ensure symmetry.

I did the same thing with the top and bottom elements + a flip vertical instead.

All of my grains had four sides, except the middle crossbar: a rectangle with a triangle on other end.

Don't worry if it's not quite perfect. A pixel off here or there will make it look more natural, since a real LCD is going to have a few imperfections too. Experiment with shapes. Give it your own style.

### When you finish drawing:

##### Edit > XML Editor.

See those seven paths under the `svg:g id="layer1"` dropdown? Bingo! Each one of these paths corresponds to the outline of one of our seven grains.

Once you have drawn the seven grains, use the fill tool to fill in the background.

Now check XML editor again. See how it's put in new paths for the fill? Since all that we care about is the fill, and we only used the outline to build containers to fill out, we can "Delete Node" on the top seven items in the list. As you delete each one, the outline will disappear from the picture.

Now you're ready to save the SVG and grab the code.

##### File > Save A Copy

Choose Plain SVG (\*.svg) for the format. This will get rid of extra Inkscape metadata.

### open that file!

Go ahead and open up the .svg file in an editor of your choice. Interesting, huh? At the bottom, you'll see a `<g>` tag that contains seven `<path>` tags. `<g>` is just a way of telling SVG that you are grouping related elements: in this case, seven `<path>` grains into a digit.

You want to grab the path information. It won't look quite the same as my example above though. Here's what one of my paths looked like before I cleaned it up:

```xml
<path
  transform="translate(0,1058.5196)"
  id="path5715"
  d="M 3.6841521,2.5178497 1.2616471,0.09369836 H 16.01122 30.760792 L 28.338287,2.5178497 25.915782,4.942001 H 16.01122 6.1066572 Z"
  style="fill:#00ff00;fill-opacity:1;stroke:#000000;stroke-width:0.01031554;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1"
/>
```

Lots of decimal places and a strange transform applied. The transform is something that happens when you're drawing in Inkscape. The transform property, path data from `d` and `viewBox` all interact to make your element display properly. However, we don't want to have this excess. Can't we just convert the path based on the `transform` and get rid of the `transform` attribute altogether?

Yes. We can use [SVGOMG](https://jakearchibald.github.io/svgomg/) to clean this up.

Go to the site and load your SVG file. In the toolbar on the right, Un-check "Merge Paths". We need separate paths in order to manipulate them individually. You can play with the precision slider. This gets rid of some of the extra decimal places. Have a look at the preview image while you do this. At a certain point, the quality of the image starts to suffer when there's not enough information in the path to provide an accurate geometric description. This is a tradeoff between file-size and rendering precision.

Grab the cleaned-up paths and you should have something like I posted above with your own SVG paths. You're now ready to wire up this display!

### hit me with those digits ~ programatically generating svg

We dynamically generate digits in the display using JS. Each path will have two attributes:

1. `d` to provide the browser data on how to draw the path
2. `style` to tell the browser how to style and fill the path it draws

`style` is invariant for each grain. We'll manipulate the fill-opacity later. All that differs is `d`.

### i need some space for my na me

If you've used VanillaJS to create an HTML element, this is similar. However, we're putting svg inside of HTML. That's fine to do, but svg and HTML do not have the same [namespace](https://developer.mozilla.org/en-US/docs/Web/SVG/Namespaces_Crash_Course).

What the heck is a namespace? Imagine you have two people at the office named Bob. The boss calls out, "Hey Bob, can you make me a copy of your report?" There are two Bobs. There are two reports. Which Bob will think it's him? Which report(s) will the boss get? There is no way to tell!

A code example: the HTML `<title>` tag [here](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/title) vs the SVG `<title>` tag, [here](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/title). Wait, did you mean "Title of the document for the title bar", or "Title of the SVG group"? Oh, SVG namespace? Okay, title of the SVG group. Now we're sorted.

*Namespaces capture the notion that the same name can mean a different thing in a different context.* Here, the contexts are HTML and SVG. We use a namespace to define the context in which we are using a name, resolving ambiguity around its meaning. We get one report, and it's the one we want.

Whenever we create an svg tag, we'll use `document.createElementNS()`, specifying the `svg` namespace before the tag name. Like this:

`var svg = document.createElementNS("http://www.w3.org/2000/svg", "svg");`

### createDigit

This is the core of the display code. It generates a 7-grained digit with a unique ID per grain. We stored each of the `d` elements from the paths we created in an array, in order in which we are going to address them (top, top-left, top-right, middle, bottom-left, bottom-right, bottom).

We create an HTML container `div` to hold the digit.
```javascript
var digit = document.createElement('div');
digit.id='digit';
```
We put the `svg` element into the container. We set the viewbox, width, and height. We set the [xlink](https://developer.mozilla.org/en-US/docs/Web/SVG/Namespaces_Crash_Course).
```javascript
var svg = document.createElementNS("http://www.w3.org/2000/svg", "svg");
svg.setAttribute('viewBox', '0 0 32 64');
svg.setAttribute('width', this.width);
svg.setAttribute('height', this.height);
svg.setAttributeNS("http://www.w3.org/2000/xmlns/", "xmlns:xlink", "http://www.w3.org/1999/xlink");
digit.appendChild(svg);
```
We append a `g` element to the `svg` element to hold our seven grains.
```javascript
var g = document.createElementNS("http://www.w3.org/2000/svg", "g");
g.setAttributeNS(null,"id",layerID);
svg.appendChild(g);    
```
We append each of the seven `path` elements to the `g` element. We stored those path `d` elements from the Inkscape file in an array.
```javascript
const path_d = [
    'M 2.102706,2.0144078 0.21966194,0.08855428 8.1139619,0.07386242 c 4.3418651,-0.0080805 11.4445871,-0.0080805 15.7838261,0 L 31.787315,0.08855428 29.918288,2.0150532 28.049261,3.9415521 16.017505,3.9409067 3.98575,3.9402615 Z',
    // ... //
    'last path'];  
];
for (var i=0;i<=6;i++) {
    var path = document.createElementNS("http://www.w3.org/2000/svg", "path");
    path.setAttributeNS(null,"id",'c_'+containerID+'_d_'+id+'_'+i);
    path.setAttributeNS(null,"d",path_d[i]);
    path.setAttributeNS(null,"style", "fill:"+this.GRAIN_COLOR+";fill-opacity:0.3;stroke:none");
    g.appendChild(path);
}
```
Finally, we return the `div` container from the function.

We can call this function as many times as we want to generate the digits for our counter. We'll get a `div` for each digit which we can then mount in a container box holding all the digits: our display.

### why is the first argument to `setAttributeNS` null?!

We do not need to provide a namespace prefix to `id`, `d`, or `style` attributes, since there is no ambiguity around their usage within our `<svg>` tag. Also, we do not want to namespace these attributes, because it will make selecting them with `document.querySelector` incredibly tricky.

### what about the weird IDs?

In order to display numbers on the LCD, we have to programmatically manipulate individual grains of digits. This means we need a unique ID for each grain and also for each digit.

In the demo you saw at the top of the page, we had to support the case of multiple counters per page. We must ensure uniqueness across these IDs as well.

This generates three levels of ID:

1. container (holding an arbitrary number of displays, which are instances of our LCD object)
2. digit (within container)
3. grain (within digit)

I represent these IDs as numeric and zero-indexed in this format: `c_{containerID}_d_{digitID}_i_{grainID}`. If there is a 3-digit display in the first container, we can access the 7 grains of the first (leftmost) digit by `c_0_d_0_i_0` through `c_0_d_0_i_6`.

### let's wire up the display!

Things are going to get fairly messy if we try to code things without organization, so let's create an Object called `LCD`. Inside the object we'll define variables and methods to display output.

We *construct* this object by giving it a number of digits, a color, a height, and a background color.

We will make everything but the number of digits optional.

We define the width in terms of the height, so we only require a height for the display.

`instanceID` is a unique numeric ID that is incremented once each time an LCD is created. We will also refer to it as `containerID`: it is what we use to uniquely access a container of digits, or a display.

`digitsOn` is an object that maps human-readable digits or letters we want to display to the indices of grains that must be turned on to display this digit (or letter). Try working your way through one of the numbers, envisioning the array's indices turned on. Remember, we index in this order: top, top-left, top-right, middle, bottom-left, bottom-right, bottom.

We set `fill-opacity` values for on (HIGH) and off (LOW).

We define a variable to hold the current value of the counter.

```javascript
var LCD = (function(){
  var id=0;
  function LCD(config={}) {
        const { numDigits, color, height, bgColor } = config;
        if (!Number.isInteger(numDigits)||numDigits<1) { throw new Error('Cannot have non-integral number of digits'); }
        this.backgroundColor = bgColor;
        this.instanceID = ++id;
        this.numDigits = numDigits || 3;
        this.GRAIN_COLOR = color || '#00FF00';
        this.height = height || '64'; //aspect ratio for the SVG is 1:2
        this.width=height/2;
        this.padding = 8;
        this.digitsOn = {   
            '0': [0, 1, 2, 4, 5, 6],
            '1': [2, 5],
            '2': [0, 2, 3, 4, 6],
            '3': [0, 2, 3, 5, 6],
            '4': [1, 2, 3, 5],
            '5': [0, 1, 3, 5, 6],
            '6': [0, 1, 3, 4, 5, 6],
            '7': [0, 2, 5],
            '8': [0, 1, 2, 3, 4, 5, 6],
            '9': [0, 1, 2, 3, 5],
            'A': [0, 1, 2, 3, 4, 5],
            'B': [1, 3, 4, 5, 6],
            'C': [0, 1, 4, 6],
            'D': [2, 3, 4, 5, 6],
            'E': [0, 1, 3, 4, 6],
            'F': [0, 1, 3, 4],
            'H': [1, 2, 3, 4, 5],
            'L': [1, 4, 6]
        };
        this.HIGH_OPACITY='1.0';
        this.LOW_OPACITY='0.15';
        this.value=null;
    }
    return LCD;
  )();
```

### a namespace for the display

We wrapped this object in an [IIFE (immediately-invoked function expression)](https://developer.mozilla.org/en-US/docs/Glossary/IIFE). This provides a private namespace for the variables within our class, allowing us to expose only certain methods. We don't want the user to be able to accidentally modify the number of digits in the display, for example.

We can *expose* methods of our choice to the user. A common exposure technique is `get`ters and `set`ters that do just what they sound like: get or set values of variables within the object.

I do not use the word class here. Why? [JavaScript does not have classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes), in the traditional object-oriented sense. Although there is a new `class` keyword in ES6, this is just syntactic sugar for [prototypal inheritance](https://en.wikipedia.org/wiki/Prototype-based_programming).

Objects inherit through other objects via the prototype chain. More to come on this topic, as it really deserves its own series.

### a touch of CSS

I have a [CSS reset](http://meyerweb.com/eric/tools/css/reset/), a little padding, and then two [flexboxes](https://css-tricks.com/snippets/css/a-guide-to-flexbox/): one to display the digits in each display, and one to display each display (how meta). I also added a CSS transition on `fill-opacity` for prettiness. The browser automatically [tweens](http://easings.net/) the values for us. Less JS to manage!

```css
body {
  padding: 10px;
  background-color: #FFE;
}

#pageContainer {
  display: flex;
  width: 700px;
  margin: auto;
  padding: 10px;
  flex-flow: row wrap;
  border: 1px solid black;
}

.lcd-container {
  margin: 10px;
  border: 1px solid black;
  display: flex;
  flex-flow: row nowrap;
  border-style: groove;
  justify-content: center;
  align-items: center;
}

div#digit {
  padding: 4px;
}

svg > path {
  transition: fill-opacity 250ms ease-in;
}
```

### A good place for you to begin.

Here's a CodePen that's unfinished. Fork it, and try building out the rest of the class.

Here are some method names that might help get you going:

* init
* unmount
* getSize
* getValue
* setDigit
* setCounter
* resetDigit
* resetDisplay
* displayError

You can always peek at the finished product for hints. But imagine that you can use the object to display '999' on the screen like this:

```javascript
var display = new LCD({
    numDigits:3,
    color:'#000000',
    height:40,
    bgColor:'#FFFFFF'
});
let C=document.createElement('div');
display.init(C);
display.setCounter(999);
document.body.appendChild(C);
```

Try to fork the CodePen below and implement it yourself.

<p data-height="300" data-theme-id="32039" data-slug-hash="OzpOdb" data-default-tab="js" data-user="thmsdnnr" data-embed-version="2" data-pen-title="SVG LCD Display: Fork and Finish!" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/OzpOdb/">SVG LCD Display: Fork and Finish!</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

### wait though, you coded more

You are correct. I included [tinycolor](http://bgrins.github.io/TinyColor/) to dynamically generate random yet contrasty background and text colors for my random timers. I also set random values for the timers to start at and a random number of digits, and then I set up a `setInterval` to have them all count upwards in unison. Because it's cool!
