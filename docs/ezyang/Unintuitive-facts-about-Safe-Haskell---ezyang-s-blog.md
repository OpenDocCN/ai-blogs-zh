<!--yml
category: 未分类
date: 2024-07-01 18:17:27
-->

# Unintuitive facts about Safe Haskell : ezyang’s blog

> 来源：[http://blog.ezyang.com/2012/09/common-misconceptions-about-safe-haskell/](http://blog.ezyang.com/2012/09/common-misconceptions-about-safe-haskell/)

## Unintuitive facts about Safe Haskell

[Safe Haskell](http://www.haskell.org/ghc/docs/7.4.2/html/users_guide/safe-haskell.html) is a new language pragma for GHC which allows you to run untrusted code on top of a trusted code base. There are some common misconceptions about how Safe Haskell works in practice. In this post, I’d like to help correct some of these misunderstandings.

### [`system 'rm -Rf /' :: IO ExitCode`] is accepted by Safe Haskell

Although an IO action here is certainly unsafe, it is not rejected by Safe Haskell per se, because the type of this expression clearly expresses the fact that the operation may have arbitrary side effects. Your obligation in the trusted code base is to not run untrusted code in the IO monad! If you need to allow limited input/output, you must define a restricted IO monad, which is described in the manual.

### Safe Haskell programs can hang

Even with `killThread`, it is all to easy to permanently tie up a capability by creating a [non-allocating infinite loop](http://hackage.haskell.org/trac/ghc/ticket/367). This bug has been open for seven years now, but we consider this a major deficiency in Safe Haskell, and are looking for ways to prevent this from occurring. But as things are now, Safe Haskell programs need to be kept under check using operating system level measures, rather than just Haskell's thread management protocols.

### Users may mistrust `Trustworthy` modules

The `Trustworthy` keyword is used to mark modules which use unsafe language features and/or modules in a “safe” way. The safety of this is vouched for by the maintainer, who inserts this pragma into the top of the module file. Caveat emptor! After all, there is no reason that you should necessarily believe a maintainer who makes such a claim. So, separately, you can trust a package, via the `ghc-pkg` database or the `-trust` flag. But GHC also allows you to take the package maintainer at their word, and in fact does so by default; to make it distrustful, you have to pass `-fpackage-trust`. The upshot is this:

| Module trusted? | (no flags) | `-fpackage-trust` |
| --- | --- | --- |
| Package untrusted | Yes | No |
| Package trusted | Yes | Yes |

If you are serious about using Safe Haskell to run untrusted code, you should always run with `-fpackage-trust`, and carefully confer trusted status to packages in your database. If you’re just using Safe Haskell as a way of enforcing code style, the default are pretty good.

### Explicit untrust is important for maintaining encapsulation

Safe Haskell offers safety inference, which automatically determines if a module is safe by checking if it would compile with the `-XSafe` flag. Safe inferred modules can then be freely used by untrusted code. Now, suppose that this module (inferred safe) was actually `Data.HTML.Internal`, which exported constructors to the inner data type which allowed a user to violate internal invariants of the data structure (e.g. escaping). That doesn’t seem very safe!

The sense in which this is safe is subtle: the correctness of the trusted code base cannot rely on any invariants supplied by the untrusted code. For example, if the untrusted code defines its own buggy implementation of a binary tree, catching the bugginess of the untrusted code is out of scope for Safe Haskell’s mission. But if our TCB expects a properly escaped `HTML` value with no embedded JavaScript, the violation of encapsulation of this type could mean the untrusted code could inject XSS.

David Terei and I have some ideas for making the expression of trust more flexible with regards to package boundaries, but we still haven't quite come to agreement on the right design. (Hopefully we will soon!)

### Conclusion

Safe Haskell is at its heart a very simple idea, but there are some sharp edges, especially when Safe Haskell asks you to make distinctions that aren't normally made in Haskell programs. Still, Safe Haskell is rather unique: while there certainly are widely used sandboxed programming languages (Java and JavaScript come to mind), Safe Haskell goes even further, and allows you to specify your own, custom security policies. Combine that with a massive ecosystem of libraries that play well with this feature, and you have a system that you really can’t find anywhere else in the programming languages universe.