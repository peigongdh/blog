- **以上笔记整理来源老钱的知乎专栏，仅作为个人笔记，如有侵权请及时联系删除**

## 《快学 Go 语言》第 1 课 —— Hello World
> https://zhuanlan.zhihu.com/p/48013279

- go不需要要设置goroot和gopath（默认~/go）

## 《快学 Go 语言》第 2 课 —— 变量基础
> https://zhuanlan.zhihu.com/p/48153187

```go
var s int = 42
var s = 42
s := 42
```

> 如果一个变量很重要，建议使用第一种显示声明类型的方式来定义，比如全局变量的定义就比较偏好第一种定义方式。如果要使用一个不那么重要的局部变量，就可以使用第三种。比如循环下标变量
> 如果在第一种声明变量的时候不赋初值，编译器就会自动赋予相应类型的「零值」，不同类型的零值不尽相同，比如字符串的零值不是 nil，而是空串，整形的零值就是 0 ，布尔类型的零值是 false。

```go
package main

import "fmt"

func main() {
    var value int = 42
    var p1 *int = &value
    var p2 **int = &p1
    var p3 ***int = &p2
    fmt.Println(p1, p2, p3)
    fmt.Println(*p1, **p2, ***p3)
}

// ----------
// 0xc4200160a0 0xc42000c028 0xc42000c030
// 42 42 42
```
> 我们又看到了久违的指针符号 * 和取地址符 &，在功能和使用上同 C 语言几乎一摸一样。同 C 语言一样，指针还支持二级指针，三级指针，只不过在日常应用中，很少遇到。

```go
package main

import "fmt"

func main() {
    // 有符号整数，可以表示正负
    var a int8 = 1 // 1 字节
    var b int16 = 2 // 2 字节
    var c int32 = 3 // 4 字节
    var d int64 = 4 // 8 字节
    fmt.Println(a, b, c, d)

    // 无符号整数，只能表示非负数
    var ua uint8 = 1
    var ub uint16 = 2
    var uc uint32 = 3
    var ud uint64 = 4
    fmt.Println(ua, ub, uc, ud)

    // int 类型，在32位机器上占4个字节，在64位机器上占8个字节
    var e int = 5
    var ue uint = 5
    fmt.Println(e, ue)

    // bool 类型
    var f bool = true
    fmt.Println(f)

    // 字节类型
    var j byte = 'a'
    fmt.Println(j)

    // 字符串类型
    var g string = "abcdefg"
    fmt.Println(g)

    // 浮点数
    var h float32 = 3.14
    var i float64 = 3.141592653
    fmt.Println(h, i)
}
// -------------
// 1 2 3 4
// 1 2 3 4
// 5 5
// true
// abcdefg
// 3.14 3.141592653
// 97
```
> 还有另外几个不常用的数据类型，读者可以暂不理会。
> - 复数类型 complex64 和 complex128
> - unicode字符类型 rune
> - uintptr 指针类型

> 简单一点说 rune 和 byte 的关系就好比 Python 里面的 unicode 和 byte 、Java 语言里面的 char 和 byte 。uintptr 相当于 C 语言里面的 void* 指针类型。

## 《快学 Go 语言》第 3 课 —— 分支与循环
> https://zhuanlan.zhihu.com/p/48300291

```go
func prize2(score int) string {
    switch {
        case score < 60:
            return "差"
        case score < 80:
            return "及格"
        case score < 90:
            return "良"
        default:
            return "优"
    }
}
```
> switch 从第一个判断表达式为 true 的 case 开始执行，如果 case 带有 fallthrough，程序会继续执行下一条 case，且它不会去判断下一个 case 的表达式是否为 true。
> switch 有两种匹配模式，一种是变量值匹配，一种是表达式匹配。switch 还支持特殊的类型匹配语法。

## 《快学 Go 语言》第 4 课 —— 低调的数组
> https://zhuanlan.zhihu.com/p/48927056

```go
package main

import "fmt"

func main() {
    var a = [9]int{1, 2, 3, 4, 5, 6, 7, 8, 9}
    var b [10]int = [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    c := [8]int{1, 2, 3, 4, 5, 6, 7, 8}
    fmt.Println(a)
    fmt.Println(b)
    fmt.Println(c)
}
// ---------------------
// [1 2 3 4 5 6 7 8 9]
// [1 2 3 4 5 6 7 8 9 10]
// [1 2 3 4 5 6 7 8]
```

