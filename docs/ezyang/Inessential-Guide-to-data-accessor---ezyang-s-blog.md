<!--yml
category: 未分类
date: 2024-07-01 18:18:22
-->

# Inessential Guide to data-accessor : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/04/inessential-guide-to-data-accessor/](http://blog.ezyang.com/2010/04/inessential-guide-to-data-accessor/)

[data-accessor](http://hackage.haskell.org/package/data-accessor-0.2.1.2) is a package that makes records *not suck.* Instead of this code:

```
newRecord = record {field = newVal}

```

You can write this:

```
newRecord = field ^= newVal
          $ record

```

In particular, `(field ^= newVal)` is now a value, *not* a bit of extra syntax, that you can treat as a first-class citizen.

I came across this module while attempting to use [Chart](http://hackage.haskell.org/package/Chart) (of criterion fame) to graph some data. I didn't recognize it at first, though; it was only after playing around with code samples did I realize that `^=` was not a combinator that Chart had invented for its own use (as opposed to the potpourri of `-->`, `<+>`, `|||` and friends you might see in an *xmonad.hs*). When utilized with Template Haskell, Data.Accessor represents something of a *replacement* for the normal record system, and so it's useful to know when a module speaks this other language. Signs that you're in a module using Data.Accessor:

*   Use of the `^=` operator in code samples
*   All of the records have underscores suffixed, such as `plot_lines_title_`
*   Template Haskell gobbledygook (including type variables that look like `x[acGI]`, especially in the "real" accessors that Template Haskell generated).
*   Unqualified `T` data types floating around. (As Brent Yorgey tells me, this is a Henning-ism in which he will define a type T or typeclass C intended to be used only with a qualified import, but Haddock throws away this information. You can use `:t` in GHC to get back this information if you're not sure.)

Once you've identified that a module is indeed using Data.Accessor, you've won most of the battle. Here is a whirlwind tutorial on how to use records that use data-accessor.

*Interpreting types.* An *accessor* (represented by the type `Data.Accessor.T r a`) is defined to be a getter (`r -> a`) and setter (`a -> r -> r`). `r` is the type of the record, and `a` is the type of the value that can be retrieved or set. If Template Haskell was used to generate the definitions, polymorphic types inside of `a` and `r` will frequently be universally quantified with type variables that `x[acGI]`, don't worry too much about them; you can pretend they're normal type variables. For the curious, these are generated by the quotation monad in Template Haskell).

*Accessing record fields.* The old way:

```
fieldValue = fieldName record

```

You can do things several ways with Data.Accessor:

```
fieldValue = getVal fieldname record
fieldValue = record ^. fieldname

```

*Setting record fields.* The old way:

```
newRecord = record {fieldName = newValue}

```

The new ways:

```
newRecord = setVal fieldName newValue record
newRecord = fieldName ^= newValue $ record

```

*Accessing and setting sub-record fields.* The old ways:

```
innerValue = innerField (outerField record)
newRecord = record {
  outerField = (outerField record) {
    innerField = newValue
  }
}

```

The new ways (this is bit reminiscent of [semantic editor combinators](http://conal.net/blog/posts/semantic-editor-combinators/)):

```
innerValue = getVal (outerField .> innerField) record
newRecord = setVal (outerField .> innerField) newValue record

```

There are also functions for modifying records inside the state monad, but I'll leave those explanations for [the Haddock documentation](http://hackage.haskell.org/packages/archive/data-accessor/0.2.1.2/doc/html/Data-Accessor.html). Now go forth and, erm, access your data *in style*!