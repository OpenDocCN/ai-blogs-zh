<!--yml
category: 未分类
date: 2024-07-01 18:17:21
-->

# Category theory for loop optimizations : ezyang’s blog

> 来源：[http://blog.ezyang.com/2013/05/category-theory-for-loop-optimizations/](http://blog.ezyang.com/2013/05/category-theory-for-loop-optimizations/)

Christopher de Sa and I have been working on a category theoretic approach to optimizing MapReduce-like pipelines. Actually, we didn’t start with any category theory—we were originally trying to impose some structure on some of the existing loop optimizations that the [Delite compiler](http://stanford-ppl.github.io/Delite/) performed, and along the way, we rediscovered the rich relationship between category theory and loop optimization.

On the one hand, I think the approach is pretty cool; but on the other hand, there’s a lot of prior work in the area, and it’s tough to figure out where one stands on the research landscape. As John Mitchell remarked to me when I was discussing the idea with him, “Loop optimization, can’t you just solve it using a table lookup?” We draw a lot of inspiration from existing work, especially the *program calculation* literature pioneered by Bird, Meertens, Malcom, Meijer and others in the early 90s. The purpose of this blog post is to air out some of the ideas we’ve worked out and get some feedback from you, gentle reader.

There are a few ways to think about what we are trying to do:

*   We would like to *implement* a calculational-based optimizer, targeting a real project (Delite) where the application of loop optimizations can have drastic impacts on the performance of a task (other systems which have had similar goals include [Yicho](http://takeichi.ipl-lab.org/yicho/doc/Yicho.html), [HYLO](http://lecture.ecc.u-tokyo.ac.jp/~uonoue/hylo/)).
*   We want to venture where theorists do not normally tread. For example, there are many “boring” functors (e.g. arrays) which have important performance properties. While they may be isomorphic to an appropriately defined algebraic data type, we argue that in a calculational optimizer, we want to *distinguish* between these different representations. Similarly, many functions which are not natural transformations *per se* can be made to be natural transformations by way of partial application. For example, `filter p xs` is a natural transformation when `map p xs` is incorporated as part of the definition of the function (the resulting function can be applied on any list, not just the original `xs`). The resulting natural transformation is *ugly* but *useful*.
*   For stock optimizers (e.g. Haskell), some calculational optimizations can be supported by the use of *rewrite rules*. While rewrite rules are a very powerful mechanism, they can only describe “always on” optimizations; e.g. for deforestation, one always wants to eliminate as many intermediate data structures as possible. In many of the applications we want to optimize, the best performance can only be achieved by *adding* intermediate data structures: now we have a space of possible programs and rewrite rules are woefully inadequate for specifying *which* program is the best. What we’d like to do is use category theory to give an account for rewrite rules *with structure*, and use domain specific knowledge to pick the best programs.

I’d like to illustrate some of these ideas by way of an example. Here is some sample code, written in Delite, which calculates an iteration of (1-dimensional) k-means clustering:

```
(0 :: numClusters, *) { j =>
  val weightedPoints = sumRowsIf(0,m){i => c(i) == j}{i => x(i)};
  val points = c.count(_ == j);
  val d = if (points == 0) 1 else points
  weightedPoints / d
}

```

You can read it as follows: we are computing a result array containing the position of each cluster, and the outermost block is looping over the clusters by index variable `j`. To compute the position of a cluster, we have to get all of the points in `x` which were assigned to cluster `j` (that’s the `c(i) == j` condition) and sum them together, finally dividing by the sum by the number of points in the cluster to get the true location.

The big problem with this code is that it iterates over the entire dataset *numClusters* times, when we’d like to only ever do one iteration. The optimized version which does just that looks like this:

```
val allWP = hashreduce(0,m)(i => c(i), i => x(i), _ + _)
val allP = hashreduce(0,m)(i => c(i), i => 1, _ + _)
(0::numClusters, *) { j =>
    val weightedPoints = allWP(j);
    val points = allP(j);
    val d = if (points == 0) 1 else points
    return weightedpoints / d
}

```

That is to say, we have to precompute the weighted points and the point count (note the two hashreduces can and should be fused together) before generating the new coordinates for each of the clusters: generating *more* intermediate data structures is a win, in this case.

Let us now calculate our way to the optimized version of the program. First, however, we have to define some functors:

*   `D_i[X]` is an array of `X` of a size specified by `i` (concretely, we’ll use `D_i` for arrays of size `numPoints` and `D_j` for arrays of size `numClusters`). This family of functors is also known as the [diagonal functor](http://en.wikipedia.org/wiki/Diagonal_functor), generalized for arbitrary size products. We also will rely on the fact that `D` is [representable](http://stackoverflow.com/questions/12963733/writing-cojoin-or-cobind-for-n-dimensional-grid-type), that is to say `D_i[X] = Loc_D_i -> X` for some type `Loc_D_i` (in this case, it is the index set `{0 .. i}`.
*   `List[X]` is a standard list of `X`. It is the initial algebra for the functor `F[R] = 1 + X * R`. Any `D_i` can be embedded in `List`; we will do such conversions implicitly (note that the reverse is not true.)

There are a number of functions, which we will describe below:

*   `tabulate` witnesses one direction of the isomorphism between `Loc_D_i -> X` and `D_i[X]`, since `D_i` is representable. The other direction is `index`, which takes `D_i[X]` and a `Loc_D_i` and returns an `X`.
*   `fold` is the unique function determined by the initial algebra on `List`. Additionally, suppose that we have a function `*` which combines two algebras by taking their cartesian product,
*   `bucket` is a natural transformation which takes a `D_i[X]` and buckets it into `D_j[List[X]]` based on some function which assigns elements in `D_i` to slots in `D_j`. This is an example of a natural transformation that is not a natural transformation until it is partially applied: if we compute `D_i[Loc_D_j]`, then we can create a natural transformation that doesn’t ever look at `X`; it simply “knows” where each slot of `D_i` needs to go in the resulting structure.

Let us now rewrite the loop in more functional terms:

```
tabulate (\j ->
  let weightedPoints = fold plus . filter (\i -> c[i] == j) $ x
      points = fold inc . filter (\i -> c[i] == j) $ x
  in divide (weightedPoints, points)
)

```

(Where `divide` is just a function which divides its arguments but checks that the divisor is not zero.) Eliminating some common sub-expressions and fusing the two folds together, we get:

```
tabulate (\j -> divide . fold (plus * inc) . filter (\i -> c[i] == j) $ x)

```

At this point, it is still not at all clear that there are any rewrites we can carry out: the `filter` is causing problems for us. However, because filter is testing on equality, we can rewrite it in a different way:

```
tabulate (\j -> divide . fold (plus * inc) . index j . bucket c $ x)

```

What is happening here? Rather than directly filtering for just items in cluster `j`, we can instead view this as *bucketing* `x` on `c` and then indexing out the single bucket we care about. This shift in perspective is key to the whole optimization.

Now we can apply the fundamental rule of natural transformations. Let `phi = index j` and `f = divide . fold (plus * inc)`, then we can push `f` to the other side of `phi`:

```
tabulate (\j -> index j . fmap (divide . fold (plus * inc)) . bucket c $ x)

```

Now we can eliminate `tabulate` and `index`:

```
fmap (divide . fold (plus * inc)) . bucket c $ x

```

Finally, because we know how to efficiently implement `fmap (fold f) . bucket c` (as a `hashreduce`), we split up the `fmap` and join the fold and bucket:

```
fmap divide . hashreduce (plus * inc) c $ x

```

And we have achieved our fully optimized program.

All of this is research in progress, and there are lots of open questions which we have not resolved. Still, I hope this post has given you a flavor of the approach we are advocating. I am quite curious in your comments, from “That’s cool!” to “This was all done 20 years ago by X system.” Have at it!