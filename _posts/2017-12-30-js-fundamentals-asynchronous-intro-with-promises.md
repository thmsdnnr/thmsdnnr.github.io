---
layout: post
title:  "Code a Weather App! [Async Code, Promises, and APIs]"
date:   2017-12-30 23:40:04 -0500
description: Asynchronous JS, APIs, and Promises
author: Thomas Danner
lang: en_US
categories: tutorials javascript projects
tags: async, asynchronous, AJAX, async, promises, API, weather, javascript, networks, API
---

# Let's make this.

<p data-height="342" data-theme-id="32039" data-slug-hash="zpwpRQ" data-default-tab="js,result" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Simple Weather App" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/zpwpRQ/">Simple Weather App</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
##### First, we need to understand a few concepts: asynchronous JavaScript, and fetching data from an API.

## I'll be done in a few.

We often want to create applications that interact with external resources. We're Imgur, and we have images hosted on a file system somewhere in the cloud. Or we're Reddit, and we have a database of posts to request from when a user visits a page.

A simpler example: a weather app that gets the user's current location and displays a forecast.

We want to get weather information, but we do not want (and cannot afford!) to buy and maintain weather stations, network them wirelessly, power them, code them, and then call to them individually to display updates.

An [Application Programming Interface (API)](https://en.wikipedia.org/wiki/Application_programming_interface) allows us to interact with other programs through the Internet. The National Weather Service has plenty of weather stations, and it's set up a server from which we can request data. Now we don't have to buy the weather stations: we just have to tell the server the data we're interested in, wait for it to arrive, and then display it in the app how we'd like.

To be specific, this sort of API is called [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer). That's an acronym for Representational State Transfer, which is really just a fancy term for a *contract that a server establishes for client requests*.

The server says: "here are my endpoints (routes you can call)", and here are the verbs you can call them with (GET/POST/PUT/DELETE). Also, here are the parameters that I understand.

### Real-world example: National Weather Service API

Check out the "API Reference" tab [here](https://forecast-v3.weather.gov/documentation?redirect=legacy) for the National Weather Service.

You'll find a table with an `Endpoint` column, `Formats`, and `Details`. The endpoint is the URL where you request the resource from. In this case, the base URL is `https://api.weather.gov`. You append the endpoint you are interested in to the base URL.

For example, if we want the weather forecast for a point (say latitude: 35.0693 longitude: -91.6716), we request from `https://api.weather.gov/points/35.0693,-91.6716/forecast`.

Let's use the metaphor of a phone call to your health insurance company. You dial the number. This is the "base URL". You hear a menu "press 1 for... press 2 for...".

You choose an option, and then tell the person what information you're interested in getting. Hey, I pressed 3 for weather info about a point. The latitude is X and the longitude is Y. In a moment, the person responds. "Here's the forecast!" Or, "Hey, we don't have any data for that request."

### JSON responses

The server responds with [json](http://www.json.org/) (JavaScript Object Notation), a common format for exchanging data on the web. It can be used with any language, not just JavaScript. It's *Js*on because it's based on a subset of the JavaScript language.

There are two methods in JavaScript that we can use to deal with this format:

1. [JSON.stringify()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)
2. [JSON.parse()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)

JSON.stringify() does just what it sounds like: it turns an object into a string that we can send across a network. JSON.parse() turns that string back into an object when it arrives at its destination. If you remember high-school math, these functions are *inverses* of one another. That means that:

+ `JSON.parse(JSON.stringify({"a":"an object with one key-value"}))`
  * yields: `{"a":"an object with one key-value"}`

+ `JSON.stringify(JSON.parse("{"a":"an object with one key-value"}"))`
  * yields: `"{"a":"an object with one key-value"}"`

Note the quote marks around the brackets there.

> Think of the object as a letter. You can put it in an envelope with an address, sealing it up with JSON.stringify(). When the sender receives it, they use JSON.parse() like a letter opener to retrieve the contents.

You'll use `JSON.parse()` whenever you are dealing with data you've fetched from a web service. In fact, as we'll discuss later in this tutorial, `fetch` has this functionality built-in.

**Pro-tip: a very common error you'll see is "unexpected token" in JSON:**

```
VM2250:1 Uncaught SyntaxError: Unexpected token a in JSON at position 0
    at JSON.parse (<anonymous>)
    at <anonymous>:1:6
```
##### This usually means that you've done something like `JSON.parse('asbsdab')`: trying to parse a string that has not been `JSON.stringify()`-d and is not in the proper format.

### what happens with your app while you're on hold?

Networks respond way faster than a customer service line for your health insurance, but there is still a delay between the request and response.

JavaScript is single-threaded. It can only execute one line of code at a time. We want to do other things in our app while we're waiting for all the data to come in. We want to display the data we have. We want to be able to update the UI to say, "Hey, it's coming, one sec.". Maybe we display and hide a spinner icon. We want to remain responsive to the user even when we're waiting for a request to finish.

Imagine what would happen if in our app we said, "Okay, request this" and then *waited* for the request to come back before the app was responsive again?

All the other lines of your code in the application would have to wait. We can't even play bad call holding music. The user gets frustrated and hangs up. Your app fails.

> We want to be able to put something in our code that says, "We're going to request this data. It'll take a little bit for it to get back, but when it arrives, perform these actions". Meanwhile, we're moving on.

Enter [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

### Promises promises...

Here's (one example of) what a promise looks like:

```javascript
function fetchWeatherData(latitude, longitude) {
  return new Promise(function(resolve, reject) {
    // request data asynchronously (from network API, file system, database...)
    if (data.error) {
      reject(data.error);
    } else {
      resolve(data);
    }
  });
}
```

A Promise is an object that takes a callback function. The callback is an interface for code that invokes the promise. We perform our asynchronous request within the body of the `Promise`, and then we define how to invoke the callback function, depending on the results returned by our request.

Here is how we would call this code above:

```javascript
var data=fetchWeatherData(35.0693,-91.6716);
```

Now we cannot just `console.log(data)` and access our data. If you do, you'll get this:

`Promise {<pending>}`

Remember, the function is going to take some unknown amount of time to respond. Instead, we say "when the promise has been resolved (with our data) or rejected (with an error), do these things".

```javascript
data.then(function(d) {
  //do something with the Resolved data
}).catch(function(err) {
  //handle the Rejected error
});
```

##### `.then()` calls `function(d)` with a resolved value; `.catch()` calls `function(err)` with a rejected value.

There are three possible states for a promise:

1. `Pending`: neither `resolve` nor `reject` have been called. This is the state that `data` is in when we first call `fetchWeatherData`.
2. `Resolved`: we called `resolve(data)` within the function with the returned data.
3. `Rejected`: we called `reject(data.error)` because the API threw an error.

### chaining Promises > Promise-ception.

We can chain promises together, or have the `resolve()` of one promise await the resolution of earlier promises. In fact, this is exactly how our wrapper function `fetchWeatherData()` will work when we fill in the missing `//fetch the data` piece.

[Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) is a Javascript interface for requesting data across a network. Hopefully you're thinking "Wait though, that's asynchronous". You're correct. Can you guess what `fetch` returns? A promise. Bingo!

```javascript
function fetchWeatherData(latitude, longitude) {
  return new Promise(function(resolve, reject) {
    fetch('https://api.weather.gov/points/'+latitude+','+longitude+'/forecast')
      .then(function(response) {
        if (response.status===200) { return response.json(); }
        else { reject(response.status); }
      })
      .then(function(parsedResponse) { resolve(parsedResponse); });
  });
}
```

This might look confusing. Let's walk through piece by piece.

`fetchWeatherData` is a wrapper that takes the `latitude, longitude` point we are interested in. It returns a `Promise` saying "Hey, I'll get you the data as soon as I can."

Your program says, "Okay, I'll do other stuff, but when you do pass me back the data, here's what I'm going to do with it." The program goes merrily along.

Within `fetchWeatherData`, we call `fetch`. Fetch does the same thing as the wrapper.

`fetch`: "Hey `fetchWeatherData`, I'm going to request data from the server. It might take a while".
`fetchWeatherData`: "No problem fetch. When you get the response, here's what I'll do."

### when fetch calls us back

When we get the response back, it will be in the JSON format. We check the `status` of the response. This is the [HTTP status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) of the request. You're probably familiar with the good ol' 404. 200 means "we're good everybody!". If the status is anything but this, we'll reject the promise with the status code.

If the status *is* 200, then we'll return `response.json()`, which parses the JSON and returns it in the object form we desire.

> Okay, in reality `.json()` does more than just parsing. The `response` body that `fetch` returns is actually a [Stream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API). Using the [`.json()`](https://developer.mozilla.org/en-US/docs/Web/API/Body/json) method on the `response` object reads the stream so that we don't have to. More to come on Streams: they deserve their own series.

If you'd like to visualize what's happening, paste this into the JavaScript console on your browser:

```javascript
fetch('https://api.weather.gov/points/35.0693,-91.6716/forecast')
.then(res=>res.json())
.then(parsed=>console.log(parsed))
```

<img src="https://i.imgur.com/x5PNhjs.png" width="420" alt="Console.log of fetch called with JSON.parse()">

See the `periods` array (`parsed.properties.periods`)? That contains 14 forecasts that we can display in our app for the requested latitude and longitude. Sweet!

Here's what it looks like when we don't parse the JSON. Note the `body` property of the response of type `ReadableStream`:

```javascript
fetch('https://api.weather.gov/points/35.0693,-91.6716/forecast')
.then(res=>console.log(res))
```

<img src="https://i.imgur.com/NdqW61O.png" width="420" alt="Console.log of fetch called without JSON.parse()">

See how initially it is a Pending Promise that then turns into the response? Cool, huh?

### let's apply this.

To fetch weather forecasts for a user, we have to get their location. We can actually use something built in called the [Geolocation API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation/getCurrentPosition) to grab the user's current position.

We call `navigator.geolocation.getCurrentPosition` and give it callback of `success` and `error` (another example of an asynchronous function!). On success, we receive an object that contains `pos.coords.latitude` and `pos.coords.longitude` which we can pass to the weather service API. We'll use the `fetchWeatherData` function we've already written (not listed below):

The Weather API's `properties.periods` is an array of objects like this:

```JSON
{
  detailedForecast: "Mostly sunny. High near 6, with temperatures falling to around 3 in the afternoon. Wind chill values as low as -20. Northwest wind 10 to 20 mph, with gusts as high as 30 mph."
  endTime: "2017-12-30T18:00:00-06:00"
  icon: "https://api.weather.gov/icons/land/day/cold?size=medium"
  isDaytime: true
  name: "Today"
  number: 1
  shortForecast: "Mostly Sunny"
  startTime: "2017-12-30T11:00:00-06:00"
  temperature: 6
  temperatureTrend: "falling"
  temperatureUnit: "F"
  windDirection: "NW"
  windSpeed: "10 to 20 mph"
}
```

We can choose which items we'd like to display. Then we just iterate through, format them, and output them to an HTML div.

This is all it takes:

```javascript
showLoader();
function success(pos) {
  let lat=pos.coords.latitude;
  let long=pos.coords.longitude;
  reverseGeocode(lat, long).then(data=>{
    if (data.numFound>0) {
      let res=data.result[0];
      const header=document.querySelector('span#location');
      header.innerHTML='Weather forecasts for '+res.city+', '+res.state;
    }
  });
  fetchWeatherData(lat, long).then(data=>{
    data.properties.periods.forEach((p,idx)=>{
      let period=document.createElement('div');
      let img='<img src=${p.icon} alt=${p.shortForecast}>';
      period.id='period_'+p.number;
      period.classList.add('forecastPeriod');
      period.innerHTML=`${img}<span id="wxHeader"><b>${p.name}</b><br />
      ${p.temperature}˚F ${p.temperatureTrend!==null ? p.temperatureTrend : ''} |
      ${p.windSpeed} ${p.windDirection}</span><br />
      ${p.detailedForecast}`;
      dataDiv.appendChild(period);
  });
  hideLoader();
});
}
navigator.geolocation.getCurrentPosition(success, error);
```

##### Note that we could extend this app to allow the user to input a zip code. You could use Geocoding with [Google Geocoder](https://developers.google.com/maps/documentation/geocoding/intro), or a simple dictionary mapping zip codes to a latitude and longitude, extracted from US Census Data like this [github Gist by erichurst](https://gist.github.com/erichurst/7882666). This would be another example of chaining promises. User enters City. Pass to Google Geocoder (or grab the lat/lon list), then await response. Then when you have the latitude and longitude, pass it to the Weather API. Await the response from that. Then pass the weather result to your user through the UI.

ReverseGeocode looks just like `fetchWeatherData`. It returns a promise, we await the response, and if the data is there, we parse and display it. Easy.

```JSON
result: {
  city:"Chicago"
  countryCode:"US"
  distance:11.884522997519374
  formatedFull:"1234, Random Street, Chicago, Illinois, United States, US"
  geocodingLevel: "HOUSE_NUMBER"
  houseNumber: "1234"
  lat: some_latitude_float
  lng: some_longitude_float
  state:"Illinois",
  streetName:"street name"
}
```

Finally, we show and hide a CSS loader by toggling the display style of the loader. We could just as easily create a CSS class `.hidden` with `display:none`, and toggle the class on the `div` with `classList.toggle('hidden')`.

```javascript
const showLoader = () => document.querySelector('div.loader').style.display='inline';
const hideLoader = () => document.querySelector('div.loader').style.display='none';
```

### mobile-first CSS

We add some CSS styling to make things pretty. [Mobile-first](https://developer.mozilla.org/en-US/Apps/Progressive/Responsive/Mobile_first) means that we design things to look nice on mobile and then define how they will grow when viewed on larger devices. If you're on Google Chrome, you can access the mobile simulation in Developer Tools by clicking the cell phone icon near the top. It gives you a dropdown where you can select a device (or choose a screen size) and then select a screen orientation: landscape or horizontal.

> The principle of mobile-first CSS is: design things to look good on a small screen first. Then define how elements will change when viewed on a larger screen.

We can define how things will look on larger screens with a [media query](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries).

```css
/*wider screens */
  @media only screen and (min-width: 751px) {
    /*any code in here overrides small screen code*/
}
```

In this case, I've chosen to make my `forecastPeriod` container change to two columns when there is enough space to display it cleanly, by changing its width to 45%. Since it's inside of a flexbox container with width > 90%, this makes two forecasts display per row.

This makes things look good on mobile in portrait mode (width 360px):

<img src="https://i.imgur.com/cVxLIG7.png" width="360" alt="Weather app viewed in portrait mode on mobile">

mobile landscape mode (width 640px):

<img src="https://i.imgur.com/Crd9MUE.png" width="360" alt="Weather app viewed in landscape mode on mobile">

desktop (width 1000px):

<img src="https://i.imgur.com/0qykLOn.png" width="360" alt="Weather app viewed in desktop mode">

### CORS-anywhere

If you peek at my code for the reverse geocoder, you'll notice something strange:

```JavaScript
fetch('https://cors-anywhere.herokuapp.com/http://services.gisgraphy.com/reversegeocoding/search?format=json&lat='+latitude+'&lng='+longitude)
```

cors-anywhere?!

[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) is Cross-Origin Resource Sharing. Browsers define guardrails when accessing resources at a different origin other than the server that a site is being served from. In this case, we're requesting resources from the National Weather Service and `gisgraphy.com`. Neither of these resources are hosted at `https://codepen.com`, so CORS headers must be present on the response for browsers to allow access to these resources.

[CORS Anywhere](https://github.com/Rob--W/cors-anywhere) proxies the request and adds CORS headers. It's hosted on Heroku, and it's `https://`. This allows us to proxy the response from services.gisgraphy.com through the server and receive a response over https:// that has the necessary headers.

The National Weather Service sets CORS headers, so we're good there. But you'll get an error if you try to fetch from http://services.gisgraphy.com, because Codepen is https:// and gisgraphy is http://. You get this:

"Mixed Content: The page at 'https://codepen.io/thmsdnnr/pen/zpwpRQ' was loaded over HTTPS, but requested an insecure resource. This request has been blocked; the content must be served over HTTPS.

You can send a `fetch` reuest to the endpoint `iscorsneeded` on the CORS-anywhere app. This will return a response *without* CORS headers set. If you try to fetch this endpoint via CodePen, you get this:

Failed to load https://cors-anywhere.herokuapp.com/iscorsneeded:
No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'https://s.codepen.io' is therefore not allowed access. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.`

### next steps

There are an [incredible number](https://www.programmableweb.com/apis) of APIs on the web. Using what you've just learned , you can now interact with *any* of them. Some require an API key or special headers, but these are just small tweaks to your `fetch` request. Try building an app that integrates data sources together in a new and exciting way. Once you learn how to interact with other services, you acquire a huge mechanical advantage when it comes to leveraging small bits of code to do great things. Code on!
