## Bert & ELMo

## EMLo

[一下是关于EMlo的细节图]

<img width="291" alt="image" src="https://github.com/user-attachments/assets/120ed4d1-d223-461d-a159-a873be3e48bc" />

EMLo的大致过程：
1. 预训练一个embedding(词嵌入) + language model(循环语句模型中获得上下文相关的表示)

   [通常来说，词嵌入捕获一般语义特征；语言模型嵌入捕获上下文相关信息]
   
2. 对句子中的每一个token使用两种模型进行表示
3. 在序列标注模型中结合两种表示：模型为每个词预测标签，输出是一个标注序列

特点：
1. 使用双重表示，为每个词提供两种互补的表示，既有静态语义信息，又有上下文相关信息
2. 语言模型增强：使用语言模型的能力来增强传统序列标注
<img width="629" alt="image" src="https://github.com/user-attachments/assets/6e62d2dc-9dd9-4ea3-b8dd-afaf2c6c2fd0" />









