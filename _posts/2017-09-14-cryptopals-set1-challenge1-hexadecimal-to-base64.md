---
layout: post
title:  "CRYPTOPALS Set 1 Challenge 1: Convert hex to base64!"
date:   2017-09-14 10:22:06 -0500
categories: cryptopals JavaScript
---

https://cryptopals.com/sets/1/challenges/1

###Tutorial:
Here's a [good blog post]: https://blogs.oracle.com/rammenon/base64-explained  from Oracle to understand the maths behind the conversion:

First, we have to split the hex (Base-16) string into groups of two characters each.  We do some simple length and regexp validation and display error messages if the input string contains an odd number of characters or invalid characters.

```javascript
if (hStr.match(/[^0-9a-f]/gi)%2!==0) { return 'Hex string can only contain characters 0-9 and A-F.'; }
hStr=hStr.split("");
if (hStr.length%2!==0) { return 'Hex string must contain an even number of characters'; }
```

The meat of the conversion happens in this function that calls helper functions (explained in depth below).
Note that this is an extremely verbose way of performing conversion that shows all the steps.
The process can be abbreviated with bitwise operators.

Also, where you see code like this: `decBin.innerHTML=JSON.stringify(arr); //2. Hex Bytes to Decimal`, that's me breaking out variable contents in intermediate steps to display on the page. I added a simple event listener on the text box to update all these variables each time the text is modified, so that you can see the different steps of code running and their effects in real-time.

For a small project like this, manually updating the view is super straightforward and makes sense, but it's the fundamental problem that a DOM-diffing view like React sets out to solve. Rather than manually figuring out what changed and updating it yourself, React will "do it for you" when state changes.

```javascript
function calcB64(hStr) {
  let arr=[];
  for (var i=0;i<hStr.length;i+=2) { arr.push([hStr[i],hStr[i+1]].join("")); } //base-16 chars
  hBytes.innerHTML=JSON.stringify(arr); //1. Hexstring to Hex Bytes
  arr=arr.map(e=>hexToDec(e)); //base-16 to base-10
  decBin.innerHTML=JSON.stringify(arr); //2. Hex Bytes to Decimal
  arr=arr.map(e=>decToBin(e)); //base-10 to base-2
  hexBinary.innerHTML=JSON.stringify(arr); //3. Decimal to Binary
  arr=binTo64(arr); //3x8 (24-bit) string into 4x6 (24-bit) strings and pad out with two bits
  return arr;
}
```

