<!--yml
category: 未分类
date: 2024-07-01 18:17:14
-->

# What’s a module system good for anyway? : ezyang’s blog

> 来源：[http://blog.ezyang.com/2014/08/whats-a-module-system-good-for-anyway/](http://blog.ezyang.com/2014/08/whats-a-module-system-good-for-anyway/)

This summer, I've been working at Microsoft Research implementing [Backpack](http://plv.mpi-sws.org/backpack/), a module system for Haskell. Interestingly, Backpack is not really a single monolothic feature, but, rather, an agglomeration of small, infrastructural changes which combine together in an interesting way. In this series of blog posts, I want to talk about what these individual features are, as well as how the whole is greater than the sum of the parts.

But first, there's an important question that I need to answer: **What's a module system good for anyway?** Why should you, an average Haskell programmer, care about such nebulous things as *module systems* and *modularity*. At the end of the day, you want your tools to solve specific problems you have, and it is sometimes difficult to understand what problem a module system like Backpack solves. As [tomejaguar puts it](http://www.reddit.com/r/haskell/comments/28v6c9/backpack_an_mllike_module_system_for_haskell/cierxc1): "Can someone explain clearly the precise problem that Backpack addresses? I've read the paper and I know the problem is 'modularity' but I fear I am lacking the imagination to really grasp what the issue is."

Look no further. In this blog post, I want to talk concretely about problems Haskellers have today, explain what the underlying causes of these problems are, and say why a module system could help you out.

### The String, Text, ByteString problem

As experienced Haskellers [are](http://blog.ezyang.com/2010/08/strings-in-haskell/) [well](http://stackoverflow.com/questions/19608745/data-text-vs-string) [aware](http://a-dimit.blogspot.com/2012/04/strings-in-haskell.html), there are multitude of string types in Haskell: String, ByteString (both lazy and strict), Text (also both lazy and strict). To make matters worse, there is no one "correct" choice of a string type: different types are appropriate in different cases. String is convenient and native to Haskell'98, but very slow; ByteString is fast but are simply arrays of bytes; Text is slower but Unicode aware.

In an ideal world, a programmer might choose the string representation most appropriate for their application, and write all their code accordingly. However, this is little solace for library writers, who don't know what string type their users are using! What's a library writer to do? There are only a few choices:

1.  They "commit" to one particular string representation, leaving users to manually convert from one representation to another when there is a mismatch. Or, more likely, the library writer used the default because it was easy. Examples: [base](https://hackage.haskell.org/package/base) (uses Strings because it completely predates the other representations), [diagrams](https://hackage.haskell.org/package/diagrams) (uses Strings because it doesn't really do heavy string manipulation).
2.  They can provide separate functions for each variant, perhaps identically named but placed in separate modules. This pattern is frequently employed to support both strict/lazy variants Text and ByteStringExamples: [aeson](http://hackage.haskell.org/package/aeson) (providing decode/decodeStrict for lazy/strict ByteString), [attoparsec](https://hackage.haskell.org/package/attoparsec) (providing Data.Attoparsec.ByteString/Data.Attoparsec.ByteString.Lazy), [lens](http://hackage.haskell.org/package/lens) (providing Data.ByteString.Lazy.Lens/Data.ByteString.Strict.Lens).
3.  They can use type-classes to overload functions to work with multiple representations. The particular type class used hugely varies: there is [ListLike](https://hackage.haskell.org/package/ListLike), which is used by a handful of packages, but a large portion of packages simply roll their own. Examples: SqlValue in [HDBC](http://hackage.haskell.org/package/HDBC), an internal StringLike in [tagsoup](https://hackage.haskell.org/package/tagsoup), and yet another internal StringLike in [web-encodings](http://hackage.haskell.org/package/web-encodings).

The last two methods have different trade offs. Defining separate functions as in (2) is a straightforward and easy to understand approach, but you are still saying *no* to modularity: the ability to support multiple string representations. Despite providing implementations for each representation, the user still has to commit to particular representation when they do an import. If they want to change their string representation, they have to go through all of their modules and rename their imports; and if they want to support multiple representations, they'll still have to write separate modules for each of them.

Using type classes (3) to regain modularity may seem like an attractive approach. But this approach has both practical and theoretical problems. First and foremost, how do you choose which methods go into the type class? Ideally, you'd pick a minimal set, from which all other operations could be derived. However, many operations are most efficient when directly implemented, which leads to a bloated type class, and a rough time for other people who have their own string types and need to write their own instances. Second, type classes make your type signatures more ugly `String -> String` to `StringLike s => s -> s` and can make type inference more difficult (for example, by introducing ambiguity). Finally, the type class `StringLike` has a very different character from the type class `Monad`, which has a minimal set of operations and laws governing their operation. It is difficult (or impossible) to characterize what the laws of an interface like this should be. All-in-all, it's much less pleasant to program against type classes than concrete implementations.

Wouldn't it be nice if I could `import String`, giving me the type `String` and operations on it, but then later decide which concrete implementation I want to instantiate it with? This is something a module system can do for you! This [Reddit thread](http://www.reddit.com/r/haskell/comments/28v6c9/backpack_an_mllike_module_system_for_haskell/cierxc1) describes a number of other situations where an ML-style module would come in handy.

(PS: Why can't you just write a pile of preprocessor macros to swap in the implementation you want? The answer is, "Yes, you can; but how are you going to type check the thing, without trying it against every single implementation?")

### Destructive package reinstalls

Have you ever gotten this error message when attempting to install a new package?

```
$ cabal install hakyll
cabal: The following packages are likely to be broken by the reinstalls:
pandoc-1.9.4.5
Graphalyze-0.14.0.0
Use --force-reinstalls if you want to install anyway.

```

Somehow, Cabal has concluded that the only way to install hakyll is to reinstall some dependency. Here's one situation where a situation like this could come about:

1.  pandoc and Graphalyze are compiled against the latest unordered-containers-0.2.5.0, which itself was compiled against the latest hashable-1.2.2.0.
2.  hakyll also has a dependency on unordered-containers and hashable, but it has an upper bound restriction on hashable which excludes the latest hashable version. Cabal decides we need to install an old version of hashable, say hashable-0.1.4.5.
3.  If hashable-0.1.4.5 is installed, we also need to build unordered-containers against this older version for Hakyll to see consistent types. However, the resulting version is the same as the preexisting version: thus, reinstall!

The root cause of this error an invariant Cabal currently enforces on a package database: there can only be *one* instance of a package for any given package name and version. In particular, this means that it is not possible to install a package multiple times, compiled against different dependencies. This is a bit troublesome, because sometimes you really do want the same package installed multiple times with different dependencies: as seen above, it may be the only way to fulfill the version bounds of all packages involved. Currently, the only way to work around this problem is to use a Cabal sandbox (or blow away your package database and reinstall everything, which is basically the same thing).

You might be wondering, however, how could a module system possibly help with this? It doesn't... at least, not directly. Rather, nondestructive reinstalls of a package are a critical feature for implementing a module system like Backpack (a package may be installed multiple times with different concrete implementations of modules). Implementing Backpack necessitates fixing this problem, moving Haskell's package management a lot closer to that of Nix's or NPM.

### Version bounds and the neglected PVP

While we're on the subject of cabal-install giving errors, have you ever gotten this error attempting to install a new package?

```
$ cabal install hledger-0.18
Resolving dependencies...
cabal: Could not resolve dependencies:
# pile of output

```

There are a number of possible reasons why this could occur, but usually it's because some of the packages involved have over-constrained version bounds (especially upper bounds), resulting in an unsatisfiable set of constraints. To add insult to injury, often these bounds have no grounding in reality (the package author simply guessed the range) and removing it would result in a working compilation. This situation is so common that Cabal has a flag `--allow-newer` which lets you override the upper bounds of packages. The annoyance of managing bounds has lead to the development of tools like [cabal-bounds](https://github.com/dan-t/cabal-bounds), which try to make it less tedious to keep upper bounds up-to-date.

But as much as we like to rag on them, version bounds have a very important function: they prevent you from attempting to compile packages against dependencies which don't work at all! An under-constrained set of version bounds can easily have compiling against a version of the dependency which doesn't type check.

How can a module system help? At the end of the day, version numbers are trying to capture something about the API exported by a package, described by the [package versioning policy](http://www.haskell.org/haskellwiki/Package_versioning_policy). But the current state-of-the-art requires a user to manually translate changes to the API into version numbers: an error prone process, even when assisted [by](http://code.haskell.org/gtk2hs/tools/apidiff/) [various](http://hackage.haskell.org/package/precis) [tools](http://hackage.haskell.org/package/check-pvp). A module system, on the other hand, turns the API into a first-class entity understood by the compiler itself: a *module signature.* Wouldn't it be great if packages depended upon signatures rather than versions: then you would never have to worry about version numbers being inaccurate with respect to type checking. (Of course, versions would still be useful for recording changes to semantics not seen in the types, but their role here would be secondary in importance.) Some full disclosure is warranted here: I am not going to have this implemented by the end of my internship, but I'm hoping to make some good infrastructural contributions toward it.

### Conclusion

If you skimmed the introduction to the Backpack paper, you might have come away with the impression that Backpack is something about random number generators, recursive linking and applicative semantics. While these are all true "facts" about Backpack, they understate the impact a good module system can have on the day-to-day problems of a working programmer. In this post, I hope I've elucidated some of these problems, even if I haven't convinced you that a module system like Backpack actually goes about solving these problems: that's for the next series of posts. Stay tuned!