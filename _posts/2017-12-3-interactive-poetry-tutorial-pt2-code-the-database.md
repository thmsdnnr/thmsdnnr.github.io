---
layout: post
title:  "Attractio 2: Wire Up a Database!"
date:   2017-12-04 14:30:04 -0500
categories: tutorials javascript attractio
---

### [>> pt. 2 repo](https://github.com/thmsdnnr/attractio-tutorial/tree/master/pt2)

#### Step 0: Let's Code the Database!

We're going to build a small, separate module for our database code. Our server will include this module and then be able to access its exposed methods `saveRoom` and `loadRoom` to store and retrieve room data.

There are a few new techniques being used here:

* Node modules: breaks up code into files based on functionality
  * [Here's a good explanation](https://darrenderidder.github.io/talks/ModulePatterns/#/6)
* Dependency injection: passes our active database connection to other modules that use them
  * [Dependency Injection design pattern](https://blog.risingstack.com/dependency-injection-in-node-js/)
* Connection pooling: enables our server to store a single connection and re-use it over time
  * [Info from mLab](https://blog.mlab.com/2017/05/mongodb-connection-pooling-for-express-applications/)

### Database Connection Code

#### db.js
```javascript
var MongoClient = require('mongodb').MongoClient;
let db;
let dbUrl=process.env.PROD_DB||'mongodb://localhost:27017/';

exports.connect = function(callback) {
  if (db===undefined) {
    MongoClient.connect(dbUrl, function(err, database){
      if(err) { return callback(err, null) };
      db=database;
      callback(null, db);
    });
  } else { callback(null, db); }
}
```

[MongoClient](https://mongodb.github.io/node-mongodb-native/api-generated/mongoclient.html)
is the NodeJS MongoDB driver, a sort of wrapper for MongoDB that makes it easy to use with Node.

The URL at which we access the database will either be given by the process that we're running in (set dynamically on Heroku) or locally at the default mongodb port, 27017 (note the `mongodb://` instead of `http`). For those of you being like, wait, no, the process shouldn't set the URL...we're going to store our Db access credentials in an environmental variable on Heroku & pass them with the URL when we access the Db so that we don't expose credentials in the repo.

The code checks to see if we already have a connection open. If we do, it passes back the connection. If we don't, it attempts to create a new connection and either passes back this newly formed connection or an error, if one is thrown.

N.B. If you're going to get much more complex than we are with database use, you'd probably want to choose something like [Mongoose](http://mongoosejs.com/) to simplify your database modeling code as well as connection handling. For our purposes though, manually coding this stuff is just fine, and good experience to see how the nuts and bolts work at a lower level.

### Basic database queries

#### db.rooms.js
```javascript
module.exports = function(dbCon) {
  if (!dbCon) { throw new Error('Module must be injected with an active database connection.'); }
  return {
    saveRoom: function (D, cb) {
      dbCon.collection("rooms").update(
        {room:D.room},
          {$inc: {visits: 1},
           $set: {room:D.room, data:D.data, lastUpdate: new Date()}
          },
        {upsert:true}, function(err, data) { console.log(data.result); return cb(err, data); });
      },
    loadRoom: function (name, cb) {
      dbCon.collection("rooms").find({room:name}).toArray(function(err,data){
        return cb(err, data);
      });
    }
  };
};
```

Since database access is [__asynchronous__](https://www.pluralsight.com/guides/front-end-javascript/introduction-to-asynchronous-javascript) (it takes longer for the result to be returned than the code that requests the result to execute), we pass the database queries a callback function. This function will be called upon query completion with the resulting data and errors, if any, that arise in the query process.

#### saveRoom()
Saves room data using [update()](https://docs.mongodb.com/manual/reference/method/db.collection.update/). We pass update an object containing a query parameter. In this case, that's the room name, `D.room`. We then tell the database what to do when it finds this room: increment the visit counter by 1, and set the room data, as well as the lastUpdated time.

In database-speak, our ["Primary Key"](https://en.wikipedia.org/wiki/Unique_key) for the "rooms" collection is the room name. A __primary key__ is an element that is related to output data in a one-to-one fashion. We won't query `{room:'cats-and-dogs'}` and receive two rooms back. We'll only ever receive nothing (the room does not exist yet) or one room (the room exists: here's its data) back.

What about that `{upsert:true}`? __Upsert__ is Mongo-speak for "update or insert". If the room we query for (`D.room`) already exists in the database, we simply update its data. If not, we insert a new entry. Each time we save the room, we increment (`$inc:`) the visit count and we set the data passed.

#### loadRoom()
Retrieves room data using [find()](https://docs.mongodb.com/manual/reference/method/db.collection.find/).

We add database connection code upon server initialization:

#### server.js
```javascript
const Db=require('./db.js');
const rooms=require('./db.rooms.js');
let dbQueries=null;

Db.connect((err, db)=>{
  dbQueries=rooms(db).catch((err)=>{
    console.error('Failed to connect to database!', err);
    process.exit(1);
  });
});
```

See the dependency injection? We pass `db` into the `rooms` module that we have included. `rooms` then uses that internally as `dbCon` to run queries against our active database connection.

Why bother? This enables us to "pool" our connections: we're not telling asking Mongo to spin up a new connection to the database each and every time that we run a query. This would be super wasteful not only due to extra network traffic, but also because it interferes with Mongo's internal performance optimizations that allow re-use of the same connection.

We also add the calls to `loadRoom` and `saveRoom` in our server's route requests:

```javascript
app.post('/:p', (req, res) => {
  if (req.params.p.match(/[^a-z0-9-]/gi)) { return false; }
  else {
    dbQueries.loadRoom(req.params.p, function(error,data) {
      if (!error) { res.json({room:req.params.p, visits:data[0].visits}); }
      else { res.json({err:error}); }
    });
  }
});

app.get('/:p', (req, res) => {
  if (req.params.p.match(/[^a-z0-9-]/gi)) { return false; }
  else {
    dbQueries.saveRoom({room:req.params.p, data:new Date()}, function(error, data) {
      if (!error) { res.sendFile(path.join(__dirname+'/index.html')); }
      else { res.json({err:error}); }
    });
  }
});
```

This might look a little weird at first. Two different functions to handle the same route? Wait, what? Note the different `method` though: POST vs GET. Get's what you get (ha!) when you type the address in your browser. POST, on the other hand, is what you get when you send a POST request: when the page you visit submits a form or, in our case, when the client sends a post request manually with [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).

We add a simple `<span id="visits">` tag to the client HTML and this script (a skeleton for our later client development) to show a visit counter.

#### attractio.js
```javascript
window.onload = function() {
  fetch(window.location.href, {method: 'POST'})
  .then(res=>res.json())
  .then(json=>{
    let visitText = json.visits>1 ? " visits" : " visit";
    document.querySelector('span#visits').innerHTML='with '+json.visits+visitText;
  });
};
```

Now when you visit a valid route, you get the number of times the route has been visited, thanks to our new database connection. Click refresh!

So you can see what we've built so far: an app that will automatically count up the number of visits to any valid route and store those results in a database. It shouldn't be hard to imagine how we can adapt this same code ever-so-slightly (just saving client data about words on a given page & their positions) to turn this into poem-saving code, right?

#### A few things I didn't talk about in the repo set-up

I did a few things to the package.json file without explaining:

* I added [concurrently](https://www.npmjs.com/package/concurrently) as a dev-dependency. Dev-dependencies are development-specific dependencies that you do not need in production.
  * Why don't we need this in production? Good question: our server and database will not be running on the same process...in our case, the server's hosted on Heroku and the database hosted on mLab.
* I modified the `npm start` script to fire up a MongoDb database instance along with our server, so that you don't have to bother with a separate Terminal window:
  * `"start": "concurrently \"mongod --dbpath=./data\" \"node server.js\""`
* I added a `data` folder to the repo for MongoDb
* I added a `.gitignore` to ignore node_modules from version tracking.
* I added an `.gitignore` to data as a little "hack" to keep the folder empty so things work straightaway when you clone the repo and run `npm start`.
