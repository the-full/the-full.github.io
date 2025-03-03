---
draft: False 
date: 
  created: 2024-11-01
  updated: 2025-01-02
categories:
  - Misc
tags:
  - OpenPCDet
  - Waymo
authors:
  - zhazi
---

# 记录：waymo 数据预处理预到的问题

在基于 OpenPCDet 进行 Waymo 数据集上的模型训练和测试时，遇到了一些环境配置问题。经过多次尝试，最终发现解决方法其实很简单。由于尝试过程中环境多次变化，具体细节已难以一一还原，这里仅记录关键问题和解决方案。
<!-- more -->

!!! note "使用的环境"

    - python==3.9
    - cudatoolkit==11.3
    - pytorch==1.12.0

在这个环境下，OpenPCDet 的安装一切顺利，但到了安装 waymo-open-dataset 时，问题就开始变得复杂了。

## 尝试：waymo-open-dataset-tf-2-5-0 | 2-6-0

最初按照官方文档的指导，安装了 waymo-open-dataset-tf-2-5-0，随后又尝试了 2-6-0 版本，但都遇到了 numpy 版本冲突的问题。waymo-open-dataset 依赖特定版本的 tensorflow，而 tensorflow 又要求比 pytorch==1.12.0 更低的 numpy 版本。尝试降低 pytorch 版本无果后，我决定尝试更高版本的 waymo-open-dataset。

## 尝试：waymo-open-dataset-tf-2-11-0 | 2-12-0

由于 2-7-0 到 2-10-0 版本并不存在，我安装了 waymo-open-dataset-tf-2-11-0，随后也尝试了 2-12-0 版本。然而，这次在 `import tensorflow` 时却报错了：

``` txt title='错误信息'
AttributeError: module 'tensorflow.python.training.experimental.mixed_precision' has no attribute '_register_wrapper_optimizer_cls'
```

经过搜索，发现问题的根源是 keras 和 tensorflow 版本不匹配。建议的解决方案是：

```bash
pip uninstall keras
pip install -U tensorflow keras
```

但这样做会安装最新的 tensorflow 和 keras，又与 waymo-open-dataset 的版本要求不兼容。可是，在官方提供的 colab 环境中，明明可以正常运行，问题究竟出在哪里？


??? Example "走进科学时间"

    最终发现是 python 版本的问题。colab 使用的是 python==3.10，而我常用的 python==3.9 却无法正常运行。
