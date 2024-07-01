<!--yml
category: 未分类
date: 2024-07-01 18:17:42
-->

# Parallelism to plug space leaks : ezyang’s blog

> 来源：[http://blog.ezyang.com/2011/07/parallelism-to-plug-space-leaks/](http://blog.ezyang.com/2011/07/parallelism-to-plug-space-leaks/)

It is [not too difficult (scroll to “Non sequitur”)](http://blog.ezyang.com/2010/11/is-multiply-carry-strongly-universal/) to create a combinator which combines two folds into a single fold that operates on a single input list in one pass. This is pretty important if your input list is pretty big, since doing the folds separately could result in a space leak, as might be seen in the famous “average” space leak:

```
import Data.List
big = [1..10000000]
sum' = foldl' (+) 0
average xs = fromIntegral (sum' xs) / fromIntegral (length xs)
main = print (average big)

```

(I’ve redefined `sum` so we don’t stack overflow.) I used to think combining functions for folds were pretty modular, since they had a fairly regular interface, could be combined together, and really represented the core notion of when it was possible to eliminate such a space leak: obviously, if you have two functions that require random access to elements of the list, they’ll retain the entirety of it all the way through.

Of course, a coworker of mine complained, “No! That’s not actually modular!” He wanted to write the nice version of the code, not some horrible gigantic fold function. This got me thinking: is it actually true that the compiler can’t figure out when two computations on a streaming data structure can be run in parallel?

But wait! We can tell the compiler to run these in parallel:

```
import Data.List
import Control.Parallel
big = [1..10000000]
sum' = foldl' (+) 0
average' xs =
    let s = sum' xs
        l = length xs
    in s `par` l `par` fromIntegral s / fromIntegral l
main = print (average big)

```

And lo and behold, the space leak goes away (don’t forget to compile with `-threaded` and run with at least `-N2`. With the power of multiple threads, both operations can run at the same time, and thus there is no unnecessary retention.

It is perhaps not too surprising that `par` can plug space leaks, given that `seq` can do so too. But `seq` has denotational content; `par` does not, and indeed, does nothing when you are single-threaded. This makes this solution very fragile: at runtime, we may or may not decide to evaluate the other thunk in parallel depending on core availability. But we can still profitably use `par` in a single-threaded context, if it can manage pre-emptive switching between two consumers of a stream. This would be a pretty interesting primitive to have, and it would also be interesting to see some sort of semantics which makes clear the beneficial space effects of such a function. Another unbaked idea is that we already have a notion of good producers and consumers for stream fusion. It doesn’t seem like too far a stretch that we could use this analysis to determine when consumers could be merged together, improving space usage.