---
layout: post
title:  "Single-byte XOR cipher"
date:   2017-09-14 15:09:06 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 1 Challenge 3](https://cryptopals.com/sets/1/challenges/3)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s1c3)
### [working demo](https://thmsdnnr.github.io/cryptopals/s1c3/index.html)

The challenge? Find the single character this ciphertext has been XOR'd against:
`1b37373331363f78151b7f2b783431333d78397828372d363c78373e783a393b3736`

We have a few helper functions hanging out in the code, and if you've been following along with previous exercises, the code to start will look pretty familiar. First, we've got to break down that ciphertext into its binary representation. Then we'll generate an array of all possible keys, XOR each and every key against every character of the ciphertext, and obtain a string. How to know which string is "best" though, computationally? Per the hint from CryptoPals, we'll generate a scoring system based on letter frequency in the English language and pick the highest-scoring string as that string corresponding to the decryption key.

#### Step 1: Hexadecimal to Binary

Split each hexadecimal into an array of characters, and group into sets of 2.

{% highlight javascript %}
function guessTheKey(cipher) {
  cipher=cipher.split("");
  let a=[];
  for (var i=0;i<cipher.length;i+=2){a.push(cipher[i].concat(cipher[i+1]));}
    a=a.map(e=>parseInt(e,16).toString(2)).map(e=>{
      while (e.length<8) { e="0"+e; }
      return e;
    });
{% endhighlight %}

#### Step 2: Create an array of all possible keys.

A single character (8-bits) can represent 2^8 (256) possibilities, or (0, 255), so we use a FOR loop to generate them all, an array of binary values from "00000000" to "11111111".

{% highlight javascript %}
let keys=[];
for (var i=0;i<256;i++) { keys.push(decToBin(i)); }
{% endhighlight %}

#### Step 3: XOR each key with the ciphertext and score the results

This is the meat of the code. Three nested for loops. Loop through each of our 256 keys. For each key, we loop through every letter in the ciphertext, stored in a. Finally in the innermost for loop, we XOR each bit of each letter in the ciphertext with the key.

We take this XOR result and convert it from its binary to decimal representation, and then we convert the decimal value to the ASCII character with `String.fromCharCode(ASCII_VALUE)`. Finally, we concatenate all these ASCII characters into a string.

We pass these strings into a scoring function (details covered in the next step). We score each text as it is generated. If the score of the text is higher than the last highest score, we update the current "highest-scoring" text, as well as the string that this highest score corresponds to. At the end of the process, `lastHighScore` holds the score of `highestScoringText`, which is our deciphered ciphertext.

{% highlight javascript %}
let texts=[];
let tempText=[];
let intermediateXOR=[];
let scoresAndStrings=[];
let score=0;
//run through each key, XOR every character of the cipher text
for (var i=0;i<keys.length;i++) {
  for (var j=0;j<a.length;j++) {
    for (var k=0;k<a[j].length;k++) {
      intermediateXOR=intermediateXOR.concat(a[j][k]^keys[i][k]);
    }
    tempText.push(String.fromCharCode(parseInt(intermediateXOR.join(""),2).toString(10)));
    intermediateXOR=[];
  }
  tempText=tempText.join("");
  texts.push(tempText);
  score=freqScoreString(tempText);
  //if this text is higher scoring than previous, store it + its score
  if (score>lastHighScore) {
    lastHighScore=score;
    highestScoringText=tempText;
  }
  tempText=[];
  score=0;
}
{% endhighlight %}

#### Step 4: Scoring the Results

We use [good ol' Wikipedia](https://en.wikipedia.org/wiki/Letter_frequency#Relative_frequencies_of_letters_in_the_English_language) to snag a letter frequency chart of characters in the English language and create a dictionary to map each letter to its score. Then to calculate the score, we just loop over each char in the string and sum up the values.

Only a little nuance here: I lowercase the string before scoring so I don't have to have two keys per character (lower and upper) in the dictionary. I also check to see if the key exists before adding the value (there is not frequency data for some letters in my variable). Otherwise you'll add a number to NaN, which will give you...not a number!

{% highlight javascript %}
const letterScores = {
  ' ':15,
  'e':12.702,
  't':9.056,
  'a':8.167,
  'o':7.507,
  'i':6.966,
  'n':6.749,
  's':6.327,
  'h':6.094,
  'r':5.987,
  'd':4.253
  ...
};

function freqScoreString(string) {
  string=string.toLowerCase("");
  let score=0;
  for (var i=0;i<string.length;i++) { if (letterScores[string[i]]) { score+=letterScores[string[i]]; } }
  return score;
}
{% endhighlight %}

#### Step 5: Recovering the decrypted ciphertext and the key

We're almost sorted. What's left? Well, we've got the highestScoringText in that variable, easy, but we also have to figure out the original character used for encryption. Luckily, there's a 1-to-1 mapping between the index of the 256 texts we generated and the index of the 256 keys we generated.

So we can use the array's .indexOf(highestScoringText) to recover the *decimal* representation of the cipher byte. We want the hexadecimal character, so we'll use:
`parseInt(texts.indexOf(highestScoringText),10).toString(16)`

In this case, the decrypted text is: `Cooking MC's like a pound of bacon`.

I won't give away the key. The exercise is left to the reader!

One thing you might have noted about letterScores: the space character is given a score of 15, even higher than the letter 'e'. Why is this? Well, for one, you'd not get the right answer if you didn't give a high score to spaces. It makes sense: an intelligible phrase *is* likely to contain spaces.

Yet this illustrates the difficulty of decrypting more complex encryption. While the human eye could easily read `asentencewithallthewordsputtogether`, this would become very expensive to compute with a brute-force method, even with a single character XOR encryption (where do you insert word spaces? how do you know what are words and what aren't?).

Good enough for a simple example, though. And it was kind of fun, no?
