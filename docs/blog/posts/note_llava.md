---
draft: true
date:
  created: 2025-02-03
categories:
  - Learning
  - Memo
tags:
  - LLM
  - Ollama
authors:
  - zhazi
---

# 学习笔记：LLaVA

!!! abstract ""

    记录学习 LLaVA (Large Language and Vision Assistant) 过程中的思考与实践。

    # TODO: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa

    ??? info "参考文献"

<!-- more -->

## Motivation

- NLP、CV 领域最开始都是不同任务使用不同的模型，LLM 的出现为 NLP 领域带来了**通解**，但 LLM 是 text-only 的，并不能解决 CV 问题
- CV 领域还缺少一个通解，而 LLaVA 要解决的就是这个问题

## 数据来源

