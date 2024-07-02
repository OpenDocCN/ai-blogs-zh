<!--yml

category: 未分类

date: 2024-07-01 18:17:11

-->

# 番茄是蔬菜的一个子类型：ezyang 的博客

> 来源：[`blog.ezyang.com/2014/11/tomatoes-are-a-subtype-of-vegetables/`](http://blog.ezyang.com/2014/11/tomatoes-are-a-subtype-of-vegetables/)

子类型是一个在你学习它时似乎很有道理的概念（“当然，敞篷车是车的一个子类型，因为所有敞篷车都是车，但并非所有车都是敞篷车”），但一旦涉及到函数类型时，情况很快变得令人困惑。例如，如果`a`是`b`的一个子类型，那么`(a -> r) -> r`是`(b -> r) -> r`的一个子类型吗？（如果你知道这个问题的答案，那这篇博客不适合你！）当我们问我们的学生这个问题时，总有一些人被引入歧途。确实，你可以通过规则来机械地解决这个问题，但直觉是什么？

或许这个例子能帮到你。让`a`代表番茄，`b`代表蔬菜。如果我们可以在期望蔬菜的任何上下文中使用番茄，那么番茄就是蔬菜的一个子类型：因为番茄（在烹饪上）是蔬菜，番茄就是蔬菜的一个子类型。

那么`a -> r`怎么样呢？让`r`表示汤：那么我们可以把`番茄 -> 汤`看作是番茄汤的食谱（拿番茄做汤）和`蔬菜 -> 汤`看作是蔬菜汤的食谱（拿任何蔬菜做汤）。作为一个简化的假设，让我们假设我们关心的只是结果是汤，而不关心是什么类型的汤。

这两种类型的食谱之间的子类型关系是什么？蔬菜汤食谱更加灵活：你可以把它当作用来制作番茄汤的食谱，因为番茄只是蔬菜的一种。但是你不能用番茄汤的食谱来做茄子汤。因此，蔬菜汤食谱是番茄汤食谱的一个子类型。

这将引导我们进入最后一种类型：`(a -> r) -> r`。什么是`(蔬菜 -> 汤) -> 汤`？嗯，想象一下以下情景...

* * *

有一天晚上，鲍勃打电话给你。他说：“嘿，我冰箱里还有些蔬菜，我知道你爸爸在发明食谱方面是个天才。你知道他有没有一个好的汤食谱吗？”

“我不知道……”你慢慢地说，“什么样的蔬菜？”

“哦，那只是些蔬菜。听着，我会用一些汤来还你的，拿着食谱过来吧！”听筒里传来一声咔哒。

你翻阅爸爸的烹饪书，找到了一份番茄汤的食谱。哎呀！你不能带这个食谱过去，因为鲍勃可能并没有番茄。就在此时，电话再次响起。爱丽丝在电话那头：“牛肉炖菜的食谱很棒；我有些番茄，打算做些番茄汤，你有那种食谱吗？”显然，这种情况经常发生在你身上。

“事实上我知道！”你转回到你的烹饪书，但令你惊讶的是，你再也找不到你的番茄汤食谱了。但是你找到了一个蔬菜汤的食谱。“蔬菜汤食谱行得通吗？”

“当然 — 我不是植物学家：对我来说，番茄也是蔬菜。非常感谢！”

你也感到宽慰，因为现在你也为 Bob 有了一个食谱。

* * *

Bob 是一个将蔬菜汤食谱变成汤的人：他是`(Vegetable -> Soup) -> Soup`。另一方面，Alice 是一个将番茄汤食谱变成汤的人：她是`(Tomato -> Soup) -> Soup`。你可以给 Alice 番茄汤食谱或者蔬菜汤食谱，因为你知道她有番茄，但是 Bob 对手头食材的模糊描述意味着你只能带一个适合所有蔬菜的食谱。像 Alice 这样的调用者更容易适应：`(Tomato -> Soup) -> Soup`是`(Vegetable -> Soup) -> Soup`的一个子类型。

实际上，正式推理出子类型关系可能比直觉推断更快；然而，希望这种情况已经解释了*为什么*规则看起来像这样。