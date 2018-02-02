---
layout: post
title:  "Attractio 3.5: Defining Our Messages!"
date:   2017-12-04 16:40:04 -0500
categories: tutorials javascript attractio
---

### [>> pt. 3 repo](https://github.com/thmsdnnr/attractio-tutorial/tree/master/pt3)
*Note on the repo: this is a point where the code starts to get very specific to my own project. In the interest of generality, I've left the client and server events sparse on both server and client, leaving only the WebSocket initiation + maintenance code.*

*You could easily put in your own events and adapt the project from here if you aren't interested in the poetry aspect. For instance, write a simple UI for a chat room.*

*You can hook into events to save off room data (just include any events you'd like __ignored__ and not saved off in `dbActionsToIgnore`)*

### Step 6: Define Client & Server Actions and Their Responses

In addition to a message's action + associated data, there are a few bits that we send with *any* message for addressing purposes.

The client will always send its own ID with the message as well as its current room so that the server knows what room the data pertains to. We could find the client ID in our room dictionary, but tagging the room name onto the messages saves us this look-up, and it also gracefully handles the case where the client is requesting a new room entirely that doesn't exist in our records.

Each message sent by the server will contain an action and data. Since the server keeps track of clients and which room they are in, it can target the message to clients with its room dictionary, so the room name is not necessary.

Message data can be broken up into two types: board data and "atoms" of data.

* Board data means sending the current state of the board: all the words and their positions.

* "Atoms" of data means sending client updates to individual elements of the board. For instance, modifying the text of a word, deleting a word, or moving its position.

Sometimes the server does not do anything to a message, other than broadcast it to all other members of the room. In the action lists below, I refer to this behavior as "relay".

#### Client sends to server:

*board data*

* stateUpdate `{action:'stateUpdate', room:roomState.room, data:roomState.data}`

*atoms*
* emptyRoom `{action:'emptyRoom'}`
* newClient `{action:'newClient'}`
* moving `{elementID:event.elementID, dx:event.dx, dy:event.dy}`
* moveEnd `{action:'moveEnd', elementID:event.target.id, finalX:thisPos.x, finalY:thisPos.y}`
* freeze `{action:'freeze', elementID:event.target.id}`
* unfreeze `{action:'unfreeze', elementID:event.target.id}`
* delete `{action:'delete', elementID:e.target.id}`
* modify `{action:'modify', elementID:e.target.id, update:{innerHTML:I.value}}`

#### Server sends to client

*board data*
* newClient `{action:'hydrate',data:data[0].data}`
* stateUpdate `{action:'rehydrate',data:message.data}`
* hydrate `{action:'hydrate',data:data[0].data}`
* rehydrate `{action:'rehydrate',data:message.data}`

*atoms*
* newRoom `{action:'newRoom'}`
* freeze `Object.assign(message,{clientName:clients[CUID].name, color:clients[CUID].color})`
* requestCurrentState `{action:'requestCurrentState'}`
* roomFull `{action:'roomFull', data:urls}`
* unfreeze -- relay
* moving -- relay
* moveEnd -- relay
* modify -- relay
* newWord -- relay
* delete -- relay

#### Client Responses to Server Messages
* roomFull -- display a room full message with three suggested empty rooms to try instead
* newRoom -- open modal to input words for new room
* requestCurrentState -- check to see if current room has state & then send:
  *  has state? `{action:'stateUpdate', room:roomState.room, data:roomState.data}`
  *  no state? `{action:'emptyRoom'}`
* rehydrate -- load the words & positions sent by server
* hydrate -- un-hide the magnetic board, add event listeners, & load words & positions sent by server
* newWord -- add div with new word & position to board
* modify -- modify word text of existing div
* delete -- delete existing div
* moving -- move existing div by dx and dy
* moveEnd -- increment z-index of just-moved element, decrement z-index of covered elements
* freeze -- add word to frozen word list, outline word with client color & name
* unfreeze -- remove word from frozen word list, remove outline with client color & name

### Step 7:

All we have to do now to harness these actions is build out Client code to wire up the UI to events. Each of the "Client Responses to Server Messages" will be tied to a function on our client side. We also will enhance our server code a little bit to respond to these events.
