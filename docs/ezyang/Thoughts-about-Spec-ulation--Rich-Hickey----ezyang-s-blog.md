<!--yml
category: 未分类
date: 2024-07-01 18:17:03
-->

# Thoughts about Spec-ulation (Rich Hickey) : ezyang’s blog

> 来源：[http://blog.ezyang.com/2016/12/thoughts-about-spec-ulation-rich-hickey/](http://blog.ezyang.com/2016/12/thoughts-about-spec-ulation-rich-hickey/)

Rich Hickey recently gave a [keynote](https://www.youtube.com/watch?v=oyLBGkS5ICk) at Clojure/conj 2016, meditating on the problems of versioning, specification and backwards compatibility in language ecosystems. In it, Rich considers the ["extremist" view](http://blog.ezyang.com/2012/11/extremist-programming/), *what if we built a language ecosystem, where you never, ever broke backwards compatibility.*

A large portion of the talk is spent grappling with the ramifications of this perspective. For example:

1.  Suppose you want to make a backwards-compatibility breaking change to a function. Don't *mutate* the function, Richard says, give the function another name.
2.  OK, but how about if there is some systematic change you need to apply to many functions? That's still not an excuse: create a new namespace, and put all the functions there.
3.  What if there's a function you really don't like, and you really want to get rid of it? No, don't remove it, create a new namespace with that function absent.
4.  Does this sound like a lot of work to remove things? Yeah. So don't remove things!

In general, Rich wants us to avoid breakage by turning all changes into *accretion*, where the old and new can coexist. "We need to bring functional programming [immutability] to the library ecosystem," he says, "dependency hell is just mutability hell." And to do this, there need to be tools for you to make a commitment to what it is that a library provides and requires, and not accidentally breaking this commitment when you release new versions of your software.

He says a lot more in the talk, so I encourage you to give it a watch if you want to hear the whole picture.

* * *

In general, I'm in favor of this line of thinking, because my feeling is that a large amount of breakage associated with software change that is just a product of negligence; breakage not for any good reason, breakage that could have been avoided if there was a little more help from tooling.

That being said, I do have some thoughts about topics that are not so prominently featured in his talk.

**Accretion is not a silver bullet... if you believe in data hiding.** In his talk, Rich implies that backwards compatibility can be maintained simply by committing not to "remove things". As a Haskeller, this sounds obviously false to me: if I change the internal representation of some abstract type (or even the internal invariants), I *cannot* just load up both old and new copies of the library and expect to pass values of this type between the two. Indeed, the typechecker won't even let you do this even if the representation hasn't changed.

But, at least for Clojure, I think Rich is right. The reason is this: [Clojure doesn't believe data hiding](http://codequarterly.com/2011/rich-hickey/)! The [prevailing style](http://clojure.org/reference/datatypes) of Clojure code is that data types consist of immutable records with public fields that are passed around. And so a change to the representation of the data is a possibly a breaking change; non-breaking representation changes are simply not done. (I suspect a similar ethos explains why [duplicated dependencies in node.js](http://stackoverflow.com/questions/25268545/why-does-npms-policy-of-duplicated-dependencies-work) work as well as they do.)

I am not sure how I feel about this. I am personally a big believer in data abstraction, but I often admire the pragmatics of "everything is a map". (I [tweeted](https://twitter.com/ezyang/status/809704816150597633) about this earlier today, which provoked some thoughtful discussion.)

**Harmful APIs.** At several points in the talk, Rich makes fun of developers who are obsessed with taking away features from their users. ("I hate this function. I hate it, I hate it, I hate that people call it, I just want it out of my life.") This downplays the very real, very important reasons why infinite backwards compatibility has been harmful to the software we write today.

One need look no further than the [systems with decades of backwards compatibility](https://youtu.be/oyLBGkS5ICk?t=1h8m18s) that Rich cites: the Unix APIs, Java and HTML. In all these cases, backwards compatibility has lead to harmful APIs sticking around far longer than they should: [strncpy](https://randomascii.wordpress.com/2013/04/03/stop-using-strncpy-already/), [gets](http://stackoverflow.com/questions/1694036/why-is-the-gets-function-so-dangerous-that-it-should-not-be-used), legacy parsers of HTML (XSS), [Java antipatterns](http://www.odi.ch/prog/design/newbies.php), etc. And there are examples galore in Android, C libraries, everywhere.

In my opinion, library authors should design APIs in such a way that it is easy to do the right thing, and hard to do the wrong thing. And yes, that means sometimes that means you that you need to stop people from using insecure or easy-to-get wrong library calls.

**Semantic versioning doesn't cause cascading version bumps, lack of version ranges is the cause.** In the slide ["Do deps force Versioning?"](https://youtu.be/oyLBGkS5ICk?t=13m49s), Rich describe a problem in the Clojure ecosystem which is that, when following semantic versioning, a new release of a package often causes cascading version bumps in the system.

While the problem of cascading version bumps is [a real question](https://github.com/mojombo/semver/issues/148) that applies to semantic versioning in general, the "cascading version bumps" Rich is referring to in the Clojure ecosystem stem from a much more mundane source: best practices is to [specify a specific version of a dependency](https://nelsonmorris.net/2012/07/31/do-not-use-version-ranges-in-project-clj.html) in your package metadata. When a new version of a dependency comes out, you need to bump the version of a package so that you can update the recorded version of the dependency... and so forth.

I'm not saying that Clojure is *wrong* for doing things this way (version ranges have their own challenges), but in his talk Rich implies that this is a failure of semantic versioning... which it's not. If you use version ranges and aren't in the habit of reexporting APIs from your dependencies, updating the version range of a dependency is not a breaking change. If you have a solver that picks a single copy of a library for the entire application, you can even expose types from your dependency in your API.

* * *

Overall, I am glad that Clojure is thinking about how to put backwards compatibility first and foremost: often, it is in the most extreme applications of a principle that we learn the most. Is it the end of the story? No; but I hope that all languages continue slowly moving towards explicit specifications and tooling to help you live up to your promises.