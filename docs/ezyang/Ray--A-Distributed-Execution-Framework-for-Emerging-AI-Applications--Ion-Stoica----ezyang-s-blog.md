<!--yml

category: 未分类

date: 2024-07-01 18:16:59

-->

# Ray：一个用于新兴 AI 应用的分布式执行框架（Ion Stoica）：ezyang's blog

> 来源：[`blog.ezyang.com/2017/12/ray-a-distributed-execution-framework-for-emerging-ai-applications-ion-stoica/`](http://blog.ezyang.com/2017/12/ray-a-distributed-execution-framework-for-emerging-ai-applications-ion-stoica/)

下面是[Ion Stoica](https://people.eecs.berkeley.edu/~istoica/)在[NIPS'17 的 ML 系统研讨会](https://nips.cc/Conferences/2017/Schedule?showEvent=8774)上关于[Ray](https://github.com/ray-project/ray)的讲话的记录。

* * *

我们在伯克利已经研究了一年多了。在过去的几年里，AI 取得了巨大的进步。广告定向、图像和语音等领域都有显著的发展。许多应用都基于使用深度神经网络的监督学习。监督学习和无监督学习是两种主要的方法。

然而，下一代 AI 应用程序将会非常不同。它们部署在关键任务场景中，需要不断地从快速变化的环境中学习。机器人技术、自动驾驶汽车、无人机、对话系统等。实现这一新一代 AI 应用程序需要更广泛的技术应用。随机优化、并行模拟等等。

Ray 为实施这些方法提供了一个统一的平台。为了激励 Ray，我将使用强化学习。RL 通过与环境交互进行学习。策略将状态/观察映射到行动，以最大化某种奖励。RL 的需求是什么？许多应用程序表现出嵌套并行性：搜索中使用数据并行 SGD，然后调用一个组件进行模拟策略评估，这在多个 CPU 上并行运行。其次，这些工作负载在硬件和时间上高度异构。许多计算不仅需要 CPU，还需要 GPU、TPU 和 FPGA。其次，这些计算可能需要非常不同的时间。模拟棋盘游戏：3 步输掉，或者 50 步赢或平局。在机器人技术中，我们需要实时处理，同时并行处理来自传感器的数据，处理时间在十几毫秒之内。

满足这些要求并不容易。为了达到这些要求，您需要一个灵活和高性能的系统。灵活性：它应该能够动态创建和调度任务，并支持任意的依赖关系。性能：它应该能够扩展到数百个节点，毫秒级延迟，数百万个任务，并能高效地共享数值数据。

接下来，我将说明我们如何应对这些挑战。灵活性？我们提供一个非常灵活的模型：动态任务图。在此基础上，我们提供两种模型：并行任务和 actors。

要讨论并行任务，这里是 Python 代码：一个从文件读取数组，另一个将两个数组相加。代码很简单：它从 file1 和 file2 创建了两个数组 a 和 b，并将它们相加。所以现在，很容易并行化这个程序。如果我们想要并行化一个函数，为了做到这一点，我们需要为每个函数添加一个 ray.remote 装饰器。当我们调用这些函数时，需要调用 remote 方法。Remote 不会返回对象本身，只返回对象标识符。这与 futures 抽象非常相似。要获取实际对象，必须对对象标识符调用 ray.get。

要更好地了解 Ray 如何执行，让我们执行一个简单的程序。假设文件存储在不同的节点上。当在 file1 上执行 read_array 时，它会安排在适当的节点上执行 read_array。远程调用会立即返回，而实际读取尚未完成。这允许驱动程序并行运行第二个任务，运行在 file2 上的节点，并启动 add remote 函数。所有函数都已远程调度，但还没有完成。要实际获取结果，你必须对结果调用 ray.get。这是一个阻塞调用，你将等待整个计算图被执行完毕。

Tasks 非常通用，但这还不够。考虑你想要运行一个模拟器，而这个模拟器是闭源的。在这种情况下，你无法访问状态。你有状态、动作、模拟，为了在模拟器中设置状态，你无法做到。所以为了解决这个问题，还有另一种用例，即状态创建成本过高的情况。例如，在 GPU 上的深度神经网络中，你希望初始化一次，并且为每次模拟重新初始化。

为了解决这些用例，我们添加了 Actor 抽象。一个 actor 只是一个远程类。如果你有一个计数器 Counter，你标记它为 ray.remote，然后在创建类或调用方法时使用 remote 关键字。这是一个非常简单的示例的计算图。注意方法调用也返回对象标识符。要获取结果，你需要对对象标识符调用 ray.get。Ray 还允许你为 actors 和 tasks 指定资源数量。

综合起来，为了提供一个更现实的例子，评估策略是 Salimans 等人在 OpenAI 中提出的一种可扩展的 RL 形式。简而言之，进化策略尝试许多策略，并尝试看哪一个运行效果最好。这是高度并行的。因此，这里是并行策略的伪代码。一个做模拟并返回奖励的工作者，创建二十个工作者，然后两百个，进行两百次模拟，更新策略。同样地，如果您想并行化此代码，我们必须添加一堆远程，并且现在在右侧，您会注意到我也在共享计算图。当您调用 Worker.remote 时，您会创建 20 个远程工作者来并行执行它。您使用远程关键字调用。再次注意，在这种情况下，结果不是奖励本身，而是奖励对象的 ID。为了获取奖励以获取策略，您必须调用 ray.get。

希望这能给你一点关于如何在 Ray 中编程的风味。下次，我会转换方向，介绍 Ray 的系统设计；Ray 如何实现高性能和可扩展性。

就像许多经典的计算框架一样，它有一个**驱动程序**和一群**工作者**。驱动程序运行一个程序，工作者远程运行任务。你可以运行和编写一群**演员**。驱动器上的演员在同一节点上，它们共享数据，在共享内存上，工作者和跨节点的演员通过我们构建的分布式对象存储进行共享。每个节点都有一个本地调度程序，因此当驱动程序想要运行另一个任务时，本地调度程序会尝试在本地进行调度。如果无法在本地调度，则调用全局调度程序，并且将在具有资源的另一节点上进行调度。演员，远程方法。最后，我们所做的，设计的一个重要部分，就是我们有一个全局控制状态。它获取系统的所有状态，并对其进行集中管理。对象表的对象的元数据，函数。这使系统成为无状态的。所有这些其他组件都可能失败，您可以将它们启动起来，并从全局控制状态获取最新的数据。它还允许我们对全局调度程序进行并行化处理，因为这些复制品将共享 GCS 中的相同状态。

拥有 GCS 的另一个好处是，它使构建一群分析和调试工具变得容易。

此设计具有高度可扩展性。让我试着说服你为什么这样。要使 GCS 可扩展，我们只需将其分片。所有这些密钥都是伪随机的，因此易于分片和负载平衡。正如您所见，调度程序是分布式的；每个节点都有一个本地调度程序，Ray 尝试调度由工作者/驱动程序生成的任务，该任务是本地生成的另一个任务。全局调度程序成为一个瓶颈，我们还可以复制它。最后，在系统中，即使调度程序超级可扩展，在 Spark 中，还有另一个瓶颈：只有驱动程序可以启动新任务。为了解决这个问题，在 Ray 中，我们允许工作者和演员启动任务。实际上，没有单一的瓶颈点。

关于实现的一些话。GCS 采用 Redis 实现。对于对象存储，我们利用 Apache Arrow。对于容错性，我们使用基于线 age 的容错性，类似于 Spark。Actor 是任务图的一部分；方法被视为任务，因此我们有一个统一的模型来提供容错性。

现在是一些评估结果。这张图表示每秒的任务数，您可以看到节点数；它线性扩展。您可以安排超过 1.8M/s。本地任务执行的延迟为 300 微秒，远程任务的延迟为 1 毫秒。这张图说明了容错性。您可能会问为什么关心容错性？问题在于，程序中可能需要模拟未完成；即使您愿意忽略一些结果，这也使程序变得更加复杂。在这个轴上，您有秒数的时间，有两个 Y 轴，系统中的节点数和吞吐量。正如您所见，节点数从 50 开始，然后是 25，然后到 10，再回到 50。在红色区域，显示每秒的任务数；正如您所预期的那样，与系统中的节点数一致。如果您仔细看一下，会有一些下降；每次您都会看到任务数下降。事实证明，这是由于对象重建引起的。当某些节点离开时，您会丢失节点上的对象，因此必须对其进行重建。Ray 和 Spark 会自动透明地重建它们。通过蓝色，您可以看到重新执行的任务。如果将它们加起来，您将得到一个非常漂亮的填充曲线。

最后，对于进化策略，我们与参考的 ES 进行了比较……我们遵循了 OpenAI 的方式，在 X 轴上，您有 CPU 的数量，解决特定问题的平均时间；模拟器，学习运行，有三点值得注意。一是如预期的那样，随着 CPU 数量的增加，解决问题的时间减少。第二是 Ray 实际上比参考 ES 更好，获得了更好的结果，即使参考 ES 专门用于击败。第三，对于非常大量的 CPU，参考 ES 无法做到，但 Ray 可以做得越来越好。我应该补充说 Ray 只需要一半的代码量，并且在几个小时内实现了。

相关工作：在这个领域，有大量的系统，这就是你在这里的原因，很多系统。Ray 与 TF、MXNet、PyTorch 等相辅相成。我们使用这些系统来实现 DNNs。我们与 TF 和 PyT 进行集成。还有一些更通用的系统，如 MPI 和 Spark；它们对嵌套并行性、计算模型有一定的支持，任务粒度更粗。

总之，Ray 是一个高性能、灵活和可扩展的系统。我们在 Ray 的基础上有两个库：RLlib 和 Ray Tune。它是开源的，请试用，我们很乐意听取您的反馈。感谢我的同事迈克尔·乔丹以及罗伯特、菲利普、亚历克斯、斯蒂芬妮、理查德、埃里克、恒、威廉等人。

Q: 在你们的系统中，你们也使用了 actor；actor 是建立在共享内存上的。你们有单独的邮箱给 actor 吗？你们是怎么做到的？

A: 不，actor 通过将参数传递给共享对象存储进行通信。

Q: 并行性的粒度是什么？它是任务原子的，还是将任务分割了？

A: 任务的粒度由启动任务和调度任务的开销决定。你看到的任务，我们的目标是任务、低延迟和少量的 ms。任务不是实现类似激活函数的东西。我们把这项工作留给更好的框架。任务是以原子方式执行的，方法在 actor 中是串行化的。

Q: 在 Spark 中关于容错性的问题：当一段时间没有响应时，它会说这个节点死了。这里，任务更多，因为 NN，类似这样的东西。所以我们没有相同的时间。

A: 我们不进行猜测；Ray 中的隐式猜测，出于你提到的原因。

Q: 你能详细说明一下参考实现，它不具备可伸缩性

A: 参考实现，这是 OpenAI 的实现，Robert 在这里可以为您提供更详细的答案。