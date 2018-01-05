---
layout: post
title:  "DOM Selectors"
date:   2018-1-4 23:56:04 -0500
description: Introduction to DOM Selectors
author: Thomas Danner
lang: en_US
categories: tutorials javascript fundamentals
tags: JS, querySelector, querySelectorAll, DOM, selectors
---

The [Document Object Model (DOM)](https://en.wikipedia.org/wiki/Document_Object_Model) is how browsers turn your code into a user experience. I know, it's just one character away from "doom", but I promise: after this tutorial, you'll gain some clarity.

### The DOM

Here's a basic webpage.

```HTML
<html>
    <head>
        <title>a cool page</title>
    </head>
    <body>
        <p>a paragraph here</p>
    </body>
</html>
```

How does the browser know to translate these HTML tags and the data they enclose into what we see?

It begins by parsing the source code into the DOM tree. In the tree, there are parents and children. The top of the tree is the `document` element. Beneath that element (in our example) is the `html` element. Nested within HTML is head -> title, and body -> p. Note the "parent -> child" relationship in the tree.

As soon as a page loads, we can use JavaScript to select and manipulate any of the elements of the page via the DOM tree. Before we are able to select or manipulate the DOM, it has to exist. This might seem intuitive, but we have to tell our code this explicitly. Code that selects or manipulates the DOM should be wrapped in a `window.onload(){}`. Otherwise you'll get an error. Things will not work.

### Selecting Elements

Our `document` provides a few methods to select its elements. Each method is passed a string: a name, tagname, ID, or className, or a CSS selector (in the case of querySelector).

#### selecting single elements

`document.querySelector()`
##### select the element returned by a css selector, e.g., 'div#content'

`getElementById()`
##### select the element with the given ID

Elements should not have the same ID. If two do, however, `getElementById` will return whichever one comes first on the page.

#### selecting multiple elements

`getElementsByClassName()`
##### select all elements with a given class name

`getElementsByName()`
##### select all elements with a given name, e.g., `<input name="username" />`

`getElementsByTagName()`
##### select all elements with a given tag name (e.g., `form` or `button`)

`document.querySelectorAll()`
##### select all the elements returned by a css selector, e.g., 'div.content'

### What to do with our selection?

`console.dir()` is your best friend. If you call it on a selected element, you'll get a huge list of properties. We won't discuss them all here, but there are a few that are commonly used:

* `classList` is the list of classes currently applied to an element.
  * `classList` has a method `.toggle()` that can be used to toggle a class on and off for an element. This can be used to show/hide an element, for example
* `style` is the CSS style that is being applied to the element.
* `innerHTML` is the HTML present within a tag
* `innerText` is the innerHTML with the tags removed (text only)
  * `textContent` is the standardized and recommended version of `innerText` to use.
* `value` is the value of an input element like a form or textarea

### a simple example: word frequency with `getElementsByTagName`

Paste this into the JS console when you're on any page with paragraphs. [Wikipedia](https://en.wikipedia.org/wiki/HTML) is good:

```javascript
let A=Array.from(document.getElementsByTagName('p'));
let joinedWords=A.map(e=>e.textContent.split(" ")
  .filter(e=>e.length>0))
  .reduce((acc,b)=>acc.concat(b),[])
let d={};
joinedWords.forEach(word=>d[word] ? d[word]++ : d[word]=1);

let kv = Object.keys(d).map(k=>[k,d[k]]);
kv.sort((a,b)=>b[1]-a[1])
console.log(kv.slice(10))
```

getElementsByTagName returns an "array-like selection", so we use `Array.from()` to give us something we can `map` over. Then we map each HTML Node to its textContent, filter out empty elements and spaces, concatenate each paragraph's text, and count the occurrences of each word using a hash. We then map the hash's keys and values to an array and sort that array in ascending order based on word frequency. Finally, we log out word frequency for the given page.

See how powerful a simple selector can be? Try using selectors in your own code or on pages you visit to select elements of interest. Try selecting and styling elements using CSS, and by toggling classes.

Once you're comfortable with selecting elements, we can discuss how to add event listeners to elements, in order to make the elements themselves responsive to changes in the page and user interaction. This is just the tip of the iceberg of what JS can do. Stay tuned!
