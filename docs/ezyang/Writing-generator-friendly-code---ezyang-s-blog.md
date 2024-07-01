<!--yml
category: 未分类
date: 2024-07-01 18:18:26
-->

# Writing generator friendly code : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/03/writing-generator-friendly-code/](http://blog.ezyang.com/2010/03/writing-generator-friendly-code/)

## Writing generator friendly code

I've come a long ways from [complaining to the html5lib list that the Python version gratuitously used generators, making it hard to port to PHP](http://www.mail-archive.com/html5lib-discuss@googlegroups.com/msg00241.html). Having now drunk the laziness kool-aid in Haskell, I enjoy trying to make my code fit the generator idiom. While Python generators have notable downsides compared to infinite lazy lists (for example, forking them for multiple use is nontrivial), they're pretty nice.

Unfortunately, the majority of code I see that expects to see lists isn't robust enough to accept generators too, and it breaks my heart when I have to say `list(generator)`. I'll forgive you if you're expecting O(1) accesses of arbitrary indexes in your internal code, but all too often I see code that only needs sequential access, only to botch it all up by calling `len()`. Duck typing won't save you there.

The trick for making code generator friendly is simple: **use the iteration interface.** Don't mutate the list. Don't ask for arbitrary items. Don't ask for the length. This also is a hint that `for range(0, len(l))` is *absolutely* the wrong way to traverse a list; if you need indices, use `enumerate`.

**Update (September 1, 2012).** Hilariously enough, PHP has [finally gotten generators.](https://wiki.php.net/rfc/generators#vote)