## Bert & ELMo

## EMLo

[一下是关于EMlo的细节图]

<img width="291" alt="image" src="https://github.com/user-attachments/assets/120ed4d1-d223-461d-a159-a873be3e48bc" />

序列标注模型：为序列中的每个元素分配一个标签

EMLo的大致过程：
1. 预训练一个embedding(词嵌入) + language model(循环语句模型中获得上下文相关的表示)

   [通常来说，词嵌入捕获一般语义特征；语言模型嵌入捕获上下文相关信息]
   
2. 对句子中的每一个token使用两种模型进行表示
3. 在序列标注模型中结合两种表示：模型为每个词预测标签，输出是一个标注序列

特点：
1. 使用双重表示，为每个词提供两种互补的表示，既有静态语义信息，又有上下文相关信息
2. 语言模型增强：使用语言模型的能力来增强传统序列标注


<img width="629" alt="image" src="https://github.com/user-attachments/assets/6e62d2dc-9dd9-4ea3-b8dd-afaf2c6c2fd0" />

右侧：预训练的双向语言模型(pretrained bi-lm)

由两个部分组成：

1. 前向语言模型(forward lm)

   从左到右的序列

2. 后向语言模型(backward lm)

   从右到左

左侧：序列标注架构

1. 捕获字符级特征(CNN等)，捕获词内形态信息 + embedding词嵌入
2. 第一层RNN：处理基本token表示，生成 $h^{LM}_1$
3. 增强表示：将预训练语言模型的嵌入表示与第一层RNN的隐藏状态连接，将语言模型的知识注入序列标注模型
4. 第二层RNN
5. 输出层：dense + crf

ELMo：在为每个单词分配词向量之前先查看整个句子，然后使用bi-LSTM来训练他对应的词向量

<img width="745" alt="image" src="https://github.com/user-attachments/assets/9d1e72db-e625-4535-91ba-6b3f2f6ea1d0" />

## semi-supervised 半监督学习

同时使用标记数据和未标记数据进行训练。

bert在预训练阶段使用大量未标记文本，模型执行自监督任务(从数据本身自动生成监督信号，不需要人工标注)；然后在微调阶段，使用有监督学习

<img width="735" alt="image" src="https://github.com/user-attachments/assets/8599c474-a30c-4f64-8fa2-e59c031d8509" />

这样的好处：可以节约大量训练语言模型所需要的时间，精力，知识和资源

## ULMFIT

用于迁移学习

主要创新点：
1. 三阶段训练过程：

* 语言模型预训练：在大规模通用语料库上训练语言模型
* 领域适应：在目标领域的未标记数据上微调语言模型
* 分类器微调：添加分类层，在有标签数据上微调整个模型

2. 引入了几种特殊的微调技术：

* 逐层解冻(gradual unfreezing)：从上到下逐层解冻
* 学习率差异化：为不同层设置不同学习率
* 学习率调度：特殊的学习率调整策略

## Bert

BERT的基础即成单元是Transformer的Encoder:

2个bert的模型都有一个很大的编码器层数，基础版本有12个，large有24个，同时也有很大的前馈神经网络(768和1024个隐藏层神经元)，还有很多attention heads.

**模型输入**

<img width="582" alt="image" src="https://github.com/user-attachments/assets/90d860a1-0e77-44f1-822d-a2b733583bd6" />

其中第一个字符为 [CLS],表示classification。与transformer的编码一样，将固定长度的字符串作为输入，数据由下而上传递计算，每一层都用到了self attention,并且通过ffn传递结果，将其交给下一个编码器。

<img width="673" alt="image" src="https://github.com/user-attachments/assets/30863de8-ef8e-47d9-8b77-4e9a7a41c4da" />

**模型输出**

每个位置返回的输出都是一个隐藏层大小的向量(例如：基础bert中为768),以文本分类为例，我们重点关注在第一个位置上的输出

<img width="571" alt="image" src="https://github.com/user-attachments/assets/c1fe74c5-ee3a-4ef8-ad0d-bcbd3b876bd1" />

这个向量可以作为我们选择的分配器的输入，可以进行分类

## OpenAI Transformer

openai transformer：仅使用了decoder,因为它可以适合语言建模(预测下一个词)，他的设计就是遮蔽未来的token；这个模型堆叠了12个decoder层，因为没有encoder，所以使用的是self-attention；通过这个结果，可以使用大量未标记数据集来训练模型预测下一个单词

<img width="427" alt="image" src="https://github.com/user-attachments/assets/47ec5856-0df5-4891-812e-c5dead94247a" />

并且openai的论文中概述了许多transformer使用迁移学习来处理不同类型NLP任务的例子：

<img width="718" alt="image" src="https://github.com/user-attachments/assets/b33c360a-4d35-4633-80c8-b2bc8cfa444d" />

但是在BERT中，使用双向注意力机制，同时可以看到左右侧的上下文，并通过随机遮蔽部分token来预测被遮蔽的词(随机遮蔽输入中15%的次元，并将他们替换为一个特殊的 **[MASK]** 标签，然后模型的任务是基于上下文来预测这些被遮蔽的词应该是什么，类似于完形填空)

<img width="680" alt="image" src="https://github.com/user-attachments/assets/ff3de478-98f8-4803-a5cd-c12979367d57" />

## 使用

1. 常用的任务

<img width="690" alt="image" src="https://github.com/user-attachments/assets/64efd1d4-da9b-4768-bb0f-44cfbbcfae54" />

[句对分类任务，单句分类任务，问答任务(模型需要定位段落中的答案位置)，序列标注任务]

2. 获得词嵌入

由于bert基于emlo，可以使用预训练的bert来创建词嵌入








