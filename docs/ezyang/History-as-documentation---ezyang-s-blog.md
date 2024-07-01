<!--yml
category: 未分类
date: 2024-07-01 18:18:26
-->

# History as documentation : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/02/history-as-documentation/](http://blog.ezyang.com/2010/02/history-as-documentation/)

## History as documentation

It's [real](http://developers.slashdot.org/story/09/11/16/1626218/If-the-Comments-Are-Ugly-the-Code-Is-Ugly) [easy](http://ask.slashdot.org/article.pl?sid=06/01/09/1544201) to [argue](http://developers.slashdot.org/story/10/01/01/226232/Myths-About-Code-Comments) about the utility, style and implementation of source code comments, those good ole' pals of code that try to add supplementary information when the [pure code isn't enough](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-7.html).

However, to focus solely on the latest snapshot of any particular source file is to miss a wealth of information that is not inside the file; namely, the history of the file and the genealogy of every line that graces the file. This is not so relevant when you are rapidly prototyping functionality and versions of the file in source control history represent incomplete, half-baked figments of thought, but once a codebase transitions into a more maintenance-oriented workflow, the history takes on a keen and unusual importance. In particular:

*   A log of the evolution of the file over time can illustrate what the *original* intent of the module was, and then how it got retrofitted or extended or hacked up over time. If you have inherited code from someone else that you need to rearchitect, what better way to get in the heads of the original designers than to study the revisions they went through.
*   Any particular line may have simply been part of the ambient code present during the initial check-in, or it may have been touched by a highly targeted commit addressing some issue. In this case, the output of `git blame` is highly relevant for identifying why that particular line might be special, or why a subtly different permutation is incorrect. In the case of delocalized changes, associating a line with a commit can give you the fast pass to understanding how one operation is orchestrated with many others for some global effect.

Developers should be highly encouraged to write impeccably descriptive commit messages (with the diff in hand: never write a commit message without the diff in hand) for the sake of those who may pick through the logs in the future. It's ok to even be a little wordier than you might be in an inline comment, since:

*   Log messages never grow old: they are always relevant to the revision they are attached to!
*   A good commit message facilitates code review, since it poses an informal specification of what the change does, which an external observer can then take and verify against the code. Otherwise, the reviewer would have to determine *both* the intended and actual semantics of the code, stylistic issues aside.

Finally, a few words about keeping the history clean and easy to use:

*   Logically organized patch sets mean that any given change is immediately relevant to the log message. If you push a big commit which contains lots of semantic changes, the reader has to disambiguate which particular semantic change is associated with which part of the diff. It is certainly worth your time to `git add -p` to stage hunks individually.
*   Make high quality diffs, which avoid touching unnecessary code. High traffic mailing lists such as LKML which receive many patches have published [patch submission guidelines](http://lxr.linux.no/#linux+v2.6.32/Documentation/SubmittingPatches) in order to make diffs as readable as possible to a possible reviewer. Even if you don't need to convince a temperamental upstream to take your changes, later in time you may care about your diffs.
*   Stylistic changes are highly disruptive the `git blame` output, since they result in a line being marked as changed even though no semantic difference took place. If you must, they should be strictly alone. Infrequent is best.
*   Utilize history rewriting to allow for cheap commits which are polished up later for submission.