We verify the hexstring is valid, split it into two-byte groups, map the groups to base-10, map the base 10 groups to base-2, combine adjacent 8-bit base-2 groups to form 24-bit base-2 groups (handling padding when there are fewer than 24-bits in the input stream or a # of bits not divisible by 24), split each of these 3x8-bit base-2 groups into 4x6-bit base-2 groups, zero pad each 6 bit group so they are now a 4x8-bit group, convert each number in this new group back to decimal, and then we map this decimal 1->1 with the base64 character set. Finally, if padding was required, this is appended to the end of the string, and the converted base-64 string is returned! Hooray!

Here are the individual steps broken down:

####Step 1: Hexstring to Hex Bytes
Convert the string to an array of individual characters.
Create a new array with two-character groups.

```javascript
hStr=hStr.split("");
let arr=[];
for (var i=0;i<hStr.length;i+=2) { arr.push([hStr[i],hStr[i+1]]); }
```

####2. Hex Bytes to Decimal
We map the two-character groups with .toUpperCase() so that we only have to have upper-case letters in our map.

The nMap array maps array index -> hexadecimal value, 0 through 15.

We break the hexadecimal into two and convert into hexadecimal (first character decimal equivalent times 16^1 plus second character decimal equivalent times 16^0).

We return the decimal.

```javascript
hex=hex.split("");
hex=hex.map(e=>e.toUpperCase());
if (hex.length!==2) { console.error('wrong string length',hex,hex.length); }
const nMap=['0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'];
let dec;
dec=nMap.indexOf(hex[0])*16+nMap.indexOf(hex[1]);
return dec;
```

####3. Decimal to Binary
Now that we have the decimal value of each hex byte we need to convert to binary, base-2.
We do this by iterating through powers of 2 for a byte and, if the value we want to represent is greater than or equal to the power of two at this position, we flip the bit to one, subtract this value from the value we wish to represent, and continue.

Clearly we can represent between 0 (00000000) and 255 (11111111) here.

Then we return the binary representation of the decimal as a string, e.g., "11111111".

```javascript
let binArr=[0,0,0,0,0,0,0,0];
for (var i=8;i>=0;i--) {
  if (dec>=Math.pow(2,i)) {
    binArr[7-i]=1;
    dec-=Math.pow(2,i);
  }
}
return binArr.join("");
```

####4, 5, & 6. Binary Octets to 24-bit Groups, 24-bit Groups to 6-bit Groups, zero-Pad 6-bit groups
Now that we have an array of base-2 binary octets, we need to convert its elements into 24-bit groups by merging every three adjacent bytes.

Then we turn each 24-bit group into 4 6-bit groups (3 octets turn into 4 encoded characters in base-64).

Each 6-bit group is zero-padded at the front with "00".

There's a bit of a catch, because there are cases where the mapping from 3x24-bit groups to 4x6-bit groups does not map one-to-one. There are two cases:

1. input string contains fewer than 24 bits
2. the input string contains > 24 bits and is not an even multiple of 24, so that bits are left over as part of an incomplete group.

In either case, we have to add padding. We treat this last group that may need padding separately, outside of the main loop. It's called `lastSixSlice` in the code below.

First, we loop over everything but the last group of three 8-bytes. This loop will only happen if there are three or more bytes in the input stream. We split the 3x8-byte group into 4x6-byte groups and zero pad each 6-byte group with two leading bits. We keep track of the last index if this loop runs in lastI, since we'll need it to figure out where to start in the array find the "last" octets.

```javascript
for (var i=0;i<dec.length-2;i+=3) { jBin.push(dec[i].concat(dec[i+1],dec[i+2])); lastI=i+3;}

for (var i=0;i<jBin.length;i++) {
  sixSlice=[jBin[i].slice(0,6),jBin[i].slice(6,12),jBin[i].slice(12,18),jBin[i].slice(18,24)];
  sixSlice=sixSlice.map(e=>"00"+e); //pad with two leading zero bits before binary conversion
  b64Res=b64Res.concat(sixSlice);
}
```

Now for the tricky part. `numLastOctets` holds the number of leftover octets. If it's zero, we're good, and we don't need any padding, so we throw all the padding logic in an if block to check this.

The first line of code in this block sets the number of remaining octets. If the remainder of the # of octets when divided by 3 isn't zero, there will be leftovers to handle.

Then we follow the padding algorithm for leftovers which is:

1. distribute the available bits from the final octets into the 4 6-bit containers, left-to-right
2. when you run out of bits, if a container is partially filled, pad it out to the right with zeroes until it is 6 bits long
3. if a container is completely empty, use the "=" character for padding

The only question is, where to find the last octets. From cases 1 and 2 above, are they:

1. at the beginning of a short string (1. input string contains fewer than 24 bits)? lastI will have been set, so we offset the array index by this much.

2. at the end of a string of complete octets (2. the input string contains > 24 bits and is not an even multiple of 24)? lastI will NOT have been set, so we so we start at the beginning.

```javascript
numLastOctets=dec.length%3;
if (numLastOctets) {
  let lastOct=[];
  for (var i=0;i<numLastOctets;i++) { lastI ? lastOct.push(dec[lastI+i]) : lastOct.push(dec[i]); }
  lastOct=lastOct.join("");
  for (var i=0;i<lastOct.length;i+=6) {
    let cSlice=lastOct.slice(i,i+6);
    if (cSlice.length===6) { lastSixSlice.push(b64[binToDec("00"+cSlice)]); }
    else if (cSlice.length>0) { //pad end until length is 6
      while (cSlice.length<6) { cSlice=cSlice+="0"; }
      lastSixSlice.push(b64[binToDec("00"+cSlice)]);
    }
  }
  while(lastSixSlice.length<4) { lastSixSlice=lastSixSlice.concat('='); }
  lastSixSlice=lastSixSlice.join("");
}
```

####7. Convert to Base-64
Once we have the binary representation of the base-64 characters, all we have to do is translate it back to base-10, and then use the map defined up top to obtain the base-64 representation of the hex string. We add on the string containing the last slice that we calculated separately (since we saw from Step 6 that it's not a simple 1-to-1 map and requires special handling, depending on the number of input bits).

```javascript
const b64="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/".split(""); //global b64 map at top of code

let b64Arr=[];
for (var i=0;i<b64Res.length;i++) { b64Arr.push(binToDec(b64Res[i])); }
return b64Arr.map(e=>b64[e]).join("")+lastSixSlice;
```
----

###TODO:
[X] Add padding (currently assume that hex string can be broken into even number of bytes)
[ ] Add tests+validation with Mocha/Chai
