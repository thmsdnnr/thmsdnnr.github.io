---
layout: post
title:  "Fixed XOR"
date:   2017-09-14 13:19:06 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 1 Challenge 2](https://cryptopals.com/sets/1/challenges/2)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s1c2)
### [working demo](https://thmsdnnr.github.io/cryptopals/s1c2/index.html)

Write a function that takes two equal-length buffers and produces their XOR combination.
[info on exclusive OR](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Bitwise_XOR)

XOR (exclusive-or) is a logical operation on two bits. In English, it means "one but not both". Therefore, if bit A and bit B are different, it returns 1. If bit A and B are the same, it returns 0.

Since XOR operates bitwise, we have to break down the buffer into single binary bits.

There are three steps involved.

1. We convert from hexadecimal to binary
2. Perform the XOR
3. Convert from binary back to hexadecimal to yield the result.

#### Step 1: Hexadecimal to Binary

Split each hexadecimal into an array of characters, and group into sets of 2.

Convert each hexadecimal character into its binary equivalent using map.

parseInt() and toString() are handy. parseInt(e,16) says "turn this string, base-16, into an integer", and toString(2) says "turn this integer into a string of base 2".

{% highlight javascript %}
function xorPartnah(first,second) {
  first=first.split("");
  second=second.split("");
  let a=[];
  let b=[];
  for (var i=0;i<first.length;i+=2){a.push(first[i].concat(first[i+1]));}
  for (var i=0;i<second.length;i+=2){b.push(second[i].concat(second[i+1]));}
  a=a.map(e=>parseInt(e,16).toString(2)).map(e=>{
    while (e.length<8) { e="0"+e; }
    return e;
  }).reduce((a,b)=>a.concat(b));
  b=b.map(e=>parseInt(e,16).toString(2)).map(e=>{
    while (e.length<8) { e="0"+e; }
    return e;
  }).reduce((a,b)=>a.concat(b));
{% endhighlight %}

Unfortunately, toString truncates leading zeroes (that is, if the result is "00011000", the string returned is "11000"). This is totally "correct" behavior, but for our purposes, we add back zeroes with a while loop to make sure that each of our binary strings is exactly 8 bits long.

There are two new "higher-order" functions you might not be familiar with here: map and reduce.

**Map** takes an array, iterates through each element, performing the specified operations, and returns a new array. In this case, we want each of the hexadecimal numbers in our array converted to binary.

**Reduce** does what it sounds like: it takes an array and performs an operation on each of its members to reduce the array to a single element. In this case, we want a single element that contains *all* the binary numbers stuck together. That'll make it easier to iterate over them and to apply the XOR operation.

Check out these MDN resources on these extremely useful functions!
[reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce?v=a), [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map). They are very powerful, and the usage you see here is just the tip of the iceberg.

Now that we have our two arrays, XOR is easy.

{% highlight javascript %}
  let xR=[];
  let aR=[];
  //XOR the strings
  for (var i=0;i<a.length;i++) { xR.push(a[i]^b[i]); }
  xR=xR.join("");
{% endhighlight %}

We're almost sorted. What's left? Well, we've now got a new binary string that's the XOR result, but we have to turn it back into hexadecimal! So, we'll go through, slice it into groups of eight, and use good ol' map again to hexadecimalize (is that a word?!) all the 8-bit binary strings.

{% highlight javascript %}
  for (var i=0;i<xR.length;i+=8) { aR.push(xR.slice(i,i+8)); }
  aR=aR.map(e=>parseInt(e,2).toString(16)).join("");
  return aR;
{% endhighlight %}

A simple .join("") zips up each element of the array into a tidy string, and we've done it! Input validation and length checking of the two strings is left as an exercise for the reader :).
