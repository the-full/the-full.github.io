# 笔记：Go 语言基础

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

# 错误处理

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

```go title="示例":
type Person interface {
 getName() string
}
```

实现了 getName，那么该变量就是 Person 接口

## 并发编程(TODO)
