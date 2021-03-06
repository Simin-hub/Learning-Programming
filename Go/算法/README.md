# 算法

主要是应对面试准备的算法。

[参考](https://algo.itcharge.cn/)、[参考](https://greyireland.gitbook.io/algorithm-pattern/)

## 数据结构

### 数组

### 链表

### 队列

### 栈

### 哈希表

### 字符串

### 树

- #### [二叉树](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E7%AE%97%E6%B3%95/%E4%BA%8C%E5%8F%89%E6%A0%91.md)

- #### [二叉搜索树](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E7%AE%97%E6%B3%95/%E4%BA%8C%E5%8F%89%E6%A0%91.md)

- #### [线段树](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E7%AE%97%E6%B3%95/%E7%BA%BF%E6%AE%B5%E6%A0%91.md)

- #### 树状数组

- #### [并查集](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E7%AE%97%E6%B3%95/%E5%B9%B6%E6%9F%A5%E9%9B%86.md)

### 图



## 基础算法

### [排序算法](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95.md)

排序算法比较

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/68747470733a2f2f696d67323031382e636e626c6f67732e636f6d2f626c6f672f3834393538392f3230313930332f3834393538392d32303139303330363136353235383937302d313738393836303534302e706e67)

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/68747470733a2f2f696d61676573323031382e636e626c6f67732e636f6d2f626c6f672f3834393538392f3230313830342f3834393538392d32303138303430323133333433383231392d313934363133323139322e706e67)

### 二分查找

