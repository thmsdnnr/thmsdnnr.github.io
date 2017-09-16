---
layout: post
title:  "Break Repeating-Key XOR"
date:   2017-09-16 11:26:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 1 Challenge 6](https://cryptopals.com/sets/1/challenges/6)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s1c6)
### [working demo](https://thmsdnnr.github.io/cryptopals/s1c6/index.html)

It's easy enough to implement repeating key XOR when encoding a plaintext. But what about recovering a plaintext from a ciphertext that has been encoded this way? Ah ah, a little trickier.

I found the tricky part of this challenge to be passing intermediate results in between functions in code, particularly functions we wrote in previous challenges.

For instance, in Javascript, the XOR (^) operator only works on a bit level. The encoded text in the challenge is encoded base64. It's easy to forget whether you should be working with bits and bytes (e.g., the blocks should be blocks of bytes, eight bits, but when you do XOR, you are manipulating bitwise, so you probably have an array with bit elements instead of byte elements).

You'll see this as we go through the steps. The challenge text gives pretty good instructions to follow for each phase, so let's go through each one.

#### Step 1: Write a Function to Caculate Hamming Distance

The Hamming Distance is calculated by comparing two strings and counting the number of changes one string must undergo (specifically, the number of bits that must be mutated) to become the same as another string. Sound similar? It's XOR again! If the bits are different, the distance is 1. If the bits are the same, the distance is 0. So to calculate the Hamming Distance, just convert the text to binary (we've seen this before), and compare each bit.

{% highlight javascript %}
function hamming(a,b) {
  let dist=0;
  a=a.split("").reduce((a,b)=>a.concat(b));
  b=b.split("").reduce((a,b)=>a.concat(b));
  for (var i=0;i<a.length;i++) { dist+=a[i]^b[i]; }
  return dist;
}
{% endhighlight %}

We could write the distance calculation in the loop two ways (same thing):

`if (a[i]!==b[i]) { dist+=1; }` or `dist+=a[i]^b[i];`
Cool, huh?

#### Step 2: Convert the Base64 Ciphertext to Binary

