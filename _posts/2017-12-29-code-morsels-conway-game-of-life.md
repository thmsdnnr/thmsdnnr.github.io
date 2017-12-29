---
layout: post
title:  "It's All In The Game: Cellular Automata"
date:   2017-12-29 8:40:04 -0500
description: John Conway's Game of Life simulation implemented in JS
author: Thomas Danner
lang: en_US
categories: tutorials javascript projects
tags: cellularAutomata, gameOfLife, algorithms, javascript, simulations, tutorial
---

## John Conway's Game of Life

<p data-height="341" data-theme-id="32039" data-slug-hash="baWVQB" data-default-tab="js,result" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Conway's Game of Life Tutorial" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/baWVQB/">Conway's Game of Life Tutorial</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

##### [The Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) is a [cellular automaton](https://en.wikipedia.org/wiki/Cellular_automaton) simulation. It provides a good example of emergent properties, or higher orders of complexity that can arise from applying a simple set of rules.

A cellular automaton consists of a finite grid of cells with a finite number of possible states. There are typically rules governing the evolution of these states over time. In the case of the Game of Life, there are only two states: alive and dead. The simulation proceeds forward one `generation` at a time by evaluating a rules for each cell and updating the simulation.

In each generation, we evaluate these rules for each in the grid to determine the next generation:

### if the cell is alive:
1. If it has fewer than two living neighbors, it dies (simulating underpopulation)
2. If it has two or three live neighbors, it continues to live
3. If it has more than three live neighbors, it dies (simulating overpopulation).

### if the cell is dead:
1. If the cell has three live neighbors, it comes back to life (simulating reproduction).

Each cell will be represented by an element in an array. Living cells are 1s and dead cells are 0s. In each generation, we compute a score of living neighbors for each cell by summing the values of its neighboring cells.

### data structure

We need an array of length `n^2` to hold our cell states.
We need a function to initialize the array with a random state, 1 or 0.

```javascript
const sideLength=10;
let states=new Array(Math.pow(sideLength,2)-1)
  .fill(0).map(e=>Math.random()>0.5?1:0);
```

### data methods

We need a function to evaluate the "living" or "dying" rules on each cell and return a new array. We need to take care not to modify the array in-place. We take a snapshot of the array and create a *new* array based on the snapshot. Mutating the snapshot would change the rules' outcome. We return a new array from the function and then set this equal to the next state of the board.

### the trickiest part

The hardest part is accessing the indices of neighbors to evaluate whether a given cell lives or dies.

First, we'll write a helper function to take the current state of a cell and its number of live neighbors. It will return the next state of the cell, alive (1) or dead (0).

```javascript
function cellDestiny(currentState, neighborCt) {
  if (currentState===1) {
    if (neighborCt<2||neighborCt>3) { return 0; }
    else { return 1; }
  }
  if (neighborCt===3) { //if dead but three live neighbors
    return 1;
  }
  return 0;
}
```

Now we'll iterate through our state array cell-by-cell. For each cell we will:

1. Identify its neighbors
2. Sum the score of its living neighbors in `liveNeighborCt` (including diagonals)
3. Call `cellDestiny` with `liveNeighborCt` and the current cell's state

We will need to bounds-check the validity of the indices so that we do not attempt to access values outside of the array (e.g., non-existent neighbors for elements on the perimeter).

Here is the strategy:

1. Take the index of the cell, `i`.
2. `i-sideLength` is the cell above.
  * Check to see if we are in the top row. If not, add the cell above. If so, set `topRow` flag.
3. `i+sideLength` is the cell below.
  * Check to see if we are in the bottom row. If not, add the cell below. If so, set `bottomRow` flag.
4. `i-1` is the cell to the left.
  * Check to see if we are in the left-most column. If not, add the cell to the left, and the cells left-above and left-below (if in bounds).
5. `i+1` is the cell to the right.
  * Check to see if we are in the right-most column. If not, add the cell to the right, and the cells right-above and right-below (if in bounds).

We can encapsulate all of this logic in a for-loop. The only tricky math involved here is figuring out if we are in the left-most or right-most column.

Whenever we are in the left-most column, the index of the cell will divide evenly into `sideLength`. Consider this diagram here:

<img src="https://i.imgur.com/1As7tcX.jpg" width="250" height="auto" alt="image of three-row array, five elements per row">

`sideLength` is 5. `0%5===0`. Same for `5%5` and `10%5`.

Similarly, whenever we are in the right-most column, the `index+1` will divide evenly into `sideLength`, since when we add one to the right-most column, we wrap around to the left-most column of the next row. `4+1=5`, and `5%5==0`. `9+1=10`, and `10%5==0`.

