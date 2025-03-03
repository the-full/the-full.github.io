---
draft: true
date:
  created: 2025-01-22
categories:
  - Misc
tags:
  - OpenPCDet
  - Waymo
authors:
  - zhazi
---

# 记录：基于 OpenPCDet 在 Waymo 数据集上训练模型

补充：数据量过大，取 `training_0000.tar, training_0001.tar, training_0002.tar` , 约 10% 数据进行训练。
<!-- more -->

## 数据集准备
在官方下载地址 [Waymo Open Dataset](https://waymo.com/open/download/) 下载对应的压缩包，解压到 `waymo` 文件夹下。

```console title="解压数据"
tar -xvf ./training/training_0000.tar -C ./waymo
```


## 数据集预处理

## 模型训练

