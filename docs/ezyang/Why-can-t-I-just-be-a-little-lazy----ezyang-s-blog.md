<!--yml
category: 未分类
date: 2024-07-01 18:17:24
-->

# Why can’t I just be a little lazy? : ezyang’s blog

> 来源：[http://blog.ezyang.com/2012/11/why-cant-i-just-be-a-little-lazy/](http://blog.ezyang.com/2012/11/why-cant-i-just-be-a-little-lazy/)

You can. Imagine a version of Haskell where every constructor was strict, e.g. every field had a `!` prefix. The semantics of this language are well defined; and in fact, the fine folks at CMU have known about this for some time:

> Up to this point we have frequently encountered arbitrary choices in the dynamics of various language constructs. For example, when specifying the dynamics of pairs, we must choose, rather arbitrarily, between the lazy dynamics, in which all pairs are values regardless of the value status of their components, and the eager dynamics, in which a pair is a value only if its components are both values. We could even consider a half-eager (or, equivalently, half-lazy) dynamics, in which a pair is a value only if, say, the first component is a value, but without regard to the second.
> 
> Similar questions arise with sums (all injections are values, or only injections of values are values), recursive types (all folds are values, or only folds of values are values), and function types (functions should be called by-name or by-value). Whole languages are built around adherence to one policy or another. For example, Haskell decrees that products, sums, and recursive types are to be lazy, and functions are to be called by name, whereas ML decrees the exact opposite policy. Not only are these choices arbitrary, but it is also unclear why they should be linked. For example, we could very sensibly decree that products, sums, and recursive types are lazy, yet impose a call-by-value discipline on functions. Or **we could have eager products, sums, and recursive types, yet insist on call-by-name.** It is not at all clear which of these points in the space of choices is right; each has its adherents, and each has its detractors.
> 
> Are we therefore stuck in a tarpit of subjectivity? No! The way out is to recognize that these distinctions should not be imposed by the language designer, but rather are choices that are to be made by the programmer. This may be achieved by recognizing that differences in dynamics reflect fundamental type distinctions that are being obscured by languages that impose one policy or another. We can have both eager and lazy pairs in the same language by simply distinguishing them as two distinct types, and similarly we can have both eager and lazy sums in the same language, and both by-name and by-value function spaces, by providing sufficient type distinctions as to make the choice available to the programmer.