```javascript
function processState(stateArr, sideLength) {
  let nextState=[];
  for (var i=0;i<stateArr.length;i++) {
    let topRow=false;
    let bottomRow=false;
    let liveNeighborCt=0;
    if (i-sideLength>=0) { //add neighbor above
      liveNeighborCt+=stateArr[i-sideLength]; // #1
    } else { topRow=true; }
    if (i+sideLength<=stateArr.length) { //add neighbor below
      liveNeighborCt+=stateArr[i+sideLength]; // #2
    } else { bottomRow=true; }
    if ((i-sideLength)%sideLength!==0) { //if we are not at the left edge
      liveNeighborCt+=stateArr[i-1]; // #3
      if (!topRow) { liveNeighborCt+=stateArr[i-sideLength-1]; } // #5
      if (!bottomRow) { liveNeighborCt+=stateArr[i+sideLength-1]; } //#6
    }
    if ((i+sideLength)%sideLength!==0) { //if we are not at the right edge
      liveNeighborCt+=stateArr[i+1]; // #4
      if (!topRow) { liveNeighborCt+=stateArr[i-sideLength+1]; } // #7
      if (!bottomRow) { liveNeighborCt+=stateArr[i+sideLength+1]; } // #8
    }
    nextState.push(cellDestiny(stateArr[i],liveNeighborCt));
  }
  return nextState;
}
```

I commented the code with the number of each neighbor that will be considered for a given square. In this diagram, you can see these neighbors:

<img src="https://i.imgur.com/ALsmm09.jpg" alt="grid of 9 squares numbering the neighbors of the middle square" width="200">

### d3 visualization

We've written all the code. We can call `processState` with our random array and get back an updated array. But it's not so interesting. We're going to create a grid of squares to visualize the patterns in the data. Each square will have a color associated with it. Live cells will have high-opacity fills and dead cells will have low-opacity fills. We'll then throw `processState` in a `setInterval` so that we can view many generations of the board and watch it evolve over time.

We can use d3 to draw the squares. We'll create an `init()` function to draw the SVG element. Then we'll bind our data array to the element and append an SVG `rect` for each element of data.

We set the `x` and `y` coordinates of the squares so that they show up as a grid. The X coordinate will be a repeating pattern for each row (since this is the X-coordinate of a column), and it is based on the side length of each square and which column we are in (given by `idx%squaresPerRow`).

The `y` coordinate is increasing for each index, but constant for each row. Thus, it is the whole part of `idx/squaresPerRow` multiplied by the side length of our squares.

```javascript
const squaresPerRow=20;
const squareSideLen=10;
const boardDim=squaresPerRow*squareSideLen;

function init() {
  let states=new Array(Math.pow(squaresPerRow,2)).fill(0).map(e=>Math.random()>0.5?1:0);
  var grid = d3.select('#grid').append('svg').attr('width', boardDim).attr('height', boardDim);  
  var cell = grid.selectAll('.cell').data(states).enter().append('rect')
    .attr('class','cell')
    .attr('height', squareSideLen+'px')
    .attr('width', squareSideLen+'px')
    .attr('margin', '1px')
    .attr('stroke', 'black')
    .attr('stroke-width', '1px')
    .attr('x',function(d,idx) {return (idx%squaresPerRow)*squareSideLen;})
    .attr('y',function(d,idx) {return Math.floor(idx/squaresPerRow)*squareSideLen;})
    .attr('fill', 'salmon')
    .attr('fill-opacity', function(d) { return d===1 ? '1.0' : '0.5'});
  }
```

Finally, we define a function to update our d3 data display. We call it with the data array that is returned from `processState`.

```javascript
function updateData(states) {
  var grid = d3.select('div#grid');
  var cells=grid.selectAll('.cell').data(states);
  cells.attr('fill-opacity', function(d) { return d===1 ? '1.0' : '0.3'});
}
```

It toggles the fill-opacity of cells based on whether or not they are alive ('1.0') or dead ('0.3').

### animation

```javascript
const RATE=100; //update every 100ms
let interval=null;

function tick(initialState) {
  var lastState=null;
  var nextState;
  interval=setInterval(()=>{
    if (!lastState) {
      nextState=processState(initialState,squaresPerRow)
    } else {
      nextState=processState(lastState,squaresPerRow)
    }
    updateData(nextState);
    lastState=nextState;
  },RATE);
}
```

### start the simulation

All that's left to do is call the functions we created when the window loads up.

```javascript
init();
let initial=new Array(Math.pow(squaresPerRow,2)).fill(0).map(e=>Math.random()>0.5?1:0);
tick(initial);
```

I added a reset button and some simple styling that you can see from the CodePen. It's neat how easy it is to set up a complex simulation and visualization using D3.

### next steps?

The Game of Life has many [emergent patterns](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life#Examples_of_patterns). Ideas for new features include:

* allow user to "paint" an initial state by clicking on cells
* pre-program known patterns and let the user click tabs to toggle between them
* recognize and count the number of pattern occurrences for a given # of generations
* graph pattern frequency of occurrence in real-time with D3
* display a different color for cells that have just died or just come back to life
* display a different color for cells as they age (tracking stability of patterns & clusters)

You could even change the rules and create your own automata. Experiment and have fun with it!
