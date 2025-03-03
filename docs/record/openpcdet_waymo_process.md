---
draft: false
date:
  created: 2025-02-19
  updated: 2025-02-22
tags:
  - OpenPCDet
  - Waymo
authors:
  - zhazi
---

# 记录：对 Waymo 数据集做加工

需要对 Waymo 数据集中的点云做一些加工，然后再用于训练模型。

## 拟定修改方案

Waymo 原数据存成 `.tfrecord`，为了尽量避免直接面对 TensforFlow，选择在 OpenPCDet 生成 **waymo_info** 的时候对读出的数据进行加工。

## 定位修改位置
首先，需要定位 OpenPCDet 从 `.tfrecord` 文件中加载点云信息的地方。依据 [Getting_STARTED](https://github.com/open-mmlab/OpenPCDet/blob/master/docs/GETTING_STARTED.md) 所述，要生成 `waymo_info` 需执行：

``` console
# only for single-frame setting
python -m pcdet.datasets.waymo.waymo_dataset --func create_waymo_infos \
    --cfg_file tools/cfgs/dataset_configs/waymo_dataset.yaml
```

在 `pcdet/datasets/waymo/waymo_dataset.py` 中可以找到：

```python title="pcdet/datasets/waymo/waymo_dataset.py" linenums="790" hl_lines="5 12"
    args = parser.parse_args()

    ROOT_DIR = (Path(__file__).resolve().parent / '../../../').resolve()

    if args.func == 'create_waymo_infos':
        try:
            yaml_config = yaml.safe_load(open(args.cfg_file), Loader=yaml.FullLoader)
        except:
            yaml_config = yaml.safe_load(open(args.cfg_file))
        dataset_cfg = EasyDict(yaml_config)
        dataset_cfg.PROCESSED_DATA_TAG = args.processed_data_tag
        create_waymo_infos(
            dataset_cfg=dataset_cfg,
            class_names=['Vehicle', 'Pedestrian', 'Cyclist'],
            data_path=ROOT_DIR / 'data' / 'waymo',
            save_path=ROOT_DIR / 'data' / 'waymo',
            raw_data_tag='raw_data',
            processed_data_tag=args.processed_data_tag,
            update_info_only=args.update_info_only
        )
```

因此，检查同文件中的 `create_waymo_infos` 函数，相关片段如下：

```python title="pcdet/datasets/waymo/waymo_dataset.py (create_waymo_finos)" linenums="702" hl_lines="4 17"
def create_waymo_infos(dataset_cfg, class_names, data_path, save_path,
                       raw_data_tag='raw_data', processed_data_tag='waymo_processed_data',
                       workers=min(16, multiprocessing.cpu_count()), update_info_only=False):
    dataset = WaymoDataset(
        dataset_cfg=dataset_cfg, class_names=class_names, root_path=data_path,
        training=False, logger=common_utils.create_logger()
    )
    train_split, val_split = 'train', 'val'

    train_filename = save_path / ('%s_infos_%s.pkl' % (processed_data_tag, train_split))
    val_filename = save_path / ('%s_infos_%s.pkl' % (processed_data_tag, val_split))

    os.environ["CUDA_VISIBLE_DEVICES"] = "-1"
    print('---------------Start to generate data infos---------------')

    dataset.set_split(train_split)
    waymo_infos_train = dataset.get_infos(
        raw_data_path=data_path / raw_data_tag,
        save_path=save_path / processed_data_tag, num_workers=workers, has_label=True,
        sampled_interval=1, update_info_only=update_info_only
    )
```

因此下一步检查 WaymoDataset 的 `get_infos` 方法，相关代码如下：

```python title="pcdet/datasets/waymo/waymo_dataset.py (WaymoDataset.get_infos)" linenums="174" hl_lines="6 7 17"
    def get_infos(self, raw_data_path, save_path, num_workers=multiprocessing.cpu_count(), has_label=True, sampled_interval=1, update_info_only=False):
        from . import waymo_utils
        print('---------------The waymo sample interval is %d, total sequecnes is %d-----------------'
              % (sampled_interval, len(self.sample_sequence_list)))

        process_single_sequence = partial(
            waymo_utils.process_single_sequence,
            save_path=save_path, sampled_interval=sampled_interval, has_label=has_label, update_info_only=update_info_only
        )
        sample_sequence_file_list = [
            self.check_sequence_name_with_all_version(raw_data_path / sequence_file)
            for sequence_file in self.sample_sequence_list
        ]

        # process_single_sequence(sample_sequence_file_list[0])
        with multiprocessing.Pool(num_workers) as p:
            sequence_infos = list(tqdm(p.imap(process_single_sequence, sample_sequence_file_list),
                                       total=len(sample_sequence_file_list)))

        all_sequences_infos = [item for infos in sequence_infos for item in infos]
        return all_sequences_infos
```

这里逻辑稍微麻烦一点，使用多进程并发执行 `process_single_sequence` 方法，每个进程处理一个序列的记录，后者是被 `functools.partial` 包装过的方法，真正执行的函数是 `waymo_utils` 中的 `process_single_sequence` 方法。因此，接下来检查 `process_single_sequence` 方法，相关内容如下：

```python title="pcdet/datasets/waymo/waymo_utils.py (process_single_sequence)" linenums="221"
    for cnt, data in enumerate(dataset):
        if cnt % sampled_interval != 0:
            continue
        # print(sequence_name, cnt)
        frame = dataset_pb2.Frame()
        frame.ParseFromString(bytearray(data.numpy()))
```

在这里将 `.tfrecord` 文件解析为 `Frame` 对象，其记录了这一帧中的所有信息。

```python linenums="228"
        info = {}
        pc_info = {'num_features': 5, 'lidar_sequence': sequence_name, 'sample_idx': cnt}
        info['point_cloud'] = pc_info

        info['frame_id'] = sequence_name + ('_%03d' % cnt)
```

**waymo_info** 中的点云信息记录了序列号和帧序号，这个坐标可以定位下一帧点云。

```python linenums="247" hl_lines="9 17"
        if has_label:
            annotations = generate_labels(frame, pose=pose)
            info['annos'] = annotations

        if update_info_only and sequence_infos_old is not None:
            assert info['frame_id'] == sequence_infos_old[cnt]['frame_id']
            num_points_of_each_lidar = sequence_infos_old[cnt]['num_points_of_each_lidar']
        else:
            num_points_of_each_lidar = save_lidar_points(
                frame, cur_save_dir / ('%04d.npy' % cnt), use_two_returns=use_two_returns
            )
        info['num_points_of_each_lidar'] = num_points_of_each_lidar

        sequence_infos.append(info)

    with open(pkl_file, 'wb') as f:
        pickle.dump(sequence_infos, f)

    print('Infos are saved to (sampled_interval=%d): %s' % (sampled_interval, pkl_file))
    return sequence_infos
```

标注框信息记录在 annos 字段中，点云数据通过 `save_lidar_points` 保存，保存路径可以通过序列号和帧序号确定，因此训练时先通过 point_cloud(或 frame_id) 字段定位保存的点云帧，然后读取点云数据。所以，下一步要检查的是 `save_lidar_points` 里面到底是怎么 save 的。相关代码片段如下：

```python title="pcdet/datasets/waymo/waymo_utils.py (save_lidar_points)" linenums="169" hl_lines="9"
def save_lidar_points(frame, cur_save_path, use_two_returns=True):
    ret_outputs = frame_utils.parse_range_image_and_camera_projection(frame)
    if len(ret_outputs) == 4:
        range_images, camera_projections, seg_labels, range_image_top_pose = ret_outputs
    else:
        assert len(ret_outputs) == 3
        range_images, camera_projections, range_image_top_pose = ret_outputs

    points, cp_points, points_in_NLZ_flag, points_intensity, points_elongation = convert_range_image_to_point_cloud(
        frame, range_images, camera_projections, range_image_top_pose, ri_index=(0, 1) if use_two_returns else (0,)
    )

    # 3d points in vehicle frame.
    points_all = np.concatenate(points, axis=0)
```

这里先从 `frame` 中解析出了 range_image 和一些其他信息，具体说明如下：

- range_images:  `dict{lider_name: [lidar_return_0, lidar_return_1]}`, 其中 `lidar_return_*` 是大小为 $[H, W, 4]$ 的 Range Image，每个像素对应一个点云中的一个点，四个通道的数据分别表示:
    - **0**, range: 点到雷达的距离。
    - **1**, intensity: 返回的激光的强度
    - **2**, elongation: 返回的激光的延伸率
    - **3**, NLZ: No Label Zooes，也即无标签区域
- camera_projections: 用于将点云投影到相机图像中，**在这里不需要关注**
- seg_labels: 语义标签，**在这里不需要关注**
- range_image_top_pose: 每个像素对应的自车姿态，能获得更精确的点云（Ref: [range_image_top_pose do what?](https://github.com/waymo-research/waymo-open-dataset/issues/51)）

然后，通过 `convert_range_image_to_point_cloud` 方法从 5 颗雷达分别返回的两张 Range Image 中提取出点云数据（自车坐标系下）。**要修改的地方就在这**，不过要注意除了点云坐标，还有 `NLZ, intensity, elongation` 三个特征信息需要处理。

??? info "更多信息"

    因此，接下来检查这个方法，相关代码如下：

    ```python title="pcdet/datasets/waymo/waymo_utils.py. (convert_range_image_to_point_cloud)" linenums="110" hl_lines="1 4"
        for c in calibrations:
            points_single, cp_points_single, points_NLZ_single, points_intensity_single, points_elongation_single \
                = [], [], [], [], []
            for cur_ri_index in ri_index:
    ```
    基本逻辑是双层遍历，遍历雷达(每个雷达对应有一个标定信息)，遍历每颗雷达两次返回的 Range Image，从每个 Range Image 中提取出点云数据，最后合并起来。

    ```python linenums="114"
                range_image = range_images[c.name][cur_ri_index]
                if len(c.beam_inclinations) == 0:  # pylint: disable=g-explicit-length-test
                    beam_inclinations = range_image_utils.compute_inclination(
                        tf.constant([c.beam_inclination_min, c.beam_inclination_max]),
                        height=range_image.shape.dims[0])
                else:
                    beam_inclinations = tf.constant(c.beam_inclinations)

                beam_inclinations = tf.reverse(beam_inclinations, axis=[-1])
                extrinsic = np.reshape(np.array(c.extrinsic.transform), [4, 4])
    ```

    要从 Range Image 中估计出点云，需要知道雷达的外参矩阵和每个激光发射器的倾角。

    ```python linenums="137" hl_lines="1"
                range_image_cartesian = range_image_utils.extract_point_cloud_from_range_image(
                    tf.expand_dims(range_image_tensor[..., 0], axis=0),
                    tf.expand_dims(extrinsic, axis=0),
                    tf.expand_dims(tf.convert_to_tensor(beam_inclinations), axis=0),
                    pixel_pose=pixel_pose_local,
                    frame_pose=frame_pose_local)
    ```

    `pixel_pose` 是可选的，通过该参数传入 `ranage_image_top_pose` 的信息，可以估计出更精确的点云坐标。接下来看 `extract_point_cloud_from_range_image` 方法，这是 Waymo 工具包中提供的方法：

    ```python title="waymo_open_dataset.range_image_utils (extract_point_cloud_from_range_image)" linenums="613" hl_lines="13"
    def extract_point_cloud_from_range_image(range_image,
                                            extrinsic,
                                            inclination,
                                            pixel_pose=None,
                                            frame_pose=None,
                                            dtype=tf.float32,
                                            scope=None):
      with tf.compat.v1.name_scope(
          scope, 'ExtractPointCloudFromRangeImage',
          [range_image, extrinsic, inclination, pixel_pose, frame_pose]):
        range_image_polar = compute_range_image_polar(
            range_image, extrinsic, inclination, dtype=dtype)
        range_image_cartesian = compute_range_image_cartesian(
            range_image_polar,
            extrinsic,
            pixel_pose=pixel_pose,
            frame_pose=frame_pose,
            dtype=dtype)
        return range_image_cartesian
    ```
    `compute_range_image_polar` 把点的 range 转为点的极坐标，接下来看 `compute_range_image_cartesian`，应该快结束了

    跑太远了，先停下吧


## 着手修改

为了保留原始的命令行为不变，考虑依据上面的函数调用过程，复制一批函数，名字后面加上后缀区分，具体来说，在 `waymo_dataset.py` 中添加如下代码：

```python title="pcdet/datasets/waymo/waymo_dataset.py" linenums="790" hl_lines="21-37"
    args = parser.parse_args()

    ROOT_DIR = (Path(__file__).resolve().parent / '../../../').resolve()

    if args.func == 'create_waymo_infos':
        try:
            yaml_config = yaml.safe_load(open(args.cfg_file), Loader=yaml.FullLoader)
        except:
            yaml_config = yaml.safe_load(open(args.cfg_file))
        dataset_cfg = EasyDict(yaml_config)
        dataset_cfg.PROCESSED_DATA_TAG = args.processed_data_tag
        create_waymo_infos(
            dataset_cfg=dataset_cfg,
            class_names=['Vehicle', 'Pedestrian', 'Cyclist'],
            data_path=ROOT_DIR / 'data' / 'waymo',
            save_path=ROOT_DIR / 'data' / 'waymo',
            raw_data_tag='raw_data',
            processed_data_tag=args.processed_data_tag,
            update_info_only=args.update_info_only
        )
    elif args.func == 'create_waymo_infos_new':
        try:
            yaml_config = yaml.safe_load(open(args.cfg_file), Loader=yaml.FullLoader)
        except:
            yaml_config = yaml.safe_load(open(args.cfg_file))
        dataset_cfg = EasyDict(yaml_config)
        dataset_cfg.PROCESSED_DATA_TAG = args.processed_data_tag
        create_waymo_infos_new(
            dataset_cfg=dataset_cfg,
            class_names=['Vehicle', 'Pedestrian', 'Cyclist'],
            # NOTE: 直接设置 data_path = ... / 'waymo', 会导致 567 行处相对路径无法计算
            data_path=ROOT_DIR / 'data' / 'waymo_new',
            save_path=ROOT_DIR / 'data' / 'waymo_new',
            raw_data_tag='raw_data',
            processed_data_tag=args.processed_data_tag,
            update_info_only=args.update_info_only
        )
```

然后按照前面梳理的调用关系，依次复制一批函数，在 `save_lidar_points_new` 中对提取出的点云进行修改。最后运行：

``` console title='OpenPCDet 根目录下'
mkdir data/waymo_new
ln -s $(pwd)/data/waymo/raw_data data/waymo_new/raw_data # 创建软链接，毕竟数据集很大

# only for single-frame setting
python -m pcdet.datasets.waymo.waymo_dataset --func create_waymo_infos_new \
    --cfg_file tools/cfgs/dataset_configs/waymo_dataset.yaml
```
