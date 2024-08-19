# 笔记：Go 语言基础

## rune 类型

- `rune` 类型是 Go 语言的内置类型，它等同于 `int32`，用于表示一个 Unicode 码点（code point）。因此，每个 `rune` 都对应一个 Unicode 字符，无论这个字符是由多少个字节组成的。
- 当字符串被转换为 `rune` 数组时，每个元素就代表字符串中的一个 Unicode 字符，这样就可以正确处理每个字符，无论字符是英文字符、中文字符还是其他语言的字符。

```go
s := "你好, 世界" // UTF-8 编码的字符串
r := []rune(s)    // 将 string 转换为 rune 切片
for i, char := range r {
    fmt.Printf("%d: %c\n", i, char)
}
```

## 数组(array)与切片(slice)

- 数组的长度不能改变。
- 切片是数组的抽象。
- 切片使用数组作为底层结构。
- 切片包含三个组件：容量，长度和指向底层数组的指针,切片可以随时进行扩展。
- 类比 c++: 数组`std::array`, 切片`std: vector`

```go title="声明切片"
slice1 := make([]float32, 0) // 长度为0的切片
slice2 := make([]float32, 3, 5) // [0 0 0] 长度为3容量为5的切片
fmt.Println(len(slice2), cap(slice2)) // 3 5
```

??? note "make 函数"

    在 Go 语言中，`Make` 函数是一个用于创建和初始化 slice、map 和 channel 的内建函数。`make` 函数的主要目的是分配内存并返回一个类型为 slice、map 或 channel 的初始化（非零）值。
    以下是 `make` 函数的基本语法：

    ```go
    make(t Type, size ...IntegerType)
    ```

    - `t` 是要创建的类型，它必须是 slice、map 或 channel 之一。
    - `size` 是一个或多个指定 slice、map 或 channel 大小或容量的整数参数。
      下面分别介绍 `make` 函数在 slice、map 和 channel 中的使用：

    ### Slice

    使用 `make` 创建 slice 时，你可以指定 slice 的长度和容量。长度是 slice 中元素的数量，容量是底层数组可以容纳的元素数量。

    ```go
    slice := make([]int, length, capacity)
    ```

    如果不指定容量，它将默认与长度相同。

    ```go
    slice := make([]int, length) // 容量等于长度
    ```

    ### Map

    使用 `make` 创建 map 时，你只需要指定 map 的初始容量。

    ```go
    m := make(map[string]int, capacity)
    ```

    如果不指定容量，map 将根据需要自动扩容。

    ```go
    m := make(map[string]int) // 容量由 Go 自动管理
    ```

    ### Channel

    使用 `make` 创建 channel 时，可以指定 channel 的缓冲区大小。

    ```go
    ch := make(chan int, buffer_size)
    ```

    如果缓冲区大小为 0，或者省略了缓冲区大小，则创建的是无缓冲的 channel。

    ```go
    ch := make(chan int) // 无缓冲 channel
    ```

    ### 为什么要使用 `make`？

    - **内存安全**：`make` 确保返回的数据结构是有效的，并已经初始化了，这意味着它们的内存已经被正确分配，并且可以立即使用。
    - **性能**：使用 `make` 创建 slice、map 或 channel 可以避免在后续操作中进行不必要的内存重新分配，这有助于提高性能。

    ### 注意事项

    - `make` 只用于创建 slice、map 和 channel，对于其他类型，如数组、结构体等，使用 `new` 或者直接声明。
    - `make` 返回的是初始化后的（非零）值，而 `new` 返回的是指向类型的零值的指针。
      使用 `make` 函数是 Go 语言中管理内存和初始化数据结构的常见做法，对于编写高效且安全的代码至关重要。

## 指针

- c 语言的指针

## 流程控制
