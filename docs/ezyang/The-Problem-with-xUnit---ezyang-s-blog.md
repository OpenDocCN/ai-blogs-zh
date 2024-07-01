<!--yml
category: 未分类
date: 2024-07-01 18:18:23
-->

# The Problem with xUnit : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/04/the-problem-with-xunit/](http://blog.ezyang.com/2010/04/the-problem-with-xunit/)

Tagline: *Assertions considered not ideal.*

I think automated tests are great. I used two particular flavors of test, the unit test and the integration test, extensively in [HTML Purifier](http://htmlpurifier.org) and they're the only reason why I feel comfortable making changes to code that I first wrote in High School. The automated tests let me hack and then figure out if I broke anything with the single stroke of a button, rather than manually shove a few inputs in and see if they "look alright." They're also an informal specification of "what I wanted the code to do" when I originally wrote it, by the fine tradition of an example.

Both unit tests and integration tests were built on top of the [SimpleTest](http://simpletest.org/) "unit testing" library. I place the "unit testing" in quotes because, while SimpleTest is great for unit testing (the testing of individual components), it also can be used for integration testing (the testing of multiple components together) and system testing (the entire system, for web applications this commonly involves writing scripts to navigate the website); in fact, it has facilities in place to make the latter two easier to do!

Perhaps a more accurate description of SimpleTest as a whole is that it is a descendant of the xUnit testing framework. You know, the "make a test function that sets some stuff up, runs some code, and makes some asserts" style of testing. The idea of an assertion is essential; sans exception handling, that's your single portal into whether or not the test code failed or succeeded.

I was writing some tests in JUnit the other day, and it reminded me a little bit why, even though automated tests are great, I'm somewhat reluctant to roll them out in the first place. They're so verbose! Every test method I have to instantiate whatever class I want, do whatever initialization I need to it, create my input data (if I'm directly building it with `new`, this can easily take several lines), run the function, and then test if the output data is what I expected (either by laborious poking at the various fields and methods in it or, if I had the foresight to implement equality, construct the expected output result and compare them.) "But wait," you say, "that's precisely what `setUp` and `tearDown` are for!" and then you move chunks of this code into those methods, but the substantial bits of boilerplate for creating inputs and verifying results remain, and you are terrified of abstracting over them because adding more code means there's more chance for your test to be wrong!

But there's not a good way out of this mess, because the list of function calls to the unit under test is truly the "input" to your test suite, and then list of expressions passed into the assertions is truly the "output" of your test suite. The particular assertion you choose to use is the "expected value" of your test suite. So why does it feel like boilerplate?

Maybe because the model of setUp and tearDown methods and test methods and assertions is the wrong one for many types of code: the correct model is the input value, output value and expected value model! And for pure code, the code that actually has a more refined notion of its input and its output than "a code listing" and "the global state of the application after you ran the code listing"; maybe it truly is just "two integers" and "an integer." And then, the test code you write should *actually reflect that!*

So how do we make this happen? You want a DSL. Some languages are strong enough that you can get away with an embedded DSL of sorts. But many languages make this too cumbersome, so they invent their own test format and write the necessary boilerplate code to parse it and marshal it around. Obviously there need to be enough tests of this form to make writing all of this infrastructure worthwhile, and so when that's not true people fall back to the quick and dirty xUnit style of testing. But by doing this, you've obscured the shape of your test, and since "quick and dirty" never means "ephemeral", your test suite grows and grows and you never end up cutting over to the right way. Ever.

At this point, it's about time for a little Haskell advocacy. How can you make your tests lest cumbersome from the get go? Use a language that encourages the construction mini-DSLs. *Haskell has flexible syntax and type facilities to make this doable, check.* Use a language that encourages you to think carefully about functions, which have clear inputs and outputs, not classes and methods and mutable state. *Haskell is a functional programming language, check.* Use a language in which abstraction is cheap and boilerplate is killed with fire. *Haskell, check.* Use a language that, once you've gotten tired of writing input and output values over and over again, and *not* the boilerplate of an entire xUnit test case, gives you the rope to automate that process too! *QuickCheck and Haskell, check.*

It's also time for a little call to action: don't conflate the unit/acceptance/system testing hierarchy with the xUnit framework/boilerplate. There's xUnit testing and then there's fully randomized input generation ala QuickCheck, but there's still room in-between these two distinct places in abstraction for people and tests to live. And of course, the xUnit style test can be useful when a code listing truly is the right paradigm for the input representation.