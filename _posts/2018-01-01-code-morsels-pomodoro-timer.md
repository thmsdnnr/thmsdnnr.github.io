---
layout: post
title:  "Let's Code a Pomodoro Timer"
date:   2018-1-1 19:40:04 -0500
description: Pomodoro Timer 1/2
author: Thomas Danner
lang: en_US
categories: tutorials javascript projects
tags: JS, pomodoro, productivity
comments: true
---

## Tomato tom-ahto

<p data-height="220" data-theme-id="32039" data-slug-hash="vpZLeN" data-default-tab="js,result" data-user="thmsdnnr" data-embed-version="2" data-pen-title="Bruschetta" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/vpZLeN/">Bruschetta</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>
##### —<a href="https://s.codepen.io/thmsdnnr/debug/vpZLeN/GnMnbVgmpDNM">full demo</a>

The [Pomodoro Technique](https://en.wikipedia.org/wiki/Pomodoro_Technique) is a time management method that tracks the number of focused work periods you perform and rewards you periodically with breaks so that you stay refreshed.

The technique is simple. Set a Work Timer (usually for 25 minutes). Work on the task.

When the timer completes, evaluate your progress.

Did you complete the period successfully, working on the task throughout? Then put a checkmark on a piece of paper.

Did you get distracted during the period and fail to work the full length? Well, then it doesn't count.

If your piece of paper has fewer than four checkmarks, start a Break Timer (usually 3-5 minutes). Take a short break. Then start the work timer again.

When you've completed four Pomodoros (four checkmarks), take a longer break (15-30 minutes), erase the checkmarks, and continue working.

In an 8 to 10 hour day, you ideally complete 16 pomodoros. You'll have 4 15-30 minute breaks, and 16 3-5 minute breaks.

This comes out to 6 hours and 40 minutes of focused work, and between 1hr 48mins and 3hrs 20 mins of breaks.

## We're going to code a Pomodoro Timer.

We'd like to be able to invoke the timer like this:

```javascript
window.onload = function() {
  const displayDiv=document.querySelector('div#timer');
  var timer = new Pomodoro({displayDiv});
  timer.init();
  timer.startTimer();
}
```

Let's code a basic object to manage our timer.

* We want to specify a duration for our work period and our two break periods, short and long.

* We'd like to be able to pause and resume the timer.

* We need to be able mark a work period as an incomplete Pomodoro.

We configure `checkMarks` (count of the number of successful Pomodoros), a `running` flag, an `interval` to hold the reference to the setInterval function that calls the timer periodically, a `timerType` ('work' or 'break'), a `displayDiv` to hold the div in which time info is displayed, and `lastTick` (the instant we last incremented the elapsed time).

`isValid` is a flag set at the start of each timer run that determines whether the Pomodoro was valid or not (did the user get distracted)? `timerLength` holds the length of the current timer (one of either workSeconds, shortBreakSeconds, or longBreakSeconds). `secondsElapsed` holds the number of seconds that have passed while the timer has been running for the given `timerType`.

```javascript
var Pomodoro=(function() {
  function Pomodoro(args={}) {
    const { workMinutes, shortBreakMinutes, longBreakMinutes, displayDiv } = args;
    //short defaults for testing
    this.workSeconds=workMinutes*60 || 10;
    this.shortBreakSeconds=shortBreakMinutes*60 || 5;
    this.longBreakSeconds=longBreakMinutes*60 || 30;
    // longer defaults for real-world:  25*60; // 5*60; // 15*60
    this.checkMarks=0;
    this.isValid=true;
    this.running=false;
    this.interval=null;
    this.lastTick=null;
    this.secondsElapsed=0;
    this.timerLength=null;
    this.timerType='work'; // 'work' or 'break'
    this.displayDiv=displayDiv;
  };
  return Pomodoro;
})();
```

### Defining Object Methods

We initialize by starting with a work period.

```javascript
Pomodoro.prototype.init = function() {
  this.timerLength=this.workSeconds;
  let res=document.createElement('div');
  res.classList.add('clock');
  res.id='clock'
  this.displayDiv.appendChild(res);
  this.resultsDiv=document.querySelector('div#clock');
  //got distracted
  let b=document.createElement('button');
  b.innerHTML='got distracted';
  this.displayDiv.appendChild(b);
  b.addEventListener('click', ()=>this.isValid=false);
  //pause timer
  let pause=document.createElement('button');
  pause.innerHTML='pause';
  this.displayDiv.appendChild(pause);
  pause.addEventListener('click', ()=>this.togglePause());
}
```

We define a few simple functions to toggle timer type and check whether the timer is a work timer or a break timer.

`wasValidPomodoro` tells us whether `isValid===true`. During a work period, if the user clicks a "got distracted", we continue with the work period, but we do not count that period toward the checkmarks for a longer break.

`displayTime` uses the provided div to output the relevant user-facing timer information. You'll notice that it uses two helper functions. `nTimes` just repeats a given character `c` `n` times and returns it in an array. We use it to update our list of checkmarks. `formatTime` takes the seconds remaining and formats them to a mm:ss display (padding out seconds with a 0 if fewer than 10).

```javascript
Pomodoro.prototype.flipTimerType = function() {
  return this.timerType = this.timerType==='work' ? 'break' : 'work';
}
Pomodoro.prototype.isWork = function() {
  return this.timerType==='work';
}
Pomodoro.prototype.isBreak = function() {
  return this.timerType==='break';
}
Pomodoro.prototype.displayTime = function() {
  this.displayDiv.innerHTML=`${this.timerType}! time left: ${Math.ceil(this.timerLength-this.secondsElapsed)} checkMarks: ${this.checkMarks}`;
}
Pomodoro.prototype.wasValidPomodoro = function () {
  return this.isValid===true;
}
Pomodoro.prototype.displayTime = function() {
  this.resultsDiv.innerHTML=`
  <b>${this.timerType}!</b> ${nTimes('&#10004;',this.checkMarks).join(" ")}
  <br />time left: ${formatTime(Math.floor(this.timerLength-this.secondsElapsed))}<br />successful pomodoros: ${this.pomodoroCount}`;
}
```

### startTimer

We call `startTimer` when the timer is first initialized and also each time the timer type changes. We set the `running` flag, reset `secondsElapsed`, and set the timer interval to call update and display every 1000ms.

```javascript
Pomodoro.prototype.startTimer = function() {
  this.secondsElapsed = 0;
  this.lastTick=Date.now();
  this.running=true;
  this.isValid=true;
  this.interval=setInterval(()=>{
      this.updateTime();
      this.displayTime();
  },1000);
}
```

### updateTime

`updateTime` increments the elapsed time by adding the difference between the current instant and the last instant the function was called. We divide by 1000 because `Date.now()` returns milliseconds, and we are measuring elapsed time in seconds.

Each tick we check if elapsed has exceeded the current timer length. If so, we stop the timer.

Note that we do not assume that `updateTime` gets called every second, even though we define the interval as 1000ms. This is because the event loop timing is [not that precise](https://johnresig.com/blog/how-javascript-timers-work/). We can't just add `1` to secondsElapsed every time `updateTime` gets fired.

Instead, each time the function is called, we calculate the difference between the two time points.

```javascript
Pomodoro.prototype.updateTime = function() {
  if (!this.running) { return false; }
  this.secondsElapsed+=((Date.now()-this.lastTick)/1000);
  if (this.timerLength-this.secondsElapsed<0) {
      this.running=false;
      return this.timerEnded();
  }
  this.lastTick=Date.now();
}
```

### timerEnded

We call `timerEnded` whenever a time interval (work or break period) ends. The timer stops the interval. Then, if we just finished a work interval, we check to see if the Pomodoro was valid, incrementing `checkMarks` if so. We flip the current timer type and set the length of the next time period, calling `startTimer` at the end.

```javascript
Pomodoro.prototype.timerEnded = function() {
  clearInterval(this.interval);
  //if timer was work, prompt user to see if they get a checkmark or not
  if (this.isWork()) { // time for a break
    if (this.wasValidPomodoro()) { this.checkMarks++; }
      if (this.checkMarks===4) { //long break
        this.timerLength = this.longBreakSeconds;
        this.checkMarks=0;
      } else { this.timerLength = this.shortBreakSeconds; }
    } else { //time to work
      this.timerLength=this.workSeconds;
  }
  this.flipTimerType();
  this.startTimer();
}
```

### togglePause

We'd like to be able to pause the timer if we step away from a work or break period and return later. If the timer is not paused when we call togglePause, we stop the interval (no need for it) and set the running flag to false. If the timer was already paused, we resume the timer.

Note how we've been able to code pause by simply composing existing object methods. Since we created small functional units that act without side effects, we can compose them in different ways to produce different behaviors. This makes code functionality easier to debug, shorter, less error prone, and easier to unit test and reason about.

```javascript  
Pomodoro.prototype.togglePause = function() {
  if (this.running) {
    this.running=false;
    clearInterval(this.interval);
  } else {
    this.running=true;
    this.interval=setInterval(()=>{
      this.updateTime();
      this.displayTime();
    },1000);
  }
}
```

## That's that!

It works! It's simple, but it's enough to use as a productivity tool. Note how it works even if it is open in a separate tab. It's also minimal in that it only requires user intervention when the user fails to complete a Pomodoro. Otherwise, it works silently in the background and can be viewed in the page title or open tab if in a browser.

## Additional Features

* We can enhance the UI to make it prettier.
* We can use the [Notifications API](https://developer.mozilla.org/en-US/docs/Web/API/notification) and [Vibration API](https://developer.mozilla.org/en-US/docs/Web/API/Vibration_API) to make subtle indications of the start and end of pomodoro periods.
* We can use LocalStorage to track "Pomodoros for the day" so that users can leave the site and return later.
* We can establish "daily pomodoro goals"
* We can track Pomodoro success over time: days, Pomodoros completed, # times distracted
