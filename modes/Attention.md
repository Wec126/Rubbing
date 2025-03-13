# Attention

## 1. A high-level look

首先将模型视为黑盒，比如在机器翻译中，接收一种语言的句子作为输入，然后将其翻译为其他语言；主要由编码组建(encoder)、解码组件(decoder)和他们之间的连接层组成

<img width="377" alt="image" src="https://github.com/user-attachments/assets/c04745e7-4337-48bb-a8a7-c9a1be3bca73" />

### 编码器 encoder

其中，编码组建有6层的encoder首位连接而成；encoder是完全结构相同的，但是并不共享参数，每个编码器都可以拆解成以下两个部分：

<img width="478" alt="image" src="https://github.com/user-attachments/assets/51f0be0f-d081-4520-849c-bfce9f5c1862" />

[一个self-attention + FFN(通常情况下FFN由两个全连接层+一个非线性激活函数)]

encoder的输入首先先经过一个self-attention,该层帮助encoder能够看到输入序列中的其他单词。self-attention的输出流向一个前向网络，每个输入位置对应的前向网络是独立互不干扰的。

### 解码器 decoder


解码器同样也有这些子层，但是在两个子层增加了attention层，可以帮助解码器能够关注到输入句子的相关部分，与 **seq2seq model**中的attention作用相似。

<img width="606" alt="image" src="https://github.com/user-attachments/assets/307ccffd-da99-4de6-8cfe-e09e22b55282" />

### Bring the tensors into the picture

对于输入来说，首先会通过 **embedding algorithm**转换为向量；然后每个编码器会接收到一个list(这个list中有多个词向量组成)；但是这种向量化只是发生在最底层的编码器的输入时，只不过其他编码器是这个前个编码器的输出。list的尺寸是可以设置的超参数

<img width="497" alt="image" src="https://github.com/user-attachments/assets/b5737ce8-df7a-473e-8235-a1e956e4edaf" />

这里也有transformer的一个关键特性，每个位置的词仅仅流过它自己的编码器路径。在self-attention中，这些路径是两两之间相互依赖的。**前向网络层没有这些依赖性**，但这些路径在流经前向网络时可以并行执行。

<img width="497" alt="image" src="https://github.com/user-attachments/assets/ffb4645d-d1fe-4b8a-bdb9-9ca7fde97c2f" />

编码器接收向量的list作为输入，然后将其送入self-attention处理，再之后送入前向网络，最后将输入传入下一个编码器中。

### Self-attention

1. 计算编码器的输入向量，生成三个向量，对于每个词向量，生成 **query-vec,key-vec,value-vec**，生成方法为分别乘以三个矩阵。(注意！！不是每个词向量独享三个matrix，而是所有输入共享三个转换矩阵，并且这些新向量的维度比输入词向量的维度要小)

<img width="498" alt="image" src="https://github.com/user-attachments/assets/8d206322-c9b2-4203-9919-40d6f1c504b8" />

2. 计算attention,需要计算每个词语thinking的评估分，这个分决定着编码某个词时，每个输入词需要几种多少关注度，这个关注度可以通过计算 **q_1和k_1**的点积

<img width="438" alt="image" src="https://github.com/user-attachments/assets/d4128827-59a5-405f-ab1f-c6d2e065ad6d" />

3. 除以 $\sqrt{d_k}$，然后加上softmax操作，归一化分值使得全是整数且加权和为1

![Uploading image.png…]()



