# Bert改进模型汇总

## 1. 介绍

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/bert%E6%A8%A1%E5%9E%8B%E6%94%B9%E8%BF%9B%E6%80%BB%E4%BD%93%E5%9B%BE1.png)

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/bert%E6%A8%A1%E5%9E%8B%E6%94%B9%E8%BF%9B%E6%80%BB%E4%BD%93%E5%9B%BE2.png)

## 2. 模型改进

### 2.1 ERNIE from Baidu

[参考](https://blog.csdn.net/sinat_25394043/article/details/104188322?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&dist_request_id=1332023.6423.16189681239640711&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)

[论文地址](https://arxiv.org/pdf/1904.09223.pdf)

[GitHub地址](https://github.com/PaddlePaddle/LARK/tree/develop/ERNIE)

百度提出的ERNIE模型主要是**针对BERT在中文NLP任务中表现不够好提出的改进**。我们知道，对于中文，**bert使用的基于字的处理，在mask时掩盖的也仅仅是一个单字**。

于是文章提出一种知识集成的BERT模型，别称ERNIE。ERNIE模型在BERT的基础上，加入了海量语料中的实体、短语等先验语义知识，建模真实世界的语义关系。

那么怎么样才能使得模型学习到文本中蕴含的潜在知识呢？不是直接将知识向量直接丢进模型，而是在训练时将短语、实体等先验知识进行mask，强迫模型对其进行建模，学习它们的语义表示。

此外，为了更好地建模真实世界的语义关系，ERNIE预训练的语料引入了多源数据知识，包括了中文维基百科，百度百科，百度新闻和百度贴吧（可用于对话训练）。

具体来说， **ERNIE采用三种masking策略：**

**Basic-Level Masking： 跟bert一样对单字进行mask，很难学习到高层次的语义信息；**

**Phrase-Level Masking： 输入仍然是单字级别的，mask连续短语；**

**Entity-Level Masking： 首先进行实体识别，然后将识别出的实体进行mask。**



### 2.2 ERNIE from THU

[参考](https://blog.csdn.net/sinat_25394043/article/details/104188322?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&dist_request_id=1332023.6423.16189681239640711&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)

[论文地址](https://arxiv.org/pdf/1905.07129.pdf)

[GitHub地址](https://github.com/thunlp/ERNIE)

已有的预训练语言模型**很少考虑知识信息**，具体而言即知识图谱（knowledge graphs，KG），知识图谱能够提供丰富的结构化知识事实，以便进行更好的知识理解。简而言之，预训练语言模型只知道语言相关的「合理性」，它并不知道语言到底描述了什么，里面是不是有什么特殊的东西。

该研究结合大规模语料库和知识图谱训练出增强版的语言表征模型 (ERNIE)，该模型可以同时充分利用词汇、句法和知识信息。实验结果表明 ERNIE 在多个知识驱动型任务上取得了极大改进，在其他 NLP 任务上的性能可以媲美当前最优的 BERT 模型。

为此，作者们提出了ERNIE模型，同时在大规模语料库和知识图谱上预训练语言模型：

**抽取+编码知识信息**： 识别文本中的实体，并将这些实体与知识图谱中已存在的实体进行实体对齐，具体做法是采用知识嵌入算法（如TransE），并将得到的entity embedding作为ERNIE模型的输入。基于文本和知识图谱的对齐，ERNIE 将知识模块的实体表征整合到语义模块的隐藏层中。

**语言模型训练**： 在训练语言模型时，除了采用bert的MLM和NSP，另外随机mask掉了一些实体并要求模型从知识图谱中找出正确的实体进行对齐（这一点跟baidu的entity-masking有点像）。
okay，接下来看看模型到底长啥样？

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/ERNIE%20from%20THU.png)

如上图，整个模型主要由两个子模块组成：

底层的textual encoder (T-Encoder)，用于提取输入的基础词法和句法信息，N个，T-Encoder 依然是对原来的文本进行编码，这部分和 BERT 是一样的；

高层的knowledgeable encoder (K-Encoder)， 用于将外部的知识图谱的信息融入到模型中，M个，在 K-Encoder 中，可以看到输入输出都变成了两个，多了 entity 的信息。

这里T-encooder跟bert一样就不再赘述，主要是将文本输入的三个embedding加和后送入双向Transformer提取词法和句法信息：
$$
｛w_{1},...,w_{n}｝ = T -Encode(｛w_{1},...,w_{n}｝)
$$
K-encoder中的模型称为aggregator，输入分为两部分：

- 一部分是底层T-encoder的输出 :$｛w_{1},...,w_{n}｝$
- 一部分是利用TransE算法得到的文本中entity embedding， $｛e_{1},...,e_{n}｝$
- 注意以上为第一层aggregator的输入，后续第K层的输入为第K-1层aggregator的输出
  接着利用multi-head self-attention对文本和实体分别处理：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/aggregator%E5%85%AC%E5%BC%8F.png)

然后就是将**实体信息和文本信息进行融合**，实体对齐函数为  :![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/20200205213830526.png)

对于有对应实体的输入：![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/2020020521384348.png)


对于没有对应实体的输入词：![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/20200205213845749.png)


上述过程就是一个aggregator的操作，整个K-encoder会叠加M个这样的block：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/20200205213854520.png)

最终的输出为最顶层的Aggregator的token embedding和entity embedding。



**训练细节**

----

从模型上实现了知识图谱信息的有效融合，那如何训练呢？如果单纯还是和 BERT 的训练方式相同，知识图谱的知识信息可能并不能如期望的那样进行有效融合，因此作者参考 Masked Language Model 设计了一个 denoising Entity Auto-encoder (dEA) 任务，用以训练模型对实体信息的感知和对齐，具体内容如下。

dEA 的目的就是要求模型能够根据给定的实体序列和文本序列来预测对应的实体，首先是实体和文本之间的对齐概率计算：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/dEA%E5%85%AC%E5%BC%8F)

这个公式也被当作训练 dEA 时的损失函数，有了目标，那么数据该如何准备呢？和 Masked Language Model 类似，作者对实体也做了如下处理： 

对于一个给定的文本-实体对应序列，5% 的情况下，实体会被替换为一个随机的实体，这么做是为了让模型能够区分出正确的实体对应和错误的实体对应； 
对于一个给定的文本-实体对应序列，15% 的情况下，实体会被 mask，这是为了保证模型能够在文本-实体没有被完全抽取的情况下找到未被抽取的对应关系； 
对于一个给定的文本-实体对应序列，剩下的 80% 的情况下，保持不变，这是为了保证模型能够充分利用实体信息来增强对文本语义的表达。 



### 2.3 MASS:Masked Sequence to Sequence Pre-training for Language Generation

[参考地址](https://blog.csdn.net/sinat_25394043/article/details/104256122?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-4.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-4.control)

[论文地址](https://arxiv.org/abs/1905.02450)

[GitHub](https://github.com/microsoft/MASS) 

BERT在自然语言理解（比如情感分类、自然语言推理、命名实体识别、SQuAD阅读理解等）任务中取得了很好的结果，受到了越来越多的关注。然而，在自然语言处理领域，除了自然语言理解任务，还有很多序列到序列的自然语言生成任务，比如机器翻译、文本摘要生成、对话生成、问答、文本风格转换等。在这类任务中，目前主流的方法是编码器-注意力-解码器框架。

编码器（Encoder）将源序列文本X编码成隐藏向量序列，然后解码器（Decoder）通过注意力机制（Attention）抽取编码的隐藏向量序列信息，自回归地生成目标序列文本Y。

专门针对序列到序列的自然语言生成任务，微软亚洲研究院提出了新的预训练方法：**屏蔽序列到序列预训练（MASS: Masked Sequence to Sequence Pre-training）**。MASS对句子随机屏蔽一个长度为k的连续片段，然后通过编码器-注意力-解码器模型预测生成该片段。

![图 1 MASS训练结构](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/MASS%E8%AE%AD%E7%BB%83%E7%BB%93%E6%9E%84.png)

如上图所示，编码器端的第3-6个词被屏蔽掉，然后解码器端只预测这几个连续的词，而屏蔽掉其它词，图中“_”代表被屏蔽的词。
MASS预训练有以下几大优势：

1.解码器端其它词（在编码器端未被屏蔽掉的词）都被屏蔽掉，以鼓励解码器从编码器端提取信息来帮助连续片段的预测，这样能促进编码器-注意力-解码器结构的联合训练；

2.为了给解码器提供更有用的信息，编码器被强制去抽取未被屏蔽掉词的语义，以提升编码器理解源序列文本的能力；

3.**让解码器预测连续的序列片段，以提升解码器的语言建模能力。**

统一的预训练框架

MASS有一个重要的超参数k（屏蔽的连续片段长度），通过调整k的大小，MASS能包含BERT中的屏蔽语言模型训练方法以及GPT中标准的语言模型预训练方法，使MASS成为一个通用的预训练框架。

MASS预训练的loss 函数如下：![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/MASS_loss.png)

当k=1时，根据MASS的设定，编码器端屏蔽一个单词，解码器端预测一个单词，如下图所示。解码器端没有任何输入信息，这时MASS和BERT中的屏蔽语言模型的预训练方法等价。

![图 2 遮蔽方法，k=1](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/MASS_k_1.png)

当k=m（m为序列长度）时，根据MASS的设定，编码器屏蔽所有的单词，解码器预测所有单词，如下图所示，由于编码器端所有词都被屏蔽掉，解码器的注意力机制相当于没有获取到信息，在这种情况下MASS等价于GPT中的标准语言模型。

![图 3 遮蔽方法，k=m](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/MASS_K_m.png)

可以看到，当K=1或者m时，MASS的概率形式分别和BERT中的屏蔽语言模型以及GPT中的标准语言模型一致。![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/MASS_K.png)



### 2.4 UNILM:UNIfied pre-trained Language Model

[参考地址](https://blog.csdn.net/sinat_25394043/article/details/104256122?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-4.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-4.control)

[论文地址](https://arxiv.org/abs/1905.03197)

[GitHub](https://github.com/microsoft/unilm)

UniLM 是一种统一的预训练语言模型，既可应用于自然语言理解（NLU）任务，也能用于自然语言生成（NLG）任务。采用BERT的模型，**使用三种特殊的Mask的预训练目标**，从而使得模型可以用于NLG，同时在NLU任务获得和BERT一样的效果。 模型使用了三种语言模型的任务：

unidirectional prediction

bidirectional prediction

seuqnece-to-sequence prediction

给定一个输入序列 x，UniLM 会获取每个 token 的基于上下文的向量表征。如图 1 所示，预训练会根据多种无监督语言建模目标优化共享的 Transformer 网络，这些目标为单向语言模型、双向语言模型、序列到序列语言模型。为了控制对所要预测的词 token 的上下文的读取，作者使用了不同的自注意掩码。换句话说，作者使用了掩码来控制在计算基于上下文的表征时 token 应该关注的上下文的量。UniLM 训练完成后，当用于下游任务时，我们可以使用特定于任务的数据来对其进行微调。![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/UniLM.png)



### 2.5 SpanBert: Improving Pre-training by Representing and Predicting Spans

[参考地址](https://blog.csdn.net/sinat_25394043/article/details/104258344?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.control)

[论文地址](https://arxiv.org/abs/1907.10529)

刚看论文题目 SpanBERT: Improving Pre-training by Representing and Predicting Spans，以为是篇水文章，Bert 遮盖（mask）掉一小段（span）的 idea 并不新了，早有人做过，如百度 ERNIE，还有 Google 放出的 WWM (Whole Word Masking) BERT 模型，都是类似做法，当然细节上会有些不同。

所以细节决定成败，当仔细读完这篇论文，才发现，真香。里面对 Bert 预训练细节的探索非常有趣，很有启发。

这篇论文的主要贡献有三：

1. 提出了**更好的 Span Mask 方案**，也再次展示了随机遮盖连续一段字要比随机遮盖掉分散字好；
2. 通过加入 **Span Boundary Objective (SBO) 训练目标**，增强了 BERT 的性能，特别在一些与 Span 相关的任务，如抽取式问答；
3. 用实验获得了和 XLNet 类似的结果，发现**不加入 Next Sentence Prediction (NSP)** 任务，直接用连续一长句训练效果更好。

整体模型结构如下：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMzLnpoaW1nLmNvbS84MC92Mi1jNjhiNDY2YzVjZmZhNWRhOGFmNjg0ZjllMTYyYjkzZV9oZC5qcGc?x-oss-process=image/format,png)



**Span Masking**

----

首先什么是 Span Masking，和一般 BERT 训练有何不同。

对于原始的 BERT，训练时，会随机选取整句中的最小输入单元 token 来进行遮盖。因为用到 **Byte Pair Encoding （BPE）**技术，所以也可以把这些最小单元当作是子词（subword），比如说superman，分成 super+man 两个子词。

但这样会让本来应该有**强相关的一些连在一起的字词**，在训练时是割裂开来的。

因此我们就会想到，那能不能遮盖掉这样连在一起的片段训练呢？当然可以。

首先想到的做法，既然现在遮盖子词，那能不能直接遮盖整个词，比如说对于 super + man，只要遮盖就两个**同时遮盖**掉，这便是 Google 放出的 **BERT WWM** 模型所做的。

于是能不能进一步，因为有些实体是几个词组成的，直接将这个实体都遮盖掉。因此百度在 **ERNIE** 模型中，就引入命名实体（Named Entity）外部知识，**遮盖掉实体单元**，进行训练。

以上两种做法比原始做法都有些提升。但这两种做法会让人认为，或许必须得引入类似词边界信息才能帮助训练。但前不久的 MASS 模型，却表明可能并不需要，随机遮盖可能效果也很好，于是就有本篇的 idea：

根据**几何分布**，先随机选择一段（span）的**长度**，之后再根据**均匀分布随机选择**这一段的起始位置，最后按照长度遮盖。文中使用几何分布取 p=0.2，最大长度只能是 10，**利用此方案获得平均采样长度分布**。



**Span Boundary Objective**

---

Span Boundary Objective 是该论文加入的新训练目标，希望被遮盖 Span 边界的词向量，能学习到 Span 的内容。或许作者想通过这个目标，让模型在一些需要 Span 的下游任务取得更好表现，结果表明也正如此。

具体做法是，在训练时取 Span 前后边界的两个词，值得指出，这两个词不在 Span 内，然后用这两个词向量加上 Span 中被遮盖掉词的位置向量，来预测原词。

对于每一个带掩膜的分词 $(x_{s},...,x_{e})$ ，使用（s, e）表示其起点和终点。对于分词中的每个单词 xi ，使用外边界单词 xs-1 和 xe+1 的编码进行表示，并添加其位置嵌入信息 pi ，如下：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/%E7%BC%96%E7%A0%81%E8%A1%A8%E7%A4%BA.png)

详细做法是将词向量和位置向量拼接起来，作者使用一个两层的前馈神经网络作为表示函数，该网络使用 GeLu 激活函数，并使用层正则化：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/20190811181328244.png)

作者使用向量表示 yi 来预测 xi ，并和 MLM 一样使用交叉熵作为损失函数，就是 SBO 目标的损失，之后将这个损失和 BERT 的 **Mased Language Model （MLM）**的损失加起来，一起用于训练模型。

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/20190811181628698.png)



**Remove Next Sentence Prediction**

---

SpanBERT 还有一个和原始 BERT 训练很不同的地方，它没用 Next Sentence Prediction (NSP) 任务，而是直接用 Single-Sequence Training，也就是根本不加入 NSP 任务来判断是否两句是上下句，直接用一句来训练。作者推测其可能原因如下：（a）更长的语境对模型更有利，模型可以获得更长上下文（类似 XLNet 的一部分效果；（b）加入另一个文本的语境信息会给MLM 语言模型带来噪音。

因此，SpanBERT 就没采用 NSP 任务，仅采样一个单独的邻接片段，该片段长度最多为512个单词，其长度与 BERT 使用的两片段的最大长度总和相同，然后 MLM 加上 SBO 任务来进行预训练。



### 2.6 RoBERTa:Robustly optimized BERT approach
[论文地址](https://arxiv.org/pdf/1907.11692.pdf) 

[Github]( https://github.com/pytorch/fairseq)

RoBerta对于Bert的主要改进有：

(1) 对模型进行更长时间、更大批量、更多数据的训练；

(2) 删除NSP；

(3) 对较长序列进行训练；

(4) 动态改变应用于训练数据的 masking 模式-----dynamic masking。



**More Data** 

---

训练数据上，RoBERTa 采用了 160G 的训练文本，而 BERT 仅使用 16G 的训练文本， 其中包括： BOOKCORPUS 和英文维基百科：原始 BERT 的训练集，大小 16GB 。 CC-NEWS：6300 万篇英文新闻，大小 76 GB（经过过滤之后）。 OPENWEBTEXT：从 Reddit 上的网页内容，大小 38 GB 。 STORIES：CommonCrawl 数据集的一个子集，大小 31GB 。



**Large Batch/More Steps/larger byte level BPE vocab**

---

RoBERTa 模型增加了训练的 batch_size，并将 adam 的 0.999 改成了 0.98， 增加了训练的 step，最后使用的 batch_size 为 8k，训练步数为 500k 步。输入的 token 编码为 BPE 编码。



**No Next Sentence Prediction**

---

类似SpanBert



**dynamic masking**

---

何为 dynamic，只因原 BERT 预处理数据时，先处理好哪些位置被 mask，于是乎，炼制时，也就用着这些 static mask。而 dynamic 却是在训练时实时 mask，纵然是同一句话，每次 mask 位置也是随机不同。

### 2.7 XLNet

[参考地址](https://zhuanlan.zhihu.com/p/71916499)



**Permutation Language Model**

---

**AR （AutoRegression， 自回归）** 和 **AE （AutoEncoder， 自编码器）**。根据前面所有信息预测后一个 token，不断重复（自回归），本质上是在进行某种 Density Estimation （密度估计）；而另一类，则是**类 BERT 的 AE 方式**，做法是类似 DAE （Denoising AutoEncoder, 去噪自编码器）中把输入破坏掉一部分，然后还原，BERT 具体做法就是**随机将一些 token 替换成 “[MASK]” 特殊符**。

在 AR 以及 AE 方式中再加入一个步骤，就能够完美地将两者统一起来，那就是 **Permutation**.

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/Permutation%20Language%20Model.jpg)

具体实现方式是，**通过随机取一句话排列的一种，然后将末尾一定量的词给“遮掩”（和 BERT 里的直接替换 “[MASK]” 有些不同）掉，最后用 AR 的方式来按照这种排列方式依此预测被“遮掩”掉的词**。

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/Permutatio2.jpg)

比如说序号依次为 1234 的句子，先随机取一种排列，3241。于是根据这个排列我们就做出类似上图的 Attention Mask，先看第1行，因为在新的排列方式中 1 在最后一个，根据从左到右 AR 方式，1 就能看到 234 全部，于是第一行的 234 位置是红色的（没有遮盖掉，会用到），以此类推，第2行，因为 2 在新排列是第二个，只能看到 3 于是 3 位置是红色，第 3 行，因为 3 在第一个，看不到其他位置，所以全部遮盖掉...

这就是这篇论文的核心思想，看到这里其实已经能去和小伙伴们吹了，接下来会介绍，**XLNet 对 PLM 理念的实现**，文末会列出一种我认为可能的另一种实现方式。



**Two-Stream Self-Attention**

---

为了实现 Permutation 加上 AR 预测过程，首先我们会发现，打乱顺序后**位置信息非常重要**，同时对每个位置来说，需要预测的是内容信息（对应位置的词），于是**输入就不能包含内容信息**，不然模型学不到东西，只需要直接从输入 copy 到输出就好了。

于是这里就造成了位置信息与内容信息的割裂，因此在 BERT 这样的位置信息+内容信息输入 Self-Attention (自注意力) 的流（Stream）之外，作者们还增加了另一个**只有位置信息作为 Self-Attention 中 query 输入的流**。文中将前者称为 **Content Stream**，而后者称为 **Query Stream**。

这样子就能利用 Query Stream 在对需要预测位置进行预测的同时，又不会泄露当前位置的内容信息。具体操作就是用两组隐状态（hidden states），g 和 h，其中 g 只有位置信息，作为 Self-Attention 里的 Q，h 包含内容信息，则作为 K 和 V.

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-abc692352f91958fb6ba6c01d33503c2_720w.jpg)

假如说，模型只有一层的话，其实这样只有 Query Stream 就已经够了。但如果将层数加上去的话，为了取得更高层的 h，于是就需要 Content Stream 了。h 同时作为 Q K V。

![img](https://pic3.zhimg.com/80/v2-62067b9a93e7ccd22000d30eeb17ad66_720w.jpg)

于是组合起来就是这样。

![img](https://pic1.zhimg.com/80/v2-83a700244f18e5492b8d83cb100d19ec_720w.jpg)

最下面一层蓝色的 Content Stream 的输入是 e(x) ，这个很好懂就是 x 对应的词向量 (Embedding)，不同词对应不同向量，但看旁边绿色的 Query Stream，就会觉得很奇怪，为什么都是一样的 w？这个和后面的Relative Positional Encoding 有关，之后细说。

（b)图中为了便于说明，只将当前位置之外的 h 作为 K 和 V，但实际上实现中应该是所有时序上的 h 都作为 K 和 V，最后再交给 （c）图中的 Query stream 的 Attention Mask 来完成位置的遮盖。



**Partial Prediction**

---

在 Permutation 后对每个位置进行预测的话，会导致优化过难，**训练难以收敛**，和 BERT 中类似的操作，训练时，**只对每句话部分位置进行预测**。

这些预测位置如何选取呢，选当前排列的最后几个位置。举个例子，假如有 1234567，先随机挑一个排列，5427163，那么假设对最后两个位置预测，于是就需要依此对6和3进行预测。通过挑结尾的位置，在 AR 中，就能在预测时用到尽可能多的可知信息。

这里再谈一个有意思的点，挑选最后几个，那么**到底该挑选几个**呢，总得给个标准吧。于是作者这里设了一个超参数 K，**K 等于总长度除以需要预测的个数**。拿上面的例子，总长为 7 而需要预测为 2，于是 K = 7/2.

而论文中实验得出的最佳 K 值介于 6 和 7 （更好）之间，其实如果我们取 K 的倒数，然后转为百分比，就会发现**最佳的比值介于 14.3% 到 16.7% 之间**，还记得 BERT 论文的同学肯定就会开始觉得眼熟了。因为 BERT 里**将 Token 遮掩成 “[MASK]” 的百分比就是 15%**，正好介于它们之间，我想这并不只是偶然，肯定有更深层的联系。

还有一点需要格外指出，被预测之前的其实取不取 Permutation 都没关系，因为本身位置信息也都在里面，permutation 反而有些更难理解。

上面就是论文的主要部分，下面是一些更细节实现，比如如何从 **Transformer-XL** 借来各种部件。



**Segment Recurrence Mechanism**

---

Transformer-XL 的重要组件之一，**Segment Recurrence Mechanism**（段循环机制）。

其实思想很简单，因为一般训练 Transformer 时，会按照一定长度，将文本处理成一段（segment）一段的。比如说 BERT 预处理时，就会先处理成一个个 512 长度的样本，即使可能处理前的文本更长。这样子的话，有些更长的上下文信息，模型就是学习不到的。

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-11705e78badc32f68f942acd02578295_720w.jpg)

于是 Segment Recurrence Mechanism 想做的就是，能不能在前一段计算完后，将它计算出的隐状态（hidden states）都保存下来，放入一个 Memory 中去，之后在当前分段计算时，**将之前存下来的隐状态和当前段的隐状态拼起来作为 Attention 机制的 K 和 V**，从而**获得更长的上下文信息**。

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-031101ce9e6374f278e8fd2182fec582_720w.jpg)

于是乎论文中 Fig1 图中

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-c1083578857ca8465a766df6a494fa9e_720w.jpg)

最左边这个一开始看很是神秘的 mem 的身份也就很明显了，就是 Segment Recurrence Mechanism 中用到的 memory，一面存放着之前 segment 的隐状态。



**Relative Positional Encoding**

------

Transformer-XL 的另一重要组件，**Relative Positional Encoding**（相对位置编码），其实很大程度上是为了解决上一个机制中位置信息表示的问题。

Transformer-XL 的另一重要组件，**Relative Positional Encoding**（相对位置编码），其实很大程度上是为了解决上一个机制中位置信息表示的问题。

这个问题是，假设在 segment1 中已经用了从 1 开始编码的绝对位置向量，那么在 segment2 中，我们该用什么样的位置编码呢，从 1 开始的绝对位置编码吗？这样的话，在复用 segment1 时，整个过程中就会有两个 1 位置，这样是会出问题的，因为模型会搞不清想让它学习的位置信息。

当然也有个做法就是从 segment1 长度 +1 开始给 segment2 加上位置编码，但这样会让位置编码表过长，而且不一定能充分学习，还有就是这样不太符合人类写作的常识，我们其实都是一段段写，不会有人会认真数我现在写到了第 1000 个字，然后第 1000 个字会和第 10 个字有什么关系，更多会关心在某一段中一个字词和其他字词的相对关系。

因此就可以用上这里提到的，**相对位置编码**，不再关心句中词的绝对信息，而是相对的，比如说两个词之间隔了多少个词这样的相对信息。Transformer-XL 中提出的相对位置编码，虽然是为了解决上面的问题，但也非常有趣，将位置信息编码分析得很透彻。

可以简单介绍一下，如何从绝对位置信息编码到相对位置信息编码的。首先，简单定义一下，**E 是词向量**，也可以把它当作主要内容承载者，**U 是绝对位置向量**，可看作绝对位置信息承载者，W 主要是用于 attention 机制 QK 的转换，于是绝对位置信息编码的注意力由下式得出：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-6a43f3e4caf33c69f00442ba732a82f9_720w.png)

乍一看，WTF，这什么鬼，其实只是简单的矩阵运算，用上些定律，简单点可当成类似下面的乘法运算：



![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-086eb756b270cb41f014adc6478d56c2_720w.png)



具体点的话就是这样：



![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-10ae3a8fb895dc80c6f9307347a070d2_720w.jpg)

这四个项也都各有各的意义，**（a）表示纯基于内容之间的寻址**，**（b）和（c）则分别是 i 位置的内容和位置信息分别相对于 j 位置的位置和内容信息进行的寻址**，**（d）则是纯基于位置之间的寻址**。于是要改进的话，就需要对后三个和位置信息相关的项进行改进。

Transformer-XL 给出的改进方案是这样：



![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-27f67c20303201bcfe8a0408055ab980_720w.png)

主要有三条改进：

- 先把有位置信息$U_{j}$的地方都替换成相对位置信息 R_ij;

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-e5fe43e43c26ec1898d0e3cbd349419f_720w.png)



- 之后将(c)和(d)里的 U_i W_q 分别替换成，u 和 v 可学习向量；

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-d30a20cab4f3c02406d6ed6d61fb70bf_720w.png)



- 最后将 K 转换中的矩阵 W_k，分成两个 W_kE 和 W_kR，分别给内容向量和相对位置向量用。

这样就获得了文中的相对位置编码方法。

那么相对位置编码是不是只有这一种，并不是，这只是一种实现方式，比如现在我们就能想出另一种实现方式：



![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-e3aff7cfad06376abc3b3f1682e443a9_720w.png)



如果再借鉴一下这里的第三条改进，就可以变成，



![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-8d12494eec7293a1c7da40b1f1b566d0_720w.png)



很自由的。

**Relative Segment Encodings**

---

为了通过输入形式 **[A, SEP, B, SEP, CLS]** 来处理句子对任务，于是需要加入标识 A 句和 B 句的段信息。BERT 里面很简单，直接准备两个向量，一个加到 A 句上，一个加到 B 句上。

但当这个遇上 Segment Recurrence Mechanism 时，和位置向量一样，也出问题了。万一出现了明明不是一句，但是相同了怎么办，于是我们就需要最后一块补丁，同样准备两个向量，**s+** 和 **s-** 分别表示在一句话内和不在一句话内。

具体实现是在计算 attention 的时候加入一项：



![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-a1cbf21c1e83c5d11d966818ec0cd7be_720w.jpg)



当 i 和 j 位置在同一段里就用 s+，反之用 s-，在 attention 计算权重的时候加入额外项。



### 2.8 ALBert

[论文地址](https://openreview.net/pdf?id=H1eA7AEtvS) 

[GitHub（非官方中文预训练ALBERT](https://github.com/brightmart/albert_zh) 

[参考地址](https://blog.csdn.net/sinat_25394043/article/details/104262455?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-11.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-11.control)



**Intro**

---

最近各种大体量的预训练模型层出不穷，经常是一个出来刷榜没几天，另外一个又出现了。BERT、RoBERTa、XLNET等等都是代表人物。这些“BERT”们虽然一个比一个效果好，但是他们的体量都是非常大的，懂不懂就几千万几个亿的参数量，而且训练也非常困难。新出的ALBERT就是为了解决模型参数量大以及训练时间过长的问题。**ALBERT最小的参数只有十几M**, 效果要比BERT低1-2个点，最大的xxlarge也就200多M。可以看到在**模型参数量上减少的还是非常明显的**，但是在速度上似乎没有那么明显。最大的问题就是这种方式其实**并没有减少计算量**，也就是**受推理时间并没有减少**，训练时间的减少也有待商榷。



**Factorized embedding parameterization**

---

原始的BERT模型以及各种依据transformer来搞的预训练语言模型在输入的地方我们会发现它的E是等于H的，其中E就是embedding size，H就是hidden size，也就是transformer的输入输出维度。这就会导致一个问题，当我们的hidden size提升的时候，embedding size也需要提升，这就会导致我们的embedding matrix维度的提升。所以这里作者**将E和H进行了解绑**，具体的操作其实就是**在embedding后面加入一个矩阵进行维度变换**。E是永远不变的，后面H提高了后，我们在E的后面进行一个升维操作，让E达到H的维度。这使得embedding参数的维度从O(V×H)到了O(V×E + E×H), 当E远远小于H的时候更加明显。



**Cross-layer parameter sharing**

---

**跨层参数共享，就是不管12层还是24层都只用一个transformer**。之前transformer的每一层参数都是独立的，包括self-attention 和全连接，这样的话当层数增加的时候，参数就会很明显的上升。之前有工作试过单独的将self-attention或者全连接进行共享，都取得了一些效果。这里作者尝试将所有的参数进行共享，这其实就导致**多层的attention其实就是一层attention的叠加**。



**Sentence Order Prediction（SOP）**

---

这里作者使用了一个新的loss，其实就是更改了原来BERT的一个子任务NSP, 原来NSP就是来预测下一个句子的，也就是一个句子是不是另一个句子的下一个句子。这个任务的问题出在训练数据上面，正例就是用的一个文档里面连续的两句话，但是负例使用的是不同文档里面的两句话。这就导致这个任务包含了主题预测在里面，而主题预测又要比两句话连续性的预测简单太多。新的方法使用了sentence-order prediction(SOP), 正例的构建和NSP是一样的，不过负例则是将两句话反过来。实验的结果也证明这种方式要比之前好很多。但是这个这里应该不是首创了，百度的ERNIE貌似也采用了一个这种的。

**SOP 目标补偿了一部分因为 embedding 和 FFN 共享而损失的性能**。Bert 原版的 NSP 目标过于简单了，它把”topic prediction”和“coherence prediction”融合了起来。SOP 对其加强，将负样本换成了同一篇文章中的两个逆序的句子，进而消除“topic prediction”。

### 2.9Electra:Efficiently Learning an Encoder that Classifies Token Replacements Accurately
[论文地址](https://openreview.net/pdf?id=r1xMH1BtvB)

[参考地址](https://blog.csdn.net/sinat_25394043/article/details/104262455?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-11.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-11.control)

**Inro**

---

ELECTRA的全称是Efficiently Learning an Encoder that Classifies Token Replacements Accurately，先来直观感受一下ELECTRA的效果：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/ELECTRA.png)

ELECTRA最主要的贡献是提出了新的预训练任务和框架，把**生成式的Masked language model(MLM)**预训练任务改成了**判别式的Replaced token detection(RTD)任务**，判断当前token是否被语言模型替换过。

作者提出的预训练模型如下图所示，包括了一个**Generator**和一个**Discriminator**，在预训练模型的训练中，需要训练两个神经网络。一个生成器G，一个分类器D，**对于生成网络G，对于给定的输入x，生成网络在t位置按照下述方式生成输出token的xt**。其中e是token对应的embedding。

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/ELECTRA_MODEL.png)

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/20200211144312107.png)

而**对于给定位置t的token，鉴别器需要去鉴别这个单词是否经过替换**。

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/20200211144333591.png)

生成器模型G会扮演一个MLM模型，对于给定的token输入x=[x1,x2,x3..xn]，它会选择一些位置，并且将token替换成[mask]，然后生成器会开始学习预测这个masked的元素。并且生成自己的预测结果，并且生成句子。

但上述结构有个问题，输入句子经过生成器，输出改写过的句子，**因为句子的字词是离散的，所以梯度在这里就断了**，判别器的梯度无法传给生成器，于是生成器的训练目标还是MLM（作者在后文也验证了这种方法更好），判别器的目标是序列标注（判断每个token是真是假），两者同时训练，但判别器的梯度不会传给生成器，目标函数如下：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/2020021114441969.png)

除了上述的基本模型以外，作者在训练中还增加了下述的方法：

**Weight Sharing**

生成器和判别器的权重共享是否可以提升效果呢？作者设置了相同大小的生成器和判别器，在不共享权重下的效果是83.6，只共享token embedding层的效果是84.3，共享所有权重的效果是84.4。作者认为生成器对embedding有更好的学习能力，因为在计算MLM时，softmax是建立在所有vocab上的，之后反向传播时会更新所有embedding，而判别器只会更新输入的token embedding。最后作者只使用了embedding sharing。

**Smaller Generators**

从权重共享的实验中看到，生成器和判别器只需要共享embedding的权重就足矣了，那这样的话是否可以缩小生成器的尺寸进行训练效率提升呢？作者在保持原有hidden size的设置下减少了层数，得到了下图所示的关系图：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/generator_size.png)

