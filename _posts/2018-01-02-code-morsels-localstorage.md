---
layout: post
title:  "WebStorage API"
date:   2018-1-2 20:04:04 -0500
description: WebStorage API, localStorage, and sessionStorage
author: Thomas Danner
lang: en_US
categories: tutorials javascript fundamentals
tags: JS, sessionStorage, localStorage, webStorage
---

The [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) allows you to store data for a given origin (page on your site) that can persist across sessions. Data is stored as key-value pairs. There can only be one value for a given key, although that value can be a `JSON.stringify()`d object. Web Storage is [well-supported across browsers](https://caniuse.com/#feat=namevalue-storage) with few limitations, aside from [browser-specific limits](http://dev-test.nemikor.com/web-storage/support-test/) on storage space.

## demo: you were here last @

<p data-height="205" data-theme-id="32039" data-slug-hash="OzjmgO" data-default-tab="js,result" data-user="thmsdnnr" data-embed-version="2" data-pen-title="LocalStorage Example — Last visited on:" class="codepen">See the Pen <a href="https://codepen.io/thmsdnnr/pen/OzjmgO/">LocalStorage Example — Last visited on:</a> by thmsdnnr (<a href="https://codepen.io/thmsdnnr">@thmsdnnr</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

Check out the `console.log()` on CodePen too. Go to Create > New Pen, and then type `localStorage` in the developer console to see the storage. Codepen uses the three variables to assign the percentage a pane takes in the editor (HTML/CSS/JS) and whether the console is toggled or not. Try dragging around the panes and watch how the `localStorage` changes.

<img src="https://i.imgur.com/lLQVUfh.png" width="700px" alt="Codepen localStorage usage in developer console">

## two types: long (session) and longer (local)

There are two types of storage that differ only in the amount of time their data is persisted.

`sessionStorage` maintains data so long as the browser program is open. It doesn't matter if the tab or window is closed. As long as the browser instance exists, `sessionStorage` persists. When the instance dies, so does the stored data.

`localStorage` persists data even when the browser instance itself is closed and reopened.

## storage methods

These methods are the same for sessionStorage and localStorage.

`.getItem(key)` returns the value associated with the given key, or null if the key does not exist

`.setItem(key, value)` sets the item with the given key to the specified value. Overwrites existing values.

`.removeItem(key)` removes the given item from storage.

`.key(i)` returns the `i`th key of localStorage, or null if the key does not exist. We can easily find all the keys in localStorage like this:

```javascript
function getLocalStorageKeys() {
  let i=0;
  let keys=[];
  while (localStorage.key(i)!==null) {
    keys.push(localStorage.key(i));
    i++;
  }
  return keys;
}
```

## storing and retrieving objects

When you store keys and values in the Storage API, they will be coerced to strings. If you have an object in valid JSON format, you have to call `JSON.stringify()` on the data that you want to store, and then use `setItem` on this stringified object.

When you retrieve the item with `getItem`, you can turn the result back into an object for your program with `JSON.parse()`.

Example:

```javascript
const allMyCats = {
  fluffy: {
    age: 4,
    nickname: 'billy'
  },
  costello: {
    age: 6,
    nickname: 'grunt'
  }
};
localStorage.setItem('cool-cats',JSON.stringify(allMyCats));
```

`localStorage.getItem('cool-cats')` will yield this string:

`"{"fluffy":{"age":4,"nickname":"billy"},"costello":{"age":6,"nickname":"grunt"}}"`

which can be parsed to give us our original `allMyCats`.

`localStorage.removeItem('cool-cats')` returns undefined and removes 'cool-cats'.

## achtung!

Web Storage should be treated as user input since it is directly modifiable by the user. If you use localStorage to persist some data for a user that you save to a database or elsewhere, be sure to validate and sanitize it. Also, do not store any sensitive data like passwords in localStorage.

## A fun project

Try writing code that remembers the window size that a user set last time they visited the site, and resize the window onLoad when the user returns. Would you use sessionStorage or localStorage? Are there ways to make such a feature more usable and less obtrusive?
