---
draft: true
date:
  created: 2024-09-06
  updated: 2024-09-08
  updated: 2024-11-24
  updated: 2025-01-02
categories:
  - Summary
tags:
  - Rethingking
  - Adversarial Attack
authors:
  - zhazi
---

# 反思三维点云对抗: C&W Attack

!!! abstract ""

    在对抗攻击方法中，**C&W Attack** 是一项具有里程碑意义的工作，后续研究不断从中借鉴并进行改进。三维对抗领域的首篇[工作](https://arxiv.org/abs/1809.07016) 就是在此方法的基础上进行修改的，而之后的研究也多多少少保留了一些 **C&W Attack** 的设计元素。这一现象部分源于研究的历史惯性(~~在前人代码上改进~~)，部分原因在于人类对点扰动的感知比对像素扰动更为敏感，而 **C&W Attack** 中包含了许多针对扰动大小优化的设计。此外，这种方法的相对“复杂性”也为后续工作提供了更多可改进的空间（~~适合水论文~~）。

    由此可见，**C&W Attack** 在三维对抗中具有重要地位。然而，各研究在借鉴和改进该方法时，通常会根据自身需求对算法的设计进行取舍与调整。这种灵活性导致尽管许多论文都声称基于 **C&W Attack**，但具体实现上常存在一定差异，从而为后续工作的公平比较带来了挑战。因此，本文旨在梳理三维对抗中经典的基于 **C&W Attack** 的攻击方法，为后续研究提供清晰的参考框架。具体而言，本文关注于以下三方面问题：

    1. 怎么理解 C&W Attack ? 它包含了哪些设计？
    1. C&W Attack 的哪些设计被广泛使用? 针对这些设计的修改有哪些？
    2. 当论文宣称其是基于 C&W Attack 的方法时，我们应该关注哪些地方？

??? info "参考文献"

    TODO
<!-- more -->

## 基础知识

### 对抗样本的经典定义

对抗样本的经典定义是主要包含两方面：

1. **对抗性（Adversarial）**：对抗样本 $x^{\text{adv}}$ 能够误导目标模型，使其输出错误的预测结果。
2. **不可感知性（Imperceptible）**：对于人类而言，很难判断输入是否被扰动，即对抗样本与原始输入在感知上几乎无法区分。

### 基于优化的对抗样本搜索方法

基于优化的方法围绕上述定义构建一个优化问题，然后通过求解优化问题的方法来制作对抗样本。主流的构建方式可以分为两种：建模为盒约束问题 和 建模为双目标优化问题。

#### 盒约束问题

在这一类方法中，攻击者将对抗样本建模为一个具有盒约束的优化问题：

$$
  \begin{equation}
  \begin{aligned}
      \underset{\mathbf{x}^{\text{adv}}}{\arg\max}\quad &{\mathcal{J}(f(\mathbf{x}^{\text{adv}}), y)} \\
      \textrm{s.t.}\quad &\Vert \mathbf{x}^{\text{adv}} - \mathbf{x}^{\text{ori}}\Vert_{\infty} < \epsilon,
  \end{aligned}
  \end{equation}
$$

其中 $f$ 是目标模型，$\mathcal{J}(\cdot, \cdot)$ 表示交叉熵损失函数，$\mathbf{x}^{\text{ori}}$ 是具有真实标签 $y$ 的干净样本。攻击者的目标是在 $\mathbf{x}^{\text{ori}}$ 的 $\ell_\infty$ 邻域中找到一个对抗样本 $\mathbf{x}^{\text{adv}}$使得 $\mathbf{x}^{\text{adv}}$ 能误导 $f$ 。

在实践中，通常将扰动 ($\Delta = \mathbf{x}^{\text{adv}} - \mathbf{x}^{\text{ori}}$) 作为优化变量，并通过迭代算法来求解这个优化问题。例如，I-FGSM 中的扰动迭代公式为：

$$
  \Delta\_{i+1} = \Delta_i + \text{Clip}_\epsilon \left\{ \alpha \cdot \text{Sign} \left( \nabla \mathcal{J}(f(\mathbf{x}^{\text{ori}} + \Delta_i), y) \right) \right\}.
$$

其中，$\text{Clip}_{\epsilon}{\cdot}$ 表示将超出预算 $epsilon$ 的部分裁剪到 $epsilon$ 的操作（也被称为投影），$alpha$ 是迭代步长。对于一个 $T$ 步的迭代过程，通常设置 $\alpha=\epsilon / T$

#### 双目标优化问题

另一类方式是将对抗样本的两方面要求都视为优化目标，即攻击者将将对抗样本建模为一个双目标优化问题。C&W Attack 就属于此类方法，原论文中将对抗性目标建模为：

$$
\left(\max_{i\neq t}(f(\mathbf{x}^{\text{adv}})_{i})-f(\mathbf{x}^{\text{adv}})_{t}\right)^{+}
$$

其中，$f(\cdot)_{i}$ 表示 $f$ 输出的预测分数中第 $i$ 类（从 0 计数）的置信度。$(\cdot)^{+}$ 表示分段函数，其将小于 0 的部分置为 0（也就是 ReLU)。

另一方面，不可感知性目标被简单地建模为：

$$
\Vert \mathbf{x}^{\text{adv}} - \mathbf{x}^{\text{ori}} \Vert_{p}
$$

其中，通过（图像中）取 $p=2$.

总之，对抗样本被表述为如下双目标优化问题：

$$
\begin{align*}
\min_{\mathbf{x}^{\text{adv}}} & \quad \left(\max_{i\neq t}(f(\mathbf{x}^{\text{adv}})_{i})-f(\mathbf{x}^{\text{adv}})_{t}\right)^{+} \\
\min_{\mathbf{x}^{\text{adv}}} & \quad \Vert \mathbf{x}^{\text{adv}} - \mathbf{x} \Vert_{p} \\
\end{align*}
$$

接下来，我们来看 C&W Attack 是怎么求解这个问题的。

## C&W Attack

### 求解思路

C&W Attack 求解上述双目标优化问题的思路是比较常规的：通过加权将双目标转化为单目标，然后通过搜索技术寻找最好的权值。具体来说，
C&W Attack 引入了一个权重系数 $c$ 将双目标优化转化为如下形式的单目标优化：

$$
\underset{\mathbf{x}^{\text{adv}}}{\arg\min} \quad c \cdot \left(\max_{i\neq t}(f(\mathbf{x}^{\text{adv}})_{i})-f(\mathbf{x}^{\text{adv}})_{t}\right)^{+} + \Vert \mathbf{x}^{\text{adv}} - \mathbf{x}^{\text{ori}} \Vert_{p}
$$

!!! note "注意"

    权重 $c$ 是给对抗性目标加权，这与我们后面将提到的三维攻击中的常见设置不同。

在解决上述方程时，较大的 $c$ 会使得所得到的解 $\mathbf{x}^{\text{adv}}$ 更加不可感知，但同时也使得优化问题可行集更小，增加了求解失败的概率。相反，较小的 $c$ 提供了更大的可行集，但代价是放宽了对不可感知性的约束。因此 C&W Attack 又引入了参数搜索技术，使用二分搜索来寻找最优的 $c$。下面我们结合代码来品鉴一下 C&W Attack 中的一些重要设计。

#### 软约束

#### 超参二分搜索

#### 记录最小扰动

## 三维对抗中的 C&W Attack

### 基于 C&W Attack 的方法

#### AdvPC

#### kNN

#### GeoA^3

#### HiTADV

### 总结

$$
$$
