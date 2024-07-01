<!--yml
category: 未分类
date: 2024-07-01 18:17:44
-->

# A Year of Notebooking (Part 2) : ezyang’s blog

> 来源：[http://blog.ezyang.com/2011/06/a-year-of-notebooking-part-2/](http://blog.ezyang.com/2011/06/a-year-of-notebooking-part-2/)

This is notebook two.

### Max Schäfer: Refactoring Java

Most Java refactoring tools built into IDEs like Eclipse are little more than glorified text manipulation macros. There are no guarantees that the result of your refactoring will have the same behavior as the original: you can even refactor code that doesn’t even compile! To prevent this, most refactorings require heavy and hard-to-understand preconditions for refactoring. Max brings two ideas to the table:

*   Rather than attempt to write a complex pre-condition that may or may not accurately reflect safety concerns, we instead do the transformation, and then verify that the refactoring did not break anything. We can do this with a dependency-based description of the behavior program, which *overspecifies* the original semantics (otherwise, such analysis wouldn’t be tractable, as it’s undecidable.)
*   Rather than attempt to write a monster refactoring that attempts to handle all possible cases, we decompose refactorings into microrefactorings on simpler versions of the source language. For example, moving a chunk of code into a method would involve closure conversion (control), lambda lifting (data), and then the actual outwards motion, which at this point is trivial. We can then resugar into the original source language. This allows us to abstract over corner cases.

Modularity is a very powerful philosophy, and one that Hoopl takes too. (One might wonder if Hoopl might be useful for refactoring, one big problem I observe is that Hoopl’s representation is too low level, and really the point of a high-level language is that you don’t need complex dataflow analysis.)

There are some bad assumptions about this, however. It assumes we know the whole program, that it’s only written in one language (no XML manifest files), and it’s statically typed and class based. Of course, one fails these assumptions for all real programs, so if we actually want people to adopt this workflow, we need a story for them. Refactoring is a transition from sloppy to structured code. (I have scribbled at the bottom of the page *responsibility calculus*, but I have no idea what that means now.)

Semantic overspecification reminds me of SLAM’s approach to iterative approximations of program behavior.

### Mike Dodds: Modular Reasoning for Deterministic Parallelism

A frequent problem with separation logics is that you need to introduce a new primitive for every sort of concurrency construct you may want to model, and so you end up with a bazillion different logics, each for your own model of concurrency. Mike presents a way of incrementally building up any sort of semantics you could possibly want, using “concurrent abstract predicates.” You use predicates to generate variables that satisfy various predicates, and then require these conditions for various other specifications of functions.

The majority of this particular talk was working through an odd concurrency construct called `wait/grant`, described in “Quasi-Static Scheduling for Safe Futures.” It’s a barrier that preserves “necessary” sequential dependencies. A bit of time was spent actually clarifying what this construct did, and why it was different from a buffered channel. Tony Hoare remarked that it resembled a “production line” in real-life, which was not very common in programming, though it happened to be quite natural for the problem the original paper authors were tackling (parallelizing sequential programs).

My notes invite me to “implement this in Haskell”, and also have a remark about “tree shaped bit vector propagation”, which is apparently the optimized version. There are also a bunch of code snippets but I’m sure I can find those in the slides.

### Tomas Petricek: Joinads

Joinads for F# are a system for pattern matching on computations (as opposed to just values). How is this any different from monads? Joinads support extra operations: `merge :: m a -> m b -> m (a, b)` and `choose :: [m (Maybe (m a))] -> m a`, which can implement special scheduling properties. This can be useful for futures (Manticore), events (FRP) or the join calculus (joins execute when a channel contains value: it’s cheap and cheerful multiplexing.) It turns out you get a Joinad for free from commutative monads, which might say something about what kind of syntax extensions would be useful for those monads.

I wasn’t particularly impressed by this talk, for whatever reason. I guess my reason was I didn’t feel any deep theoretical reason why Joinads might be a particularly interesting theoretical construct to study, nor did it seem that that they were in any sense minimal. Also, in Haskell, we implement multiplexing by just forking off extra green threads, which is a much nicer model, in my opinion.

### Computing with Real Numbers

Dan Piponi has written on this topic before, but this talk really helped put things in perspective for me.

There are many types of numbers for which we can easily compute with: the field of two elements (booleans), integers (well, discounting overflow), and rationals (pairs of integers). But real numbers pose some difficulties. They are the metric closure of rational numbers Q, i.e. everything that can be a limit of a Cauchy sequence of Q. This is reminiscent of a decimal expansion.

We now want to consider functions over reals. How do we know if something is computable? We might try saying it is a relation from strings of digits to strings of digits, where any finite prefix of `F(p)` can be uniformly computed from some finite prefix of sufficient length of `p`. But this is clearly insufficient for something like 0.3333, where we need to read infinitely many digits to see whether or not this is truly one third. (I remark that infinity is a bit like oracles in Turing Machine land.)

Instead, we say that a sequence of rationals `q_i` (with `i` ranging over naturals) represents a real number `x` if `|x - q_i| < 2^-i` for all `i`. (the base doesn’t really matter). Let `p(q) = x`, then `f` is computable if there exists an `F` such that `p(F(prefix)) = f(p(prefix)`. With this definition, it turns out that addition, multiplication, division, trigonometric functions, exponentiation and logarithms are computable.

An interesting restriction of function is that of continuity, familiar from any treatment in a Calculus textbook: a function `f : R -> R` is continuous if for all `x` in the domain of `f`, and for all `ε > 0`, there exists a `δ > 0` that for all `y` in the domain of f, `|x - y| < δ` implies `|f(x) - f(y)| < ε`. It has been proposed that every computable function is continuous: thus, computability stems from our ability to approximate things, which we can’t do for discontinuous functions (which side of the discontinuity are we on?) We can further restrict functions to `C(R,R)`, which are the set of continuous functions which can also be approximated by infinite sequences (e.g. polynomials.)

Consider the monotone intermediate value thoerem, which states that if `f` is monotone, ```f(0) < 0``and ``f(1) > 0```, there exists some x such that `f(x) = 0`. Can we compute this number? Bisection doesn’t work, since determining if a number is less than or greater than another is in general incomputable. (The general intermediate value theorem is not computable, since we could get infinitely close to the origin line.) We can use trisection instead. Compute `f(0.3)` and `f(0.7)`, and concurrently perform the following comparisons:

1.  If `f(0.3) < 0`, the new range is `[0.3, 1]`
2.  If `f(0.3) > 0`, the new range is `[0, 0.3]`
3.  If `f(0.7) < 0`, the new range is `[0.7, 1]`
4.  If `f(0.7) > 0`, the new range is `[0, 0.7]`

You’ll have to do a little work to get a formal proof of correctness, but it’s easy to see why this intuitively might work: comparisons take infinite time only if the number is close to the point of comparison, but then our other comparison is guaranteed to succeed, since it is some non-infinitesimal distance away. (I have a little margin note: can we use this to do cryptography?)

It turns out there are various schools on the computability of real numbers. The perspective I have just described is the Polish School. However, the Russian School says that uncomputable points don’t exist: the code fragment is the real number. Thus, effective analysis is not classical analysis. There is a correspondence between constructive and classical mathematics. Something about Banach spaces (linear operator has to be bounded), so differentiation is not computable in general, though integration is! (This is quite different from the symbolic evaluation world.) For further reading, see *Computable Analysis*, Springer, 2000.

### Martin Escardó: Selection Functions Everywhere

I didn’t understand this talk. I don’t understand most of Martin’s work, really. But I have a bunch of quotes:

*   “I don’t know why it’s called the continuation monad; there is no control, this talk is completely out of control.”
*   “I’m not going to explain what this means, but I’m going to explain how this works.”
*   “[Incomprehensible statement.] Which is not too deep.”
*   “And the reason is, uh, probably in the next slide.”
*   “Every game is of length 2: you have the first move, and then all the others.”
*   “You’ll get to use the monoidal monad structure to calculate these two people.”

Selection functions select individuals with the “highest” truth value. Max-value theorem, min-value theorem and universal-value theorem (Drinker’s paradox) are all uncomputable. Selection functions are a two stage process. `K A -> A` is double negation elimination, `J A -> A` is Peirce’s law. Bekic’s lemma gives a fixpoint. Bar recursion and do induction on `p` (continuous function) with p as a tree. We can calculate the optimal strategy. System T plus an equation for J-shift is strongly normalizing. We can encode the axiom of countable choice (but it’s not choice anymore!). Dependent choice: (AC classical choice, Tychonoff theory, etc. See that paper.) Why is J a monad?

### Sir Roger Penrose: Twistor Theory

Penrose... doesn’t give very comprehensible talks. Well, he explained the Riemann sphere (a stereographic projection) quite well, but then things got very smooshy. Some quotes: “And that’s all you have to do in mathematics: imagine it.” “Small does not apply to distance.” “So my best way to do that is draw it like a sausage.” “I don’t think I should explain this picture.” “First I’ll confuse you more.” “It can be done in cricket, and we hope it can be done in Twistor theory too.” There was an interesting aside about Cohomology: a precise nonlocal measure about the degree of impossibility; I think I perked up when I saw the Escher picture. Not even the physicists think Twistor Theory reflects reality. It’s a mathematical plaything.

On a completely unrelated note, that night RAG blind date was going on.

### Conor McBride: The Kleisli Arrows of Outrageous Fortune

“Consider the following program, which is, alas, Poor!” With a slide of code containing variables `b` and `c`: “b or not b, that is the question! ... or to take arms against a c of troubles.” “The world’s most successful dependently typed language: Haskell.” Wave hello to Simon Peyton-Jones in the room.

I’ve been meaning to write a blog post about this talk, and maybe still will, although the talk is no longer as fresh in my mind. Conor describes the technical machinery necessary to simulate dependent types even when you can’t obviously push values into the types. Programs should instead be strategy trees, which cover all possible responses (even though reality will select one.) We don’t need dependent types: we can use parametrized monads to encode the pre and post conditions of Hoare logic. (Resulting in a predictable world.) These are the so called “braces of upwards mobility” (upwards mobility referring to values becoming types.) This talk made me wonder if the Strathclyde Haskell Extension would be a good platform for session types, which suffer tremendously from the lack of true dependent types. (It also made me wonder about *efficient* type-level computation.)

The devil is ∀ (see the horns), e.g. a universal quantifier in the wrong place (I thought maybe you could view them as skolem variables). Terms hold evidence on types. (Typical Conor to make a joke about that.) Kleisli Arrow is the Hoare Triple (think about the programmable semicolon), and with bind you don’t get to choose what state you end up in (there’s an existential.) However, we can use an alternative bind which forces the demon to give us the value: Atkey parametrized monads. We also need to make sure that our naive free monad doesn’t blow up our runtime. This is not very coherent, and cleaning up this story was what I needed to do before making a real post.

Conor’s main message was that data are witness to the proofs! “I’m in the witness protection program.” If we have some piece of data, we have discharged some proof obligations. (But I wonder, what about polymorphism?)

### Petri nets for concurrency

We want performance nondeterminism, so we use petrinets to encode the nondeterminism contracts, and then we push a flow through the graph to figure out what kind of performance we need. Scheduling and placement is easy to handle, because they are just modifications on the petri net. But I wasn’t so sure if this approach would work, because petri nets can get pretty bit and you may not get the performance you need to do interesting analysis.

### Grant Passmore: Strategy in Computer Algebra

Automated reasoning problems are undecidable, and user-guided. Computer algebra problems are unfeasible, but the algorithms tend to be black box, secret sauce solutions (e.g. Mathematica) with all sorts of tirkcs like Groebner Bases, Quantifier Elimination, SAT for nonlinear complex arithmetic (Hilbert’s Weak Nullstellensatz) and reduction to rewriting.

Grant wants to put the choice back in computer algebra. There are many choice points: quadratic preprocessing, choice of pairs, growth of basis, forward simplification. He gives a comparison of ATP and GB. With LCF, functional parameters are key. Bag of tricks is OK, but the proof procedure is a strategy.

1.  What about an algorithm can be changed while preserving correctness?
2.  What arbitrary choices are my algorithm making?

(Margin note: can we understand fusion by understanding the mutable state it creates?)

### (Untitled Software Engineering talk)

A persistent problem with formal methods is they operate in a domain of assumptions and requirements, and there is no way to tell if these assumptions or requirements are right! It’s a rather philosophical question: can we have “recognized assurance deficits”: a known unknown? These are potential sources of counter-evidence. What are the worst implications of not knowing? What claims have you made? (It would be interesting to see what elements of philosophy are relevant here.) Sometimes safety arguments are just assertions “because I say so”: there is epistemic uncertainty. The talk giver argues we should swap probabilistic integrity arguments with qualitative confidence arguments: it’s a question of safety versus confidence. We should explicitly acknowledge assurance deficits, and stop taking too much on trust.

I ask the question: is there incentive for this debate to happen? (Engineers want to look good.) Giving it to evaluators is certainly worse, since they are not in a proper position to assess the system. He didn’t have an answer, but said, “Not doing so is unacceptable.”

I somewhat amusedly note that near the end he pushed some technical software for handling this, ostensibly a product of his own research program. I remain unsure about a technical solution to this particular problem.

### Approximating Markov Chains

Labeled Markov processes encode continuous state spaces, and for the longest time, we’ve noted that bisimulation works for discrete state spaces, but not continuous ones. We view the final state as a probability distribution, and we press buttons on our machine (labels) to change the distribution: each label is a stochastic kernel (a generalized binary relation.) Of course, reasoning about continuous state spaces is important: they cover things like Brownian motion, performance, probabilistic process algebras with recursion, hybrid control systems (flight management systems), as well complex discrete phenomena such as population growth and stock prices.

I didn’t understand most of the technical details, but the main point was that co-bisimulation was the correct way to do this: it is only a coincidence that bisimulation works for discrete systems. The projective limit is exactly the smallest bisimilar process. Also some material on metric spaces. This certainly seems like a field where we’re just importing well-known results from esoteric fields of mathematics.

### Financial Cryptography

Unbeknown to Americans, whose financial system is still stuck on magnetic stripe readers, bank cards in Europe have moved on to EMV, in which a mini-processor is embedded in your chip, which can do true challenge-response authentication. This talk looks at how we might bootstrap a P2P transaction system that bypasses banks, using the existing EMV hardware.

How might we do this? We can use a device called the CAP, which has a small display that when given a pin gives an authentication code (two factor.) We associate transactions with CAP code, so the merchant has agreed to receive money. But you still need collaboration from the banks to do this. (This is the SDA method.) So we get rid of the bank all-together and use Diffie-Hellman (DDA). The cards are simply convenient existing infrastructure to get name authentication. There are some technical details, since we can only sign 32-bits at a time, and we usually need a bit more than that. (Margin note: “the threat model was privacy advocates.”)

The talk was short, so afterwards we had a discussion why this scheme wouldn’t actually work. My objection was that it banks would simply make it illegal to use their bank cards in this way. There was a discussion whether or not they could technically enforce this: Marcus Kuhn used passports as an example, where you can’t read passports if you don’t have Internet access, since the passport itself has an embedded monotonic clock that refuses to give information to the scanner if the scanner software is not up to date. How does the passport know what the latest version is? Its clock gets updated when it sees a new scanner.) Passport security technology is pretty interesting! They invented a block cipher over the alphabet for this purpose.

### Verifying the validity of QBF

SAT is exponential, and when you add quantifiers to the mix, you get “another” exponential, but this time in the certificate. How do you verify that a universally quantified formula is in fact true? The certificate in this case is extension variables and witnesses: we provide concrete implementations for all existentially quantified variables, which can then be substituted in to give a traditional SAT problem. So once we have a certificate, what was a PSPACE problem is now “just” a NP problem.

(Technical note: We want to eliminate hypotheses in topological order (using Refl, Gen and Exists). Get the quantifiers in the right order, witnesses depend on existential variables, extension variables depend on it.)

The talk described how he hooked into the internals of Squolem to get actually get these certificates. It turned out that de Bruijn was faster than name carrying (which is different from typical QBF invalidity checking.) He even found a bug in the non-LCF style validator (due to a lack of a cyclic check.)

Applications: model checking (bounded and unbounded), PSPACE problems. (Margin note: "comparison of BDD and QBF?")

This concludes notebook two.