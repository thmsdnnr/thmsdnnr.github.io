---
layout: post
title:  "ECB Byte At a Time (Harder!)"
date:   2017-09-28 07:24:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 2 Challenge 14](https://cryptopals.com/sets/2/challenges/14)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s2c14)
### [demo](https://thmsdnnr.github.io/cryptopals/s2c14/)

#### Step 1: We've (Almost) Done This Before

The only difference between this challenge and [#12](https://thmsdnnr.github.io/tutorials/javascript/cryptopals/2017/09/26/cryptopals-set2-challenge-12-byte-at-a-time-ecb-decryption.html) is that we are now prepending a random-sized block of random bytes before the string we enter. We can solve this challenge as soon as we reduce the problem to that of 12, where the string we enter *begins* the plaintext.

We want to know how much to pad before we reach the first block we "control" the completely -- the point where the next character we enter is the *first* in a block of 16. And, the kicker: how can we do this without knowing the length of the number of bytes prior to our message in the plaintext?

`[random # of bytes] + [our bytes...how many?] + [known length, unknown values]`

We know repeated plaintexts yield repeated ciphertexts in ECB mode. We've already used the technique of adding values one at a time until we note a "jump" in ciphertext length > 1 to determine blocksize ([see: #12.1](https://thmsdnnr.github.io/tutorials/javascript/cryptopals/2017/09/26/cryptopals-set2-challenge-12-byte-at-a-time-ecb-decryption.html#step-1-whats-my-blocksize-again)). The challenge hint? "Stimulus and response".

#### Step 2: Give a Little to Get a Little

What's the trick to solving this? Add padding until we see a repeated pattern in the ciphertext.

**We assume:**

* no repeated 16-byte blocks in the randomly-prepended bytes at the beginning -- good assumption
  + a `(1/256)^32 * (1/32) = 1 in 3.1e78 ~ #atoms in universe chance`
* no repeated 16-byte blocks in the appended ciphertext -- bad assumption, *but*
  + we are looking for the first repeat, so even if there are repeats later on, it's okay

{% highlight javascript %}
let txt=repeatN('A',32);
let res=randomCrypto(txt);
while (!areRepeats(res,16)) {
  txt+='A';
  res=randomCrypto(txt);
}
console.log(areRepeats(res,16,true));
{% endhighlight %}

We can be smart about where to start guessing if we think through the problem.

* Best-case scenario? The randomly prepended bytes have `length%16==0`, meaning none of our padding bytes are "used up" to fill out the remainder of the 16 byte block.

* Worst-case scenario? The random bytes have `length%16==15`, meaning we have to use 15 `A`s to fill out the block, and we'll only see repeats after adding 16 x 2 more `A`s after these first 15.

So at best, 32 `A`s will produce repeats (start there); at worst, 15+(16*2) `A`s will be required. If we see more than 47 `A`s being used, we know there's a bug in the code.

#### Step 3: Got a Little, Decrypt the Lot

Now we just need to change a few parameters in our solving code from the previous challenge.

I modified `areRepeats` to take an optional `idx` parameter, which returns the index of the first repeated ciphertext block): `if (i!==lIdx) { return idx ? i : true; }`. We also know the number of `A`s added to obtain these repeats, and we know that the repeated blocks require 32 `A`s to produce. So we can subtract `total #As - 32` to get the # required to produce the "first block that we control" in the plaintext. I call this `padOffset`, and it's added in my for loop to the `A` padding I produce for any given block.

`let oneFewer=repeatN('A',(padOffset+i+j-1));`

Before, we set `numBlocks=Math.ceil(thing.length/16)-1;`. This number of blocks is now offset by `index`, so we have `numBlocks=Math.ceil(thing.length/16)-1 + index;`.

Also, our stop point will have to be changed, since we don't want to bother trying to decrypt the random bytes prepended to the message.

Before, we had `for (var i=numBlocks*16;i>=0;i-=16)`; now we will have `for (var i=numBlocks*16;i>=index*16;i-=16)`.

Then the problem is just Challenge 12. We only had to change the loop indices! Here's what the loop looks like now:

{% highlight javascript %}
function loopThat(padOffset,blockOffset) {
  let numBlocks=blockOffset+Math.ceil(thing.length/16)-1;
  let prevBlockGuessed=[];
  let jStopIdx=(numBlocks+1)*16-thing.length-(blockOffset*16);
  for (var i=numBlocks*16;i>=blockOffset*16;i-=16) {
    for (var j=16;j>0;j--) { //letter in the given block we're guessing, 1-16
      if (i===blockOffset*16&&j<=jStopIdx) { j=0; break; }
      let oneFewer=repeatN('A',(padOffset+i+j-1));
      let oneShort=randomCrypto(oneFewer).slice(i,(numBlocks+blockOffset+1)*16).join("");
      for (var k=0;k<smartIter.length;k++) { //guess all possibilities, smartly
        let char=String.fromCharCode(smartIter[k]);
        let str=oneFewer+prevBlockGuessed.join("")+char;
        postMessage({'type':'intermediate', 'res':str});
        if (oneShort===randomCrypto(str).slice(i,(numBlocks+blockOffset+1)*16).join("")) {
          prevBlockGuessed.push(String.fromCharCode(smartIter[k]));
          break;
        }
      }
    }
  }
  postMessage({'type':'final', 'res':prevBlockGuessed.join("")});
}
{% endhighlight %}
