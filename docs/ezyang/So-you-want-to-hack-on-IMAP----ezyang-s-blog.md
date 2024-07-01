<!--yml
category: 未分类
date: 2024-07-01 18:17:27
-->

# So you want to hack on IMAP… : ezyang’s blog

> 来源：[http://blog.ezyang.com/2012/08/so-you-want-to-hack-on-imap/](http://blog.ezyang.com/2012/08/so-you-want-to-hack-on-imap/)

## So you want to hack on IMAP…

(Last IMAP themed post for a while, I promise!)

Well, first off, you’re horribly misinformed: you do not *actually* want to hack on IMAP. But supposing, for some masochistic reason, you need to dig in the guts of your mail synchronizer and fix a bug or add some features. There are a few useful things to know before you start your journey...

*   Read your RFCs. [RFC 3501](http://tools.ietf.org/html/rfc3501) is the actual specification, while [RFC 2683](http://tools.ietf.org/html/rfc2683) gives a lot of helpful tips for working around the hairy bits of the many IMAP servers out there in practice. You should also know about the UIDPLUS extension, [RFC 4315](http://tools.ietf.org/html/rfc4315), which is fairly well supported and makes a client implementor’s life *a lot* easier.
*   IMAP is fortunately a text-based protocol, so you can and should play around with it on the command line. A great tool to use for this is `imtest`, which has all sorts of fancy features such as SASL authentication. (Don’t forget to `rlwrap` it!) Make sure you prefix your commands with an identifier (`UID` is a valid identifier, so typing `UID FETCH ...` will *not* do what you want.)
*   It is generally a better idea to use UIDs over sequence numbers, since they are more stable, but be careful: as per the specification, `UID` prefixed commands *never fail*, so you will need to check untagged data in the response to see if anything actually happened. (If you have a shitty IMAP library, it may not clear out untagged data between requests, so watch out for stale data!) Oh, and look up `UIDVALIDITY`.
*   There exist a lot of software that interfaces with IMAP, all of which has accreted special cases for buggy IMAP servers over the years. It is well worth sourcediving a few to get a sense for what kinds of things you will need to handle.