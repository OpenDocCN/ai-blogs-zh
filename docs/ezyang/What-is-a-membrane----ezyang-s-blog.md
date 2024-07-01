<!--yml
category: 未分类
date: 2024-07-01 18:17:23
-->

# What is a membrane? : ezyang’s blog

> 来源：[http://blog.ezyang.com/2013/03/what-is-a-membran/](http://blog.ezyang.com/2013/03/what-is-a-membran/)

If you hang out long enough with a certain crowd (in my case, it was the [ECMAScript TC39 committee](http://wiki.ecmascript.org/doku.php)), you will probably hear the term **membrane** tossed around. And eventually, you will start to wonder, “Well, what *is* a membrane, anyway?”

As is the case with many clever but simple ideas, membranes were first introduced as a footnote [1] in [a PhD thesis.](http://www.erights.org/talks/thesis/) Suppose that you are building distributed system, in which you pass references to objects between two separate nodes. If I want to pass a reference to `foo` in process `A` to process `B`, I can hardly just hand over an address—the memory spaces are not the same! So instead, I need to create a wrapper object `wrappedFoo` representing `foo` in `B`, which knows how to access the original object in `A`. So far so good.

Now here’s the catch: what if I pass a reference to `wrappedFoo` back to process `A`? If I were not very clever, I’d do the same thing as I did originally: create a new wrapper object `wrappedWrappedFoo` in `A` which knows how to access `wrappedFoo` in `B`. But this is silly; really, when I cross back over to `A`, I want to get back the original `foo` object.

This wrap-unwrap behavior is *precisely* what a membrane is. We consider the original object `foo` to be “inside” the membrane (a so-called wet object), and as it exits the membrane, it is wrapped with its own little membrane. However, when the object returns to its original membrane, the wrapper goes away. Just like in biology!

There is one last operation, called a “gate”: this occurs when you invoke a method on a wrapped object. Since the wrapper cannot actually perform the method, it has to forward the request to the original object. However, the *arguments* of the method need to get wrapped (or unwrapped) as they get forwarded; as you might expect.

While I used an RPC-like system to demonstrate the basic principle of membranes, a more conventional use is to enforce access control. Membranes are quite important; [Mozilla](https://developer.mozilla.org/en-US/docs/XPConnect_security_membranes) relies on them extensively in order to enforce access restriction between objects from different websites which may interact with each other, but need security checks. (Actually, did you know that Mozilla is using a capability-based system for their security? Kind of neat!) It’s important to notice that when we unwrap, we are skipping security checks—the only reason this is acceptable is because the only objects that will be able to access the unwrapped object are precisely those objects in the same domain. For a more modern treatment of the subject, check out a more recent paper, [Trustworthy Proxies: Virtualizing Objects with Invariants](http://research.google.com/pubs/pub40736.html), which includes a lucid explanation of membranes.

[1] Well, actually it was a figure; figure 9.3 on page 71, to be precise!