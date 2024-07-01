<!--yml
category: 未分类
date: 2024-07-01 18:18:11
-->

# A radical Hackage social experiment : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/08/the-radical-hackage-social-experiment/](http://blog.ezyang.com/2010/08/the-radical-hackage-social-experiment/)

*Prologue.* This post is an attempt to solidify some of the thoughts about the upcoming Hackage 2.0 that have been discussed around the Galois lunch table. Note that I have never overseen the emergence of a language into mainstream, so take what I say with a grain of salt. The thesis is that Hackage can revolutionize what it means to program in Haskell if it combines the cathedral (Python), the bazaar (Perl/CPAN), and the wheels of social collaboration (Wikipedia, StackOverflow, Github).

New programming languages are a dime a dozen: one only needs to stroll down the [OSCON Emerging Languages track](http://emerginglangs.com/speakers/) to see why. As programmers, our natural curiosity is directed towards the language itself: “What problems does it solve? What does it look like?” As engineers, we might ask “What is its runtime system?” As computer scientists, we might ask: “What novel research has been incorporated into this language?” When a language solves a problem we can relate to or shows off fancy new technology, our interest is whetted, and we look more closely.

But as the language grows and gains mindshare, as it moves beyond the “emerging” phase and into “emergent”, at some point, *the language stops being important.* Instead, it is the community around the language that takes over: both socially and technically. A community of people and a community of code—the libraries, frameworks, platforms. An engineer asks: “Ok. I need to do X. Is there a library that fills this need?”

The successful languages are the ones that can unambiguously answer, “Yes.” It’s a bit of an obvious statement, really, since the popular languages attract developers who write more libraries which attracts more developers: a positive feedback loop. It’s also not helpful for languages seeking to break into the mainstream.

Tune down the popularity level a little, and then you can see languages defined by the mechanism by which developers can get the functionality they need. Two immediate examples are Python and Perl.

Python has the mantra: “batteries included,” comparing a language without libraries to a fancy piece of technology that doesn’t have batteries: pretty but—at the moment—pretty useless. The [Python documentation](http://www.python.org/about/) boasts about the fact that any piece of basic functionality is only an import away on a vanilla Python install. The Python standard library itself follows a cathedral model: commits are restricted to members of [python-dev](http://www.python.org/dev/committers), a list of about 120 trusted people. Major additions to the standard library, including the [addition of new modules](http://www.python.org/dev/peps/pep-0002/) most go through a [rigorous proposal process](http://www.python.org/dev/peps/) in which they demonstrate that your module is accepted, widely used and will be actively maintained. If a maintainer disappears, python-dev takes stewardship of the module while a new maintainer is found, or deprecates the module if no one is willing to step up to maintain it. This model has lead to over [three hundred relatively high-quality modules](http://docs.python.org/library/) in the standard library.

On the other hand, Perl has adopted the bazaar model with [CPAN](http://www.cpan.org), to the point where the slow release cycle of core Perl has meant that some core modules have been dual-lifed: that is, they exist in both the core and CPAN. Absolutely anyone can upload to CPAN: the result is over 20,000 modules and a resource many Perl developers consider indispensable. Beyond its spartan home interface, there is also [massive testing infrastructure](http://deps.cpantesters.org/) for all of CPAN and a [ratings system](http://cpanratings.perl.org/) (perhaps of dubious utility). CPAN has inspired similar bazaar style repositories across many programming languages (curiously enough, some of the most popular langauges—C and Java—have largely resisted this trend).

It’s a tall order for any language to build up over a hundred trusted committers or a thriving community on the scale of CPAN. But without this very mechanism, the language is dead out of the water. The average engineer would have to rewrite too much functionality for it to be useful as a general purpose language.

Which brings us back to the original point: where does Hackage stand?

The recent results from the [State of the Haskell 2010 survey](http://blog.johantibell.com/2010/08/results-from-state-of-haskell-2010.html) gives voice to the feeling that any Haskell programmer who has attempted to use Hackage has gotten. There are *too many libraries without enough quality.*

How do we fix this? After all, it is all open source made by volunteers: you can’t go around telling people to make their libraries better. Does one increase the set of core modules—that is, the Haskell platform—and the number of core contributors, requiring a rigorous quality review (the Python model)? Or do you let natural evolution take place and add mechanisms for measuring popularity (the Perl model)?

To succeed, I believe Hackage needs to do both. And if it succeeds, I believe that it may become *the* model for growing your standard library.

The cathedral model is the obvious solution to rapidly increase the quality of a small number of packages. Don Stewart has employed this to good effect before: [bytestring](http://hackage.haskell.org/package/bytestring) started off as a hobby project, before the Haskell community realized how important efficiently packed strings were. A “strike team” of experienced Haskellers was assembled and the code was heavily improved, fleshed out and documented, generating several papers in the process. Now bytestring is an extremely well tuned library that is the basis for efficient input and output in Haskell. Don has suggested that we should adopt similar strike teams for the really important pieces of functionality. We can encourage this process by taking libraries that are deemed important into a shared repository that people not the primary maintainer can still help do basic maintenance and bugfixes.

But this process is not scalable. For one, growing a set of trusted maintainers is difficult. The current base libraries are maintained by a very small number of people: one has to wonder how much time the Simons spend maintaining `base` when they could be doing work on GHC. And you can only convince most people to take maintainership of `X` packages before they wise up. (Active maintainership of even a single package can be extremely time consuming.)

[Hackage 2.0](http://cogracenotes.wordpress.com/2010/08/08/hackage-on-sparky/) is directed at facilitating the Bazaar model. Package popularity and reverse dependencies can help a developer figure out whether or not it is something worth using.

But if we consider both developers and package maintainers, we are tackling a complex socio-technical problem, for which we don’t have a good idea what will revolutionize the bazaar. Would a StackOverflow style reputation system encourage maintainers to polish their documentation? Would a Wikipedian culture of rewarding contributors with increased privileges help select the group of trusted stewards? Would the ability to fork any package instantly ala GitHub help us get over our obsession with official packages? Most of these ideas have not been attempted with a system so integral to the fabric of a programming language, and we have no way of telling if they will work or not without implementing them!

I am cautiously optimistic that we are at the cusp of a major transformation of what Hackage represents to the Haskell community. But to make this happen, we need your help. Vive la révolution!

*Credit.* Most of these ideas are not mine. I just wrote them down. Don Stewart, in particular, has been thinking a lot about this problem.