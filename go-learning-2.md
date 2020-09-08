- **以上笔记整理来源老钱的知乎专栏，仅作为个人笔记，如有侵权请及时联系删除**

## 《快学 Go 语言》第 6 课 —— 字典
> https://zhuanlan.zhihu.com/p/50047198

> 如果你可以预知字典内部键值对的数量，那么还可以给 make 函数传递一个整数值，通知运行时提前分配好相应的内存。这样可以避免字典在长大的过程中要经历的多次扩容操作。

```go
var m = make(map[int]string, 16)
```

> 同 Python 语言一样，字典可以使用中括号来读写内部元素，使用 delete 函数来删除元素。

```go
package main

import "fmt"

func main() {
    var fruits = map[string]int {
        "apple": 2,
        "banana": 5,
        "orange": 8,
    }
    // 读取元素
    var score = fruits["banana"]
    fmt.Println(score)

    // 增加或修改元素
    fruits["pear"] = 3
    fmt.Println(fruits)

    // 删除元素
    delete(fruits, "pear")
    fmt.Println(fruits)
}

// -----------------------
// 5
// map[apple:2 banana:5 orange:8 pear:3]
// map[orange:8 apple:2 banana:5]
```

> **删除操作时，如果对应的 key 不存在，delete 函数会静默处理。遗憾的是 delete 函数没有返回值，你无法直接得到 delete 操作是否真的删除了某个元素。你需要通过长度信息或者提前尝试读取 key 对应的 value 来得知。**

> 读操作时，如果 key 不存在，也不会抛出异常。它会返回 value 类型对应的零值。如果是字符串，对应的零值是空串，如果是整数，对应的零值是 0，如果是布尔型，对应的零值是 false。

> **你不能通过返回的结果是否是零值来判断对应的 key 是否存在，因为 key 对应的 value 值可能恰好就是零值，比如下面的字典你就不能判断 "durin" 是否存在**

```go
package main

import "fmt"

func main() {
    var fruits = map[string]int {
        "apple": 2,
        "banana": 5,
        "orange": 8,
    }

    // var names = make([]string, 0, len(fruits))
    // var scores = make([]int, 0, len(fruits))

    var names = make([]string, len(fruits))
    var scores = make([]int, len(fruits))

    for name, score := range fruits {
        names = append(names, name)
        scores = append(scores, score)
    }

    fmt.Println(names, scores)
}

// ----------
// [apple banana orange] [0 0 0 2 5 8]
```

- 这里需要留意如果使用这样的方式获取keys，那么会在[0 0 0]的基础上append
```go
var names = make([]string, len(fruits))
```

## 《快学 Go 语言》第 7 课 —— 字符串
> https://zhuanlan.zhihu.com/p/50399072

> **字符串通常有两种设计，一种是「字符」串，一种是「字节」串。「字符」串中的每个字都是定长的，而「字节」串中每个字是不定长的。Go 语言里的字符串是「字节」串，英文字符占用 1 个字节，非英文字符占多个字节。这意味着无法通过位置来快速定位出一个完整的字符来，而必须通过遍历的方式来逐个获取单个字符。**

> 我们所说的字符通常是指 unicode 字符，你可以认为所有的英文和汉字在 unicode 字符集中都有一个唯一的整数编号，一个 unicode 通常用 4 个字节来表示，对应的 Go 语言中的字符 rune 占 4 个字节。在 Go 语言的源码中可以找到下面这行代码，rune 类型是一个衍生类型，它在内存里面使用 int32 类型的 4 个字节存储。

>> type rune int32

> 使用「字符」串来表示字符串势必会浪费空间，因为所有的英文字符本来只需要 1 个字节来表示，用 rune 字符来表示的话那么剩余的 3 个字节都是零。但是「字符」串有一个好处，那就是可以快速定位。

> 其中 codepoint 是每个「字」的其实偏移量。Go 语言的字符串采用 utf8 编码，中文汉字通常需要占用 3 个字节，英文只需要 1 个字节。len() 函数得到的是字节的数量，通过下标来访问字符串得到的是「字节」。

```go
package main

import "fmt"

func main() {
    var s = "嘻哈china"
    for i:=0;i<len(s);i++ {
        fmt.Printf("%x ", s[i])
    }

}

// -----------
// e5 98 bb e5 93 88 63 68 69 6e 61
```

