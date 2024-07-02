<!--yml

category: 未分类

date: 2024-07-01 18:17:46

-->

# 如果有很多注释，那可能就有 bug：ezyang 的博客

> 来源：[`blog.ezyang.com/2011/05/byron-cook-sla/`](http://blog.ezyang.com/2011/05/byron-cook-sla/)

## 如果有很多注释，那么它可能有 bug

昨天，我们邀请到了特邀演讲嘉宾[拜伦·库克](http://research.microsoft.com/en-us/people/bycook/)，他来讲解关于[SLAM](http://research.microsoft.com/en-us/projects/slam/)的话题，这是一个将定理证明技术应用于设备驱动程序的很好的实际例子。

拜伦曾在设备驱动程序开发方面有过一些非常搞笑（和有趣）的评论。毕竟，当设备驱动程序崩溃时，责任不在设备驱动程序编写者身上，而是在微软身上。他指出，在硬件公司，“如果你不够聪明，你会被分配去写软件驱动程序。聪明人会去做硬件工作”，以及当你阅读设备驱动程序代码时，“如果有很多注释而且拼写错误，那可能就有 bug。” 尖锐！我们一直习惯于赞扬注释代码的好处，但毫无疑问，编写注释可以帮助澄清对自己而言令人困惑的代码，而如果代码一开始就不那么令人困惑，你就不会感到有必要写注释了。因此，有时候是过去的某位大师写了非常聪明的代码，然后你来到这里，却不够聪明完全理解当时的情况，因此在修改时写了很多注释来解释代码。嗯，这不是注释的错，但事实上代码对你来说太聪明了，可能意味着在修改时引入了 bug。

SLAM 用于处理指数级状态空间爆炸的方法也非常有趣。他们的做法是尽可能地丢弃多余的状态（而不是消除错误），然后查看简化后的程序是否触发错误。通常情况下会触发，尽管由于虚假的转换，所以他们会引入足够的额外状态来消除这个虚假路径，然后重复这个过程，直到简化后的程序被认为满足断言（成功），或者我们在简化后的程序中发现一个在真实程序中不是虚假的路径。另一个非常有趣的地方是，他们选择的规约语言本质上是增强的断言。在像时态逻辑这样的学术课程中，你会花费大部分时间研究诸如 CTL 和 LTL 之类的逻辑，这些对于设备驱动程序编写者来说很陌生和奇怪；断言则更容易让人们开始。我确实可以看到这个方法也适用于形式验证的其他领域（基于断言的类型注释，任何人？）

*附言.* 我有一些绝对巨大的帖子即将推出，但在复习考试和临时复习会议之间，我还没有说服自己在考试前完成这些帖子是一个好的时间利用。但它们最终会来！很快！希望！