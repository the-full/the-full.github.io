---
draft: true
date:
  created: 2025-01-21
categories:
  - Learning
  - Memo
tags:
  - HTTP
  - Framework
authors:
  - zhazi
---


# 青训营笔记：HTTP 框架

!!! abstract ""

    记录学习 HTTP 框架过程中的思考和实践。

    TODO: 讲解逻辑：给出一个示例，分解示例的每个部分，由每个分解出的部分引出其背后的内容，或直接由示例引出背后的内容
<!-- more -->


## HTTP 协议

HTTP: **超文本**传输协议

!!! note "文本与超文本"

我们为什么需要协议？最主要的原因是为了确保通信双方能够正确理解和解析传输的数据

### 协议的要素

- 明确的边界：通过特定的分隔符或长度标识来界定每个消息的边界
- 信息的类型：通过头部字段或协议规范来定义传输数据的格式和语义

### HTTP/1.1 协议里面有什么？

先从一个具体的例子开始（请求和响应）

请求分为：xxx, xxx, xxx,对应的，相应由：xxx, xxx, xxx 组成

响应分为：xxx, xxx, xxx
!!! note "请求行/状态行"

!!! note "请求头/响应头"

!!! note "请求体/响应体"


### Hertz 框架的请求流程

### HTTP => QUIC ？

#### HTTP1

#### HTTP2

#### QUIC


## HTTP 框架的设计与实现

make it work, make it right.

## 分层设计

### 应用层

### 中间件层

### 路由层

### 协议层

### 传输层

### 公共模块



## make it fast

## 企业实践
