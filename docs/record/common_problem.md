---
draft: false
date:
  created: 2025-02-17
categories:
  - Memo
authors:
  - zhazi
---

# 记录：小问题归档

记录遇见的一些小问题。
<!-- more -->

## 问题 1

### 报错信息
```console
fatal: detected dubious ownership in repository at '<REDACTED_PATH>'
To add an exception for this directory, call:

        git config --global --add safe.directory <REDACTED_PATH>
```
### 解释

该信息是 Git 发出的安全警告，表明在 <REDACTED_PATH> 目录下的 Git 仓库检测到了可疑的所有权。这意味着该仓库的文件所有权或权限设置不符合 Git 的安全预期，可能存在安全风险。通常情况是，Git 检测到该仓库中的某些文件的所有者与当前用户不一致，因此会触发此错误。解决方案包括更改文件的所有权，或按照提示信息添加该目录为信任目录。

### 解决方案
```console
  git config --global --add safe.directory <REDACTED_PATH>
```

## 问题 2

### 报错信息
```console
RuntimeError:
The detected CUDA version (12.2) mismatches the version that was used to compile
PyTorch (11.1). Please make sure to use the same CUDA versions.
```
### 解释
 服务器上同时安装了 `CUDA 11.1` 和 `CUDA 12.2` 两个版本的运行时库。系统管理员将 `CUDA` 符号链接设置为指向 `CUDA 12.2` 以满足最新的需要。然而，代码运行环境的 `PyTorch` 是基于 `CUDA 11.1` 编译的，当尝试运行这些代码时，由于加载的是 `CUDA 12.2` 运行时库，导致代码无法正常执行，出现了运行时错误。

### 解决方案

为了不影响其他用户使用，使用环境变量来解决问题，在运行代码前临时修改 `PATH` 和 `LD_LIBRARY_PATH` 环境变量，使其指向 `CUDA 11.1` 的安装路径。
```console
export PATH=/usr/local/cuda-11.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-11.1/lib64:$LD_LIBRARY_PATH
```

