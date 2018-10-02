---
layout: post
title:  "Great Flock of Boids: Generative Systems"
date:   2018-10-02 06:56:04 -0500
description: Coding 
author: Thomas Danner
lang: en_US
categories: compsci
tags: JS, boids, computer science, generative systems
---

## Boids

// TODO: CodePen / GitHub example link, wikipedia article, figure out Rule 2, consider use Points instead of overloading the V2 to act as both vector and point.

[Boids]() are a simulation program, originally conceived by Craig Reynolds, to model the cooperative movement of animals in nature, like flocks of birds and schools of fish. One of the super cool things about boids is the complex flocking behavior and coordinated movement that emerges from the application of a few simple rules to a group, along with some constants that dictate how these rules are applied. This emergent behavior is similar to [John Conway's Game of Life](TODO LINK HERE).

Conrad Parker [has written an excellent resource](http://www.vergenet.net/~conrad/boids/pseudocode.html) on how to code the simulation using psuedocode, which will allow you to reproduce the simulation in your own language of choice.

For my case, I chose to implement boids using JavaScript and HTML5 canvas to visualize the flock.

### Vectors and Scalars

A brief introduction to terms. A vector is a mathematical tool that describes a direction as well as a magnitude. A scalar simply represents a magnitude.

The common example is velocity and speed. If you tell me you're going 55 miles per hour on the highway, I know how much distance you're covering, but I have no idea where you're going toward or coming from.

On the other hand, if you tell me that you're traveling 55 miles per hour (direction) *and* that you're going north on I-5 (magnitude), I can locate you on a map. 

But "north on I-5" is too specific. To capture the notion of direction and magnitude in the abstract, vectors have a coordinate for each direction of motion.

For a 2-dimensional vector, there is an X-component and a Y-component. For a 3-D vector, there is also a Z-component.

JavaScript has numbers to handle scalars, no problem. There's no built in Vector object though, so let's make one now to handle our 2D vectors.

```javascript
function V2 (x, y) {
  // 2D vector class
  this.x = x || 0
  this.y = y || 0

  this.add = addedVector =>
    new V2(this.x + addedVector.x, this.y + addedVector.y)
  this.subtract = subtractedVector =>
    new V2(this.x - subtractedVector.x, this.y - subtractedVector.y)
  this.multiply = magnitude => new V2(this.x * magnitude, this.y * magnitude)
  this.magnitude = () => Math.sqrt(Math.pow(this.x, 2) + Math.pow(this.y, 2))
}
```

We intiialize the default x and y copmonent to 0 and add four methods.

##### Addition
To add two vectors, you simply create a new vector, where each component is the sum of the added vectors' components.

For instance: `[1, 2] + [3, 4]` => `[4, 6]`

##### Subtraction
Subtraction is the inverse of addition. Just subtract instead.

##### Multiplication
To multiply a vector by a scalar, create a new vector with the 

##### Magnitude
To calculate magnitude, we use the Pythagorean Theorum: the magnitude of a vector is equal to the square root of the sum of the squares of each component.

### Boids
Now that we have vectors, we can make boids.

Boids have positions and velocities. We also want to be able to identify individual boids, since some rules will modify a given boid's velocity by calculating some factor of all boids *except* the given boid. To do this, we can create an autoincrementing integer on the Boid object that increases by one each time we create a new boid.

#### Positions

A position is an X, Y coordinate pair. A velocity is a 2D vector that operates on those pairs. A velocity is a change in position over time. 

```javascript
function Boid (initialPosition, initialVelocity) {
  this.position = initialPosition || new V2(0, 0)
  this.velocity = initialVelocity || new V2(0, 0)
  Boid.numInstances = (Boid.numInstances || 0) + 1
  this.id = Boid.numInstances // autoincrementing unique ID
}
```

### Rules for Boid Behavior

Each of these rules considers an individual `boid` with respect to the group of all boids `boidList`, an array of `boid`s.

Each rule takes the boid in consideration and the entire list of boids, and then it returns a vector.

In each tick of time, we evaluate each rule on every boid, which gives us three new vectors for each boid. We then add the three vectors to the boid's position to generate the next state of the boid flock.

#### RULE 1: // Boids try to align themselves with the average position of the flock

First, we calculate the average position of the flock (excluding the current boid). We calculate this just like any other average: sum each vector and then divide by the length of the list of boids (minus one, because we're excluding the current boid).

We subtract the current boid's position from the average position. This generates a vector pointing from the current boid to the average position of the flock.

Finally, we multiply this alignment vector by a `COHESION_FACTOR`: this can be used to temper the effect of the forced alignment. 

```javascript
function rule1 (boid, boidList) {
  // Boids try to align themselves with the average position of the flock
  let theOtherBoids = boidList.filter(boidId => boid.id != boidId.id)
  let avgPosition = new V2()
  theOtherBoids.forEach(boid => (avgPosition = avgPosition.add(boid.position)))
  return avgPosition
    .multiply(1 / theOtherBoids.length - 1)
    .subtract(boid.position)
    .multiply(COHESION_FACTOR)
}
```

#### RULE 2: // Boids try to maintain a minimum distance between themselves and others

We can set a constant `TOO_CLOSE_MAGNITUDE` that the boids will use to figure out if they are too close to one another. First, we make a list of the boids that the current boid is too close too. For each boid that's too close, we generate a vector pulling this boid away from these boids.

```javascript
function rule2 (boid, boidList) {
  // Boids try to maintain a minimum distance between themselves and others
  let newPositionVector = new V2()
  boidList
    .filter(boidId => boid.id !== boidId.id)
    .map(otherBoid => otherBoid.position)
    .filter(
      otherBoidPosition =>
        otherBoidPosition.subtract(boid.position).magnitude() <
        TOO_CLOSE_MAGNITUDE
    )
    .map(tooCloseBoid =>
      newPositionVector.subtract(tooCloseBoid.subtract(boid.position))
    ) // TODO: is this adding to newPositionVector actually?

  return newPositionVector
}
```


#### RULE 3: // Boids try to match their velocity with that of the flock

We can calculate the average velocity in the same way that we calculated average position. Then, multiply by a `VELOCITY_MATCH_FACTOR`.


```javascript
function rule3 (boid, boidList) {
  // Boids try to match their velocity with that of the flock
  return boidList
    .filter(boidId => boid.id !== boidId.id)
    .map(otherBoid => otherBoid.velocity)
    .reduce((boidA, boidB) => boidA.add(boidB), new V2())
    .multiply(1 / boidList.length - 1)
    .multiply(VELOCITY_MATCH_FACTOR)
}
```

## Simulation

### Set up a Canvas

`<canvas id="birdflock"></canvas>`

If you have a retina display, canvas pixels look weird unless you cale them down by a factor of 2. So, we'll use a `scaleFactor` of the pixel ratio (2x for retina, 1x for non-retina) to augment our calculations.

We'll make the canvas the height & width of the window with 100 pixel margins. 

The `ctx.translate(0.5, 0.5)` moves the screen by a half so that the edge of pixels aren't jagged. 

```javascript
const scaleFactor = window.devicePixelRatio
const MAX_X = window.innerWidth * scaleFactor - 100
const MAX_Y = window.innerHeight * scaleFactor - 100

c = document.getElementById('birdflock')
c.width = MAX_X
c.height = MAX_Y
c.style.width = `${c.width / scaleFactor}px`
c.style.height = `${c.height / scaleFactor}px`

ctx = c.getContext('2d')
ctx.translate(0.5, 0.5)
ctx.scale(scaleFactor, scaleFactor)
```

### Teach the Boids to Draw Themselves

We can make each boid draw itself by just generating a rectangle on the canvas based on the boid's x and y coordinates (taking into account the `scaleFactor` of devicePixelRatio):

```javascript
this.draw = ctx => {
  ctx.fillStyle = 'gold'
  ctx.fillRect(
    this.position.x / scaleFactor,
    this.position.y / scaleFactor,
    BOID_WIDTH,
    BOID_HEIGHT
  )
}
```

### Introduce the element of time

Now that we have Boids and rules for their behavior, we have to introduce an element of time into our simulation. We can do this using `window.requestAnimationFrame`. In each tick of time, we'll first check to see if we should draw a frame, based on our `fpsInterval`, which is 1000ms / frames-per-second. If we haven't reached this amount of time since we last drew, we'll do nothing.

The main loop looks like this:

```javascript
// How to know if we should draw in the next frame we're given
let then = Date.now()

function step () {
  window.requestAnimationFrame(step)
  now = Date.now()
  elapsed = now - then
  if (elapsed < fpsInterval) {
    return
  }
  then = now - elapsed % fpsInterval
  ctx.clearRect(0, 0, c.width, c.height)
  ctx.fillStyle = 'black'
  ctx.fillRect(0, 0, c.width, c.height)
  updateBoidPositions(boidList)
  drawBoidList(boidList, ctx)
}
```

With each tick we:

1. clear the canvas and fill it with black
2. call `updateBoidPositions`, which applies the rules to each boid, and then performs a final adjustment to make sure the boid isn't off screen.
3. tell the boids to draw themselves

It helps to impose a velocity limit on the boids, so that the flocks do not hurt your eyes as they flutter about. `VELOCITY_LIMIT` is a constant, and `limitVelocity` can be called on the resultant velocity:

```javascript
const limitVelocity = v =>
  (v.magnitude > VELOCITY_LIMIT ? v / v.magnitude * VELOCITY_LIMIT : v)
```

In addition, in order to keep the boids within the viewport (the part of the screen that we display on the canvas), we have to limit their coordinates to the bounds of the viewport.

So, in each frame, for each boid, we'll make sure the X and Y coordinates are greater than zero, and we'll make sure that the X and Y coordinates are less than some maximum value (in this case, the width and height of the viewport).

So that the boid doesn't go *all* the way off the screen, we'll make the bounds slightly smaller, by a factor of the BOID_WIDTH and BOID_HEIGHT for the X and Y components, respectively.

If te boid is out of bounds on either coordinate, we apply an opposing factor to the velocity `FLINGBACK_VELOCITY` and multiply by -1 if we need the boid to go in the opposite direction from which it's going (for instance, if its X or Y coordinate is too large, it needs to make this coordinate smaller).

```javascript
this.normalizePosition = (maxX, maxY) => {
  // If it's too far left or too far right
  if (this.position.x < BOID_WIDTH) {
    this.velocity.x += FLINGBACK_VELOCITY
  } else if (this.position.x > MAX_X - BOID_WIDTH) {
    this.velocity.x += -1 * FLINGBACK_VELOCITY
  }
  // Or too far up or too far down
  if (this.position.y < 5 - BOID_HEIGHT) {
    this.velocity.y += FLINGBACK_VELOCITY
  } else if (this.position.y > MAX_Y - BOID_HEIGHT) {
    this.velocity.y += -1 * FLINGBACK_VELOCITY
  }
}
```

In each tick, to update the positions we just apply the 3 rules to each boid. This gives us the new vector for that boid. We add it, and then we make sure the boid's not now going too fast or flying off of the screen:

```javascript
function updateBoidPositions (boidList) {
  boidList.forEach(boid => {
    boid.velocity = limitVelocity(
      boid.velocity.add(
        rule1(boid, boidList)
          .add(rule2(boid, boidList))
          .add(rule3(boid, boidList))
      )
    )
    // Add the calculated ∆velocity to get the new position
    boid.position = boid.position.add(boid.velocity)
    // Make sure we aren't making the boids go off into the ether
    boid.normalizePosition(MAX_X, MAX_Y)
  })
}
```

Conveniently, boids know how to draw themselves, so to draw them all:

```javascript
const drawBoidList = (boidList, ctx) => boidList.forEach(boid => boid.draw(ctx))
```

### Starting the simulation

We need to generate some random boids and then feed them to the loop we wrote.

We'll make a function to generate boids up to `BOID_START_CT` with a random position (though roughly at the center of the screen) and a random velocity. These parameters are set up so that the boids are initially flying in opposite directions.

```javascript
function fillBoidList () {
  // Make some random boids
  let list = []
  for (var i = 0; i < BOID_START_CT; i++) {
    // Give a little randomization
    // TODO: would be cool to cast them off equally in all directions
    // as though away from the center of a circle, e.g., boidCt / 360
    // each one every 3.6 degrees difference
    let dir = Math.random() > 0.5 ? 1 : -1
    list.push(
      new Boid(
        new V2(MAX_X / 2 + Math.random() * 25, MAX_Y / 2 + Math.random() * 25),
        new V2(2 * Math.random() * dir, 2 * Math.random() * dir)
      )
    )
  }
  return list
}
```

#### Then, to kick the whole thing off:

```javascript
let animFrameHandler;

function resetBoidsAndStartAnimationFrames () {
  if (animFrameHandler) {
    window.cancelAnimationFrame(animFrameHandler)
  }
  boidList = fillBoidList()
  animFrameHandler = window.requestAnimationFrame(step)
}

function fly () {
  setUpWindow()
  resetBoidsAndStartAnimationFrames()
}

// Kick off the simulation
document.addEventListener('DOMContentLoaded', fly)
```

## More stuff

In my CodePen, I added some sliders so that you can vary the the rule's parameter factors dynamically, adjust the frames-per-second, and a reset button to start again from scratch.

You can have a lot o fun with this. [Conrad Parker's paper](http://www.vergenet.net/~conrad/boids/pseudocode.html) details some of these ideas, like introducing a wind as a constant vector for all the boids in the flock. You can also set up obstacles for boids (an obstacle is just a boid that ... never moves). You could easily have the canvas react to mouse clicks or toggle the simulation parameters.