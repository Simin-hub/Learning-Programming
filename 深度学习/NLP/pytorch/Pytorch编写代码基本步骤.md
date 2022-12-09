# Pytorch编写代码基本步骤

[参考地址](https://zhuanlan.zhihu.com/p/149579648)

## **一、前言**

在我们要用pytorch构建自己的深度学习模型的时候，基本上都是下面这个流程步骤，写在这里让一些新手童鞋学习的时候有一个大局感觉，无论是从自己写，还是阅读他人代码，按照这个步骤思想（默念4大步骤，找数据定义、找model定义、(找损失函数、优化器定义)，主循环代码逻辑），直接去找对应的代码块，会简单很多。

## **二、基本步骤思想**

所有的深度学习模型过程都可以形式化如下图：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-0c2366935331fa521a2a7eef2bb34a9f_720w.jpg?token=AKS6BK7JEQC477UL45PSWLLBUNUHM)

分为四大步骤：

**1、输入处理模块** (X 输入数据，变成网络能够处理的Tensor类型)

**2、模型构建模块** (主要负责从输入的数据，得到预测的y^, 这就是我们经常说的前向过程)

**3、定义代价函数和优化器模块** (注意，前向过程只会得到模型预测的结果，并不会自动求导和更新，是由这个模块进行处理)

**4、构建训练过程 （**迭代训练过程，就是上图表情包的训练迭代过程**）**

这几个模块分别与上图的数字标号1，2，3，4进行一一对应！

## **三、实例讲解**

知道了上面的宏观思想之后，后面给出每个模块稍微具体一点的解释和具体一个例子，再帮助大家熟悉对应的代码！

**1.数据处理**

对于数据处理，最为简单的⽅式就是将数据组织成为⼀个 。但许多训练需要⽤到mini-batch，直 接组织成Tensor不便于我们操作。pytorch为我们提供了Dataset和Dataloader两个类来方便的构建。

**torch.utils.data.Dataset**

继承Dataset 类需要override 以下⽅法：

![img](https://pic1.zhimg.com/80/v2-8e72fa62fcf8e616f4363926a77d9890_720w.jpg)

**torch.utils.data.DataLoader**

```python3
torch.utils.data.DataLoader(dataset, batch_size=1, shuffle=False)
```

DataLoader Batch。如果选择shuffle = True，每⼀个epoch 后，mini-Batch batch_size 常⻅的使⽤⽅法如下：

![img](https://pic3.zhimg.com/80/v2-f39282982dd78edb57180c6c1bb6590a_720w.jpg)

**2. 模型构建**

所有的模型都需要继承torch.nn.Module ， 需要实现以下⽅法：

![img](https://pic2.zhimg.com/80/v2-115e01def4e0bcecd4fd83a32c3cdab9_720w.jpg)

其中forward() ⽅法是前向传播的过程。在实现模型时，我们不需要考虑反向传播。

**3. 定义代价函数和优化器**

![img](https://pic2.zhimg.com/80/v2-2c694c6cb91757a45b3e47d2bc83a141_720w.jpg)

这部分根据⾃⼰的需求去参照doc

**4、构建训练过程**

pytorch的训练循环⼤致如下：

![img](https://pic2.zhimg.com/80/v2-0ef6151ec640449398f199a19b61d6ad_720w.jpg)

下面再用一个简单例子，来巩固一下：

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-5db27e7adb9b18d58008cf4c08b812f9_720w.jpg?token=AKS6BK73ZCEZ3DSRJJNKNJTBUNUH4)

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-0c50e64873e8b57e95ee1a02f922b007_720w.jpg?token=AKS6BK6JKATG73FHOVRINTTBUNUIG)![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-aa56b65ff2ee5d6ee4ac402b913ec9cc_720w.jpg?token=AKS6BK6CL6C2CPVVBFD6HFTBUNUIM)

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-f66acb878392d68ac9ae383bbdc51705_720w.jpg?token=AKS6BKZTM4YTO4FNCN6MNPLBUNUIS)

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/v2-05912fdd7d10a2ec7265291565af2e9b_720w.jpg?token=AKS6BK72HAKVRNIC7TEVKCDBUNUI2)