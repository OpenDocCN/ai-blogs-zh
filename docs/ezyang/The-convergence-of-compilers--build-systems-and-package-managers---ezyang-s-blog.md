<!--yml
category: 未分类
date: 2024-07-01 18:17:07
-->

# The convergence of compilers, build systems and package managers : ezyang’s blog

> 来源：[http://blog.ezyang.com/2015/12/the-convergence-of-compilers-build-systems-and-package-managers/](http://blog.ezyang.com/2015/12/the-convergence-of-compilers-build-systems-and-package-managers/)

Abstract. *The traditional abstraction barriers between compiler, build system and package manager are increasingly ill-suited for IDEs, parallel build systems, and modern source code organization. Recent compilers like go and rustc are equipped with a fully-fledged build systems; semantic build systems like Bazel and Gradle also expect to manage the packaging of software. Does this mean we should jettison these abstraction barriers? It seems worthwhile to look for new interfaces which can accommodate these use-cases.*

Traditionally, one can understand the tooling of a programming language in three parts:

*   The **compiler** takes a single source file and transforms it into an object file. (Examples: `ghc -c`, `go tool 6g`, `javac` and `gcc -c`.)
*   The **build system** takes a collection of source files (and metadata) and transforms them into the final build product. It does this by invoking the compiler multiple times. (Examples: `go build`, `Setup build`, `make`, `ant compile`.) Often, the build system also knows how to *install* the build product in question.
*   The **package manager** takes a package name, and retrieves and builds the package and its dependencies, and installs them into some store. It does this by invoking the build systems of each package. (Examples: `cabal install`, `cargo install`, `maven package`.)

This separation constitutes an abstraction barrier which allows these components to be separately provided. For example, a single build system can work with multiple different compilers (gcc versus clang); conversely, a compiler may be invoked from a user's custom build system. A library may be packaged for both its native language package manager as well as a Linux distribution's packaging system; conversely, a package manager may be indifferent to how a library actually gets built. In today's software ecosystem, these abstraction barriers are used heavily, with good effect!

However, there are an increasing number of use-cases which cannot be adequately handled using these abstraction barriers:

*   A build system needs to know what order to build source files in; however, the canonical source for this information is inside the import/include declarations of the source file. This information must either be duplicated inside the build system, or the build system must call the compiler in order to compute the dependency graph to be used. In any case, a compiler cannot *just* be a dumb source-to-object-file converter: it must know how to emit dependencies of files (e.g., `gcc -M`). There is no standardized format for this information, except perhaps a `Makefile` stub.
*   The dependency problem is further exacerbated when module dependencies can be cyclic. A build system must know how to resolve cycles, either by compiling strongly connected components of modules at a time, or compiling against "interface" files, which permit separate compilation. This was one of the problems which [motivated the Rust developers](https://github.com/rust-lang/rfcs/pull/1317#issuecomment-161729336) to not expose a one-source-at-a-time compiler.
*   The best parallelization can be achieved with a fine-grained dependency graph over source files. However, the most desirable place to implement parallelization is the package manager, as an invocation of the package manager will cause the most code to be compiled. Thus, a system like Bazel unifies both the build system and the package manager, so that parallelism can be achieved over the entire build. (Another example is GHC's build system, which parallelizes compilation of all wired-in packages on a per-module basis.)
*   IDEs want in-depth information from the compiler beyond a `-c` style interface. But they cannot invoke the compiler directly, because the only way to invoke the compiler with the right flags and the right environment is via the build system / the package manager. Go's built in build-system means that it can more easily provide a tool like `go oracle`; otherwise, `go oracle` would need to be able to accommodate external build systems.
*   Certain language features are actively hostile to build systems; only the compiler has enough smarts to figure out how to manage the build. Good examples include macros (especially macros that can access the filesystem), other forms of compile-time metaprogramming, and compiler plugins.

Thus, the temptation is to roll up these components into a single monolithic tool that does everything. There are many benefits: a single tool is easier to develop, gives a more uniform user experience, and doesn't require the developers to specify a well-defined API between the different components. The downside? You can't swap out pieces of a monolithic system.

I think it is well worth considering how we can preserve this separation of concerns, even in the face of these features. Unfortunately, I don't know what the correct API is, but here is a strawman proposal: every compiler and build system writer should have an alternative mode which lets a user ask the query, "How do I make `$output` file?" This mode returns (1) the dependencies of that file, and (2) a recipe for how to make it. The idea is to place the dependency-finding logic in the compiler (the canonical place to put it), while letting an external tool actually handle building the dependencies. But there are a lot of details this proposal doesn't cover.

What do you think about the convergence of compiler, build system and package manager? Do you think they *should* be monolithic? If not, what do you think the right API to support these new use cases should be? I'd love to know what you think.