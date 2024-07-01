<!--yml
category: 未分类
date: 2024-07-01 18:18:27
-->

# Scheming environments : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/02/scheming-environment/](http://blog.ezyang.com/2010/02/scheming-environment/)

Environments are [first-class objects in MIT/GNU Scheme](http://www.gnu.org/software/mit-scheme/documentation/mit-scheme-ref/Environment-Operations.html#Environment-Operations). This is neat, because an integral part of the lexical structure of a Scheme is also a data-structure in its own right, able to encode data and behavior. In fact, the environment data structure is precisely what Yegge calls [property lists](http://steve-yegge.blogspot.com/2008/10/universal-design-pattern.html), maps that can be linked up with inheritance. So not only is it a data structure, it's a highly general one too.

Even without the ability to pass around an environment as a first class variable, you can still leverage its syntactic ties to [stash away local state](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-21.html#%_sec_3.2.3) in an object-oriented manner. The data is only accessible inside procedures that pointed to the appropriate environment frame, and traditionally the closure returns a lambda (with its double-bubble pointed to the newly born environment frame) that acts as the only interface into the enclosing state. This requires a fair amount of boilerplate, since this lambda has to support every possible operation you might want to do to the innards of the function.

With a first class environment object, however, you can futz around with the closure's bindings arbitrarily. Unfortunately, there's no way to directly `get-current-environent` (except for the top-level REPL environment, which doesn't count), so we resort to the following trick:

```
(procedure-environment (lambda () '()))

```

`procedure-environment` can grab the environment pointer from the double-bubble for some procedure. So, we force the creation of a double-bubble pointing to the environment we care about with the empty lambda.

I most recently used this technique in a 6.945 project, in which I used a lambda to generate a bunch of procedures with various parameters swapped out (encouraging code-reuse), something akin to the time-honored trick of including C files multiple times with different macro definitions. Instead of returning these procedures as a hash-table which then people would have to explicitly call, I just returned the environment, and thus any consumer could enter "a different universe" by using an appropriate environment.

Scheme is pretty unique in its celebration of environments as first-class objects. I could try this trick in Python, but the `func_closure` attribute on functions is read-only, and also Python has some pretty lame scoping rules. A shame, since this technique allows for some lovely syntactic simplifications.

*Comment.* Oleg Kiselyov writes in to mention that "*MIT Scheme* specifically is unique in its celebration of environments as first class environments," and notes that even some developers of MIT scheme have [second](http://people.csail.mit.edu/gregs/ll1-discuss-archive-html/msg03947.html) [thoughts](http://www.mail-archive.com/r6rs-discuss@lists.r6rs.org/msg01137.html) about the feature. It makes code difficult to optimize, and is both theoretically and practically dangerous: theoretically dangerous since environments are really an implementation detail, and practically dangerous because it makes it very difficult to reason about code.

From in-person discussions, I'm not surprised that Sussman's favored dialect of Scheme allows such a dangerous feature; Sussman has always been in favor of letting people have access to dangerous toys and trusting them to use them correctly.