---
draft: false
date:
  created: 2024-09-18
  updated: 2025-01-02
  updated: 2025-01-14
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

    自从我开始使用云服务器，就时不时需要将本地文件传输到远端服务器。虽然有很多软件可以实现这一功能，但这里我想记录一种可以直接在终端中完成的解决方案：**rsync** 命令。rsync 是 Linux 系统下的数据镜像备份工具，可以实现本地主机和远端服务器之间的文件同步。前面提到的需求——向远端服务器传输文件，可以视为将本地文件同步到远端，因此可以使用 rsync 来完成。此外，rsync 采用**增量同步**技术，能够实现断点续传的效果，为大文件、多文件的传输提供了便利。

    另一方面，在查阅了参考文献1.以及 rsync 的官方手册之后，我才发现最初在网上找到的一些教程存在误导。希望通过这篇文章，能够记录下更为准确的知识和理解。

??? info "参考文献"

    1. [骏马金龙: 第2章 rsync(一)：基本命令和用法](https://blog.csdn.net/qq_32706349/article/details/91451053)
    2. [CSDN：揭秘强大的文件同步利器Rsycn](https://blog.csdn.net/hy199707/article/details/137499793)
    3. [博客园：通过rsycn实现数据同步](https://www.cnblogs.com/feng0919/p/11223537.html)
<!-- more -->

## 基本概念

参考手册，rsync 可以在远程主机和本地主机**之间**复制文件（不直接支持在两个远程主机之间复制文件）。使用该命令涉及两对概念：

- **源路径/目标路径**：rsync 支持本地与远端的互传，这一对概念明确了以哪边的文件为同步基准。
- **客户端/服务端**：rsync 将本地端称为客户端，将远程端称为服务端。

## 使用方式

参考手册，rsync 命令有三种使用方式：

### 一、本地同步

!!! info "命令格式"

    ```bash
    rsync [OPTION...] SRC... [DEST]
    ```
    在本地将源路径下的内容同步到目标路径下。

### 二、通过远程 shell 与服务端通信

!!! info "命令格式"

    #### 拉取（Pull）：
    ```console
    rsync [OPTION...] [USER@]HOST:SRC... [DEST]
    ```
    以某用户的身份登录到服务端，将远端源路径下的内容同步到本地的目标路径下

    #### 推送（Push）：
    ```console
    rsync [OPTION...] SRC... [USER@]HOST:DEST
    ```
    以某用户的身份登录到服务端，将本地源路径下的内容同步到远端的目标路径下

### 三、通过 TCP 与服务端的 rsync 守护进程通信

!!! info "命令格式"

    #### 拉取（Pull）：
    ```console
    rsync [OPTION...] [USER@]HOST::SRC... [DEST]
    rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
    ```

    - 通过 rsync 守护进程将远端源路径下的内容同步到本地目标路径下。
    - 在这种情况下，将直接连接到远程的 rsync 守护进程，通常使用 TCP 端口 873。

    #### 推送（Push）：
    ```console
    rsync [OPTION...] SRC... [USER@]HOST::DEST
    rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST
    ```

    - 通过 rsync 守护进程将本地源路径下的内容同步到远端的目标路径下。
    - 在这种情况下，将直接连接到远程的 rsync 守护进程，通常使用 TCP 端口 873。

!!! note "小结"

    - rsync 针对的是本地与远端之间的文件同步
    - rsync 支持远程 shell 通信和 TCP 通信两种通信方式，后者通过 rsync 守护进程进行通信。
    - rsync 的命令格式为：**rsync [选项] [源路径] [目标路径]**，通过 `User`, `@`, `HOST` 这些额外信息来区分远端和本地端

## 常用选项

### 基本选项
- `-v`: **详细模式，显示传输过程**
- `-q`: 静默模式，不显示传输信息  
- `-P`: 等同于 `--partial --progress`，显示传输进度并支持断点续传
- `-n`: 模拟运行，不实际执行同步

??? note "`-P` 的补充说明"

    - `--partial`: 保留部分传输的文件，支持断点续传
    - `--progress`: 显示文件传输的实时进度

### 传输控制
- `-a`: **归档模式，相当于 `-rlptgoD`，保留文件属性**
- `-r`: 递归同步目录
- `-z`: **压缩传输，节省带宽**
- `--delete`: 删除远端多余的文件，保持两端完全一致

??? note " `-a` 的补充说明"

    - `-r`：递归同步目录
    - `-l`：保留符号链接
    - `-p`：保留文件权限
    - `-t`：保留文件时间戳
    - `-g`：保留文件属组(group)
    - `-o`：保留文件属主(owner)
    - `-D`：保留设备文件和特殊文件

### 文件处理
- `-b`: 备份文件，在目标端创建备份
- `-c`: 基于文件校验和进行同步

### 远程连接
- `-e ssh`: **指定远程连接命令**

## 实际场景

!!! question "场景1"

    本地主机可以登录到两台 Linux 服务器 A 和 B 上，现在需要在两台服务器之间进行文件传输：将服务器 A 上的 `/path/to/source_dir` 文件夹内容传输到服务器 B 上的 `/path/to/sync_dir` 文件夹下。其中，服务器 A 使用密码登录，端口设置为 1111，服务器 B 禁用了密码登录，只能使用密钥登录，端口设置为 2222。

??? tip "方案1: 从 A 推送 B"

    从 Server A 推送文件到 Server B，这需要 Server A 能够远程登录到 Server B。由于 B 禁用了密码登录，需要将 A 上的公钥放到 B 的 `.ssh/authorized_keys` 中，然后在 A 上执行对应格式的命令如下：
    ```console
    $ rsync -azv -e "ssh -p 2222" /path/to/source_dir {User name}@{Server B IP}:/path/to/sync_dir
    ```

??? tip "方案2: 从 B 拉取 A"

    从 Server B 推送文件到 Server A，这需要 Server B 能够远程登录到 Server A。这里 A 没有禁止密码登录，所以可以直接在 B 中执行下面的命令：
    ```console
    $ rsync -azv -e "ssh -p 1111" {User name}@{Server A IP}:/path/to/source_dir /path/to/sync_dir 
    ```

!!! question "场景2"

    该博客在一台 2核2G 的云服务器上使用 neovim 编写，编写时希望利用 mkdocs 实时同步修改的功能查看本次修改对构建的页面的影响，然而 2核2G 的云服务器在使用 nvim 编写博客时已经显的比较吃力，再在服务器上跑一个博客程序更是捉襟见肘。幸运的是本地有一台性能良好的主机，因此计划将博客的编写转移到本地，通过 rsync 将修改同步到云服务器上，这样云服务器只需要负责构建页面即可。

??? tip "方案: 设置按键命令，快速的将本地更新推送到远端"

    基本思路是将 `ss` 映射为同步操作，自动切换命令模式并执行本地到远端的同步命令。就像下面这样：

    ```lua title='nvim/lua/config/keymaps.lua' linenums="1" hl_lines="4"
    -- 定义一个函数来执行 rsync 命令
    function sync_to_remote()
      local remote_path = "{User name}@{Server IP}:/path/to/myblog/" 
      local command = "rsync -aqz -e 'ssh -p xxx' /path/to/myblog/ " .. remote_path
      vim.fn.system(command)
    end

    vim.api.nvim_set_keymap("n", "ss", ":lua sync_to_remote()", { noremap = true, silent = false })
    ```

    这里没有把 `<CR>` 加入到映射后的命令中，所以需要再按下确认键才会执行命令，是一个防呆设计。不过这么做的要要求本地的 `myblog` 和服务器上的 `myblog` 都要固定路径，不能随意移动。所以这只是一个不成熟的方案，但目前够用。