We use the `.atob()` function [(MDN docs)](https://developer.mozilla.org/en-US/docs/Web/API/WindowBase64/Base64_encoding_and_decoding) to decode the base64 string, then split the array into single characters, convert each character into its ASCII code (a base-10 decimal value) with `.charCodeAt()`, and finally convert each ASCII code to binary.

{% highlight javascript %}
let binaryArr=atob(b64cipher).split("")
.map(e=>decToBin(e.charCodeAt()))
.reduce((a,b)=>a.concat(b));
{% endhighlight %}

#### Step 3: Guess the Encrypted Text's Key Size

Now things get a little bit tricky. Frankly, this was one of the hardest parts to wrap my mind around. The challenge suggests the following algorithm to guess the key size.

We know the ciphertext has been encoded with repeating-key XOR. This means that the key (of some length, we don't know what) is iterated over and sequentially XOR'd with the plaintext. For instance, if the key is 4 bytes long, plaintext byte 1 is XOR'd with the same byte as byte 5, 9, 13, and so on.

So we'll try a whole bunch of key sizes, from 2 bytes to 40. We'll break the ciphertext's bytes up into blocks of each one of these sizes, and then we'll compare each pair of bytes by using the Hamming function we wrote. In order to make these comparisons meaningful, we'll normalize each comparison by dividing by key size.

On each go, we'll store the result in a dictionary with the score and the guessed keysize. We can sort the dictionary in descending order of score and then grab the top (i.e., lowest) few scores. The lowest score is most likely to be the block size. In this case, I got 29.

Here's what I got when I console.logged my ham:

`{score: 2.5172413793103448, keysize: 29}
{score: 2.8333333333333335, keysize: 24}
{score: 2.8421052631578947, keysize: 19}
{score: 2.888888888888889, keysize: 18}`

Looks like the key size might be 29. Unlike the Cryptopals challenge suggestion to compare just the first two or four blocks of the ciphertext, I calculated *all* the blocks in the ciphertext. The first few blocks were not enough to give me the correct answer for block size.

{% highlight javascript %}
let ham=[];
let one, two;
let runningT=[];
for (var i=2;i<=40;i++) { //try keysizes from 2 thru 40
  for (var j=1;j<binaryArr.length-j*24*i;j+=8*i) {
    one=binaryArr.slice(j*8*i,j*16*i);
    two=binaryArr.slice(j*16*i,j*24*i);
    runningT.push(hamming(one,two)/i);
  }
  runningT=runningT.reduce((a,b)=>a+b,0)/runningT.length;
  ham.push({score:runningT,keysize:i});
  runningT=[];
}
  ham=ham.sort((a,b)=>{
    if (a.score>b.score) { return 1; }
    else if (b.score>a.score) { return -1; }
    else { return 0; }
  }).slice(0,4);

  let blockSz=ham[0].keysize;
{% endhighlight %}

[Here's a great explanation](https://crypto.stackexchange.com/a/8118) of why guessing the right key size shortens the Hamming Distance. There are two reasons:

1. If you XOR a thing with itself, the result is zero. So if you guess the correct block size, XORing the ciphertext sized to this block length means that the key "drops out" of the Hamming calculation, reducing the Hamming distance. If you guess the wrong block size, then the two blocks you compare each have a _different_ key.

2. Intelligible English text exhibits more uniformity than randomly chosen bits. If we guess the block size correctly, it'll fall within the ASCII values for A-Za-z0-9 instead of being anywhere between 0 and 255.

#### Step 4: Break the ciphertext into blocks of KEYSIZE length.

This is a simple FOR loop. The key is to make sure that each block contains KEYSIZE **bytes**, not KEYSIZE bits. I pad out the last block with one 1 and then 0s [per the Wikipedia](https://en.wikipedia.org/wiki/Padding_(cryptography)#Bit_padding)!

{% highlight javascript %}
  let blockedBarre=[];
  for (var i=0;i<binaryArr.length;i+=blockSz*8) {
    blockedBarre.push(binaryArr.slice(i,i+blockSz*8));
  }
  blockedBarre[blockedBarre.length-1]=blockedBarre[blockedBarre.length-1]+"1";
  while (blockedBarre[blockedBarre.length-1].length<blockSz*8) {
  blockedBarre[blockedBarre.length-1]=blockedBarre[blockedBarre.length-1]+"0";
}
{% endhighlight %}

#### Step 5: Transpose the blocks

e.g., "make a block that is the first byte of every block, and a block that is the second byte of every block, and so on."

This step was easy to implement but its purpose was not intuitive to me at first. What's happening here is that since we know the key length, we know what parts of the ciphertext were encoded with the same character of the key: namely, every `i-th` character of the ciphertext, where `i` is the index of this repeated character in the key.

This transposition groups together all the bytes that were encoded with the same character of the key. This way, we can try every possibility for this key character (ASCII 0-255) on each group.

{% highlight javascript %}
let transposedBytes=[];
let interN=[];
for (var i=0;i<blockSz;i++) { //loop 29 times
  for (var j=0;j<blockedBarre.length;j++) { //through each block of size 29*8
    //each time take the i-th byte and assemble new array
    interN.push(blockedBarre[j].slice(i*8,i*8+8));
  }
  transposedBytes.push(interN);
  interN=[];
}
{% endhighlight %}

#### Step 6: Solve Each Block with Single-Character XOR Technique

We transpose, and then we use the same code as from [challenge 3](http://localhost:4000/tutorials/javascript/cryptopals/2017/09/14/cryptopals-set1-challenge-3-single-byte-XOR-cipher.html) to try each possible character (ASCII 0-255) for each byte position of the key. Each character is XOR'd with the corresponding blocks in the cryptotext, the XOR result is converted to ASCII, and this string is scored using English language text character frequencies.

When we use the correct key, we will obtain the greatest number of commonly-appearing letters in an English text (since the correct key decodes a plaintext). We pick the highest scoring character for each position in the key.

{% highlight javascript %}
  let keys=[];
  for (var i=0;i<256;i++) { keys.push(decToBin(i)); }

  let keyChars=[];
  for (var i=0;i<transposedBytes.length;i++) { keyChars.push(guessTheKey(transposedBytes[i])); }
  keyChars=keyChars.map(e=>String.fromCharCode(e)).join("");
{% endhighlight %}

keyChars holds the secret key! Vanilla Ice again. `Terminator X: Bring the noise`. In order to decrypt the original texts given the key, we

That's really all there is to it. The bass kicked in and the fingers are jumping. Don't worry, it'll get trickier soon when we try to reverse this process :)

#### Step 7: We Have the Key. Unlock the Door!

Once we obtain the key, we can decrypt the original text by simply replying XOR again between the ciphertext and the key we discovered. This works because XOR is commutative.

To get a little more intuition and see visually how repeating-key XOR works, I recommend going to the [working demo](https://thmsdnnr.github.io/cryptopals/s1c6/index.html). You'll start off getting a full decryption of the text. Try substituting a single character of the key. Note the periodicity with which the plain text is disturbed & becomes intelligible (this is the effect of the key size). Awesome!
