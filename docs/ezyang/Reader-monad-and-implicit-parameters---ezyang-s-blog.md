<!--yml
category: 未分类
date: 2024-07-01 18:18:13
-->

# Reader monad and implicit parameters : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/07/implicit-parameters-in-haskell/](http://blog.ezyang.com/2010/07/implicit-parameters-in-haskell/)

*For when the Reader monad seems hopelessly clunky*

The Reader monad (also known as the Environment monad) and implicit parameters are remarkably similar even though the former is the standard repertoire of a working Haskell programmer while the latter is a GHC language extension used sparingly by those who know about it. Both allow the programmer to code as if they had access to a global environment that can still change at runtime. However, implicit parameters are remarkably well suited for cases when you would have used a stack of reader transformers. Unfortunately, unlike many type system extensions, GHC cannot suggest that you enable `ImplicitParams` because the code you innocently wrote is not valid Haskell98 but would be valid if you enabled this extension. This post intends to demonstrate one way to discover implicit parameters, with a little nudging.

*Reader monad in practice.* The Reader monad is really quite simple: after all, it is isomorphic to `(->) r`, the only real difference being a newtype. Because of this, in engineering contexts, it is rarely used as-is; in particular:

*   It is used as a transformer, endowing an “environment” to whatever application-specific monad you are building, and
*   It is used with a record type, because an environment of only one primitive value is usually not very interesting.

These choices impose some constraints on how code written for a Reader monad can be used. In particular, baking in the environment type `r` of `ReaderT r` means that your monadic code will not play nicely with some other monadic code `ReaderT r2` without some coaxing; additionally, I can’t build up a complicated record type `Record { field1 :: Int; field2 :: String; field3 :: Bool}` incrementally as I find out values of the environment. I could have my record type be a map of some sort, in which case I could place arbitrarily values in it, but in this case I have no static assurances of what values will or will not be in the map at a given point in time.

*Stacked Reader transformers.* To allow ourselves to incrementally build up our environment, one might consider stacking the Reader monad transformers. Consider the type `ReaderT a (ReaderT b (ReaderT c IO)) ()`. If we desugar this into function application, we find `a -> (b -> (c -> IO ()))`, which can be further simplified to `a -> b -> c -> IO ()`. If `a`, `b` and `c` happen to be the same type, we don’t have any way of distinguishing the different values, except for the location in the list of arguments. However, instead of writing out the parameters explicitly in our function signature (which, indeed, we are trying to avoid with the reader monad), we find ourselves having to lift `ask` repeatedly (zero times for `a`, once for `b` and twice for `c`). Unlike the record with three fields, there is no name for each environment variable: we have to refer to them by using some number of lifts.

> *Aside*. In fact, this is a [De Bruijn index](http://en.wikipedia.org/wiki/De_Bruijn_index), which [Oleg](http://okmij.org/ftp/) helpfully pointed in out in an email conversation we had after my post about [nested loops and continuations](http://blog.ezyang.com/2010/02/nested-loops-and-continuation/). The number of lifts is the index (well, the Wikipedia article is 1-indexed, in which case add one) which tells us how many reader binding scopes we need to pop out of. So if I have:
> 
> ```
> runReaderT (runReaderT (runReaderT (lift ask) c) b) a
> \------- outermost/furthest context (3) ------------/
>            \--- referenced context (2; one lift) -/
>                        \--- inner context (1) -/
> 
> ```
> 
> I get the value of `b`. This turns out to be wonderful for the lambda-calculus theoreticians (who are cackling gleefully at trouble-free α-conversion), but not so wonderful for software engineers, for whom De Bruijn indexes are equivalent to the famous antipattern, the magic number.

With typeclass tricks, we can get back names to some extent: for example, Dan Piponi [renames the transformers with singleton data types or “tags”](http://blog.sigfpe.com/2010/02/tagging-monad-transformer-layers.html), bringing in the heavy guns of `OverlappingInstances` in the process. Oleg [uses lexical variables that are typed to the layer they belong to](http://okmij.org/ftp/Haskell/regions.html#light-weight) to identify different layers, although such an approach is not really useful for a Reader monad stack, since the point of the Reader monad is not to have to pass any lexical variables around, whether or not they are the actual variables or specially typed variables.

*Implicit parameters.* In many ways, implicit parameters are a cheat: while Dan and Oleg’s approaches leverage existing type-level programming facilities, implicit parameters define a “global” namespace (well known to Lispers as the dynamic scope) that we can stick variables in, and furthermore it extends the type system so we can express what variables in this namespace any given function call expects to exist (without needing to use monads, the moxy!)

Instead of an anonymous environment, we assign the variable a name:

```
f :: ReaderT r IO a
f' :: (?implicit_r :: r) => IO a

```

`f'` is still monadic, but the monad doesn’t express what is in the environment anymore: it’s entirely upon the type signature to determine if an implicit variable is passed along:

```
f  = print "foobar" >> g 42 -- Environment always passed on
f' = print "foobar" >> g 42 -- Not so clear!

```

Indeed, `g` could have just as well been a pure computation:

```
f' = print (g 42)

```

However, if the type of is:

```
g :: IO a

```

the implicit variable is lost, while if it is:

```
g :: (?implicit_r :: r) => IO a

```

the variable is available.

While `runReader(T)` was our method for specifying the environment, we now have a custom let syntax:

```
runReaderT f value_of_r
let ?implicit_r = value_of_r in f

```

Besides having ditched our monadic restraints, we can now easily express our incremental environment:

```
run = let ?implicit_a = a
          ?implicit_b = b
          ?implicit_c = c
      in h

h :: (?implicit_a :: a, ?implicit_b :: b, ?implicit_c :: c) => b
h = ?implicit_b

```

You can also use `where`. Note that, while this looks deceptively like a normal `let` binding, it is quite different: you can’t mix implicit and normal variable bindings, and if you have similarly named implicit bindings on the right-hand side, they refer to their values *outside* of the `let`. No recursion for you! (Recall `runReaderT`: the values that we supply in the second argument are pure variables and not values in the Reader monad, though with `>>=` you could instrument things that way.)

*Good practices.* With monadic structure gone, there are fewer source-level hints on how the monomorphism restriction and polymorphic recursion apply. Non-polymorphic recursion *will* compile, and cause unexpected results, such as your implicit parameter not changing when you expect it to. You can play things relatively safely by making sure you always supply type signatures with all the implicit parameters you are expecting. I hope to do a follow-up post explaining more carefully what these semantics are, based off of formal description of types in the [relevant paper](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.46.9849).