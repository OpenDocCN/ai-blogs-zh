<!--yml
category: 未分类
date: 2024-07-01 18:17:03
-->

# A tale of backwards compatibility in ASTs : ezyang’s blog

> 来源：[http://blog.ezyang.com/2016/12/a-tale-of-backwards-compatibility-in-asts/](http://blog.ezyang.com/2016/12/a-tale-of-backwards-compatibility-in-asts/)

Those that espouse the value of backwards compatibility often claim that backwards compatibility is simply a matter of never *removing* things. But anyone who has published APIs that involve data structures know that the story is not so simple. I'd like to describe my thought process on a recent BC problem I'm grappling with on the Cabal file format. As usual, I'm always interested in any insights and comments you might have.

**The status quo.** The `build-depends` field in a Cabal file is used to declare dependencies on other packages. The format is a comma-separated list of package name and version constraints, e.g., `base >= 4.2 && < 4.3`. Abstractly, we represent this as a list of `Dependency`:

```
data Dependency = Dependency PackageName VersionRange

```

The effect of an entry in `build-depends` is twofold: first, it specifies a version constraint which a dependency solver takes into account when picking a version of the package; second, it brings the modules of that package into scope, so that they can be used.

**The extension.** We added support for "internal libraries" in Cabal, which allow you to specify multiple libraries in a single package. For example, suppose you're writing a library, but there are some internal functions that you want to expose to your test suite but not the general public. You can place these functions in an internal library, which is depended upon by both the public library and the test suite, but not available to external packages.

For more motivation, see the original [feature request](https://github.com/haskell/cabal/issues/269), but for the purpose of this blog post, we're interested in the question of how to specify a dependency on one of these internal libraries.

**Attempt #1: Keep the old syntax.** My first idea for a new syntax for internal libraries was to keep the syntax of `build-depends` *unchanged*. To refer to an internal library named `foo`, you simply write `build-depends: foo`; an internal library shadows any external package with the same name.

Backwards compatible? Absolutely not. Remember that the original interpretation of entries in `build-depends` is of *package* names and version ranges. So if you had code that assumed that there actually is an external package for each entry in `build-depends` would choke in an unexpected way when a dependency on an internal library was specified. This is exactly what happened with cabal-install's dependency solver, which needed to be updated to filter out dependencies that corresponded to internal libraries.

One might argue that it is acceptable for old code to break if the new feature is used. But there is a larger, philosophical objection to overloading package names in this way: don't call something a package name if it... isn't actually a package name!

**Attempt #2: A new syntax.** Motivated by this philosophical concern, as well as the problem that you couldn't simultaneously refer to an internal library named `foo` and an external package named `foo`, we introduce a new syntactic form: to refer to the internal library `foo` in the package `pkg`, we write `build-depends: pkg:foo`.

Since there's a new syntactic form, our internal AST also has to change to handle this new form. The obvious thing to do is introduce a new type of dependency:

```
data BuildDependency =
  BuildDependency PackageName
                  (Maybe UnqualComponentName)
                  VersionRange

```

and say that the contents of `build-depends` is a list of `BuildDependency`.

When it comes to changes to data representation, this is a "best-case scenario", because we can easily write a function `BuildDependency -> Dependency`. So supposing our data structure for describing library build information looked something like this:

```
data BuildInfo = BuildInfo {
    targetBuildDepends :: [Dependency],
    -- other fields
  }

```

We can preserve backwards compatibility by turning `targetBuildDepends` into a function that reads out the new, extend field, and converts it to the old form:

```
data BuildInfo = BuildInfo {
    targetBuildDepends2 :: [BuildDependency],
    -- other fields
  }

targetBuildDepends :: BuildInfo -> [Dependency]
targetBuildDepends = map buildDependencyToDependency
                   . targetBuildDepends2

```

Critically, this takes advantage of the fact that record selectors in Haskell look like functions, so we can replace a selector with a function without affecting downstream code.

Unfortunately, this is not actually true. Haskell also supports *record update*, which lets a user overwrite a field as follows: `bi { targetBuildDepends = new_deps }`. If we look at Hackage, there are actually a dozen or so uses of `targetBuildDepends` in this way. So, if we want to uphold backwards-compatibility, we can't delete this field. And unfortunately, Haskell doesn't support overloading the meaning of record update (perhaps the lesson to be learned here is that you should never export record selectors: export some lenses instead).

It is possible that, in balance, breaking a dozen packages is a fair price to pay for a change like this. But let's suppose that we are dead-set on maintaining BC.

**Attempt #3: Keep both fields.** One simple way to keep the old code working is to just keep both fields:

```
data BuildInfo = BuildInfo {
    targetBuildDepends  :: [Dependency],
    targetBuildDepends2 :: [BuildDependency],
    -- other fields
  }

```

We introduce a new invariant, which is that `targetBuildDepends bi == map buildDependencyToDependency (targetBuildDepends2 bi)`. See the problem? Any legacy code which updates `targetBuildDepends` probably won't know to update `targetBuildDepends2`, breaking the invariant and probably resulting in some very confusing bugs. Ugh.

**Attempt #4: Do some math.** The problem with the representation above is that it is redundant, which meant that we had to add invariants to "reduce" the space of acceptable values under the type. Generally, we like types which are "tight", so that, as Yaron Minsky puts it, we "make illegal states unrepresentable."

To think a little more carefully about the problem, let's cast it into a mathematical form. We have an `Old` type (isomorphic to `[(PN, VR)]`) and a `New` type (isomorphic to `[(PN, Maybe CN, VR)]`). `Old` is a subspace of `New`, so we have a well-known injection `inj :: Old -> New`.

When a user updates `targetBuildDepends`, they apply a function `f :: Old -> Old`. In making our systems backwards compatible, we implicitly define a new function `g :: New -> New`, which is an extension of `f` (i.e., `inj . f == g . inj`): this function tells us what the *semantics* of a legacy update in the new system is. Once we have this function, we then seek a decomposition of `New` into `(Old, T)`, such that applying `f` to the first component of `(Old, T)` gives you a new value which is equivalent to the result of having applied `g` to `New`.

Because in Haskell, `f` is an opaque function, we can't actually implement many "common-sense" extensions. For example, we might want it to be the case that if `f` updates all occurrences of `parsec` with `parsec-new`, the corresponding `g` does the same update. But there is no way to distinguish between an `f` that updates, and an `f` that deletes the dependency on `parsec`, and then adds a new dependency on `parsec-new`. (In the bidirectional programming world, this is the distinction between [state-based and operation-based approaches](https://www.cis.upenn.edu/~bcpierce/papers/lenses-etapsslides.pdf).)

We really only can do something reasonable if `f` only ever adds dependencies; in this case, we might write something like this:

```
data BuildInfo = BuildInfo {
    targetBuildDepends :: [Dependency],
    targetSubLibDepends :: [(PackageName, UnqualComponentName)],
    targetExcludeLibDepends :: [PackageName],
    -- other fields
  }

```

The conversion from this to `BuildDependency` goes something like:

1.  For each `Dependency pn vr` in `targetBuildDepends`, if the package name is not mentioned in `targetExcludeLibDepends`, we have `BuildDependency pn Nothing vr`.
2.  For each `(pn, cn)` in `targetSubLibDepends` where there is a `Dependency pn vr` (the package names are matching), we have `BuildDependency pn (Just cn) vr`.

Stepping back for a moment, *is this really the code we want to write*? If the modification is not monotonic, we'll get into trouble; if someone reads out `targetBuildDepends` and then writes it into a fresh `BuildInfo`, we'll get into trouble. Is it really reasonable to go to these lengths to achieve such a small, error-prone slice of backwards compatibility?

**Conclusions.** I'm still not exactly sure what approach I'm going to take to handle this particular extension, but there seem to be a few lessons:

1.  Records are bad for backwards compatibility, because there is no way to overload a record update with a custom new update. Lenses for updates would be better.
2.  Record update is bad for backwards compatibility, because it puts us into the realm of *bidirectional programming*, requiring us to reflect updates from the old world into the new world. If our records are read-only, life is much easier. On the other hand, if someone ever designs a programming language that is explicitly thinking about backwards compatibility, bidirectional programming better be in your toolbox.
3.  Backwards compatibility may be worse in the cure. Would you rather your software break at compile time because, yes, you really do have to think about this new case, or would you rather everything keep compiling, but break in subtle ways if the new functionality is ever used?

What's your take? I won't claim to be a expert on questions of backwards compatibility, and would love to see you weigh in, whether it is about which approach I should take, or general thoughts about the interaction of programming languages with backwards compatibility.