<!--yml
category: 未分类
date: 2024-07-01 18:17:06
-->

# Announcing cabal new-build: Nix-style local builds : ezyang’s blog

> 来源：[http://blog.ezyang.com/2016/05/announcing-cabal-new-build-nix-style-local-builds/](http://blog.ezyang.com/2016/05/announcing-cabal-new-build-nix-style-local-builds/)

`cabal new-build`, also known as “Nix-style local builds”, is a new command inspired by Nix that comes with cabal-install 1.24\. Nix-style local builds combine the best of non-sandboxed and sandboxed Cabal:

1.  Like sandboxed Cabal today, we build sets of independent local packages deterministically and independent of any global state. new-build will never tell you that it can't build your package because it would result in a “dangerous reinstall.” Given a particular state of the Hackage index, your build is completely reproducible. For example, you no longer need to compile packages with profiling ahead of time; just request profiling and new-build will rebuild all its dependencies with profiling automatically.
2.  Like non-sandboxed Cabal today, builds of external packages are cached globally, so that a package can be built once, and then reused anywhere else it is also used. No need to continually rebuild dependencies whenever you make a new sandbox: dependencies which can be shared, are shared.

Nix-style local builds work with all versions of GHC supported by cabal-install 1.24, which currently is GHC 7.0 and later. Additionally, cabal-install is on a different release cycle than GHC, so we plan to be pushing bugfixes and updates on a faster basis than GHC's yearly release cycle.

Although this feature is in only beta (there are bugs, see “Known Issues”, and the documentation is a bit sparse), I’ve been successfully using Nix-style local builds exclusively to do my Haskell development. It's hard to overstate my enthusiasm for this new feature: it “just works”, and you don't need to assume that there is a distribution of blessed, version-pegged packages to build against (e.g., Stackage). Eventually, `new-build` will simply replace the existing `build` command.

### Quick start

Nix-style local builds “just work”: there is very little configuration that needs to be done to start working with it.

1.  Download and install cabal-install 1.24:

    ```
    cabal update
    cabal install cabal-install

    ```

    Make sure the newly installed `cabal` is in your path.

2.  To build a single Cabal package, instead of running `cabal configure; cabal build`, you can use Nix-style builds by prefixing these commands with `new-`; e.g., `cabal new-configure; cabal new-build`. `cabal new-repl` is also supported. (Unfortunately, other commands are not yet supported, e.g. `new-clean` ([#2957](https://github.com/haskell/cabal/issues/2957)) or `new-freeze` ([#2996](https://github.com/haskell/cabal/issues/2996)).)

3.  To build multiple Cabal packages, you need to first create `cabal.project` file in some root directory. For example, in the Cabal repository, there is a root directory with a folder per package, e.g., the folders `Cabal` and `cabal-install`. Then in `cabal.project`, specify each folder:

    ```
    packages: Cabal/
              cabal-install/

    ```

    Then, in the directory for a package, you can say `cabal new-build` to build all of the components in that package; alternately, you can specify a list of targets to build, e.g., `package-tests cabal` asks to build the `package-tests` test suite and the `cabal` executable. A component can be built from any directory; you don't have to be cd'ed into the directory containing the package you want to build. Additionally, you can qualify targets by the package they came from, e.g., `Cabal:package-tests` asks specifically for the `package-tests` component from Cabal. There is no need to manually configure a sandbox: add a `cabal.project` file, and it just works!

Unlike sandboxes, there is no need to `add-source`; just add the package directories to your `cabal.project`. And unlike traditional `cabal install`, there is no need to explicitly ask for packages to be installed; `new-build` will automatically fetch and build dependencies.

There is also a convenient [script](https://github.com/hvr/multi-ghc-travis/blob/master/make_travis_yml_2.hs) you can use for hooking up `new-build` to your [Travis builds](https://github.com/hvr/multi-ghc-travis).

### How it works

Nix-style local builds are implemented with these two big ideas:

1.  For external packages (from Hackage), prior to compilation, we take all of the inputs which would influence the compilation of a package (flags, dependency selection, etc.) and hash it into an identifier. Just as in Nix, these hashes uniquely identify the result of a build; if we compute this identifier and we find that we already have this ID built, we can just use the already built version. These packages are stored globally in `~/.cabal/store`; you can list all of the Nix packages that are globally available using `ghc-pkg list --package-db=$HOME/.cabal/store/ghc-VERSION/package.db`.
2.  For local packages, we instead assign an `inplace` identifier, e.g., `foo-0.1-inplace`, which is local to a given `cabal.project`. These packages are stored locally in `dist-newstyle/build`; you can list all of the per-project packages using `ghc-pkg list --package-db=dist-newstyle/packagedb`. This treatment applies to any remote packages which depend on local packages (e.g., if you vendored some dependency which your other dependencies depend on.)

Furthermore, Nix local builds use a deterministic dependency solving strategy, by doing dependency solving independently of the locally installed packages. Once we've solved for the versions we want to use and have determined all of the flags that will be used during compilation, we generate identifiers and then check if we can improve packages we would have needed to build into ones that are already in the database.

### Commands

#### `new-configure FLAGS`

Overwrites `cabal.project.local` based on FLAGS.

#### `new-build [FLAGS] [COMPONENTS]`

Builds one or more components, automatically building any local and non-local dependencies (where a local dependency is one where we have an inplace source code directory that we may modify during development). Non-local dependencies which do not have a transitive dependency on a local package are installed to `~/.cabal/store`, while all other dependencies are installed to `dist-newstyle`.

The set of local packages is read from `cabal.project`; if none is present, it assumes a default project consisting of all the Cabal files in the local directory (i.e., `packages: *.cabal`), and optional packages in every subdirectory (i.e., `optional-packages: */*.cabal`).

The configuration of the build of *local* packages is computed by reading flags from the following sources (with later sources taking priority):

1.  `~/.cabal/config`
2.  `cabal.project`
3.  `cabal.project.local` (usually generated by `new-configure`)
4.  `FLAGS` from the command line

The configuration of non-local packages is only affect by package-specific flags in these sources; global options are not applied to the build. (For example, if you `--disable-optimization`, this will only apply to your local inplace packages, and not their remote dependencies.)

`new-build` does not read configuration from `cabal.config`.

#### Phrasebook

Here is a handy phrasebook for how to do existing Cabal commands using Nix local build:

| old-style | new-style |
| --- | --- |
| `cabal configure` | `cabal new-configure` |
| `cabal build` | `cabal new-build` |
| `cabal clean` | `rm -rf dist-newstyle cabal.project.local` |
| `cabal run EXECUTABLE` | `cabal new-build; ./dist-newstyle/build/PACKAGE-VERSION/build/EXECUTABLE/EXECUTABLE` |
| `cabal repl` | `cabal new-repl` |
| `cabal test TEST` | `cabal new-build; ./dist-newstyle/build/PACKAGE-VERSION/build/TEST/TEST` |
| `cabal benchmark BENCH` | `cabal new-build; ./dist-newstyle/build/PACKAGE-VERSION/build/BENCH/BENCH` |
| `cabal haddock` | does not exist yet |
| `cabal freeze` | does not exist yet |
| `cabal install --only-dependencies` | unnecessary (handled by `new-build`) |
| `cabal install` | does not exist yet (for libraries `new-build` should be sufficient; for executables, they can be found in `~/.cabal/store/ghc-GHCVER/PACKAGE-VERSION-HASH/bin`) |

### cabal.project files

`cabal.project` files actually support a variety of options beyond `packages` for configuring the details of your build. Here is a simple example file which displays some of the possibilities:

```
-- For every subdirectory, build all Cabal files
-- (project files support multiple Cabal files in a directory)
packages: */*.cabal
-- Use this compiler
with-compiler: /opt/ghc/8.0.1/bin/ghc
-- Constrain versions of dependencies in the following way
constraints: cryptohash < 0.11.8
-- Do not build benchmarks for any local packages
benchmarks: False
-- Build with profiling
profiling: true
-- Suppose that you are developing Cabal and cabal-install,
-- and your local copy of Cabal is newer than the
-- distributed hackage-security allows in its bounds: you
-- can selective relax hackage-security's version bound.
allow-newer: hackage-security:Cabal

-- Settings can be applied per-package
package cryptohash
  -- For the build of cryptohash, instrument all functions
  -- with a cost center (normally, you want this to be
  -- applied on a per-package basis, as otherwise you would
  -- get too much information.)
  profiling-detail: all-functions
  -- Disable optimization for this package
  optimization: False
  -- Pass these flags to GHC when building
  ghc-options: -fno-state-hack

package bytestring
  -- And bytestring will be built with the integer-simple
  -- flag turned off.
  flags: -integer-simple

```

When you run `cabal new-configure`, it writes out a `cabal.project.local` file which saves any extra configuration options from the command line; if you want to know how a command line arguments get translated into a `cabal.project` file, just run `new-configure` and inspect the output.

### Known issues

As a tech preview, the code is still [a little rough around the edges](https://github.com/haskell/cabal/labels/nix-local-build). Here are some more major issues you might run into:

*   Although dependency resolution is deterministic, if you update your Hackage index with `cabal update`, [dependency resolution will change too](https://github.com/haskell/cabal/issues/2996). There is no `cabal new-freeze`, so you'll have to manually construct the set of desired constraints.
*   A new feature of new-build is that it avoids rebuilding packages when there have been no changes to them, by tracking the hashes of their contents. However, this dependency tracking is not 100% accurate (specifically, it relies on your Cabal file accurately reporting all file dependencies ala `sdist`, and it doesn't know about search paths). There's currently no UI for forcing a package to be recompiled; however you can induce a recompilation fairly easily by removing an appropriate cache file: specifically, for the package named `p-1.0`, delete the file `dist-newstyle/build/p-1.0/cache/build`.
*   On Mac OS X, Haskell Platform, you may get the message “Warning: The package list for 'hackage.haskell.org' does not exist. Run 'cabal update' to download it.” That is [issue #3392](https://github.com/haskell/cabal/issues/3392); see the linked ticket for workarounds.

If you encounter other bugs, please let us know on [Cabal's issue tracker](https://github.com/haskell/cabal/issues/new?labels=nix-local-build).