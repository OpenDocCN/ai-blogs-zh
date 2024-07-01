<!--yml
category: 未分类
date: 2024-07-01 18:17:15
-->

# Haskell for Coq programmers : ezyang’s blog

> 来源：[http://blog.ezyang.com/2014/03/haskell-for-coq-programmers/](http://blog.ezyang.com/2014/03/haskell-for-coq-programmers/)

So you may have heard about this popular new programming language called Haskell. What's Haskell? Haskell is a non-dependently typed programming language, sporting general recursion, type inference and built-in side-effects. It is true that dependent types are considered an essential component of modern, expressive type systems. However, giving up dependence can result in certain benefits for other aspects of software engineering, and in this article, we'd like to talk about the omissions that Haskell makes to support these changes.

### Syntax

There are a number of syntactic differences between Coq and Haskell, which we will point out as we proceed in this article. To start with, we note that in Coq, typing is denoted using a single colon (`false : Bool`); in Haskell, a double colon is used (`False :: Bool`). Additionally, Haskell has a syntactic restriction, where constructors must be capitalized, while variables must be lower-case.

Similar to my [OCaml for Haskellers](http://blog.ezyang.com/2010/10/ocaml-for-haskellers/) post, code snippets will have the form:

```
(* Coq *)

```

```
{- Haskell -}

```

### Universes/kinding

A universe is a type whose elements are types. They were originally introduced to constructive type theory by Per Martin-Löf. Coq sports an infinite hierarchy of universes (e.g. `Type (* 0 *) : Type (* 1 *)`, `Type (* 1 *) : Type (* 2 *)`, and so forth).

Given this, it is tempting to draw an analogy between universes and Haskell’s kind of types `*` (pronounced “star”), which classifies types in the same way `Type (* 0 *)` classifies primitive types in Coq. Furthermore, the sort *box* classifies kinds (`* : BOX`, although this sort is strictly internal and cannot be written in the source language). However, the resemblance here is only superficial: it is misleading to think of Haskell as a language with only two universes. The differences can be summarized as follows:

1.  In Coq, universes are used purely as a sizing mechanism, to prevent the creation of types which are too big. In Haskell, types and kinds do double duty to enforce the *phase distinction*: if `a` has kind `*`, then `x :: a` is guaranteed to be a runtime value; likewise, if `k` has sort box, then `a :: k` is guaranteed to be a compile-time value. This structuring is a common pattern in traditional programming languages, although knowledgeable folks like [Conor McBride](https://twitter.com/pigworker/status/446784239754022912) think that ultimately this is a design error, since one doesn’t [really need a kinding system to have type erasure.](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.105.788&rep=rep1&type=pdf)
2.  In Coq, universes are cumulative: a term which has type `Type (* 0 *)` also has type `Type (* 1 *)`. In Haskell, there is no cumulativity between between types and kinds: if `Nat` is a type (i.e. has the type `*`), it is not automatically a kind. However, in some cases, partial cumulativity can be achieved using [datatype promotion](http://www.haskell.org/ghc/docs/latest/html/users_guide/promotion.html), which constructs a separate kind-level replica of a type, where the data constructors are now type-level constructors. Promotion is also capable of promoting type constructors to kind constructors.
3.  In Coq, a common term language is used at all levels of universes. In Haskell, there are three distinct languages: a language for handling base terms (the runtime values), a language for handling type-level terms (e.g. types and type constructors) and a language for handling kind-level terms. In some cases this syntax is overloaded, but in later sections, we will often need to say how a construct is formulated separately at each level of the kinding system.

One further remark: `Type` in Coq is predicative; in Haskell, `*` is *impredicative*, following the tradition of System F and other languages in the lambda cube, where kinding systems of this style are easy to model.

### Function types

In Coq, given two types `A` and `B`, we can construct the type `A -> B` denoting functions from A to B (for A and B of any universe). Like Coq, functions with multiple arguments are natively supported using currying. Haskell supports function types for both types (`Int -> Int`) and kinds (`* -> *`, often called *type constructors*) and application by juxtaposition (e.g. `f x`). (Function types are subsumed by pi types, however, we defer this discussion for later.) However, Haskell has some restrictions on how one may construct functions, and utilizes different syntax when handling types and kinds:

For *expressions* (with type `a -> b` where `a, b :: *`), both direct definitions and lambdas are supported. A direct definition is written in an equational style:

```
Definition f x := x + x.

```

```
f x = x + x

```

while a lambda is represented using a backslash:

```
fun x => x + x

```

```
\x -> x + x

```

For *type families* (with type `k1 -> k2` where `k1` and `k2` are kinds), the lambda syntax is not supported. In fact, no higher-order behavior is permitted at the type-level; while we can directly define appropriately kinded type functions, at the end of the day, these functions must be fully applied or they will be rejected by the type-checker. From an implementation perspective, the omission of type lambdas makes type inference and checking much easier.

1.  *Type synonyms*:

    ```
    Definition Endo A := A -> A.

    ```

    ```
    type Endo a = a -> a

    ```

    Type synonyms are judgmentally equal to their expansions. As mentioned in the introduction, they cannot be partially applied. They were originally intended as a limited syntactic mechanism for making type signatures more readable.

2.  *Closed type (synonym) families*:

    ```
    Inductive fcode :=
      | intcode : fcode
      | anycode : fcode.
    Definition interp (c : fcode) : Type := match c with
      | intcode -> bool
      | anycode -> char
    end.

    ```

    ```
    type family F a where
      F Int = Bool
      F a   = Char

    ```

    While closed type families look like the addition of typecase (and would violate parametricity in that case), this is not the case, as closed type families can only return types. In fact, closed type families correspond to a well-known design pattern in Coq, where one writes inductive data type representing *codes* of types, and then having an interpretation function which interprets the codes as actual types. As we have stated earlier, Haskell has no direct mechanism for defining functions on types, so this useful pattern had to be supported directly in the type families functionality. Once again, closed type families cannot be partially applied.

    In fact, the closed type family functionality is a bit more expressive than an inductive code. In particular, closed type families support *non-linear pattern matches* (`F a a = Int`) and can sometimes reduce a term when no iota reductions are available, because some of the inputs are not known. The reason for this is because closed type families are “evaluated” using unification and constraint-solving, rather than ordinary term reduction as would be the case with codes in Coq. Indeed, nearly all of the “type level computation” one may perform in Haskell, is really just constraint solving. Closed type families are not available in a released version of GHC (yet), but there is a [Haskell wiki page describing closed type families in more detail](http://www.haskell.org/haskellwiki/GHC/Type_families#Closed_family_simplification).

3.  *Open type (synonym) families*:

    ```
    (* Not directly supported in Coq *)

    ```

    ```
    type family F a
    type instance F Int = Char
    type instance F Char = Int

    ```

    Unlike closed type families, open type families operate under an open universe, and have no analogue in Coq. Open type families do not support nonlinear matching, and must completely unify to reduce. Additionally, there are number of restrictions on the left-hand side and right-hand side of such families in order maintain decidable type inference. The section of the GHC manual [Type instance declarations](http://www.haskell.org/ghc/docs/latest/html/users_guide/type-families.html#type-instance-declarations) expands on these limitations.

Both closed and type-level families can be used to implement computation at the type-level of data constructors which were lifted to the type-level via promotion. Unfortunately, any such algorithm must be implemented twice: once at the expression level, and once at the type level. Use of metaprogramming can alleviate some of the boilerplate necessary; see, for example, the [singletons](https://hackage.haskell.org/package/singletons) library.

### Dependent function types (Π-types)

A Π-type is a function type whose codomain type can vary depending on the element of the domain to which the function is applied. Haskell does not have Π-types in any meaningful sense. However, if you only want to use a Π-type solely for polymorphism, Haskell does have support. For polymorphism over types (e.g. with type `forall a : k, a -> a`, where `k` is a kind), Haskell has a twist:

```
Definition id : forall (A : Type), A -> A := fun A => fun x => x.

```

```
id :: a -> a
id = \x -> x

```

In particular, the standard notation in Haskell is to omit both the type-lambda (at the expression level) and the quantification (at the type level). The quantification at the type level can be recovered using the explicit universal quantification extension:

```
id :: forall a. a -> a

```

However, there is no way to directly explicitly state the type-lambda. When the quantification is not at the top-level, Haskell requires an explicit type signature with the quantification put in the right place. This requires the rank-2 (or rank-n, depending on the nesting) polymorphism extension:

```
Definition f : (forall A, A -> A) -> bool := fun g => g bool true.

```

```
f :: (forall a. a -> a) -> Bool
f g = g True

```

Polymorphism is also supported at the kind-level using the [kind polymorphism extension](http://www.haskell.org/ghc/docs/latest/html/users_guide/kind-polymorphism.html). However, there is no explicit forall for kind variables; you must simply mention a kind variable in a kind signature.

Proper dependent types cannot be supported directly, but they can be simulated by first promoting data types from the expression level to the type-level. A runtime data-structure called a *singleton* is then used to refine the result of a runtime pattern-match into type information. This pattern of programming in Haskell is not standard, though there are recent academic papers describing how to employ it. One particularly good one is [Hasochism: The Pleasure and Pain of Dependently Typed Haskell Program](https://personal.cis.strath.ac.uk/conor.mcbride/pub/hasochism.pdf), by Sam Lindley and Conor McBride.

### Product types

Coq supports cartesian product over types, as well as a nullary product type called unit. Very similar constructs are also implemented in the Haskell standard library:

```
(true, false) : bool * bool
(True, False) :: (Bool, Bool)

```

```
tt : unit
() :: ()

```

Pairs can be destructed using pattern-matching:

```
match p with
  | (x, y) => ...
end

```

```
case p of
  (x, y) -> ...

```

Red-blooded type theorists may take issue with this identification: in particular, Haskell’s default pair type is what is considered a *negative* type, as it is lazy in its values. (See more on [polarity](http://existentialtype.wordpress.com/2012/08/25/polarity-in-type-theory/).) As Coq’s pair is defined inductively, i.e. positively, a more accurate identification would be with a strict pair, defined as `data SPair a b = SPair !a !b`; i.e. upon construction, both arguments are evaluated. This distinction is difficult to see in Coq, since positive and negative pairs are logically equivalent, and Coq does not distinguish between them. (As a total language, it is indifferent to choice of evaluation strategy.) Furthermore, it's relatively common practice to extract pairs into their lazy variants when doing code extraction.

### Dependent pair types (Σ-types)

Dependent pair types are the generalization of product types to be dependent. As before, Σ-types cannot be directly expressed, except in the case where the first component is a type. In this case, there is an encoding trick utilizing data types which can be used to express so-called *existential types*:

```
Definition p := exist bool not : { A : Type & A -> bool }

```

```
data Ex = forall a. Ex (a -> Bool)
p = Ex not

```

As was the case with polymorphism, the type argument to the dependent pair is implicit. It can be specified explicitly by way of an appropriately placed type annotation.

### Recursion

In Coq, all recursive functions must have a structurally decreasing argument, in order to ensure that all functions terminate. In Haskell, this restriction is lifted for the expression level; as a result, expression level functions may not terminate. At the type-level, by default, Haskell enforces that type level computation is decidable. However, this restriction can be lifted using the `UndecidableInstances` flag. It is generally believed that undecidable instances cannot be used to cause a violation of type safety, as nonterminating instances would simply cause the compiler to loop infinitely, and due to the fact that in Haskell, types cannot (directly) cause a change in runtime behavior.

### Inductive types/Recursive types

In Coq, one has the capacity to define inductive data types. Haskell has a similar-looking mechanism for defining data types, but there are a number of important differences which lead many to avoid using the moniker *inductive data types* for Haskell data types (although it’s fairly common for Haskellers to use the term anyway.)

Basic types like boolean can be defined with ease in both languages (in all cases, we will use the [GADT syntax](http://www.haskell.org/ghc/docs/latest/html/users_guide/data-type-extensions.html#gadt) for Haskell data-types, as it is closer in form to Coq’s syntax and strictly more powerful):

```
Inductive bool : Type :=
  | true : bool
  | false : bool.

```

```
data Bool :: * where
  True :: Bool
  False :: Bool

```

Both also support recursive occurrences of the type being defined:

```
Inductive nat : Type :=
  | z : nat
  | s : nat -> nat.

```

```
data Nat :: * where
  Z :: Nat
  S :: Nat -> Nat

```

One has to be careful though: our definition of `Nat` in Haskell admits one more term: infinity (an infinite chain of successors). This is similar to the situation with products, and stems from the fact that Haskell is lazy.

Haskell’s data types support parameters, but these parameters may only be types, and not values. (Though, recall that data types can be promoted to the type level). Thus, the standard type family of vectors may be defined, assuming an appropriate type-level nat (as usual, explicit forall has been omitted):

```
Inductive vec (A : Type) : nat -> Type :=
  | vnil  : vec A 0
  | vcons : forall n, A -> vec A n -> vec A (S n)

```

```
data Vec :: Nat -> * -> * where
  VNil  :: Vec Z a
  VCons :: a -> Vec n a -> Vec (S n) a

```

As type-level lambda is not supported but partial application of data types is (in contrast to type families), the order of arguments in the type must be chosen with care. (One could define a type-level flip, but they would not be able to partially apply it.)

Haskell data type definitions do not have the [strict positivity requirement,](http://blog.ezyang.com/2012/09/y-combinator-and-strict-positivity/) since we are not requiring termination; thus, peculiar data types that would not be allowed in Coq can be written:

```
data Free f a where
   Free :: f (Free f a) -> Free f a
   Pure :: a -> Free f a

data Mu f where
   Roll :: f (Mu f) -> Mu f

```

### Inference

Coq has support for requesting that a term be inferred by the unification engine, either by placing an underscore in a context or by designating an argument as *implicit* (how one might implement in Coq the omission of type arguments of polymorphic functions as seen in Haskell). Generally, one cannot expect all inference problems in a dependently typed language to be solvable, and the inner-workings of Coq’s unification engines (plural!) are considered a black art (no worry, as the trusted kernel will verify that the inferred arguments are well-typed).

Haskell as specified in Haskell'98 enjoys principal types and full type inference under Hindley-Milner. However, to recover many of the advanced features enjoyed by Coq, Haskell has added numerous extensions which cannot be easily accomodated by Hindley-Milner, including type-class constraints, multiparameter type classes, GADTs and type families. The current state-of-the-art is an algorithm called [OutsideIn(X)](http://research.microsoft.com/en-us/um/people/simonpj/papers/constraints/jfp-outsidein.pdf). With these features, there are no completeness guarantee. However, if the inference algorithm accepts a definition, then that definition has a principal type and that type is the type the algorithm found.

### Conclusion

This article started as a joke over in OPLSS'13, where I found myself explaining some of the hairier aspects of Haskell’s type system to Jason Gross, who had internalized Coq before he had learned much Haskell. Its construction was iced for a while, but later I realized that I could pattern the post off of the first chapter of the homotopy type theory book. While I am not sure how useful this document will be for learning Haskell, I think it suggests a very interesting way of mentally organizing many of Haskell’s more intricate type-system features. Are proper dependent types simpler? Hell yes. But it’s also worth thinking about where Haskell goes further than most existing dependently typed languages...

### Postscript

Bob Harper [complained over Twitter](http://storify.com/ezyang/bob-harper-comments-on-haskell-for-coq-programmers) that this post suggested misleading analogies in some situations. I've tried to correct some of his comments, but in some cases I wasn't able to divine the full content of his comments. I invite readers to see if they can answer these questions:

1.  Because of the phase distinction, Haskell’s *type families* are not actually type families, in the style of Coq, Nuprl or Agda. Why?
2.  This post is confused about the distinction between elaboration (type inference) and semantics (type structure). Where is this confusion?
3.  Quantification over kinds is not the same as quantification over types. Why?