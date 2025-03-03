---
draft: false
date:
  created: 2025-02-17
  updated: 2025-02-18
categories:
  - Misc
tags:
  - OpenPCDet
  - Waymo
authors:
  - zhazi
---

# 记录：基于 OpenPCDet 在 Waymo 数据集上训练模型

因数据量过大，且下载的 `training_0000.tar, training_0001.tar` 记录损坏, 取 `training_0002.tar, training_0003.tar, training_0004.tar` 约 10% 数据进行训练。

## 数据集准备
从官方下载地址 [Waymo Open Dataset](https://waymo.com/open/download/) 下载数据压缩包，然后解压到 `waymo/raw_data` 目录下。

## 安装 OpenPCDet

基于前期经验([记录：waymo 数据预处理预到的问题](./waymo_perprocess.md))，安装方案如下：

```console
conda create -n pcdet pip python==3.10
conda activate pcdet
conda install pytorch==1.12.1 torchvision==0.13.1 torchaudio==0.12.1 cudatoolkit=11.3 -c pytorch
python setup.py develop  # OpenPCDet 项目根目录下执行
pip3 install pyquaternion
pip3 install opencv-python
pip3 install av2
pip3 install numpy==1.*
pip3 install kornia==0.5.*
pip3 install waymo-open-dataset-tf-2-12-0 --user
# pip3 install numba==0.60.0         # 最后一步中的报错处理，不确定是否需要
pip3 install spconv-cu113==2.*       # train 模型的时候需要
pip3 install numba==0.60.0           # 默认的 numba==0.61.0 与 numpy==1.23(tf 2.12.0 要求) 冲突
conda install pytorch-scatter -c pyg # 训练 voxel rcnn 需要
```

## 数据预处理

只考虑单帧检测，依据 [Getting_STARTED](https://github.com/open-mmlab/OpenPCDet/blob/master/docs/GETTING_STARTED.md) 所述，执行：

``` console
# only for single-frame setting
python -m pcdet.datasets.waymo.waymo_dataset --func create_waymo_infos \
    --cfg_file tools/cfgs/dataset_configs/waymo_dataset.yaml
```

## 训练

需要训练 `CenterPoint` 和 `PV-RCNN` 模型，依据 [Getting_STARTED](https://github.com/open-mmlab/OpenPCDet/blob/master/docs/GETTING_STARTED.md) 所述，执行：

``` console
CUDA_VISIBLE_DEVICES=0 python train.py --cfg_file ./cfgs/waymo_models/centerpoint.yaml
CUDA_VISIBLE_DEVICES=1 python train.py --cfg_file ./cfgs/waymo_models/pv_rcnn.yaml
```
