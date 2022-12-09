# ResNet

[参考地址](https://www.zhihu.com/search?type=content&q=%E6%AE%8B%E5%B7%AE%E5%B1%82%E7%9A%84%E4%BD%9C%E7%94%A8)

## **残差网络的由来**

残差操作这一思想起源于论文《Deep Residual Learning for Image Recognition》，目前的引用量已达3万多。这篇文章发现，**如果存在某个![[公式]](https://www.zhihu.com/equation?tex=K)层的网络![[公式]](https://www.zhihu.com/equation?tex=f)是当前最优的网络，那么可以构造一个更深的网络，其最后几层仅是该网络![[公式]](https://www.zhihu.com/equation?tex=f)第![[公式]](https://www.zhihu.com/equation?tex=K)层输出的恒等映射(Identity Mapping)，就可以取得与![[公式]](https://www.zhihu.com/equation?tex=f)一致的结果；也许![[公式]](https://www.zhihu.com/equation?tex=K)还不是所谓“最佳层数”，那么更深的网络就可以取得更好的结果**。总而言之，与浅层网络相比，更深的网络的表现不应该更差。但是如下图所示，56层的神经网络表现明显要比20层的差。证明更深的网络在训练过程中的难度更大，因此作者提出了残差网络的思想。![img](https://pic2.zhimg.com/v2-a16f6011fbae00e134f3d8cdb623e5f5_b.jpg)

## 残差网络的定义

残差网络依旧让非线形层满足 ![[公式]](https://www.zhihu.com/equation?tex=H%5Cleft%28x%2C+w_%7Bh%7D%5Cright%29) ，然后从输入直接引入一个短连接到非线形层的输出上，使得整个映射变为

![[公式]](https://www.zhihu.com/equation?tex=%5Cmathrm%7By%7D%3DH%5Cleft%28x%2C+w_%7Bh%7D%5Cright%29%2Bx)

这就是残差网路的核心公式，换句话说，**残差是网络搭建的一种操作**，任何使用了这种操作的网络都可以称之为残差网络。

一个具体的残差模块的定义如下图：

![img](https://pic3.zhimg.com/v2-39dabdf70ac19c0c92b86d53375d8c06_b.jpg)残差模块（由于先敲公式后引得图，容易混淆，图中的F(x)就是上文所说的H(x,w)，下面也一样替换)

## 残差网络的优势

残差模块为什么有效，有很多的解释，这里提供两个方面的理解，**一方面是残差网络更好的拟合分类函数以获得更高的分类精度**，**另一方面是残差网络如何解决网络在层数加深时优化训练上的难题**。

**1.残差网络拟合函数的优越性**

首先从**万能近似定理**（Universal Approximation Theorem）入手。这个定理表明，一个前馈神经网络（feedforward neural network）如果具有线性输出层，同时至少存在一层具有任何一种“挤压”性质的激活函数（例如logistic sigmoid激活函数）的隐藏层，那么只要给予这个网络足够数量的隐藏单元，它就可以以任意的精度来近似任何从一个有限维空间到另一个有限维空间的波莱尔可测函数(Boral Measurable Function)。

万能近似定理意味着我们在构建网络来学习什么函数的时候，我们知道一定存在一个多层感知机（Multilayer Perceptron Model，MLP）能够表示这个函数。然而，我们不能保证训练算法能够学得这个函数。因为即使多层感知机能够表示该函数，学习也可能会失败，可能的原因有两种。

（1）用于训练的优化算法可能找不到用于期望函数的参数值。

（2）训练算法可能由于过拟合而选择了错误的函数。

第二种过拟合情况不在我们的讨论范围之内，因此我们聚焦在前一种情况，为何残差网络相比简单的多层网络能更好的拟合分类函数，即找到期望函数的参数值。

对于普通的不带短连接的神经网络来说，存在这样一个命题。

命题1:假设 ![[公式]](https://www.zhihu.com/equation?tex=f_%7B%5Cmathcal%7BN%7D%7D%3A+%5Cmathbb%7BR%7D%5E%7Bd%7D+%5Crightarrow+%5Cmathbb%7BR%7D) 为普通的带激活函数的全连接网络 ![[公式]](https://www.zhihu.com/equation?tex=N) 。 

​														![[公式]](https://www.zhihu.com/equation?tex=%5Cmathrm%7BP%7D%3D%5C%7B%5Cmathrm%7Bx%7D+%5Cin+%5Cleft.%5Cmathbb%7BR%7D%5E%7Bd%7D+%7C+f_%7B%5Cmathcal%7BN%7D%7D%28x%29%3E0%5Cright%5C%7D) 

为 ![[公式]](https://www.zhihu.com/equation?tex=f_N) 的正等值面，假如 ![[公式]](https://www.zhihu.com/equation?tex=N) 的每个层的激活函数都至多只有![[公式]](https://www.zhihu.com/equation?tex=d) 个神经元，那么

![[公式]](https://www.zhihu.com/equation?tex=%5Clambda%28P%29%3D0+%5Ctext+%7B+or+%7D+%5Clambda%28P%29%3D%2B%5Cinfty)

![[公式]](https://www.zhihu.com/equation?tex=%5Clambda) 为勒贝格测度。换句话说，这样狭窄的全连接网络表示的函数要么没有边界约束，要么恒为0。因此，即使层数无限加深，整个网络的表现力也受网络的宽度限制而无法近似一个带边界的区域。而对于残差网络来讲，拟合函数的能力则完全不受网路宽度的影响，上述命题1对于残差网络并不适用。

下面从一个简单的二维例子来说明这一点，这样可以进行方便的可视化。我们随机生成一组测试点 ![[公式]](https://www.zhihu.com/equation?tex=%5Cleft%28z_%7Bi%7D%2C+y_%7Bi%7D%5Cright%29_%7Bi%3D1+%5Cldots+n%7D+%5Cin+%5Cmathbb%7BR%7D%5E%7B2%7D+%5Ctimes%5C%7B-1%2C1%5C%7D) ，满足

![[公式]](https://www.zhihu.com/equation?tex=y_%7Bi%7D%3D%5Cleft%5C%7B%5Cbegin%7Barray%7D%7Bc%7D%7B1%2C+%5Ctext+%7B+if+%7D%5Cleft%5C%7Cz_%7Bi%7D%5Cright%5C%7C_%7B2%7D+%5Cleq+1%7D+%5C%5C+%7B-1%2C+i+f+%5Cleq+2%5Cleft%5C%7Cz_%7Bi%7D%5Cright%5C%7C_%7B2%7D+%5Cleq+3%7D%5Cend%7Barray%7D%5Cright.)

我们手动构造一个清晰的分类边界使得整个任务更容易一点，损失函数采用逻辑回归损失![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B1%7D%7Bn%7D+%5Csum+%5Clog+%5Cleft%281%2Be%5E%7B-y_%7Bi%7D+%5Chat%7By_%7B%5Cmathrm%7Bt%7D%7D%7D%7D%5Cright%29) ，其中 ![[公式]](https://www.zhihu.com/equation?tex=%5Chat%7By%7D_%7B%5Ciota%7D%3Df_%7B%5Cmathcal%7BN%7D%7D%5Cleft%28z_%7Bi%7D%5Cright%29) 为网络对于样本 ![[公式]](https://www.zhihu.com/equation?tex=i) 的实际输出。经过训练后，分析网络不同深度下得到的训练边界，如图可以发现宽度比输入维度小的残差网络的训练边界明显更加接近真实边界，也不受命题1的限制。

![img](https://pic3.zhimg.com/v2-8160e22f45869596f18a1692df9d7ef2_b.jpg)

不同网络结构拟合函数边界的结果。左上角为函数的真实边界。第一行是简单的全连接网络，每层的神经元个数为2；第二行为带短连接的网络，每层神经元个数为1。从左到右的网络层数依次递增，分别为1-5层。

事实上对于高维函数，这一特点依然适用。因此，当函数的输入维度非常高时，这一做法就变的非常有意义。尽管在高维空间这一特点很难被可视化，但是**这个理论给了一个很合理的启发，就是原则上，带短连接的网络的拟合高维函数的能力比普通连接的网络更强**。这部分我们讨论了残差网络有能力拟合更高维的函数，但是在实际的训练过程中仍然可能存在各种各样的问题使得学习到最优的参数非常困难，因此下一小节讨论残差在训练过程中的优越性。

**2.残差网络训练过程的优越性**

这个部分我们讨论为什么残差能够缓解深层网络的训练问题，以及探讨可能的短连接方式和我们最终选择的残差的理由。正如本章第三部分讨论的一样，整个残差卷积神经网络是由以上的残差卷积子模块堆积而成。如上一小节所定义的，假设第 ![[公式]](https://www.zhihu.com/equation?tex=l) 层的残差卷积字子模块的映射为

![[公式]](https://www.zhihu.com/equation?tex=F%5Cleft%28x_%7Bl%7D%2C+w_%7Bf%7D%5Cright%29%3Dx_%7Bl%7D%2BH%5Cleft%28x_%7Bl%7D%2C+w_%7Bl%7D%5Cright%29)

![[公式]](https://www.zhihu.com/equation?tex=x_l) 是第 ![[公式]](https://www.zhihu.com/equation?tex=l) 层的输入， ![[公式]](https://www.zhihu.com/equation?tex=w_%7Bl%7D%3D%5Cleft%5C%7Bw_%7Bl%2C+k%7D+%7C+1+%5Cleq+k+%5Cleq+K%5Cright%5C%7D) 是第 ![[公式]](https://www.zhihu.com/equation?tex=l) 层的参数， ![[公式]](https://www.zhihu.com/equation?tex=K) 是残差单元层数。

那么第 ![[公式]](https://www.zhihu.com/equation?tex=l%2B1) 层的输入为

![[公式]](https://www.zhihu.com/equation?tex=x_%7Bl%2B1%7D%3DF%5Cleft%28x_%7Bl%7D%2C+w_%7Bf%7D%5Cright%29)

因此得到

![[公式]](https://www.zhihu.com/equation?tex=x_%7Bl%2B1%7D%3Dx_%7Bl%7D%2BH%5Cleft%28x_%7Bl%7D%2C+w_%7Bl%7D%5Cright%29)

循环带入这个式子 ![[公式]](https://www.zhihu.com/equation?tex=x_%7Bl%2B2%7D%3Dx_%7Bl%2B1%7D%2BH%5Cleft%28x_%7Bl%2B1%7D%2C+w_%7Bl%2B1%7D%5Cright%29%3Dx_%7Bl%7D%2BH%5Cleft%28x_%7Bl%7D%2C+w_%7Bl%7D%5Cright%29%2BH%5Cleft%28x_%7Bl%2B1%7D%2C+w_%7Bl%2B1%7D%5Cright%29) ，我们可以得到

![[公式]](https://www.zhihu.com/equation?tex=x_%7BL%7D%3Dx_%7Bl%7D%2B%5Csum_%7Bi%3Dl%7D%5E%7BL-1%7D+F%5Cleft%28x_%7Bi%7D%2C+w_%7Bi%7D%5Cright%29) （1）

对于任何深度的L来讲，上述式子（1）显示了一些良好的特性。

（1）第![[公式]](https://www.zhihu.com/equation?tex=L)层的特征![[公式]](https://www.zhihu.com/equation?tex=x_L)可以分为两个部分，第一部分是浅层的网络表示![[公式]](https://www.zhihu.com/equation?tex=x_l)加上一个残差函数映射 ![[公式]](https://www.zhihu.com/equation?tex=%5Csum_%7Bi%3Dl%7D%5E%7BL%3D1%7D+F%5Cleft%28x_%7Bi%7D%2C+w_%7Bi%7D%5Cright%29) ，表明模型在任意单元内都是一个残差的形式。

（2）对于任意深度 ![[公式]](https://www.zhihu.com/equation?tex=L) 的特征 ![[公式]](https://www.zhihu.com/equation?tex=x_L) 来讲，它是前面所有残差模块的和，这与简单的不加短连接的网络完全相反。原因是，不加短连接的网络在第 ![[公式]](https://www.zhihu.com/equation?tex=L) 层的特征 ![[公式]](https://www.zhihu.com/equation?tex=x_L) 是一系列的向量乘的结果，即 ![[公式]](https://www.zhihu.com/equation?tex=%5CPi_%7Bi%3D0%7D%5E%7BL-1%7D+W_%7Bi%7D+x_%7B0%7D) (在忽略batch normalization和激活函数的情况下)。

同样，上述式子显示有非常好的反向传播特性，假设损失为 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvarepsilon) ，根据链式求导法则，我们可以得到

![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_%7Bl%7D%7D%3D%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_%7BL%7D%7D+%5Cfrac%7B%5Cpartial+x_%7BL%7D%7D%7B%5Cpartial+x_%7Bl%7D%7D%3D%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_%7BL%7D%7D%5Cleft%281%2B%5Cfrac%7B%5Cpartial%7D%7B%5Cpartial+x_%7Bl%7D%7D+%5Csum_%7Bi%3Dl%7D%5E%7BL-1%7D+F%5Cleft%28x_%7Bi%7D%2C+w_%7Bi%7D%5Cright%29%5Cright%29) （2）

显示梯度![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_%7BL%7D%7D)由两个部分组成，一部分 ![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+x_%7Bl%7D%7D%7B%5Cpartial+x_%7BL%7D%7D) 是不用经过任何权重加权的信息流，另一部分是通过加权层的 ![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_%7BL%7D%7D%5Cleft%28%5Cfrac%7B%5Cpartial%7D%7B%5Cpartial+x_%7Bl%7D%7D+%5Csum_%7Bi%3Dl%7D%5E%7BL-1%7D+F%5Cleft%28x_%7Bi%7D%2C+w_%7Bi%7D%5Cright%29%5Cright%29) ，两部分连接的线形特性保证了信息可以直接反向传播到浅层。同时式子还说明对于小的batch而言，梯度 ![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_%7Bl%7D%7D) 不太可能会消失，因为通常 对于小的batch来讲不会总是为1，那么这表示即使权重非常小，梯度也不会为0，不存在梯度消失的问题。

总之，式子（1）和（2）表明信号**无论是在前向传播还是反向传播的过程中，都是可以直接通过的**。

