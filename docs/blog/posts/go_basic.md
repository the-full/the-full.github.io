---
draft: false
date:
  created: 2024-09-01
  updated: 2025-01-02
categories:
  - Learning
  - Memo
tags:
  - note
  - go
authors:
  - zhazi
---

# 笔记：Go 语言基础

!!! info "参考文献"

    - [Go 语言简明教程](https://geektutu.com/post/quick-golang.html)
    - [Go Wiki: Go Modules](https://go.dev/wiki/Modules#new-concepts)

<!-- more -->

## rune 类型

- `rune` 类型等同于 `int32`，用于表示一个 Unicode 码点（code point）。
- 用于处理多字节字符，比如汉字

```go title="示例"
s := "你好, 世界" // UTF-8 编码的字符串
r := []rune(s)    // 将 string 转换为 rune 切片
for i, char := range r {
    fmt.Printf("%d: %c\n", i, char)
}
```

## 数组(array)与切片(slice)

- 数组的长度不能改变。
- 切片包含三个组件：容量，长度和指向底层数组的指针,切片可以随时进行扩展。
- 切片是数组的抽象。
- 类比 c++: 数组 => `std::array`, 切片 => `std: vector`
- 切片使用数组作为底层结构。

```go title="声明切片"
slice1 := make([]float32, 0) // 长度为0的切片
slice2 := make([]float32, 3, 5) // [0 0 0] 长度为3容量为5的切片
fmt.Println(len(slice2), cap(slice2)) // 3 5
```

??? note "make 函数"

    注：来自 ai

    在 Go 语言中，`Make` 函数是一个用于创建和初始化 slice、map 和 channel 的内建函数。`make` 函数的主要目的是分配内存并返回一个类型为 slice、map 或 channel 的初始化（非零）值。

    ```go
    make(t Type, size ...IntegerType)
    ```

    - `t` 是要创建的类型，它必须是 slice、map 或 channel 之一。
    - `size` 是一个或多个指定 slice、map 或 channel 大小或容量的整数参数。
      下面分别介绍 `make` 函数在 slice、map 和 channel 中的使用：

    ### 为什么要使用 `make`？

    - **内存安全**：`make` 确保返回的数据结构是有效的，并已经初始化了，这意味着它们的内存已经被正确分配，并且可以立即使用。
    - **性能**：使用 `make` 创建 slice、map 或 channel 可以避免在后续操作中进行不必要的内存重新分配，这有助于提高性能。

    ### 注意事项

    - `make` 只用于创建 slice、map 和 channel，对于其他类型，如数组、结构体等，使用 `new` 或者直接声明。
    - `make` 返回的是初始化后的（非零）值，而 `new` 返回的是指向类型的零值的指针。
      使用 `make` 函数是 Go 语言中管理内存和初始化数据结构的常见做法，对于编写高效且安全的代码至关重要。

## 指针

- c 语言的指针

## For 循环

- 支持 for range 遍历

```go title="示例"
nums := []int{10, 20, 30, 40}
for i, num := range nums {
 fmt.Println(i, num)
}
// 0 10
// 1 20
// 2 30
// 3 40
m2 := map[string]string{
 "Sam":   "Male",
 "Alice": "Female",
}

for key, value := range m2 {
 fmt.Println(key, value)
}
// Sam Male
// Alice Female
```

## 函数

- 支持**多个返回值**

```go title="语法"
func funcName(param1 Type1, param2 Type2, ...) (return1 Type3, ...) {
    // body
}
```

### 错误处理

- 多返回值的特性使得函数可以额外返回一个反映运行状态的变量
- error 往往是能预知的错误，但是也可能出现一些不可预知的错误，例如数组越界，这种错误可能会导致程序非正常退出，在 Go 语言中称之为 panic。
- 有一套类似 `try-catch` 的异常处理机制 `defer-recover`

```go title='示例'
func get(index int) (ret int) {
  defer func() {
    if r := recover(); r != nil {
    fmt.Println("Some error happened!", r)
    ret = -1
    }
  }()
  arr := [3]int{2, 3, 4}
  return arr[index]
}

func main() {
  fmt.Println(get(5))
  fmt.Println("finished")
}
```

## 结构体和方法

- 使用 `Name{field: value, ...}` 的形式创建结构体的实例。
- 实现方法与实现函数的区别在于，在 func 和函数名之间，加上该方法对应的结构体名称

```go title="语法"
func (param *amSName)funcName(param1 Type1, param2 Type2, ...) (return1 Type3, ...) {
    // body
}
```

## 接口

- 鸭子类型，实现了所有方法，那就是可以视为对应类型。
- 接口定义了一组方法的集合，接口是一个类型，但不能被实例化。

```go title="示例"
type Person interface {
 getName() string
}
```

实现了 getName，那么该变量就是 Person 接口

## 并发编程

### goroutine

- `goroutine` 是一种轻量级线程，由 Go 运行时（runtime）管理。
- `goroutine` 的栈内存初始很小，在需要时可以动态地伸缩。
- 通过在函数调用前加上 `go` 关键字来启动一个新的 `goroutine`。
- `goroutine` 默认的调度是由 Go 运行时非抢占式地完成的

### channel

- channel 提供了一种在多个 goroutine 之间传递值的方式。
- 定义 channel 需要指定元素类型
- 使用 `<-` 操作符发送（chan <- value）和接收（value <- chan）数据。
- 默认情况下，发送和接收操作是阻塞的，这提供了自然的同步机制。
- 发送者可以通过 close(chan) 来关闭一个 channel

### sync

- sync 是 go 中的一个包，提供了一些用于 goroutine 同步的组件
- 常用的组件有：互斥锁（Mutex）、读写锁（RWMutex）、等待组（WaitGroup）
- 等待组是一种信号量机制

??? info "一个简单的示例"

    ```go title="main.go" linenums="1"
    package main

    import (
        "fmt"
        "sync"
        "time"
    )

    // 共享资源
    var count int

    // 互斥锁，用于保护共享资源
    var mutex sync.Mutex

    // 读写锁，用于保护共享资源
    var rwMutex sync.RWMutex

    func worker(id int) {
        // 使用互斥锁保护共享资源
        mutex.Lock()
        count++
        fmt.Printf("Worker %d is working, count: %d\n", id, count)
        mutex.Unlock()

        // 模拟耗时工作
        time.Sleep(time.Second)

        // 使用读写锁保护共享资源
        rwMutex.RLock()
        fmt.Printf("Worker %d is reading count: %d\n", id, count)
        rwMutex.RUnlock()

        wg.Done()
    }

    func main() {
        var wg sync.WaitGroup

        // 启动多个工作 goroutine
        for i := 1; i <= 3; i++ {
            wg.Add(1)
            go worker(i)
        }

        // 等待所有工作 goroutine 完成
        wg.Wait()
        fmt.Println("All workers are done")
    }
    ```

## 单元测试

- 通常使用 testing 包
- 测试文件的命名规范是 被测试文件名 + '\_test.go'
- 通常把被测试文件和测试文件放在导在同一个包下，这样测试函数可以直接访问被测试函数

??? note "testing 包"

    `testing` 包提供了编写和运行 Go 语言测试代码所需的结构和函数。主要包括：

    1. **测试函数的框架**：定义了如何命名和编写测试函数（如 `TestXxx`）。
    2. **测试日志和报告**：提供了 `t.Errorf`、`t.Fatalf` 和 `t.Logf` 等函数，用于记录测试错误、失败和日志信息。
    3. **测试覆盖率分析**：支持通过 `go test -cover` 命令分析测试覆盖率。
    4. **测试参数和辅助函数**：提供了处理测试参数和设置测试环境的功能。

    这些功能使得在 Go 语言中编写和运行测试变得更加方便和高效。

??? info "一个简单的示例"

    被测文件:
    ``` go title="mypackage.go" linenums="1"
    package mypackage

    // Add takes two integers and returns the sum.
    func Add(a, b int) int {
        return a + b
    }
    ```

    测试文件:
    ``` go title="mypackage_test.go" linenums="1"
    package mypackage

    import "testing"

    // TestAdd tests the Add function with several pairs of numbers.
    func TestAdd(t *testing.T) {
        testCases := []struct {
            name     string
            a, b     int
            expected int
        }{
            {"two positive numbers", 2, 3, 5},
            {"negative and positive", -1, 5, 4},
            {"two negative numbers", -2, -3, -5},
        }

        for _, tc := range testCases {
            t.Run(tc.name, func(t *testing.T) {
                result := Add(tc.a, tc.b)
                if result != tc.expected {
                    t.Errorf("Add(%d, %d) = %d; expected %d", tc.a, tc.b, result, tc.expected)
                }
            })
        }
    }
    ```

## 包(Package)和模块(Modules)

### package

- `package` 应该被组织在一个文件夹下。
- 同一个 `package` 内部的所有源文件共享相同的命名空间。即：同一个 `package` 内部变量、类型、方法等定义可以相互看到。
- 如果类型/接口/方法/函数/字段的首字母大写，则是 **Public** 的，对其他 `package` 可见，如果首字母小写，则是 **Private** 的，对其他 `package` 不可见。

### modules

- Go Modules 是 Go 1.11 版本之后引入的，Go 1.11 之前使用 $GOPATH 机制。
- $GOPATH 机制和 $PATH 类似，通过环境变量设置路径，用户需要自己管理依赖的代码(`go get`)
- Go Modules 算是较为完善的包管理工具，同时支持代理。
- [参考链接](https://go.dev/wiki/Modules#new-concepts)

- 模块是相关 Go 包的集合。
- 模块记录精确的依赖关系需求并创建可重复的构建。
- 存储库包含一个或多个 Go 模块。
- 每个模块包含一个或多个 Go 包。
- 每个包由单个目录中的一个或多个 Go 源文件组成。
