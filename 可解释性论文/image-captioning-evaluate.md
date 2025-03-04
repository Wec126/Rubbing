# 关于评估图像描述任务性能的工具

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
