---
layout: post
title:  "Let's Implement Cipher Block Chaining!"
date:   2017-09-24 10:20:04 -0500
categories: tutorials javascript cryptopals
---

### [Cryptopals Set 2 Challenge 10](https://cryptopals.com/sets/2/challenges/10)
### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s2c10)
### [working demo](https://thmsdnnr.github.io/cryptopals/s2c10/index.html)

#### Step 1: How Is CBC Different from ECB?

We already implemented AES-ECB ("electronic codebook"). The primary weakness of ECB-mode of encryption, and why it should never be used really, is that identical plaintext blocks produce identical outputs. You saw this in [Challenge 8](https://thmsdnnr.github.io/tutorials/javascript/cryptopals/2017/09/22/cryptopals-set1-challenge-8-detecting-aes-in-ecb-mode.html). We're able to detect the use of AES-ECB by looking for repeated ciphertext blocks. These blocks are repeated because of repetitions in the plaintext.

CBC produces different output blocks, even for identical plaintexts. This is done by XOR-ing together adjacent blocks. During encryption, each plaintext block is XOR'd with the previous ciphertext block result before it is encrypted itself. Since the first block has no previous ciphertext block, an artificial "Block Zero" called an Initialization Vector is used for this first XOR.

This IV is also needed in the decryption process.

[These Wikipedia diagrams](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_Block_Chaining_.28CBC.29) are particularly helpful in visualizing the process. At first I didn't understand what the "block cipher encryption" and "block cipher decryption" boxes meant. In the case of adapting ECB code to CBC, these blocks just mean "then encrypt with CBC". I refactored my existing ECB code. Before I had a for loop that chopped the blocks individually and then ran them through the encryption/decryption process in a massive loop. I created an inner function that processes one block at a time, and then the outer function that feeds it 16-byte blocks. This way it's very easy to implement the additional XOR process required in CBC.

Here's what the new encryption function looks like.

{% highlight javascript %}
function aesEncryptCBC(plainText, key, IV, numBits) {
  if (!plainText||plainText.length%16!==0) { throw new Error('Block size is not an even multiple of 16.'); }
  else if (!key||key.length!==numBits) { throw new Error(`Key size is not an even multiple of ${numBits}.`); }
  else if (!IV) { throw new Error(`No initialization vector provided.`); }
  const numRounds = { '16':9,'24':11,'32':13 };
  if (!numRounds[numBits]) { return null; }
  let totalRounds = numRounds[numBits];
  let expKey=generateExpandedKey(key,numBits);
  let currentState=[];
    for (var i=0;i<plainText.length;i+=16) { //block feeder
      let cur=plainText.slice(i,i+16);
      if (i===0) { cur=addRoundKey(cur, IV); }
      else { cur=addRoundKey(cur, currentState[(i/16)-1]); }
      currentState.push(processBlock(cur));
    }
    function processBlock(block) {
      let curKey=expKey.slice(0,16);
      block=addRoundKey(block,curKey);
      for (var i=0;i<totalRounds+1;i++) {
        curKey=expKey.slice(i*16+16,i*16+2*16);
        if (i<totalRounds) { block=addRoundKey(mixColumns(sRow(bSub(block,true),true),true),curKey); }
        else { block=addRoundKey(mToBytes(sRow(bSub(block,true),true)),curKey); }
      }
      return block;
    }
    return flattenArray(currentState).map(e=>hexPad(e));
  }
{% endhighlight %}

The only addition to the parameters is the Initialization Vector. If it's the first block, we XOR it (which is just the good old addRoundKey function again) with the Initialization Vector. If it's block 1 or later, we have a previous ciphertext block to XOR with, so we grab that from `currentState[(i/16)-1])`.

Decryption is very similar (an exercise for the reader!). The only difference is the ordering. You take the current block post-decryption rather than pre-decryption, and you XOR it with the previous block *before* that block has been decrypted rather than after it has been encrypted. Just look at the picture. Words make it complicated.

It's possible to validate your results using openssl on the command line as well as Ruby. Store the plaintext value in a text file after the -in flag, set an output filename after the -out flag, pass the key after -K in padded hexadecimal, and then the initialization vector after -iv in padded hexadecimal (0x00 => 00, *not* 0). The -a flag converts the output to base64.

```
openssl enc -aes-128-cbc -in sec.txt -out put.txt -K '5468617473206D79204B756E67204675' -iv '000102030405060708090a0b0c0d0e0f' -a
```

Note that the IV must be padded. Otherwise the Initialization Vector gets treated as half-hex characters and half zeroes, if you pad with hex characters that are only single digits. That is, these two commands would give identical results:

```
openssl enc -aes-128-cbc -in sec.txt -out put.txt -K '5468617473206D79204B756E67204675' -iv '000102030405060708090a0b0c0d0e0f' -a
openssl enc -aes-128-cbc -in sec.txt -out put.txt -K '5468617473206D79204B756E67204675' -iv '0123456789abcdef' -a
```

This can be really confusing at first when you're trying to validate results. If you're using Ruby, you could do this:

{% highlight ruby %}
require 'openssl'
c=OpenSSL::Cipher.new('aes-128-cbc')
c.encrypt
c.key='Thats my Kung Fu'
c.iv=['000102030405060708090a0b0c0d0e0f'].pack("H*") #have to pack IV as hexadecimal data!
result=c.update('testing') << c.final
puts result.unpack("H*")
{% endhighlight %}

Once you code the core of AES, it's easy to switch the mode between ECB and the more secure CBC.