> 按字符 rune 遍历

```go
package main

import "fmt"

func main() {
    var s = "嘻哈china"
    for codepoint, runeValue := range s {
        fmt.Printf("%d %d ", codepoint, int32(runeValue))
    }
}

// -----------
// 0 22075 3 21704 6 99 7 104 8 105 9 110 10 97
```

> 对字符串进行 range 遍历，每次迭代出两个变量 codepoint 和 runeValue。codepoint 表示字符起始位置，runeValue 表示对应的 unicode 编码（类型是 rune）。

> 在使用 Go 语言进行网络编程时，经常需要将来自网络的字节流转换成内存字符串，同时也需要将内存字符串转换成网络字节流。Go 语言直接内置了字节切片和字符串的相互转换语法。

## 《快学 Go 语言》第 8 课 —— 结构体
> https://zhuanlan.zhihu.com/p/50654803

> Circle 结构体内部有三个变量，分别是圆心的坐标以及半径。特别需要注意是结构体内部变量的大小写，首字母大写是公开变量，首字母小写是内部变量，分别相当于类成员变量的 Public 和 Private 类别。内部变量只有属于同一个 package（简单理解就是同一个目录）的代码才能直接访问。

> 结构体的第二种创建形式是不指定字段名称来顺序字段初始化，需要显示提供所有字段的初值，一个都不能少。这种形式称之为「顺序形式」。

```go
package main

import "fmt"

type Circle struct {
    x int
    y int
    Radius int
}

func main() {
    var c Circle = Circle {100, 100, 50}
    fmt.Printf("%+v\n", c)
}

// -------
// {x:100 y:100 Radius:50}
```

> 最后我们再将三种零值初始化形式放到一起对比观察一下

```go
var c1 Circle = Circle{}
var c2 Circle
var c3 *Circle = new(Circle)
```

> nil 结构体是指结构体指针变量没有指向一个实际存在的内存。这样的指针变量只会占用 1 个指针的存储空间，也就是一个机器字的内存大小。

```go
var c *Circle = nil
```

> 结构体的拷贝

> 结构体之间可以相互赋值，它在本质上是一次浅拷贝操作，拷贝了结构体内部的所有字段。结构体指针之间也可以相互赋值，它在本质上也是一次浅拷贝操作，不过它拷贝的仅仅是指针地址值，结构体的内容是共享的。

> Go 语言的结构体方法里面没有 self 和 this 这样的关键字来指代当前的对象，它是用户自己定义的变量名称，通常我们都使用单个字母来表示。

> **结构体的指针方法**

> 如果使用上面的方法形式给 Circle 增加一个扩大半径的方法，你会发现半径扩大不了。
>这是因为上面的方法和前面的 expandByValue 函数是等价的，只不过是把函数的第一个参数挪了位置而已，参数传递时会复制了一份结构体内容，起不到扩大半径的效果。这时候就必须要使用结构体的指针方法

```go
func (c Circle) expand() {
  c.Radius *= 2
}

func (c *Circle) expand() {
  c.Radius *= 2
}
```

> 结构体指针方法和值方法在调用时形式上是没有区别的，只不过一个可以改变结构体内部状态，而另一个不会。指针方法使用结构体值变量可以调用，值方法使用结构体指针变量也可以调用。

> **匿名内嵌结构体**

> 还有一种特殊的内嵌结构体形式，内嵌的结构体不提供名称。这时外面的结构体将直接继承内嵌结构体所有的内部字段和方法，就好像把子结构体的一切全部都揉进了父结构体一样。匿名的结构体字段将会自动获得以结构体类型的名字命名的字段名称

```go
package main

import "fmt"

type Point struct {
    x int
    y int
}

func (p Point) show() {
    fmt.Println(p.x, p.y)
}

type Circle struct {
    Point // 匿名内嵌结构体
    Radius int
}

func main() {
    var c = Circle {
        Point: Point {
            x: 100,
            y: 100,
        },
        Radius: 50,
    }
    fmt.Printf("%+v\n", c)
    fmt.Printf("%+v\n", c.Point)
    fmt.Printf("%d %d\n", c.x, c.y) // 继承了字段
    fmt.Printf("%d %d\n", c.Point.x, c.Point.y)
    c.show() // 继承了方法
    c.Point.show()
}

// -------
// {Point:{x:100 y:100} Radius:50}
// {x:100 y:100}
// 100 100
// 100 100
// 100 100
// 100 100
```

