---
draft: true
date:
  created: 2024-09-10
  updated: 2025-01-02
categories:
  - Learning
tags:
  - Sparse Tensor
authors:
  - zhazi
---

# 稀疏卷积技术初探

!!! abstract ""

    最近研究的内容涉及到稀疏张量及其相关操作，主要是 **spconv** 和 **MinkowskiEngine** 这两个库的使用，故在此记录。本文只关注三件事：

    - 稀疏操作的原理
    - 怎么使用 spconv / MinkowskiEngine
    - spconv 和 MinkowskiEngine 的异同

??? info "参考资料"

    - spconv ：[github](https://github.com/traveller59/spconv/tree/master)
    - MinkowskiEngine：[github](https://github.com/NVIDIA/MinkowskiEngine)
    - https://nvidia.github.io/MinkowskiEngine/tutorial/sparse_tensor_basic.html
    - https://github.com/traveller59/spconv/blob/master/docs/USAGE.md
    - https://pytorch.org/docs/stable/sparse.html#sparse-coo-docs

<!-- more -->

## 稀疏张量表示

### PyTorch: Sparse COO tensors

PyTorch 中，稀疏张量是和通常使用的张量相对的概念，后者在此上下文中称为稠密张量。PyTorch 支持一种 Coordinate List 格式的稀疏张量，存储格式为 `(indices, values, size)`，翻译一下就是：在 `size` 大小的张量中，由二维张量 `indices` 中第 $i$ 个元素所表示的坐标处，其上的值为 `values` 中存储的第 $i$ 个的稠密张量。这种下面的示例说明了稀疏张量与稠密张量间的转化关系。

```python linenums="1", title='example', hl_lines="5-8 11-12"
>>> i = [[0, 1, 1],
>>>      [2, 0, 2]]
>>> v =  [3, 4, 5]
>>> s = torch.sparse_coo_tensor(i, v, (2, 3))
tensor(indices=tensor([[0, 1, 1],
                       [2, 0, 2]]),
       values=tensor([3, 4, 5]),
       size=(2, 3), nnz=3, layout=torch.sparse_coo)
>>> s
>>> s.to_dense()
tensor([[0, 0, 3],
        [4, 0, 5]])
```

### spconv: Sparse Conv Tensors

`spconv` 库特化了 PyTorch 的稀疏张量表示，提供了针对点云数据的的**稀疏 3D卷积**操作。在 `spconv` 中，数据被表示为 `(batch, feature, <index>)`，其中 `feature` 是特征向量。其中 `<index>` 表示数据的索引，通常为 3 (比如体素的 $x, y, z$ 整数坐标)。 `spconv` 特化的稀疏张量 **SparseConvTensor** 的存储格式为`(faetures, indices, spatial_shape, batch_size)`，其中 `features` 对应于 Sparse COO tensor 的 `values`， 其元素被限制为一维张量；`spatial_shape` 和 `batch_size` 对应于 Sparse COO tensor 的 `size`，将 `batch_size` 维度分离在外面以方便卷积操作。更详细的描述如下：

- **features**：形状为 `[N, num_channels]` ，其中 `N` 是非零元素的数量，`num_channels` 是每个非零元素的特征维度。
- **indices**：形状为 `[N, ndim + 1]` ，其中 `ndim` 通常是3（对于体素化的点云数据），表示每个非零元素的索引位置，多出的一维是 Batch 的索引。
- **spatial_shape**：形状为 `[ndim]` ，描述了稀疏张量的空间维度。
- **batch_size**：一个整数，表示 Batch 的大小。

举个例子，考虑一批点云数据，我们对点云数据进行体素化，假设在 $x, y, z$ 方向上分别有 15、20 和 3 个体素，每个体素的特征为：所包含的平均坐标 + 包含的点数，因此，如果体素中不包含任何点，其特征为全零，假设特征非零的体素数量为 30 个，再假设 `batch_size=5`, 那么有：

- **features**：形状为 `[30, 4]` ，因为非零特征数量 `N = 30`，每个非零特征的维度 `num_channels = 3 (xyz 坐标) + 1 (点数)` 。
- **indices**：形状为 `[30, 3 + 1]` ，因为坐标的索引维度 `ndim = 3`。
- **spatial_shape**：形状为 `[3]` 具体值为 `[15, 20, 3]` 。
- **batch_size**：其值为 `5`。

进而我们可以将数据转为稀疏张量，如下所示：

```python title="example", linenums="1"
import spconv.pytorch as spconv
import torch

features = torch.randn(30, 4)  # 30 个非空体素，每个体素的特征维度为 4

# 30 个数据点的索引，包括 batch 索引和 坐标索引
x_indices = torch.randint(0, 15, (30,)).int()
y_indices = torch.randint(0, 20, (30,)).int()
z_indices = torch.randint(0, 3, (30,)).int()
b_indices = torch.randint(0, 5, (30,)).int()
indices = torch.stack((b_indices, x_indices, y_indices, z_indices), dim=-1)

spatial_shape = (15, 20, 3)  # 空间维度
batch_size = 5  # 批次大小

# 创建 SparseConvTensor
x = spconv.SparseConvTensor(features, indices, spatial_shape, batch_size)

x_dense = x.dense()   # 转换为 NCDHW 格式的密集张量
```
