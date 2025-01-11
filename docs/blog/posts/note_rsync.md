---
draft: true
date:
  created: 2024-09-18
  updated: 2025-01-02
categories:
  - Learning
  - Memo
tags:
  - Linux
authors:
  - zhazi
---

# 笔记：linux 服务器间同步命令 -- rsync

!!! abstract ""

    随着我开始使用云服务器，我开始经常遇到要向远端服务器传输文件的情况，有很多软件可以做到这件事，这里记录的一种可以在终端中完成的解决方案：使用 **rsync** 命令。rsync 是 Linux 系统下的数据镜像备份工具，向远端服务器传输文件这个需求可以视为将本地的文件同步到远端，因此可以使用 rsync 来完成。此外，rsync 采用**增量同步**技术，可以实现断点续传的效果，为大文件、多文件的传输提供了便利。

??? info "参考文献"

    1. [骏马金龙: 第2章 rsync(一)：基本命令和用法](https://blog.csdn.net/qq_32706349/article/details/91451053)
    2. [CSDN：揭秘强大的文件同步利器Rsycn](https://blog.csdn.net/hy199707/article/details/137499793)
    3. [博客园：通过rsycn实现数据同步](https://www.cnblogs.com/feng0919/p/11223537.html)
<!-- more -->

## 基本概念

参考手册，Rsync 可以在远程主机和本地主机**之间**复制文件（它不支持在两个远程主机之间复制文件）。使用该命令涉及两对概念：

- **源路径 & 目标路径**：rsync 支持本地与远端的互传，因此使用源路径和目标路径的明确以哪边文件为同步基准。
- **客户端 & 服务端**：rsync 将本地端称为客户端，将远程端称为服务端。

## 使用方式

参考手册，rsync 命令有三种使用方式：

### 本地同步：
```bash
rsync [OPTION...] SRC... [DEST]
```
- **描述**：在本地将源路径下的内容同步到目标路径下。

### 通过远程 shell 与服务端通信：

#### 拉取（Pull）：
```bash
rsync [OPTION...] [USER@]HOST:SRC... [DEST]
```
- **描述**：以某用户的身份登录到服务端，将远端源路径下的内容同步到本地的目标路径下

#### 推送（Push）：
```bash
rsync [OPTION...] SRC... [USER@]HOST:DEST
```
- **描述**：以某用户的身份登录到服务端，将本地源路径下的内容同步到远端的目标路径下

### 通过 TCP 与服务端的 rsync 守护进程通信：

#### 拉取（Pull）：
```bash
rsync [OPTION...] [USER@]HOST::SRC... [DEST]
rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
```
- **描述**：通过 rsync 守护进程将远端源路径下的内容同步到本地目标路径下。
- 在这种情况下，将直接连接到远程的 rsync 守护进程，通常使用 TCP 端口 873。

#### 推送（Push）：
```bash
rsync [OPTION...] SRC... [USER@]HOST::DEST
rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST
```
- **描述**：通过 rsync 守护进程将本地源路径下的内容同步到远端的目标路径下。
- 在这种情况下，将直接连接到远程的 rsync 守护进程，通常使用 TCP 端口 873。

!!! note "小结"

    - rsync 针对的是本地与远端之间的文件同步
    - rsync 支持远程 shell 通信和 TCP 通信两种通信方式，后者通过 rsync 守护进程进行通信。
    - rsync 的命令格式为：rsync 选项 源路径 目标路径，通过 `User`, `@`, `HOST` 这些额外信息来区分远端和本地端

## 实际场景

!!! info "场景1"

    需要在两台 Linux 服务器之间进行文件传输，将服务器 A 上的 /path/to/source_dir 文件夹内容传输到服务器 B 上的 /path/to/sync_dir 文件夹下。这两台服务器之间是连通的，服务器 A 使用密码登录，端口设置为 1111，服务器 B 禁用了密码登录，只能使用密钥登录，端口设置为 2222。


```bash
rsync -azv -e "ssh -p 2222" /path/to/source_dir {User}@{Server B IP}:/path/to/sync_dir
```