> Go 会在编译后的代码中插入下标越界检查的逻辑，所以数组的下标访问效率是要打折扣的，比不得 C 语言的数组访问性能。

## 《快学 Go 语言》第 5 课 —— 灵活的切片
> https://zhuanlan.zhihu.com/p/49415852

> 上图中一个切片变量包含三个域，分别是底层数组的指针、切片的长度 length 和切片的容量 capacity。切片支持 append 操作可以将新的内容追加到底层数组，也就是填充上面的灰色格子。如果格子满了，切片就需要扩容，底层的数组就会更换。

```go
package main

import "fmt"

func main() {
 var s1 []int = make([]int, 5, 8)
 var s2 []int = make([]int, 8) // 满容切片
 fmt.Println(s1)
 fmt.Println(s2)
}
// -------------
// [0 0 0 0 0]
// [0 0 0 0 0 0 0 0]
```
> make 函数创建切片，需要提供三个参数，分别是切片的类型、切片的长度和容量。其中第三个参数是可选的，如果不提供第三个参数，那么长度和容量相等，也就是说切片的满容的

> 切片的初始化
> 使用 make 函数创建的切片内容是「零值切片」，也就是内部数组的元素都是零值。Go 语言还提供了另一个种创建切片的语法，允许我们给它赋初值。使用这种方式创建的切片是满容的。

> 空切片
> 在创建切片时，还有两个非常特殊的情况需要考虑，那就是容量和长度都是零的切片，叫着「空切片」，这个不同于前面说的「零值切片」。

```go
package main

import "fmt"

func main() {
 var s1 []int
 var s2 []int = []int{}
 var s3 []int = make([]int, 0)
 fmt.Println(s1, s2, s3)
 fmt.Println(len(s1), len(s2), len(s3))
 fmt.Println(cap(s1), cap(s2), cap(s3))
}
// -----------
// [] [] []
// 0 0 0
// 0 0 0
```

> 上面三种形式创建的切片都是「空切片」，不过在内部结构上这三种形式是有差异的，甚至第一种都不叫「空切片」，而是叫着「 nil 切片」。但是在形式上它们一摸一样，用起来没有区别。所以初级用户可以不必区分「空切片」和「 nil 切片」，到后续章节我们会仔细分析这两种形式的区别。

> 切片的赋值
> 切片的赋值是一次浅拷贝操作，拷贝的是切片变量的三个域，你可以将切片变量看成长度为 3 的 int 型数组，数组的赋值就是浅拷贝。拷贝前后两个变量共享底层数组，对一个切片的修改会影响另一个切片的内容，这点需要特别注意。
```go
package main

import "fmt"

func main() {
 var s1 = make([]int, 5, 8)
 // 切片的访问和数组差不多
 for i := 0; i < len(s1); i++ {
  s1[i] = i + 1
 }
 var s2 = s1
 fmt.Println(s1, len(s1), cap(s1))
 fmt.Println(s2, len(s2), cap(s2))

 // 尝试修改切片内容
 s2[0] = 255
 fmt.Println(s1)
 fmt.Println(s2)
}

// --------
// [1 2 3 4 5] 5 8
// [1 2 3 4 5] 5 8
// [255 2 3 4 5]
// [255 2 3 4 5]
```

> 从上面的输出中可以看到赋值的两切片共享了底层数组。

> 文章开头提到切片是动态的数组，其长度是可以变化的。什么操作可以改变切片的长度呢，这个操作就是追加操作。切片每一次追加后都会形成新的切片变量，如果底层数组没有扩容，那么追加前后的两个切片变量共享底层数组，如果底层数组扩容了，那么追加前后的底层数组是分离的不共享的。如果底层数组是共享的，一个切片的内容变化就会影响到另一个切片，这点需要特别注意。