> 这里的继承仅仅是形式上的语法糖，c.show() 被转换成二进制代码后和 c.Point.show() 是等价的，c.x 和 c.Point.x 也是等价的。

> Go 语言不是面向对象语言在于它的结构体不支持多态，它不能算是一个严格的面向对象语言。多态是指父类定义的方法可以调用子类实现的方法，不同的子类有不同的实现，从而给父类的方法带来了多样的不同行为。下面的例子呈现了 Java 类的多态性。

## 《快学 Go 语言》第 9 课 —— 接口
> https://zhuanlan.zhihu.com/p/50942676

> **空接口**

> 如果一个接口里面没有定义任何方法，那么它就是空接口，任意结构体都隐式地实现了空接口。

> **Go 语言为了避免用户重复定义很多空接口，它自己内置了一个，这个空接口的名字特别奇怪，叫 interface{} ，初学者会非常不习惯。之所以这个类型名带上了大括号，那是在告诉用户括号里什么也没有。我始终认为这种名字很古怪，它让代码看起来有点丑陋。**

> 空接口里面没有方法，所以它也不具有任何能力，其作用相当于 Java 的 Object 类型，可以容纳任意对象，它是一个万能容器。比如一个字典的 key 是字符串，但是希望 value 可以容纳任意类型的对象，类似于 Java 语言的 Map 类型，这时候就可以使用空接口类型 interface{}。

> **接口变量的本质**

> 在使用接口时，我们要将接口看成一个特殊的容器，这个容器只能容纳一个对象，只有实现了这个接口类型的对象才可以放进去。

> 接口变量作为变量来说它也是需要占据内存空间的，通过翻阅 Go 语言的源码可以发现，接口变量也是由结构体来定义的，这个结构体包含两个指针字段，一个字段指向被容纳的对象内存，另一个字段指向一个特殊的结构体 itab，这个特殊的结构体包含了接口的类型信息和被容纳对象的数据类型信息。

> 接口变量的赋值

> 变量赋值本质上是一次内存浅拷贝，切片的赋值是拷贝了切片头，字符串的赋值是拷贝了字符串的头部，而数组的赋值呢是直接拷贝整个数组。接口变量的赋值会不会不一样呢？接下来我们做一个实验

```go
package main

import "fmt"

type Rect struct {
    Width int
    Height int
}

func main() {
    var a interface {}
    var r = Rect{50, 50}
    a = r

    var rx = a.(Rect)
    r.Width = 100
    r.Height = 100
    fmt.Println(rx)
}

// ------
// {50 50}
```

> 指向指针的接口变量

>  如果将上面的例子改成指针，将接口变量指向结构体指针，那结果就不一样了

```go
package main

import "fmt"

type Rect struct {
    Width int
    Height int
}

func main() {
    var a interface {}
    var r = Rect{50, 50}
    a = &r // 指向了结构体指针

    var rx = a.(*Rect) // 转换成指针类型
    r.Width = 100
    r.Height = 100
    fmt.Println(rx)
}

// -------
// &{100 100}
```

> 从输出结果中可以看出指针变量 rx 指向的内存和变量 r 的内存是同一份。因为在类型转换的过程中只发生了指针变量的内存复制，而指针变量指向的内存是共享的。

## 《快学 Go 语言》第 10 课 —— 错误与异常
> https://zhuanlan.zhihu.com/p/51164928

> Go 语言规定凡是实现了错误接口的对象都是错误对象，这个错误接口只定义了一个方法。

```go
type error interface {
  Error() string
}
```

## 《快学 Go 语言》第 11 课 —— 千军万马跑协程
> https://zhuanlan.zhihu.com/p/51516757

```go
package main

import "fmt"
import "time"

func main() {
    fmt.Println("run in main goroutine")
    n := 3
    for i:=0; i<n; i++ {
        go func() {
            fmt.Println("dead loop goroutine start")
            for {}  // 死循环
        }()
    }
    for {
        time.Sleep(time.Second)
        fmt.Println("main goroutine running")
    }
}
```

