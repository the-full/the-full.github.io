---
draft: true
date:
  created: 2025-02-17
categories:
  - Memo
authors:
  - zhazi
---

# 笔记：常见问题归档

记录频繁遇见的一些小问题。

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
