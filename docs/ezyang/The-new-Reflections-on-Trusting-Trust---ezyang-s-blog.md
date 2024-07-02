<!--yml

category: 未分类

date: 2024-07-01 18:17:39

-->

# 新的《反思信任的信任》：ezyang 的博客

> 来源：[`blog.ezyang.com/2011/10/the-new-reflections-on-trusting-trust/`](http://blog.ezyang.com/2011/10/the-new-reflections-on-trusting-trust/)

在他的经典文章[反思信任的信任](http://cm.bell-labs.com/who/ken/trust.html)中，Ken Thompson 描述了一种通过源代码检查无法检测到的自复制编译器错误。这种自复制是由于大多数编译器是自编译的：旧版本的编译器用于编译新版本，如果旧版本是恶意的，则在检测到它正在编译自身时它可以滑入相同的错误。

一个新的趋势恰恰是这种自托管的过程，但对于[自我认证的类型检查器](http://research.microsoft.com/en-us/projects/fstar/)：用于证明其自身正确性的类型检查器。（请注意，这些是强大的类型检查器，几乎能够检查关于代码的任意定理。）这可能看起来有点奇怪，因为我可以编写一个微不足道的类型检查器，它总是声称自己是正确的。为了解决这个问题，我们必须通过在另一种语言中（在 F*的情况下，这种语言是 Coq）证明类型检查器的正确性来启动正确性证明的引导过程。一旦完成了这个过程，我们就可以使用这个经过验证的类型检查器来检查它自身的规范。这个过程如下图所示。

那么问题是是否类似的自我认证的类型检查器对于 Ken 所描述的自托管编译器问题同样容易受到攻击。从论据上来说，让我们假设后端编译器和运行时是验证过的（这是一个几乎普遍不成立的强假设，包括对于 F*也是如此）。由于类型检查器无法在它编译的程序中插入恶意的 bug（它只是，你知道，类型检查），一个人必须依赖于源代码本身的 bug。当然，这样的 bug 肯定是显而易见的！

这是不清楚的：我们已经验证了我们的实现，但是我们的规范呢？在 Coq 中，我们证明了关于我们类型系统的声音性和适当性的各种定理，这至少让我们有些希望它在我们期望的方式上是正确的。但是这些证明在解放的 F* 世界中无处可见。如果我们想要发展我们的规范（对于一个全面依赖类型的语言来说可能性较小，但对于一个不那么强大的语言来说可能性在可能的范围内），我们必须回到 Coq 并调整相关的定理。否则，我们面临将我们的类型系统改变为不完备的风险。

幸运的是，这就是我们所要做的：我们可以使用旧的 F*类型检查器来验证新的类型检查器，而不是试图导出证书并用它们重新验证 Coq。总的来说，不要把你的 Coq 代码扔掉... 至少，如果你认为你的类型系统可能在未来发生变化的话。