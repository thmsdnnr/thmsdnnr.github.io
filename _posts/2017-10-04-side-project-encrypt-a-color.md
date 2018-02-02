---
layout: post
title:  "Encrypt A Color!"
date:   2017-10-05 14:40:04 -0500
categories: tutorials javascript metacrypto
---

### [code repo](https://github.com/thmsdnnr/cryptopals/tree/master/s3c17)
### [demo](https://thmsdnnr.github.io/cryptopals/s3c17/)

#### Step 0: Why Not Encrypt A Color?

We've been doing a lot of heavy work, so let's use that good ol' `aesEncryptECB` for something a bit more lighthearted and visual.

I was wondering the other day: what would it look like to encrypt a color? Naturally the question is a bit of nonsense, but also it's not, not quite. Letters in words are represented as hex values, and we
