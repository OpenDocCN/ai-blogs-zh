<!--yml
category: 未分类
date: 2024-07-01 18:17:39
-->

# The new Reflections on Trusting Trust : ezyang’s blog

> 来源：[http://blog.ezyang.com/2011/10/the-new-reflections-on-trusting-trust/](http://blog.ezyang.com/2011/10/the-new-reflections-on-trusting-trust/)

In his classic essay [Reflections on Trusting Trust](http://cm.bell-labs.com/who/ken/trust.html), Ken Thompson describes a self-replicating compiler bug which is undetectable by source code inspection. The self-replication is made possible by the fact that most compilers are self-compiling: old versions of a compiler are used to compile new ones, and if the old version is malicious, it can slip the same bug when it detects it is compiling itself.

A new trend is precisely this self-hosting process, but for [self-certifying typecheckers](http://research.microsoft.com/en-us/projects/fstar/): typecheckers which are used to prove their own correctness. (Note that these are powerful typecheckers, close to being able to check arbitrary theorems about code.) This may seem a little odd, since I could write a trivial typechecker which always claimed it was correct. In order to work around this, we must bootstrap the correctness proof by proving the typechecker correct in another language (in the case of F*, this language is Coq). Once this has been done, we can then use this verified typechecker to check a specification of itself. This process is illustrated below.

The question then is whether or not such self-certifying typecheckers are similarly vulnerable to the problem Ken described for self-hosting compilers. For arguments sake, let's assume that the backend compiler and runtime are certified (a strong assumption that is almost universally untrue, including for F*). Since the typechecker can't insert malicious bugs into the programs it compiles (it only, you know, typechecks), one would have to rely on a bug in the source code itself. Surely such a bug would be obvious!

This is unclear: we have certified our implementation, but what of our specification? In Coq, we proved various theorems about the soundness and adequacy of our type system, which give us at least some hope that it is correct in the way we expect. But these proofs are nowhere to be seen in the emancipated F* world. If we want to evolve our specification (less plausible for a full-blooded dependently typed language, but within the realm of possibility for a less powerful one), we must turn back to Coq and adjust the relevant theorems. Otherwise, we run the risk of changing our type system to an unsound one.

Fortunately, that's all we have to do: we can use the old F* type checker to certify the new one, rather than attempt to export certificates and reverify with them Coq. All told, though, don't throw out your Coq code yet... not, at least, if you think your type system may change in the future.