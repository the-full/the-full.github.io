---
draft: true
date:
  created: 2025-02-01
categories:
  - Learning
  - Memo
tags:
  - LLM
  - Ollama
authors:
  - zhazi
---


# 学习笔记：Ollama

!!! abstract ""

    记录学习 Ollama 过程中的思考与实践。

    讲解逻辑：Ollama 的核心功能是简化本地部署 LLM 的过程，所以先介绍 Ollama 模型的概念，然后依次介绍 CLI、API、UI、Python Library 四种模型交互方式。前两种方式展示基本功能，UI 展示在基础功能的基础上能实现的功能，Python Library 提供开发基础，从这里开始，由理论向实践过渡。

    ??? info "参考文献"

        - [【Ollama教程：从入门到精通本地AI开发】](https://www.bilibili.com/video/BV1xMfGYAEAF?vd_source=cc4cc357a788870e853d296da5fdbbd1)
<!-- more -->

## 05: Ollama 深度解析

### What is Ollama?

- 开源工具，用于简化本地运行 LLM 的过程

### What problem does it solve?

- 教程里讲的多数好处实际都是本地部署的好处而不是 Ollama 的好处
- 本地部署有好处（安全、可控等），而 Ollama 简化了本地部署

!!! warning "无用"

    这节课没什么营养。

## 06-08: 关于怎么下载 Ollama

!!! warning "无用"

    这些内容毫无营养。


## 09: Ollama 模型

!!! warning "一般"

    对着模型的页面讲了半天，鸡肋。

## 10: LLM 参数探讨

- 使用 `/show info` 命令可以获得一些模型信息
- quantization 表示量化方式

```console
>>> /show info
  Model
    architecture        llama     
    parameters          8.0B      
    context length      131072    
    embedding length    4096      
    quantization        Q4_K_M    
```

!!! warning "一般"

    鸡肋。

## 11: 理解模型基准

!!! warning "无用"

    不同的模型使用的基准不尽相同，呈现方式也五花八门，无非就是多看文档。

## 12: 命令行基础: 拉取和测试模型

- 类似 Docker 的 cli
- 可以用 `/bye` 退出交互

!!! warning "一般"

    所谓的测试模型只是指运行模型后的交互，鸡肋。

## 13: 使用 llava 多模态模型进行图像标题生成

- llava 没学过，要补一下
- 对话中的路径似乎可以被解析为**添加文件**操作

!!! info "差强人意"

    找到一个需要查缺补漏的地方。还行。

## 14: 使用 llama 进行情感分析 & 使用 modelfile 自定义模型

- 可以像 Docker 编写 dockerfile 一样编写 modelfile

```txt title='Modelfile 示例'
FROM llama3.2

# 是的，我们可以设置模型推理时的超参数
PARAMETER termperature 0.3

# 是的，我们可以设置模型的初始提示词
SYSTEM """
    You are James, ...
"""
```

通过 `create` 子命令的 `-f` 参数可以指定从 Modelfile 构建模型。 
```console
ollama create james -f /path/to/Modelfile
```

!!! info "可以"

    学习了 Modelfile 的概念。

## 15-16: Ollama REST API

- Ollama 提供了 RESTful 的 API 接口，后面升级 TinyRAG 可以建立在这些 API 上
- CLI 也是建立在 API 上的，所以 CLI 能做的，也可以通过其他程序来做，重点关注一下 `generate`, `chat` API
- [Ollama API 文档](https://github.com/ollama/ollama/blob/main/docs/api.md)

!!! note "很好"

    确定了 TinyRAG 的改进方案

## 17: Ollama 模型支持不同任务

- 依据任务场景选择最合适的模型

!!! warning "一般"

    鸡肋。

## 18-20: Ollama 模型支持不同的交互模式

- CLI
- UI-based interface (Msty)
- API
- **Ollama python Library**

!!! note "很好"

    理解了讲解方式。提供的 python 开发包让改进方案实现起来更容易了。

---

**以此为界，进入实践课程**

---

## 21: 通过 Ollama REST API 在 python 中与 LLM 交互

!!! warning "一般"

    鸡肋。

## 22: 通过 ollama 库在 python 中与 LLM 交互

!!! note "很好"

    ollama