```go
package main

import "fmt"

func main() {
 var s1 = []int{1,2,3,4,5}
 fmt.Println(s1, len(s1), cap(s1))

 // 对满容的切片进行追加会分离底层数组
 var s2 = append(s1, 6)
 fmt.Println(s1, len(s1), cap(s1))
 fmt.Println(s2, len(s2), cap(s2))

 // 对非满容的切片进行追加会共享底层数组
 var s3 = append(s2, 7)
 fmt.Println(s2, len(s2), cap(s2))
 fmt.Println(s3, len(s3), cap(s3))
}

// --------------------------
// [1 2 3 4 5] 5 5
// [1 2 3 4 5] 5 5
// [1 2 3 4 5 6] 6 10
// [1 2 3 4 5 6] 6 10
// [1 2 3 4 5 6 7] 7 10
```

> **正是因为切片追加后是新的切片变量，Go 编译器禁止追加了切片后不使用这个新的切片变量，以避免用户以为追加操作的返回值和原切片变量是同一个变量。**

```go
package main

import "fmt"

func main() {
 var s1 = []int{1,2,3,4,5}
 append(s1, 6)
 fmt.Println(s1)
}

// --------------
// ./main.go:7:8: append(s1, 6) evaluated but not used
```

> 还需要注意的是追加虽然会导致底层数组发生扩容，更换的新的数组，但是旧数组并不会立即被销毁被回收，因为老切片还指向这旧数组。

```go
package main

import "fmt"

func main() {
 var s1 = []int{1,2,3,4,5,6,7}
 // start_index 和 end_index，不包含 end_index
 // [start_index, end_index)
 var s2 = s1[2:5] 
 fmt.Println(s1, len(s1), cap(s1))
 fmt.Println(s2, len(s2), cap(s2))
}

// ------------
// [1 2 3 4 5 6 7] 7 7
// [3 4 5] 3 5
```
- **记住前闭后开，[start_index, end_index)**

```go
package main

import "fmt"

func main() {
 var s1 = []int{1, 2, 3, 4, 5, 6, 7}
 var s2 = s1[:5]
 var s3 = s1[3:]
 var s4 = s1[:]
 fmt.Println(s1, len(s1), cap(s1))
 fmt.Println(s2, len(s2), cap(s2))
 fmt.Println(s3, len(s3), cap(s3))
 fmt.Println(s4, len(s4), cap(s4))
}

// -----------
// [1 2 3 4 5 6 7] 7 7
// [1 2 3 4 5] 5 7
// [4 5 6 7] 4 4
// [1 2 3 4 5 6 7] 7 7
```

> 细心的同学可能会注意到上面的 s1[:] 很特别，它和普通的切片赋值有区别么？答案是没区别
> 使用过 Python 的同学可能会问，切片支持负数的位置么，答案是不支持，下标不可以是负数。

> 数组变切片
> 对数组进行切割可以转换成切片，切片将原数组作为内部底层数组。也就是说修改了原数组会影响到新切片，对切片的修改也会影响到原数组。

> copy 函数
> **Go 语言还内置了一个 copy 函数，用来进行切片的深拷贝。不过其实也没那么深，只是深到底层的数组而已。如果数组里面装的是指针，比如 []*int 类型，那么指针指向的内容还是共享的。**

```go
func copy(dst, src []T) int

```

> **copy 函数不会因为原切片和目标切片的长度问题而额外分配底层数组的内存，它只负责拷贝数组的内容，从原切片拷贝到目标切片，拷贝的量是原切片和目标切片长度的较小值 —— min(len(src), len(dst))，函数返回的是拷贝的实际长度。我们来看一个例子**

```go
package main

import "fmt"

func main() {
 var s = make([]int, 5, 8)
 for i:=0;i<len(s);i++ {
  s[i] = i+1
 }
 fmt.Println(s)
 var d = make([]int, 2, 6)
 var n = copy(d, s)
 fmt.Println(n, d)
}

// -----------
// [1 2 3 4 5]
// 2 [1 2]
```

> 当比较短的切片扩容时，系统会多分配 100% 的空间，也就是说分配的数组容量是切片长度的2倍。但切片长度超过1024时，扩容策略调整为多分配 25% 的空间，这是为了避免空间的过多浪费。试试解释下面的运行结果。

