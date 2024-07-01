<!--yml
category: 未分类
date: 2024-07-01 18:17:21
-->

# (Homotopy) Type Theory: Chapter One : ezyang’s blog

> 来源：[http://blog.ezyang.com/2013/06/homotopy-type-theory-chapter-one/](http://blog.ezyang.com/2013/06/homotopy-type-theory-chapter-one/)

In what is old news by now, the folks at the Institute for Advanced Study have released [Homotopy Type Theory: Univalent Foundations of Mathematics](http://homotopytypetheory.org/book/). There has been some (meta)commentary ([Dan Piponi](https://plus.google.com/107913314994758123748/posts/VzWAsojiifE), [Bob Harper](http://existentialtype.wordpress.com/2013/06/22/whats-the-big-deal-with-hott/), [Andrej Bauer](http://math.andrej.com/2013/06/20/the-hott-book/), [François G. Dorais](http://dorais.org/archives/1425), [Steve Awodey](http://homotopytypetheory.org/2013/06/20/the-hott-book/), [Carlo Angiuli](http://www.carloangiuli.com/blog/homotopy-type-theory-univalent-foundations-of-mathematics/), [Mike Shulman](http://golem.ph.utexas.edu/category/2013/06/the_hott_book.html), [John Baez](https://plus.google.com/117663015413546257905/posts/cm1sKge8qxX)) on the Internet, though, of course, it takes time to read a math textbook, so don’t expect detailed technical commentary from non-authors for a while.

Of course, being a puny grad student, I was, of course, most interested in the book’s contribution of *yet another Martin-Löf intuitionistic type theory introduction*, e.g. chapter one. The classic introduction is, of course, the papers that Martin Löf wrote (nota bene: there were many iterations of this paper, so it’s a little hard to find the right one, though it seems Giovanni Sambin’s notes are the easiest to find), but an introduction of type theory for *homotopy type theory* has to make certain adjustments, and this makes for some novel presentation. In particular, the chapter’s discussion of *identity types* is considerably more detailed than I have seen elsewhere (this is not surprising, since identity is of central importance to homotopy type theory). There is also a considerable bit of pedantry/structure in the discussion of the types that make up the theory, reminiscent of the [PFPL](http://existentialtype.wordpress.com/2012/12/03/pfpl-is-out/) (though I believe that this particular chapter was mostly written by others). And, of course, there are many little variations in how the theory is actually put together, expounded upon in some detail in the chapter notes.

In more detail:

**Definitional and propositional equality.** The chapter spends a little bit of time carefully distinguishing between definitional equality (a purely syntactic notion up to computation) and propositional equality (which involves evidence), which I appreciated. The difference between connectives which show up inside and outside the deductive system was a major point of confusion for me when I was originally learning logic.

**The general pattern of the introduction of a new kind of type.** The modern style for introducing logical connectives is to classify the rules into various kinds, such as introduction rules and elimination rules, and then hew to this regularity in the presentation. Often, readers are expected to “see it”, but this book makes a helpful remark laying out the style. I found a useful exercise was to take the rules and reorganize them so that, for example, all of the elimination rules are together and compare them.

**Recursion and induction.** [I’ve written about this subject before](http://blog.ezyang.com/2013/04/the-difference-between-recursion-induction/), arguing that recursion and induction aren’t the same thing, since induction needs to work over indexed types. This is true, but there is an important point I did not make: *induction is generalized recursion*. This is because when you specify your type family *P* to be the *constant type family* which ignores its index, the dependence is erased and you have an ordinary recursor. In fact, this is a [CPDT exercise](http://adam.chlipala.net/cpdt/html/InductiveTypes.html); I think it clarifies things to see this in both Coq and informal mathematics, as the informal presentation makes the dimension of generalization clearer.

**Identity types.** I won’t lie: I had a difficult time with this section, and I don’t think I fully understand why path induction works, even after a very long remark at the end of the section. (Additionally, while the notes point to some prior literature about the subject, I took a look at the papers and I did not see anything that resembled their presentation of path induction.) By default, Coq thinks the inductive principle for equality types should be what is referred to in this book as the indiscernability of identicals:

```
> Check eq_rect.
eq_rect
     : forall (A : Type) (x : A) (P : A -> Type),
       P x -> forall y : A, x = y -> P y

```

(As a tangent, the use of family *C* is confusingly overloaded; when discussing the generalization of the previous principlem the reader is required to imagine `C(x) -> C(y)  ===  C(x, y)`—the C’s of course being distinct.) Path induction asks for more:

```
eq_ind
     : forall (A : Type), forall (C : forall (x y : A), x = y -> Type),
       (forall (x : A), C x x (eq_refl x)) -> forall (x y : A), forall (p : x = y), C x y p

```

This is perhaps not too surprising, since this machinery is principally motivated by homotopy type theory. Additionally, the inductive principle follows the same pattern as the other inductive principles defined for the other types. The trouble is a frustrating discussion of why this inductive principle valid, even when you might expect, in a HoTT setting, that not all equality was proven using reflexivity. My understanding of the matter is that is has to do with the placement of the `forall (x : A)` quantifier. It is permissible to move one of the x's to the top level (based path induction), but not *both*. (This is somewhat obscured by the reuse of variable names.) There is also a geometric intuition, which is that when both or one endpoints of the path are free (inner-quantification), then I can contract the path into nothingness. But I have a difficult time mapping this onto any sort of rigorous argument. Perhaps you can help me out.

> As an aside, I have some general remarks about learning type theory from a functional programming background. I have noticed that it is not too hard to use Coq without knowing much type theory, and even easier to miss the point of why the type theory might be helpful. But in the end, it is really useful to understand what is going on, and so it’s well worth studying *why* dependent products and sums generalize the way they do. It also seems that people find the pi and sigma notation confusing: it helps if you realize that they are algebraic puns. Don’t skip the definition of the inductive principles.

I apologize if any of this post has been inaccurate or misleadingly skewed. My overall impression is that this first chapter is a very crisp introduction to type theory, but that the segments on identity types may be a little difficult to understand. Now, onwards to chapter two!