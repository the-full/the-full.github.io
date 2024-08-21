# 关于 ModelNet40 数据集

!!! Abstract "导言"

    ModelNet40 是三维对抗攻防中一个非常常用的数据集，绝大多数方法都会在该数据集上进行评估。然而，在阅读相关论文时，会发现这个数据集有很多变体，通常是前人工作中预处理过后的版本，这导致论文（代码）中提到 ModelNet40 数据集时往往带有多意性。因此，在这里对相关变体进行记录和区分。本文将忽略数据集的存储细节也不会包括如何实现 DataSet 类来读取数据的内容。
    注：数据集构建细节部分是在 ai 对论文的总结上修改而来。

## Root: ModelNet

ModelNet 的[官网](https://modelnet.cs.princeton.edu/) 包含两个版本：ModelNet10 和 ModelNet40。每个数据是代表一个 CAD 模型，以对象文件格式(OFF, Object File Format) 存储，其中 ModelNet10 是手动对齐过的。后来 Orientation-boosted Voxel Nets for 3D Object Recognition 的作者提供了 ModelNet40 的对齐版本。数据集组织方式如下图所示：

![ModelNet数据集文件结构](./images/ModelNet文件架构示意图.png){style: width=100% height=auto}

??? tip "构建细节"

    1. **数据收集**：从 3D Warehouse 和 Yobi3D 搜索引擎（索引了 261 个 CAD 模型网站）下载 3D CAD 模型。

    2. **类别选择**：基于 SUN 数据库中的常见对象类别进行查询，选择每个类别至少包含 20 个实例的类别，排除了搜索结果太少的类别。

    3. **数据清洗**：
        1. **移除误分类模型**：由 Amazon Mechanical Turk 上的工作人员确认类别标签是否与模型匹配以识别和去除错误分类的模型。
        2. **移除无关对象**：作者手动检查每个3D模型，从每个CAD模型中移除不相关对象（例如，地板、缩略图、站在物体旁边的人物等），确保每个网格模型只包含属于标注类别的一个对象。
        3. **移除不现实的模型**：包括过于简化的模型或仅包含对象图片的模型。
        4. **去除重复模型**：确保数据集中的模型不重复。

## Version 1: ModelNet40 Hdf5

该版本来自于

??? tip "构建细节"

    - **数据集划分**：将数据集分为训练集（9,843 个模型）和测试集（2,468 个模型）。
    - **数据预处理**：
        1. **数据采样**：从每个 3D 模型的网格面上均匀采样 1024 个点，采样基于网格面积进行。
        2. **数据归一化**：将采样的点云归一化到单位球内。

## Version 2: ModelNet40 Resampled

## Version 3: ModelNet10 250 sample