> **Go 语言「切片」的三种特殊状态**
> https://zhuanlan.zhihu.com/p/49529590

> 我们如何来分析三面四种形式的内部结构的区别呢？接下里要使用到 Go 语言的高级内容，通过 unsafe.Pointer 来转换 Go 语言的任意变量类型。
> 因为切片的内部结构是一个结构体，包含三个机器字大小的整型变量，其中第一个变量是一个指针变量，指针变量里面存储的也是一个整型值，只不过这个值是另一个变量的内存地址。我们可以将这个结构体看成长度为 3 的整型数组 [3]int。然后将切片变量转换成 [3]int。

```go
type slice struct {
  array unsafe.Pointer
  length int
  capcity int
}
```

```go
package main

import "fmt"

func main() {
    var s1 []int
    var s2 = []int{}
    var s3 = make([]int, 0)
    var s4 = *new([]int)

    var a1 = *(*[3]int)(unsafe.Pointer(&s1))
    var a2 = *(*[3]int)(unsafe.Pointer(&s2))
    var a3 = *(*[3]int)(unsafe.Pointer(&s3))
    var a4 = *(*[3]int)(unsafe.Pointer(&s4))
    fmt.Println(a1)
    fmt.Println(a2)
    fmt.Println(a3)
    fmt.Println(a4)
}
// ---------------------
// [0 0 0]
// [824634199592 0 0]
// [824634199592 0 0]
// [0 0 0]
```

> 从输出中我们看到了明显的神奇的让人感到意外的难以理解的不一样的结果。
> **其中输出为 [0 0 0] 的 s1 和 s4 变量就是「 nil 切片」，s2 和 s3 变量就是「空切片」。824634199592 这个值是一个特殊的内存地址，所有类型的「空切片」都共享这一个内存地址。**

-- 我本地运行的结果是：
> [842350747160 0 0]
> [842350747160 0 0]

```go
var s2 = []int{}
var s3 = make([]int, 0)

var a2 = *(*[3]int)(unsafe.Pointer(&s2))
var a3 = *(*[3]int)(unsafe.Pointer(&s3))
fmt.Println(a2)
fmt.Println(a3)

var s5 = make([]struct{ x, y, z int }, 0)
var a5 = *(*[3]int)(unsafe.Pointer(&s5))
fmt.Println(a5)

// --------
// [824634158720 0 0]
// [824634158720 0 0]
// [824634158720 0 0]
```

```go
//// runtime/malloc.go

// base address for all 0-byte allocations
var zerobase uintptr

// 分配对象内存
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
    ...
    if size == 0 {
        return unsafe.Pointer(&zerobase)
    }
    ...
}

//// runtime/slice.go
// 创建切片
func makeslice(et *_type, len, cap int) slice {
    ...
    p := mallocgc(et.size*uintptr(cap), et, true)
    return slice{p, len, cap}
}
```

- 个人理解，空切片与nil切片的区别在于切片的数组地址指向的究竟是zerobase（空切片，调用了makeslice）还是0（nil切片）

```go
package main

import "fmt"

func main() {
    var s1 []int
    var s2 = []int{}

    fmt.Println(s1 == nil)
    fmt.Println(s2 == nil)

    fmt.Printf("%#v\n", s1)
    fmt.Printf("%#v\n", s2)
}

// -------
// true
// false
// []int(nil)
// []int{}
```

> 所以为了避免写代码的时候把脑袋搞昏的最好办法是不要创建「 空切片」，统一使用「 nil 切片」，同时要避免将切片和 nil 进行比较来执行某些逻辑。这是官方的标准建议。

>> The former declares a nil slice value, while the latter is non-nil but zero-length. They are functionally equivalent—their len and cap are both zero—but the nil slice is the preferred style.

> 「空切片」和「 nil 切片」有时候会隐藏在结构体中，这时候它们的区别就被太多的人忽略了，下面我们看个例子

```go
type Something struct {
    values []int
}

var s1 = Something{}
var s2 = Something{[]int{}}
fmt.Println(s1.values == nil)
fmt.Println(s2.values == nil)

// --------
// true
// false
```

> **「空切片」和「 nil 切片」还有一个极为不同的地方在于 JSON 序列化**