> 通过调整上面代码中的变量 n 的值可以发现一个有趣的现象，当 n 值大于 3 时，主协程将没有机会得到运行，而如果 n 值为 3、2、1，主协程依然可以每秒输出一次。要解释这个现象就必须深入了解协程的运行原理

- 这里的n设定为cpu逻辑核心数

> **操作系统对线程的调度是抢占式的，也就是说单个线程的死循环不会影响其它线程的执行，每个线程的连续运行受到时间片的限制。**

> **Go 语言运行时对协程的调度并不是抢占式的。如果单个协程通过死循环霸占了线程的执行权，那这个线程就没有机会去运行其它协程了，你可以说这个线程假死了。不过一个进程内部往往有多个线程，假死了一个线程没事，全部假死了才会导致整个进程卡死。**

## 《快学 Go 语言》第 12 课 —— 通道
> https://zhuanlan.zhihu.com/p/51710515

> 不同的并行协程之间交流的方式有两种，一种是通过共享变量，另一种是通过队列。Go 语言鼓励使用队列的形式来交流，它单独为协程之间的队列数据交流定制了特殊的语法 —— 通道。

> 通道是协程的输入和输出。作为协程的输出，通道是一个容器，它可以容纳数据。作为协程的输入，通道是一个生产者，它可以向协程提供数据。通道作为容器是有限定大小的，满了就写不进去，空了就读不出来。通道还有它自己的类型，它可以限定进入通道的数据的类型。

> 创建通道只有一种语法，那就是 make 全局函数，提供第一个类型参数限定通道可以容纳的数据类型，再提供第二个整数参数作为通道的容器大小。大小参数是可选的，如果不填，那这个通道的容量为零，叫着「非缓冲型通道」，非缓冲型通道必须确保有协程正在尝试读取当前通道，否则写操作就会阻塞直到有其它协程来从通道中读东西。非缓冲型通道总是处于既满又空的状态。与之对应的有限定大小的通道就是缓冲型通道。在 Go 语言里不存在无界通道，每个通道都是有限定最大容量的。

> 通道作为容器，它可以像切片一样，使用 cap() 和 len() 全局函数获得通道的容量和当前内部的元素个数。通道一般作为不同的协程交流的媒介，在同一个协程里它也是可以使用的。

> **读写阻塞**

> **通道满了，写操作就会阻塞，协程就会进入休眠，直到有其它协程读通道挪出了空间，协程才会被唤醒。如果有多个协程的写操作都阻塞了，一个读操作只会唤醒一个协程。**
> **通道空了，读操作就会阻塞，协程也会进入睡眠，直到有其它协程写通道装进了数据才会被唤醒。如果有多个协程的读操作阻塞了，一个写操作也只会唤醒一个协程。**

> **关闭通道**

> **Go 语言的通道有点像文件，不但支持读写操作， 还支持关闭。读取一个已经关闭的通道会立即返回通道类型的「零值」，而写一个已经关闭的通道会抛异常。如果通道里的元素是整型的，读操作是不能通过返回值来确定通道是否关闭的。**

> **当通道空了，循环会暂停阻塞，当通道关闭时，阻塞停止，循环也跟着结束了。当循环结束时，我们就知道通道已经关闭了。**

> **通道写安全**

> **确保通道写安全的最好方式是由负责写通道的协程自己来关闭通道，读通道的协程不要去关闭通道。**

```go
package main

import "fmt"

func send(ch chan int) {
 ch <- 1
 ch <- 2
 ch <- 3
 ch <- 4
 close(ch)
}

func recv(ch chan int) {
 for v := range ch {
  fmt.Println(v)
 }
}

func main() {
 var ch = make(chan int, 1)
 go send(ch)
 recv(ch)
}

// -----------
// 1
// 2
// 3
// 4
```

> **这个方法确实可以解决单写多读的场景，可要是遇上了多写单读的场合该怎么办呢？任意一个读写通道的协程都不可以随意关闭通道，否则会导致其它写通道协程抛出异常。这时候就必须让其它不相干的协程来干这件事，这个协程需要等待所有的写通道协程都结束运行后才能关闭通道。那其它协程要如何才能知道所有的写通道已经结束运行了呢？这个就需要使用到内置 sync 包提供的 WaitGroup 对象，它使用计数来等待指定事件完成。**

