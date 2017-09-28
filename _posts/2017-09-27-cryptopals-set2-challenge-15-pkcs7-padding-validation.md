---
layout: post
title:  "PKCS#7 padding validation"
date:   2017-09-28 11:10:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 2 Challenge 15](https://cryptopals.com/sets/2/challenges/15)
### no repo for this challenge
### no demo for this challenge

#### Step 1: We've Already Done This (but let's talk abt it)

The challenge is to write a function that takes a string and determines if it is PKCS#7 [(doc)](https://en.wikipedia.org/wiki/Padding_(cryptography)#PKCS7) padded. If it is, then we remove the padding; if it is not, we throw an error.

{% highlight javascript %}
function chopPKCS7(padArr) { //accepts padded hex array, returns non-padded array
  if (!Array.isArray(padArr)||!padArr.length) { return null; }
  let last=Number(parseInt(padArr[padArr.length-1],16).toString(10));
  if (last>0&&last<256) {
    let possibleSlice=padArr.slice(padArr.length-last);
    let lastCh=padArr[padArr.length-1];
    if (last>=padArr.length) { throw new Error('Padding >= array length! Are you sure you wanna do that?'); }
    else if (!possibleSlice.every(c=>c==lastCh)) { throw new Error('String is not PKCS7-padded.'); }
    return padArr.slice(0,padArr.length-last);
  }
  return padArr;
}
{% endhighlight %}

This function takes an array and checks the last character. It converts that character into a number and, if that number is between 0 and 256, it checks with `every` [(mdn docs)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every) to see if that number of characters (`possibleSlice`) from the end of the string are all equal to the same value: namely, the length of the padding characters. If they are, then the numbers are removed and the chopped array is returned. If they aren't, then an error is thrown. Done!