可以看到，生成器的大小在判别器的1/4到1/2之间效果是最好的。作者认为原因是过强的生成器会增大判别器的难度（判别器：小一点吧，我太难了）。

训练算法：除了文中上述的两种方法，作者还实验了另外两种训练算法：

1. Adversarial Contrastive Estimation：ELECTRA因为上述一些问题无法使用GAN，但也可以以一种对抗学习的思想来训练。作者将生成器的目标函数由最小化MLM loss换成了最大化判别器在被替换token上的RTD loss。但还有一个问题，就是新的生成器loss无法用梯度上升更新生成器，于是作者用强化学习Policy Gradient的思想，最终优化下来生成器在MLM任务上可以达到54%的准确率，而之前MLE优化下可以达到65%。

2. Two-stage training：即先训练生成器，然后freeze掉，用生成器的权重初始化判别器，再接着训练相同步数的判别器。



### 2.10 DistillBert:**a distilled version of BERT: smaller, faster, cheaper and lighter** 

[论文地址]( https://arxiv.org/abs/1909.10351)

[参考地址](https://blog.csdn.net/sinat_25394043/article/details/104263160?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-4.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-4.control)



从应用落地的角度来说，bert虽然效果好，但有一个短板就是预训练模型太大，预测时间在平均在300ms以上（一条数据），无法满足业务需求。知识蒸馏是在较低成本下有效提升预测速度的方法。

关于知识蒸馏可以看下这篇[文章]( https://blog.csdn.net/nature553863/article/details/80568658)。

DistillBert是在bert的基础上用知识蒸馏技术训练出来的小型化bert。整体上来说这篇论文还是非常简单的，只是引入了知识蒸馏技术来训练一个小的bert。具体做法如下：

1. 给定原始的bert-base作为teacher网络。

2. 在bert-base的基础上将网络层数减半（也就是从原来的12层减少到6层）。

3. 利用teacher的软标签和teacher的隐层参数来训练student网络。


训练时的损失函数定义为三种损失函数的线性和，三种损失函数分别为：

1. $L_{ce}$ 这是teacher网络softmax层输出的概率分布和student网络softmax层输出的概率分布的交叉熵（注：MLM任务的输出）。

2. $L_{mlm}$ 这是student网络softmax层输出的概率分布和真实的one-hot标签的交叉熵

3. $L_{cos}$ 这是student网络隐层输出和teacher网络隐层输出的余弦相似度值，在上面我们说student的网络层数只有6层，teacher网络的层数有12层，因此个人认为这里在计算该损失的时候是用student的第1层对应teacher的第2层，student的第2层对应teacher的第4层，以此类推。


整体计算公式为：

Loss= 5.0\*Lce+2.0\* Lmlm+1.0\* Lcos 



![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi1lNmU2MWM4YjUzMGVhNjE3ZjUyNTE2MzMzZGNhN2E1M19oZC5qcGc?x-oss-process=image/format,png)

### 2.11 TinyBert:: DISTILLING BERT FOR NATURAL LANGUAGE UNDERSTANDING

论文地址 https://arxiv.org/abs/1909.10351

[参考地址](https://blog.csdn.net/sinat_25394043/article/details/104263160?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-4.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-4.control)

TinyBert也是采用了知识蒸馏的方法来压缩模型的，只是在设计上较distillBert做了更多的工作，作者提出了两个点：针对Transformer结构的知识蒸馏和针对pre-training和fine-tuning两阶段的知识蒸馏。

作者在这里构造了四类损失函数来对模型中各层的参数进行约束来训练模型，具体模型结构如下：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/TinyBert_overview.png)

 作者构造了四类损失，分别针对embedding layer，attention 权重矩阵，隐层输出，predict layer。可以将这个统一到一个损失函数中：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/20200211153129504.png)

针对上面四层具体的损失函数表达式如下：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/20200211153145261.png)

　　以上四种损失函数是作者针对transformer提出的知识蒸馏方法。除此之外作者认为除了对pre-training蒸馏之外，在fine-tuning时也利用teacher的知识来训练模型可以取得在下游任务更好的效果。因此作者提出了两阶段知识蒸馏，如下图所示：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/TinyBert_illustration.png)

 　　本质上就是在pre-training蒸馏一个general TinyBERT，然后再在general TinyBERT的基础上利用task-bert上再蒸馏出fine-tuned TinyBERT。
## 3. 个人想法

将ERNIE的实体MASK和XLNet的思想结合起来，对实体层面进行操作，