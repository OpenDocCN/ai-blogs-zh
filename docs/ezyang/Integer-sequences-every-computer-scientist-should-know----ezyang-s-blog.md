<!--yml
category: 未分类
date: 2024-07-01 18:18:02
-->

# Integer sequences every computer scientist should know? : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/11/integer-sequences-every-computer-scientist-should-know/](http://blog.ezyang.com/2010/11/integer-sequences-every-computer-scientist-should-know/)

The [On-Line Encyclopedia of Integer Sequences](http://oeis.org/Seis.html) is quite a nifty website. Suppose that you’re solving a problem, and you come up with the following sequence of integers: `0, 1, 0, 2, 0, 1, 0, 3, 0, 1, 0, 2...` and you wonder to yourself: “huh, what’s that sequence?” Well, just [type it in](http://oeis.org/search?q=0%2C+1%2C+0%2C+2%2C+0%2C+1%2C+0%2C+3%2C+0%2C+1%2C+0%2C+2&sort=&language=english&go=Search) and the answer comes back: [A007814](http://oeis.org/A007814), along with all sorts of tasty tidbits like constructions, closed forms, mathematical properties, and more. Even simple sequences like [powers of two](http://oeis.org/A000079) have a bazillion alternate interpretations and generators.

This got me wondering: what are integer sequences that every computer scientist should know? That is, the ones they should be able to see the first few terms of and think, “Oh, I know that sequence!” and then rack their brains for a little bit trying to remember the construction, closed form or some crucial property. For example, almost anyone with basic math background will recognize the sequences [1, 2, 3, 4, 5](http://oeis.org/A000027); [0, 1, 4, 9, 16](http://oeis.org/A000290); or [1, 1, 2, 3, 5](http://oeis.org/A000045). The very first sequence I cited in this article holds a special place in my heart because I accidentally derived it while working on my article [Adventures in Three Monads](http://blog.ezyang.com/2010/01/adventures-in-three-monads/) for The Monad.Reader. Maybe a little less familiar might be [1, 1, 2, 5, 14, 42](http://oeis.org/A000108) or [3, 7, 31, 127, 8191, 131071, 524287, 2147483647](http://oeis.org/A000668), but they are still quite important for computer scientists.

So, what integer sequences every computer scientist should know? (Alternatively, what’s your favorite integer sequence?)