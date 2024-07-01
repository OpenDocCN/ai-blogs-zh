<!--yml
category: 未分类
date: 2024-07-01 18:16:59
-->

# A Machine Learning Approach to Database Indexes (Alex Beutel) : ezyang’s blog

> 来源：[http://blog.ezyang.com/2017/12/a-machine-learning-approach-to-database-indexes-alex-beutel/](http://blog.ezyang.com/2017/12/a-machine-learning-approach-to-database-indexes-alex-beutel/)

The below is a transcript of a talk by [Alex Beutel](http://alexbeutel.com/) on [machine learning database indexes](https://arxiv.org/abs/1712.01208), at the [ML Systems Workshop](https://nips.cc/Conferences/2017/Schedule?showEvent=8774) at NIPS'17.

* * *

DB researchers think about there research differently. You have a system that needs to work for all cases. Where as in ML, we have a unique circumstance, I'll build a model that works well. In DB, you have to fit all.

To give an example of this is a B-tree. A B-tree works for range queries. We have records, key, we want to find all records for range of keys. 0-1000, you build tree on top of sorted array. To quickly look up starting point in range. What if all my data, all of the keys, from zero to million... it becomes clear, you don't need the whole tree above. You can use the key itself as an offset into the array. Your lookup is O(1), O(1) memory, no need for extra data structure.

Now, we can't go for each app, we can't make a custom implementation to make use of some pattern. DB scale to any application, we don't want to rebuild it any time.

But ML excels in this situation. It works well for a wide variety of distributions, learn and make use of them effectively.

This is the key insight we came to. Traditional data structures make no assumptions about your data. They work under any distribution, and generally scale O(n). Interestingly, learning, these data distributions, can offer a huge win. What we're trying to go to, is instead of scaling to size of data, we scale to complexity of it. With linear data, it's O(1). For other distributions, can we leverage this?

There are three dat structures underlying databases. There are B-Trees; range queries, similarity search. Main index. Hash maps for point lookups; individual records. This is more common throughout CS. And bloom filters, are really common for set-inclusion queries. Do I have a key. If your record is stored on disk, checking first if there's a record with that key is worthwhile. We're going to focus entirely on B-trees.

B-trees take a tree like structure with high branching factor. What makes it really effective is that it's cache efficient. You can store top level nodes in your cache where it's fast to look it up, maybe others in main memory, and the actual memory on disk. By caching the hierarchy appropriately, it makes it efficiently. At a high level, a B-tree maps a key to a page, some given place in memory. Once it finds that page, it will do some local search to find the particular range of that key. That could be a scan or binary search; we know the range will be the position from start of page to page size.

An abstract level, the Btree is just a model. It's taking the position of the key, and trying to estimate the position. What we have in this case, we want to search in this error range to find the ultimate record. At a high level, it would mean that we can't use any model. We need err_min and err_max. But we have all the data. If you have all the data, you know at index construction time, you know all the data you're executing against, and you can calculate what the model's min and max error is.

One interesting thing is this is just a regression problem. What you're really modeling is just the CDF. On the X axis on this plot here, the X axis is your keys, Ys your position. This is modeling where your probability mass is located; where your data is in the keyspace. CDFs are studied somewhat, but not a ton, in the literature. This is a nice new implication of research.

We thought, OK, let's try this out straightaway. Train a model, see how fast it is. We looked at 200M server logs, timestamp key, 2 layer NN, 32-width, relatively small by ML. We train to predict position, square error. A B-Tree executes in 300ns. Unfortunately, with the model, it takes 80000ns. By most ML model speeds, this is great. If you're looking at executing on server, great. But this doesn't work for a database.

There are a bunch of problems baked into this. TF is really designed for large models. Think about translation or superresolution images; these are hefty tasks. We need to make this fast for database level speed. Second, b-trees are great for overfitting. There's no risk of over-fitting in this context. They're also cache efficient; that's not looked at in ML. The last thing is local search in the end. Is that really the most effective way of ultimately finding that key? I'm skipping that part because it's fairly detailed, I'll focus on first three.

The first part is just the raw speed fo execution of ML model. This was built really by Tim, this Learning Index Framework program. What it does is it lets you create different indexes under different configurations. For one thing, it lets you do code compilation for TF, ideas from Tupleware, where you can take a linear model and execute it extremely quickly. We can also train simple models. Use TF for more complex gradient descent based learning; extract weights, and have inference graph be codegenned. And we can do a lot of autotuning, to find what the best model architecture is. We know ahead of time what the best training is. We can make pretty smart decisions about what works best.

The next problem is accuracy and sepeed. If I have 100M records, I narrow down quickly from 1.5M to 24K, with each step down this tree. Each one of those steps is 50-60 cycles to look through that page, and to find what the right branch is. So we have to get to an accurracy of 12000, within 500 mul/add, to beat these levels of hierarchy, which are in cache. This is a steep task. The question is what is the right model? a really wide network? Single hidden layer? This scales nicely, we can fit in 256 layer reasonably. We could go deeper... the challenge is we have width^2, which need to be parallelized somehow. The challenge is, how do we effectively scale this. We want to add capacity to the model, make it more and more accurate, with increased size, without becoming to.

We took a different approach, based on mixed experts. We'll have a key, have a really simple classifier. We get an estimate. Then we can use that estimate to find it at the next stage. Narrow down the CDF range, and try to be more accurate in the subset of space. It will still get key as input; given key, give position, but more narrow space of keys. We build this down, and we'll walk down this hierarchy. This decouples model size and complexity. We have a huge model, overfitting, but we don't have to execute all of the sparsity that you would have to do from a pure ML view. We can decouple it usefully. The nice thing we can do is fall back to B-trees for subsets that are difficult to learn in a model. The LIF framework lets us substitute it in easily. In the worst case, B-tree. Best case, more efficient.

The quick results version here, is we find we have four different data sets. Most are integer data sets; last one is string data set. We're trying to save memory and speed; we save memory hugely; these are really simple models. Linear with simple layer, with possibly two stages. We're able to get a significant speedup in these cases. Server logs one is interesting. It looks at a high level very linear, but there's actually daily patterns to this data accessed. Maps is more linear; it's longitudes of spaces. We created synthetic data that's log normal, and here we see we can model it effectively. Strings is an interesting challenge going forward; your data is larger and more complicated, building models that are efficient over a really long string is different; the overall patterns are harder to have intuition about. One thing really worth noting here, it's not using GPUs or TPUs; it's pureely CPU comparison. Apples-to-apples.

This is mostly going into the B-tree part. This is a regression model looking at CDF of data. We can use these exact same models for hash maps. With bloom filters, you can use binary classifiers. I have a bunch of results in the poster in the back.

A few minutes to talk about rooms for improvement. There are a bunch of directions that we're excited to explore. Obvious one is GPUs/TPUs. It's cPUs because that's when B-trees are most effective; but scaling is all about ML. Improving throughput and latency for models with GPUs, exciting going forward. Modeling themselves; there's no reason to believe hierarchy of models is the right or best choice; it's interesting to build model structures that match your hardware. Memory efficient, underlying architecture of GPUs. In the scale of ns we need for database. Multidimensional indexes; ML excels in high numbers of dimension; most things are not looking at a single integer feature. There's interesting question about how you map to multidimensional indexes that are difficult to scale. If we have a CDF, you can approximately sort it right there. And inserts and updates, assumed read-only databases. Large class of systems, but we get more data. How do we balance overfitting with accuracy; can we add some extra auxiliary data structures to balance this out?

Q: One thing is that when... this problem, we solved pretty well without ML. When we introduce ML, we should introduce new metrics. We shouldn't make our system more fragile, because distribution changes. What would be the worst case when distribution changes?

A: As the data becomes updated... in the case of inference and updates, there's a question about generalization. I think you could look at it from the ML point of view: statistically, test model today on tomorrows inserts. (It's a method. If I use this method, and then train it with data that I don't yet have... and do.) The typical extrapolation to future generalization of ML. Guarantees are hard. There will be a worst case that is awful... but the flip side, that's the ML side... generalization. There's also a point of view, I couple this with classic data structure. we coupled modeling with classic data structures: search, bloom filter case, so you don't actually have this work. You catch worst case.

Let me add to that. If you assume that the inserts follow the same distribution as trained model, then the inserts become all one operation. They're even better. Suppose they don't follow the same distribution? you can still do delta indexing. Most systems do do delta indexing. So inserts are not a big problem.

Q: (Robert) Most of the inputs were one or two real numbers, and outputs are a single real number. how does it work if you use a low degree polynomial, or a piecewise linear classifier on the different digits?

A: In the case of strings, it's not a single input. (Treat it as integer?) Well, it's possibly a thousand characters long. It's not the best representation. Different representations work really well. The last thing I want to say, piecewise linear could work, but when you run 10k, 100k submodels, it's slow. Hierarchy helps. Polynomials are interesting, depends on data source.

Q: Can you comment how bad your worst case is? Average numbers?

A: We specifically always have a spillover. The worst case is defaulting to typical database. We haven't had a case where you do worse, because we'll default to B-tree. (Deterministic execution?) Not inference time.