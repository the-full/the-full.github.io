---
draft: true
date: 2024-11-01
categories:
  - misc
tags:
  - OpenPCDet
  - Waymo
---

# 与 OpenPCDet 搏斗：waymo-open-dataset

!!! abstract "导言"

    在使用 OpenPCDet 进行 Waymo 数据集上模型的训练和测试时，遇到了一些问题，结果发现解决方法很简单，因此记录之。
    注：由于尝试过程中环境多次变化，具体过程已经难以一一记录，只能记录下一些片段。

<!-- more -->

我最开始使用的环境是：python==3.9, cudatoolkit==11.3, pytorch==1.12.0, 在这个环境下顺利安装了 OpenPCDet，然后在安装 waymo-open-dataset 时事情就变的抽象了起来。

### waymo-open-dataset-tf-2-5-0 | 2-6-0

一开始按文档指导，安装了 waymo-open-dataset-tf-2-5-0，后面又尝试了 2-6-0, 都是发生了 numpy 版本冲突，waymo-open-dataset 要求对应版本的 tensorflow，后者要求的 numpy 版本都比 pytorch==1.12.0 要求的版本更低，尝试调低 pytorch 版本无果后，决定调高 waymo-open-dataset 的版本。

### waymo-open-dataset-tf-2-11-0 | 2-12-0

2-7-0 到 2-10-0 都不存在，因此安装了 waymo-open-dataset-tf-2-11-0，后面也尝试了 2-6-0，这回是都在 `import tensorflow` 时报错：

``` txt title='fuck'
AttributeError: module 'tensorflow.python.training.experimental.mixed_precision' has no attribute '_register_wrapper_optimizer_cls'
```
查到的解决方案说是 keras 和 tensorflow 版本不匹配，建议：

```bash title='just try this'
pip uninstall keras
pip install -U tensorflow keras
```

这样会安装最新的 tensorflow 和 keras，又和 waymo-open-dataset 不匹配了。但是，在官方提供了 colab 上我明明成功运行了，所以问题出在哪里？


!!! warning "走进科学时间"

    最后发现是 python 版本问题，colab 上使用的是 python==3.10, 而我最常用的 python==3.9 不行。。。。
