---
draft: true
date: 
  created: 2024-10-31
  updated: 2025-01-02
categories:
  - Misc
  - Learning
tags:
  - 3D Detection 
  - OpenPCDet
authors:
  - zhazi
---

# 笔记：OpenPCDet 相关

!!! abstract ""

    出于项目需求，我们需要在自己的数据集上训练一些 3D 检测器用于做为基线，计划基于 OpenPCDet 来实现。初步需求包括三部分：
    
    - 需求1：使用 OpenPCDet 在自定义的数据集上训练
    - 需求2：使用 OpenPCDet 在自定义的数据集上评估训练好的模型，要支持 Kitti Style 和 Waymo Style 的评估
    - 需求3（低优先级）： 使用 OpenPCDet 在 Waymo 数据集上训练

    没想到为了添加 Waymo Style 的评估，花了很多时间在 OpenPCDet 的实现细节上:face_with_spiral_eyes:，不过也因此对 OpenPCDet 有了一些新的认识，就用这篇笔记记录一下相关的经验总结吧。

    ??? info "参考文献"

        - [OpenPCDet 官方仓库](https://github.com/open-mmlab/OpenPCDet)
<!-- more -->

## 建立初步印象

OpenMMLab 的开源库常被诟病的一个点是对初学者不够友好，修改起来难度较大。在实际使用 OpenPCDet 的过程中，我也确实有过类似的体验。

??? warning "网络评论（不代表本人观点）"

    摘自互联网：
    > - 学术宝藏，工业垃圾
    > - 很差劲，完全看不懂，一套模型一套参数，找个配置文件能找一个多小时，模型的代码被模块化到各个角落，可读性非常差，如果原作者有版本还是读作者的比较好  
    > - 确实，调他这配置调的我火大，就不能像torch那样把接口做出来我们自己写代码调用吗？非要搞他那吊配置  
    > - 可读性很差，如果有作者版本的代码，还是建议看作者的源代码，这个框架很难用  
    > - 对于初学者来说和核心代码不相干的东西有点多，前期看起来有点累
    > - 使用比较受限制，比如说只能完成通常目标检测 （读取一张图片，输出bbox的检测结果）  
    > - 如果需要进行其他比较小众的任务，可能在pipline，以及其他的各个环节都需要进行一些更改  

就个人的体验来说，几次接触 OpenPCDet 遇到的问题包括：

- 搭环境困难。安装 pcdet 包时就遇到一些问题，后面考虑 Waymo 数据集的时候又遇到一些问题，因为 Waymo 数据集的工具包基于 Tensorflow，后者的引入导致 numpy 版本的选择更加困难，正如[杂记](./waymo_open_dataset.md)所记录的那样。
- 无从下手。什么都要写配置，但又没有文档告知每条配置是什么意思，不知道要达成目的应该修改哪条配置。
- 不敢下手。不懂配置的情况下，修改起来很容易出错，模块化的设计导致报错难以排查，搞不明白配置上的修改是如何导致某个函数出错的。
- 读不懂代码。尝试理解配置到实例的过程，以及实例化后训练和测试过程，不知道从合处读起，相关方法往往跨越个函数来回调用，需要在多个文件中跳转，经常陷入到一些非核心代码中。

因此，这里先花点时间构建一个正确的初步印象，以便之后在查看解决方案时能够做到知其然也知其所以然。

### 文件结构

OpenPCDet 采用模块化和集中化的文件组织结构，其中数据集相关文件统一存放在 `data` 目录下，训练、测试和评估的脚本代码集中在 `tools` 目录中，而支持数据管理、模型训练和点云操作的核心代码则位于 `pcdet` 目录下。更多信息如下所示：

```console
OpenPCDet
├── data                 # 存放数据集相关文件，包括原始数据、预处理后的数据、标注信息等
│   ├── argo2
│   ├── custom_data      # 自定义数据集相关文件
│   ├── kitti
│   ├── lyft
│   ├── once
│   └── waymo
├── docker               # 提供 Docker 镜像和相关配置，用于快速搭建开发环境
├── docs                 # 存放项目文档，包括使用指南、API 文档、教程等
├── pcdet                # 核心代码库，包含模型定义、数据集处理、数据加载器、三维点云操作等核心功能
│   ├── datasets         # 数据集处理相关代码
│   ├── models           # 模型定义相关代码
│   ├── ops              # 点云操作相关代码（Cuda 代码
│   ├── utils            # 工具函数
│   └── ...
├── tools                # 存放训练、测试、评估等工具脚本，以及配置文件
│   ├── cfgs             # 存放数据集、模型配置 
│   └── ...
└── ...                  # 其他辅助文件和目录
```

### 设计模式

1. 数据-模型分离架构

  
  这里的数据和模型是在软件设计和开发中的概念，分离的是数据的存储和处理逻辑。

  从代码实现来看，检测器的实例化需要传入一个数据集实例，两者并不是分开构建的。而构建检测器实例时传入的数据集实例是可以替换的，因此能支持自定义数据集。

2. 统一点云坐标系
3. 统一的标注框格式
4. 模块化设计

## 理解配置系统

TODO

初次接触 OpenPCDet，很不习惯这种通过写配置来设置模型的，配置系统是带来复杂性

讲解时会尽量结合代码进行，以便后续阅读代码时也能事半功倍。

### 如何理解 3D 检测器？

??? note "模型超参数 & 数据集元数据"

    模型参数通常分为两类：

    1. **可训练参数**：这些是模型在训练过程中通过优化算法（如梯度下降）自动学习和调整的参数。例如，神经网络中的权重和偏置就是典型的可训练参数。
    2. **超参数**：这些是模型训练前需要手动设置的参数，它们控制着模型的结构和训练过程。例如，学习率、批量大小、网络层数等都是超参数。超参数的选择对模型性能有重要影响，通常需要通过实验或经验来确定。

    数据集数据通常也包含两部分：

    1. **样本数据**：这是模型训练和评估所使用的实际样本。例如，在3D检测任务中，数据可能包括点云、图像、标注框等。
    2. **元数据**：这是描述数据的数据，通常包含数据的结构、格式、来源等信息。例如，数据集的类别标签、数据的分割方式（训练集、验证集、测试集）、数据的预处理方法等都属于元数据。元数据对于理解和使用数据集非常重要，尤其是在处理自定义数据集时

    模型超参数和数据集元数据之间往往存在紧密的关联，这种关联导致模型和数据集之间存在难以避免的耦合。例如：

    - **分类模型的分类头**：分类模型的输出层（分类头）的维度通常需要与数据集的类别数一致。这意味着如果数据集的类别数发生变化，模型的分类头结构也需要相应调整。
    - **目标检测模型的锚点设计**：目标检测模型中使用的锚点（anchors）通常需要根据数据集中目标的大小和宽高比进行设计。不同数据集的目标分布可能差异较大，因此锚点设计需要针对具体数据集进行调整。
    - **数据预处理参数**：数据预处理步骤（如归一化、数据增强等）的参数通常需要根据数据集的统计特性（如均值、方差、数据分布等）进行设置。这些参数直接影响模型的输入数据分布，进而影响模型的训练效果。

- 我们通常将模型实例和数据集实例分开构建，但在 OpenPCDet 中，模型实例化时需要传入数据集实例
- 一个检测器实例是一个训练好的网络，它不仅取决于它的网络结构，还取决于用来训练它的那个数据集。
- 1. 状态机：空壳 ==build_network + add_module ==> 网络 ==train(dataset)==> 训练好的网络 ==mode切换==> 推理使用的网络

!!! note "nn.Module.add_module 方法"

    在 PyTorch 中，`add_module` 是 `torch.nn.Module` 类的一个方法，用于向模块动态地添加子模块。这个方法在创建复杂的神经网络结构时特别有用，尤其是当需要在**运行时**动态地构建模型时。

### 如何理解

## TODO

CustomDataset 中包含了几个很奇怪的函数 `get_infos`, `create_groundtruth_database`, `create_custom_infos`，用来创建数据集的 info file 和 Database file。

### Info File 

- TODO: 图
- INFO 按点云帧来记录信息，每帧点云的信息被记录为一个字典，所有帧构成一个字典列表
- point_cloud 记录了 LiDAR 点云的路径
- annos 将这一帧的所有标注聚合成一个字典
- 用于在 getitem 中快速读取数据


### Database Info File

- TODO: 图
- DB_INFO 按实例来记录信息，DB_INFO.keys() 是所有的类别名，每个实例被记录到对应的类别名下。
- 比如 bbox 信息、包含点数、检测难度等信息，不同数据集之间的记录项可能略有差异
- 用于在 evaluate 中快速评估结果

所以，为了训练，我需要我的自定义数据集生成 info file，这需要用到 `custom_dataset.py` 中提供的 `create_custom_infos` 函数。标准用法应该是 `python -m xxxx/custom_dataset.py`。这个函数的具体定义如下：


``` py title='custom_dataset.py' hl_lines=15,23
def create_custom_infos(dataset_cfg, class_names, data_path, save_path, workers=4):
    dataset = CustomDataset(
        dataset_cfg=dataset_cfg, class_names=class_names, root_path=data_path,
        training=False, logger=common_utils.create_logger()
    )
    train_split, val_split = 'train', 'val'
    num_features = len(dataset_cfg.POINT_FEATURE_ENCODING.src_feature_list)

    train_filename = save_path / ('custom_infos_%s.pkl' % train_split)
    val_filename = save_path / ('custom_infos_%s.pkl' % val_split)

    print('------------------------Start to generate data infos------------------------')

    dataset.set_split(train_split)
    custom_infos_train = dataset.get_infos(
        class_names, num_workers=workers, has_label=True, num_features=num_features
    )
    with open(train_filename, 'wb') as f:
        pickle.dump(custom_infos_train, f)
    print('Custom info train file is saved to %s' % train_filename)

    dataset.set_split(val_split)
    custom_infos_val = dataset.get_infos(
        class_names, num_workers=workers, has_label=True, num_features=num_features
    )
    with open(val_filename, 'wb') as f:
        pickle.dump(custom_infos_val, f)
    print('Custom info train file is saved to %s' % val_filename)

    print('------------------------Start create groundtruth database for data augmentation------------------------')
    dataset.set_split(train_split)
    dataset.create_groundtruth_database(train_filename, split=train_split)
    print('------------------------Data preparation done------------------------')
```
代码逻辑很简单，先拼出文件路径，然后使用 dataset.get_infos 生成要存储的数据，最后写入对应的文件中。这里实例出的 dataset 的作用只是提供一个 get_infos 函数。这个函数的定义如下：

```py title='custom_dataset.py' hl_lines=12
def get_infos(self, class_names, num_workers=4, has_label=True, sample_id_list=None, num_features=4):
    import concurrent.futures as futures

    def process_single_scene(sample_idx):
        print('%s sample_idx: %s' % (self.split, sample_idx))
        info = {}
        pc_info = {'num_features': num_features, 'lidar_idx': sample_idx}
        info['point_cloud'] = pc_info

        if has_label:
            annotations = {}
            gt_boxes_lidar, name = self.get_label(sample_idx)
            annotations['name'] = name
            annotations['gt_boxes_lidar'] = gt_boxes_lidar[:, :7]
            info['annos'] = annotations

        return info

    sample_id_list = sample_id_list if sample_id_list is not None else self.sample_id_list

    # create a thread pool to improve the velocity
    with futures.ThreadPoolExecutor(num_workers) as executor:
        infos = executor.map(process_single_scene, sample_id_list)
    return list(infos)
```

逻辑也很简单，就是依次生成 self.sample_id_list 中的编号对应的点云帧的表单信息。只需要搞明白 self.sample_id_list 和 self.get_label 就好了。self.sample_id_list 的定义有两处，两处的代码是一样的如下:
```py title='self.sample_id_list'
self.sample_id_list = [x.strip() for x in open(split_dir).readlines()] if os.path.exists(split_dir) else None
```
逻辑很简单，所以这个属性就是 split 文件里面存的内容。对于 self.get_label 其定义如下：

```py title='get_label'
def get_label(self, idx):
    label_file = self.root_path / 'labels' / ('%s.txt' % idx)
    assert label_file.exists()
    with open(label_file, 'r') as f:
        lines = f.readlines()

    # [N, 8]: (x y z dx dy dz heading_angle category_id)
    gt_boxes = []
    gt_names = []
    for line in lines:
        line_list = line.strip().split(' ')
        gt_boxes.append(line_list[:-1])
        gt_names.append(line_list[-1])

    return np.array(gt_boxes, dtype=np.float32), np.array(gt_names)
```
这个逻辑更简单了，依据编号，拼出标签文件，然后读取，返回。


略
