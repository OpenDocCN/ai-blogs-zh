<!--yml
category: 未分类
date: 2024-07-01 18:18:21
-->

# Creative catamorphisms : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/04/creative-catamorphisms/](http://blog.ezyang.com/2010/04/creative-catamorphisms/)

*The bag of programming tricks that has served us so well for the last 50 years is the wrong way to think going forward and must be thrown out.*

Last week, Guy Steele came in and did a guest lecture ["The Future is Parallel: What's a Programmer to Do?"](http://groups.csail.mit.edu/mac/users/gjs/6.945/readings/MITApril2009Steele.pdf) for my advanced symbolic class (6.945). It's a really excellent talk; such an excellent talk that I had seen the slides for prior to the talk. However hearing Guy Steele talk about it in person really helped set things in context for me.

One of the central points of the talk is the call for more *creative catamorphisms.* Well, what is a creative catamorphism? To answer this question, we first have to understand what a catamorphism is. The functional programming crowd is well familiar with a few relatively banal examples of the catamorphism, namely the left fold and the right fold. One way to think about folds is simply a "level of abstraction" above a loop one might write in an imperative language. Another way to think of the fold is replacing the type constructor for the list (the cons or `:` operation) with another function, as seen in Cale Gibbard's excellent diagrams:

The point of the catamorphism is that this doesn't need to apply just to lists; in fact, we can run a catamorphism on *any* recursive data structure! Just make a function for each constructor in the type, with the appropriate arity (so a ternary tree would require functions that take three arguments, and so forth), and let her rip! This is vitally important because the old left and right fold are the "wrong way to think"; by the very nature of their structure they require you to evaluate sequentially. But set things up in a binary tree, and you can evaluate all the subtrees first before combining them at the end.

So what is a *creative* catamorphism? It's when the original recursive data structure doesn't map cleanly on to the atoms that your computation wants to deal with. The example Guy Steele discusses in his talk is the time honored task of breaking a string into its words. A string is merely a list of characters, which only lets us handle it character by character (traditional sequential), or a naive transformation into a binary tree, which only gives us efficient bisection (parallelizable). The trouble with naive bisection is that it might split in the middle of the word, so our combining function has to account for this case. How to deal with this is left as an exercise for the reader (or you can go read the slides.)

In fact, this was the critical moment when I understood the global reasoning behind what Edward Kmett was talking about when he gave his (in my opinion pretty crazy) talk on ["A Parallel Parsing Trifecta: Iteratees, Parsec, and Monoids"](http://comonad.com/reader/wp-content/uploads/2009/08/A-Parsing-Trifecta.pdf). The goal of this code is to massively parallelize parsing by splitting up the input document into chunks and then recombining them with the parsing function. He has to deal with the same problems that showed up in the toy example in Steele's talk, and he pulls out all sorts of tricks to get things pumping.

I will admit, the work is complicated, and at times, it feels like overkill. But it's a brave new parallel world, and it's time we fully explore the designs and implications of it. With any luck, we will be able to write parallel programs as naturally we can write sequential programs, but it's a long way getting there.

* * *

**Update (2013-05-21).** Oleg writes in to tell me that there is actually a name for these types of tricks: an *almost homomorphism*. It is not surprising to see that the [work described in the Skepara project](http://research.nii.ac.jp/~hu/project/skepara.html) collaborated with Guy Steele and the Fortress project; it is well worth checking out for a *calculational* approach for deriving these catamorphisms.