---
layout: post
title:  "Byte at a Time ECB Decryption"
date:   2017-09-26 12:03:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 2 Challenge 12](https://cryptopals.com/sets/2/challenges/12)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s2c12)
### [decryption demo](https://thmsdnnr.github.io/cryptopals/s2c12/)

#### Step 1: What's My Blocksize Again?

First the challenge has us verify the block size and determine the encryption is using ECB mode.

We know that we've reached the block size boundary when adding one additional character to the plaintext results in a change in the ciphertext output length of greater than 1. It will jump by a factor greater than one due to PKCS7 padding (remember, when we reach a block boundary, PKCS7 adds an entire additional block consisting of blockSize # `String.fromCharCode(blockSize)` characters).

Increase the input string one character at a time until the jump is > 2. Larger jump size === block size.

{% highlight javascript %}
let str='A';
let res=randomCrypto(str);
let lastSizeJump=1;
let lastSize=res.length;
while (lastSizeJump<2) {
  let res=randomCrypto(str);
  str+='A'
  lastSizeJump=res.length-lastSize;
  lastSize=res.length;
}
console.log(lastSizeJump, lastSize, res, res.length);
{% endhighlight %}

#### Step 2: Call Me ECB (Maybe)

We test whether we're in ECB mode by feeding our encryption function a really long string of the same character and looking for repeated blocks in the ciphertext (just like we did in the last challenge).

{% highlight javascript %}
console.log(areRepeats(randomCrypto('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA').join(""),16));
{% endhighlight %}

True. It's true.

#### Step 3: Set the Controls for the Heart of the...Random Encryptor Function

We modify our random encryption function from [Challenge 11](https://thmsdnnr.github.io/tutorials/javascript/cryptopals/2017/09/24/cryptopals-set2-challenge-11-an-aes-ecb-cbc-detection-oracle.html). It's now going to accept a plaintext, append an unknown string, and return an AES-ECB encryption result of the concatenation of these two strings, using a randomly generated key. Then we're going to find that unknown string.

The new function looks like this:

{% highlight javascript %}
let theGreatUnknown=hexByTwo(b64ToHex('Um9sbGluJyBpbiBteSA1LjAKV2l0aCBteSByYWctdG9wIGRvd24gc28gbXkgaGFpciBjYW4gYmxvdwpUaGUgZ2lybGllcyBvbiBzdGFuZGJ5IHdhdmluZyBqdXN0IHRvIHNheSBoaQpEaWQgeW91IHN0b3A/IE5vLCBJIGp1c3QgZHJvdmUgYnkK'));
let randomKey=randomBytes(16).map(e=>hexPad(e));

function randomCrypto(pText) {
  pText=txtToHex(pText).map(e=>hexPad(e));
  let input=PKCS7(pText.concat(theGreatUnknown),16);
  let eRes;
  eRes=aesEncryptECB(input,randomKey,16);
  return eRes;
}
{% endhighlight %}

#### Step 4: Wait, What--How Do We Figure Out `theGreatUnknown` (String)?

We know that repeated plaintext yields repeated ciphertext when encryption happens in ECB mode [(challenge 8)](https://thmsdnnr.github.io/tutorials/javascript/cryptopals/2017/09/22/cryptopals-set1-challenge-8-detecting-aes-in-ecb-mode.html). We also know that we can brute force a single character key by trying all possible values and comparing the ciphertext outputs [(challenge 3)](https://thmsdnnr.github.io/tutorials/javascript/cryptopals/2017/09/14/cryptopals-set1-challenge-3-single-byte-XOR-cipher.html).

Let's combine these ideas. We focus on one block of ciphertext at a time. Let's start by just decrypting the first 16 characters of the unknown string, working only with the block containing bytes `0-16`.

1. We can manipulate the plaintext input so that a ciphertext block consists of 15 of our characters and 1 character of the ciphertext.
  - Feed the encryptor function a string `S` with length 15 (one fewer than the block size).
  - The first block of plaintext consists of `S + [character 1 of unknown string]`.
  - Save this encryption result as `oneFewer`.
2. We guess character 1 by enumerating every possible combination of `S` with characters 0-255.
3. We check the encrypted block result of each guess against `oneFewer` until we find a match.
  - This works because it's ECB encryption. Unlike CBC, in ECB, each block's encryption result is entirely independent of neighboring blocks; in CBC, subsequent encryption block results depend on previous blocks.
4. We've found one character. Now iterate and find them all.

**Let's look at steps 1 through 3 first.** I *highly recommend* getting one character decoded before, y'know, nesting for loops like birds in springtime. Hey, I was optimistic too. When I tried to run my code without testing intermediate results, haphazardly nesting three for loops and setting loop variables with whimsy, my computer began heating up and the fan came on, which was unusual.

"Maybe it's working," I thought. Minutes pass. I pulled out the ol' calculator and multiplied the upper bounds of each of the nested arrays. Over a million. That's how I knew I was on the wrong track.

Here's what my code looks like for guessing the first block, with comments referencing the steps given above. Note the addition of the solved characters so far to the variable `str`, which is a first step toward achieving #4, iterating this process for the entire string.

{% highlight javascript %}
for (var j=16;j>0;j--) { //j == index of the letter in the block we're guessing
  let oneFewer=repeatN('A',(j-1)); // Step 1
  let oneShort=randomCrypto(oneFewer).slice(0,16).join("");
  for (var k=0;k<256;k++) { // Step 2
    let char=String.fromCharCode(k);
    let str=oneFewer+prevBlockGuessed.join("")+char;
    if (oneShort===randomCrypto(str).slice(0,16).join("")) { // Step 3
      // We found a match!
      prevBlockGuessed.push(String.fromCharCode(smartIter[k])); //store it
      break; //then stop checking this character
    }
  }
}
{% endhighlight %}

The implementation of #4 is left as an exercise for the reader. Or, you know, just look at the Github repo. A few hints: you'll need to have *at least* as many `A` characters as there are letters in the string you are attempting to decode. Think about how you found the first character. How many `A`s will you need to add, given an unknown string's length, in order to ensure the block you are guessing contains 15 known characters and one unknown character?

You'll also want to stop once you've finished guessing all the letters in the unknown string, since unless the unknown string is a multiple of the block size, you'll have `A`s left over at the end, and that's just untidy! You can calculate when that 'halting index' will be, and then run an index check on your innermost loop to break out when you have arrived at your destination.

Finally, the code above is quite inefficient. Sure, you could simply loop through every single possible character, (0-255). But if you look at an ASCII chart (`man ascii`) or [this link](http://en.cppreference.com/w/cpp/language/ascii), you'll quickly see a better way of guessing possible letters.

I created an array of ASCII characters indices to loop through, ordered by those I thought to most commonly appear in readable strings. This hearkens back to how we [used character frequency in English texts to score plaintexts](http://localhost:4000/tutorials/javascript/cryptopals/2017/09/14/cryptopals-set1-challenge-3-single-byte-XOR-cipher.html#step-4-scoring-the-results) generated from guessing all possible values for an XOR key.

 {% highlight javascript %}
  //32, 97 to 122, 65 to 90, 39, 9 to 13
  //40-64, 33-38, 91-96, 1-8, 14-31, 123-255
  let smartIter=`32,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,
  65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,39,9,10,11,12,13,40,41,42,
  43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,33,34,35,36,37,38,91,92,93,94,95,96,1,
  2,3,4,5,6,7,8,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,123,124,125,126,127,128,129,130,131,
  132,133,134,135,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,153,154,155,156,157,
  158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,
  184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,204,205,206,207,208,209,
  210,211,212,213,214,215,216,217,218,219,220,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,
  236,237,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,255`.split(",");
 {% endhighlight %}

I look for spaces first, then lower-case letters, upper-case letters, newline/tab/etc., numbers, then all the rest. Using this shortcut saves a *lot* of loop iterations. Assume the guessed character is a letter. You save 65 iterations per character. In the challenge case there are 137 letters (mostly) to guess, so that's about 9,000 fewer loops performed.

Just because we are using brute force does not mean that we cannot also use finesse.

#### Step 5: Let's Display the Data...Besides Using `Console.log()`

Oh wait, the loop function we've created for decryption takes a really, really long time to run.

There are a few techniques for displaying intermediate results of an executing code block in Javascript. One common example is the "print the current iterator value in a for loop. The gotcha is that you can't just write this:

{% highlight javascript %}
for (var i=0;i<10;i++) {
  setTimeout(function() { console.log(i); },i);
}
{% endhighlight %}

unless you want the output to be 10 `10`s. Why is that? Well, setTimeout says "execute this thing after `i` milliseconds have passed, if you don't have anything else to do, and if this current call is at the top of the event loop". Even setting `i` to zero by leaving it blank says "execute as soon as you're done". The problem is that when the loop's done, the value of i is 10, and so each invocation of the `setTimeout` function will just log what `i` is, 10 times.

There are two ways of fixing this. One more modern way in ES6 is to use `let` instead of `var` to declare the iterator in the for loop instantiation: e.g., `for (let i=0;i<10;i++)`. `Let` declares a *block-scoped* variable rather than a *globally scoped* variable [MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let).

Another solution is to use an IIFE, or "Immediately Invoked Function Expression" [MDN docs](https://developer.mozilla.org/en-US/docs/Glossary/IIFE). That would look like this:

{% highlight javascript %}
for (var i=0;i<10;i++) {
  (function(i) {
      setTimeout(function() { console.log(i); }, i);
  })(i);
}
{% endhighlight %}

The IIFE creates a new scope via the wrapping parentheses. It calls the setTimeout() creation function immediately, passing it the value of `i` at that time. This way, each setTimeout on the event loop has its own copy of `i`, namely a copy that reflects the value of `i` when the timeout was *created*, in the loop's `i`th iteration itself, rather than at the time it was eventually called.

But y'know what? This *still* didn't work, to display the data. I realized that the reason why is that the loop takes a lot of processing power, and so any setTimeouts that we call from within the loop are not going to be executed until that loop is done, full stop.

This brings up something that some perceive to be a limitation about the JavaScript language as a whole, and that is its single-threaded nature.

If you use Javascript to execute an asynchronous function (a function whose return value is not known and must be awaited), like reading a file from the filesystem or requesting responses from a server, you have to pass something known as a "callback". This allows the code to continue executing while it is still waiting on the results of the previous operation.

My for loop is essentially an asynchronous in this way. However, it is asynchronous *on the same machine*. And unlike asynchronous functions, I don't want to wait until the decryption is done to display the result. I want to display intermediate results in real-time so you can see what's going in the loop.

#### Step 6: Multithreading with Web Workers!

The answer: threading! Using something called Web Workers [(MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers), I can run the decryption code in a separate thread. The page code is simply a message receiver.

{% highlight javascript %}
window.onload = function () {
  let res=document.querySelector('div#result');
  var worker = new Worker('decryptTask.js');
  worker.addEventListener('message', function(e) {
    e.data.res=e.data.res.replace(/\n/,'<br />');
    if (e.data.type=='intermediate') { res.innerHTML=e.data.res; }
  });
  worker.postMessage('gogogo');
};
{% endhighlight %}

That's all the page does! It loads the worker script file, adds an event listener to wait for messages from the worker, invokes the worker with the initial postMessage() and then updates a div with the intermediate results. It can display these intermediate results of the decryption as it is proceeding in another thread, because the display code doesn't have to wait on the calculation code to complete before it can function. (I replace the `\n`s in the string with `<br />`s so newlines render as HTML).

The web worker just contains a function to invoke the decryption loop when it receives a message:

`onmessage = function(e) { loopThat(); };`

and then calls `postMessage()` within the for loop every time it has an intermediate result:

`postMessage({'type':'intermediate', 'res':str});`.

Whoa, that is majorly cool, right? I felt this was a really great use case for a multithreading example, displaying the results of a process in real-time. This only scratches the surface. If you want to test the Web Worker code locally (at least in Chrome), you need to serve your code with an HTTP server, since [Chrome doesn't allow loading Web Workers from the filesystem](https://stackoverflow.com/questions/21408510/chrome-cant-load-web-worker).

No worries, just enter: `python -m SimpleHTTPServer` from your code directory in terminal.
Point your browser to `http://localhost:8000`.

Voila! Icy cold.
