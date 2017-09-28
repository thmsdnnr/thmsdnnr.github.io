---
layout: post
title:  "ECB Cut and Paste"
date:   2017-09-27 21:00:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 2 Challenge 13](https://cryptopals.com/sets/2/challenges/13)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s2c13)
### [demo](https://thmsdnnr.github.io/cryptopals/s2c13/)

#### Step 1: Don't Use ECB

Really. Just don't. This challenge has us make a few functions to simulate a user object in a database, stored with an e-mail, user ID, and a role. We write a parser `keyValueParse()` to convert a string like `email=foo@bar.com&uid=10&role=user` into an object like `{'email': 'foo@bar.com', 'uid':10, 'role':'user'}` and a function `keyValueObjToString` to convert the string back to an object. Simple mapping and dictionary stuff.

{% highlight javascript %}
const isAmpEql = (t) => t.match(/\=\&/) ? true : false;
const keyValueObjToString = (obj) => Object.keys(obj).map(k=>`${k}=${obj[k]}`).join("&");

function keyValueParse(top) {
  let amps=top.split("&");
  let equals=amps.map(e=>e.split("="));
  if (!amps.length||equals.some(e=>isAmpEql(e.join(""))||e.length!==2)) { throw new Error('Invalid. Keys and values cannot contain metacharacters.'); }
  let dict={};
  equals.forEach(v=>dict[v[0]]=v[1]);
  return dict;
}
{% endhighlight %}

Then we write a function to encrypt the profile, and one final function that generates a new profile given an e-mail. Using the output of this function `profileFor`, we are challenged to create a ciphertext in which the `role=admin`.

{% highlight javascript %}
const profileFor = (e) => keyValueObjToString(keyValueParse(`email=${e}&uid=10&role=user`));
const decryptAndParse = (eProf) => chopPKCS7(aesDecryptECB(eProf,randomKey,16).map(hexToText)).join("");
function encryptProf(pText) {
  pText=PKCS7(txtToHex(pText).map(e=>hexPad(e)),16);
  return aesEncryptECB(pText,randomKey,16);
}
{% endhighlight %}

#### Step 2: Why You Shouldn't Use ECB

As we've seen before, identical plaintexts yield identical ciphertext outputs. All we've got to do in order to make our `role=admin` ciphertext is craft an input to `profileFor` that will strategically break the parts we'd like to recombine into separate 16 byte blocks. We want a block that has `email=`, a block that has `foo@bar.com`, a block that has `&uid=10&role=`, and a block that has `admin`. There's a lot of ways you can do this by padding with spacing, but here's how I did it:

{% highlight javascript %}
profileFor("          foo@bar.com     admin           AAAAAAAAAAAAA      ");
{% endhighlight %}

gives me the following (broken out into 16-byte blocks):
  * A. email=          
  * B. foo@bar.com     
  * C. admin           
  * D. AAAAAAAAAAAAA   
  * E.     &uid=10&role=

So all that we have to do is to combine A+B+E+C into a ciphertext and decrypt it.

{% highlight javascript %}
E=encryptProf(PO);
let newCipher=E.slice(0,32).concat(E.slice(64,80),E.slice(32,48)); //A+B, E, C
console.log(decryptAndParse(newCipher));
{% endhighlight %}

Huzzah!
```
original padded profile:
email= foo@bar.com admin AAAAAAAAAAAAA &uid=10&role=user
manipulated ciphertext:
be53b1cdc24102eadb565a260a457877bfe37794e1df771fc3caceb8c5d77c8c172e72efcc5ede1c26edb3c9ef39de9e1144aeb403674bae926509af651da098
decrypted manipulation:
email= foo@bar.com &uid=10&role=admin
```

And we're done. Don't use ECB.
