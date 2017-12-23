---
layout: post
title:  "Intro to D3 Graphs + visualize sorting algorithms"
date:   2017-12-22 6:40:04 -0500
description: Visualizing Sorting Algorithms in D3
author: Thomas Danner
lang: en_US
categories: tutorials javascript codeMorsels algorithms
tags: D3, sorting, algorithms, javascript, dataViz, tutorial
---

### let's make some pictures

Insertion and Binary Insertion Sort, Selection Sort, Bubble Sort, and Quicksort.

Quicksort dominates. Selection and Insertion sort aren't bad. And, Bubble Sort (as is expected) takes a very long time.

Try the `camelBack` beginning arrays. You'll see that these take Bubble Sort the longest to complete and particularly struggles with "turtles" (small values near the end of the initial array). Shell sort is an implementation we'll talk about that utilizes elements of insertion and bubble sort, aiming to reduce the effect of these "turtles" on performance.

### d3 Library

[d3](https://d3js.org/) is a fantastic SVG-based library for data visualization. You can make all sorts of graphs and charts using it. It allows you to bind data to the DOM and then interact with it directly, either by creating HTML elements for display or SVG elements for real-time visualization.

For some inspiration and great links to a "code playground" of d3 examples, check out [bl.ocks.org](https://bl.ocks.org).

Some say that d3 has a large learning curve, but here we'll just cover the basics.

Goals:

1. Create a d3 bar graph
2. Enable the graph to dynamically update given new data
3. Learn the three different phases of d3 data updating, `enter`, `update`, and `append`

### let's make a simple graph

We'd like to create a bar graph from a simple array of numbers.

All graphs in d3 require some initial configuration. We must specify a height, width, and padding, which is typically written as a `margin` object with keys `top`, `left`, `right`, and `bottom`.

```javascript
var width=300;
var height=200;
const padding=10;
var margin = {top: padding, right: padding, bottom: padding, left: padding};
width-=(margin.left+margin.right);
height-=(margin.top+margin.bottom);

var svg = d3.select('div#C').append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform",
          "translate(" + margin.left + "," + margin.top + ")");
```

Setting the height, width, and margin is simple enough, but what's going on with `d3.select`? D3 select works just like `document.querySelector`, only it returns an empty selection instead of `null` if there are no matches. `d3.selectAll` works like `document.querySelectorAll`.

To select all the paragraphs on a page in d3, use `d3.selectAll('p');` To select a DOM node stored in a variable `v`, use `d3.select(v)`.

We append an `svg` element to our container div and adjust its position based on our margins.

Now we just need to add some data to the picture. We'll define a function called `update` that takes the `data` array. Each time `update` is called, it will bind the data it is passed to DOM elements on the page and update those elements accordingly.

### update()

First we define a function for our `x` and `y` axis using d3's scaling functions. These functions scale the domain (input values) of our x and y coordinates to a specified range (output values) based on the dimensions of our graph.

We also define `barHeight`, so that all the bars will be proportionately sized to fill the graph container.

When we call `x(coord)` or `y(coord)`, `coord` is scaled appropriately to fit the graph on the screen.

```javascript
var x = d3.scaleLinear()
  .domain([0, d3.max(data, function(d) { return d; })])
  .range([0, width]);
var y = d3.scaleBand()
  .domain(data.map(function(d,i) { return i; }))
  .range([height, 0]);
var barHeight = Math.floor((height-padding)/data.length);
```

Why is this necessary? For instance, in our case, the graph is `300px` wide. What if we want to display a value of `100,000`? We can't do that. In addition, we need to display data proportionately: if there are `300px` of width to display a value, a value A 10x bigger than B needs to have 10x the number of pixels of width. This would all get very messy, very fast.

#### bind the data

```javascript
var bars = svg.selectAll(".bar").data(data);
```

#### create DOM elements (bars)
```javascript
bars.enter().append("rect")
  .attr("class", "bar")
  .attr("y", function(d,i) { return i*barHeight+margin.top; })
  .attr("width", function(d) { console.log(d, x(d)); return x(d); })
  .attr("x", function(d,i) { return margin.left; })
  .attr("height", barHeight-1);
```

`enter()` creates elements for each element of data that does not have a matching DOM element. In this case, that means each bar in our bar graph. It then passes this newborn element down the chain of methods. For each data element that does not have a matching DOM element, we append an SVG `rect` element, and we assign it attributes based on the data.

We add a class to the `rect` element to style the bars. In our CSS, we can style the bar how we like using the `rect.bar` selector.

There are four attributes for each data element we need to consider: x, y, height, and width.

X and Y are the actual pixel positions of the bar on the page, starting at the upper-left corner.

Height and width specify the size of the bar.

The `.attr` method allows us to set the value of each of these attributes using a callback. The callback contains two values, `d`, and `i`. `d` is the actual value of the data at this point. In our case, this is just the array element, but in more complex data, it is whatever data keyed at this index, `i`, which is also passed.

* The `y` position of each element will just be the element index, multiplied by the `barHeight`, plus `margin.top`.
* The `width` of the element will be the scaled value of `d`, `x(d)`
* The `x` position of the element will be the same for each. This is the "baseline" of the graph, so all bars start from the same place.
* The `height` of the element will be the bar height, with 1 pixel subtracted so there is padding between the bars.

#### remove DOM elements no longer needed

When we update the data, we might have too many elements on the page. So we need to `.remove()` these elements.

```javascript
bars.exit().remove();
```

That's it. If we wanted to do something special with those elements `.exit()`ing the page, we could do that before we call `.remove()` on them which removes them from the DOM.

#### merge in new data

We will modify our `.enter().append()` chain slightly to include the `merge()` method.

```JavaScript
.merge(bars)
.transition()
.duration(500)
```

### codepen
https://codepen.io/thmsdnnr/full/VyKrpj/

<!-- <p data-height="590" data-width="800" data-theme-id="32039" data-slug-hash="VyKrpj" data-default-tab="result" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Data Visualization of Sorting Algos w/D3" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/VyKrpj/">Data Visualization of Sorting Algos w/D3</a> by Thomas (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script> -->
