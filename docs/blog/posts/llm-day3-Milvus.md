---
draft: true
search:
  exclude: true
date:
  created: 2025-01-02
categories:
  - Learning
tags:
  - LLM
  - RAG
authors:
  - zhazi
---

# LLM 学习：Day3 - Milvus

!!! abstract ""

    上一阶段部署的 TinyRAG 部署采用了简单的 JSON 文件存储向量化结果。这种方案虽然易于实现，但可以预见的是，随着知识库规模的扩大，性能瓶颈逐渐显现：查询效率下降、存储空间占用过大、数据维护困难等问题接踵而至。而这，就是各种向量数据库发挥作用的地方了。

    day3 计划：升级 TinyRAG 的存储方案，使用向量数据库 Milvus 管理数据
    Day4 计划：进一步学习 Milvus，然后完成这篇博客

??? notes "Milvus"

    Milvus 是一个高性能、高扩展性的**向量数据库**，Milvus 支持多种数据类型和搜索功能，包括 ANN 搜索、过滤搜索和全文搜索等。Milvus 的设计考虑了硬件优化和高效的搜索引擎，因而具有良好的性能。此外，Milvus 还提供了多种部署模式，以适应不同的数据规模和应用需求。
<!-- more -->
