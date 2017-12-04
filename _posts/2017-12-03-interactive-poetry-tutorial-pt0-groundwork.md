---
layout: post
title:  "Attractio 0: Project Background"
date:   2017-12-03 09:00:04 -0500
categories: tutorials javascript attractio
---

### *[the end result](https://attractio.herokuapp.com/what-fun)*
![image of attractio, magnetic poetry board](https://i.imgur.com/ohXoqQH.png)

### Prerequisites:

* You have a machine with a basic development environment going
  * You have [Node](https://nodejs.org/en/), [NPM](https://www.npmjs.com/get-npm), [Git](https://git-scm.com/downloads), & [MongoDB](https://www.mongodb.com/download-center#community) installed locally
* You know your way around Github enough to have an account, set up a basic remote repository, link it to your local repository, and push commits to it.

You can find instructions for your system by clicking each of the links above, as well as by Googling YOUR OS + install + DEV TOOL LISTED ABOVE.

### Step 0: What We'll Do

Do you remember those little magnetic rectangles with words on them you could stick on a refrigerator and rearrange to create funny poems? Let's build this on the web. Some design goals:

- Easy: simple features. things should just work. no assembly required.
- Interactive: multiple users can manipulate words on the same refrigerator at one time.
- Dynamic: users can provide word lists, or choose word lists seeded by poetry.
- Social: users can share their boards easily via Twitter

#### Learning goals for this tutorial:

1. Build a Client and Server that talk to one another with WebSockets
2. Use a minimal number of libraries to illustrate how far you can go with VanillaJS
- but also see the points where incorporating libraries saves you time and much hair-pulling
3. Write tests for your code so things don't break without you knowing why
4. Deploy to production

#### You'll learn the basics of how to set up and use:

1. [mLab](https://mlab.com/) -- a Database-as-a-Service running MongoDB
2. [Heroku](http://heroku.com) -- a Platform-as-a-Service running apps in the cloud
3. [Jest](https://facebook.github.io/jest/) -- popular JS testing framework

### What to do right now

* Sign up for a free account for mLab and Heroku.
* Sign up for GitHub if you haven't already.
* Install the software in the prerequisites.

### Step 1: Let's Get Started!

If you've completed the previous steps, you can either go ahead to the next tutorial, or you can read some of the "Big Picture" design considerations listed below to warm up your mind on some of the concepts we'll be implementing.

### Big-picture design considerations

#### Concurrent access to data

What makes this project challenging? It's the same sort of challenge when writing collaborative software anywhere: multiple users can view and modify data at the same time. We have to make sure two users don't try to simultaneously modify the same piece of data.

We also have to make sure that the users' views are refreshed and up-to-date when changes occur, since up-to-date views change user actions: you're not going to try to modify a word anymore that someone has just deleted. Maybe you'll create a new one instead.

The considerations for building this project are essentially the same as one would have when writing database code or filesystem access code. We can have multiple concurrent users accessing and modifying each word on the screen. Each user can add a new word, delete an existing word, and modify an existing word, either by changing its text or screen position.

In software testing terms, we're trying to prevent [race conditions](https://en.wikipedia.org/wiki/Race_condition). Havoc will ensue if two users are allowed to modify the same word at the same time. What's the source of truth? What displays on one user's screen if another user is modifying that word? We need to ensure that one and only one user can modify/update/delete a word at a time.

To enforce some sanity here, we need to use the concept of a "Freeze". Whenever a user begins to move or modifies a word, we place a freeze on that word until the user is done moving or modifying. Before we allow a user to move or modify a word, we check to see if the word is frozen. We'll also need to provide some feedback to the user on which words are currently frozen, so that the app doesn't appear broken or unresponsive when they try to modify a frozen word.

#### Client and server communication

We want to support multiple users on the same board, but we also want to support multiple boards. We need to have a server that can store boards and board states (words on that board and their position).

On first page load, the server must be able to deliver the client the up-to-date board. When clients make modifications to the board, the server must receive these modifications from the client, transmit the modifications to other connected clients, and save the current board state in the database.

The client and server will be communicating via [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API). WebSockets allow for bi-directional communication between a client and server.

We'll need to define minimal set of messages that clients can send to the server and the server can send out to clients.

These messages will be JavaScript objects and framed as "actions". One example of a client message and server response might be:
* `{action:'newClient', room:'kitten-party'}`
* `{action:'hydrate', data:[{word:'cats', xPos:'36', yPos:'55'} ... etc]}`

Every few messages, the server will also save off the current state of the board to the database so that if the clients in a room leave, the board will still exist when people return to the link.

#### Keeping track of rooms and clients

On the server, we'll need to keep track of all the current rooms that exist as well as a list of all clients connected to those rooms.

We need these lists so that we can broadcast changes specific to a room. If one user moves the word "kitten" in "kitten-party", we don't want to also move "kitten" in "cats-and-dogs-living-happily". The client list is also important because it enables us to broadcast messages to all clients *except* the one that originally transmitted the message.

#### Identifying words uniquely

We need a concept of a "Primary Key" for each word on each board. We are representing each word "magnet" by a div, and the key by the div's `id`. This key allows us to uniquely identify a word across all clients on a given board. This key is tied to the position, which we need to update whenever a word is moved.

The key is crucial to keeping boards in sync. Imagine if Client A has div-0 tied to "cats" and Client B has div-0 tied to "fire". B's moving "fire" moves A's "cats". We've lost the source of truth.

We'll store a word's numeric primary key in the div ID prefaced with a hyphen (since numeric values are not valid CSS IDs): e.g., `<div id="div-1">`.

#### Drag and drop

HTML5 has [its own drag-and-drop](https://www.w3schools.com/html/html5_draganddrop.asp) functionality. For our purposes, I've chosen a lightweight library called [Interact.js](http://interactjs.io/) to help manage word dragging. It gives us `onMoveEnd` and `onMoveStart` events that we can hook into in order to send our freezeStart and freezeEnd messages. It also calculates some inertia for us and bounding boxes so that we don't have to.

#### Overlapping divs

Since words are represented by divs that hold them, we need to make a way for divs to overlap that feels "magnetic". That is, if you place one div on top of another, that div should cover the div below it. We also want to implement it so that you can drag divs out from under other divs. We'll get to this later, but we'll use a simple collision detection method to "lift up" a div that was just moved by incrementing its z-index, and to "sink" the divs that it is covering, by reducing their z-index by one.

That way, we can have things that look like this:

<img src="https://i.imgur.com/80kGuHz.png" width="256" height="168" alt="image of attractio, words overlapping gracefully">

#### Simple modals

I've chosen [Tingle](https://robinparisi.github.io/tingle/), a simple modal library written in pure JS to handle our page-load modals. No need to reinvent the wheel.

#### Random words to seed new poem boards

[PoetryDb](http://poetrydb.org/index.html) gives us lightweight endpoints to grab lines of poetry from a large list of poets. We'll use this to populate a dropdown of poets that users can choose from when creating a new board. This will grab some words from those poems and throw them in the board!

#### Persisting board state across client sessions

Finally, we need to store the state of the board so that there is persistence of state across user sessions. We'll need to have the server maintain a simple database that contains a list of rooms and the current words and positions in those rooms.

### Are you ready?

This is going to be awesome, imperfect, and fun. Just like life.
