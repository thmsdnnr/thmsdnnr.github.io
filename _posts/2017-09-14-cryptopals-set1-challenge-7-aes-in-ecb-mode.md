---
layout: post
title:  "Let's Code AES! (AES in ECB Mode)"
date:   2017-09-21 21:26:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 1 Challenge 7](https://cryptopals.com/sets/1/challenges/7)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s1c7)
### [working demo](https://thmsdnnr.github.io/cryptopals/s1c7/index.html)

#### Step 0: Just Don't Code Your Own Crypto

Really, don't do. At least that's what people say. There are so many little things you'll miss. Not to mention that ECB's deterministic nature makes it awful to use for cryptography (so if you do roll your own, don't do it with AES-ECB). But, if you must do it (like I did), you'll learn quite a lot of things in the process. And truthfully, it is a bit cool to have a working implementation of a crypto standard that's not just a wrapper of OpenSSL a la Ruby, etc. The reward is intangible! But I felt all warm & fuzzy inside.

#### Step 1: Learn All The Things.

Implementing AES is a relatively simple process that combines over a dozen relatively simple processes, which makes it, in fact, not quite so simple as one thought at the start. AES is a block encryption algorithm known as an "iterated block cipher". It uses a fixed block size and the same key for both encryption and decryption. The key is expanded for each round of encryption to form an "Expanded Key". In each round, a slice is taken from the Expanded Key and only used for this round.

It's sort of an advancement of repeated-key XOR. What's different? To name a few:

* there are multiple rounds of XORing the key with the plaintext
* there are other bit scrambling processes besides XOR to make the message harder to decrypt, like:
  - mixing columns
  - shifting rows
  - multiplication in a Galois field
  - bitwise transformation using lookup tables
* the key doesn't repeat: an "expanded key" is produced long enough for a fresh slice in each round

I won't go into every step of the implementation. The following resources proved invaluable.

* [NIST standard](http://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf)
  - good for validating expanded keys

* [every step laid out, easy to follow pdf](http://www.adamberent.com/documents/AESbyExample.pdf)
  - good for coding individual functions

* [step-by-step for a single message](https://kavaliro.com/wp-content/uploads/2014/03/AES.pdf)
  - good for validating each and every step of your functions for a 16-byte input (contains all the intermediate state matrices)

* [stick figure guide](http://www.moserware.com/2009/09/stick-figure-guide-to-advanced.html)

#### Step 2: Some Key Concepts

I found the most difficult part of implementing AES to be making all the pieces talk to one another. There are many conversions that take place. Between plaintext and hexadecimal. Between hexadecimal and base64. Base64 back to hexadecimal. The fact that you use hexadecimal as an array index for lookup but that internally, JavaScript stores hex values less than 16 in unpadded form (0x0A is stored as 'A').

I created lots of helper functions to solve these language idiosyncrasies and to keep the format of my data the same when it was passed between functions. I made the following design decisions:

* Represent everything as hexadecimal values internally, padded two two characters per hex value
* Unless performing an operation on the 4x4 state matrix, keep the state as a simple 1xNUMBER_BYTES array
* Create functions that can be reused for both encryption and decryption with, for instance, a flag (example: shiftRows slices "forward" or "backward") based on whether we are encrypting or decrypting
* Minimize the number of conversions between a flat byte array and a 4x4 state matrix
* Try to encapsulate functionality and ensure reuse (e.g., my padding implementation of PKCS7 can be generalized to any block size, not just 16 bytes)

{% highlight javascript %}
function PKCS7(arr,blockSz) { //returns PKCS7 padded array
    if ((!arr||!blockSz)||(!Array.isArray(arr)||!arr.length)) { return null; }
    let padN=blockSz*Math.floor(arr.length/blockSz)+blockSz;
    let fillCharacter=hexPad(parseInt((padN-arr.length),10).toString(16));
    if (arr.length%blockSz!==0) { arr=padToN(arr,padN,fillCharacter); } //add fillCharacter bytes to make len%16==0
    else { arr=arr.concat(new Array(blockSz).fill('10')); } //add a new 16-byte block
    return arr;
  }
{% endhighlight %}

#### Step 3: Code All The Pieces

Go through the docs and simply code the pieces one-by-one. You can look at a bit of my code to see how. Most of the pieces are very straightforward. One thing that is conceptually a bit confusing at first is that when multiplying numbers together, you are using a [finite field](https://en.wikipedia.org/wiki/Finite_field). This means that there are a finite number of elements, so multiplication does *not* give the results you are used to seeing with good old decimal numbers in the reals.

While you don't need to know the maths in detail to code the multiplication (you can just code in the lookup tables to translate hex values for encryption and decryption), it helps to understand what's going on conceptually. It's easy enough: do a lookup of both numbers on one table, sum the results, if the sum is greater than 255, wrap around, then look up the result on the results table.

{% highlight javascript %}
let eT='01 03 05 0F 11 33 55 FF 1A 2E 72 96 A1 F8 13 35 5F E1 38 48 D8 73 95 A4 F7 02 06 0A 1E 22 66 AA E5 34 5C E4 37 59 EB 26 6A BE D9 70 90 AB E6 31 53 F5 04 0C 14 3C 44 CC 4F D1 68 B8 D3 6E B2 CD 4C D4 67 A9 E0 3B 4D D7 62 A6 F1 08 18 28 78 88 83 9E B9 D0 6B BD DC 7F 81 98 B3 CE 49 DB 76 9A B5 C4 57 F9 10 30 50 F0 0B 1D 27 69 BB D6 61 A3 FE 19 2B 7D 87 92 AD EC 2F 71 93 AE E9 20 60 A0 FB 16 3A 4E D2 6D B7 C2 5D E7 32 56 FA 15 3F 41 C3 5E E2 3D 47 C9 40 C0 5B ED 2C 74 9C BF DA 75 9F BA D5 64 AC EF 2A 7E 82 9D BC DF 7A 8E 89 80 9B B6 C1 58 E8 23 65 AF EA 25 6F B1 C8 43 C5 54 FC 1F 21 63 A5 F4 07 09 1B 2D 77 99 B0 CB 46 CA 45 CF 4A DE 79 8B 86 91 A8 E3 3E 42 C6 51 F3 0E 12 36 5A EE 29 7B 8D 8C 8F 8A 85 94 A7 F2 0D 17 39 4B DD 7C 84 97 A2 FD 1C 24 6C B4 C7 52 F6 01';
eT=eT.split(" ");
let lT='00 00 19 01 32 02 1A C6 4B C7 1B 68 33 EE DF 03 64 04 E0 0E 34 8D 81 EF 4C 71 08 C8 F8 69 1C C1 7D C2 1D B5 F9 B9 27 6A 4D E4 A6 72 9A C9 09 78 65 2F 8A 05 21 0F E1 24 12 F0 82 45 35 93 DA 8E 96 8F DB BD 36 D0 CE 94 13 5C D2 F1 40 46 83 38 66 DD FD 30 BF 06 8B 62 B3 25 E2 98 22 88 91 10 7E 6E 48 C3 A3 B6 1E 42 3A 6B 28 54 FA 85 3D BA 2B 79 0A 15 9B 9F 5E CA 4E D4 AC E5 F3 73 A7 57 AF 58 A8 50 F4 EA D6 74 4F AE E9 D5 E7 E6 AD E8 2C D7 75 7A EB 16 0B F5 59 CB 5F B0 9C A9 51 A0 7F 0C F6 6F 17 C4 49 EC D8 43 1F 2D A4 76 7B B7 CC BB 3E 5A FB 60 B1 86 3B 52 A1 6C AA 55 29 9D 97 B2 87 90 61 BE DC FC BC 95 CF CD 37 3F 5B D1 53 39 84 3C 41 A2 6D 47 14 2A 9E 5D 56 F2 D3 AB 44 11 92 D9 23 20 2E 89 B4 7C B8 26 77 99 E3 A5 67 4A ED DE C5 31 FE 18 0D 63 8C 80 C0 F7 70 07';
lT=lT.split(" ");

function multiplyG(A,B) { //Galois multiplication of two hex values, correcting for overflow
  let a10=parseInt(A,16).toString(10);
  let b10=parseInt(B,16).toString(10);
  if (a10==0||b10==0) { return '00'; }
  let sum=Number(parseInt(lT[a10],16).toString(10))+Number(parseInt(lT[b10],16).toString(10));
  let res=sum<256 ? sum : sum-255;
  return eT[res];
}
{% endhighlight %}

#### Step 3: Validate Your Results

Using [step-by-step for a single message](https://kavaliro.com/wp-content/uploads/2014/03/AES.pdf), walk through your encryption and decryption by logging out your state matrix at every step for a 16-byte encryption round. Decryption is just encryption steps in reverse, so walk backwards. Make sure your expanded key is correct for every round.

To validate your padding, you can use Ruby's openssl wrapper. There's a simple script in my repo, but you can also do it using the command line.

{% highlight ruby %}
require 'openssl'
c=OpenSSL::Cipher.new('aes-256-ecb')
#or 'aes-128-ecb' (16-bit) or 'aes-192-ecb' (24-bit)
c.encrypt
#or c.decrypt -> you MUST call this before setting the key
c.key='catsandkudzuotherplants&animals!'
data='Here: 16 letters'
#you can set c.padding=0 if you don't want PKCS7 padding
#once you call update, you cannot call it again on that cipher object
eRes=c.update(data).unpack("H*") << c.final().unpack("H*")
#c.final() contains the padding, or "" if you set c.padding=0
puts eRes.join.to_s
{% endhighlight %}

At first I couldn't figure out how Ruby was padding, but it turned out to be [simple PKCS7](https://en.wikipedia.org/wiki/Padding_%28cryptography%29#PKCS7).

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

PKCS7 padding is simple to understand. If the size of the message is not an integer multiple of the block size, bytes are added until it becomes an integer multiple. The value of the bytes added is equal to the number of bytes that are added. So if the message is three bytes short of an integer multiple (say, 13 bytes), three 0x03 bytes are added to the end of the block.

On the other hand, if the message *is* an integer multiple of the block size, an entire block of 0x15 is added (16 0x15 characters) to the end of the message. Why? In order to signify that the end of the message has been reached in encryption. It's a sort of checksum (when you hit this repeated block, you know you've received the entire message). In the decryption process, the extra characters can be chopped off. I left them on in my demo for demonstration purposes.

#### Step 4: WRITE TESTS

I wrote a full test suite for all my functions using Chai and Mocha. [Here's a great link](https://nicolas.perriault.net/code/2013/testing-frontend-javascript-code-using-mocha-chai-and-sinon/) on how to use them in the browser, independent from NPM.

**You absolutely must write tests**, or debugging will become hopelessly complex.

* Write tests before you write functions.
* Make sure the individual functions work before plugging them together. Make sure to double-check the format and data type of each input and output when you are plugging functions together (is that an array, or a string? is that in hex or binary or text or base64?).

#### Step 5: Now Code It Yourself (or don't!)

The entire purpose of coding this was sort of a contrived project to show that I can implement a relatively complex standard and that the results of my code correspond to existing implementations of AES-ECB e.g., in OpenSSL with the Ruby wrapper. It also gave me a better understanding of how AES encryption functions, which is the purpose of the Cryptopals cryptography challenges. JavaScript is perhaps not the ideal language to code AES in, but it was certainly a fun exercise and skill sharpener.
