<!--yml
category: 未分类
date: 2024-07-01 18:16:59
-->

# Proposal: Suggest explicit type application for Foldable length and friends : ezyang’s blog

> 来源：[http://blog.ezyang.com/2017/03/proposal-suggest-explicit-type-application-for-foldable-length/](http://blog.ezyang.com/2017/03/proposal-suggest-explicit-type-application-for-foldable-length/)

**tl;dr** *If you use a Foldable function like length or null, where instance selection is solely determined by the input argument, you should make your code more robust by introducing an explicit type application specifying which instance you want. This isn't necessary for a function like fold, where the return type can cross-check if you've gotten it right or not. If you don't provide this type application, GHC should give a warning suggesting you annotate it explicitly, in much the same way it suggests adding explicit type signatures to top-level functions.*

Recently, there has been some dust kicked up about [Foldable instances causing "bad" code to compile](https://mail.haskell.org/pipermail/libraries/2017-March/027716.html). The prototypical example is this: you've written `length (f x)`, where `f` is a function that returns a list `[Int]`. At some future point in time, a colleague refactors `f` to return `(Warnings, [Int])`. After the refactoring, will `length (f x)` continue to type check? Yes: `length (f x)` will always return 1, no matter how long the inner list is, because it is using the `Foldable` instance for `(,) Warnings`.

The solution proposed in the mailing list was to remove `Foldable` for `Either`, a cure which is, quite arguably, worse than the disease. But I think there is definitely merit to the complaint that the `Foldable` instances for tuples and `Either` enable you to write code that typechecks, but is totally wrong.

[Richard Eisenberg](https://mail.haskell.org/pipermail/libraries/2017-March/027743.html) described this problem as the tension between the goals of "if it compiles, it works!" (Haskell must *exclude* programs which don't work) and general, polymorphic code, which should be applicable in as many situations as possible. I think there is some more nuance here, however. Why is it that `Functor` polymorphic code never causes problems for being "too general", but `Foldable` does? We can construct an analogous situation: I've written `fmap (+2) (f x)`, where `f` once again returns `[Int]`. When my colleague refactors `f` to return `(Warnings, [Int])`, `fmap` now makes use of the `Functor` instance `(,) Warnings`, but the code fails to compile anyway, because the type of `(+1)` doesn't line up with `[Int]`. Yes, we can still construct situations with `fmap` where code continues to work after a type change, but these cases are far more rare.

There is a clear difference between these two programs: the `fmap` program is *redundant*, in the sense that the type is constrained by both the input container, the function mapping over it, and the context which uses the result. Just as with error-correcting codes, redundancy allows us to detect when an error has occurred; when you reduce redundancy, errors become harder to detect. With `length`, the *only* constraint on the selected instance is the input argument; if you get it wrong, we have no way to tell.

Thus, the right thing to do is *reintroduce* redundancy where it is needed. Functions like `fold` and `toList` don't need extra redundancy, because they are cross-checked by the use of their return arguments. But functions like `length` and `null` (and arguably `maximum`, which only weakly constrains its argument to have an `Ord` instance) don't have any redundancy: we should introduce redundancy in these places!

Fortunately, with GHC 8.0 provides a very easy way of introducing this redundancy: an **explicit type application.** (This was also independently [suggested by Faucelme](https://www.reddit.com/r/haskell/comments/5x4yka/deprecate_foldable_for_either/def96j4/).) In this regime, rather than write `length (f x)`, you write `length @[] (f x)`, saying that you wanted length for lists. If you wanted length for maps, you write `length @(Map _) (f x)`. Now, if someone changes the type of `f`, you will get a type error since the explicit type application no longer matches.

Now, you can write this with your FTP code today. So there is just one more small change I propose we add to GHC: let users specify the type parameter of a function as "suggested to be explicit". At the call-site, if this function is used without giving a type application, GHC will emit a warning (which can be disabled with the usual mechanism) saying, "Hey, I'm using the function at this type, maybe you should add a type application." If you really want to suppress the warning, you could just type apply a type hole, e.g., `length @_ (f x)`. As a minor refinement, you could also specify a "default" type argument, so that if we infer this argument, no warning gets emitted (this would let you use the list functions on lists without needing to explicitly specify type arguments).

That's it! No BC-breaking flag days, no poisoning functions, no getting rid of FTP, no dropping instances: just a new pragma, and an opt-in warning that will let people who want to avoid these bugs. It won't solve all `Foldable` bugs, but it should squash the most flagrant ones.

What do people think?