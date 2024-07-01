<!--yml
category: 未分类
date: 2024-07-01 18:18:10
-->

# Defining “Haskelly” : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/08/defining-haskelly/](http://blog.ezyang.com/2010/08/defining-haskelly/)

## Defining “Haskelly”

At risk of sounding like a broken record, the topic of this post also sprang from [abcBridge](http://blog.ezyang.com/2010/08/galois-tech-talk-abcbridge-functional-interfaces-for-aigs-and-sat-solving/). John Launchbury asked a question during my presentation that got me thinking about API design in Haskell. (By the way, [the video for the talk](http://vimeo.com/14432112) is out! Unfortunately, the second half had to be cut out due to technical difficulties, but you can still check out the slides.)

His question was this:

> You’ve presented this in a very imperative style, where you’ve got this AIG structure in the ABC tool, and what you’ve really done is given me a nicely typed Haskell typed interface that allows you to go in a put a new gate or grab a structure, and I’m left wondering, what is the reason for needing this tight tie-in with what’s going on in that space? Here is a thought experiment: I could imagine myself having a purely functional data structure that is describing the data structure...and you end up with a functional description of what you want your graph to look like, and then you tell ABC to go and build the graph in one go.

I had claimed that abcBridge was a “functional API” for manipulating and-inverter graphs; perhaps I was lying! Is abcBridge—with its close correspondence to the underlying imperative code—truly “functional?” Or, if it’s not functional, does it at least have a “Haskelly” API? (What does it even mean for an API to be Haskelly?) Why does the purely functional interface seem morally better than the imperative interface? It’s a question of philosophical import as well as practical import—why do we prefer the functional interface which might require a more complex underlying implementation?

My conjecture is that the faithfulness of an API to its host language is based on how much it utilizes particular features that a language makes easy to use. Haskell is frequently introduced as a “purely functional, lazy, strongly statically typed programming language.” Looking at each of these terms in turn (informally)...

*   *Purely functional* indicates APIs that eschew destructive updates, instead opting for immutability and persistent data structures. Language features that make it easier to write in this style include the `final` and `const` keywords, algebraic data types, pattern-matching and a library of persistent data structures to write more persistent data structures with.
*   *Lazy* indicates APIs that utilize laziness to build custom control structures and generate infinite data structures. The poster child language feature for laziness is, well, lazy evaluation by default, but explicit laziness annotations in a strict language or even a convenient lambda abstraction encourages lazy style. (Python does not have a convenient lambda, which is why asynchronous frameworks like Twisted are so painful!)
*   *Strongly statically typed* indicates APIs that encode invariants of all shapes and sizes into the static type system, so that programmer errors can be caught at compile time, not run time. The type system is the obvious language feature here, with its expressiveness defining what you can easily add to your system.

We associate programs that take advantage of these language features as “Haskelly” because Haskell makes it easy—both syntactically and conceptually—to use them! But at the same time, these are all (mostly) orthogonal language features, and for any given API you might write, you may opt to ditch some of these properties for others: maybe the feature just doesn’t matter in your problem domain, maybe the feature imposes an unacceptable performance penalty or is an insufficiently sealed abstraction.

With abcBridge as our concrete example, here is how you might make such classifications in practice:

*   The monadic interface for constructing networks is about as far from purely functional as you can get, which was an explicit design choice in the name of performance and control. (Fortunately, we can build a nicer API on top of this one—in fact, I did an experimental implementation of one.) However, when you’re dealing with fully constructed networks the API takes a purely functional style, doing copying and `unsafePerformIO` behind the scenes to preserve this illusion.
*   abcBridge does not directly use laziness: in particular the monadic code is very structured and doesn’t have a lot of flow control in it.
*   The static type system is a huge part of abcBridge, since it is operating so close to the bare metal: it has two monads, with an intricate set of functions for running and converting the monads, and the low level FFI bindings make every attempt to enhance the existing C-based type system. Notice the interesting play between the types and a functional interface: if we had a purely functional interface, we probably could have ditched most of these complicated types! (Imperative code, it seems, needs stronger type system tricks.)

As far as pure Haskell libraries go, abcBridge is very un-Haskelly: I would certainly expect more from an equivalent library implemented in pure Haskell. But it is leaps and bounds better than the C library it sprang from. How far should one push the envelope? It is all about striking the right balance—that is why API design is an art.