<!--yml
category: 未分类
date: 2024-07-01 18:17:21
-->

# HotOS “Unconference” report:Verifying Systems : ezyang’s blog

> 来源：[http://blog.ezyang.com/2013/05/hotos-unconference-reportverifying-systems/](http://blog.ezyang.com/2013/05/hotos-unconference-reportverifying-systems/)

[Ariel Rabkin](http://www.eecs.berkeley.edu/~asrabkin/) has some code he'd like to verify, and at this year’s HotOS he appealed to participants of one “unconference” (informal breakout sessions to discuss various topics) to help him figure out what was really going on as far as formal verification went.

He had three questions: "What can we verify? What is impossible to verify? How can we tell that the verification is correct?" They seemed like impossibly large questions, so we drilled in a little bit, and found that Ariel had the very understandable question, "Say I have some C++ code implementing a subtle network protocol, and I'd like to prove that the protocol doesn't deadlock; how do I do that?"

I wish the formal verification community had a good answer to a question like this, but unfortunately we don't. The largest verification projects include things like verified kernels, which are written in fully specified subsets of C; which assume the translation performed by the compiler is correct, formalize C in a theorem prover, and then verify there. This is the "principled approach". It's just not feasible to take C or C++ in its entirety and try to formalize it; it's too complicated and too ill-specified. The easiest thing to do is formalize a small fragment of your algorithm and then make a hand-wavy argument that your implementation is adequate.

[Martin Abadi](http://users.soe.ucsc.edu/~abadi/home.html) remarked that before you embark on a verification project, you have to figure out where you'll get the most bang for your buck. Most of the time, a formalization won't get you "full correctness"; the "electrons may be faulty", as the case may be. But even a flawed verification forces you to state your assumptions explicitly, which is a good thing.

We then circled around to the subject of, well, what can be verified. Until the 90s, the formal verification community limited itself to only complete and sound analyses—and failed. The relaxation of this restriction lead to a renaissance of formal verification work. We talked about who was using formal verification, and the usual suspects showed up: safety critical software, cache coherence protocols (but one participant remarked that this was only a flash in the pan, as far as applications goes—he asserted that these companies are likely not going to use these methods any longer in the future), etc. Safety critical software is also likely to use coprocessors (since hardware failure is a very real issue), but [Gernot Heiser](http://www.cse.unsw.edu.au/~gernot/) noted that these folks are trying to get away from physical separation: it is expensive in terms of expense, weight and energy. Luckily, the costs of verification, as he recounted, are within a factor of two of normal industrial assurance, and half the cost of military assurance (though, he cautioned that this was for a specific project, and for a specific size of code.) He also remarked that as far as changes to code requiring changes to the proofs, the changes in the proofs seemed to be linear in the complexity (conceptual or implementation-wise) of the change, which is a good sign!

Well, supposing that you decide that you actually want to verify your software, how do you go about doing it? Unfortunately, it takes a completely different set of skills to build verified software versus normal software. Everyone agreed, "Yes, you need to hire a formal methods guy" if you're going to make any progress. But that's just not enough. The formal methods guy has to talk to the systems guy. Heiser recounted a very good experience hiring a formal methods person who was able to communicate with the other systems researchers working on the project; without this line of communication, he said, the project likely would have failed. And he mentioned another project, which had three times as much funding, but didn't accomplish nearly as much their team had. (Names not mentioned to protect the guilty.)

In the end, it seemed that we didn’t manage to give Ari a quite satisfactory answer. As one participant said, “You’ll probably learn the most by just sitting down and trying to formalize the thing you are interested in.” This is probably true, though I fear most will be scared off by the realization of how much work it actually takes to formalize software.

* * *

Hey guys, I’m [liveblogging HotOS at my research Tumblr](http://ezyang.tumblr.com). The posts there are likely to be more fragmented than this, but if people are interested in any particular topics I can inflate them into full posts.