# Concept Bottleneck Models[https://arxiv.org/pdf/2007.04612]
[以医学诊断为背景]

概念瓶颈模型主要分为一下两步工作：
1. 首先预测人类可解释的概念(例如X光中的‘骨刺’)
2. 使用这些预测的概念做出最终预测(例如关节炎严重程度)

这种方法提供了一定的优势：
1. 允许人类通过编辑预测的概念来进行干扰
2. 使用有意义的临床术语或者属性实现解释
3. 促进更好的人机交互

CBM：首先预测人类指定的中间概念集 $c$，然后使用 $c$预测目标 $y$

模型训练：
* 使用三元组数据点 $(x,c,y)$进行训练，其中输入 $x$同时被概念 $c$和目标 $y$标注
* 在测试时，模型接收输入 $x$,预测概念 $\hat{c}$，然后使用这些概念预测目标 $\hat{y}$

本篇论文中提出一种直接方法，将在训练时有概念标注的端到端神经网络转变为概念瓶颈模型，主要方法为：调整网络中某一层的大小以匹配提供的概念数量，并训练一个中间损失函数，促使该层中的神经元与提供的概念逐个对应,这种方式训练的概念瓶颈模型能够达到与标准模型**相当甚至更高**的任务准确率。并且，测试时**不需要概念标注**，模型预测概念，然后使用预测的概念做出最终预测

**概念干预(concepts intervention)**

1. 瓶颈模型允许通过编辑概念预测 $\hat{c}$并将这些变换传播到目标预测 $\hat{y}$来进行概念干预

2. 使用这种人类知识注入时，发现准确率远超 **标准模型**

3. 通过操作概念 $\hat{c}$并观察模型的响应，我们可以获得反事实解释；相比之下，之前关于端到端模型高级概念解释的工作只能应用于事后解释

4. 模型干预的有效性取决于其预测概念 $\hat{c}$和真实概念 $c$之间的对应关系；可以通过测量模型在保留验证集上的概念准确率来估计这种对应关系
5. 非瓶颈模型通常将人类制定的概念用作多任务设置中的辅助目标或者辅助特征，这些模型不支持概念干预；例如在多任务模型( $c\leftarrow x \to y$)

**研究贡献**

1. 在两个数据集(CUB * OAI)上评估了概念瓶颈模型，发现与标准的端到端模型相当，同时也达到了高概念准确性
2. 证明了通过在测试时干预这些瓶颈模型来纠正模型在概念上的错误，可以显著提高模型准确性
3. 最后发现，被引导正确学习概念的瓶颈模型对协变量偏移**更具有鲁棒性**
4. cbm可以表示 $x\to c\to y$之间的因果关系，但有灵活性优势，并不要求 $c$导致 $y$，只关注其预测性

## Setup

**回归问题**

输入： $x\in R^d$ 预测目标 $y \in R$, 设置训练点 $\lbrace (x^{(i)},y^{(i)},c^{(i)})\rbrace^n_{i=1}$,其中 $c\in R^k$是k个概念的向量，设置 $f(g(x))$为瓶颈模型，其中 $g: R^d \to R^k$将输入 $x$映射到概念空间，而 $f:R^k \to R$将概念映射到最终预测

[在当前的数据集环境下，我们可以认为c为骨刺，f得到的结果为关节炎严重程度]

上面成为概念瓶颈模型，因为他们的预测 $\hat{y} = f(g(x))$完全通过瓶颈 $\hat{c} = g(x)$依赖于 $x$，我们训练该瓶颈使得其逐分量与概念 $c$对齐。将任务准确率定义为 $f(g(x))$预测 $y$的准确程度，将概念准确率定义为 $g(x)$预测 $c$的准确程度，即将 $g(\cdot)$称为预测 $x\to c$,将 $f(\cdot)$称为预测 $c\to y$

初次之外，关于损失函数：

*  $L_{C_j}$： $R \times R \to R^+$为衡量预测概念与真实第 $j$个概念之间差异的损失函数
*   $L_Y: R \times R \to R^+$ 衡量预测目标与真实目标之间的差异

关于cbm的三种不同学习方法：

