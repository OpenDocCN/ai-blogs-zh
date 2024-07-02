<!--yml

category: 未分类

date: 2024-07-01 18:17:24

-->

# 为什么我不能有点懒呢？：ezyang 的博客

> 来源：[`blog.ezyang.com/2012/11/why-cant-i-just-be-a-little-lazy/`](http://blog.ezyang.com/2012/11/why-cant-i-just-be-a-little-lazy/)

你可以。想象一下，有一个版本的 Haskell，其中每个构造器都是严格的，例如每个字段都有一个 `!` 前缀。这种语言的语义是明确定义的；事实上，CMU 的好同志们早就知道这一点：

> 到目前为止，我们经常在各种语言构造的动态中遇到任意选择。例如，在指定对偶的动态时，我们必须选择一个相当随意的方式，是全懒惰的动态，即所有对偶都是值，而不管其组成部分的值状态，还是急迫的动态，即只有其组成部分都是值时，对偶才是值。我们甚至可以考虑半急迫（或等效地，半懒惰）的动态，即一个对偶只有在第一个组成部分是值的情况下才是值，而不考虑第二个组成部分。
> 
> 关于和求和（所有的注射是值，或者只有值的注射是值），递归类型（所有的折叠是值，或者只有值的折叠是值），以及函数类型（函数应该按名字调用还是按值调用）等类似的问题也会出现。整个语言围绕着坚持某一政策或另一政策而建立。例如，Haskell 规定产品、求和和递归类型是懒惰的，并且函数按名字调用，而 ML 则规定完全相反的政策。这些选择不仅是随意的，而且也不清楚为什么它们应该被联系起来。例如，我们可以非常合理地规定产品、求和和递归类型是懒惰的，但在函数上实施按值调用的纪律。或者**我们可以急迫地使用产品、求和和递归类型，但坚持按名字调用。**这些选择在空间的哪一个点是正确的一点都不清楚；每一个都有其拥护者，也都有其反对者。
> 
> 因此，我们是否陷入了主观性的困境中？不！走出这一困境的方法是意识到这些差异不应该由语言设计者来强加，而是应该由程序员来做出选择。这可以通过意识到动态的差异反映了正在被语言所模糊的基本类型区别来实现。我们可以在同一语言中同时拥有急迫和懒惰的对偶，同样地，我们也可以在同一语言中同时拥有急迫和懒惰的求和，以及按名字和按值的函数空间，通过提供足够的类型区别使得这些选择对程序员可用。