[参考](https://algo.itcharge.cn/01.Array/03.Array-Binary-Search/01.Array-Binary-Search/)

二分查找（Binary Search）也叫作[折半查找](http://data.biancheng.net/view/55.html)。二分查找有两个要求，一个是数列有序，另一个是数列使用顺序存储结构（比如[数组](http://data.biancheng.net/view/181.html)）。

#### 二分查找细节

真正在解决二分查找题目的时候还是需要考虑很多细节的。比如说以下几个问题：

1. **区间的开闭问题**：区间应该是左闭右闭，还是左闭右开？
2. `mid` 的取值问题：`mid = (left + right) // 2`，还是 `mid = (left + right + 1) // 2`？
3. 出界条件的判断：`left <= right`，还是 `left < right`？
4. 搜索区间范围的选择：`left = mid + 1`、`right = mid - 1`、 `left = mid `、`right = mid` 应该怎么写？



### 双指针

[参考](https://algo.itcharge.cn/01.Array/04.Array-Two-Pointers/01.Array-Two-Pointers)

#### 双指针简介

> 双指针（Two Pointers）：指的是在遍历元素的过程中，不是使用单个指针进行访问，而是使用两个指针进行访问，从而达到相应的目的。**如果两个指针方向相反，则称为「对撞时针」。如果两个指针方向相同，则称为「快慢指针」。如果两个指针分别属于不同的数组 / 链表，则称为「分离双指针」**。

在数组的区间问题上，暴力算法的时间复杂度往往是 O(n^2)。而双指针利用了区间「单调性」的性质，从而将时间复杂度降到了 o(n)。

#### 双指针总结 

双指针分为「对撞指针」、「快慢指针」、「分离双指针」。

- 对撞指针：两个指针方向相反**。适合解决查找有序数组中满足某些约束条件的一组元素问题、字符串反转问题**。
- 快慢指针：两个指针方向相同。**适合解决数组中的移动、删除元素问题，或者链表中的判断是否有环、长度问题**。
- 分离双指针：两个指针分别属于不同的数组 / 链表。**适合解决有序数组合并，求交集、并集问题**。

### 滑动窗口

[地址](https://algo.itcharge.cn/01.Array/05.Array-Sliding-Window/01.Array-Sliding-Window/)

#### 1. 滑动窗口算法介绍

> 滑动窗口（Sliding Window）：在给定数组 / 字符串上维护一个固定长度或不定长度的窗口。可以对窗口进行滑动操作、缩放操作，以及维护最优解操作。

- 滑动操作：窗口可按照一定方向进行移动。最常见的是向右侧移动。
- 缩放操作：对于不定长度的窗口，可以从左侧缩小窗口长度，也可以从右侧增大窗口长度。

**滑动窗口利用了双指针中的快慢指针技巧**，我们可以将滑动窗口看做是快慢指针两个指针中间的区间，也可以将滑动窗口看做是快慢指针的一种特殊形式。

#### 

### 枚举算法

 [原地址](https://algo.itcharge.cn/09.Algorithm-Base/01.Enumeration-Algorithm/01.Enumeration-Algorithm/#枚举算法知识)

#### 1. 枚举算法简介

> **枚举算法（Enumeration Algorithm）**：也称为穷举算法，指的是按照问题本身的性质，一一列举出该问题所有可能的解，并在逐一列举的过程中，将它们逐一与目标状态进行比较以得出满足问题要求的解。在列举的过程中，既不能遗漏也不能重复。

枚举算法是设计最简单、最基本的搜索算法，其核心思想是通过列举问题的所有状态，将它们逐一与目标状态进行比较，从而得到满足条件的解。

- 枚举算法的优点：
  - 容易编程实现，也容易调试。
  - 建立在考察大量状态、甚至是穷举所有状态的基础上，所以算法的正确性比较容易证明。
- 枚举算法的缺点：
  - 效率比较低，不适合求解规模较大的问题。

所以，枚举算法通常用于求解问题规模比较小的问题，或者作为求解问题的一个子算法出现，通过枚举一些信息并进行保存，而这些消息的有无对主算法效率的高低有着较大影响。

#### 2. 枚举算法解题思路 

采用枚举算法解题的一般思路如下：

1. 确定枚举对象、枚举范围和判断条件，并判断条件设立的正确性。
2. 一一枚举可能的情况，并验证是否是问题的解。
3. 考虑提高枚举算法的效率。

我们可以从下面几个方面考虑提高算法的效率：

1. 抓住问题状态的本质，尽可能缩小问题状态空间的大小。
2. 加强约束条件，缩小枚举范围。
3. 根据某些问题特有的性质，例如对称性等，避免对本质相同的状态重复求解。



### 递归算法

[原地址](https://algo.itcharge.cn/09.Algorithm-Base/02.Recursive-Algorithm/01.Recursive-Algorithm/#1-%e9%80%92%e5%bd%92%e7%ae%80%e4%bb%8b)

**递归（Recursion）**：指的是一种**通过重复将原问题分解为同类的子问题而解决的方法**。在绝大数编程语言中，可以通过在函数中再次调用函数自身的方式来实现递归。

### 分治算法

[原地址](https://algo.itcharge.cn/09.Algorithm-Base/03.Divide-And-Conquer-Algorithm/01.Divide-And-Conquer-Algorithm/#1-%e5%88%86%e6%b2%bb%e7%ae%97%e6%b3%95%e7%ae%80%e4%bb%8b)

> **分治算法（Divide and Conquer）**：字面上的解释是「分而治之」，就是把一个复杂的问题分成两个或更多的相同或相似的子问题，直到最后子问题可以简单的直接求解，原问题的解即子问题的解的合并。

简单来说，分治算法的基本思想就是： **把规模大的问题不断分解为子问题，使得问题规模减小到可以直接求解为止。**

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/20220413153059.png)

### 回溯算法

 [原地址](https://algo.itcharge.cn/09.Algorithm-Base/04.Backtracking-Algorithm/01.Backtracking-Algorithm/#1-回溯算法简介)

> **回溯算法（Backtracking）**：一种能避免不必要搜索的穷举式的搜索算法。采用试错的思想，在搜索尝试过程中寻找问题的解，当探索到某一步时，发现原先的选择并不满足求解条件，或者还需要满足更多求解条件时，就退回一步（回溯）重新选择，这种走不通就退回再走的技术称为「回溯法」，而满足回溯条件的某个状态的点称为「回溯点」。

简单来说，回溯算法采用了一种 **「走不通就回退」** 的算法思想。

回溯算法通常用简单的递归方法来实现，在进行回溯过程中更可能会出现两种情况：

1. 找到一个可能存在的正确答案；
2. 在尝试了所有可能的分布方法之后宣布该问题没有答案。

###  贪心算法

 [原地址](https://algo.itcharge.cn/09.Algorithm-Base/05.Greedy-Algorithm/01.Greedy-Algorithm/#1-贪心算法简介)

#### 贪心算法的定义

> **贪心算法（Greedy Algorithm）**：一种在每次决策时，总是采取在当前状态下的最好选择，从而希望导致结果是最好或最优的算法。

贪心算法是一种改进的「分步解决算法」，其核心思想是：将求解过程分成「若干个步骤」，然后根据题意选择一种「度量标准」，每个步骤都应用「贪心原则」，选取当前状态下「最好 / 最优选择（局部最优解）」，并以此希望最后得出的结果也是「最好 / 最优结果（全局最优解）」。

换句话说，贪心算法不从整体最优上加以考虑，而是一步一步进行，每一步只以当前情况为基础，根据某个优化测度做出局部最优选择，从而省去了为找到最优解要穷举所有可能所必须耗费的大量时间。

当然，使用贪心算法所得到的最终解并不一定就是全局最优解。但是对许多问题来说，确实可以通过局部最优解而得到整体最优解或者是整体最优解的近似解。

一般来说，这些能够使用贪心算法解决的问题必须满足下面的两个特征：「贪⼼选择性质」和「最优子结构」。



### 位运算



### 动态规划