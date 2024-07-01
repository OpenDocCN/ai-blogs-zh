<!--yml
category: 未分类
date: 2024-07-01 18:17:29
-->

# Polymorphic variants in Ur/Web : ezyang’s blog

> 来源：[http://blog.ezyang.com/2012/07/polymorphic-variants-in-urweb/](http://blog.ezyang.com/2012/07/polymorphic-variants-in-urweb/)

This document explains how **polymorphic variants** in [Ur/Web](http://www.impredicative.com/ur/) work. It was written because the [official tutorial](http://www.impredicative.com/ur/tutorial/) has no mention of them, the manual only devotes a paragraph to the topic, and there are some useful tricks for dealing with them that I picked up while wrangling with them in [Logitext](http://logitext.mit.edu/main).

### What are polymorphic variants?

Polymorphic variants may be [familiar to OCaml users](http://caml.inria.fr/pub/docs/manual-ocaml/manual006.html): they permit you to use the tags of variants in multiple types, and not just in the original algebraic data type a constructor was defined in. Instead having to keep names unique:

```
datatype yn = Yes1 | No1
datatype ynm = Yes2 | No2 | Maybe2

```

We can just reuse them:

```
con yn = variant [Yes = unit, No = unit]
con ynm = variant [Yes = unit, No = unit, Maybe = unit]

```

This is extremely convenient if you have a lot of constructors which you want to share between multiple logical types. Unfortunately, they have a number of [nasty](http://t-a-w.blogspot.com/2006/05/variant-types-in-ocaml-suck.html) [effects](https://ocaml.janestreet.com/?q=node/99), largely stemming from the fact that they introduce subtyping to the language. In Ur, this inherent subtyping is modulated by the same row types that power Ur’s record system, so handling polymorphic variants is quite like handling records, and both are based off of Ur/Web’s type-level records.

### How do I make a polymorphic variant?

To make a polymorphic variant type, instead of applying the `$` operator, instead apply the `variant` to a type-level record. So:

```
$[A = int, B = bool]

```

generates a record with two fields, `A` containing an `int` and `B` containing a `bool`, whereas:

```
variant [A = int, B = bool]

```

generates a variant with two constructors, `A` containing just an `int`, or `B` containing just a `bool`.

To make a polymorphic variant value, use the `make` function, which requires a label (indicating the constructor) and the value:

```
make [#A] 2

```

Technically, during the construction of a polymorphic variant, you also need to know what the full set of constructors this value will be used with are. Normally Ur/Web will infer this for you, but this is an important restriction which will affect code that operates on variants. The full signature of `make` is this:

```
val make : nm :: Name
        -> t ::: Type
        -> ts ::: {Type}
        -> [[nm] ~ ts]
        => t -> variant ([nm = t] ++ ts)

```

The function of `nm` and `t` should be self-evident, and `ts` is the type-level record for the rest of the values in the variant, concatenated with `[nm = t]` to produce a type-level record which is guaranteed to contain `nm`.

### How do I destruct a polymorphic variant?

Use the `match` function, which takes a variant and a record of functions which indicate how to process each possible constructor of the variant:

```
match t { A = fn a => a + 2,
          B = fn b => if b then 3 else 6 }

```

Indeed, the variant and the record use the *same* type-level record, though the types of the record are a little different, as seen in the type of match:

```
val match : ts ::: {Type}                (* the type-level record *)
         -> t ::: Type
         -> variant ts                   (* the variant *)
         -> $(map (fn t' => t' -> t) ts) (* the record *)
         -> t

```

### What other operations can I perform on variants?

`make` and `match` are the only primitives you need: everything else can derived. However, the [meta](http://hg.impredicative.com/meta/) library has a `Variant` module which contains of useful derived functions for operating with variants. For example, this pair of functions:

```
val read : r ::: {Unit} -> t ::: Type -> folder r
        -> $(mapU t r) -> variant (mapU {} r) -> t
val write : r ::: {Unit} -> t ::: Type -> folder r
        -> $(mapU t r) -> variant (mapU {} r) -> t -> $(mapU t r)

```

Allow you to use variants as labels, to project and edit values from homogeneously typed records. The signatures are not too difficult to read: `r` is the type-level record which defines the variant, `t` is the type of the homogeneous record, `folder r` is a folder for the record (which usually will get inferred), `$(mapU t r)` is the type of a homogeneous record (we didn’t write `$r` because that would be a record containing only unit) and `variant (mapU {} r)` is the variant serving as a “label”. Here are some example uses of some of the simpler functions in this library:

```
read {A = 1, B = 2} (make [#A] ())
== 1

write {A = 1, B = 2} (make [#B] ()) 3
== {A = 1, B = 3}

search (fn v => match v {A = fn () => None, B = fn () => Some 2})
== Some 2

find (fn v => match v {A = fn () => True, B = fn () => False})
== Some (make [#A] ())

test [#A] (make [#A] 2)
== Some 2

weaken (make [#A] 2 : variant [A = int])
== make [#A] 2 : variant [A = int, B = int]

eq (make [#A] ()) (make [#B] ())
== False

mp (fn v => match v {A = fn () => 2, B = fn () => True})
== {A = 2, B = True}

fold (fn v i => match v {A = fn () => i + 1, B = fn () => i + 2}) 0
== 3

mapR (fn v x => match v {A = fn i => i * 2, B = fn i => i * 3}) { A = 2 , B = 3 }
== { A = 4, B = 9 }

```

### What does destrR do?

This function has a somewhat formidable type:

```
val destrR : K --> f :: (K -> Type) -> fr :: (K -> Type) -> t ::: Type
          -> (p :: K -> f p -> fr p -> t)
          -> r ::: {K} -> folder r -> variant (map f r) -> $(map fr r) -> t

```

But really, it’s just a more general `match`. `match` can easily be implemented in terms of `destrR`:

```
match [ts] [t] v fs =
  destrR [ident] [fn p => p -> t] (fn [p ::_] f x => f x) v fs

```

`destrR` affords more flexibility when the record is not quite a function, but a record containing a function, or a type class, or even if the variant is the function and the record the data.

### Is there a more concise way to match over multiple constructors?

Polymorphic variants frequently have a lot of constructors, all of which look basically the same:

```
con tactic a =
  [Cut = logic * a * a,
   LExact = int,
   LConj = int * a,
   LDisj = int * a * a,
   LImp = int * a * a,
   LIff = int * a,
   LBot = int,
   LTop = int * a,
   LNot = int * a,
   LForall = int * universe * a,
   LExists = int * a,
   LContract = int * a,
   LWeaken = int * a,
   RExact = int,
   RConj = int * a * a,
   RDisj = int * a,
   RImp = int * a,
   RIff = int * a * a,
   RTop = int,
   RBot = int * a,
   RNot = int * a,
   RForall = int * a,
   RExists = int * universe * a,
   RWeaken = int * a,
   RContract = int * a]

```

Filling out records to `match` against quickly gets old, especially if the functionality for any two constructors with the same type of data is the same:

```
let fun empty _ = True
    fun single (_, a) = proofComplete a
    fun singleQ (_, _, a) = proofComplete a
    fun double (_, a, b) = andB (proofComplete a) (proofComplete b)
in match t {Cut       = fn (_, a, b) => andB (proofComplete a) (proofComplete b),
            LExact    = empty,
            LBot      = empty,
            RExact    = empty,
            RTop      = empty,
            LConj     = single,
            LNot      = single,
            LExists   = single,
            LContract = single,
            LWeaken   = single,
            LTop      = single,
            RDisj     = single,
            RImp      = single,
            LIff      = single,
            RNot      = single,
            RBot      = single,
            RForall   = single,
            RContract = single,
            RWeaken   = single,
            LForall   = singleQ,
            RExists   = singleQ,
            LDisj     = double,
            LImp      = double,
            RIff      = double,
            RConj     = double
            }
end

```

Adam Chlipala and I developed a nice method for reducing this boilerplate by abusing *local type classes*, which allow us to lean of Ur/Web’s inference engine to automatically fill in the function to handle an element of a particular type. Here is that recursive traversal again, using our new method:

```
let val empty   = declareCase (fn _ (_ : int) => True)
    val single  = declareCase (fn _ (_ : int, a) => proofComplete a)
    val singleQ = declareCase (fn _ (_ : int, _ : Universe.r, a) => proofComplete a)
    val double  = declareCase (fn _ (_ : int, a, b) => andB (proofComplete a) (proofComplete b))
    val cut     = declareCase (fn _ (_ : Logic.r, a, b) => andB (proofComplete a) (proofComplete b))
in typeCase t end

```

For every “type” in the variant, you write a `declareCase` which takes that type and reduces it to the desired return type. (You also get, as the first constructor, a constructor function to create the original constructor; e.g. `declareCase (fn f x => f x)` is the identity transformation. Then you run `typeCase` and watch the magic happen. There are more detailed usage instructions in [variant.urs](http://hg.impredicative.com/meta/file/f55f66c6fdee/variant.urs).

### How do I make the type of my variant bigger?

When writing metaprograms that create variant types, a common problem is that the variant you just created is too narrow: that is, the `ts` in `variant ts` doesn’t have enough entries in it. This is especially a problem when `ts` is the record you are folding over. Consider a simple example where we would like to write this function, which generates a record of constructors for each constructor of the variant:

```
fun ctors : ts ::: {Type} -> fl : folder ts -> $(map (fn t => t -> variant ts) ts)

```

Ur/Web is not clever enough to figure out the naive approach:

```
fun ctors [ts] fl =
  @fold [fn ts' => $(map (fn t => t -> variant ts) ts')]
      (fn [nm ::_] [v ::_] [r ::_] [[nm] ~ r] n => n ++ {nm = make [nm]})
      {} fl

```

because it has no idea that `nm` is a member of the type-level record `ts` (Ur/Web doesn’t directly have an encoding of field inclusion.)

The way to fix this problem is to use a trick that shows up again and again in variant metaprograms: make the accumulator polymorphic in the fields that are already processed. It is the same trick that is employed in value level programs, when you reverse a list by `foldr` by observing that you want to make the accumulator a function:

```
con accum r = s :: {Type} -> [r ~ s] => $(map (fn t => t -> variant (r ++ s)) r)
fun ctors [ts] fl =
  @fold [accum]
        (fn [nm::_] [v::_] [r::_] [[nm] ~ r]
            (k : accum r)
            [s::_] [[nm = v] ++ r ~ s] => k [[nm = v] ++ s] ++ {nm = make [nm]})
        (fn [s::_] [[] ~ s] => {}) fl [[]] !

```

`accum` is the type of the accumulator, and we can see it has a new type argument `s :: {Type}`. This argument is concatenated with the fields that are to be processed `r` and the current field `nm` in order to provide the full set of fields `ts`. During a fold over a record like `[A = int, B = bool, C = string]`, we see:

```
r = [],                  nm = A, s = [B = bool, C = string]
r = [A = int],           nm = B, s = [C = string]
r = [A = int, B = bool], nm = C, s = []

```

`r` builds up fields as usual in a fold, but `s` builds up its fields in reverse, because, like list reverse, `s` is not determined until we’ve folded over the entire structure, and now evaluate the pile of type functions outside-in. Thus, it’s easy to see `k [[nm = v] ++ s]` will always have the correct type.

### Conclusion

Polymorphic variants in Ur/Web are quite useful, and avoid many of the problems associated with unrestricted subtyping. Logitext wasn’t originally intending on using polymorphic variants, but we adopted them when they were found to be the most reliable method by which we could quickly implement JSON serialization via metaprogramming, and we’ve come to appreciate their metaprogrammability in a variety of other contexts too. Probably their biggest downside over traditional algebraic data types is the lack of recursion, but that too can be simulated by manually implementing the mu operator using Ur/Web’s module system. I hope this tutorial has given you enough meat to use polymorphic variants on your own, and maybe do a little metaprogramming with them too.