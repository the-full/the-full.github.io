---
draft: true
date:
  created: 2024-09-01
  updated: 2025-01-02
categories:
  - Reading 
tags:
  - Adversarial Attack
authors:
  - zhazi
---

# Geometry-Aware Generation of Adversarial Point Clouds

!!! abstract ""

    最近在阅读三维对抗方向的经典论文，目前的计划是复现相关方法来构建一个基础的研究环境，而这篇论文是计划中的一篇。


??? info "论文基本信息"

    - 论文链接：[arxiv](https://arxiv.org/abs/1912.11171)
    - 代码链接：[github](https://github.com/Gorilla-Lab-SCUT/GeoA3)
    - AI 摘要：这篇论文介绍了一种名为 $GeoA^3$ 的新方法，用于生成能够欺骗 3D 点云分类器的对抗性点云。研究者们发现，通过考虑人类对3D形状的感知方式，可以设计出难以被察觉的对抗性扰动。他们提出的 $GeoA^3$ 方法结合了目标攻击和几何感知正则化，以生成更隐蔽且有效的对抗性点云。此外，还提出了一个算法 $Geo_+A^3$-IterNormPro$，用于生成具有表面级别对抗性的形状。实验表明，这些方法在攻击成功率和难以察觉性方面均优于现有技术。

<!-- more -->

## TODO
