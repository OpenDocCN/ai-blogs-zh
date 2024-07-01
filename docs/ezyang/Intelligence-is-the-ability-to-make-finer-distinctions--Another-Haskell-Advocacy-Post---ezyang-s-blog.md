<!--yml
category: 未分类
date: 2024-07-01 18:18:04
-->

# Intelligence is the ability to make finer distinctions: Another Haskell Advocacy Post : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/10/finer-distinctions/](http://blog.ezyang.com/2010/10/finer-distinctions/)

My parents like foisting various self-help books on me, and while I sometimes groan to myself about it, I do read (or at least skim) them and extract useful bits of information out of them. This particular title quote is from Robert Kiyosaki’s “rich dad” in *Rich Dad, Poor Dad.*

“Intelligence is the ability to make finer distinctions” really spoke to me. I’ve since found it to be an extremely effective litmus test to determine if I’ve really understood something. A recent example comes from my concurrent systems class, where there are many extremely similar methods of mutual exclusion: mutexes, semaphores, critical regions, monitors, synchronized region, active objects, etc. True knowledge entails an understanding of the conceptual details differentiating these gadgets. What are the semantics of a *signal* on a condition variable with no waiting threads? Monitors and synchronized regions will silently ignore the signal, thus requiring an atomic release-and-wait, whereas a semaphore will pass it on to the next *wait*. Subtle.

* * *

We can frequently get away with a little sloppiness of thinking, indeed, this is the mark of an expert: they know precisely how much sloppiness they can get away with. However, from a learning perspective, we’d like to be able to make as fine a distinction as possible, and hopefully derive some benefit (either in the form of deeper understanding or a new tool) from it.

Since this is, after all, an advocacy piece, how does learning Haskell help you make finer distinctions in software? You don’t have to look hard:

> Haskell is a standardized, general-purpose **purely** functional programming language, with **non-strict semantics** and strong static typing.

These two bolded terms are concepts that Haskell asks you to make a finer distinction on.

* * *

*Purity.* Haskell requires you to make the distinction between pure code and input-output code. The very first time you get the error message “Couldn't match expected type `[a]` against inferred type `IO String`” you are well on your way to learning this distinction. Fundamentally, it is the difference between *computation* and *interaction with the outside world*, and no matter how imperative your task is, both of these elements will be present in a program, frequently lumped together with no indication which is which.

Pure code confers tremendous benefits. It is automatically thread safe and asynchronous exception safe. It has no hidden dependencies with the outside world. It is testable and deterministic. The system can speculatively evaluate pure code with no commitment to the outside world, and can cache the results without fear. Haskellers have an obsession with getting as much code outside of IO as possible: you don’t have to go to such lengths, but even in small doses Haskell will make you appreciate how what is considered good engineering practice can be made rigorous.

* * *

*Non-strict semantics.* There are some things you take for granted, the little constants in life that you couldn’t possibly imagine be different. Perhaps if you stopped and thought about it, there was another way, but the possibility never occurred to you. Which side of the road you drive is one of these things; strict evaluation is another. Haskell asks you to distinguish between strict evaluation and lazy evaluation.

Haskell isn’t as in your face about this distinction as it is about purity and static typing, so it’s possible to happily hack along without understanding this distinction until you get your first stack overflow. At which point, if you don’t understand this distinction, the error will seem impenetrable (“but it worked in the other languages I know”), but if you are aware, a stack overflow is easily fixed—perhaps making the odd argument or data constructor explicitly strict.

Implicit laziness has a number of notable benefits. It permits user-level control structures. It encodes streams and other infinite data structures. It is more general* than strict evaluation. It is critical in the construction of amortized persistent data structures. (Okasaki) It also is not always appropriate to use: Haskell fosters an appreciation of the strengths and weaknesses of strictness and laziness.

> * Well, almost. It only fully generalizes strict evaluation if you have infinite memory, in which case any expression that strictly evaluates also lazy evaluates, while the converse is not true. But in practice, we have such pesky things as finite stack size.

* * *

*The downside.* Your ability to make finer distinctions indicates your intelligence. But on the same token, if these distinctions don’t become second nature, they impose a cognitive overhead whenever you need invoke them. Furthermore, it makes it difficult for others who don’t understand the difference to effectively hack on your code. (Keep it simple!)

Managing purity is second-nature to experienced Haskellers: they’ve been drilled by the typechecker long enough to know what’s admissible and what’s not, and given the mystique of monads, it’s usually something people actively try to learn when starting off with Haskell. Managing strictness also comes easily to experienced Haskellers, but has what I perceive to be a higher learning curve: there is no strictness analyzer yelling at you when you make a suboptimal choice, and you can get away with not thinking about it most of the time. Some might say that lazy-by-default is not the right way to go, and are [exploring the strict design space](http://trac.haskell.org/ddc/). I remain optimistic that we Haskellers can build up a body of knowledge and teaching techniques to induct novices into the mysteries and wonders of non-strict evaluation.

* * *

So there they are: purity and non-strictness, two distinctions that Haskell expects you to make. Even if you never plan on using Haskell for a serious project, getting a visceral feel for these two concepts will tremendously inform your other programming practice. Purity will inform you when you write thread-safe code, manage side-effects, handle interrupts, etc. Laziness will inform you when use generators, process streams, control structures, memoization, fancy tricks with function pointers, etc. These are seriously powerful engineering tools, and you owe it to yourself to check them out.