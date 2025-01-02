---
draft: true
date:
  created: 2024-09-05
  updated: 2025-01-02
categories:
  - Learning
tags:
  - note
  - go
authors:
  - zhazi
---

# 笔记：7-day golang 系列：Web 框架 Gee

!!! info "参考文献"

    - [七天用Go从零实现系列](https://geektutu.com/post/gee.html)

<!-- more -->

## day1 整理

- %q：表示将字符串k和v格式化为双引号包围的字符串，并在需要时对字符串内的特殊字符进行转义。
- gee/gin 框架建立在 net/http 框架上，后者提供了基础的Web功能：监听端口，映射静态路由，解析HTTP报文。
  基本逻辑是建立一张静态路由表，将 HTTP 保文映射给 HandlerFunc 处理。
- day1 的 gee 框架通过实现 ServeHTTP 接口注册到 net/http 中，其对接管的路径进行进一步的静态路由。

!!! info "go mod"

## day2 整理

- 在Go语言中，interface{}是一个空接口，表示它不包含任何方法。这意味着interface{}可以表示任何类型的值。
- http.Request 的 FormValue(key) 方法接受一个字符串key作为参数，返回请求中与该key关联的表单值。如果请求包含多个具有相同键的值，FormValue返回第一个值。
- Query()：是 url.URL 类型的一个方法，它**解析URL中的查询字符串**并返回一个url.Values类型的多映射。
- 这里多映射指的是一个将字符串键映射到字符串列表的映射。
- ...是一个语法糖，用于函数的参数列表，表示可变参数。在函数内部，这些参数被视为该类型的切片（slice）。结合 interface{} 类型，可以表示任意类型的可变参数。
- Sprintf 写入 String, Fprintf 写入 File

- 上下文 ~ 构造消息, 核心是将响应写入 Header 和 Body，相同类型的响应有相同的写入方式，Context 封装常见的响应类型的写入，提供更简洁的接口
