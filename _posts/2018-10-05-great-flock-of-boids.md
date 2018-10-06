---
layout: post
title:  "A Great Scarf of Boids"
date:   2018-10-05 06:56:04 -0500
description: Craig Reynolds' boids simulation using JavaScript and canvas, along with basic explanation of vector math to run the particle simulation.
author: Thomas Danner
lang: en_US
categories: fun projects
tags: JS, boids, computer science, generative systems, particle system
---

<img src="/assets/pics/boidflock.gif" alt="A flock of Boids flying freely" width="700px">
[CodePen Demo](https://codepen.io/thmsdnnr/full/JmRboR/) // [github repo](https://github.com/thmsdnnr/boids/)

## Boids

[Boids](https://en.wikipedia.org/wiki/Boids) are a simulation program, originally conceived by Craig Reynolds, to model the cooperative movement of animals in nature, like flocks of birds and schools of fish.

Boids are fascinating, because from the application of a few simple rules to a group of entities, complex coordinated behavior emerges.

Conrad Parker [has written an excellent resource](http://www.vergenet.net/~conrad/boids/pseudocode.html) on how to code the simulation using psuedocode, which will allow you to reproduce the simulation in your own language of choice.

Here, we'll use vanilla JavaScript and an HTML canvas to visualize the flock.

### Vectors and Scalars

A brief introduction to terms.

* Scalars: represent a magnitude
* Vectors: represent direction as well as magnitude

The classic examples are speed and velocity. 

Imagine you're driving a car, and you give me your current position on a map and tell me you're going 55 miles per hour.

I know how much distance you're covering in a given time, but I have no idea where you're going toward or coming from.

On the other hand, if you tell me that you're traveling 55 miles per hour (direction) *and* that you're going north on I-5 (magnitude), I can tell you where you'll be in a given period of time, given your starting position.

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

We intialize the default x and y copmonent to 0 and add four methods.

#### Addition
To add two vectors, you simply create a new vector, where each component is the sum of the added vectors' components.

For instance: `[1, 2] + [3, 4]` => `[4, 6]`

#### Subtraction
Subtraction is the inverse of addition. Just subtract instead.

For instance: `[1, 2] - [3, 4]` => `[-2, -2]`

#### Multiplication
To multiply a vector by a scalar, create a new vector with each component multiplied by the scalar.

For instance: `[1, 2] * -1` => `[-1, -2]`

#### Magnitude
To calculate magnitude, we use the Pythagorean Theorum: the magnitude of a vector is equal to the square root of the sum of the squares of each component.

For instance: `[-1, 2] => sqrt(-1 * -1 + 2 * 2) => sqrt(5)`

### Boids
Now that we have vectors, we can make boids.

Boids have positions and velocities. We also want to be able to identify individual boids. To do this, we can create an autoincrementing instance count on the Boid object.

Velocity is the change of position over time (distance / time). Acceleration is the change of velocity over time (distance / time * 2). You can describe this using calculus (integrating the velocity over a time interval gives the distance travelled), but for our simulation, all this means is that if we update at a constant frames-per-second, we can update the position by adding the change in velocity over that time interval.

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

Each rule returns a velocity vector.

#### 1. Boids try to align themselves with the average position of the flock

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

#### 2. Boids try to maintain a minimum distance between themselves and others

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
    )
  return newPositionVector
}
```


#### 3. Boids try to match their velocity with that of the flock

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

We'll make the canvas the height & width of the window with 100 pixel margins. 

`ctx.translate(0.5, 0.5)` moves the canvas grid by a half-pixel for smoothing.

If you have a retina display, canvas pixels look weird unless you scale them down by a factor of 2. So, we'll use a `scaleFactor` of the pixel ratio (2x for retina, 1x for non-retina) to augment our calculations.

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

Now that we have Boids and rules for their behavior, we have to introduce an element of time into our simulation. We can do this using `window.requestAnimationFrame`.

In each tick of time, we'll first check to see if we should draw a frame, based on our `fpsInterval`, which is 1000ms / frames-per-second. If we haven't reached this amount of time since we last drew, we'll do nothing.

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

1. clear the canvas and paint it black
2. call `updateBoidPositions`, which applies the rules to each boid, and then performs a final adjustment to make sure the boid isn't off screen
3. tell the boids to draw themselves

### Refining boid behavior

It helps to impose a velocity limit on the boids, so that the flocks do not hurt your eyes as they flutter about. `VELOCITY_LIMIT` is a constant, and `limitVelocity` can be called on the resultant velocity:

```javascript
const limitVelocity = v =>
  (v.magnitude() > VELOCITY_LIMIT ? v.multiply(1 / (v.magnitude() * VELOCITY_LIMIT)) : v)
```

In addition, in order to keep the boids within the viewport (the part of the screen that we display on the canvas), we have to limit their coordinates to the bounds of the viewport.

So, in each frame, for each boid, we'll make sure the X and Y coordinates are greater than zero, and we'll make sure that the X and Y coordinates are less than some maximum value (in this case, the width and height of the viewport).

So that the boid doesn't go *all* the way off the screen, we'll make the bounds slightly smaller, by a factor of the BOID_WIDTH and BOID_HEIGHT for the X and Y components, respectively.

If the boid is out of bounds on either axis, we apply an opposing factor to the velocity `FLINGBACK_VELOCITY` and multiply by -1 if the value is too large.

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

## More ideas

In my CodePen, I added some sliders so that you can vary the the rule's parameter factors dynamically, adjust the frames-per-second, and a reset button to start again from scratch.

[Conrad Parker's paper](http://www.vergenet.net/~conrad/boids/pseudocode.html) details some of these ideas. You could try:

1. introducing a wind as a constant vector for all the boids in the flock
2. setting up obstacles for boids to fly around
3. have the canvas react to mouse clicks to generate or direct flocks
4. add another dimension and use [https://threejs.org/](https://threejs.org/) to make a VR simulation