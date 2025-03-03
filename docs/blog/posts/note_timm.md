---
draft: true
date:
  created: 2025-01-21
categories:
  - Learning
  - Memo
tags:
  - CV
authors:
  - zhazi
---


# DL 补习笔记：timm & Transformer

!!! abstract ""

    三维做多了，发现错过了很多二维的东西，我要补习！这篇记录学习 huggingface 的 timm 库和 Transformer 库过程中的思考。

    ??? note "timm"

        Py**T**orch **Im**age **M**odels（timm）是一个图像模型集合，包括层、实用工具、优化器、调度器、数据加载器/增强以及参考训练/验证脚本，旨在汇集多种最新的 SOTA 模型，并具备复现 **ImageNet** 训练结果的能力。

    ??? note "Transformer"

        最先进的机器学习技术，适用于PyTorch、TensorFlow和JAX。

        🤗 Transformers 提供了 API 和工具，可以轻松下载和训练最先进的预训练模型。

        - 📝 自然语言处理：文本分类、命名实体识别、问题回答、语言建模、代码生成、摘要、翻译、多项选择和文本生成。
        - 🖼️ 计算机视觉：图像分类、目标检测和分割。
        - 🗣️ 音频：自动语音识别和音频分类。
        - 🐙 多模态：表格问题回答、光学字符识别、从扫描文档中提取信息、视频分类和视觉问题回答。

        🤗 Transformers 支持在 PyTorch、TensorFlow 和 JAX 之间的框架互操作性。这为在模型的每个生命周期阶段使用不同的框架提供了灵活性；可以在一个框架中使用三行代码训练模型，并在另一个框架中加载它以进行推理。模型还可以导出为 ONNX 和 TorchScript 等格式，以便在生产环境中部署。


    ??? note "参考文献"
        
        - [Timm 手册](https://huggingface.co/docs/timm/feature_extraction)
<!-- more -->

## 特征提取

timm 提供了在 ImageNet 数据集上预训练的图像分类模型，这是其核心功能之一。基于这些预训练模型，timm 提供了灵活的特征提取机制，能够获取适用于各种下游任务的特征表示。


## 训练模型

timm 还提供了完整的训练工具链，包括优化器、学习率调度器和数据增强等功能，可用于模型的微调和迁移学习。不过现在不需要了解，略去。