1.  **independent**, 独立地学习 $\hat{f}$和 $\hat{g}$

    $\hat{f} = argmin_f \sum_i L_Y (f(c^{(i)});y^{(i)})$

    $\hat{g} = argmin_g \sum_{i,j} L_{C_j}(g_j(x^{(i)});c^{(i)})$

   虽然 $\hat{f}$是使用真实的 $c$训练的，但是在测试时它仍以 $\hat{g}(x)$作为输入

2. **sequential**，顺序瓶颈模型，首先以上述相同的方式学习 $\hat{g}$，然后使用概念预测 $\hat{g}(x)$来学习：

    $\hat{f} = argmin_f \sum_i L_Y(f(\hat{g}(x^{(i)}));y^{(i)})$

3. **joint**，联合瓶颈模型，最小化加权和：

     $\hat{f},\hat{g} = argmin_{f,g}\sum_i[L_Y(f(g(x^{(i)}));y^{(i)})+\sum_j\lambda L_{C_j}(g(x^{(i)});c^{i})]$

   关于超参数 $\lambda$，在联合瓶颈模型中， $\lambda$控制概念损失与任务损失之间的权衡，当 $\lambda \to 0$时，联合瓶颈模型等同于标准模型；顺序瓶颈模型可以看作是 $\lambda \to \infty$

不同瓶颈模型的比较：

* 与独立瓶颈相比，顺序瓶颈允许模型的 $c\to y$部分适应它对 $x\to c$的预测能力
* 联合瓶颈进一步允许模型对概念的版本进行调整，以提高预测性能

初次之外，还设置了对照组，**standard model**标准模型，这种模型忽略概念，直接最小化：

  $\hat{f},\hat{g} = argmin_{f,g} \sum_i L_Y(f(g(x^{(i)}));y^{(i)})$

将端到端神经网络转变为cbm的简单方案：
* 将网络中的一层调整为 $k$个神经元，以匹配概念数量 $k$

**分类问题**

* 在分类任务中， $f$和 $g$计算实指分数
* 然后将这些分数转换为概念预测，如 $P(\hat{c}_j = 1) = \sigma(\hat{l_j})$，其中 $\sigma$为逻辑回归的sigmoid函数
* 这不会改变独立瓶颈模型，因为 $f$直接在二值 $c$上训练
* 对于顺序和联合瓶颈模型，将 $f$连接到对数几率 $\hat{l}$，即：

    * $P(\hat{c_j} = 1) = \sigma(\hat{g}_j(x))$
    * $P(\hat{y} = 1) = \sigma(\hat{f}(\hat{g}(x)))$

## Benchmarking bottleneck model accuray

### 1. Applications

**dataset**
主要在一下两个数据集上进行了实验：

1. OAI -> regression (实例级概念标注)

   * 根据X光片预测关节炎的登记，一共有4级，用于衡量骨关节炎的严重程度，分数越高说明疾病越严重
   * 共有10个有序变量作为概念
   * 每个x光有独特的概念标注

2. CUB -> classification (类级概念标注)

   * 从200个可能的选项中分类正确的鸟类物种
   * 共有112个属性作为概念
   * 使用类级概念(按照鸟的种类来区分概念的)，因为提供的概念有噪声，通过多数投票进行去燥，例如，如果数据中超过 50%的乌鸦有黑色翅膀，那么将所有乌鸦设置为黑色翅膀，即假设训练数据中同一物种的所有鸟类共享相同的概念标注
  
**model**

根据测试，可以确定使用的 $\lambda$值，其中OAI数据集使用 $\lambda = 1$，CUB数据集使用 $\lambda = 0.01$

1. OAI -> model

   * 学习 $g$(从 $x\to c$)：Resnet18
   * 学习 $f$(从 $c \to y$)：一个三层的mlp

2. CUB -> model

   * 学习 $g$: inception v3
   * 学习 $f$: 单个线性层

### 2. Task and concept accuracies

<img width="798" alt="image" src="https://github.com/user-attachments/assets/5b2e328f-5665-4d00-9c2f-6283cad6d344" />
[上图记录的是损失，越小越好]

根据实验结果可以看到：

* 尽管有瓶颈约束，但是cbm还是在两个任务熵到达了与标准黑盒模型相当的任务准确率
* 在OAI上，联合瓶颈和顺序瓶颈模型在RMSE上实际比**标准的还好**
* 在CUB上，顺序和独立瓶颈模型比标准略差，但是联合模型弥补了大部分差距