```go
package main

import "fmt"
import "time"
import "sync"

func send(ch chan int, wg *sync.WaitGroup) {
 defer wg.Done() // 计数值减一
 i := 0
 for i < 4 {
  i++
  ch <- i
 }
}

func recv(ch chan int) {
 for v := range ch {
  fmt.Println(v)
 }
}

func main() {
 var ch = make(chan int, 4)
 var wg = new(sync.WaitGroup)
 wg.Add(2) // 增加计数值
 go send(ch, wg)  // 写
 go send(ch, wg)  // 写
 go recv(ch)
 // Wait() 阻塞等待所有的写通道协程结束
 // 待计数值变成零，Wait() 才会返回
 wg.Wait()
 // 关闭通道
 close(ch)
 time.Sleep(time.Second)
}

// ---------
// 1
// 2
// 3
// 4
// 1
// 2
// 3
// 4
```

> 多路通道

> 在真实的世界中，还有一种消息传递场景，那就是消费者有多个消费来源，只要有一个来源生产了数据，消费者就可以读这个数据进行消费。这时候可以将多个来源通道的数据汇聚到目标通道，然后统一在目标通道进行消费。

```go
package main

import "fmt"
import "time"

// 每隔一会生产一个数
func send(ch chan int, gap time.Duration) {
 i := 0
 for {
  i++
  ch <- i
  time.Sleep(gap)
 }
}

// 将多个原通道内容拷贝到单一的目标通道
func collect(source chan int, target chan int) {
 for v := range source {
  target <- v
 }
}

// 从目标通道消费数据
func recv(ch chan int) {
 for v := range ch {
  fmt.Printf("receive %d\n", v)
 }
}


func main() {
 var ch1 = make(chan int)
 var ch2 = make(chan int)
 var ch3 = make(chan int)
 go send(ch1, time.Second)
 go send(ch2, 2 * time.Second)
 go collect(ch1, ch3)
 go collect(ch2, ch3)
 recv(ch3)
}

// ---------
// receive 1
// receive 1
// receive 2
// receive 2
// receive 3
// receive 4
// receive 3
// receive 5
// receive 6
// receive 4
// receive 7
// receive 8
// receive 5
// receive 9
// ....
```

> 但是上面这种形式比较繁琐，需要为每一种消费来源都单独启动一个汇聚协程。Go 语言为这种使用场景带来了「多路复用」语法糖，也就是下面要讲的 select 语句，它可以同时管理多个通道读写，如果所有通道都不能读写，它就整体阻塞，只要有一个通道可以读写，它就会继续。下面我们使用 select 语句来简化上面的逻辑

```go
package main

import "fmt"
import "time"

func send(ch chan int, gap time.Duration) {
 i := 0
 for {
  i++
  ch <- i
  time.Sleep(gap)
 }
}

func recv(ch1 chan int, ch2 chan int) {
 for {
  select {
   case v := <- ch1:
    fmt.Printf("recv %d from ch1\n", v)
   case v := <- ch2:
    fmt.Printf("recv %d from ch2\n", v)
  }
 }
}

func main() {
 var ch1 = make(chan int)
 var ch2 = make(chan int)
 go send(ch1, time.Second)
 go send(ch2, 2 * time.Second)
 recv(ch1, ch2)
}

// ------------
// recv 1 from ch2
// recv 1 from ch1
// recv 2 from ch1
// recv 3 from ch1
// recv 2 from ch2
// recv 4 from ch1
// recv 3 from ch2
// recv 5 from ch1
```

> 上面是多路复用 select 语句的读通道形式，下面是它的写通道形式，只要有一个通道能写进去，它就会打破阻塞。

```go
select {
  case ch1 <- v:
      fmt.Println("send to ch1")
  case ch2 <- v:
      fmt.Println("send to ch2")
}
```

