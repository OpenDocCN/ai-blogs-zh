<!--yml
category: 未分类
date: 2024-07-01 18:17:40
-->

# Polyglot programming : ezyang’s blog

> 来源：[http://blog.ezyang.com/2011/10/polyglot-programming/](http://blog.ezyang.com/2011/10/polyglot-programming/)

## Polyglot programming

Being back in town over MIT's *Infinite Activities Period* is making me think about what kind of short lecture I want to try teaching. I've been turning over the idea of a polyglot programming class in my head: the idea is that while most people feel really comfortable with only one or two programming languages, you can learn how to make this knowledge work for you in almost any programming language you could possible language.

Unfortunately, I don’t have a good idea of what these skills actually are, nor do I have a sense for what kinds of things people would want to know. Nor do I think that I could jam this into two hours of lecturing: topics that I probably would want to cover are:

*History of Programming Languages.* Knowing how all the lineages tie together will help you figure out when a language feature will work the way you expect it to (since they just stole it from another language in the same line), and when, actually, it won’t work at all. It lets you nicely encapsulate the main big ideas of language features, which you can then explore the infinite variations of. It gives you groups of languages which mostly have the same idioms.

*Street smarts and bootstrapping.* What are the first things you should look for when you’re getting acquainted with a new language? Syntax? Cheat sheets? Tutorials? How to organize this torrent of information, what to do first, where to ask questions, what to learn how to do. How to interpret error messages you know nothing about. How to navigate the development ecosystem and assess libraries you know nothing about. How to source dive code in languages you know nothing about. Common bumps on the road towards Hello World. Unusual and universal ways of debugging.

*Interoperability and FFI.* What are the basic building blocks for higher-level data types in most of these languages, and what do they look like in memory? How do you make lots of different languages talk to each other efficiently! How about garbage collection, reference pinning and concurrency? Common impedance mismatches between languages.

Suggestions and comments would be appreciated.