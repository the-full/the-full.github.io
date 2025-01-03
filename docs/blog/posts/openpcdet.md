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

    出于某些原因，最近又需要使用 OpenPCDet 进行模型的训练。具体来说，我们要在 waymo 数据集的点云上做一些修改来验证一件事。我的任务是使用 OpenPCDet 在修改后的数据集上训练一些常用的三维目标检测器做为基线。因此有了这篇笔记。

??? info "参考文献"

    TODO

<!-- more -->

## 一些基本概念

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
