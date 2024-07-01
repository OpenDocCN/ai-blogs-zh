<!--yml
category: 未分类
date: 2024-07-01 18:17:56
-->

# Interleaving stream processors : ezyang’s blog

> 来源：[http://blog.ezyang.com/2011/03/interleaving-stream-processor/](http://blog.ezyang.com/2011/03/interleaving-stream-processor/)

*Ghost in the state machine*

A long time ago (circa 2007-2008), I wrote perhaps the single most complicated piece of code in HTML Purifier—one of those real monsters that you don’t think anyone else could ever understand and that you are really grateful you have a comprehensive test suite for. The idea was this: I had a state machine that modified a stream of tokens (since this was a stream of HTML tags and text, the state machine maintained information such as the current nesting stack), and I wanted to allow users to add extra functionality on top of this stream processor (the very first processor inserted paragraph tags when double-newlines were encountered) in a modular way.

The simplest thing I could have done was abstract out the basic state machine, created a separate processor for every transformation I wanted to make, and run them one pass after another. But for whatever reason, this idea never occurred to me, and I didn’t want to take the performance hit of having to iterate over the list of tokens multiple times (premature optimization, much?) Instead, I decided to add hooks for various points in the original state machine, which plugins could hook into to do their own stream transformations.

Even before I had more than one “injector”, as they are called, I had decided that I would have to do something sensible when one injector created a token that another injector could process. Suppose you ran injector A, and then injector B. Any tokens that injector A created would be picked up by injector B, but not vice versa. This seemed to me a completely arbitrary decision: could I make it so that order didn't matter? The way I implemented this was have the managing state machine figure out what new tokens any given injector had created, and then pass them to all the other injectors (being careful not to pass the token back to the originating injector.) Getting this right was tricky; I originally stored the ranges of “already seen” tokens separately from the token stream itself, but as other injectors made changes it was extremely easy for these ranges to get out of sync, so I ended up storing the information in the token themselves. Another difficulty is preventing A from creating a token which B converts into another token which A converts to B etc; so this skip information would have to be preserved over tokens. (It seems possible that this excluded some possible beneficial interactions between two injectors, but I decided termination was more important.)

Extra features also increased complexity. One particular feature needed the ability to rewind back to an earlier part of the stream and reprocess all of those tokens; since most other injectors wouldn’t be expecting to go back in time, I decided that it would be simplest if other injectors were all suspended if a rewind occurred. I doubt there are any sort of formal semantics for what the system as a whole is supposed to do, but it seems to work in practice. After all, complexity isn’t created in one day: it evolves over time.

There are several endings to this story. One ending was that I was amused and delighted to see that the problem of clients making changes, which then are recursively propagated to other clients, which can make other changes, is a fairly common occurrence in distributed systems. If you compress this into algorithmic form, you get (gasp) research papers like Lerner, Grove and Chamber’s *Composing Dataflow Analyses and Transformations*, which I discovered when brushing against Hoopl (note: I am not belittling their work: dataflow analysis for programming languages is a lot more complicated than hierarchy analysis for HTML, and they allow for a change made by A to affect a change B makes that can affect A again: they cut the knot by ensuring that their analysis eventually terminates by bounding the information lattice—probably something to talk about in another post).

Another ending is that, fascinatingly enough, this complex system actually was the basis for the first [external contribution](http://htmlpurifier.org/phorum/read.php?5,2519,2519) to HTML Purifier. This contributor had the following to say about the system: “I have to say, I'm impressed with the design I see in HTML Purifier; this has been pretty easy to jump into and understand, once I got pointed to the right spot.” The evolution of systems with complex internal state is, apparently, quite well understood by practicing programmers, and I see [experienced developers tackling this subsystem](http://stackoverflow.com/questions/2638640/html-purifier-removing-an-element-conditionally-based-on-its-attributes), usually with success. From an experience standpoint, I don’t find this too surprising—years after I originally wrote the code, it doesn’t take me too long to recall what was going on. But I do wonder if this is just the byproduct of many long hacking sessions on systems with lots of state.

The final ending is a what-if, the “What if the older Edward came back and decided to rewrite the system.” It seems strange, but I probably wouldn’t have the patience to do it over again. Or I might have recognized the impending complexity and avoided it. But probably the thing that has driven me the most crazy over the years, and which is a technical problem in its own right, is despite the stream-based representation, everything HTML Purifier processes is loaded up in memory, and we don't take advantage of streams of token at all. Annoying!