---
layout: post
title:  "Attractio Finale: Deploy to the Cloud(s)!"
date:   2017-12-03 07:40:04 -0500
categories: tutorials javascript interactivePoetry
---


### Step 1: Link Your Github Repo to Heroku


### Step 2: Set Up Continuous Integration


### Step 3: Set up mLab


### Step 4: Add Environmental Variables to Heroku

### Step 5: Deploy!

We're going to build a small, separate module for our database code. Our server will include this module and then be able to access its exposed methods (`saveRoom` and `loadRoom`) to store and retrieve room data.

Our server can store a single database connection and re-use it over time.

[MongoClient](https://mongodb.github.io/node-mongodb-native/api-generated/mongoclient.html) is just the NodeJS MongoDB driver, a sort of wrapper for MongoDB that makes it easy to use with Node.

N.B. If you're going to get much more complex than we are with database use, you'd probably want to choose something like [Mongoose](http://mongoosejs.com/) to simplify your database modeling code. We're not going to get into that today though.

//TODO connection pooling and reconnect handling!

```javascript
var MongoClient = require('mongodb').MongoClient;
let db;
let dbUrl=process.env.PROD_DB||'mongodb://localhost:27017/';

function connect(callback) {
  if (db===undefined) {
    MongoClient.connect(dbUrl, function(err, database){
      if(err) { return callback(err) };
      db=database;
      callback(null, db);
    });
  }
  else { callback(null, db); }
}

connect(function(err, db){
  if (!err) { console.log('Connected to the database'); }
  else { //pass error to handler
       }
});
```

We'll write our functions so that we call them with two parameters, `data` and `callback`. Data contains the data we're sending (in the case of `saveRoom`, an object with the room name and the room data; in the case of `loadRoom`, simply the room name to load). Callback is what our code calls after the database loads the data and is ready to send it back to our server.

There are really two callbacks happening here: we pass a callback to the Database itself with the Database request, and the server passes a callback to the Database code with the request for data.

//error handling
//asynchronous functions

In: Server -> calls database code w/callbackOne -> database code calls database with callbackTwo
Out: Database calls callbackTwo with data, database code has data, calls callbackOne, server has data, continues

```javascript
exports.saveRoom = function(D, cb) {
  if (!D.data) { cb('THERE IS NO DATA BEING SENT', null); }
  db.collection("rooms").update({room:D.room}, {room:D.room, data:D.data, createdOn: new Date()}, {upsert:true},
  function(err, data) { cb(err, data); });
}

exports.loadRoom = function(name, cb) {
  db.collection("rooms").find({room:name}).toArray(function(err,data){
    cb(err, data);
  });
}
```
