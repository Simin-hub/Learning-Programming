# Dataloader

[参考地址](https://zhuanlan.zhihu.com/p/270028097)

## 1. 源码解析

PyTorch的数据加载模块，一共涉及到Dataset，Sampler，Dataloader三个类

**Dataset**负责对raw data source封装，将其封装成Python可识别的数据结构，其必须提供提取数据个体的接口。Dataset共有Map-style datasets和Iterable-style datasets两种：

- **map-style dataset**：实现了__getitem__和__len__接口，表示一个从索引/key到样本数据的map。比如：datasets[10]，就表示第10个样本。
- **iterable-style dataset**：实现了__iter__接口，表示在data samples上的一个Iterable（可迭代对象），这种形式的dataset非常不适合随机存取（代价太高），但非常适合处理流数据。比如：iter(datasets)获得迭代器，然后不断使用next迭代从而实现遍历。

**Sampler**负责提供一种遍历数据集所有**元素索引**的方式。

**Dataloader**负责加载数据，同时支持map-style和iterable-style Dataset，支持单进程/多进程，还可以设置loading order, batch size, pin memory等加载参数。

这三者的关系就一目了然了。

1. 设置Dataset，将数据data source包装成Dataset类，暴露提取接口。
2. 设置Sampler，决定采样方式。我们是能从Dataset中提取元素了，还是需要设置Sampler告诉程序提取Dataset的策略。
3. 将设置好的Dataset和Sampler传入DataLoader，同时可以设置shuffle，batch_size等参数。使用DataLoader对象可以快捷方便地在给定数据集上遍历。

总结来说，即Dataloader负责总的调度，命令Sampler定义遍历索引的方式，然后用索引去Dataset中提取元素。于是就实现了对给定数据集的遍历。

### 1.1 Dataset

**torch.utils.data.Dataset**：抽象基类

- 所有的Dataset相关类都应该继承自torch.utils.data.Dataset这个类
- 这些子类**必须要实现方法__getitem__()**，来支持可以给定一个key（即索引）来获取对应的数据样本
- 这些类可以实现方法__len__()，来返回数据集的大小规模

Dataset实现非常简洁，就只是提供了__getitem__ 和 __add__这两个接口

具体实践中，我们需要使用Dataset的子类，**自己实现的或者现成的**。

我们可以来看看PyTorch为我们提供的现成的Dataset子类：

- TensorDataset
- IterableDataset
- ConcatDataset
- ChainDataset
- Subset

下面着重介绍**TensorDataset**和**IterableDataset**.

**CLASS torch.utils.data.TensorDataset(\*tensors)**

包装了Tensor的Dataset子类，map-style dataset

每个样本可以通过tensors第一个维度的索引获取

**CLASS torch.utils.data.IterableDataset**

内部样本的组织形式是Iterable的所有dataset类都是IterableDataset类的子类，即：**所有iterable-style dataset都是IterableDataset的子类。**

这种形式的dataset对于处理流数据是非常有用的。

所有这些子类需要实现__iter__方法（而不是__getitem__方法了），需要据此来返回样本的迭代器，从而遍历dataset（实际代码中常使用iter+next来遍历）

### 1.2 Sampler

**CLASS torch.utils.data.Sampler(data_source: Optional[collections.abc.Sized])**

所有Samplers的基类

Sampler的所有子类都需要实现__iter__，用来提供遍历dataset索引的方式。我们获得不同的索引遍历，就能以不同的方式遍历dataset，这就是samplers的目的。

PyTorch为我们提供了几种现成的Sampler子类：

- SequentialSampler
- RandomSampler
- SubsetRandomSampler
- WeightedRandomSampler
- BatchSampler
- DistributedSampler

下面我着重介绍一下**SequentialSampler，RandomSampler**和**BatchSampler**

**CLASS SequentialSampler(Sampler[int])**

SequentialSampler指定总是按照相同的次序，顺序地采样元素

**CLASS torch.utils.data.RandomSampler**

RandomSampler提供了随机采样元素的方式。

如果replacement\==False，则随机采样整个数据集，即num_samples==len(dataset)。此时sampler提供给dataloader以一种随机的次序遍历dataset.

如果replacement==True，则从数据集中随机采样num_samples个样本

**CLASS torch.utils.data.BatchSampler**

BatchSampler包装另一个sampler（输入参数），用来产生一个mini-batch大小的索引，相当于是为dataloader提供了提取dataset的1个mini-batch样本的索引。

### 1.3 DataLoader

铺垫了这么多，终于讲到DataLoader了。

在训练/测试深度学习网络的程序中，我们直接遍历Dataloader来获取数据（data，label等），并将数据feed给网络用于前向传播和反向传播。

```
batch_size = 32
# train_token_ids必须是tensor类型
train_dataset = TensorDataset(train_token_ids, train_attention_masks, train_labels)
train_sampler = RandomSampler(train_dataset)
train_dataloader = DataLoader(train_dataset, sampler=train_sampler, batch_size=batch_size)
```

1. 加载数据，提取出feature和label，并转换成tensor
2. 传入`TensorDataset`中，实例化`TensorDataset`为datsset
3. 再将dataset传入到Dataloader中，最后通过enumerate输出我们想要的经过shuffle的bachsize大小的feature和label数据