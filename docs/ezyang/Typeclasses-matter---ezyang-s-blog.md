<!--yml
category: 未分类
date: 2024-07-01 18:18:29
-->

# Typeclasses matter : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/01/typeclasses-matter/](http://blog.ezyang.com/2010/01/typeclasses-matter/)

## Typeclasses matter

Typeclasses matter. In fact, I'll go as far to say that they have the capacity to [replace](http://www.haskell.org/haskellwiki/OOP_vs_type_classes) what is traditional object-oriented programming. To understand why, however, we have to review the traditionally recognized benefits of object-oriented programming:

*   *Organization.* For C-inspired languages that don't have a module system, this is so incredibly important; without a discipline for organizing code finding the location of any given function is difficult unless you are intimately familiar with the problem domain. With object-oriented programming, all of these aspects are obvious: classes map into obvious filenames, methods go in obvious places, and overall organization is a function of how well the object model is designed, not how well thought out the include files were.
*   *Encapsulation.* Objects were the first widely utilized method of hiding data and code from clients. Declare something `private` or `protected`, and you have compile-time guarantees that your clients won't have their grubby fingers all over your innards. Used properly, *modularity* follows.
*   *Polymorphism.* The ability to change behavior based on data is a powerful idea dating back to the days of `(void *)`, which can lead to incomprehensible code flow but more often is a more elegant and concise way of writing complicated interactions than a giant `switch` statement. These benefits compound in situations that *multiple dispatch* is appropriate, and *interfaces* can lead to compile-time assurances that a particular class does what it says it does.
*   *Inheritance.* While a problematic facet of object-oriented programming (especially when manifest as *multiple inheritance*), inheritance is still an extremely powerful mechanism of code reuse in object-oriented designs. Subclasses get a default implementation for methods, as well as the ability to break through a level of encapsulation and use `protected` methods.

Typeclasses directly fulfill some of these requirements, while others are achieved due to Haskell's strict types and module system.

*   *Organization.* At first blush, this seems strictly worse: we can no longer read off class name and find the right file. However, a combination of `ghci`, which lets you run `:info` to find the location of any declaration in scope, as well as [Hoogle](http://haskell.org/hoogle/), which lets you find the function you want from just a type signature. These capabilities make it incredibly easy not only to find functions that you know exist, but also find ones you don't know exist. Static typing to the rescue!
*   *Encapsulation.* This feature is implemented by Haskell's module export system: essentially, if you don't export a constructor for any given data type, the end user cannot create or introspect inside that type; they have to use the functions you define to manipulate them. Additionally, if functions specify that their input types should be instances of a typeclass, there is a statically-checked type guarantee that the function will only use functions defined by the typeclass (i.e. no unsafe downcasts).
*   *Polymorphism.* This is the most obvious application of typeclasses; when explaining them to those coming in from imperative languages, the most common analogy made is that typeclasses are like interfaces. They are far more expressive than interfaces, however: functions can trivially specify that an incoming data type must satisfy multiple typeclasses, and parametrized types (those with extra type variables in their declaration) can have type classes constraining their type parameters. Furthermore, code can be written to be fully general over a typeclass, to be instantiated later down the line once an explicit type is inferred.
*   *Inheritance.* Interface inheritance is a straightforward subset of type parametrization; instead of saying `class Monad m`, we say `class Functor m => Monad m`, thus stating that any `m` with a Monad instance must also have a Functor instance (and thus we may freely use any Monad as if it were a Functor). The ability to specify default implementations (often self referential, as `x /= y = not (x == y)` and `x == y = not (x /= y)` of the Eq class attest) goes a long way to make writing new instances easy.

Classic object hierarchies are an excellent mechanism for modelling "is a" relationships, but very few things in this world are actually cleanly "is a", as opposed to "acts like a"; and inheritance has been abused by many developers who have created large object hierarchies (cough GUI toolkits cough), when really, all that is really being exercised is inheritance's code reuse mechanism. The emphasis on typeclasses/interfaces gets back to the heart of the problem:

> What can I do with this type?

No more, no less.