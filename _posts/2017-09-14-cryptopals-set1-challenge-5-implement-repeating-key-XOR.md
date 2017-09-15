---
layout: post
title:  "Implement Repeating-Key XOR"
date:   2017-09-14 21:21:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 1 Challenge 5](https://cryptopals.com/sets/1/challenges/5)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s1c5)
### [working demo](https://thmsdnnr.github.io/cryptopals/s1c5/index.html)

Repeating-key XOR is slightly more nuanced than the single-byte XOR cipher in challenge 3 [tutorial HERE](https://thmsdnnr.github.io/tutorials/javascript/cryptopals/2017/09/14/cryptopals-set1-challenge-3-single-byte-XOR-cipher.html). The approach is similar, however. Instead of XORing each bit of the plaintext with a single hex character, we XOR each bit with each character of a repeating key.

#### Step 1: Convert Plaintext and Key to Binary

This is pretty trivial. I wrote a helper function that splits a given string into individual characters and maps each character to its base-10 ASCII representation. You obtain this ASCII value with `.charCodeAt()`. Then, this value is converted to base 2, and padded out with zeros to exactly eight bits.

{% highlight javascript %}
function textToBinary(string) {
  return string.split("").map(e=>{
    let bin=parseInt(e.charCodeAt(),10).toString(2);
    while (bin.length<8) { bin="0"+bin; }
    return bin;
  });
}
{% endhighlight %}

#### Step 2: Hash Plaintext with Repeating Key

Once we get both the plaintext and key in binary, all that's left is to XOR the bits together. The only nuance here is the indexing of the repeating key in the inner loop. The outer loop takes every byte of the plaintext and then, in the inner loop, iterates over each bit.

We use the outer loop index, i, to determine which letter in the K to use in the XOR with `i%K.length`. In the case of the key "ICE", `i%K.length` will repeat: 0, 1, 2, 0, 1, 2... and hence K[i%K.length] will repeat 'I', 'C', 'E', though of course with the binary representation of the ASCII value of these integers.

You could also imagine laying out each character of the plain text in a row and below it writing ICEICEICEICEICEICEICEICEICEICE over and over until you reach the end of the plain text. This is what we're doing programatically.

{% highlight javascript %}
function repeatXOR(text,key) {
  let O=textToBinary(text);
  let K=textToBinary(key);
  let XOR=[];
  let intXOR=[];
  for (var i=0;i<O.length;i++) {
    for (var j=0;j<O[i].length;j++) { intXOR.push(O[i][j]^K[i%K.length][j]); }
    XOR.push(intXOR.join(""));
    intXOR=[];
  }
  return XOR.map(e=>{
    let hex=parseInt(e,2).toString(16);
    return hex.length<2 ? "0"+hex : hex;
  }).join("");
}
{% endhighlight %}

That's really all there is to it. The bass kicked in and the fingers are jumping. Don't worry, it'll get trickier soon when we try to reverse this process :)
