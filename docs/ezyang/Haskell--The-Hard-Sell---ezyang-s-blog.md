<!--yml
category: 未分类
date: 2024-07-01 18:18:25
-->

# Haskell, The Hard Sell : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/03/haskell-the-hard-sell/](http://blog.ezyang.com/2010/03/haskell-the-hard-sell/)

## Haskell, The Hard Sell

Last week I talked about how [we replaced a small C program with an equivalent piece of Haskell code.](http://blog.ezyang.com/2010/03/replacing-small-c-programs-with-haskell/) As much as I'd like to say that we deployed the code and there was much rejoicing and client side caching, the real story is a little more complicated than that. There were some really good questions that we had to consider:

1.  *How many maintainers at any given time know the language?* The [Scripts](http://scripts.mit.edu) project is student-run, and has an unusually high turnover rate: any given maintainer is only guaranteed to be around for four to five years (maybe a little longer if they stick around town, but besides a few notable exceptions, most people move on after their time as a student). This means at any given point we have to worry about whether or not the sum knowledge of the active contributors is enough to cover all facets of the system, and facility in a language is critical to being able to administrate the component effectively (students we are, we frequently don both the sysadmin and developer hats). In a corporate setting, this is less prominent, but it still plays a factor: employees switch from one group to another and eventually people leave or retire. We have two current maintainers who are fairly fluent in Haskell. The long-term sustainability of this approach is uncertain, and hinges on our ability to attract prospective students who know or are interested in learning Haskell; in the worst case, people may crack open the code, say "what the fuck is this" and rewrite it in another language.

2.  *How many maintainers at any given time feel comfortable hacking in the language?* While superficially similar to the first point, it's actually quite different; posed differently, it's the difference between "can I write a full program in this language" and "can I effectively make changes to a program written in this language." At a certain level of fluency, a programmer picks up a special feat: the ability to look at any C/Fortran derived language and lift any knowledge they need about the syntax from the surrounding code. It's the difference between learning syntax, and learning a new programming paradigm. We may not be simultaneously Python/Perl/PHP/Java/Ruby/C experts, but the lessons in these languages carry over to one another, and many of us have working "hacker" knowledge in all of them. But Haskell is different: it's lineage is among that of Lisp, Miranda and ML, and the imperative knowledge simply *does not translate.* One hopes that it's still possible to tell what any given chunk of Haskell code does, but it's a strictly read-only capability.

3.  *Who else uses it?* For one of the team members, migrating from Subversion to Git was a pretty hard sell, but at this point, minus the missing infrastructure for doing the migration properly, he's basically been convinced that this is the right way forward. One of the big reasons this was ok, though, was because they were able to list of projects (Linux, our kernel; AFS, our filesystem; Fedora, our distro) that they used regularly that also used Git. We can't say the same for Haskell: the "big" open-source high-visibility applications in Haskell are Xmonad and Darcs, of which many people have never used. As a student group, we have far more latitude to experiment with new technology, but lack of ubiquity means greater risk, and corporations are allergic to that kind of risk.

4.  *Is the ecosystem mature?* Internally, we've given the Ruby maintainers and packagers a lot of flak for a terrible record at backwards compatibility (one instance left us unable to globally update our Rails instances because the code would automatically break the site if it detected a version mismatch). You see a little bit of the same in Haskell: static-cat doesn't actually build on a stock Fedora 11 server with the default packages installed, due to an old version of the cgi module that uses the Exception backwards compatibility wrapper and thus is incompatible with the rest of the exception handling code in the program. Further investigation reveals that the cgi module is not actually being actively maintained, and the Fedora `cabal2spec` script is buggy. I've personally had experiences of coming back to some Haskell code with up-to-date libraries from Hackage and finding that API drift has made my code not compile anymore. Cabal install refuses to upgrade all of your packages in one go.

    There are many ways to work around this. A mitigating factor is that once you've compiled a Haskell program, you don't have to worry about package composition anymore. Workarounds include rewriting our code to be forwards and backwards compatible, doing stupid Fedora packaging tricks to make both versions of cgi live on our servers, convincing upstream that they really want to take the new version, or maintaining a separate system wide cabal install. But it's not ideal, and it makes people wonder.

I'm quite blessed to be working in an environment where the first point is really *the* point. Can we introduce Haskell into the codebase and expect to be able to maintain it in the long run? There'll always be C hackers on the team (or at least, there better be; some of our most important security properties are wrapped up in a patch to a kernel module), but will there always be Haskell hackers on the team? There's no way to really know the answer to the question.

I personally remain optimistic. It's an experiment, and you're not going to get any better chance to make this happen than in this environment. The presence of Haskell code may attract contributors to the project that may not have been originally drawn by the fact that, down beneath it all, we're a "gratis shared web hosting provider" for our community. Haskell seems singularly aligned to be *the* language to break into mainstream (sorry Simon!) And when was there ever any innovation without a little risk?