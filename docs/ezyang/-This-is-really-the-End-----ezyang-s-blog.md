<!--yml
category: 未分类
date: 2024-07-01 18:17:27
-->

# “This is really the End.” : ezyang’s blog

> 来源：[http://blog.ezyang.com/2012/09/feit-thompson-true/](http://blog.ezyang.com/2012/09/feit-thompson-true/)

*Done.* This adjective rarely describes any sort of software project—there are always more bugs to fix, more features to add. But on September 20th, early one afternoon in France, Georges Gonthier did just that: he slotted in the last component of a six year project, with the laconic commit message, "This is really the End."

It was complete: [the formalization of the Feit-Thompson theorem.](http://www.msr-inria.inria.fr/events-news/feit-thompson-proved-in-coq)

If you’re not following the developments in interactive theorem proving or the formalization of mathematics, this achievement may leave you scratching your head a little. What is the Feit-Thompson theorem? What does it mean for it to have been formalized? What was the point of this exercise? Unlike the [four coloring theorem](http://en.wikipedia.org/wiki/Four_color_theorem) which Gonthier and his team tackled previously in 2005, the Feit-Thompson theorem (also known as the odd order theorem) is not easily understandable by non-mathematician without a background in group theory (I shall not attempt to explain it). Nor are there many working mathematicians whose lives will be materially impacted by the formalization of this particular theorem. But there is a point; one that can be found between the broad motivation behind the computer-assisted theorem proving and the fascinating social context surrounding the Feit-Thompson theorem.

While the original vision of automated theorem proving, articulated by David Hilbert, was one where computers completely replaced mathematicians in the work of proving theorems (mathematicians would be consigned to dreaming up interesting theorem statements to prove), modern researchers in the formalization of mathematics consider the more modest question of whether or not a proof is true. That is to say, humans are fallible and can make mistakes while reviewing the proofs of their colleagues, while the computer is an idiot-savant who, once spoon-fed the contents of a proof, is essentially infallible in its judgment of correctness. The proofs of undergraduate mathematics classes don’t really need this treatment (after all, they have been inflicted on countless math majors over the centuries), but there are a few areas where this extra boost of confidence is really appreciated:

*   In theorems which are found in conventional software systems, due to the large number of incidental details making proofs superficial but large,
*   In extremely tricky, consistently unintuitive areas of mathematics, including questions of provability and soundness, and
*   In extremely long and involved proofs, for which traditional verification by human mathematicians is a Herculean task.

It is in this last area that we can really place both the Four Color Theorem and the Feit-Thompson Theorem. The Four Color Theorem’s original proof was controversial because of its reliance on a computer to solve 1936 special cases which the proof depended on. The Feit-Thompson Theorem is a bit of an annoyance to many mathematicians, due to its length: 255 pages, the global structure of which has resisted over half a century of attempts at simplification. And it itself is just a foothill on the way to the monster theorem that is the classification of finite simple groups (comprising tens of thousands of pages of proof.)

Formalizing the entirety of the Feit-Thompson is a technical achievement which applies computer-assisted theorem proving to a tremendously nontrivial mathematical proof, and does so in a setting where the ability to state “Feit-Thompson is True” has non-zero information content. It demonstrates that theorem proving in Coq *does* scale (at least, well enough pull off a proof of this scale) and it has generated a large toolbox for handling features of real mathematics including heavily overloaded notation and handing the composition of many, disparate theories (all of which the proof draws upon).

Of course, there is one niggling concern: what if there is a bug, and what Gonthier and his team have proved is not *actually* the Feit-Thompson Theorem? A story I once heard while I was at Galois was of a research intern who had been tasked with proving some theorems. He had some trouble at first, but eventually pulled off the first theorem and moved to the next, gradually gaining speed. Eventually, his advisor took a look at these proofs and realized that he had been taking advantage of a soundness problem in the proof checker to solve the proofs. I wonder if this is a story that wizened PhD students tell the first years to give them nightmares.

But I think the risk of this is very low. The mechanized proof follows the original quite closely, and so a wrong definition or a soundness bug is likely only to pose a local problem for the proof that can be easily solved. Indeed, if there is a problem with the proof that is so great that the proof must be thrown out and done again, that would be very interesting—it would say something about the original proof as well. That would be surprising, and worthy of attention, for it could mean the end of the proof of the Feit-Thompson Theorem as we know it. But I don’t think we live in that world.

Pop the champagne, and congratulations to Gonthier and his team!