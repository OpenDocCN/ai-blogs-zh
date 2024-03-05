# 电子商务行业中的数据科学和机器学习:内部人士谈论工具、用例、问题等等

> 原文：<https://web.archive.org/web/https://neptune.ai/blog/data-science-and-machine-learning-in-the-e-commerce-industry-tools-use-cases-problems>

机器学习已经毫无保留地吞没了我们的个人和私人空间，扩展到了只受我们理解能力限制的领域。曾经用于统计的数学模式现在正被用于理解和预测人类行为。

[神经和语言计算](https://web.archive.org/web/20221207182621/https://www.aclweb.org/anthology/L16-1330.pdf)的进步为复杂的应用开辟了道路，通过求解[矩阵分解](https://web.archive.org/web/20221207182621/https://machinelearningmastery.com/introduction-to-matrix-decompositions-for-machine-learning/)方程来增强用户体验和客户指控。产品/客户到产品/客户的关系可以在稀疏矩阵中映射出来，以深入研究购买模式，并应用顺序模式分析来获得对客户群的一致理解。尽管机器学习的基本思想(减少错误)保持不变，但成本函数却截然不同

更令人印象深刻的是，用于计算这些模式的算法还很不成熟，因此这些算法还有很大的完善空间。这些算法是计算密集型的，因为它们中的大多数处理使用高阶矩阵。

目前，大多数电子商务的特定应用围绕着客户细分模型、矩阵分解模型、市场购物篮分析等，偶尔使用最新技术(神经矩阵分解)的实现。

云平台也注意到了零售和电子商务行业中的机会，并推出了易于与 web 应用程序集成的服务/工具。这导致了旨在利用这些云服务为特定用例创建定制模型的多个白皮书的产生。

**本文意在从一个数据科学家的角度给大家描绘一幅** [**电商行业**](https://web.archive.org/web/20221207182621/https://www.jigsawacademy.com/how-data-science-and-machine-learning-is-transforming-the-e-commerce-industry-in-2019/) **的图景。请继续阅读！**

## 我作为电子商务数据科学家的典型一天

在电子商务领域，数据科学家的日常生活极具挑战性。你永远无法预测人类的行为，但你有责任去做。当然，你可以分析并获得关于顾客购买模式的随机灵感，也可以见证一个营销决策如何在一夜之间改变你的假设。但是没有什么能比得上看到你的工作节省时间，增加客户的参与体验，同时实现客户的愿景。

举一个假设的例子，一家中小企业的营销活动针对的是错误的客户群，并收到了令人难以接受的结果。尽管该公司没有意识到这一点，但快速的数据分析会指出这一点，并增加其接触适当客户的机会。你每天都能解决这样的问题，这很令人满意。

## 电子商务中的数据科学和机器学习用例是什么

作为一名在零售领域工作的数据科学家，你所有的首要目标都围绕着“客户”。咄！你可以获得客户，也可以留住客户。因此，选择和优化的问题。在大多数情况下，这两个问题都有独立的解决方案，但在一些罕见的情况下，您可以构建一个解决方案来解决这两个问题。尽管解决方案大致分为两类，但实现这些解决方案的方式却是多种多样的。我可以继续讨论用例，但是我将把这个部分限制在 3 个主要的用例上。

1.推荐引擎

### [推荐引擎](https://web.archive.org/web/20221207182621/https://www.kdnuggets.com/2019/09/machine-learning-recommender-systems.html)是在线零售领域机器学习用例的缩影。就像知道你想要什么的销售人员一样，推荐引擎会得到你！它根据你过去的购买和互动来理解你，并给你个性化的建议。大多数推荐引擎使用的机器学习技术是矩阵分解算法。尽管背后的数学有点复杂(在下一节的矩阵分解部分会涉及到)，但是推荐引擎所能达到的效果是无与伦比的。

推荐引擎可用于定制您客户的体验。您可以为任何客户端快速部署它，因为它们在不同的语言中都可以使用，比如 R、Python、WEKA 等。像网飞、YouTube、亚马逊等公司使用用 Python 构建的推荐引擎，可以很容易地与他们的微服务交互。推荐引擎还允许你携带其他客户的经验，并推荐客户周围的人正在购买的产品。对推荐引擎的唯一警告是高度研究的“冷启动”问题。您需要历史客户数据来构建推荐引擎。没有这一点，就无法构建一个复杂的推荐引擎。有一些变通办法和大量的研究正在着手解决这个问题。

2.客户细分

### [客户细分](https://web.archive.org/web/20221207182621/https://morphio.ai/blog/machine-learning-for-customer-segmentation/)在很长一段时间内都被用于目标营销活动。基于客户之间的相似性(例如，地理相似性、购买力等)，可以将他们聚集成群体，如忠诚客户、高消费群体等。简而言之，客户细分将客户分成更小的群体，这些群体具有适当的针对性。可以逻辑地或者使用机器学习算法来识别片段。后者几乎总是更有效，因为有时客户之间的相似之处并不明显。

简单地说，客户细分模型是通过聚类技术创建的。有许多可用的技术，但最常用的方法是使用 K-means 聚类技术。一旦集群被创建，高产集群被确定，并集中营销活动运行。识别这些集群需要对你要销售的产品的零售空间有深刻的理解。根据市场动态和全球经济变化，有时集群可能不会像预期的那样运行。因此，在使用客户细分时，你需要考虑风险因素。

3.隐马尔可夫模型

### [隐马尔可夫模型](https://web.archive.org/web/20221207182621/https://towardsdatascience.com/markov-and-hidden-markov-model-3eec42298d75)或 hmm 是数据科学用例的最新发展。它们有着广泛的应用，并且在推动洞察力方面非常富有成效。通过利用丰富的基于位置的数据来获得关于客户的见解，hmm 可用于检测基于位置的购买。HMM 从学习给定用户的位置序列和客户购买的最可能序列开始。概率分数被分配给新的购买，并且这些新的购买基于它们的可能性被推荐给客户。

hmm 还可以用于客户分离，其中它们可以通过马尔可夫序列分析(通过将隐藏状态分配给客户的消费活动)来捕捉客户行为的动态。hmm 也可以用来做流失建模和拖欠预测。使用 hmm 可以做很多事情，因为它们有助于验证客户的信誉，并基于动态数据挖掘预测行为。

零售客户可以使用多种数据科学工具和框架。本节分为三小节，涵盖各种算法、低代码工具和代码密集型工具。

电子商务行业使用的机器学习算法

### 1.矩阵分解

#### 这种算法也称为矩阵分解法，用于有效地解决复杂的矩阵运算。随着使用神经元优化矩阵计算的深度学习技术的出现，一个简单的嵌入模型就可以完成这项工作。嵌入模型可以是基于用户的嵌入模型或基于产品的嵌入模型。使用神经网络的矩阵分解算法的一种广泛使用的实现是神经协同过滤模型(NCF)。NCF 是一种独特的基于深度学习的推荐引擎架构。它结合了标准矩阵分解的有效性以及神经网络的复杂性，以产生用户/项目的完整表示。

矩阵分解算法用于创建复杂的推荐引擎。矩阵分解模型创建了用户和他们交互过的产品之间的映射。以下面的表格为例，用户 1 购买口罩和消毒剂，而用户 2 购买燕麦片。创建具有这些参考的稀疏矩阵，并且基于用户之间的相似性向相应的新用户推荐这些产品之一。

用户/产品

| 面罩 | 消毒剂 | 燕麦片 |  |
| --- | --- | --- | --- |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

虽然这是矩阵分解模型的一个非常简单的例子，但是实际的算法基本上没有什么不同。用户之间的相似性可以使用客户终身价值或交互得分(通常作为模型的辅助输入)等指标来计算。这些度量既可以手动计算，也可以通过其密集层之一留给神经协同过滤模型。NCF 自动识别用户-产品对，并基于生成的概率分数建议可能的产品。

2.使聚集

聚类是营销团队用来将客户分成逻辑部分(客户细分)的一种非常常用的算法。k 表示聚类是一种广泛使用的聚类变体，因为它在创建有意义的聚类方面非常有效。还有其他技术，如基于密度的聚类、基于网格的聚类等，但它们很少用于创建电子商务用例的模型。理论上，基于密度的聚类应该显示出最好的结果，但实际上 K 意味着聚类几乎总是优于它。

#### 可视化集群的最佳方式是通过气泡。下面展示了如何使用气泡形成星团。

聚类取决于两个重要因素:

您希望创建的分段数(聚类数)

不同段之间的相似性(聚类之间的距离)

1.  虽然最佳聚类的数量是数据科学家可以控制的，但聚类之间的距离取决于他们正在处理的数据。您可以使用剪影评分或 Davies-Bouldin 指数等技术来确定最佳聚类，但在某种程度上，您选择的方法将始终取决于您拥有的数据。聚类在帮助你识别可以用于目标营销的客户群方面做得很好。
2.  3.分类

分类是一种常见的机器学习方法，有大量的用例。粗略定义的分类是根据历史数据将某些数据点归类的过程。这可以帮助我们将新用户分为潜在买家、忠实客户、购物车放弃者等。分类通常与电子商务用例的推荐引擎一起使用，因为单独的分类不能证明是有效的。矩阵分解使用分类变量(通常是将记录标记为销售或非销售的列)创建用户-产品对。然后，该信息被用于训练分类模型，以将特定的用户-产品对标记为销售或非销售。这可以用来有效地锁定那些最有可能促成销售的产品。

有一些问题陈述可以使用分类直接解决。例如，您的客户想要阻止不在您的网站上进行任何购买的 IP(称为“旁观者”)。旁观者通常无意购买，而且几乎总是不利于网上交易。把一个旁观者变成一个买家是一项不必要的任务，并且没有长期的收益。因此，一些服务器点击率有限的客户端可能会选择阻止 IPs。先发制人地对这些 IP 进行分类可以为在线业务省去很多麻烦。

#### 现在你已经知道了电子商务用例中使用的不同算法，让我们深入到可以帮助你在网站上实现机器学习的工具中。主要有两类，第一类不需要专门的内部数据科学家，而第二类需要内部数据科学家。

在您的电子商务用例中实现机器学习的低代码工具(成本密集型)

1.GA360

[Google Analytics 360](https://web.archive.org/web/20221207182621/https://support.google.com/analytics/answer/3437434) 是目前最复杂的零售加速器之一。它很容易与 GCP 服务(如 BigQuery)集成，并可用于获取洞察力。GA360 还提供了一个基于启发式的分割模型。虽然它不是一个机器学习实现，但将 GA360 与 BigQuery 结合起来可以帮助您构建一个由用户交互驱动的推荐引擎。由于 GA360 捕捉诸如点击次数、在页面上花费的时间、点击/购买的产品等信息，因此可以根据这些数据点来建模高效的推荐引擎。当然，你将不得不为这两种服务支付高额费用，但这种结合的好处是，即使没有历史用户数据，你也可以定制你的用户体验。因此，这是一个完美的工具，为任何客户寻找建立自己的网上存在。

### 2.BigQueryML

#### [BigQueryML](https://web.archive.org/web/20221207182621/https://cloud.google.com/bigquery-ml/docs/introduction) 是谷歌最复杂的低代码机器学习方法之一，用于构建复杂的机器学习模型。尽管这种方法要求您拥有历史用户数据，但是您不需要担心构建一个好的推荐引擎所需的复杂实现。使用 BigQueryML，您需要做的就是指定您希望运行的矩阵分解的类型，并让 BigQueryML 为您管理一切。假设您的团队缺乏对大量矩阵分解变体的理解，或者假设您想要加快构建推荐引擎的过程，而不需要停下来了解它是如何工作的——输入 BigQueryML 推荐引擎。一个完全托管的服务，它不仅选择合适的模型，还负责部署模型并为您提供推理。权力越大，费用越大！这两种服务都很昂贵，需要相当长的时间才能开始产生回报。

许多电子商务网站都有一个手控聊天机器人，帮助客户进行查询，并根据概述的要求推荐产品。构建一个定制的聊天机器人是一项具有挑战性的任务，但有很多好处。Dialogflow 是一种非常方便的将对话式人工智能与网站、web 应用程序、移动应用程序等集成的方式。您可以使用 DialogFlow CX 来构建一个定制的聊天机器人，它可以很容易地与大多数零售用例集成。这是集成人工智能助手的最快方法之一，人工智能助手不仅可以为客户提供个性化的体验，还可以解决电子商务网站的问题，如购物车故障、购物车放弃、支付声明或交易失败等。DialogFlow 还支持多种语言，不需要数据科学家来配置。这个工具是谷歌云平台套件的一部分，因此不能免费使用。

#### 4.使颗粒化

还记得我们在分类部分谈到的将用户分类为购买者和非购买者对企业的重要性吗？Granify 就是这么做的！它不仅通过识别不会购买产品的客户，还通过使用机器学习来诱使他们购买产品，从而帮助在线零售商。Granify 使用从分类、矩阵分解、图像处理等广泛的技术来准确地描绘客户旅程，并确定客户可能成为买家的最佳旅程点。他们确实有一个非常大胆的目标，即在实施的 90 天内实现大约 4%的收入增长，但这是一个可以使用的创新工具。Granify 不仅在你的零售网站上自动实施机器学习，而且还提供了分析到达你的网站的流量的巧妙方法，完全不需要数据科学团队。你所需要的是一群对他们的业务充满热情并对他们的客户群有深刻了解的人。

低代码方法是为您的在线业务实现方便的机器学习解决方案的快速方法。但这是以向第三方公开/共享您的客户信息为代价的。虽然他们中的大多数都有不使用个人身份信息的政策(PII)，但一些客户更进一步，雇佣内部数据科学家来创建他们的定制集成。对于此类用例，在为您的零售客户构建解决方案时，以下工具/技术会派上用场

在您的电子商务用例中实现机器学习的代码密集型工具(经济高效)

#### TensorFlow 是一个开源的 Python 库，用于创建深度学习机器学习模型。该库的一个非常具体的变体是在 TensorFlow 模型花园中实现神经矩阵分解。这是迄今为止实现复杂的协同过滤推荐引擎最快的方法。因为它是开源的，所以可以免费实现，并且很容易与大多数基础设施集成。但是需要一个强大的 Python 开发人员和一个 ML 工程师在生产环境中训练和部署 NeuMF 模型。

NCF 是用于推荐的协同过滤的通用框架，其中神经网络体系结构被用于对用户-项目交互进行建模。不同于传统的模型，NCF 不求助于矩阵分解(MF)与用户和项目的潜在特征的内积。它用多层感知器取代了内积，可以从数据中学习任意函数。

NCF 的两个实例是广义矩阵分解(GMF)和多层感知器(MLP)。GMF 应用线性核来模拟潜在的特征相互作用，而 MLP 使用非线性核从数据中学习相互作用函数。NeuMF 是 GMF 和 MLP 的融合模型，以更好地建模复杂的用户-项目交互，并统一了 MF 的线性和 MLP 的非线性的优势，用于建模用户-项目潜在结构。NeuMF 允许 GMF 和 MLP 学习单独的嵌入，并通过连接它们的最后一个隐藏层来组合这两个模型。虽然 NeuMF 在 JavaScript 中没有直接实现，但是随着 TensorFlow Js 的出现，您可以将 TensorFlow NeuMF 代码直接导出为 JavaScript 对象，并将其与您的电子商务网站集成。但是请记住，NCF 模型通常很大，需要在您的服务器上有一个专用的存储容器。

Python 是创建机器学习模型和人工智能应用程序最常用的编程语言之一。虽然 Python 本身有许多框架来帮助构建电子商务网站(例如 Flask)，但它通常用于使用 SKLearn、TensorFlow、Pytorch、Theano 等库来构建、训练和部署复杂的模型。Python 还打开了构建自然语言处理模型或创建聊天机器人框架的大门。

### 借助强大的数据科学团队，您可以构建最适合客户用例的定制机器学习模型。此外，一些电子商务网站很容易使用基于 Python 的 web 框架来构建。这样的网站将很容易与你基于 python 的机器学习模型集成。您还可以围绕节点服务器创建包装器，或者使用双节点 Python 服务器来部署您的模型。还可以创建一个专用的微服务或 API，从您的模型中生成推理。因此，使用 Python 的可能性是无限的。

与其他行业有什么不同？

作为一名与零售客户打交道的数据科学家，很难忽视零售领域的独特方面。电子商务行业在很大程度上依赖于客户的互动，而客户是这一切的中心。有人可能会说，你需要一个现有的供应链，一个创新的产品，对该产品的需求，等等，但无论有没有必要的先决条件，一个电子商务网站需要客户访问网站并与之互动。只要你能为你的网站带来流量，你就已经成功了一半。以下一组属性将电子商务行业与任何其他行业区分开来

互联网

互联网是一个巨大的空间，任何人/每个人都在上面。你客户的竞争对手可能在上面，你的潜在买家可能在上面，或者一个阅读这篇文章的技术专家可能在上面。这有一种两重性:你的客户群几乎是无限的，但相比之下，你的买家是微不足道的。这是一个相当大的难题，因此为目标营销、区域限制等开辟了道路。任何初创企业/中小企业(SME)的一个经验法则是，它们不应该增长太快。在互联网上，你无法控制观众。因此，随着顾客的突然到来，你的生意很有可能蒸蒸日上。如果没有合适的基础设施，这可能会产生不利影响。基于互联网的服务的另一个方面是顾客评论像火一样传播。还记得 MySpace 吗？你们中的一些人肯定没听说过 MySpace，但它就是今天的脸书。你在任何其他行业都看不到如此不稳定的客户群！

个性化

## 众所周知，电子商务行业提供定制体验。让顾客觉得自己与众不同是一种强有力的策略，而且非常有用。没有它是一个明确的失望，并可能导致客户流失。而且你不为体验向顾客收费！这在其他任何行业都很少见。个性化几乎总是一种附加产品或“奢侈”产品。但是对于电子商务行业来说，这是必须要做的事情！

客户范围

### 预测一个顾客是否能够/愿意购买你的产品只有在网上零售行业才有可能。如果使用得当，网上零售业务可以接触到正确的观众，而不用迈出第一步。这是因为网上有大量的客户。今天，每个人都可以以某种形式访问互联网。互联网上的人数只会增加。这为一般的在线零售商创造了一个更大的接触人群的媒介。将社交媒体加入其中，你就有了一个合适的组合，可以接触到你的受众，而不用花钱在昂贵的实体广告牌上。

分析学

### 虽然这不是电子商务特有的，但它被在线企业大量使用。事实上，分析的诞生可以追溯到互联网泡沫。分析作为一个领域在零售企业，尤其是在线零售企业中非常普遍。这是因为在线零售商不仅拥有更大的覆盖范围，而且拥有将顾客立即转化为买家的媒介。

搜索引擎排名

### 搜索引擎排名是一个有争议的问题。这是有争议的，因为要想排名更高，你必须遵守某个营销集团的规则。这些规则通常冗长且不总是一致的。为什么在搜索引擎上排名靠前很重要？普通人的注意力持续时间每年都在急剧下降。这导致一大群人查看呈现给他们的前 4-5 个链接。因此，这是一个相当独特的挑战，因为你不只是在与你的竞争对手竞争，而是与整个互联网竞争。

刚开始做电商最让我惊讶的是什么？

### 除了这些不同点，电子商务是最酷的数据科学行业之一。我总是惊讶于一个简单的网站如何能够成为几个不同的机器学习微服务的高潮——所有这些服务一起工作以提供整体体验。应用不仅限于一个研究领域。你可以使用计算机视觉、自然语言处理、深度学习、模糊逻辑等等。

令我惊讶的是，电子商务运营转移到云端的速度有多快。在电子商务领域有大量业务的大型零售客户要么提出他们的云基础设施(咳咳！亚马逊)或将他们的整个基础设施转移到一个新兴的云服务提供商(沃尔玛转移到谷歌)。电子商务不再是一个由 Web 开发者驱动的行业。这让我对我能够处理的不同用例有了新的认识。

### 电子商务行业为自己开辟了一个利基市场，为就业和创新创造了大量空间。虽然这是一个让大型零售商获得更大客户群的空间，但也为小型在线企业推广其产品/服务创造了机会。我一直相信歌利亚粉碎大卫·习语，一个巨大的企业集团最终会通过大规模生产他们的产品来粉碎一个小企业。但是我已经看到许多小企业由于他们的在线存在而蓬勃发展。他们中的一些人利用基本的数据科学技术来接触他们的受众，而其他人则使用第三方广告工具来增加他们网站的流量。

虽然可以写一篇单独的文章来回答这个问题，但零售领域永远不会停止给你带来惊喜。

作为电子商务领域的数据科学家，最大的挑战是什么？

作为在线企业的数据科学家，最具挑战性的任务之一是精心设计一个成立的假设！在任何行业中，提出一个假设通常都是相当具有挑战性的，但是由于电子商务行业的动态性质及其完全不可预测的性质，一个经过深思熟虑的假设往往会失败。它失败不是因为不正确的假设或缺乏所有变量的因素；它之所以失败，是因为客户行为的根本转变。那可能是你需要经常重建和重新部署一个机器学习模型。

另一个挑战是电子商务客户有非常不同的业务需求。这产生了一个尖锐的学习曲线，重叠范围非常小。而客户终身价值、市场篮子分析等基本指标和策略保持不变。它们对特定业务的实现取决于该业务的目标和期望。因此，领域知识只能被部分重用。

另一个挑战是用户个性化选项的饱和。虽然基于 UI 的个性化非常多，但用于增强客户体验的机器学习用例几乎总是推荐引擎。构建推荐引擎永远不会出错，但大多数在线企业都会雇佣数据科学家来构建某种形式的推荐引擎。对一些人来说，这似乎不是一个挑战，但它必须达到一个饱和点。

结论

## 我希望这篇文章能够让您对电子商务行业有所了解。有许多数据科学用例可供您开始使用。我在下面提供了几个链接，以增强你对我提到的一些概念的理解。更令人兴奋的是，电子商务行业几乎总是在招聘数据科学家。喜欢这篇文章的某些方面，还是想让我详细说明某些方面？请提供您的宝贵反馈

资源:

请务必阅读这篇文章，并在你的网络中分享。

Another challenge is the saturation of options for user personalization. While UI based personalizations are plenty, a machine learning use case for enhancing customer experience is almost always a Recommendation Engine. You can never really go wrong with building a Recommendation Engine but most online businesses hire Data Scientists to build some form of recommendation engine. This might not seem like a challenge to some but it’s got to reach a point of saturation.

## Conclusion

I hope this article was able to give you a few insights into the E-commerce industry. There are a lot of Data Science use cases out there that you can start and get your hands dirty with. I have provided a few links below to enhance your understanding of some of the concepts I touched upon. What’s even more exciting is that the E-commerce industry is almost always hiring for a Data Scientist. Like something about the article or want me to elaborate on something? Please do provide your valuable feedback

Resources:

Do read this and share it across your network.