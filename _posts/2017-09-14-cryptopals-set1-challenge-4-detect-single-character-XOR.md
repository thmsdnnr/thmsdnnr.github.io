---
layout: post
title:  "Detect Single-character XOR"
date:   2017-09-14 16:01:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 1 Challenge 4](https://cryptopals.com/sets/1/challenges/4)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s1c4)
### [working demo](https://thmsdnnr.github.io/cryptopals/s1c4/index.html)

The challenge? Given a text file of over 300 60-character hex strings, find which one has been encrypted using single-byte XOR.

This one's really short. We lean heavily on the code already generated for challenge 3. All we need to do is load in the strings from the text file, apply the key guessing procedure to each one, and find the string with the highest score from the process. This will be the most likely string that was encoded in this manner. Though not certain: just consider removing the highest scoring "answer": you'll always have a high score, but that in itself is not meaningful.

#### Easy enough!

We store the texts as a string by enclosing the block in backticks, an ES6 feature allowing for multiline strings (\`), and then we split the block based on the newline (\\n) character, giving us an array with each element as one of the potential encoded strings. Then, we loop over this array of strings and pass it into guessTheKey.

The return method of guessTheKey is changed slightly to return an array containing both the high score and the highest scoring string `[lastHighScore,highestScoringText]`. Each time we call guessTheKey, we'll score the highestScoringText and highScore so that at the end, we can reover the actual text.

We can find the max score in an array using the ES6 spread operator:
`Math.max(...scores)`. When we find the maximum score, we can use it to index on all the texts that we tested to find the highest-scoring text.

{% highlight javascript %}
function goThroughTheFiles(txts) {
  //takes an array of texts
  let scores=[];
  let texts=[];
  for (var i=0;i<txts.length;i++) {
    let res=guessTheKey(txts[i]);
    scores.push(res[0]);
    texts.push(res[1]);
  }
  console.log('gttf scores:',scores,Math.max(...scores),texts[scores.indexOf(Math.max(...scores))]);
}
{% endhighlight %}

We get the result of `Now that the party is jumping` and a score of 215.223 using our character scoring rubric.

I'm sensing a Vanilla Ice vibe.
