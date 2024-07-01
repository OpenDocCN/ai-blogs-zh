<!--yml
category: 未分类
date: 2024-07-01 18:17:24
-->

# Functional Encryption : ezyang’s blog

> 来源：[http://blog.ezyang.com/2012/11/functional-encryption/](http://blog.ezyang.com/2012/11/functional-encryption/)

## Functional Encryption

Joe Zimmerman recently shared with me a cool new way of thinking about various encryption schemes called *functional encryption.* It’s expounded upon in more depth in a very accessible [recent paper by Dan Boneh et al.](http://eprint.iacr.org/2010/543.pdf). I’ve reproduced the first paragraph of the abstract below:

> We initiate the formal study of functional encryption by giving precise definitions of the concept and its security. Roughly speaking, functional encryption supports restricted secret keys that enable a key holder to learn a specific function of encrypted data, but learn nothing else about the data. For example, given an encrypted program the secret key may enable the key holder to learn the output of the program on a specific input without learning anything else about the program.

Quite notably, functional encryption generalizes many existing encryption schemes, including [public-key encryption](http://en.wikipedia.org/wiki/Public-key_cryptography), [identity-based encryption](http://en.wikipedia.org/wiki/ID-based_encryption) and [homomorphic encryption](http://en.wikipedia.org/wiki/Homomorphic_encryption). Unfortunately, there are some impossibility results for functional encryption in general in certain models of security (the linked paper has an impossibility result for the simulation model.) There’s no Wikipedia page for [functional encryption](http://en.wikipedia.org/w/index.php?title=Functional_encryption&action=edit&redlink=1) yet; maybe you could write it!

*Apropos of nothing,* a math PhD friend of mine recently asked me, “So, do you think RSA works?” I said, “No, but probably no one knows how to break it at the moment.” I then asked him why the question, and he mentioned he was taking a class on cryptography, and given all of the assumptions, he was surprised any of it worked at all. To which I replied, “Yep, that sounds about right.”