---
layout: post
title:  "Histograms in D3 and Observable HQ"
date:   2018-3-5 06:56:04 -0500
description: An introduction to creating histograms in D3 and the use of Observable HQ to make interactive notebooks for data exploration.
author: Thomas Danner
lang: en_US
categories: fundamentals
tags: JS, fundamentals, D3, observableHQ, histograms, frequency, dataScience
comments: true
---

## [Handy Dandy (ObservableHQ) Notebook!](https://beta.observablehq.com/@thmsdnnr/divvy-rides-per-hour-in-chicago)
<img src="/assets/pics/histo.png" alt="Histogram of seven days of divvy ride data, binned into 24 hour-long intervals for each day" width="500px">
##### I made an ObservableHQ notebook that you can use to visualize some of the code from this tutorial. You can toggle the sliders to change the start date dynamically and observe the graph updating in real-time. I'd encourage you to follow along in another tab with the notebook and to play around with the code.


## How Many Times Did It Happen?

[Histograms](https://en.wikipedia.org/wiki/Histogram) visualize a collection of items based on the number of things from that collection that fit into given bins. The bins are most often of equal width. Histograms help [visualize the distribution of data](http://www.itl.nist.gov/div898/handbook/eda/section3/histogra.htm) and can be used to estimate the probability distributions.

## Getting The Data

I grabbed historical Divvy data from the [City of Chicago Data Portal](https://data.cityofchicago.org/), an awesome open-source resource with hundreds and hundreds of datasets to choose from. [Here's the API docs](https://dev.socrata.com/foundry/data.cityofchicago.org/fg6s-gzvg) for Divvy trips in particular.

I'm interested in calculating the data from a given date seven days into the future, so I need to filter my query by start_time. This is pretty simple.

```javascript
let querystring = `$where=start_time between '${startTime}' and '${endTime}'&$limit=9999999`
let data = d3.json('https://data.cityofchicago.org/resource/fg6s-gzvg.json?'+querystring)
```

The trickiest part is obtaining a value for "seven days from now" without using some date library like Moment.js. We know that a day is 86,400 seconds, or 86,400,000ms. So to add n days, we just add n*86400000ms to a previous date.

```javascript
function addDaysToDate(start, numDays=6) {
  let startDate=new Date(start);
  startDate.setTime( startDate.getTime() + numDays * 86400000 );
  return startDate.toISOString().split("T")[0];
}
```

I can call this function like so:
```javascript
let startTime = "2014-12-25T00:00:00";
let endTime = addDaysToDate(startTime, 6)+"T23:59:59"
```

The weirdness around splitting on T is because we don't want the default date timestamp. Instead, I want to go to the very end of this day (23:59:59). The reason we convert to a date format internally and then grab the date again is that it's far easier to do this than to think about how many days are in a given month, is it a leap year, etc. We convert into and out of the date object to obtain this result. Then we append the timestamp we desire.

## Binning The Data, Times Two

For reference, my data is an array of objects, each corresponding to a single ride.

```javascript
{ bike_id: "1162"
  birth_year: "1987"
  from_latitude: "41.904613"
  from_location: {type: "Point", coordinates: Array(2)}
  from_longitude: "-87.640552"
  from_station_id: "138"
  from_station_name: "Clybourn Ave & Division St"
  gender: "Male"
  start_time: "2014-12-25T00:00:00.000"
  stop_time: "2014-12-25T00:04:00.000"
  to_latitude: "41.911386"
  to_location: {type: "Point", coordinates: Array(2)}
  to_longitude: "-87.638677"
  to_station_id: "118"
  to_station_name: "Sedgwick St & North Ave"
  trip_duration: "205"
  trip_id: "4396063"
  user_type: "Subscriber"
}
```

Since I already have a timestamp in each of these rides as a start time, it's really easy to bin out the data. Here's what I do:

```javascript
function binData(rides) {
  var timeAndDates = new WeakMap();
  rides.forEach((ride)=>{
    const timeBin = ride.start_time ? String(+ride.start_time.split("T")[1].split(":")[0]) : -1;
    const dateBin = ride.start_time.split("T")[0];
    if (!timeAndDates[dateBin]) { timeAndDates[dateBin] = []; }
    timeAndDates[dateBin][timeBin] ? timeAndDates[dateBin][timeBin]++ : timeAndDates[dateBin][timeBin]=1
  })
  return timeAndDates;
}
```
##### I'm going to bin rides by days of the week and then, within each day, by the 24 hours of the day.

For each ride, I key the day of the week with the date part of the `start_time` property (e.g., "2014-12-25"). If the date does not already exist, I add an empty slot for it. If it does exist already, I check to see if the second-level key (date + time bin) exists. If it does, increment the value by one. If it doesn't, set up the bin, and put the item in (set the value to 1). I establish the bins by grabbing the hours of the start time, converting it to a number (to remove e.g., the preceding "0" in "00" — "09"), then re-casting to a String.

I call a generateReport method with the result of binData.

```javascript
function generateReport(binned) {
  var dates = ['Sun', 'Mon', 'Tues', 'Weds', 'Thurs', 'Fri', 'Sat'];
  var newInfo=[];
  Object.keys(binned).forEach((key)=>{
    if (binned[key].length<24) { //missing keys
      for (var i=0;i<24;i++) {
        if (!binned[key][i]) {
          binned[key][i] = 0; //set to zero
        }
      }
    }
    const maxV=max(binned[key]);
    const minV=min(binned[key]);
    newInfo.push({
      data: binned[key],
      statMd: `max: ${maxV.value} @ ${maxV.hour}:00 // min: ${minV.value} @ ${minV.hour}:00`,
      day: key.split("-").slice(1).join("/"),
      name: dates[new Date(key).getUTCDay()]
    });
  });
  return newInfo;
}
```

`generateReport` does two things. First, it accounts for missing keys (if there were no rides in a given period on that day, there will be no value in that entry of the array, and the array length will be less than 24). We set those values to zero. Then I create a string with the min and max values of the day and include the time at which those min and max values occurred. It's a nice property of equally-binned histogram data that in our case, the index of the array corresponds to the hour (indices 0 through 23 -> 0:00 to 23:00).

I also include the day and the day name: it could be the case that the user selects a day other than Monday, which means the days will not run from Sunday to Saturday in order — in this case, we'll display Weds, Thurs, Fri, Sat, Sun, Mon, Tues, for example. Since we keyed the timeAndDates object with the timestamp itself, this data is easy to extract.

## Graphing The Data

I write a function to graph the data "sparkline-style" when passed an individual data array, such as the ones corresponding to each day we already created.

```javascript
function graphData(data) {
  var margin = ({top: 1, right: 1, bottom: 1, left: 1});
  var x = d3.scaleLinear()
    .domain([0,24])
    .range([margin.left+5, width - margin.right-5])
  var y = d3.scaleLinear()
    .domain([0, d3.max(data)]).nice()
    .range([height - margin.bottom-5, margin.top+5])
  var line = d3.line()
    .x((d,idx) => x(idx))
    .y((d,idx) => y(d) ? y(d) : (idx>0) ? y(data[idx-1]) : y(data[idx+1]))

  const svg = d3.select(DOM.svg(width, height));
  svg.append("path")
    .datum(data)
    .attr("fill", "none")
    .attr("stroke", "steelblue")
    .attr("stroke-width", 1)
    .attr("d", line);
  svg.selectAll(".dot")
    .data(data)
    .enter().append("circle")
    .attr("class", "dot")
    .attr("cx", function(d, i) { return x(i) })
    .attr("cy", (d,idx) => y(d) ? y(d) : (idx>0) ? y(data[idx-1]) : y(data[idx+1]))
    .attr("r", 2)
  return svg.node();
}
```

This code is simple: we establish x and y scaling functions, a line function, and then we append the line (path) and the individual points (circle) to the graph.

The only tricky thing here is the ternary expression in the y values for the graph: it could be the case that the value is undefined due to missing data. In this case, we want to interpolate forward or backward (whichever is possible). If `y(d)` is not truthy, set the data equal to the previous element if idx!==0, or current element+1, otherwise.

## Reporting The Data

It's trivial to report the data once we have arranged it so nicely.
```md
| day        | 24-hr graph           | stats  |
| -------------: |:-------------:| :-----|
| ${R[0].name} ${R[0].day} | ${graphData(R[0].data)} | ${R[0].statMd} |
| ${R[1].name} ${R[1].day} | ${graphData(R[1].data)} | ${R[1].statMd} |
| ${R[2].name} ${R[2].day} | ${graphData(R[2].data)} | ${R[2].statMd} |
| ${R[3].name} ${R[3].day} | ${graphData(R[3].data)} | ${R[3].statMd} |
| ${R[4].name} ${R[4].day} | ${graphData(R[4].data)} | ${R[4].statMd} |
| ${R[5].name} ${R[5].day} | ${graphData(R[5].data)} | ${R[5].statMd} |
| ${R[6].name} ${R[6].day} | ${graphData(R[6].data)} | ${R[6].statMd} |
```

Observable notebooks support markdown elements with the opening tag md\` and closing tag \`.

We throw in a year, month, and date slider using [viewof](https://beta.observablehq.com/@mbostock/a-brief-introduction-to-viewof) to dynamically track and set the value:
```javascript
viewof year = html`<input type=range min=2014 max=2018 step=1 value=2014>`
```

## Observables?

Observable notebooks keep track of updated values and update things accordingly. When we move the date sliders, it updates the start data, which updates the end date, which updates the querystring, which gets called by fetch, which updates data, which is called by binData, which is called by generateReport, which updates the data, which is referenced by the report. The notebook performs this [reactive pattern](https://en.wikipedia.org/wiki/Reactive_programming) handling for us.

Observers make total sense for a Mathematica-like notebook. They're also handy for solving problems in day-to-day programming. [RxJS](http://reactivex.io/rxjs/) is a popular library for using observers in JS. It's worth looking at to see how it can streamline and clarify your own code.

## TIL

That's about it! You can use Observable notebooks a lot like you can Mathematica. They're a great way to write about data in a manner that encourages exploration and discovery on the part of the reader.

Check out the [introductory tutorials](https://beta.observablehq.com/collection/introduction) and build a notebook of your own to explore some cool data that you're interested in.

Does your city have a great Open Data portal too? If so, use it. If not, advocate!
