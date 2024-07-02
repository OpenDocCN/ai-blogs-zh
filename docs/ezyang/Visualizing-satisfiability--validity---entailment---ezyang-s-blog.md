<!--yml

category: 未分类

date: 2024-07-01 18:17:26

-->

# 可视化可满足性、有效性和蕴涵性：ezyang 的博客

> 来源：[`blog.ezyang.com/2012/10/visualizing-satisfiability-validity-and-entailment/`](http://blog.ezyang.com/2012/10/visualizing-satisfiability-validity-and-entailment/)

你正在半枯燥地处理命题逻辑问题集（毕竟，作为一名计算机科学家，你知道 AND 和 OR 是什么），突然问题集给出一个真正难解的问题：

> 是否真的有Γ ⊢ A 意味着Γ ⊢ ¬A 是假的？

然后你想，“双重否定，没问题！”并说，“当然！”当然，这是错误的：在你交卷后，你会想，“哎呀，如果Γ包含矛盾，那么我可以证明 A 和¬A。”然后你会想，“嘿，该死，我对这个东西一点直觉都没有。”

实际上，你可能已经对这类问题有了很好的直觉，只是你还不知道。

我们要做的第一件事是为命题逻辑句子建立一个视觉语言。当我们讨论命题句子如 A ∨ B 时，有一些需要赋值的命题变量，例如 A 为真，B 为假。我们可以将这些赋值看作是形成大小为`2^n`的集合，其中`n`是正在考虑的命题变量的数量。如果`n`很小，我们可以简单地画一个 Venn 图，但由于`n`可能相当大，我们将其可视化为一个圆形：

我们感兴趣的是分配的子集。有很多方法来定义这些子集；例如，我们可以考虑将 A 分配为真的分配集。但我们将对一种特定类型的子集感兴趣：特别是，使某个命题句子为真的分配子集。例如，“A ∨ B”对应于集合`{A=true B=true, A=true B=false, A=false B=true}`。我们将像这样图形化地绘制一个子集：

逻辑连接词直接对应于集合操作：特别是，合取（AND ∧）对应于集合交（∩），析取（OR ∨）对应于集合并（∪）。注意对应的运算符看起来非常相似：这不是偶然的！（当我首次学习我的逻辑运算符时，就是这样使它们清晰明了的：U 代表并集，从而一切就水到渠成。）

现在我们可以开始进入问题的核心了：比如“不可满足性”、“可满足性”和“有效性”（或者说是重言式）这样的陈述，实际上只是关于这些子集形状的陈述。我们可以通过视觉表达每一个：它们分别对应于空集、非空集和完整集：

这一切听起来很好，但我们还没有讨论“⊢”（即逻辑蕴涵）如何融入其中。实际上，当我说“B ∨ ¬B 是有效的”时，我实际上是在说“⊢ B ∨ ¬B 是真实的”；也就是说，无论我被允许使用什么假设，我总是能证明“B ∨ ¬B”。

所以大问题是：当我添加一些假设时会发生什么？如果我们考虑这里正在发生的事情，当我添加一个假设时，在某种意义上我使自己的生活变得“更容易”：我添加的假设越多，更多的命题句就是真实的。反过来说，我添加的假设越多，我需要担心的分配空间就越小：

Γ ⊢ φ为真所需的一切是Γ中的所有分配引起φ为真，即Γ必须包含在φ中。

太好了！所以让我们再次看看这个问题：

> Γ ⊢ A 是否意味着Γ ⊢ ¬A 为假？

重新表述为一个集合论问题，即：

> 对于所有的Γ和 A，Γ ⊂ A 是否意味着Γ ⊄ A^c（集合的补集）是真的？

我们考虑了一会儿，意识到：“不！因为空集是所有集合的子集是真的！”当然，空集恰好是一个矛盾：在所有事情的子集中（ex falso），而且仅仅是它自己的超集（只有矛盾暗示矛盾）。

* * *

结果证明，Γ也是一个集合，并且人们可能会想问Γ上的集合运算是否与我们的集合论模型中的集合运算有任何关系。这是非常诱人的，因为合并Γ似乎非常有效：`Γ ∪ Δ`似乎给我们Γ和Δ的合取（如果我们通过 AND 操作它们的所有元素来解释集合）。但最终，给出的最佳答案是“不”。特别是，Γ上的集合交是不连贯的：`{A} ∩ {A ∧ A}`应该是什么？一个严格的语法比较会说`{}`，即使明显`A ∧ A = A`。真正正确的做法是进行一个析取，但这要求我们说`{A} ∩ {B} = {A ∨ B}`，这是令人困惑的，最好放在一边不予理会。