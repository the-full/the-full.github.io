---
draft: true
date:
  created: 2024-09-06
  updated: 2024-09-08
categories:
  - read paper
tags:
  - Thingking
---

!!! abstract "导言"

    在对抗攻击方法中，**C&W Attack** 是一个里程碑式的工作，不断的被后续工作借鉴与改进，三维对抗中第一篇 [工作](https://arxiv.org/abs/1809.07016) 正是在这一方法的基础上修改而来的，并且之后的研究总是或多或少的保留了一些 C&W Attack 的设计，这部分是出于研究的历史惯性，但更大的原因是人眼对点扰动的感知比对像素扰动的感知更敏感，而 C&W Attack 包含了很多优化扰动大小的设计。因此，在三维对抗中， **C&W Attack** 有着重要地位，然而，各方在从中借鉴和改进时往往是根据自己的需要对算法的各个设计进行取舍和微调，这导致尽管许多论文都宣称其是一种 C&W Attack，但在具体实现上总是存在一些差异，为后续工作进行公平比较带来了困难。因此，本文的希望对三维对抗中的经典的基于 C&W Attack 的攻击方法进行梳理，为后续研究开展提供清晰的认识。总的来说，本文只关注两方面问题：

    1. C&W Attack 的哪些设计被广泛使用? 针对这些设计的修改有哪些？
    2. 当论文宣称其是基于 C&W Attack 的方法时，我们应该关注哪些地方？

## 背景

### 对抗样本的经典定义

### 基于优化的对抗样本搜索方法

#### 建模为盒约束问题

#### 建模为双目标优化问题

### C&W Attack

#### 软约束

#### 超参二分搜索

#### 最小扰动记录

## 三维对抗中的 C&W Attack

### 基于 C&W Attack 的方法

#### AdvPC

#### kNN

#### GeoA^3

#### HiTADV

### 总结