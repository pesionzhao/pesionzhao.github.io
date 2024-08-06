## gpt2详解

[图解gpt2](https://jalammar.github.io/illustrated-gpt2/)

**什么是自注意力机制?**

- 注意力机制就是特征层与输出相乘,越大的地方就说明该地方更应该被注意
- 自注意力机制就是特征层和特征层相乘,越大说明这两个位置越相关

首先我们把一个句子分段

- 每个单词需要经过一个词嵌入矩阵(wte)得到其对应的向量: 这一过程可以理解为把单词编码成计算机可以理解的数字
- 单词的位置也很重要,在经过词嵌入之后要加位置编码矩阵(wpe),才能送入transformer block
- 这个向量会经过自注意力模块,前向网络,得到结果后压入栈用于下一个单词的计算

一个简单的例子

>First Law of Robotics
>>A robot may not injure a human being or, through inaction, allow a human being to come to harm.

>Second Law of Robotics
>>A robot must obey the orders given **it** by human beings except where **such orders** would conflict with the **First Law**.

标黑字体的单词我们着重关注

- it 指 robot
- such orders 指上文中的 "the orders given it by human beings"
- The First Law 指上文的 "First Law of Robotics"

自注意力机制就是计算当前词和之前词的关联性,在送入网络之前,需要把当前单词及其相关单词进行融合,再送入网络,这里就引出三个很重要的矩阵query,key,value

- query: 当前单词用于评价其他单词的矩阵(当前单词的query乘其他单词的key就可以知道其他单词相对于当前单词的分数)
- key: 可以理解为当前单词的一种特征,在计算其他单词时用于和其他单词的query匹配
- value: 当前单词的硬指标,如果说前面两个矩阵计算和其他单词的相关性,这个就是单词本身的价值


用博客中的类比就是:把它想象成在文件柜里搜寻。query就是你需要寻找东西的便利贴。key就像柜内文件夹的标签。当您将标签与便利贴相匹配时，我们取出该文件夹的内容，这些内容就是value。只不过您不仅要寻找一个值，而且要从混合文件夹中寻找混合值。

将Q乘以每个K向量生成每个文件夹的score(点积后+softmax)。将每个score乘value并相加,得到自注意力的输出

得到自注意力的输出后,再经过神经网络,编码器,得到输出向量,再与词嵌入矩阵(wte)点乘,得到了词嵌入矩阵中每个单词的分数(看看输出向量和词嵌入矩阵中哪个分数高),找最高的单词作为token,也可以通过概率的方式抽取,或者取前40个大的token,这就完成对一个单词的操作,我们还要将这样的操作重复很多次直到遍历完了整个串

训练时是长序列中的多个token,预测时是单个token的预测

transformer使用了很多层的归一化


pre-train: 用于让模型可以生成文本(通过前面的词生成下一个词)
 
fine-tuning: 根据问题生成答案

stage3: compare answer

未来的LLM

- 可以读取并生成文本
- 浏览文件,网页
- 使用工具: e.g 计算器,编译器
- 生成图像,视频
- 可以听,说,写歌
- 可以长时间思考(system 2)而不是根据经验(system 1)
- 可以有自我奖励机制进行自我改进,针对特定任务进行微调
- 成为一个操作系统 LLM OS!!!!
