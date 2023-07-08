---
layout: post
title: NuevoMatch	
date: 2021-04-23
tags: packet classification 
mathjax: true
---

# Nuevomatch

**RQ-RMI:**Range Query Recursive Model Index

两类包分类算法

1. 基于TCAM

2. 基于软件（TupleMerge、CutSplit、NeuroCuts、多维切割、TSS、EffiCuts、PartitionSort)

   >**决策树**
   >
   >
   >
   >**哈希策略**
   >
   >TM、TSS（PSTSS)
![](/images/avatar.jpg)

![overview](/images/NuevoMatch/Figure 1,overview.jpg)
<!-- ![image-20210423141557181](.\..\images\NuevoMatch\Figure 1,overview.jpg) -->

**NuevoMatch overview**

RMI:The Case for Learned Index Structures

> learn the mapping function betweeen the keys and the indexes of their values in the array
>
> RMI用来已知key预测index（利用CPU cache）可能是占用空间更小

RMI不能直接用于包分类的原因：

> packet field没有精确匹配一个value，对于range域需要枚举全部可能的值，导致变慢
>
> 多维度也是RMI的一个问题

RQ-RMI 	match keys to ranges  

![image-20210423200339742](\images\NuevoMatch\Figure 2 Ruleeg.jpg)

**RMI 思想**

> 传统方法使用B-tree，完成key-value，RMI试图使用机器学习模型完成key-value pairs。

**RMI结构**

>多层结构，首层一个submodel，每层最多$W_i$个submodel，submodel可以选择B-tree, regression model等多种结构完成。最后一层产生预测的value
>
>$W_i$,submodel的类型，层数都是手动配置好的。后面使用三层全连接的神经网络替换了submodel

**RMI查询过程**

> 首先根据$\hat{h}(key)$得到$j$，然后并在邻域$\epsilon$内二次查找得到预测index

**RMI流程**

> $y=h(x)$,都是从$[0,1]\rightarrow[0,1]$的映射，且一一对应，RMI使用机器学习去训练得$h$，训练结果为$\hat{h}(x)\rightarrow\hat{y}=\hat{h}(key)$以保证训练结果为$|\hat{h}(key)-h(key)\leq \epsilon|$

**RMI结构模型**

![RMI model](\images\NuevoMatch\Figure 3 RMI model.jpg)

> 每个节点都进行训练，然后最终叶子节点产生，预测的序号

**训练过程**

> **首层**：训练整个数据集，然后分为$W_1$个子集，有$\hat{j} = \hat{h}(key),j = \lfloor \hat{j}\cdot W_1 \rfloor$ ,将当前key放到$m_{1,j}$中
>
> **中间层节点：**
>
> **最后一层：**由于不是百分百匹配准确，所以采用在一个范围$\epsilon$内搜索。**
>
> **别管那么多，反正就是个使用神经玩过完成从**key-value**映射的玩意

**RMI限制**

> 是适应key-value，不适应range-value，需要存储整个range中的每个的map

### 3.3 一维RQ-RMI

![RQ-RMI training process](\images\NuevoMatch\Figure 5 submodel training process.jpg)

### RQ-RMI总结

**training**

> 分割输入为iSet
>
> 对每个iSet训练RQ-RMI
>
> 构建扩展分类器

**lookup**

> 查RQ-RMI
>
> 查扩展分类器
>
> 返回最高优先级

### 实验细节

#### RQ-RMI结构

![RQ-RMI configurations](\images\NuevoMatch\Table 4 RQ-RMI configurations.jpg)

上表记录每一层的节点数与规则数的对应关系

#### 中间节点结构

三层全连接的神经网络，中间隐藏层有8个神经元，使用ReLU激活函数，1 input 1output

#### 训练过程

每个submodel需要几分钟的样子

#### iSet分割



还有一些并行计算的东西，

### **多维度处理**

![partition rules](/images/NuevoMatch/Figure 6 partition Rules.jpg)

> **将Rule分割到不相交的维度iSets**
>
> 使用启发式算法完成分割

#### iSet Partitioning

使用贪心算法（木棍问题），按照每个规则的upper bound 先进行排序，每次选择上界最小的规则，

每次都根据不同的field进行选择，然后选择出规则数最多的iSet进行分割，这样分完的iSet，保证起码在一个field上没有重叠

> **对每个规则执行多字段验证。**