<!--yml
category: 未分类
date: 2024-07-01 18:17:05
-->

# The Edit-Recompile Manager : ezyang’s blog

> 来源：[http://blog.ezyang.com/2016/09/the-edit-recompile-manager/](http://blog.ezyang.com/2016/09/the-edit-recompile-manager/)

## The Edit-Recompile Manager

A common claim I keep seeing repeated is that there are too many language-specific package managers, and that we should use a distribution's package manager instead. As an example, I opened the most recent [HN discussion](https://news.ycombinator.com/item?id=12187888) related to package managers, and sure enough the [third comment](https://news.ycombinator.com/item?id=12189483) was on this (very) dead horse. ([But](https://news.ycombinator.com/item?id=12026745) [wait!](https://news.ycombinator.com/item?id=11469315) [There's](https://news.ycombinator.com/item?id=11088125) [more](https://news.ycombinator.com/item?id=10662927).) But it rarely feels like there is any forward progress on these threads. Why?

Here is my hypothesis: these two camps of people are talking past each other, because the term "package manager" has been overloaded to mean two things:

1.  For end-users, it denotes an install manager, primarily responsible for *installing* some useful software so that they can use it. Software here usually gets installed once, and then used for a long time.
2.  For developers, it denotes an **edit-recompile manager**: a piece of software for letting you take a software project under development and (re)build it, as quickly as possible. The installation of packages is a *means*, but it is not the *end*.

It should be clear that while these two use-cases have some shared mechanism, the priorities are overwhelmingly different:

*   End-users don't care about how a package is built, just that the things they want to install have been built. For developers, speed on rebuild is an *overriding* concern. To achieve this performance, a deep understanding of the structure of the programming language is needed.
*   End-users usually just want one version of any piece software. Developers use multiple versions, because that is the cost of doing business with a diverse, rapidly updated, decentralized package ecosystem.
*   End-users care about it "just working": thus, a distribution package manager emphasizes control over the full stack (usually requiring root.) Developers care about flexibility for the software they are rebuilding and don't mind if a little setup is needed.

So the next time someone says that there are too many language-specific package managers, mentally replace "package manager" with "edit-recompile manager". Does the complaint still make sense? Maybe it does, but not in the usual sense: what they may actually be advocating for is an *interface* between these two worlds. And that seems like a project that is both tractable and worth doing.