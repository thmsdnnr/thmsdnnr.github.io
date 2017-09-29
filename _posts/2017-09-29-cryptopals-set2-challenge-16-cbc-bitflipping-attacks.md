---
layout: post
title:  "CBC Bitflipping Attacks"
date:   2017-09-29 13:24:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 2 Challenge 16](https://cryptopals.com/sets/2/challenges/16)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s2c16)
### [demo](https://thmsdnnr.github.io/cryptopals/s2c16/)

#### Step 1: Flipping Bits, Wearing Knits

In [CBC decryption](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_Block_Chaining_.28CBC.29), the plaintext of each block's post-decryption is XOR'd with the previous block's ciphertext. Altering ciphertext block `N-1` (C<sub>n-1</sub>) thus effects the `N`th block of resulting plaintext (P<sub>n</sub>). CBC cares not whether a message has been tampered with. It has no [MAC](https://en.wikipedia.org/wiki/Message_authentication_code). We can XOR arbitrary bytes into the ciphertext and have them appear in the plaintext under two conditions:

* We know the plaintext
* We know the ciphertext

#### Step 2: But How Can We Know the Plaintext?

Well, if there's a user component, the plaintext can be whatever we'd like it to be. Lots of `A`s.

The challenge has us write some basic one-liners to escape and de-escape the special characters `;` and `=` (to prevent the trivial solution of just entering these directly into the user-supplied data).

{% highlight javascript %}
const enc={';':'%3B','=':'%3D'};
const decr={'%3B':';','%3D':'='};
const quoteOut = (txt) => txt.replace(/(\;|\=)/g,($1)=>enc[$1]);
const quoteIn = (esc) => esc.replace(/(\%3B|\%3D)/g,($1)=>decr[$1]);
{% endhighlight %}

We write a random encryption function that prepends the escaped string `comment1=cooking%20MCs;userdata=` to user-supplied data, appends the escaped `;comment2=%20like%20a%20pound%20of%20bacon`, PKCS#7 pads, and encrypts under a random key and IV using CBC.

{% highlight javascript %}
function randomEncrypt(pText) {
  pText=txtToHex(quoteOut(pText)).map(e=>hexPad(e));
  let input=PKCS7(pre.concat(pText,post),16);
  return aesEncryptCBC(input,randomKey,randomIV,16);
}
{% endhighlight %}

We write a function to decrypt and unescape the special characters.

{% highlight javascript %}
const randomDecrypt = (c) => quoteIn(aesDecryptCBC(c,randomKey,randomIV,16).map(hexToText).join(""));
{% endhighlight %}

We're then challenged to manipulate the ciphertext results of the encryption function to produce a ciphertext that decrypts to contain the string `;admin=true;`.

#### Step 3: Twiddles and Bits (Theory)

You can write to the plaintext in contiguous blocks of `blockSize` characters. Writing to block P<sub>n+1</sub> requires modifying block C<sub>n-1</sub> and creates gibberish in P<sub>n</sub>, since the error introduced into C<sub>n-1</sub> propagates one block forward to C<sub>n</sub>. If you'd like to write more than a `blockSize` worth, write to every other plaintext block, so long as you do not care about intervening gibberish plaintext blocks.

What and how to write? Think about the decryption process (`n==block, i==block byte index`):

C<sub>n-1</sub>[i] XOR C<sub>n</sub>[i] (post-decryption) = P<sub>n</sub>[i].

We want to change P<sub>n</sub>[i] to a given value, and we need to know what to change C<sub>n-1</sub>[i] to in order to do this. So just solve for it.

C<sub>n-1</sub>[i] = P<sub>n</sub>[i] XOR C<sub>n</sub>[i] (post-decryption)

* P<sub>n</sub>[i] is just the current plaintext value of the block we wish to modify
* C<sub>n</sub>[i] (post-decryption) is the value we wish the plaintext to have
* C<sub>n-1</sub>[i] is the byte we flip (XOR-ing in (P<sub>n</sub>[i] XOR C<sub>n</sub>[i])) in the ciphertext

Note the special case when `n=1`, and the previous ciphertext block is the CBC [initialization vector](https://en.wikipedia.org/wiki/Initialization_vector).

Here's one more visual, in case ASCII does it for you better:

```
Plaintext
|comment1%3Dcooki||ng%20MCs%3Buserd|
     Block 1           Block 2

Ciphertext
|----------------||----------------|
     Block 1           Block 2
```

Let's say I want to flip 'n' in Block 2 to 'Z'.

* P<sub>n</sub>[0] = 'n' (0x6e)
* C<sub>n</sub>[0] (post-decryption) is the value we wish the plaintext to have Z (0x5A)
* C<sub>n-1</sub>[0] is the byte we need to flip in the ciphertext

0x63 XOR 0x5A = 0x39. Then just set byte 1 of block 1 of the ciphertext to ^=(0x39). The first block of plaintext in the decryption output is jumbled, but the second block reads `Zg%20MCs%3Buserd`. Zing!

#### Step 4: Twiddles and Bits (Praxis)

We know that the blocks of plaintext (chunked 16 byte sections) look like this when userdata is `''`:

```
comment1%3Dcooki
ng%20MCs%3Buserd
ata%3D%3Bcomment
2%3D%20like%20a%
20pound%20of%20b
acon
```

We know that we want to have a `%3B` at the end of our string. But if we put it in ourselves, we'll be over 16 characters, which is the limitation for contiguous character writing with `blockSize==16`. So I'll use the existing `%3B`, before 'comment', padding out `userdata` with 25 `0`s.

`randomEncrypt('0000000000000000000000000')` gives us this:

```
comment1%3Dcooki
ng%20MCs%3Buserd
ata%3D0000000000
000000000000000%
3Bcomment2%3D%20
like%20a%20pound
%20of%20bacon
```

When we decrypt, we map the escaped characters `;` and `=` back from their escaped forms, so we should really make the replacement of `%3Badmin%3Dtrue%3B`. I'll calculate my array of replacements and XOR each one with our padding character ('0' == 0x30).

```
'%3Badmin%3Dtrue'.split("").map(e=>parseInt(e.charCodeAt(),10).toString(16)).map(e=>hexXOR(e,'30'));
=> ["15", "3", "72", "51", "54", "5d", "59", "5e", "15", "3", "74", "44", "42", "45", "55"]
```

We can XOR in our 15 hex characters with ciphertext block 3, and the result will end up in plaintext block 4, followed by the `%3B` we borrowed.

{% highlight javascript %}
for (var i=0;i<theReplacements.length;i++) {
  e[32+i]=hexXOR(e[32+i],theReplacements[i]);
}
randomDecrypt(e);
{% endhighlight %}

```comment1=cooking%20MCs;userdo|=¦1^<ÉTv¤Ï;admin=true;comment2=%20like%20a%20pound%20of%20bacon```

This replacement occurs without any direct knowledge of the encryption key or the initialization vector, simply a manipulation of the ciphertext output. What have we learned today? Use a MAC!

The challenge to write a function that lets you XOR arbitrary lengths of text into the ciphertext is left as an exercise for the reader. Though perhaps not a valuable one, what with the interleaved mangled plaintext block every `blockLength` chars.