> 前面我们讲的读写都是阻塞读写，Go 语言还提供了通道的非阻塞读写。当通道空时，读操作不会阻塞，当通道满时，写操作也不会阻塞。非阻塞读写需要依靠 select 语句的 default 分支。当 select 语句所有通道都不可读写时，如果定义了 default 分支，那就会执行 default 分支逻辑，这样就起到了不阻塞的效果。下面我们演示一个单生产者多消费者的场景。生产者同时向两个通道写数据，写不进去就丢弃。

```go
package main

import "fmt"
import "time"

func send(ch1 chan int, ch2 chan int) {
 i := 0
 for {
  i++
  select {
   case ch1 <- i:
    fmt.Printf("send ch1 %d\n", i)
   case ch2 <- i:
    fmt.Printf("send ch2 %d\n", i)
   default:
  }
 }
}

func recv(ch chan int, gap time.Duration, name string) {
 for v := range ch {
  fmt.Printf("receive %s %d\n", name, v)
  time.Sleep(gap)
 }
}

func main() {
 // 无缓冲通道
 var ch1 = make(chan int)
 var ch2 = make(chan int)
 // 两个消费者的休眠时间不一样，名称不一样
 go recv(ch1, time.Second, "ch1")
 go recv(ch2, 2 * time.Second, "ch2")
 send(ch1, ch2)
}

// ------------
// send ch1 27
// send ch2 28
// receive ch1 27
// receive ch2 28
// send ch1 6708984
// receive ch1 6708984
// send ch2 13347544
// send ch1 13347775
// receive ch2 13347544
// receive ch1 13347775
// send ch1 20101642
// receive ch1 20101642
// send ch2 26775795
// receive ch2 26775795
// ...
```

- 注意此处，在非阻塞的channel中，往一个不带缓冲的队列中写入数据时可能会失败（数据被丢弃）

> 通道在其它语言里面的表现形式是队列，在 Java 语言里，带缓冲通道就是并发包内置的 java.util.concurrent.ArrayBlockingQueue，无缓冲通道也是并发包内置的 java.util.concurrent.SynchronousQueue。ArrayBlockingQueue 的内部实现形式是一个数组，多线程读写时需要使用锁来控制并发访问。不过像 Go 语言提供的多路复用效果，Java 语言就没有内置的实现了。

> Go 语言的通道内部结构是一个循环数组，通过读写偏移量来控制元素发送和接受。它为了保证线程安全，内部会有一个全局锁来控制并发。对于发送和接受操作都会有一个队列来容纳处于阻塞状态的协程。

```go
type hchan struct {
  qcount uint  // 通道有效元素个数
  dataqsize uint   // 通道容量，循环数组总长度
  buf unsafe.Pointer // 数组地址
  elemsize uint16 // 内部元素的大小
  closed uint32 // 是否已关闭 0或者1
  elemtype *_type // 内部元素类型信息
  sendx uint // 循环数组的写偏移量
  recvx uint // 循环数组的读偏移量
  recvq waitq // 阻塞在读操作上的协程队列
  sendq waitq // 阻塞在写操作上的协程队列
  
  lock mutex // 全局锁
}
```

## 《快学 Go 语言》第 13 课 —— 并发与安全
> https://zhuanlan.zhihu.com/p/52376005

> **日常应用中，大多数并发数据结构都是读多写少的，对于读多写少的场合，可以将互斥锁换成读写锁，可以有效提升性能。sync 包也提供了读写锁对象 RWMutex，不同于互斥锁只有两个常用方法 Lock() 和 Unlock()，读写锁提供了四个常用方法，分别是写加锁 Lock()、写释放锁 Unlock()、读加锁 RLock() 和读释放锁 RUnlock()。写锁是排他锁，加写锁时会阻塞其它协程再加读锁和写锁，读锁是共享锁，加读锁还可以允许其它协程再加读锁，但是会阻塞加写锁。**

> 读写锁在写并发高的情况下性能退化为普通的互斥锁。

## 《快学 Go 语言》第 14 课 —— 魔术变性指针
> https://zhuanlan.zhihu.com/p/52756600

> 在 Go 语言里不同类型之间的转换是要受限的。普通的基础变量转换成不同的类型需要进行内存浅拷贝，而指针变量类型之间是禁止直接转换的。要打破这个限制，unsafe.Pointer 就可以派上用场，它允许任意指针类型的互转。

