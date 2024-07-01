<!--yml
category: 未分类
date: 2024-07-01 18:17:37
-->

# You could have invented fractional cascading : ezyang’s blog

> 来源：[http://blog.ezyang.com/2012/03/you-could-have-invented-fractional-cascading/](http://blog.ezyang.com/2012/03/you-could-have-invented-fractional-cascading/)

Suppose that you have *k* sorted arrays, each of size *n*. You would like to search for single element in each of the *k* arrays (or its predecessor, if it doesn't exist).

Obviously you can binary search each array individually, resulting in a ![O(k\lg n)](img/2eb720762ef6ea22926b0070a6df6d6d.png "O(k\lg n)") runtime. But we might think we can do better that: after all, we're doing the same search *k* times, and maybe we can "reuse" the results of the first search for later searches.

Here's another obvious thing we can do: for every element in the first array, let's give it a pointer to the element with the same value in the second array (or if the value doesn't exist, the predecessor.) Then once we've found the item in the first array, we can just follow these pointers down in order to figure out where the item is in all the other arrays.

But there's a problem: sometimes, these pointers won't help us at all. In particular, if a later lists is completely "in between" two elements of the first list, we have to redo the entire search, since the pointer gave us no information that we didn't already know.

So what do we do? Consider the case where *k = 2*; everything would be better if only we could guarantee that the first list contained the right elements to give you useful information about the second array. We could just merge the arrays, but if we did this in the general case we'd end up with a totally merged array of size ![kn](img/b2931d4dac72237e5ab420c094722a83.png "kn"), which is not so good if *k* is large.

But we don't need all of the elements of the second array; every other item will do!

Let's repeatedly do this. Take the last array, take every other element and merge it into the second to last array. Now, with the new second to last array, do this to the next array. Rinse and repeat. How big does the first array end up being? You can solve the recurrence: ![T(k) = n + T(k-1)/2](img/ae7430846af6a2f79ef9b1e4192acce5.png "T(k) = n + T(k-1)/2"), which is the geometric series ![n + n/2 + n/4 + n/8 + \ldots = 2n](img/550ef24dffb758e390d735b13c34e555.png "n + n/2 + n/4 + n/8 + \ldots = 2n"). Amazingly, the new first list is only twice as large, which is only one extra step in the binary search!

What we have just implemented is **fractional cascading**! A fraction of any array cascades up the rest of the arrays.

There is one more detail which has to be attended to. When I follow a pointer down, I might end up on an element which is not actually a member of the current array (it was one that was cascaded up). I need to be able to efficiently find the next element which is a member of the current array (and there might be many cascaded elements jammed between it and the next member element, so doing a left-scan could take a long time); so for every cascaded element I store a pointer to the predecessor member element.

Fractional cascading is a very useful transformation, used in a variety of contexts including *layered range trees* and *3D orthogonal range searching*. In fact, it can be generalized in several ways. The first is that we can cascade some fixed fraction α of elements, rather than the 1/2 we did here. Additionally, we don't have to limit ourselves to cascading up a list of arrays; we can cascade up an arbitrary graph, merging many lists together as long as we pick α to be less than *1/d*, where *d* is the in-degree of the node.

*Exercise.* Previously, we described [range trees](http://blog.ezyang.com/2012/02/visualizing-range-trees/). How can fractional cascading be used to reduce the query complexity by a factor of ![O(\lg n)](img/0fd0ce085d9f23267b01ffb501477a39.png "O(\lg n)")?

*Exercise.* There is actually another way we can setup the pointers in a fractionally cascaded data structure. Rather than have downward pointers for every element, we only maintain pointers between elements which are identical (that is to say, they were cascaded up.) This turns out to be more convenient when you are constructing the data structure. However, you now need to maintain another set of pointers. What are they? (Hint: Consider the case where a search lands on a non-cascaded, member element.)