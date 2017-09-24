---
layout: post
title:  "An AES CBC/ECB Detection Oracle!"
date:   2017-09-24 10:20:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 2 Challenge 11](https://cryptopals.com/sets/2/challenges/11)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s2c11)
### [working demo](https://thmsdnnr.github.io/cryptopals/s2c11/index.html)

#### Step 0: What is an Oracle?

This isn't *The Matrix*. But you can pretend it is, you know, if it makes you feel better abt yourself. An oracle is a function that can ascertain some information about an output.

First, we need to generate a function to feed data to the function that's going to be guessing what kind of input it's getting.

The function relies a lot on random bytes:
  - to append or prepend with the plaintext
  - to generate a key and IV

{% highlight javascript %}
function randBytes (num) {
  if (!num||num<1) { throw new Error ('Must create at least one byte!'); }
  let res=[];
  for (var i=0;i<num;i++) {
    res.push(parseInt(Math.floor(Math.random()*255),10).toString(16));
  }
  return res;
{% endhighlight %}

At first I misunderstood the challenge. I didn't understand that I could vary the input to my encryption function. I thought, okay, based on some encryption results of a *single* plaintext (like "testing"), I have to figure out whether the output is ECB or CBC with a randomized key and IV each time. This is actually impossible. It would amount to breaking AES encryption as a whole, since I have no knowledge of the key or the ciphertext.

If you are able to manipulate the input that the encryption function receives though, things get much easier. We already know how to [detect ECB mode (Challenge 8)](https://thmsdnnr.github.io/tutorials/javascript/cryptopals/2017/09/22/cryptopals-set1-challenge-8-detecting-aes-in-ecb-mode.html). We're able to detect the use of AES-ECB by looking for repeated ciphertext blocks. These blocks are repeated because of repetitions in the plaintext. So, just feed the encryption function some repeated plaintext. `AAAAAAAAAAAAAAAA`!

One or two blocks of plaintext input? No better than random guessing. Then a big jump when you get close to 3 blocks (3x16 bytes), and perfect results at 4 blocks. This makes sense if you think about the random encryption function. It's prepending and appending 5-10 random bytes to the plaintext. If they're random, we assume there's no repetition. So, let's think about this:

How many bytes do we have to put between the two random chunks in order to assure that at least *two blocks* contains only `A`s (so that we have a definite repeated block)?

Worst case scenario, starts with 5 random bytes, so that the first 11 `A`s end out that block, then 32 As for adjacent repeating blocks, so 43 As should give a perfect result all the time. Best case scenario, start with 10 random bytes so we only need 6 `A`s to close out the block, then 32 `A`s, so 38 total.

I threw the guessing machinery in a for loop and ran 1000 trials with different lengths of the `AAA` string. Here's what I got:

% Right | length:
--- | ---:
*49%* | 34
*50.3%* | 35
*52.2%* | 36
*56.3%* | 38
*75.1%* | 40
*100%* | 43

It basically matches exactly what we'd expect from classical probability. If we assume the chances of picking a 5, 6, 7, 8, 9, or 10 is totally random, then there's a (1/6) chance of getting the "best-case-scenario" (only needing 38 As to guess correctly).

`(1/6)^1000` is basically zero, so we'd expect that the chance of guessing well with the ideal number of bytes (38) will be pretty low. Surprisingly though, it's about 6% better than flipping a coin. This is a hint: `Math.random()` is *pseudorandom*. We're not flipping coins here. This is one of the pitfalls of implementing your own crypto: generating true randomness.

For every extra random byte, you need an extra `A` to get duplicate ciphertext blocks. So with two extra As, you can have 5, 6, or 7, picked randomly and still get duplicates. We see what we'd expect there: 75% (half the time we get 5, 6, and 7 and score duplicates; the other half of the time, we don't).
