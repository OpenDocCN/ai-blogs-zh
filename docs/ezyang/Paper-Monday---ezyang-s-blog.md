<!--yml
category: 未分类
date: 2024-07-01 18:18:12
-->

# Paper Monday : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/08/paper-monday/](http://blog.ezyang.com/2010/08/paper-monday/)

Over the weekend, I took the Greyhound up to Seattle to meet up with some friends. The Greyhound buses was very late: forty-five minutes in the case of the trip up, which meant that I had some time to myself in the Internet-less bus station. I formulated the only obvious course of action: start working on the backlog of papers in my queue. In the process, I found out that a paper that had been languishing in my queue since December 2009 actually deals directly with a major problem I spent last Thursday debugging (unsuccessfully) at Galois.

Here are the papers and slide-decks I read—some old, some new—and why you might care enough to read them too. (Gosh, and they’re not all Haskell either!)

* * *

[Popularity is Everything](http://research.microsoft.com/apps/pubs/?id=132859) (2010) by Schechter, Herley and Mitzenmacher. Tagline: *When false positives are a good thing!*

> We propose to strengthen user-selected passwords against statistical-guessing attacks by allowing users of Internet-scale systems to choose any password they want-so long as it's not already too popular with other users. We create an oracle to identify undesirably popular passwords using an existing data structure known as a [count-min sketch](http://www.eecs.harvard.edu/~michaelm/CS222/countmin.pdf), which we populate with existing users' passwords and update with each new user password. Unlike most applications of probabilistic data structures, which seek to achieve only a maximum acceptable rate false-positives, we set a minimum acceptable false-positive rate to confound attackers who might query the oracle or even obtain a copy of it.

[Nelson](http://blog.nelhage.com/) informed me of this paper; it is a practical application of probabilistic data structures like [Bloom filters](http://en.wikipedia.org/wiki/Bloom_filter) that takes advantage of their false positive rate: attackers who try to use your password popularity database to figure out what passwords are popular will get a large number of passwords which are claimed to be popular but are not. The data structure is pretty easy too: someone should go integrate this with the authentication mechanism of a popular web framework as weekend project!

* * *

[Ropes: an Alternative to Strings](http://www.cs.ubc.ca/local/reading/proceedings/spe91-95/spe/vol25/issue12/spe986.pdf) (1995) by Boehm, Atkinson and Plass. Tagline: *All you need is concatenation.*

> Programming languages generally provide a ‘string’ or ‘text’ type to allow manipulation of sequences of characters. This type is usually of crucial importance, since it is normally mentioned in most interfaces between system components. We claim that the traditional implementations of strings, and often the supported functionality, are not well suited to such general-purpose use. They should be confined to applications with specific, and unusual, performance requirements. We present ‘ropes’ or ‘heavyweight’ strings as an alternative that, in our experience leads to systems that are more robust, both in functionality and in performance.

When is the last time you indexed into a string to get a single character? If you are dealing with a multibyte encoding, chances are this operation doesn't even mean anything! Rather, you are more likely to care about searching or slicing or concatenating strings. Practitioners may dismiss this as a preoccupation with asymptotic and not real world performance, but the paper makes a very good point that text editors are a very practical illustration of traditional C strings being woefully inefficient. Ropes seem like a good match for web developers, who spend most of their time concatenating strings together.

* * *

[Autotools tutorial](http://web.mit.edu/~ezyang/Public/autotools.pdf) (last updated 2010) by Duret-Lutz. (Rehosted since the canonical site seems down at time of writing.) Tagline: *Hello World: Autotools edition.*

> This presentation targets developers familiar with Unix development tools (shell, make, compiler) that want to learn Autotools

Despite its unassuming title, this slide deck has become the default recommendation by most of my friends if you want to figure out what this “autogoo” thing is about. In my case, it was portably compiling shared libraries. Perhaps what makes this presentation so fantastic is that it assumes the correct background (that is, the background that most people interested but new to autotools would have) and clearly explains away the black magic with many animated diagrams of what programs generate what files.

* * *

[Fun with Type Functions](http://research.microsoft.com/~simonpj/papers/assoc-types/fun-with-type-funs/typefun.pdf) (2009) by Oleg Kiselyov, Simon Peyton Jones and Chung-chieh Shan. See also [Haskellwiki](http://www.haskell.org/haskellwiki/Simonpj/Talk:FunWithTypeFuns). Tagline: *Put down those GHC docs and come read this.*

> Haskell's type system extends Hindley-Milner with two distinctive features: polymorphism over type constructors and overloading using type classes. These features have been integral to Haskell since its beginning, and they are widely used and appreciated. More recently, Haskell has been enriched with type families, or associated types, which allows functions on types to be expressed as straightforwardly as functions on values. This facility makes it easier for programmers to effectively extend the compiler by writing functional programs that execute during type-checking.

Many programmers I know have an aversion to papers and PDFs: one I know has stated that if he could, he’d pay people to make blog posts instead of write papers. Such an attitude would probably make them skip over a paper like this, which truly is the tutorial for type families that you’ve been looking for. There is no discussion of the underlying implementation: just thirty-five pages of examples of type level programming. Along the way they cover interfaces for mutable references (think STRef and IORef), arithmetic, graphs, memoization, session types, sprintf/scanf, pointer alignment and locks! In many ways, it’s the cookbook I mentioned I was looking for in [my post Friday](http://blog.ezyang.com/2010/08/the-gateway-drug-to-type-programming/).

* * *

[Purely Functional Lazy Non-deterministic Programming](http://www.cs.rutgers.edu/~ccshan/rational/lazy-nondet.pdf) (2009) by Sebastian Fischer, Oleg Kiselyov and Chung-chieh Shan. Tagline: *Sharing and caring can be fun!*

> Functional logic programming and probabilistic programming have demonstrated the broad benefits of combining laziness (non-strict evaluation with sharing of the results) with non-determinism. Yet these benefits are seldom enjoyed in functional programming, because the existing features for non-strictness, sharing, and non-determinism in functional languages are tricky to combine.
> 
> We present a practical way to write purely functional lazy non-deterministic programs that are efficient and perspicuous. We achieve this goal by embedding the programs into existing languages (such as Haskell, SML, and OCaml) with high-quality implementations, by making choices lazily and representing data with non-deterministic components, by working with custom monadic data types and search strategies, and by providing equational laws for the programmer to reason about their code.

This is the paper that hit right at home with of some code I’ve been wrangling with at work: I’ve essentially been converting a pure representation of a directed acyclic graph into a monadic one, and along the way I managed to break sharing of common nodes so that the resulting tree is exponential. The explicit treatment of sharing in the context of nondeterminism in order to get some desirable properties helped me clarify my thinking about how I broke sharing (I now fully agree with John Matthews in that I need an explicit memoization mechanism), so I’m looking forward to apply some of these techniques at work tomorrow.

* * *

That’s it for now, or at least, until the next *Paper Monday!* (If my readers don’t kill me for it first, that is. For the curious, the current backlog is sixty-six papers long, most of them skimmed and not fully understood.)