## Go基础知识

### 变量定义

* 使用var关键字
  - var a,b,c bool
  - var s1,s2 string = "hello", "world"
  - var()集中定义
  - 可以放在函数体内，或者放在包内
  - a := 1(使用冒号代替)



### 内建变量类型

* bool, string
* (u)int,(u)int8,(u)int16,(u)int32,(u)int64, uintptr
* byte,rune
* float32,float64,complex64,complex128
* 强制类型转换 var c int = int(math.Sqrt(float64(a\*a + b\*b)))



### 常量与枚举

* const filename = "abc.txt"
* 枚举

```go
const(
	cpp = iota
    java
    golang
    javascript
)
```



### 条件语句和循环

* for, if后没有小括号
* if 条件里也可定义变量
* 没有while
* switch不需要break，也可以直接switch多个条件
* 死循环

```go
for {
    
}
```

### 函数

* func eval(a,b int, op string) int
* 返回值类型写在最后面
* 可返回多个值
* 函数作为参数
* 没有默认参数，可选参数

```go
//函数式编程
func app(op func(int, int) int, a, b int) int {
    return op(a,b)
}
apply(
    func(a int, b int) int {
        return a + b
    }, 1, 2)

//可变参数列表
func sum(numbers ...int) int {
    s := 0
    for i:= range numbers{
        s += numbers[i]
    }
    return s
}
```



### 指针

* 指针不能运算
* 参数传递：只有值传递一种方式

```go
func swap(*a, *b int){
    *a, *b = *b, *a
}

func swap(a, b int) (int, int){
    return b, a
}
```

