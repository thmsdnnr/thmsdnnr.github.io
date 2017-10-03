---
layout: post
title:  "Let's Implement a Stream Cipher (CTR)!"
date:   2017-10-03 07:40:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 3 Challenge 18](https://cryptopals.com/sets/3/challenges/18)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s3c18)
### [demo](https://thmsdnnr.github.io/cryptopals/s3c18/)

#### Step 0: Little/Big Endian

If you've implemented the rest of AES encryption, this is literally the hardest part of the challenge.

You have a number represented by one byte. Which way do you read the bits? Most significant (leftmost bit) to least significant (rightmost bit).

Now say you have a number represented by two bytes. You know which way to read the bits, but which way do you read the *bytes*? `\x01\x0F` or `\x0F\x01`? Both of these numbers can represent either 31, if the left one (`\x01\x0F`) is read as big-endian and the right one (`\x0F\x01`) is read as little-endian. [More from wikipedia](https://en.wikipedia.org/wiki/Endianness).

The format specified for the Stream Cipher is:

> format=64 bit unsigned little endian nonce, 64 bit little endian block count (byte count / 16)

Unsigned means that none of the bytes in the nonce are used to specify positive or negative. All of them represent numbers.

Little endian means that the *least* significant or smallest values come first. That means that if we're representing the value 15 for block count, the block looks like this:

`\x0F\x00\x00\x00\x00\x00\x00\x00` instead of this:
`\x00\x00\x00\x00\x00\x00\x00\x0F`

I wrote a function to "little-endianize" a number into its hex representation.

{% highlight javascript %}
function lilEndian(number) {
    //returns quadword number in little endian, hexadecimal representation
    if (!Number.isInteger(number)||number>Number.MAX_SAFE_INTEGER) { throw new Error('Whoa nelly, can\'t convert that!'); }
    //max safe is 9007199254740991
    let arrResult=[];
    let temp;
    for (var i=63;i>=0;i--) { //64 bits
      if (number>=Math.pow(2,i)) {
        temp=Math.floor(number/Math.pow(2,i));
        arrResult.unshift(temp);
        number-=temp*Math.pow(2,i);
      } else { arrResult.unshift(0); }
    }
    arrResult=arrResult.map((e,idx)=>idx%8==0?arrResult.slice(idx,idx+8):null).filter(e=>e);
    //we have to reverse because parseInt is big endian but we store the bits little endian
    arrResult=arrResult.map(e=>hexPad(parseInt(e.reverse().join(""),2).toString(16)));
    return arrResult;
  }
{% endhighlight %}

The perceptive reader will notice that I restrict super large numbers but quip, "Wait tho, that's smaller than a quadword!" You are right, dear reader. Technically, a quadword can hold a number up to `9,223,372,036,854,775,807`, but the [MAX_SAFE_INTEGER constant](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER) in JavaScript is `9,007,199,254,740,991`.

The counter that we encrypt, which increments by 1 for every 16 bytes, is really, really unlikely to ever get that large (if it does, we throw an error and stop encrypting). We'd have to be encrypting about 200 petabytes (PB) 1 PB = 10<sup>15</sup> bytes = 1,000 terabytes, and we'd be using the same nonce to do it. No no.

The chosen nonce could possibly be larger than MAX_SAFE_INTEGER, so for a proper implementation that provides the maximum [noncespace](https://en.wikipedia.org/wiki/Cryptographic_nonce), we'd create a way of safely handling numbers larger than JavaScript's internal limit so that we could represent the full range of numbers a quadword can store.

#### Step 1: Let's Stream That (Block) Cipher

Here's info on CTR mode [(wikipedia)](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Counter_.28CTR.29). Notable points:

* Encryption and decryption *both* use the AES encryption process on the keystream bytes.
  - All we do to encrypt or decrypt the plaintext/ciphertext is XOR with this encrypted keystream.
* CTR converts our block cipher into a stream cipher.
* No plaintext padding is required.
  - Just keep XORing with fresh keystream until you run out of plaintext or ciphertext.

I create a simple object that will give me 16 bytes of encrypted keystream each time I call `.next()`:

{% highlight javascript %}
var Counter = {
  nonce: null,
  count: null,
  next: function() { //returns next 16 bytes
    if (!this.nonce) { throw new Error('You must set a nonce with setNonce()!'); }
    this.count!==null ? this.count++ : this.count=0;
    return this.nonce.concat(lilEndian(this.count));
  },
  setNonce: function(n) {
    if (this.nonce) { throw new Error('Nonces are immutable. Create another counter.');}
    this.nonce=lilEndian(n);
  }
};
{% endhighlight %}

Then, here's all that I have to do to encrypt (*or* decrypt -- the process is identical!):

{% highlight javascript %}
let Z, dataText;
  let ctr=Object.create(Counter);
  ctr.setNonce(N);
  let res=[];
  let keyStream=aesEncryptECB(ctr.next(),key,bits);
  for (var i=0;i<I.length;i+=16) {
    let block=I.slice(i,i+16);
    let next=ctr.next();
    console.log(next);
    res=res.concat(addRoundKey(block,keyStream));
    keyStream=aesEncryptECB(next,key,bits);
  }
  Z=res.map(hexToTxt).join("");
{% endhighlight %}

Chunk the plaintext up into 16 byte blocks and convert to hexadecimal. Get a block of AES-encrypted keystream. XOR each plaintext byte with its corresponding keystream byte. Done! Literally that simple.
