<!--yml
category: 未分类
date: 2024-07-01 18:17:45
-->

# A Year of Notebooking (Part 1) : ezyang’s blog

> 来源：[http://blog.ezyang.com/2011/06/a-year-of-notebooking-part-1/](http://blog.ezyang.com/2011/06/a-year-of-notebooking-part-1/)

Over the year, I’ve accumulated three notebooks worth of miscellaneous notes and musings. Since these notebooks are falling apart, I’ve decided to transfer their contents here. Warning: they might be slightly incoherent! This is the first of three notebooks. I recommend skimming the section headers and seeing if any of them pop out.

### Tony Hoare: Abstract Separation Algebra

Tony Hoare wants to leverage “the hard work [that] is already solved” by placing the formalism of separation logic (e.g. Hoare triples) into an abstract algebra. The idea is that by encoding things in pairs, not triples, we can take advantage of the numerous results in algebra. The basic idea is we take a traditional triple `{p} q {r}` and convert it into a ordered semigroup relation `p; q <= r`, where `;` is a monoidal operation. In the end we end up with a separation algebra, which is a monoidal lattice with an extra star operator. The choice of axioms is all: “This is abstract algebra, so you should be willing to take these axioms without having any model in mind.” (Scribbled here: “Inception as a metaphor for mathematical multi-level thinking.”) We have a homomorphism (not isomorphism) between implementations and specifications (the right direction is simplification, the left direction is a Galois connection.) In fact, as a commenter in the audience points out, this is known as the Stone Dualities—something like how two points determine a line—with contravariant points and properties. I believe Tony’s been thinking about this topic a bit since I went to this talk at the very beginning of this year, so its likely some or all of this has been superseded by later discoveries. C'est la vie!

### Satnam Singh: Multi-target parallel processing

Can we write parallel code that can execute on multiple types of hardware: e.g. vectorized operations on a traditional CPU, a GPU or an FPGA? He presents an EDSL that can be embedded in any language (well, for this particular representation, C#), with constructs like `newFloatParallelArray`, `dx9Target.toArray1D(z)` and overloaded operators. In my notes, I remark: can this representation be implemented taglessly, or do we always pay the cost of building a description of the system before we can execute it? Pushing software to hardware is especially important in the face of heterogenous processors (e.g. Metropolis). Satnam was a very engaging speaker, and many of the [quotes here](http://blog.ezyang.com/2010/10/quote-day/) are attributed to him—though one quote I do have is “I hope this is not going to be quoted” (don’t worry, I haven’t quoted that sentence). Dancing is a metaphor for parallel processing (though I don’t remember what the metaphor was.) What about self-modifying hardware: we mmap the circuit description and let the hardware reprogram the FPGA!

Higher level information is crucial to optimization: thus we may want a symbolic evaluator with just in time compilation (except we can’t do that for FPGAs.) Memory access fusion is important: we want to get rid of accidental semicolons. `Array -> Stream / Shift = Delay`. Research idea: geometric encoding of common concurrency problems. Matrix inversion is a problem (so don’t invert the matrix, silly), local memory bounds GPU versus FPGA, and scheduling problem of *energy*.

### Streambase

Streambase is a company that implements a visual, event stream processing language. I interviewed with them to possibly work on their compiler; while I ended up turning them down, it was a very interesting interview and I think I would have had a lot of fun working for them (though working in Java, not so much!) The interview was very fun: one question was, “Explain monads to me.” Man, I still don’t know how to do that properly. (Side note: people really, really like side effects. The programming folklore around writing performant programs that work on persistent data is very new, perhaps I’d say even more so than the folklore around lazy evaluation).

### Skepticism

One of the things Alexander Bird’s book *Philosophy of Science* taught me was how to identify unproductive skepticism, even when it is not obvious, in things such as Hume’s problem of induction. My essay on Reliabilism was quite good; it was the only essay I managed to get a first on in my philosophy supervisions during the year. Like type theory, justification is stratified, with layers upon layers.

### Simon Peyton Jones: Let Should Not Be Generalized

Programmers in Hindley-Milner type systems have long enjoyed the benefits of practical type inference: we can generally expect the most general type to be inferred, and syntactic substitution of expressions in their use-sites will always typecheck. Of course, type inference algorithms are in general EXPTIME-complete, but type theorists can’t complain too much about that, since for more powerful logics inference is usually undecidable. (BTW: dictionaries constitute *runtime* evidence. Good way of thinking about it.) Curiously enough, in the presence of more advanced type features, writing type signatures can actually make the type checker’s job harder, but they add local equality assumptions that need to be handled by the constraint solver. Generalized let means that all of these constraints cannot be solved until we reach the call site. Can we work around this problem by doing on the fly solving of equality constraints? The French have a paper about this, but Peyton Jones recommends carrying along a jar of aspirins if you decide to read the paper. After his talk, one of the grad students remarked that Hindley-Milner is, in many ways, an anomaly: users of Agda have an expectation of needing to specify all type signatures, except in some special cases where they can eliminate them, whereas users of Haskell have an expectation of needing to specify no type signatures, except in special cases.

### Stream processing of trees

A long-standing problem with the Document Object Model is that it requires the entire document to be loaded up in memory. In cases like PHP’s documentation manual, the memory usage can be over a gigabyte large. Unfortunately, the mental model for manipulating a DOM is much more natural than that for manipulating a stream of XML tag events. Is there a way to automatically project changes to the DOM into changes on a stream? We’d like to construct an isomorphism between the two. I'm seeking a functional representation of the DOM, for manipulation (you're still going to need mutability for a DOM-style event programming model.) “Clowns to the left of me, jokers to the right” emphasizes the difference between local and global analysis. One way you might look at traversal of a token stream is simply traversal, with a zipper to keep track of where you are. Of course, the zipper takes up memory (in effect, it forms something like a stack, which is exactly how you would convert a token stream into a tree.) So we can efficiently build the tree representation without mutation, but we still end up with a tree representation. At this point, I have written down, “Stop hitting yourself.” Indeed. Can we take advantage of domain specific knowledge, a claim that I promise not to go beyond this point? The idea of projecting DOM operations into XML stream operations, and using this as a sort of measurement for how costly something is may be profitable. Of course, now I should do a literature search.

### Regular expression edit distance

Given a regular expression and a non-matching string, what is the minimum number of edits necessary to make the string match? There may be multiple answers, and the algorithm should allow you to weight different changes differently.

### Frank Tip: Test Generation and Fault Localization for Web Applications

Apollo takes a hybrid approach to testing web applications, combining concrete and symbolic execution. The idea is that most web applications have diffuse, early conditionalization, with no complex state transformations or loops. So, we generate path constraints on the controller and solve them, and then generate inputs which let us exercise all of the control paths. Data is code: we want to describe the data. I must not have been paying very close attention to the presentation, because I have all sorts of other things scribbled in: “Stacks are the wrong debugging mechanism for STGs” (well, yes, because we want to know where we *come from*. Unfortunately, knowing where we are *going* isn’t very useful either) and “Can we automatically generate QuickCheck shrink implementations using execution traces?” (a sort of automated test-case minimization) and a final musing, “Haskell is not a good langauge for runtime inspection or fault localization.”

### Benjamin Pierce: Types and Programming Languages

It would be very cool if someone made an interactive visualization of how type systems grow and are extended as you add new features to them, a sort of visual diff for typing rules and operational semantics.

### Smoothed trending

As a blogger, my page view counts tend to be very spiky, corresponding to when my post hits a popular news site and gets picked up (to date, my Bitcoin post has had 22k views. Not bad!) But this doesn’t help me figure out long term trends for my website. Is there a way to smooth the trends so that spikes become simply “popular points” on a longer term trend?

### User interfaces

I want a bible of minimal technical effort best practice user interfaces, patterns that are easy to implement and won’t confuse users too much. UI design is a bit too tweaky for my tastes. In the case of intelligent interfaces, how do we not piss off the user (e.g. Google Instant.) We have a user expectation where the computer will not guess what I want. That’s just strange.

### Page 32

In big letters I have: “Prove locality theorems. No action at a distance.” Almost precisely these same words Norman Ramsey would tell me while I was working with Hoopl. I think this is a pretty powerful idea.

### Separation Logic and Graphical Models

I have some mathematical definitions written down, but they’re incomplete. I don’t think I wrote anything particularly insightful. This says something about the note-taking enterprise: you should record things that you would not be able to get later, but you should also make sure you follow up with all the complete information you said you’d look up later.

### Jane Street

I have two pages of scribblings from solving problems over a telephone interview. I quite enjoyed them. One was a dynamic programming question (I moffed the recurrence at first but eventually got it), the second was implementing a functional programming feature in OCaml. Actually, I wanted to write a blog post about the latter, but it’s so far been consigned to my drafts bin, awaiting a day of resurrection. Later in my notes (page 74) I have recorded the on-site interview questions, unfortunately, I can’t share them with you.

### Quote

“It’s like hiring an attorney to drive you across town.” I don’t remember what the context was, though.

### Mohan Ganesalingam: Language of Mathematics

I really enjoyed this talk. Mohan looks at applying NLP at a domain which is likely to be more tractable than the unrestricted human corpus: the domain of mathematical language. Why is it tractable? Math defines its lexicon in text (mathematical words must be explicitly defined), we mix symbols and natural language, and the grammar is restricted. Montague Grammars are in correspondence with Denotational Semantics. Of course, like normal language, mathematical language is heavily ambiguous. We have lexical ambiguity (“prime” can describe numbers, ideals, etc.), structural ambiguity (p is normal if p *generates* the *splitting field* of *some polynomial* **over F_0**—is F_0 referring to the generating or the polynomial?), symbolic ambiguity (`d(x + y)`, and this is not just operator overloading because parse trees can change: take for example `(A+B)=C` versus `λ+(M=N)`), and combined symbolic and textual ambiguity. It turns out the linguistic type system of maths, which is necessary to get the correct parse trees, is not mathematical at all: integers, reals and friends are all lumped into one big category of numbers and types are not extensional (objects have different types depending on contents.) We need a dynamic type system, not a structural or nominal one, and we need to infer types *while* parsing.

### Writing notes

From December 1st. I seem to need to write concluding paragraphs that are more concluding, and use shorter sentences. Summarize parts of my arguments, give more detail about experiments, and not to forget that a large part of historic mathematics was geometry. Aim for more in less sentences. Amen!

Another set of notes: all questions are traps: the examiner wants you to think about what is asked. Think about the broader context around events. You may not have enough time to compare with contemporary perspectives. **Put guideposts in your essay.** Be careful about non sequiturs. Colons are good: they add emphasis (but use them carefully.) **Short sentences!**

### Principia Mathematica

What a wonderful conference! There were a lot of talks and I should have taken more notes, but this is what I have, some quotes and sketches.

The algebraist versus the analyst. “Four riding in on a bicycle and then riding off again.” Numbers as moments, not an object (though it doesn’t lose generality.) “Cantor is *hopeless* at it.” (on zero.) “Do [numbers] start with 0 or 1? Yes and yes.” Frege and Russell finally give proper status to zero. The misreading of counting: does arithmetic start from counting? Number sequence is already in place, rather, we construct an isomorphism. There is a mistaken belief we count from one. Isomorphisms avoid counting, give proper status to zero, and sidestep the issue of how counting actually works (a transitive verb: pre-counting, we must decide what to count.) Contrary to popular depiction in Logicomix, Gödel and Russell did meet. Quine logic and Church logic. “It is not true that the square root of two is not irrational” requires every number be rational or irrational.

Why do we care about old people? How do we make progress in philosophy? Orders were syntactic rather than semantic: Kripke and Tarksi developed a hierarchy of truths. Free variable reasoning helped resolve nominala nd typical ambiguity: a scientific approach to a philosophical problem. “What nowadays constitutes research—namely, Googling it.” Nominal ambiguity: assert “x is even”, actually “forall x, x is even.” Quote: “It’s clear from the letter that he didn’t get past page five [of the Principia].” The word variable is deeply misleading, it’s not a variable name (progress!) “There are no undetermined men.” Anaphoric pronoun. We can’t express rules of inference this way.

Types: variables must have ranges. Hardly any theorems (all of the statements were schemas): we wanted to prove things about all types, but couldn’t on pain of contradiction. So all of the variables are **type** ically ambiguous. There is an argument in volume 2 respecting infinity, but a small world gives you the *wrong* mathematics (positivism.) But there was a bright idea:even if there aren’t enough things in the world, if there are k things, there are 2^k classes of things, etc. Go up the hierarchy. *This* is the interpretation for typical ambiguity. Whitehead thought theories were meaningless strings without types (a sort of macro in theory-land). ST made the language/metalanguage distinction!! “Seeing” is determining the type. The logocentric predicament is you’re supposed to use reasoning, but this reasoning is outside the formal system. Puns of operators on higher types, decorating all operators with a type label. The stratification of types.

Free variable reasoning is the same for typically ambiguous reasoning. Abbreviation for quantified reasoning (needs messy rules for inside quantifiers), indefinite names (can’t be the variable name, can’t lead to indefinite things), schematic names (lambdas: correct for variables, and modern for types.) But arguments that don’t convince someone unless they believe it (skepticism) sees: if the correct logic is type theoretic and oustide it, then we don’t have a position outside reasoning. (It’s a one way direction.) **I think there is a way of talking about a system from within it.** We have a weakened sense of truth: if you already believe it, it’s OK, but there is no convincing power.

The next lecture came from the computer scientist world. “Arguably, the more [programming languages] took from formal logic, the better it is.” Otherwise, it is the “ad-hoc craetion of electricians”. Computers allow easy formal manipulation and correctness checks. But for the mathematics? There isn’t very much of it. Proofs can be checked algorithmically (with formal inference rules). “Because there are many philosophers here, I hope I can answer questions in a suitably ambiguous manner.” Symbolism allows us to do “easy” things mechanically (Whitehead quote.) Do we need formal methods? In 1994 Pentium was found to have an error in floating point division. Robin’s conjecture was incorrectly proved. Different proof systems: de Bruijn generates proofs which are checked by a separate checker, LCF reduces all rules to primitive inferences checked by a logical kernel. After all, why don’t we prove that our proof assistants work? HOL Light (Principia) is only 430 lines of code. Schaffer’s joke: Ramseyfied Types. Right now, formal logic is on the edge of 20th century research mathematics, proofs needing “only 10k lines of code.” Maintenance of formal proofs is a big problem: we need intermediate declarative abstract models. Check out Flyspeck.

I had some scribblings in the margins: “references in logic?” (I think that’s linear logic), how about performance proofs (guaranteed to run in such-and-such time, realtime proofs), or probabilistically checkable proofs. Maybe complexity theory has something to say here.

### Turing Machines

Their method of efficient access is... a zipper. Oh man!

### GHC

My scribblings here are largely illegible, but it seems a few concepts initially gave me trouble:

*   Stack layout, keeping up and down straight, info tables, and the motion of the stack pointer. I have a pretty good idea how this all works now, but in the beginning it was quite mysterious.
*   `CmmNode` constructors have a lot of field, and constructing a correspondence with printed C-- is nontrivial.
*   Sizes of variables.
*   Headers, payloads and code.
*   Pointer tagging, esp. with respect to values living in registers, on the stack, and what the tag bits mean depending on context (functions or data). I never did figure out how the compacting GC worked.

This concludes the first notebook.