```go
package main

import "fmt"
import "unsafe"

type Rect struct {
    Width int
    Height int
}

func main() {
    var r = Rect {50, 50}
    // var pw *int
    var pw = (*int)(unsafe.Pointer(&r))
    // var ph *int
    var ph = (*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&r)) + unsafe.Offsetof(r.Height))
    *pw = 100
    *ph = 100
    fmt.Println(r.Width, r.Height)
}

// --------
// 100 100
```

> 从上面 5 个 问题，我们可以得出结论，接口类型和结构体类型似乎是两个不同的世界。只有接口类型之间的赋值和转换会共享数据，其它情况都会复制数据，其它情况包括结构体之间的赋值，结构体转接口，接口转结构体。不同接口变量之间的转换本质上只是调整了接口变量内部的类型指针，数据指针并不会发生改变。

## 《快学 Go 语言》第 15 课 —— 反射
> https://zhuanlan.zhihu.com/p/53114706

> reflect.Kind，reflect 包定义了十几种内置的「元类型」，每一种元类型都有一个整数编号，这个编号使用 reflect.Kind 类型表示。不同的结构体是不同的类型，但是它们都是同一个元类型 Struct。包含不同子元素的切片也是不同的类型，但是它们都会同一个元类型 Slice。

```go
type Kind uint

const (
    Invalid Kind = iota // 不存在的无效类型
    Bool
    Int
    Int8
    Int16
    Int32
    Int64
    Uint
    Uint8
    Uint16
    Uint32
    Uint64
    Uintptr // 指针的整数类型，对指针进行整数运算时使用
    Float32
    Float64
    Complex64
    Complex128
    Array // 数组类型
    Chan // 通道类型
    Func  // 函数类型
    Interface  // 接口类型
    Map // 字典类型
    Ptr // 指针类型
    Slice // 切片类型
    String // 字符串类型
    Struct // 结构体类型
    UnsafePointer // unsafe.Pointer 类型
)
```

> 它是一个接口类型，里面定义了非常多的方法用于获取和这个类型相关的一切信息。这个接口的结构体实现隐藏在 reflect 包里，每一种类型都有一个相关的类型结构体来表达它的结构信息。

```go
type Type interface {
  ...
  Method(i int) Method  // 获取挂在类型上的第 i'th 个方法
  ...
  NumMethod() int  // 该类型上总共挂了几个方法
  Name() string // 类型的名称
  PkgPath() string // 所在包的名称
  Size() uintptr // 占用字节数
  String() string // 该类型的字符串形式
  Kind() Kind // 元类型
  ...
  Bits() // 占用多少位
  ChanDir() // 通道的方向
  ...
  Elem() Type // 数组，切片，通道，指针，字典(key)的内部子元素类型
  Field(i int) StructField // 获取结构体的第 i'th 个字段
  ...
  In(i int) Type  // 获取函数第 i'th 个参数类型
  Key() Type // 字典的 key 类型
  Len() int // 数组的长度
  NumIn() int // 函数的参数个数
  NumOut() int // 函数的返回值个数
  Out(i int) Type // 获取函数 第 i'th 个返回值类型
  common() *rtype // 获取类型结构体的共同部分
  uncommon() *uncommonType // 获取类型结构体的不同部分
}
```

> 所有的类型结构体都包含一个共同的部分信息，这部分信息使用 rtype 结构体描述，rtype 实现了 Type 接口的所有方法。剩下的不同的部分信息各种特殊类型结构体都不一样。可以将 rtype 理解成父类，特殊类型的结构体是子类，会有一些不一样的字段信息。

```go
// 基础类型 rtype 实现了 Type 接口
type rtype struct {
  size uintptr // 占用字节数
  ptrdata uintptr
  hash uint32 // 类型的hash值
  ...
  kind uint8 // 元类型
  ...
}

// 切片类型
type sliceType struct {
  rtype
  elem *rtype // 元素类型
}

// 结构体类型
type structType struct {
  rtype
  pkgPath name  // 所在包名
  fields []structField  // 字段列表
}

...
```

> 不同于 reflect.Type 接口，reflect.Value 是结构体类型，一个非常简单的结构体。

```go
type Value struct {
  typ *rtype  // 变量的类型结构体
  ptr unsafe.Pointer // 数据指针
  flag uintptr // 标志位
}
```