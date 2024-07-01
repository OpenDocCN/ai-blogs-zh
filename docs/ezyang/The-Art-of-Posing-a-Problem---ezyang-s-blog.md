<!--yml
category: 未分类
date: 2024-07-01 18:18:26
-->

# The Art of Posing a Problem : ezyang’s blog

> 来源：[http://blog.ezyang.com/2010/02/art-of-posing-a-problem/](http://blog.ezyang.com/2010/02/art-of-posing-a-problem/)

## The Art of Posing a Problem

Last week, I was talking with [Alexey Radul](http://web.mit.edu/~axch/www/) to figure out some interesting research problems that I can cut my teeth on. His [PhD thesis](http://web.mit.edu/~axch/www/phd-thesis.pdf) discusses "propagation networks", which he argues is a more general substrate for computation than traditional methods. It's a long work, and it leaves open many questions, both theoretical and practical. I'm now tackling one very small angle with regards to the implementation of the system, but while we were still figuring a problem out, Alexy commented, "the more work I realize it takes to do a good job of giving someone a problem."

I wholeheartedly agree, though my experiences come from a different domain: [SIPB](http://sipb.mit.edu). Some of the key problems with assigning interested prospectives projects to work on include:

*   Many projects are extremely large and complex, and in many cases it's simply not possible to assign someone an interesting, high-level project and expect them to make significant headway. They're more likely to progress on a [wax on wax off](http://tvtropes.org/pmwiki/pmwiki.php/Main/WaxOnWaxOff) style training, but that's not interesting.
*   No one ever tells you what they're interested in! Even if you ask, you'll probably get the answer, "Eh, I'd be up for anything." As someone who has used this phrase before, I also emphatically understand that this is not true; people have different interests and will enjoy the same task dramatically differently.
*   It's easy to exert too much or too little control over the direction of the project. Too much control and you've defined the entire technical specification for the person, taken away their creative input, made them feel bad when they've not managed to get work done, and are bound to be dismayed when they failed to understand your exacting standards in the first place. Too little control and the person can easily get lost or waste hours fighting incidental issues and not the core of the problem.
*   Being a proper mentor is a time-consuming process, even if you exert minimal control. Once the person comes back with a set of patches, you still have to read them, make sure they're properly tested, and send back reviews on how the patches need to be reviewed (and for all but the most trivial of changes, this will be inevitable). You might wonder why you didn't just do the damn task yourself. Reframing the problem as a purely educational exercise can also be disappointing, if not done properly.
*   As people refine the art of bootstrapping, the number of possible projects they can work on explode, and what makes you think that they're going to work on *your* project? People decide what they want to work on, whether it's because they made it themselves, or it's in a field they're interested in, or it's a tool they use day-by-day, and if you don't get the person to buy in, you can easily loose them.

I imagine similar tensions come up for open-source project maintainers, internship programs and Google Summer of Code organizers. And I still have no feeling for what strategies actually *work* in this space, even though I've certainly been on both sides of the fence. I'd love to hear from people who have tried interesting strategies and had them work!