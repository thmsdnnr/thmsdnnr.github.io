---
layout: post
title:  "Attractio 3: Plug in the WebSockets!"
date:   2017-12-06 06:40:04 -0500
categories: tutorials javascript attractio
---

### [>> pt. 3 repo](https://github.com/thmsdnnr/attractio-tutorial/tree/master/pt3)
*Note on the repo: this is a point where the code starts to get very specific to my own project. In the interest of generality, I've left the client and server events sparse on both server and client, leaving only the WebSocket initiation + maintenance code and a few simple actions demoing echo and broadcast.*

*You could easily put in your own events and adapt the project from here if you aren't interested in the poetry aspect. For instance, write a simple UI for a chat room.*

*You can hook into events to save off room data (just include any events you'd like __ignored__ and not saved off in `dbActionsToIgnore`)*

### Step 0: Let's Code the WebSockets!

As we did with the database, we're also going to break out the WebSockets code into a module. We move the database handling code from the server into the WebSockets module itself, since we are going to load and save room data in response to WebSocket messages from clients.

#### server.js
The server code for connecting to the database now looks like this.

```javascript
const Ws=require('./ws.js');
// .. //
Db.connect((err,db)=>{
  if (err) {
    console.error('Failed to make all database connections!');
    console.error(err);
    process.exit(1);
  }
  Ws.initServer(server, db);
});
```

### Step 1: Book Larnin'

What are WebSockets? For the verbose-minded, you can read [RFC 6455](https://tools.ietf.org/html/rfc6455). Also, two good MDN articles on [servers](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_servers) and [clients](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications).

Here's the basic anatomy of a WebSocket session:

1. Server listens for connection requests sent under the `ws://` protocol.
2. Client requests to connect. Client and server perform a [handshake](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_servers#The_WebSocket_Handshake).
3. If the handshake is successful, the connection is initiated
4. While the connection is open, messages can be sent back and forth between server & client
5. When the client sends a close message, we terminate the connection

We format these messages with [Javascript Object Notation (JSON)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON) using [JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) before sending (to convert the object into a string) and [JSON.parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse) upon receipt (to convert the string back into a Javascript object with accessible properties).

### Step 2: Initiate, Maintain, and Close Connections

Here's what the code looks like in our WebSockets module to initiate and close a connection:

```javascript
const WebSocket=require('ws');
const UTILS=require('./utils.js');

exports.initServer = (server, dbCon) => {
  let clientID=0;
  let clients={}; //map of clientIDs to their WS Client objects
  let rooms={}; //map of roomIDs to an array of clientIDs
  let wss=new WebSocket.Server({server: server});

  wss.on('connection', function connection(client) {
    client.isAlive = true;
    client.on('pong', heartbeat.bind(client));
    clientID++;
    let CUID=clientID;
    clients[clientID]={obj: client, color:UTILS.randomRGB(), name:UTILS.randomString(CUID), id:clientID};
    if (client.readyState===client.OPEN) {
      client.sendMsg = function(message) {
        let payload=JSON.stringify(message);
        client.send(payload);
      };
    }
    client.on('close', function close() { terminateClient(clients[CUID]); });
  });
```

We dependency-inject the `server` object from our Express server and use it to initiate the WebSocket server. The two can share the same port. [`ws`](https://www.npmjs.com/package/ws) is a WebSocket module for Node.js. The `wss` object holds the initialized WebSocket server.

#### When A Client Joins:

We assign them a unique numeric client ID.

We maintain a dictionary of ClientIDs, and we map these numeric IDs to the client objects, as well as a random color and name. We also keep a dictionary of rooms, which maps room IDs to an array of ClientIDs that are currently in a room. We also set the client's `_CURRENT_ROOM` property to the name of the room they send the `newClient` message from.

*NB We can enforce a maximum room size by sending back a `roomFull` message if the client array for a room exceeds a desired length. In response, the client can suggest other room URLs to visit instead.*

#### Maintaining Client Connections:

The client and server play ping-pong. We use the `heartbeat` function that we bound to the `pong` event on the server when the client joined us.

The server will sends a `ping` message to all connected clients at a predetermined interval. `ping` gives clients this amount of time to respond, else their connection will be terminated.

Per the WebSocket specs, the WebSocket object in the client will respond automatically with a pong.

```javascript
function heartbeat() { this.isAlive = true; }
const interval = setInterval(function ping() {
  Object.keys(clients).forEach(function each(clientID) {
    let client = clients[clientID];
    let cObj=client.obj;
    if (cObj.isAlive === false) { terminateClient(client); }
    cObj.isAlive=false;
    cObj.ping('', false, true);
    });
  }, UTILS.$.wsPingInterval);

function terminateClient(client) {
  let cObj=client.obj;
  let CUID=client.id;
  cObj.terminate();
  let R=cObj._CURRENT_ROOM;
  if (R && rooms[R]) { rooms[R]=UTILS.remove(rooms[R],CUID); }
  delete clients[CUID];
}
```

#### When A Client Leaves:

When a client closes the connection (or they timeout), we call `terminateClient` with the client object to close the WebSocket connection, delete them from a room array, and delete their ID from the Clients dictionary.

### Step 3: Communication Over The Connection

Now for the fun stuff. When we have an active connection between client and server, the two can exchange messages in response to user events.

We categorize messages by `actions` and associated `data`. Messages also have a `destination` (room name) and transmission type (`broadcast` or `sendPrivateMessage`).

#### 1. broadcast

```javascript
wss.broadcast = function broadcast(data, room, sendingClient) {
  rooms[room].forEach(function(clientID) {
    if (clientID!==sendingClient) {
      let client=clients[clientID].obj;
      if (client&&client.readyState===client.OPEN) { client.sendMsg(data); }
    }
  });
};
```

This function takes the data, room, and the sendingClient. It retrieves all the clients in the room list other than `sendingClient` (since we do not want to send the message to the original sender). It then sends the data out to all the clients in the list that have a `readyState` equal to OPEN.

Clients with another type of `readyState` will either get updated when the server pings the client list, or they will be disconnected entirely due to inactivity.

#### 2. sendPrivateMessage

```javascript
const sendPrivateMessage = (msg, recClient) => {
  let C=clients[recClient].obj;
  if (C) { C.sendMsg(msg); }
  else { return false; }
};
```

This function takes a message and a receiving client and sends the message to only this client.

### Step 4: Maintaining Client List Ordered By Last Active

Before we parse and respond to client messages, we can define things that we want to have happen generically to every message (or a large subset of messages). One of those things is to "promote" the client that sent the message to the front of the room array, if that client is sending us data.

```javascript
const promote = (arr, ele) => {
  if (!arr||!ele) { return false; }
  if (arr.length===1) { return arr; }
  let idx = arr.indexOf(ele);
  return (idx!==-1) ? [ele].concat(arr.slice(0,idx),arr.slice(idx+1)) : arr;
}

client.on('message', function incoming(message) {
  if (!message) { return false; }
  message=JSON.parse(message);
  if (client._CURRENT_ROOM&&message.data) {
    let R=client._CURRENT_ROOM;
    rooms[R]=UTILS.promote(rooms[R], CUID);
  }
```

Why bother doing this? It maintains an ordered list of the most active clients in a given room. If we want to ask a client to send us a given room's state, we'll request that state from the most active client in the room array.

We use this most active client as a source of truth for room state, and we use this room state in our database saving strategy.

Consider a room with two clients, A and B. They start with identical state. Client B moves a word and sends a move message.

Client A will receive this move message from the server.

But wait. Now Client C comes along. Which client should we ask for the most up-to-date information? Client B, since Client B's board reflects the most recent modification to board state. If we ask Client A, it is possible that the message will be received prior to the `move` message from Client B. The board state will be skewed across clients and impossible to recover.

By requesting the state from active clients in a room, rather than the database every single time, we can reduce the number of hits to the database and increase the speed of the application. However, clients leave the page, and we want to periodically save off the state of the room a client is in so that when they leave, we'll have a record of all the words and their positions in the database.

### Step 5: Periodically Save Board State to Database

In addition to promoting clients, we will save off the board state to the database when we receive events of specified types. We exclude some events because they will not be tied to valid board data.

```javascript
const dbActionsToIgnore=['emptyRoom','newClient','freeze','moving'];
if (!dbActionsToIgnore.includes(message.action)) {
  Db.saveRoom({room:message.room, data:message.data}, function(err, data) {
    if (err) { console.error(err); }
  });
}
```

New clients will not have active board data. A freeze is not a good time to save off data to the Db, because we know it will soon be followed by an unfreeze with updated data. Similarly, a `moving` event is not a good time to save data, because it's dynamic: we know the data is changing.

Basically the strategy for saving off information to the database is:

1. Grab the most recent board state from the most recently-updated client that has data
2. Save off the state to the database regularly, but not __wastefully-many__ times

How to define __wastefully-many__? It'd be any call to `saveRoom` that contained duplicate data, for sure. It'd also be any call to `saveRoom` that contained data *nearly* duplicate (say, we move a word a few pixels). We'll sacrifice perhaps losing a little bit of position data if something drastic happens with the app for the performance we gain from not updating *every single time*.

### Step 6: Let's See It In Action

Rather than post a massive block of code with all the client and server events for the finished app, I've created this repo to show the basics of how to connect the client and server and exchange messages.

Here's what our `onMessage` code looks like on the server:

```javascript
client.on('message', function incoming(message) {
  if (!message) { return false; }
  message=JSON.parse(message);
  if (client._CURRENT_ROOM&&message.data) {
    let R=client._CURRENT_ROOM;
    rooms[R]=UTILS.promote(rooms[R], CUID);
  }
  if (!dbActionsToIgnore.includes(message.action)) {
    //Database hook: can save state to db here
  }
  switch(message.action) {
    case 'newClient':
      let R=message.room;
      client._CURRENT_ROOM=R;
      rooms[R] = rooms[R] ? rooms[R].concat(CUID) : [CUID];
      wss.broadcast({action:'talkToRoom', data:'Welcome, client '+CUID+'!'},message.room,CUID);
      break;
    case 'echo': sendPrivateMessage(message, CUID); break;
    case 'talkToRoom': wss.broadcast(message,message.room,CUID); break;
    default: wss.broadcast(message,message.room,CUID); break;
  }
});
```

`{action:'echo'}` just sends back the message directly to the client that sent it using `sendPrivateMessage`. `{action:'talkToRoom'}` shows how broadcast works, by sending a welcome message when clients join up to the room.

Here's what we're doing on the client-side:

```javascript
let G={};

window.onload = function() {
  sockets();
  setInterval(function echoPlex() {
    let currentTime=new Date(Date.now())
    G.sendMsg({action:'echo', data: currentTime});
  },4000);
};

function beforeUnload() {
  G.sendMsg({action:'talkToRoom', data:'Goodbye!'});
  G.socket.close();
}

window.addEventListener('beforeunload', beforeUnload);
```

`beforeUnload` will be called whenever the window is about to be closed by the browser. `sockets` sets up our webSocket connection. We'll tell the server to echo back to us the timestamp every 4s.

```javascript
function sockets() {
  let messageList=document.querySelector('ul#messages');
  var host = window.location.origin.replace(/^http/, 'ws');
  G.socket = new WebSocket(host);
  G.room=window.location.href.split("/");
  G.room=G.room[G.room.length-1];
  G.sendMsg = function(msg) { //socket message wrapper
    if (G.socket.readyState===G.socket.OPEN) {
      let alwaysSendData={room:G.room, pieceB:'I will always get sent!'};
      let payload=JSON.stringify(Object.assign(msg, alwaysSendData));
      G.socket.send(payload);
    }
    else { //need to recover connection
      if (G.reconnectAttempts<5) { G.socket = new WebSocket(host); }
      G.reconnectAttempts++;
    }
  };
  G.socket.addEventListener('open', (event) => {
    G.sendMsg({action:'newClient'});
  });
```

The code at the top here grabs the URL of the client and replaces its protocol with `ws` so that the server will receive the WebSocket connection at whatever address the code is hosted at (whether localhost or in the cloud). `room` parses out the last bit of the URL after the final forward-slash and sends that to the server so it knows which room to assign the client to.

Finally, we add an event listener to handle messages on the client side. In this case, whenever we receive a message of type `echo` or `talkToRoom`, we simply prepend it to a list and display the text on-screen.

```javascript
  G.socket.addEventListener('message', (event) => {
    event=JSON.parse(event.data);
    let newItem=document.createElement('li');
    switch(event.action) {
      case 'echo':
        newItem.innerHTML='Echo timestamp: '+event.data;
        messageList.prepend(newItem);
      break;
      case 'talkToRoom':
        newItem.innerHTML='Message broadcast: '+event.data;
        messageList.prepend(newItem);
      break;
      default: console.log('Message received', event); break;
    }
  });
}
```

Note the similarity between the message handling on the server and client.

### Step 7: This is a LOT OF CODE

Don't be overwhelmed! Just read some of the docs to get a little background, clone the repo, and set up everything locally so you can get a sense of how the client and server talk to each other. Try creating a button on the page that creates an event when you click it & send that event to other clients.

The messaging pattern is pretty simple once you get down to it. Just remember the basic constraints we set up:

* Clients live in rooms and have unique IDs
* A client can only be in one room at a time
* Messages can be sent to individual clients or all clients in a room
* Messages are simple JavaScript objects containing `actions` and `data`
* We pass the message into our send functions using `JSON.stringify()` and re-objectify it at the source with `JSON.parse()`

Play around and build something cool! :)
