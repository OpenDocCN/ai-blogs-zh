<!--yml
category: 未分类
date: 2024-07-01 18:17:27
-->

# OfflineIMAP sucks : ezyang’s blog

> 来源：[http://blog.ezyang.com/2012/08/offlineimap-sucks/](http://blog.ezyang.com/2012/08/offlineimap-sucks/)

I am going to share a dirty little secret with you, a secret that only someone who uses and hacks on OfflineIMAP could reasonably know: OfflineIMAP sucks. Of course, you can still use software that sucks (I do all the time), but it’s useful to know what some of its deficiencies are, so that you can decide if you’re willing to put up with the suckage. So why does OfflineIMAP suck?

> This is not really a constructive post. If I were actually trying to be constructive, I would go and fix all of these problems. But all of these are big problems that require substantial amounts of effort to fix... and unfortunately I don’t care about this software enough.

### Project health is anemic

The original author, [John Goerzen](http://www.complete.org/JohnGoerzen), has moved on to greener, more Haskell-like pastures, and the current maintainership is having difficulty finding the time and expertise to do proper upkeep on the software. Here is [one of the most recent calls for maintainers](http://comments.gmane.org/gmane.mail.imap.offlineimap.general/5754), as both of the two co-maintainers who were maintaining OfflineIMAP have failed to have enough free time to properly keep track of all submitted patches. There still seem to be enough people with a vested interest in seeing OfflineIMAP not bitrot that the project should continue to keep working for the foreseeable future, but one should not expect any dramatic new features or intensive work to be carried out on the codebase.

### Nearly no tests

For most of OfflineIMAP’s history, there were no tests. While there is now a dinky little test suite, it has nowhere near the coverage that you would want out of such a data-critical program. Developers are not in the habit of adding new regression tests when they fix bugs in OfflineIMAP. But perhaps most perniciously, there is no infrastructure for testing OfflineIMAP against as wide a range of IMAP servers as possible. Here are where the really *bad* bugs can show up, and the project has none of the relevant infrastructure.

### Over-reliance on UIDs

OfflineIMAP uses UIDs as its sole basis for determining whether or not two messages correspond to each other. This works almost most of the time, except when it doesn’t. When it doesn’t, you’re in for a world of hurt. OfflineIMAP does not support doing consistency checks with the `Message-Id` header or the checksum of the file, and it’s `X-OfflineIMAP` hack for servers that don’t support `UIDPLUS` ought to be taken out back and shot. To it’s credit, however, it has accreted most of the special casing that makes it work properly in all of the weird cases that show up when you have UIDs.

### Poor space complexity

The memory usage of OfflineIMAP is linear with the number of messages in your inbox. For large mailboxes, this effectively means loading hundreds of thousands of elements into a set and doing expensive operations on it (OfflineIMAP consistently pegs my CPU when I run it). OfflineIMAP should be able to run in constant space, but zero algorithmic thought has been put into this problem space. It also has an extremely stupid default status folder implementation (think repeatedly writing 100MB to disk for *every single file you upload*), though you can fix that fairly easily by setting `status_backend = sqlite`. Why is it not default? Because it’s still experimental. Hm...

### Unoptimized critical path

OfflineIMAP was never really designed for speed. This shows up in the synchronization time it takes, even in the common cases of no changes or just downloading a few messages. If one’s goal is to download your new messages as quickly as possible, a lot of adjustments could be made, including reducing the number of IMAP commands (esp. redundant selects and expunges), reducing the number of times we touch the filesystem, asynchronous filesystem access, not loading the entirety of a downloaded message in memory, etc. A corollary is that OfflineIMAP doesn’t really seem to understand what data it is allowed to lose, and what data it must fsync before carrying on to the next operation: “safety” operations are merely sprinkled through the code without any well-defined discipline. Oh, and how about some inotify?

### Brain-dead IMAP library

OK, this one is not really OfflineIMAP’s fault, but `imaplib2` really doesn’t protect you from the pointy-edged bits of the IMAP protocol (and how it is implemented in the real-world) at all. You have to do it all yourself. This is dumb, and a recipe for disaster when you forget to check UIDVALIDITY in that new IMAP code you were writing. Additionally, it encodes almost no knowledge of the IMAP RFC, with respect to responses to commands. Here is one place where some more type safety would really come in handy: it would help force people to think about all of the error cases and all of the data that could occur when handling any given command.

### Algorithmica obscura

OfflineIMAP has a fair bit of debugging output and UI updating code interspersed throughout its core algorithms, and the overall effect is that it’s really hard to tell what the overall shape of the algorithm being employed is. This is not good if the algorithm is kind of subtle, and relies on some global properties of the entire execution to ensure its correctness. There is far too much boilerplate.

### Conclusion

In conclusion, if you would like to use OfflineIMAP on a well-behaved, popular, open-source IMAP server which a maintainer also happens to use with a relatively small number of messages in your INBOX and are willing to put up with OfflineIMAP being an immutable black box that consumes some non-zero amount of time synchronizing in a wholly mysterious way, and never want to hack on OfflineIMAP, there is no finer choice. For everyone else, well, good luck! Maybe it will work out for you! (It mostly does for me.)