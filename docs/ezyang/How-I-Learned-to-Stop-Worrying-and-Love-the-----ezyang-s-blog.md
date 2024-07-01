<!--yml
category: 未分类
date: 2024-07-01 18:18:01
-->

# How I Learned to Stop Worrying and Love the ⊥ : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/12/how-i-learned-to-stop-worrying-and-love-the-bottom/](http://blog.ezyang.com/2010/12/how-i-learned-to-stop-worrying-and-love-the-bottom/)

*An extended analogy on the denotational and game semantics of ⊥*

This is an attempt at improving on the [Haskell Wikibooks article on Denotational Semantics](http://en.wikibooks.org/wiki/Haskell/Denotational_semantics) by means of a Dr. Strangelove inspired analogy.

* * *

*The analogy.* In order to prevent Brigadier General Jack D. Ripper from initiating a nuclear attack on Russia, the Pentagon decides that it will be best if every nuclear weapon requires two separate keys in order to be activated, both of which should not be known by the same person at the same time under normal circumstances. Alice is given one half of the key, Bob the other half. If Ripper asks Alice for her half of the key, she can tell him her key, A. However, asking Alice for Bob’s key won’t work, since she doesn't know what Bob's key is.

Suppose Ripper asked Alice anyway, and she told him "I don't know Bob's key." In this case, Ripper now have a concrete piece of information: Alice does not have Bob's key. He can now act accordingly and ask Bob for the second key. But suppose that, instead of telling him outright that she didn't know the key, she told him, "I can tell you, but can you wait a little bit?" Ripper decides to wait—he’d probably have a time convincing Bob to hand over the key. But Alice never tells Ripper the key, and he keeps waiting. Even if Ripper decides to eventually give up waiting for Alice, it’s a lot harder for him to strategize when Alice claims she has the key but never coughs it up.

Alice, curious what would happen if she tried to detonate the nuclear bomb, sets off to talk to Larry who is responsible for keying in the codes. She tells the technician, “I have Alice’s key and I have Bob’s key.” (We, of course, know that she doesn’t actually have Bob’s key.) Larry is feeling lazy, and so before asking Alice for the keys, he phones up the Pentagon and asks if nuclear detonation is permitted. It is not, and he politely tells Alice so. Unruffled, Alice goes off and finds Steve, who can also key in the codes. She tells Steve that she has Alice’s key and Bob’s key. Steve, eager to please, asks Alice, “Cool, please tell me your key and Bob’s key.” Alice hands over her key, but stops on Bob’s key, and the conversation never finishes.

Nevertheless, despite our best efforts, Ripper manages to get both keys anyway and the world is destroyed in nuclear Armageddon anyway. ☢

* * *

*Notation.* Because this key is in two parts, it can be represented as a tuple. The full key that Ripper knows is (A, B), what Alice knows about the full key is (A, ⊥), and what Bob knows is (⊥, B). If I am (clueless) civilian Charlie, my knowledge might be (⊥, ⊥). We can intuitively view ⊥ as a placeholder for whenever something is not known. (For simplicity, the types of A and B are just unit.)

*I know more than you.* We can form a *partial ordering* of who knows more than who. Ripper, with the full key, knows more than Alice, Bob or Charlie. Alice knows more than Charlie, and Bob knows more than Charlie. We can’t really say that Alice knows more than Bob, or vice versa, since they know different pieces of data. ⊥ is at the bottom of this ordering because, well, it represents the least possible information you could have.

*The difference between nothing and bottom.* Things play out a bit differently when Alice says “I don’t know” versus when Alice endlessly delays providing an answer. This is because the former case is not bottom at all! We can see this because Alice actually says something in the first case. This something, though it is not the key, is information, specifically the Nothing constructor from Maybe. It would be much more truthful to represent Alice's knowledge as (Just A, Nothing) in this case. In the second case, at any point Alice *could* give a real answer, but she doesn’t.

*A strange game. The only winning move is not to play.* There is a lot of emphasis on people asking other people for pieces of information, and those people either responding or endlessly delaying. In fact, this corresponds directly to the notion of bottom from game semantics. When Ripper asks Alice for information about her key, we can write out the conversation as the sequence: “tell me the first value of the tuple”, “the value is A”, “tell me the second value of the tuple”, “...” Alice is speechless at the last question, because in game semantics parlance, she doesn’t have a strategy (the knowledge) for answering the question “tell me the second value of the tuple.” Clueless Charlie is even worse off, having no strategy for either question: the only time he is happy is if no one asks him any questions at all. He has the empty strategy.

*Don’t ask, don’t tell.* Consider function application. We might conceptualize this as “Here is the value A, here is the value B, please tell me if I can detonate the nuclear device.” This is equivalent to Steve’s strict evaluation. But we don’t have to setup the conversation this way: the conversation with Larry started off with, “I have the first key and I have the second key. Please tell me if I can detonate the nuclear device.” Larry might then ask Alice, “Ok, what is the first key?”—in particular, this will occur if Larry decides to do a case statement on the first key—but if Larry decides he doesn’t need to ask Alice for any more information, he won’t. This will make Charlie very happy, since he is only happy if he is not asked any questions at all.

*Ask several people at the same time.* In real life, if someone doesn’t give us an answer after some period of time, we can decide to stop listening and go do something else. Can programs do this too? It depends on what language you’re in. In Haskell, we can do this with nondeterminism in the IO monad (or push it into pure code with some caveats, as [unamb](http://haskell.org/haskellwiki/Unamb) does.)

*What’s not in the analogy.* Functions are data too: and they can be partially defined, e.g. partial functions. The fixpoint operator can be thought to use less defined versions of a function to make more defined versions. This is very cool, but I couldn’t think of an oblique way of presenting it. Omitted are the formal definitions from denotational semantics and game semantics; in particular, domains and continuous functions are not explained (probably the most important pieces to know, which can be obscured by the mathematical machinery that usually needs to get set up before defining them).

*Further reading.* If you think I’ve helped you’ve understand bottom, go double check your understanding of the [examples for newtype](http://www.haskell.org/haskellwiki/Newtype#Examples), perhaps one of the subtlest cases where thinking explicitly about bottom and about the conversations the functions, data constructors and undefineds (bottoms) are having. The strictness annotation means that the conversation with the data constructor goes something like “I have the first argument, tell me what the value is.” “Ok, what is the first argument?” These [notes on game semantics (PDF)](http://www.pps.jussieu.fr/~curien/Game-semantics.pdf) are quite good although they do assume familiarity with denotational semantics. Finding the formal definitions for these terms and seeing if they fit your intuition is a good exercise.