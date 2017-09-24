---
layout: post
title:  "Let's Implement PKCS7 Padding!"
date:   2017-09-22 21:26:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 2 Challenge 9](https://cryptopals.com/sets/2/challenges/9)

#### Step 0: We Basically Already Did This in Challenge 7

But here we'll focus on it primarily. And cover the purpose behind cryptographic padding in a block cipher for purposes of both encryption and decryption.

[Good Old PKCS7](https://en.wikipedia.org/wiki/Padding_%28cryptography%29#PKCS7).

{% highlight javascript %}
function padToN(msg,num,padC) { //pads array msg num bytes with padC hexadecimal character
  if ((!msg||num<msg.length)||(!Array.isArray(msg)||padC.length!==msg[0].length)) { return null; }
  else { while (msg.length<num) { msg.push(padC); } }
  return msg;
}

function PKCS7(arr,blockSz) { //returns PKCS7 padded array
  if ((!arr||!blockSz)||(!Array.isArray(arr)||!arr.length)) { return null; }
  let padN=blockSz*Math.floor(arr.length/blockSz)+blockSz;
  let fillCharacter=hexPad(parseInt((padN-arr.length),10).toString(16));
  if (arr.length%blockSz!==0) { arr=padToN(arr,padN,fillCharacter); } //add fillCharacter bytes to make len%16==0
  else { arr=arr.concat(new Array(blockSz).fill('10')); } //add a new 16-byte block
  return arr;
}
{% endhighlight %}

Above is my implementation. You can see some of the helper functions going on here (hexPad to make sure each hex character is length==2). I generalized the padding behavior in `padToN` to pad any array with any number of characters. I then used this to pad out the key used if it is fewer than the number of bits of encryption, or to slice it if it is greater than the number of bits of encryption.

PKCS7 padding is simple to understand. If the size of the message is not an integer multiple of the block size, bytes are added until it becomes an integer multiple. The value of each added byte is the total number of bytes that were added. For example, if the message is 3 bytes short of an integer multiple of 16, three 0x03 bytes are added to the end of the block.

On the other hand, if the message *is* an integer multiple of the block size, an entire block of 0x15 is added (16 0x15 characters) to the end of the message. Why? It's a sort of checksum, to signify that you've reached the end of the message during decryption and that no more data follows. The assumption here is that the encrypted message (at least the *end* of it) does not contain ASCII characters 0-15. Here's an example of how you can code the chopping:

{% highlight javascript %}
function chopPKCS7(padArr) { //accepts padded hex array, returns non-padded array
  if (!Array.isArray(padArr)||!padArr.length) { return null; }
  let last=Number(parseInt(padArr[padArr.length-1],16).toString(10));
  if (last>0&&last<=16) {
    let possibleSlice=padArr.slice(padArr.length-last);
    let lastCh=padArr[padArr.length-1];
    if (last>=padArr.length) { throw new Error('Padding >= array length! Are you sure you wanna do that?'); }
    else if (!possibleSlice.every(c=>c==lastCh)) { throw new Error('String is not PKCS7-padded.'); }
    return padArr.slice(0,padArr.length-last);
  }
  return padArr;
}
{% endhighlight %}

This is a lot of function for what it does, which is this:

* Read the last character (N) of the decoded message.
* If it is between [0x00...0x0F], determine if the message is PKCS7-padded
* If it is, chop off N characters from the end & return
* If it seems like it's a malformed PKCS7 message, throw an error
* If it is not PKCS7-padded, return the decoded message unmodified

Error checks are important whenever you're truncating strings. Even though it's rare to have the ASCII characters 0-15 in a string, it could still happen. PKCS7 says we only chop off these characters *if there are that many* characters. If there aren't, it could indicate that e.g., if the message is being transmitted over a network, we are in the middle of a block of ASCII 0-15 and *not* at the end of the message. While it's not the job of the padding code to ensure proper network transmission, error checks here make the code more robust and bugs easier to identify.
