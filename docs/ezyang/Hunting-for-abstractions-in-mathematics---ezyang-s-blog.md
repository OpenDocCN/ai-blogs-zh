<!--yml
category: 未分类
date: 2024-07-01 18:18:24
-->

# Hunting for abstractions in mathematics : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/03/abstractions-in-mathematics/](http://blog.ezyang.com/2010/03/abstractions-in-mathematics/)

> **Abstraction** (n.) The act or process of separating in thought, of considering a thing independently of its associations; or a substance independently of its attributes; or an attribute or quality independently of the substance to which it belongs. (Oxford English Dictionary)

Abstraction is one of the most powerful beasts in the landscape of programming, but it is also one of the most elusive to capture. The places where abstraction may be found are many:

*   *Good artists copy. Great artists steal.* Abstractions that other people have (re)discovered, (re)used and (re)implemented are by far the easiest to find.
*   *First time you do something, just do it. Second time, wince at duplication. Third time, you refactor.* Refactoring introduces small pieces of ad hoc abstraction. The quality varies though: the result might be something deeper, but it might just be mundane code reuse.
*   *Grow your framework.* A lengthy process where you build the abstraction and the application, build another distinct application on the abstraction, and reconcile. This takes a lot of time, and is entirely dependent on people's willingness to break BC and make sweeping changes.
*   *Give the problem to a really smart person.* Design is a creative process, [design by committee](http://en.wikipedia.org/wiki/Design_by_committee) results in moldy code. A single person unifies the abstraction and picks the battles to fight when an abstraction needs to change.
*   *Turn to nature.* User interfaces often introduce a form of abstraction, and it's common to turn to real life and see what's there. Note that there are really good abstractions that don't exist in real-life: the concept of undo is positively ridiculous in the real world, but probably one of the best inventions in computers.

I'd like to propose one other place to turn when you're hunting for abstractions: pure mathematics.

*"Pure mathematics?"*

Yes, pure mathematics. Not applied mathematics, which easily finds its place in a programmers toolbox for tackling specific classes of problems.

*"Ok, mathematicians may do neat things... but they're too theoretical for my taste."*

But that's precisely what we're looking for! Pure mathematics is all about manipulating abstract objects and deductively proving properties about them. The mathematicians aren't talking about a different kind of abstraction, they're just starting off from different concrete objects than a programmer might be. The mathematicians have just have been doing it much longer than programmers (set the mark at about 600BC with the Greeks.) Over this period of time, mathematicians have gotten pretty damn good at creating abstractions, handling abstractions, deducing properties of abstractions, finding relationships between abstractions, and so forth. In fact, they're so good that advanced mathematics has a reputation for abstracting concepts way beyond what any "normal" person would tolerate: Lewis Carroll is one prominent figure known for satirizing what he saw as ridiculous ideas in mid-19th century mathematics.

Of course, you can't take an arbitrary mathematical concept and attempt to shoehorn it into your favorite programming language. The very first step is looking at some abstract object and looking for concrete instances of that object that programmers care about. Even structures that have obvious concrete instances have more subtle instances that are just as useful.

It is also the case that many powerful mathematical abstractions are also unintuitive. Programmers naturally shy away from unintuitive ideas: it is an example of being overly clever. The question of what is intuitive has shaped discussions in mathematics as well: when computability theory was being developed, there were many competing models of computation vying for computer scientists' attentions. Alonzo Church formulated lambda calculus, a highly symbolic and abstract notion of computation; Alan Turing formulated Turing machines, a very physical and concrete notion of computation. In pedagogy, Turing machines won out: open any introductory computability textbook (in my case, [Sipser's](http://www.amazon.com/Introduction-Theory-Computation-Michael-Sipser/dp/053494728X) textbook) and you will only see the Turing machines treated to the classic proofs regarding the undecidability of halting problems. The Turing machine is much simpler to understand; it maps more cleanly to the mental model of a mathematician furiously scribbling away a computation.

But no computer scientist (and certainly not Alonzo Church) would claim that Turing machines are the only useful model to study. Lambda calculus is elegant; after you've wrapped your head around it, you can express ideas and operations concisely which, with Turing machines, would have involved mucking around with encodings and sweeping the head around and a lot of bookkeeping. Writing Turing machines, put bluntly, is a pain in the ass.

I now present two examples of good ideas in mathematics resulting in good ideas in programming.

The first involves the genesis of Lisp. As Sussman tells me, Lisp was originally created so that McCarthy could prove Gödel's incompleteness theorem without having to resort to number theory. (McCarthy is quoted as saying something a little weaker: Lisp was a ["way of describing computable functions much neater than the Turing machines or the general recursive definitions used in recursive function theory."](http://www-formal.stanford.edu/jmc/history/lisp/node3.html)) Instead of taking the statement "this statement cannot be proven" and rigorously encoding in a stiff mathematical formalism, [it could simply be described in Lisp (PDF)](http://publications.csail.mit.edu/lcs/pubs/pdf/MIT-LCS-TR-131.pdf). Thus, its original formulation featured m-expressions, which looked a little like `function[arg1 arg2]` and represented the actual machine, as opposed to the s-expression which was the symbolic representation. It wasn't until later that a few graduate students thought, "Hm, this would actually be a useful programming language," and set about to actually implement Lisp. Through Gödel's incompleteness theorem, the powerful notion of code as data was born: no other language at the time had been thinking about programs in this way.

The second is the success of Category Theory in Haskell. The canonical example is monads, the mathematical innovation that made input/output in lazy languages not suck (although a mathematical friend of mine tells me monads are not actually interesting because they're not general enough.) But the ideas behind the functor and applicative functor encapsulate patterns pervasive in all programming languages. An example of a reformulation of this concept can be seen in numpy's [universal functions](http://docs.scipy.org/doc/numpy/reference/ufuncs.html). They don't call it a functor, instead they use terms such as "broadcasting" and "casting" and discuss the need to use special universal versions of functions on numpy arrays to get element-by-element operations. The interface is usable enough, but it lacks the simplicity, elegance and consistency that you get from actually realizing, "Hey, that's just a functor..."

Those mathematicians, they're smart folk. Perhaps we programmers could learn a thing or two from them.

*Postscript.* Thanks to [Daniel Kane](http://www.math.harvard.edu/~dankane/) for answering my impromptu question "What is mathematics about?" and suggesting a few of the examples of mathematics leaking back into computer engineering.