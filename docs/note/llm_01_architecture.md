---
draft: true
date:
  created: 2025-02-02
categories:
  - Learning
  - Memo
tags:
  - LLM
authors:
  - zhazi
---

# LLM 笔记：大模型学习方法

!!! abstract ""

    记录学习 LLM 过程中的思考与实践。

    讲解逻辑：按领域发展过程来介绍，从 token 到 embedding 到 预训练 到 transformer，然后是基于 transformer 的预训练模型三大技术路线

    ??? info "参考文献"

<!-- more -->


## Retrieval Augmented Generation (RAG)


### Why LLM work in everaywhere？

!!! quote "摘要"

    任何信息都可以编码为 token，而任何 token 都可以被 LLM 学习（与泛化）

### Tokenization

- token -> embbeding

### Pre-trained Embedding in Old Days

- 训练出的词词向量是 context-free 的

### ELMo: Contextualized Pre-trainning Starts Here

!!! note "Contextualized"

    上下文相关的

- Contextualized (上下文相关的)
- Deep

### Transformer

- 注意力机制天然适合 Contextualized，因此很快出现了基于 Transformer 的预训练模型
- 三条技术路线：
    - BERT (Encoder only)
    - GPT (Deconder only)
    - T5 (Encoder & Decoder)

#### A Quick Look of GPT

- left-to-right 的语言模型
- 预测下一个 token

#### A Quick Look of BERT

- 在当时几乎所有任务上都 SOTA，甚至部分超越人类
- 奠定了 PLM 的主流地位
- 预测中间的 token + 预测下一个句子(研究表明这一项不够本质)
- 指明了语言模型中 bigger == better 的基本规律 

#### A Quick Look of T5

- 证明了所有 NLP 任务都可以用到同一个语言模型中

### Pre-trainning

- BERT: Masked Language Modeling
- GPT: Auto-regressive
- T5: Seq2Seq

### GPT 的语言建模

!!! note "Language Modeling"
  

!!! quote "摘录"

    GPT 语言建模可能是一种极致的多任务学习

!!! note "Perplexity 困惑度"

### GPT-1
