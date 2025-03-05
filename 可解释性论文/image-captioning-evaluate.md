# 关于评估图像描述任务性能的工具[https://github.com/bckim92/language-evaluation]

## CocoEvaluator

基于COCO数据集开发的评估框架,包含一系列广泛使用的自然语言评估指标

**BLUE1-4(Bilingual Evaluation Understudy)**：评估生成文本与参考文本在**n-gram**上的重叠度，数字1-4表示考虑的**n-gram**大小(单词序列长度)

  **n-gram**： 指的是从文本或语音中连续提取的n个项目的序列，n-gram重叠度用于测量生成文本与参考文本的相似程度
  
  * 1-gram：单个单词
  * 2-gram：两个连续单词
  * 3-gram：三个连续单词
  * 4-gram：四个连续单词

**METEOR(Metric of Evaluation of Translation with Explicit ORdering)** : 考虑同义词、词干和语序的评估指标，通常比 **BLEU**更好与人类判断相关

**ROUGE(Recall-Oriented Understudy of Gisting Evalaution)**：主要关注召回率，评估生成文本包含参考文本内容的程度

**CIDEr(Consensus-based Image Description Evaluation)**: 专为图像描述任务设计，使用TF-IDF加权来强调不常见但信息量大的n-gram

**SPICE(Semantic Propositional Image Caption Evaluation)**:基于语义命题的评估指标，将描述转换为图结构并比较语义相似度，特别关注对象、属性和关系


## RoughEvaluator

Recall-oriented understudy of gisting evaluation一种用于评估自动摘要和机器翻译质量的指标集合

**ROUGE-1**：测量候选句子与参考句子之间单个词(unigrams)的重合度，它只关注词汇重合，不考虑词序

**ROUGE-2**：测量候选句子与参考句子之间二元词组(bigrams)的重合度，可以捕捉一些词序的流畅性信息

**ROUGE-L** ：基于最长公共子序列(LCS)的度量，不要求匹配的单词必须相邻，但必须按照相同的顺序出现，可以捕捉句子结构的相似性

f-measure：表示这些ROUGE评分是通过计算精确率、召回率以及二者的调和平均数获得的，具体来说：

* 精确率：候选句子中有多少词出现在参考句子中
* 召回率：参考句子中有多少词出现在候选句子中
* F值(F-measure)：精确率和召回率的调和平均数，通常表示为F1分数

ROUGE指标通常用于评估摘要系统或机器翻译系统的输出质量，它在定量评估生成文本与参考文本的相似度方面非常有用。

## Rouge155Evaluator

是执行摘要级别(summary-level)ROUGE评估的工具，其中的155指的是原版ROUGE-1.5.5实现版本；相比于上面的句子级别的rougeevaluator来说，这个是用于评估整个摘要文本(可能包含多个句子)，而并非一个句子，它计算一下指标：

* ROUGE-1：评估参考摘要与系统生成摘要之间的单词重合度
* ROUGE-2：评估参考摘要与系统生成摘要之间的二元词组重合度
* ROUGE-L：使用最长公共子序列(LCS)度量来评估摘要文本间的结构相似性，能够捕捉更长距离的词序关系
