---
layout: post
title:  "Let's Detect AES! (AES in ECB Mode)"
date:   2017-09-22 9:32:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 1 Challenge 8](https://cryptopals.com/sets/1/challenges/8)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s1c8)
### [working demo](https://thmsdnnr.github.io/cryptopals/s1c8/index.html)

### Now That We Coded It, Let's Detect It

If we can code encryption and decryption, surely we can code code to detect which ciphertext out of many has been encoded with AES-ECB, right? It's actually quite simple, and the concept is very similar to how we were able to detect single-character XOR back in [challenge #4](https://thmsdnnr.github.io/tutorials/javascript/cryptopals/2017/09/14/cryptopals-set1-challenge-4-detect-single-character-XOR.html).

We'll grab the texts and split each one by newlines into an array. We'll then break down each text in the array into 16-byte chunks. The fact that we know what we're looking for (AES-ECB) and that it's block size is 16-bytes makes this even easier than the single-character XOR hunt, where we had to try all possible block sizes between 2 and 40.

{% highlight javascript %}
function blockChunk(texts) {
  let blocks=[];
  let textArr=[];
  let repeatedElements=[];
  for (var i=0;i<texts.length;i++) {
    for (var j=0;j<texts[i].length;j+=16) {
      blocks.push(texts[i].slice(j,j+16));
    }
    textArr.push(blocks);
    blocks=[];
  }
  for (var i=0;i<textArr.length;i++) {
    for (var j=0;j<textArr[i].length;j++) {
      if (textArr[i].indexOf(textArr[i][j])!==textArr[i].lastIndexOf(textArr[i][j])) {
        repeatedElements.push([textArr[i][j], i, j]);
      }
    }
  }
  return repeatedElements;
}
{% endhighlight %}

Finally, we'll iterate through each 16-byte-chunked array, checking to see if this instance of the chunk is the last instance of the chunk in the array. If it is not, then there's a duplicate element.

We find a lot of duplicate elements in one of the arrays:

`"d880619740a8a19b",  "7840a8a31c810a3d",  "08649af70dc06f4f",  "d5d2d69c744cd283",  "e2dd052f6b641dbf",  "9d11b0348542bb57",  "08649af70dc06f4f",  "d5d2d69c744cd283",  "9475c9dfdbc1d465",  "97949d9c7e82bf5a",  "08649af70dc06f4f",  "d5d2d69c744cd283",  "97a93eab8d6aecd5",  "66489154789a6b03",  "08649af70dc06f4f",  "d5d2d69c744cd283",  "d403180c98c8f6db",  "1f2a3f9c4040deb0",  "ab51b29933f2c123",  "c58386b06fba186a"`

Do you see the repetition? Check out the two columns there on the right.

What does the duplication signify? Since the output of blocks encrypted with AES-ECB do not depend on the value of previous blocks, as in a stream cipher, an identical plaintext input yields an identical ciphertext output. Thus, if there are any repeated elements in the plaintext, there will be repeated ciphertext blocks. This knowledge alone is not enough to decrypt the ciphertext without knowing the key, but it does illustrate a major downfall of using AES in ECB mode: namely, it does not thoroughly obfuscate the plaintext. The repetition in the plaintext makes through to the encrypted message.

As a bit of a lookahead to an ECB/CBC oracle (a function that can determine whether a ciphertext was encrypted with ECB or CBC), I changed blockChunk a little bit.

{% highlight javascript %}
function blockChunk(txt,sZ) {
  if (sZ<2) { return null; }
  txt=txt.split("");
  let blocks=[];
  let rep=[];
  for (var i=0;i<txt.length;i+=sZ) { blocks.push(txt.slice(i,i+sZ).join("")); }
  for (var i=0;i<blocks.length;i++) {
    let lIdx=blocks.lastIndexOf(blocks[i])
    if (i!==lIdx) { rep[blocks[i]] ? rep[blocks[i]].push(i) : rep[blocks[i]]=[i]; }
  }
  Object.keys(rep).forEach(key=>{ rep[key].push(blocks.lastIndexOf(key)); });
  return rep;
}
{% endhighlight %}

I created a demo that lets you chunk up a text (specifying chunk size) and highlights repeated elements automatically. I generate a random color for each highlight, and then set it as the background-color of a span enclosing the text. By setting the opacity using the alpha-channel in RGBA, I can ensure that the text color of black still shows up decently well with any color, no matter how extreme. Otherwise you'll get black text over full-on red, blue, or green, which is not fun to read.

{% highlight javascript %}
function rC() {
  let R=Math.floor(Math.random()*256);
  let G=Math.floor(Math.random()*256);
  let B=Math.floor(Math.random()*256);
  return `rgba(${R},${G},${B},0.4)`;
}
{% endhighlight %}

The trickiest part of this code was getting the RegExp to work. I use a regular expression to replace matches that I find in the text from `blockChunk` using the good ol' lastIndexOf method. However, when I add the formatting to the text, sometimes the color that I have highlighted a previous match with is the *same value* as the thing that I am matching on. For instance, I'm matching on hex 0x30 and I just replaced something else further down in the string with `rgba(30,30,30,0.4)`. Not good: it'll mangle the text and actually insert chunks of raw HTML into it.

The fix was to use a [Negative Lookahead](http://www.rexegg.com/regex-lookarounds.html) (thanks RexEgg!) to assert that whatever match I grabbed was *not* followed by a comma and other numbers, which would indicate it was part of a style tag rather than the text that I was styling. I also used a capturing group (that's the $1 deal in the replace) to re-incorporate the match from the text into the replaced result. Basically, I go through every repeated text chunk that I found with `blockChunk`, and use replace to surround each repeat with a style tag with a matching random color.

{% highlight javascript %}
function highlightSpecified(txt,indices) {
    Object.keys(indices).forEach(idx=>{
      var re = new RegExp('('+idx+')(?![,[0-9]]{0,3})',"g");
      txt=txt.replace(re, `<span style="background-color:${rC()}">$1</span>`);
    });
    return txt;
  }
{% endhighlight %}

Sweet, all done with Set 1 of Cryptopals challenges. Can't wait for Set 2!
