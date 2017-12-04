---
layout: post
title:  "Attractio 1: Build The Server!"
date:   2017-12-04 09:06:04 -0500
categories: tutorials javascript attractio
---

### [>> pt. 1 repo](https://github.com/thmsdnnr/attractio-tutorial/tree/master/pt1)

#### Step 0: Let's Build The Server!

Our server is going to need to do a few things.

1. Serve static files for the client HTML, CSS, and images whenever users hit a valid route
2. Serve generic "Oops, that route's invalid, here's how to use the app" when the route's invalid
3. Store and retrieve the state of a given poetry board when a user loads a route from the database
4. Store and maintain a list of connected clients via WebSockets
5. Facilitate client communication within a room and server communication with the client

Today we're going to do #1 and #2. We'll cover database handling in Part 3, WebSocket set-up in Part 4, and then we'll build out the client skeleton and deploy the app.

#### Step 1: Serving Static Files via Express

```javascript
const express=require('express');
const path=require('path');
const http=require('http');
const app=express();
const appPORT=process.env.PORT || 8080;
const server=http.createServer(app);

app.use('/static',express.static(path.join(__dirname,'/static')));

app.get('/:p', (req, res) => {
  if (req.params.p.match(/[^a-z0-9-]/gi)) { return false; }
  else { res.sendFile(path.join(__dirname+'/index.html')); }
});

app.get('/*', (req, res) => res.sendFile(path.join(__dirname+'/usage.html')));

server.listen(appPORT);
```

OK, what's going on here? You'll often see code that looks like this for a NodeJS server that's delivering web content. We include a few modules at the top: `express`, `path`, and `http`.

Let's have a quick run-down on what these things do:

#### [Express](https://expressjs.com/)

Express helps with routing HTTP requests. An HTTP request is sent when a browser sends a message to your server, either requesting content (like visiting a webpage) or posting data (like form submission during login).

#### [path](https://www.npmjs.com/package/path)

`path` helps us define relative paths to files across operating systems. This path is a file system path on the server itself, *not* a web address. `__dirname` grabs the current directory that we are in and then prepends it to the given folder (in this case, `/static`, which is where we're storing our static web files, like the client-side JS script we're going to write, as well as HTML and CSS files)

Now that you know what `path` does, this line should make sense:

`app.use('/static',express.static(path.join(__dirname,'/static')));`

This says "Hey, whenever you get a request in `/static`, look for it inside the `static` directory in the filesystem". This request can be a `<script>` or `<style>` from a file, an `<img>`, anything. This way we don't have to rewrite paths whenever the location of our code changes: we can define the location of resources relative to the rest of our project, and `path` syncs up to match our environment.

#### [http](https://www.npmjs.com/package/http)
`http` is a built-in NodeJS module for handling HTTP requests. Express routes requests, and `http` passes those requests to Express. Every time a server makes a request to our app on the provided port, `http` calls the function we pass it.

`const server=http.createServer(app)` tells the `http` module what function to call when it is sent a request, and then `server.listen(appPORT)` tells `http` which port to listen for requests on. We define this port here: `const appPORT=process.env.PORT || 8080`.

What's the `process.env.PORT` thing? Well, when we deploy the server to Heroku, it will be assigned a port dynamically that is automatically proxied to port 80, the default port for HTTP requests. We don't know what port we'll get, so we grab it dynamically from the process our server lives in.

If the server doesn't receive a port from `process.env`, we're in a dev environment, and the server will listen on port 8080. Then you can access the server when it's running locally by entering `http://localhost:8080` in your browser.

#### Now the routes!

```javascript
app.get('/:p', (req, res) => {
  if (req.params.p.match(/[^a-z0-9-]/gi)) { return false; }
  else { res.sendFile(path.join(__dirname,'/index.html')); }
});

app.get('/*', (req, res) => res.sendFile(path.join(__dirname,'/usage.html')));
```

OK, a user made a request to our server. `http` passed the request to Express. How do we know what the request is and what we should do? Express filters requests into categories based on two criteria:

1. route (the URL that the browser requested)
2. method [(list of HTTP methods)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods). Common methods include
* `GET`: retrieve data at the requested route (read-only)
* `POST`: submit data to the server at the specified route

[app.get()](http://expressjs.com/en/api.html#app.get.method) takes a path and then a callback, which is called with a `request` and `response`. You let Express know what routes to match on for a given request type, and then when a request matches that route, Express passes the function you define the request information in `req` and the response object that you can use to respond to that request.

Note that Express passes the request to the *first* match, __not__ every match.

Our routes are very simple here, only two `get` routes. We're going to handle "posting" via WebSocket messages. All that our routes do is check to see if the URL passed is valid.

  - if so, serve up client HTML for our app
  - if not, serve up client HTML to display an "error" page showing proper usage

Any valid route will consist of letters and numbers, upper-case or lower-case, and hyphens. We'll check to see if the route requested matches this pattern. If so, it's valid. If not, we return false, and the request goes to the next `app.get` statement.

`app.get('/:p', (req,res) => {` allows us to access `req.params.p` in the function body. This variable will be equal to whatever the user put in the address bar after the slash. So if they access `http://localhost:8080/fun-cuddly-cats`, `req.params.p` will be `fun-cuddly-cats`.

The `*` character is a wildcard to match any route. Our second route `app.get('/*', (req, res) =>` matches anything our regex rejects as invalid, like `http://localhost:8080/mon$t3r5!`.

Routes with a `*` are called "catch-alls". They should be placed last in your code, since you usually want to match on a more specific route. They are analogous to the `default` in a `switch` statement.

#### Set up simple files to serve

Now that we've set up our server to post routes, we'll set up some simple files to serve. They'll go in our `/static` folder. You can take a look at the repo for this tutorial if you're following along yourself.

#### Install the project!

If you've already followed the instructions to grab the project files, you can navigate to that folder on your computer in your Terminal. `cd` to `pt1`, and run the following commands:

* `npm install`
* `npm start`

Navigate to [http://localhost:8080/](http://localhost:8080/) and try navigating to valid [http://localhost:8080/cats-and-dogs](http://localhost:8080/cats-and-dogs) and invalid [http://localhost:8080/c4t55ndd0g5](http://localhost:8080/c4t55ndd0g5) routes. The server recognizes the routes and displays the appropriate page depending on the URL's validity.

It works! More to come :)
