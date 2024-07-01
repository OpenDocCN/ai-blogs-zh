<!--yml
category: 未分类
date: 2024-07-01 18:18:00
-->

# Generalization and vagueness in specifications : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/12/generalization-and-vagueness-in-specifications/](http://blog.ezyang.com/2010/12/generalization-and-vagueness-in-specifications/)

*What semantics has to say about specifications*

Conventional wisdom is that premature generalization is bad (architecture astronauts) and vague specifications are appropriate for top-down engineering but not bottom-up. Can we say something a little more precise about this?

Semantics are formal specifications of programming languages. They are perhaps some of the most well-studied forms of specifications, because computer scientists love tinkering with the tools they use. They also love having lots of semantics to pick from: the more the merrier. We have small-step and big-step operational semantics; we have axiomatic semantics and denotational semantics; we have game semantics, algebraic semantics and concurrency semantics. Describing the programs we actually write is difficult business, and it helps to have as many different explanations as possible.

In my experience, it’s rather rare to see software have multiple specifications, each of them treated equally. Duplication makes it difficult to evolve the specification as more information becomes available and requirements change (as if it wasn’t hard enough already!) Two authoritative sources can conflict with each other. One version of the spec may dictate how precisely one part of the system is to be implemented, where the other leaves it open (up to some external behavior). What perhaps is more common is a single, authoritative specification, and then a constellation of informative references that you might actually refer to on a day-to-day basis.

Of course, this happens in the programming language semantics world all the time. On the subject of conflicts and differing specificity, here are two examples from denotational semantics (Scott semantics) and game semantics.

*Too general?* Here, the specification allows for some extra behavior (parallel or in PCF) that is impossible to implement in the obvious way (sequentially). This problem puzzled researchers for some time: if the specification is too relaxed, do you add the feature that the specification suggests (PCF+por), or do you attempt to modify the semantics so that this extra behavior is ruled out (logical relations)? Generality can be good, but it frequently comes at the cost of extra implementation complexity. In the case of parallel or, however, this implementation complexity is a threaded runtime system, which is useful for unrelated reasons.

*Too vague?* Here, the specification fails to capture a difference in behavior (seq and pseq are (Scott) semantically equivalent) that happens to be important operationally speaking (control of evaluation order). Game semantics neatly resolves this issue: we can distinguish between ``x `pseq` y`` and ``y `pseq` x`` because in the corresponding conversation, the expression asks for the value of x first in the former example, and the value of y first in the latter. However, vague specifications give more latitude to the compiler for optimizations.

Much like the mantra “the right language for the job”, I suspect there is a similar truth in “the right style of specification for the job.” But even further than that, I claim that looking at the same domain from different perspectives deepens your understanding of the domain itself. When using semantics, one includes some details and excludes others: as programmers we do this all the time—it’s critical for working on a system of any sort of complexity. When building semantics, the differences between our semantics give vital hints about the abstraction boundaries and potential inconsistencies in our original goals.

There is one notable downside to a lot of different paradigms for thinking about computation: you have to learn all of them! Axiomatic semantics recall the symbolic manipulation you might remember from High School maths: mechanical and not very interesting. Denotational semantics requires a bit of explaining before you can get the right intuition for it. Game semantics as “conversations” seems rather intuitive (to me) but there are a number of important details that are best resolved with some formality. Of course, we can always fall back to speaking operationally, but it is an approach that doesn’t scale for large systems (“read the source”).