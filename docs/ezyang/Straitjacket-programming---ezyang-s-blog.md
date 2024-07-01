<!--yml
category: 未分类
date: 2024-07-01 18:18:25
-->

# Straitjacket programming : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/03/straitjacket-programming/](http://blog.ezyang.com/2010/03/straitjacket-programming/)

The importance of constraint is one well known to those who embark on creative endeavors. Tell someone, "you can do anything you want: anything at all," and they will blank, paralyzed by the infinite possibility. Artists welcome constraint. Writers like the constraint of a sonnet because it imposes form and gives a place to start; roleplaying groups like the constraint of a campaign setting because it imposes rules and sets the scene for the story to be told; jazz musicians like the constraint of the chords underlying an improvisation because it keeps the soloist anchored to the source tune and suggests ideas for the melody.

However, many programmers don't the like the constraint of a type system. "The static type system doesn't let me do what I want to." "I needed to write four classes for what would have been two lines of Python!" "What? I can't do that? Why not?" For them, it's like a straightjacket. How does anyone ever get *anything* done when constraint ties you up?

I beg to differ. *Accept* the straightjacket. The things it will let you do... are *surprising.*

* * *

The straitjacket was historically used as an implement to prevent dangerous individuals from harming themselves and others. Programmers are not quite mental asylum inmates, though at a glance it may seem that we've been trying to reduce the ways for us to hurt ourselves. But such changes have often brought with them benefits, and many have eagerly traded away pointers and manual memory management for increased expressiveness.

Static types, however, are still a pain point for many people, and Haskell is an unusually constrained language due to its type system. An overenthusiastic user of Haskell's type system might exclaim, "after I made it typecheck, it just worked!" Of course, this statement is not actually true; there is a certain essential complexity to classes of algorithms that mean the type system won't catch the fact that you seeded your hash function with the wrong magic number.

But not all code is like this. A lot of code is just plain *boring*. It's the code that generates your website, or logs your errors; it's the code that serves as the glue for your build infrastructure, or it shuffles data from a file into an in-memory representation into a database. It's the code is foundational; it is the code that lets you express simple ideas simply. When you look at the development of this code, the errors being made are very simple mental typos, they're the ones that take a total of fifteen seconds to track down and fix once they manifest, but if rolled up in the time it takes to run your test suite or, dare I say it, *manually* test, quickly ticks to the minutes. A fast static type checker saves you so much pain, whether or not it is a Haskell compiler or `pylint -e`. The difference is that `pylint -e` is optional; there is no guarantee that any given Python project will play nicely with it, and it is frequently wrong. The Haskell compiler is not.

This is a specific manifestation of a more general phenomenon: types reduce the number of ways things can go wrong. This applies for complicated code too; `(a -> r) -> r` may not illuminate the meaning of the continuation to you, but it certainly puts a lot of restrictions on how you might go about implementing them. This makes it possible to look at the types without any understanding of what they mean, and mechanically derive half of the solution you're looking for.

This is precisely how types increase expressiveness: it's really hard for people to understand dense, highly abstracted code. Types prevent us from wading too far off into the weeds and make handling even more powerful forms of abstractions feasible. You wouldn't rely on this in Python (don't write Haskell in Python!), and in the few cases I've written higher-order functions in this language, I've been sure to also supply Haskell style type signatures. As Simon Peyton Jones has said, the type offers a "crisp" succinct definition of what a function does.

Even more striking is Haskell's solution to the null pointer problem. The exception that strikes terror in the hearts of the Java programmer is the `NullPointerException`: it's a [runtime exception](http://java.sun.com/j2se/1.4.2/docs/api/java/lang/RuntimeException.html), which means that it doesn't need to be explicitly declared in the `throws` specification of a method; a testament to the fact that basically any dereference could trigger this exception. Even in Java, a language of static typing, the type system fails to encode so basic a fact as "am I guaranteed to get a value here?"

Haskell's answer to this problem is the `Maybe` type, which explicitly states in the type of a function that the value could be `Nothing` (null) or `Just a` (the value). Programmers are forced to recognize that there might not be anything, and explicitly handle the failure case (with `maybe`) or ignore it (with `fromJust`, perhaps more appropriately named `unsafeFromJust`). There's nothing really special about the data type itself; I could have written a Java generic that had the same form. The key is the higher order functions that come along with the Functor, Applicative, Monad, MonadPlus, Monoid and other instances of this type. I'd run straight into a wall if I wanted to write this in Java:

```
pureOperation <$> maybeVal

```

`<$>`, a higher order function also known as `fmap`, is critical to this piece of code. The equivalent Java would have to unpack the value from the generic, perform the operation on it, and the pack it up again (with conditionals for the case that it was empty). I could add a method that implements this to the Maybe interface, but then I wouldn't have an elegant way of passing `pureOperation` to these method without using anonymous classes... and you've quickly just exploded into several (long) lines of Java. It becomes dreadfully obvious why the designers didn't opt for this approach: an already verbose language would get even more verbose. Other languages aren't quite as bad, but they just don't get close to the conciseness that a language that celebrates higher order operators can give you.

In summary, while it may seem odd to say this about a language that has (perhaps undeservedly) earned a reputation for being hard to understand, but the constraint of Haskell's type system increases the tolerance of both writer and reader for abstraction that ultimately increases expressiveness. Problems that people just shrugged and claimed, "if you want to fix that, you'll have to add tons of boilerplate," suddenly become tractable. That's powerful.

One final note for the escape artists out there: if you need the dynamic typing (and I won't claim that there aren't times when it is necessary), you can [wriggle out of the static type system completely!](http://www.haskell.org/ghc/docs/6.12.1/html/libraries/base-4.2.0.0/Data-Dynamic.html) Just do it with caution, and not by default.