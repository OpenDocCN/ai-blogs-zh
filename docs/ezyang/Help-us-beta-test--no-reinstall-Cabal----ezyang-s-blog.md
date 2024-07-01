<!--yml
category: 未分类
date: 2024-07-01 18:17:06
-->

# Help us beta test “no-reinstall Cabal” : ezyang’s blog

> 来源：[http://blog.ezyang.com/2015/08/help-us-beta-test-no-reinstall-cabal/](http://blog.ezyang.com/2015/08/help-us-beta-test-no-reinstall-cabal/)

## Help us beta test “no-reinstall Cabal”

Over this summer, Vishal Agrawal has been working on a GSoC project to [move Cabal to more Nix-like package management system](https://ghc.haskell.org/trac/ghc/wiki/Commentary/GSoC_Cabal_nix). More simply, he is working to make it so that you'll never get one of these errors from cabal-install again:

```
Resolving dependencies...
In order, the following would be installed:
directory-1.2.1.0 (reinstall) changes: time-1.4.2 -> 1.5
process-1.2.1.0 (reinstall)
extra-1.0 (new package)
cabal: The following packages are likely to be broken by the reinstalls:
process-1.2.0.0
hoogle-4.2.35
haskell98-2.0.0.3
ghc-7.8.3
Cabal-1.22.0.0
...

```

However, these patches change a nontrivial number of moving parts in Cabal and cabal-install, so it would be very helpful to have willing guinea pigs to help us iron out some bugs before we merge it into Cabal HEAD. As your prize, you'll get to run "no-reinstall Cabal": Cabal should **never** tell you it can't install a package because some reinstalls would be necessary.

Here's how you can help:

1.  Make sure you're running GHC 7.10\. Earlier versions of GHC have a hard limitation that doesn't allow you to reinstall a package multiple times against different dependencies. (Actually, it would be useful if you test with older versions of GHC 7.8, but only mostly to make sure we haven't introduced any regressions here.)
2.  `git clone https://github.com/ezyang/cabal.git` (I've added some extra corrective patches on top of Vishal's version in the course of my testing) and `git checkout cabal-no-pks`.
3.  In the `Cabal` and `cabal-install` directories, run `cabal install`.
4.  Try building things without a sandbox and see what happens! (When I test, I've tried installing multiple version of Yesod at the same time.)

It is NOT necessary to clear your package database before testing. If you completely break your Haskell installation (unlikely, but could happen), you can do the old trick of clearing out your `.ghc` and `.cabal` directories (don't forget to save your `.cabal/config` file) and rebootstrapping with an old `cabal-install`.

Please report problems here, or to [this PR in the Cabal tracker](https://github.com/haskell/cabal/pull/2752). Or chat with me in person next week at ICFP. :)