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

**在测试时**，都是使用的预测得到的 $\hat{c}$来预测结果的

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

<img width="616" alt="image" src="https://github.com/user-attachments/assets/c8a5abbe-da6e-44e5-a8b8-f662dfdf80e2" />

[平均概念损失]

* 可以看到相比于standard来说，使用cbm得到的平均概念误差更小
* 低概念误差表示模型学到的概念与真实概念是一致的，这意味着研究者可以对这些概念进行intervention
* 没有观察到任务精度和概念精度之间存在tradeoff，也就是说*将瓶颈层使用概念c并不会影响模型预测y的能力，即使是在瓶颈层进行联合训练的情况下也是这样的*

  <img width="365" alt="image" src="https://github.com/user-attachments/assets/8475481a-eacf-40f4-a627-9adea90cbd30" />

[在神经网络中，提高模型对中间概念的理解通常会以牺牲最终任务性能为代价，但是这个论文的实验表明，至少在他们的任务中，可以达到较高水平]

**Additional baselines**

1. 测试了一个没有瓶颈层的standard模型

   结果表明，cbm与standard模型相似

2. 多任务设置(multitask)

   使用辅助损失函数来鼓励最后一层的激活值能够预测概念c，对这个辅助损失的不同权重进行了超参数搜索

**Data efficienty**

<img width="490" alt="image" src="https://github.com/user-attachments/assets/2f4f9c30-4f02-4433-a9e0-6aeca658a13f" />

通过测量数据效率来对比不同模型的方法：即为达到期望准确度水平所需的训练数据量

1. 在OAI上，CBM特别有效，使用大约25%的全量数据集的sequential bottleneck model表现出与使用100%的标准模型相当
2. 在CUB上，joint bollteneck和standard models在整个过程中表现地更好

**关于为什么cbm在regression上的效果更好**

1. 在论文中的classification的demo中CUB数据集的模型架构的最后一层是一个线性层，只考虑了线性关系，但是在真实的情况下，线性层无法很好的表达一些概念关系(很多概念都是使用非线性层得到的)，无法表达复杂的概念组合关系 ！！！！改一下试试
2. 回归任务中的概念组合往往可以通过简单的加权求和等线性操作得到合理的结果


## Benchmarking post-hoc concept analysis

CBM可以将瓶颈层中的人类指定的概念c一一对应，对于测试输入 $x$，可以直接从瓶颈层读取预测的概念，并且可以通过操作预测概念 $\hat{c}$并且检查最终预测 $\hat{y}$的变化来进行概念干预

post-hoc：先训练一个直接预测模型 $x\to y$，然后使用探针probe从模型的激活值中恢复已知概念

*关于post-hoc的局限性*

1. 高概念准确性是事后解释的必要条件
2. 事后分析不允许对概念进行干预
3. 没有干预能力，基于概念的解释只是暗示性的，也就是说我们能说'模型认为关节空间狭窄'，但是很难测试这是否真正影响了最终预测

<img width="471" alt="image" src="https://github.com/user-attachments/assets/802483d0-d6a8-4df1-8369-d1760c56d293" />

研究者发现，probe的准确率低于直接从瓶颈模型读取概念

## Test-time intervention

<img width="908" alt="image" src="https://github.com/user-attachments/assets/91f6585a-e71d-4a4c-90b7-c21b75ab9466" />

1. OAI上的干预


**干预方法**

* cbm中首先从输入 $x$预测概念值 $\hat{c}$，然后使用这些预测的概念值预测目标 $\hat{y}$
* 在OAI数据集上，干预第 $j$个概念被定义为用 **oracle**提供的真实值 $c_j$代替预测值 $\hat{c}_j$,然后更新预测结果 $\hay{y}$
* 可以通过同时替换多个响应的预测概念值进行多概念干预

<img width="479" alt="image" src="https://github.com/user-attachments/assets/d31f5bae-8927-40fa-8413-e9979c06cbc7" />



根据上图可以看到，干预显著提高了OAI数据集上的任务准确性，仅仅改变两个概念，就可以将任务的RMSE从 $>0.4$降到 $\approx 0.3$

对于independent来说，所有概念都被替换时获得了更好的测试误差，但在没有任何干预时表现略差

**independent瓶颈模型**

对于independent瓶颈模型来说，预测 $y$的部分 ($c\to y$)是直接用真实概念 $c$进行训练的；但是joint和sequence来说，他们是给定的预测概念 $\hat{c}$，然后预测输出 $y$

但是在测试时，由于所有的模型使用的都是预测得到的 $\hat{c}$，因此得到的结果很差；但是将所有的概念替换为真实的概念 $c$(也就是换到independent训练时用的概念),效果会恢复正常

**影响干预有效性的因素(基于消融实验)**

在joint中 $\lambda$的值用于控制模型在训练时对预测概念 $c$和预测任务目标 $y$的重视程度； $lambda$ 越大，模型越注重预测概念， $\lambda$越小，模型越注重最终任务目标 $y$

但是，它可能导致模型内部使用的概念与我们认为的概念有不同的含义或表示；因此，当使用真实概念 $c$来替换预测概念 $\hat{c}$时，模型表现反而很差，因为真实 $c$与模型内部使用的概念表现不匹配

将 $c\to y$模型从3层感知器改为单线性层时，测试的干预效果减弱

<img width="855" alt="image" src="https://github.com/user-attachments/assets/e6bafdc8-3839-4dd4-a073-82de1aaa3abe" />

这说明，即使模型能够准确预测概念和最终任务，但是其内部处理方式还是有很大的差异的；非线性可以捕捉到概念之间更复杂的关系，使其能够更好地适应干预带来的分布变化；线性虽然简单，但是无法灵活地来处理干预引入的变化

2. CUB上的干预

**挑战**

由于在CUB中，最终预测器 $y$接收的是预测概念的logits，因此无法使用真实概念 $c_j$替换预测概念

在这里的修改方法是修改logits

对相关概念进行分组并且一起干预，这是因为许多改变编码相同的底层属性(例如翅膀等)


<img width="441" alt="image" src="https://github.com/user-attachments/assets/e4aa0a36-89a2-4c39-a4c6-2323e46a4007" />

可以看到，与OAI类似，测试时干预在Independent的效果更好；这可能是因为sequential和joint中使用的第5/95百分位方法不够精准

因此，规划了下面的实验：

标准模型：概念层输出logits $\hat{l}_j$，然后直接连接到分类器

修改版模型：概念层输出logits,然后使用sigmoid转换为概率之后，再连接到分类器
(也就是将当前的概率放在整体中进行了压缩)

可以看到，虽然出版的错误率更高(可能是因为sigmoid的挤压效应，将梯度变小，导致优化困难)，但是其干预效果更好，并且干预过程更加直接

## Robustness to blackground shifts

为了探讨概念瓶颈模型是否比标准模型更能抵抗训练分布中存在但测试分布中存在在的虚假相关性，例如在CUB中，模型可能错误地将图像背景与标签关联起来

<img width="457" alt="image" src="https://github.com/user-attachments/assets/84ea138d-6f4e-4d6b-9539-82b367632464" />

[将每只鸟从原始背景中裁剪出来，放在places数据集上]


<img width="559" alt="image" src="https://github.com/user-attachments/assets/67385800-8c22-42c7-9525-1778d2c2fb95" />

可以看到cbm的表现优于standard模型；说明cbm较少依赖于背景特征，因为每个概念在多个鸟类之间共享



