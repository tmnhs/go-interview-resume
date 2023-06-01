# 基础语法

### 01 `=` 和 `:=` 的区别？

=是赋值变量，:=是定义变量。

### 02 指针的作用

一个指针可以指向任意变量的地址，它所指向的地址在32位或64位机器上分别**固定**占4或8个字节。指针的作用有：

- 获取变量的值

```go
 import fmt
 
 func main(){
  a := 1
  p := &a//取址&
  fmt.Printf("%d\n", *p);//取值*
 }
```

- 改变变量的值

```
 // 交换函数
 func swap(a, b *int) {
     *a, *b = *b, *a
 }
```

- 用指针替代值传入函数，比如类的接收器就是这样的。

```
 type A struct{}
 
 func (a *A) fun(){}
```

### 03 Go 允许多个返回值吗？多返回值怎么实现的？

可以。通常函数除了一般返回值还会返回一个error。

**实现原理**

> FP 栈底寄存器，指向一个函数栈的底部;PC 程序计数器，指向下一条执行指令;SB 指向静态数据的基指针，全局符号;SP 栈顶寄存器。

Go 传参和返回值是通过 FP+offset 实现，并且存储在调用函数的栈帧中。

### 04 Go 有异常类型吗？

有。Go用error类型代替try...catch语句，这样可以节省资源。同时增加代码可读性：

```
 _, err := funcDemo()
if err != nil {
    fmt.Println(err)
    return
}
```

也可以用errors.New()来定义自己的异常。errors.Error()会返回异常的字符串表示。只要实现error接口就可以定义自己的异常，

```
 type errorString struct {
  s string
 }
 
 func (e *errorString) Error() string {
  return e.s
 }
 
 // 多一个函数当作构造函数
 func New(text string) error {
  return &errorString{text}
 }
```

### 05  ❤ 什么是协程（Goroutine）。进程、线程、协程有什么区别？（必问）

进程：是应用程序的启动实例，**进程是资源调度的基本单位**，运行一个可执行程序会创建一个或多个进程 。

线程：线程是程序执行（CPU调度）的基本单位，是轻量级的进程

协程：**用户态轻量级线程**，它是**线程调度的基本单位**。通常在函数前加上go关键字就能实现并发。一个Goroutine会以一个很小的栈启动2KB或4KB，当遇到栈空间不足时，栈会**自动伸缩**， 因此可以轻易实现成千上万个goroutine同时启动。

### 06 ❤ 如何高效地拼接字符串

拼接字符串的方式有：`+` , `fmt.Sprintf` , `strings.Builder`, `bytes.Buffer`, `strings.Join`

1 "+"

使用`+`操作符进行拼接时，会对字符串进行遍历，计算并开辟一个新的空间来存储原来的两个字符串。

2 fmt.Sprintf

由于采用了接口参数，必须要用反射获取值，因此有性能损耗。

3 strings.Builder：

用WriteString()进行拼接，内部实现是指针+切片，同时String()返回拼接后的字符串，它是直接把[]byte转换为string，从而避免变量拷贝。

4 bytes.Buffer

`bytes.Buffer`是一个一个缓冲`byte`类型的缓冲器，这个缓冲器里存放着都是`byte`，

`bytes.buffer`底层也是一个`[]byte`切片。

5 strings.join

`strings.join`也是基于`strings.builder`来实现的,并且可以自定义分隔符，在join方法内调用了b.Grow(n)方法，这个是进行初步的容量分配，而前面计算的n的长度就是我们要拼接的slice的长度，因为我们传入切片长度固定，所以提前进行容量分配可以减少内存分配，很高效。

**性能比较**：

strings.Join ≈ strings.Builder > bytes.Buffer > "+" > fmt.Sprintf

5种拼接方法的实例代码

```
func main(){
	a := []string{"a", "b", "c"}
	//方式1：+
	ret := a[0] + a[1] + a[2]
	//方式2：fmt.Sprintf
	ret := fmt.Sprintf("%s%s%s", a[0],a[1],a[2])
	//方式3：strings.Builder
	var sb strings.Builder
	sb.WriteString(a[0])
	sb.WriteString(a[1])
	sb.WriteString(a[2])
	ret := sb.String()
	//方式4：bytes.Buffer
	buf := new(bytes.Buffer)
	buf.Write(a[0])
	buf.Write(a[1])
	buf.Write(a[2])
	ret := buf.String()
	//方式5：strings.Join
	ret := strings.Join(a,"")
}

```

> 参考资料：[字符串拼接性能及原理 | Go 语言高性能编程 | 极客兔兔](https://link.zhihu.com/?target=https%3A//geektutu.com/post/hpg-string-concat.html)

### 07 什么是 rune 类型

golang中的字符串底层实现是通过byte数组的，中文字符在unicode下占2个字节，在utf-8编码下占3个字节，而golang默认编码正好是utf-8

byte 等同于int8，常用来处理ascii字符

rune 等同于int32,常用来处理unicode或utf-8字符

```go
sample := "我爱GO"
runeSamp := []rune(sample)
runeSamp[0] = '你'
fmt.Println(string(runeSamp))  // "你爱GO"
fmt.Println(len(runeSamp))  // 4

```

### 08 如何判断 map 中是否包含某个 key ？

```go
var sample map[int]int
if _, ok := sample[10]; ok {

} else {

}

```

### 09 Go 支持默认参数或可选参数吗？

不支持。但是可以利用**结构体参数**，或者传入参数**切片数组**。

可选参数的话可以使用**选项模式**

```go
// 这个函数可以传入任意数量的整型参数
func sum(nums ...int) {
    total := 0
    for _, num := range nums {
        total += num
    }
    fmt.Println(total)
}
```

### 10 defer 的执行顺序

defer执行顺序和调用顺序相反，类似于栈**后进先出**(LIFO)。

defer在return之后执行，但在函数退出之前，defer可以修改返回值（对于有名返回值）。下面是一个例子：

```go
func test() int {
	i := 0
	defer func() {
		fmt.Println("defer1")
	}()
	defer func() {
		i += 1
		fmt.Println("defer2")
	}()
	return i
}

func main() {
	fmt.Println("return", test())
}
// defer2
// defer1
// return 0

```

上面这个例子中，test返回值并没有修改，这是由于Go的返回机制决定的，执行Return语句后，Go会创建一个临时变量保存返回值。如果是有名返回（也就是指明返回值`func test() (i int)`）

```
func test() (i int) {
	i = 0
	defer func() {
		i += 1
		fmt.Println("defer2")
	}()
	return i
}

func main() {
	fmt.Println("return", test())
}
// defer2
// return 1

```

这个例子中，返回值被修改了。对于有名返回值的函数，执行 return 语句时，并不会再创建临时变量保存，因此，defer 语句修改了 i，即对返回值产生了影响。

### 11 如何交换 2 个变量的值？

对于变量而言`a,b = b,a`； 对于指针而言`*a,*b = *b, *a`

### 12 Go 语言 tag 的用处？

tag可以为结构体成员提供属性。常见的：

1. json序列化或反序列化时字段的名称
2. db: sqlx模块中对应的数据库字段名
3. form: gin框架中对应的前端的数据字段名
4. binding: 搭配 form 使用, 默认如果没查找到结构体中的某个字段则不报错值为空, binding为 required 代表没找到返回错误给前端

### 13 ❤如何获取一个结构体的tag？tag是怎么实现的？

利用反射：

```go
import reflect
type Author struct {
	Name         int      `json:"jsonname"`
	Publications []string `json:"jsonpublication"`
}

func main() {
	t := reflect.TypeOf(Author{})
	for i := 0; i < t.NumField(); i++ {
		s := t.Field(i).Tag
		fmt.Println(s.Get("json"))
	}
}

```

上述例子中，`reflect.TypeOf`方法获取对象的类型，之后`NumField()`获取结构体成员的数量。 通过`Field(i)`获取第i个成员的名字。 再通过其`Tag` 方法获得标签。

Go 中解析的 tag 是通过**反射**实现的。

### 14 如何判断 2 个字符串切片（slice) 是相等的？

`reflect.DeepEqual()` ， 但反射非常影响性能。

### 15 结构体打印时，`%v` 和 `%+v` 的区别

`%v`输出结构体各成员的值；

`%+v`输出结构体各成员的**名称**和**值**；

`%#v`输出结构体名称和结构体各成员的名称和值

### 16 Go 语言中如何表示枚举值(enums)？

在常量中用iota可以表示枚举。iota从0开始。

```go
const (
	B = 1 << (10 * iota)
	KiB 
	MiB
	GiB
	TiB
	PiB
	EiB
)
```

### 17 空 struct{} 的用途

- 用map模拟一个set，那么就要把值置为struct{}，struct{}本身不占任何空间，可以避免任何多余的内存分配。

```go
type Set map[string]struct{}

func main() {
	set := make(Set)

	for _, item := range []string{"A", "A", "B", "C"} {
		set[item] = struct{}{}
	}
	fmt.Println(len(set)) // 3
	if _, ok := set["A"]; ok {
		fmt.Println("A exists") // A exists
	}
}
```

- 有时候给通道发送一个空结构体实现并发控制,channel<-struct{}{}，也是节省了空间。

```go
func main() {
	ch := make(chan struct{}, 1)
	go func() {
		<-ch
		// do something
	}()
	ch <- struct{}{}
	// ...
}
```

- 仅有方法的结构体

```go
type Lamp struct{}
```

### **18 go里面的int和int32是同一个概念吗？**

不是一个概念！千万不能混淆。go语言中的int的大小是和操作系统位数相关的，如果是32位操作系统，int类型的大小就是4字节。如果是64位操作系统，int类型的大小就是8个字节。除此之外uint也与操作系统有关。

int8占1个字节，int16占2个字节，int32占4个字节，int64占8个字节。

### 19❤new和make的区别（基本必问）？

1）**作用变量类型**不同，new给string,int和数组分配内存，make给切片，map，channel分配内存；

2）**返回类型**不一样，new返回指向变量的指针，make返回变量本身；

3）new 分配的空间被清零（分配的内存置为零，也就是类型的零值）。make 分配空间后，会进行初始化，但是不是置为零值；

4) 字节的面试官还说了另外一个区别，就是分配的位置，在堆上还是在栈上？这块我比较模糊，大家可以自己探究下，我搜索出来的答案是golang会弱化分配的位置的概念，因为编译的时候会自动内存逃逸处理，懂的大佬帮忙补充下：make、new内存分配是在堆上还是在栈上？

### 20请你讲一下Go面向对象是如何实现的？

Go实现面向对象的两个关键是**struct和interface**。

封装：对于同一个包，对象对包内的文件可见；对不同的包，需要将对象以大写开头才是可见的。

[^封装]: 两层含义：一层含义是把对象的属性和行为看成一个密不可分的整体，将这两者“封装”在一个不可分割的独立单元(即对象)中；另一层含义指“信息隐藏”，把不需要让外界知道的信息隐藏起来，有些对象的属性及行为允许外界用户知道或使用，但不允许更改，而另一些属性或行为，则不允许外界知晓，或只允许使用对象的功能，而尽可能隐藏对象的功能实现细节。

继承：继承是编译时特征，在struct内加入所需要继承的类即可：

```
type A struct{}
type B struct{
A
}
```

多态：多态是运行时特征，Go多态通过interface来实现。类型和接口是松耦合的，某个类型的实例可以赋给它所实现的任意接口类型的变量。

[^多态]: 多态是同一个行为具有多个不同表现形式或形态的能力。

Go支持多重继承，就是在类型中嵌入所有必要的父类型。

### 21uint型变量值分别为 1，2，它们相减的结果是多少？

```
	var a uint = 1
	var b uint = 2
	fmt.Println(a - b)
```

答案，结果会溢出，如果是32位系统，结果是2^32-1，如果是64位系统，结果2^64-1.

### 22讲一下go有没有函数在main之前执行？怎么用？

go的init函数在main函数之前执行

```go
func init() {
	...
}
```

**怎么用**：

- 初始化不能采用初始化表达式初始化的变量；
- 程序运行前执行注册
- 实现sync.Once功能
- 不能被其它函数调用，init函数没有入口参数和返回值：
- 每个包可以有多个init函数，**每个源文件也可以有多个init函数**。
- 同一个包的init执行顺序，golang没有明确定义，编程时要注意程序不要依赖这个执行顺序。
- 不同包的init函数按照包导入的依赖关系决定执行顺序。

**go初始化**：

init()函数是go初始化的一部分，由runtime初始化每个导入的包，初始化不是按照从上到下的导入顺序，而是按照解析的依赖关系，没有依赖的包最先初始化。

每个包首先初始化包作用域的常量和变量（常量优先于变量），然后执行包的`init()`函数。

执行顺序：**import –> const –> var –>`init()`–>`main()**



### 23下面这句代码是什么作用，为什么要定义一个空值？

```
type GobCodec struct{
	conn io.ReadWriteCloser
	buf *bufio.Writer
	dec *gob.Decoder
	enc *gob.Encoder
}

type Codec interface {
	io.Closer
	ReadHeader(*Header) error
	ReadBody(interface{})  error
	Write(*Header, interface{}) error
}

var _ Codec = (*GobCodec)(nil)
```

答：将nil转换为*GobCodec类型，然后再转换为Codec接口，如果转换失败，说明*GobCodec没有实现Codec接口的所有方法，用来**判断GobCodec是否实现了Codec接口的所有方法**。



### 24如果若干个goroutine，有一个panic会怎么做？

有一个panic，那么剩余goroutine也会退出，程序退出。如果不想程序退出，那么必须通过调用 recover() 方法来捕获 panic 并恢复将要崩掉的程序。

> 参考理解：[goroutine配上panic会怎样](https://link.zhihu.com/?target=https%3A//blog.csdn.net/huorongbj/article/details/123013273)。

### 25defer可以捕获goroutine的子goroutine吗？

不可以。它们处于不同的调度器P中。对于子goroutine，必须通过recover()机制来进行恢复，然后结合日志进行打印（或者通过channel传递error），下面是一个例子：

```go
// 心跳函数
func Ping(ctx context.Context) error {
    ... code ...
 
	go func() {
		defer func() {
			if r := recover(); r != nil {
				log.Errorc(ctx, "ping panic: %v, stack: %v", r, string(debug.Stack()))
			}
		}()
 
        ... code ...
	}()
 
    ... code ...
 
	return nil
}
```

### 27channel 死锁的场景

> channle中的死锁，是指在程序的主线程中发生的情况，如果上述的情况发生在非主线程中，读取或者写入的情况是发生堵塞的，而不是死锁。实际上，阻塞情况省去了我们加锁的步骤，反而是更加有利于代码编写，要合理的利用阻塞。。

- 当一个`channel`中没有数据，而直接读取时，会发生死锁：

```go
q := make(chan int,2)
<-q
```

​	解决方案是采用select语句，再default放默认处理方式：

```
q := make(chan int,2)
select{
   case val:=<-q:
   default:
         ...

}
```

- 当channel数据满了，再尝试写数据会造成死锁：

```
q := make(chan int,2)
q<-1
q<-2
q<-3
```

​	解决方法，采用select

```
func main() {
	q := make(chan int, 2)
	q <- 1
	q <- 2
	select {
	case q <- 3:
		fmt.Println("ok")
	default:
		fmt.Println("wrong")
	}

}
```

- 向一个关闭的channel写数据。（会panic）

  注意：一个已经关闭的channel，只能读数据，不能写数据。

参考资料：[Golang关于channel死锁情况的汇总以及解决方案](https://link.zhihu.com/?target=https%3A//blog.csdn.net/qq_35976351/article/details/81984117)

### 28对已经关闭的chan进行读写会怎么样？

- 读已经关闭的chan能一直读到东西，但是读到的内容根据通道内关闭前是否有元素而不同。
  - 如果chan关闭前，buffer内有元素还未读,会正确读到chan内的值，且返回的第二个bool值（是否读成功）为true。
    - 如果chan关闭前，buffer内有元素已经被读完，chan内无值，接下来所有接收的值都会非阻塞直接成功，返回 channel 元素的零值，但是第二个bool值一直为false。


- 写已经关闭的chan会panic。

### 30 2 个 interface 可以比较吗 ？

Go 语言中，interface 的内部实现包含了 2 个字段，类型 `T` 和 值 `V`，interface 可以使用 `==` 或 `!=` 比较。2 个 interface 相等有以下 2 种情况

1. 两个 interface 均等于 nil（此时 V 和 T 都处于 unset 状态）
2. 类型 T 相同，且对应的值 V 相等。

看下面的例子：

```go
type Stu struct {
     Name string
}

type StuInt interface{}

func main() {
     var stu1, stu2 StuInt = &Stu{"Tom"}, &Stu{"Tom"}
     var stu3, stu4 StuInt = Stu{"Tom"}, Stu{"Tom"}
     fmt.Println(stu1 == stu2) // false
     fmt.Println(stu3 == stu4) // true
}

```

`stu1` 和 `stu2` 对应的类型是 `*Stu`，值是 Stu 结构体的地址，两个地址不同，因此结果为 false。
`stu3` 和 `stu4` 对应的类型是 `Stu`，值是 Stu 结构体，且各字段相等，因此结果为 true。

### 31 2 个 nil 可能不相等吗？

可能不等。interface在运行时绑定值，只有值为nil接口值才为nil，但是与指针的nil不相等。举个例子：

```go
var p *int = nil
var i interface{} = nil
if(p == i){
	fmt.Println("Equal")
}
```

两者并不相同。总结：**两个nil只有在类型相同时才相等**。

### 32 函数返回局部变量的指针是否安全？

这一点和C++不同，在Go里面返回局部变量的指针是安全的。因为Go会进行**逃逸分析**，如果发现局部变量的作用域超过该函数则会**把指针分配到堆区**，避免内存泄漏。

### 34 非接口的任意类型 T() 都能够调用 `*T` 的方法吗？反过来呢？

一个T类型的值可以调用*T类型声明的方法，当且仅当T是**可寻址的**。（比如被gc掉了）

反之：*T 可以调用T()的方法，因为指针可以解引用。

### 35 go slice是怎么扩容的？

如果当前容量小于1024，则判断所需容量是否大于原来容量2倍，如果大于，当前容量加上所需容量；否则当前容量乘2。

如果当前容量大于1024，则每次按照1.25倍速度递增容量，也就是每次加上cap/4。

### 36进程被kill，如何保证所有goroutine顺利退出

goroutine监听SIGKILL信号，一旦接收到SIGKILL，则立刻退出。可采用select方法。

```go
var wg = &sync.WaitGroup{}

func main() {
	wg.Add(1)

	go func() {
		c1 := make(chan os.Signal, 1)
		signal.Notify(c1,syscall.SIGKILL, syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT)
		fmt.Printf("goroutine 1 receive a signal : %v\n\n", <-c1)
		wg.Done()
	}()

	wg.Wait()
	fmt.Printf("all groutine done!\n")
}
```

### **37❤数组和切片的区别 （基本必问）**

**相同点：**

1)只能存储一组**相同类型**的数据结构

2)都是通过下标来访问，并且有容量长度，长度通过 len 获取，容量通过 cap 获取

**区别：**

1）数组是**定长**，访问和复制不能超过数组定义的长度，否则就会下标越界，切片长度和容量可以**自动扩容**

2）**数组是值类型，切片是引用类型**，每个切片都引用了一个底层数组，切片本身不能存储任何数据，都是这底层数组存储数据，所以修改切片的时候修改的是底层数组中的数据。切片一旦扩容，指向一个新的底层数组，内存地址也就随之改变

**简洁的回答：**

1）定义方式不一样 2）初始化方式不一样，数组需要指定大小，大小不改变 3）在函数传递中，数组切片。

```go
//数组的定义
var a1 [3]int
var a2 [...]int{1,2,3}

//切片的定义

var a1 []int
var a2 :=make([]int,3,5)

//数组的初始化
a1 := [...]int{1,2,3}
a2 := [5]int{1,2,3}

//切片的初始化
b:= make([]int,3,5)
```

### 38for range 的时候它的地址会发生变化么？

答：不会，在 for a,b := range c 遍历中， a 和 b 在内存中只会存在一份，即之后每次循环时遍历到的数据都是以**值覆盖**的方式赋给 a 和 b，a，b 的内存地址**始终不变**。由于有这个特性，for 循环里面如果开协程，不要直接把 a 或者 b 的地址传给协程。

解决办法：在每次循环时，创建一个**临时变量**。

### **39go defer，多个 defer 的顺序，defer 在什么时机会修改返回值？**

作用：defer延迟函数，释放资源，收尾工作；如释放锁，关闭文件，关闭链接；捕获panic;

避坑指南：defer函数紧跟在资源打开后面，否则defer可能得不到执行，导致内存泄露。

多个 defer 调用顺序是 **LIFO（后入先出）**，defer后的操作可以理解为压入栈中

defer，return，return value（函数返回值） 执行顺序：首先return，其次return value，最后defer。defer可以修改函数最终返回值，

修改时机：**有名返回值或者函数返回指针**

### **40调用函数传入结构体时，应该传值还是指针？ （Golang 都是传值）**

> 值传递：指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。
>
> 引用传递是指在调用函数时将实际参数的**地址**传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数

**在 Golang 中所有函数参数传递都是值拷贝**，传指针只是拷贝了一份**指针副本**，同时指向原对象。

在函数传参过程中，需要合理使用传值、传指针。一般情况下，需要改变原始对象值、传递大的结构体，传指针是最合适的，因为传一个内存地址的开销很小。反之，如果变量不可变更、map 或 slice 应该选择传值方式。

### **41讲讲 Go 的 slice 底层数据结构和一些特性？**

Go 的 slice 底层数据结构是由一个 **array 指针指向底层数组**，len 表示切片长度，cap 表示切片容量。

slice 的主要实现是**扩容**。对于 append 向 slice 添加元素时，假如 slice 容量够用，则追加新元素进去，slice.len++，返回原来的 slice。当原容量不够，则 slice 先扩容，扩容之后 slice 得到新的 slice，将元素追加进新的 slice，slice.len++，返回新的 slice。

对于切片的扩容规则：

当切片比较小时（容量小于 1024），则采用较大的扩容倍速进行扩容（新的扩容会是原来的 2 倍），避免频繁扩容，从而减少内存分配的次数和数据拷贝的代价。

当切片较大的时（原来的 slice 的容量大于或者等于 1024），采用较小的扩容倍速（新的扩容将扩大大于或者等于原来 1.25 倍），主要避免空间浪费，网上其实很多总结的是 1.25 倍，那是在不考虑内存对齐的情况下，实际上还要考虑内存对齐，扩容是大于或者等于 1.25 倍。

（关于刚才问的 slice 为什么传到函数内可能被修改，如果 slice 在函数内没有出现扩容，函数外和函数内 slice 变量指向是同一个数组，则函数内复制的 slice 变量值出现更改，函数外这个 slice 变量值也会被修改。如果 slice 在函数内出现扩容，则函数内变量的值会新生成一个数组（也就是新的 slice，而函数外的 slice 指向的还是原来的 slice，则函数内的修改不会影响函数外的 slice。）

## map相关

### 42map 使用注意的点，是否并发安全？

map的类型是map[key]，key类型的key必须是可比较的，通常情况，会选择内建的基本类型，比如整数、字符串做key的类型。如果要使用struct作为key，要保证struct对象在逻辑上是不可变的。在Go语言中，map[key]函数返回结果可以是一个值，也可以是两个值。map是无序的，如果我们想要保证遍历map时元素有序，可以使用辅助的数据结构，例如orderedmap。

**第一，**一定要先**初始化**，否则panic

**第二，**map类型是容易发生**并发访问问题**的。不注意就容易发生程序运行时并发读写导致的panic。 Go语言内建的map对象不是线程安全的，并发读写的时候运行时会有检查，遇到并发问题就会导致panic。

### 43map 循环是有序的还是无序的？

**无序的,** map 因扩张⽽重新哈希时，各键值项存储位置都可能会发生改变，顺序自然也没法保证了，所以官方避免大家依赖顺序，直接打乱处理。就是 for range map 在开始处理循环逻辑的时候，就做了随机播种

### 44map 中删除一个 key，它的内存会释放么？（常问）

> 如果删除的元素是值类型，如int，float，bool，string以及数组和struct，map的内存不会自动释放
>
> 如果删除的元素是引用类型，如指针，slice，map，chan等，map的内存会自动释放，但释放的内存是子元素应用类型的内存占用
>
> 将map设置为nil后，内存被回收。
>
> **这个问题还需要大家去搜索下答案，我记得有不一样的说法，谨慎采用本题答案。**

以下是本人参考官方**https://github.com/golang/go/issues/20135**总结出来的，和上面的差不多，建议使用下面的说法

这个问题是关于哈希桶bucket的，包括标准桶和溢出桶。当从map中删除元素时，**map不会缩小哈希桶数或释放溢出桶**。**当元素被删除时，map将桶中的槽*清零* ，不会释放内存**。

键和值本身的空间不会被回收，因为该空间是哈希桶的一部分。只有键和值*引用*的东西才会被收集。

从理论上讲，map总是在增长。使用指针，我们能够收集指向的空间，但桶的大小永远不会。



### 4、怎么处理对 map 进行并发访问？有没有其他方案？ 区别是什么？

![img](https://pic2.zhimg.com/80/v2-1107961e741b834eb5fc071ff68da831_1440w.webp)

**方式一、使用内置sync.Map**

sync.Map支持并发读写，采取了**空间换时间**的机制，冗余了两个数据结构，分别是read和dirty

```go
type Map struct {
	mu Mutex

	// read contains the portion of the map's contents that are safe for
	// concurrent access (with or without mu held).
	// The read field itself is always safe to load, but must only be stored with
	// mu held.
	read atomic.Value // readOnly

	// dirty contains the portion of the map's contents that require mu to be
	// held. To ensure that the dirty map can be promoted to the read map quickly,
	// it also includes all of the non-expunged entries in the read map.
	dirty map[interface{}]*entry
	misses int
}

```

和原始map+RWLock实现并发的方式相比，减少了加锁对性能的影响，它做了一些优化，可以**无锁访问**read map,而且会优先操作read map,在某些特定场景（读多写少）中，他发生锁竞争的频率会远远小于map+RWLock的实现方式

**方式二、使用读写锁实现并发安全map**

并发下写多读少的优先考虑带锁map，读多写少的优先考虑sync.map

### 45nil map 和空 map 有何不同？

> nil map和empty map的关系，就像nil slice和empty slice一样，两者都是空对象，未存储任何数据，但前者不指向底层数据结构，后者指向底层数据结构，只不过指向的底层对象是空对象。
>
> ```go
> package main
> func main() {
>     var nil_map map[string]string
>     println(nil_map)
>  
>     emp_map := map[string]string{}
>     println(emp_map)
> }
> ```

**nil map 未初始化,等同于 var m map[string]int，空map是长度为空,空map表示map已经被初始化，只是长度为0，还并未赋于键值对,**

① 直接读取nil map：m[“a”] 并不会报错，会返回默认类型的空值
② 直接给nil map赋值：m[“a”] = 1 直接报错
③ 需要通过map == nil 来判断，是否为nil map

### 46slices能作为map类型的key吗？

当时被问的一脸懵逼，其实是这个问题的变种：golang 哪些类型可以作为map key？

答案是：**在golang规范中，可比较的类型都可以作为map key；**这个问题又延伸到在：golang规范中，哪些数据类型可以比较？

**不能作为map key 的类型包括：**

- slices
- maps
- functions

### 48讲讲 Go 中主协程如何等待其余协程退出?

答：Go 的 **sync.WaitGroup** 是等待一组协程结束，sync.WaitGroup 只有 3 个方法，Add()是添加计数，Done()减去一个计数，Wait()阻塞直到所有的任务完成。Go 里面还能通过有缓冲的 channel 实现其阻塞等待一组协程结束，这个不能保证一组 goroutine 按照顺序执行，可以并发执行协程。Go 里面能通过无缓冲的 channel 实现其阻塞等待一组协程结束，这个能保证一组 goroutine 按照顺序执行，但是不能并发执行。

**啰嗦一句：**循环智能二面，手写代码部分时，三个协程按交替顺序打印数字，最后题目做出来了，问我代码中Add()是什么意思，我回答的不是很清晰，这家公司就没有然后了。Add()表示协程计数，可以一次Add多个，如Add(3),可以多次Add(1);然后每个子协程必须调用done（）,这样才能保证所有子协程结束，主协程才能结束。

### 49Go 语言中不同的类型如何比较是否相等？

答：像 string，int，float interface 等可以通过 reflect.DeepEqual 和等于号进行比较，像 slice，struct，map 则一般使用 reflect.DeepEqual 来检测是否相等。

### 50Go 中 uintptr 和 unsafe.Pointer 的区别？

- unsafe.Pointer 是通用指针类型，它不能参与计算，任何类型的指针都可以转化成 unsafe.Pointer，unsafe.Pointer 可以转化成任何类型的指针，uintptr 可以转换为 unsafe.Pointer，unsafe.Pointer 可以转换为 uintptr。


- uintptr 是指针运算的工具，但是它不能持有指针对象（意思就是它跟指针对象不能互相转换），unsafe.Pointer 是指针对象进行运算（也就是 uintptr）的桥梁。

### 51.问个小细节， JSON 标准库对 nil slice 和 空 slice 的处理是一致的吗？

首先Go的JSON 标准库对 nil slice 和 空 slice 的处理是**不一致**。

- slice := make([]int,0）：slice不为nil，已经初始化，但是slice没有值，slice的底层的空间是空的。
- var slice []int ：slice的值是nil，未初始化，可用于需要返回slice的函数，当函数出现异常的时候，保证函数依然会有nil的返回值。

# 实现原理

## GC垃圾回收和内存管理

### 01 ❤简述 Go 语言GC(垃圾回收)的工作原理

> - 引用计数：每个对象维护一个引用计数，当被引用对象被创建或被赋值给其他对象时引用计数自动加 +1；如果这个对象被销毁，则计数 -1 ，当计数为 0 时，回收该对象，Python，PHP等语言使用。
>   - 优点：对象可以很快被回收，不会出现内存耗尽或到达阀值才回收。
>   - 缺点：不能很好的处理循环引用
>
>
> - 分代收集：按照对象生命周期长短划分不同的代空间，生命周期长的放入老年代，短的放入新生代，不同代有不同的回收算法和回收频率，java使用。
>   - 优点：回收性能好
>   - 缺点：算法复杂
>
> 引用计数和分代收集了解即可

垃圾回收机制是Go一大特(nan)色(dian)。Go1.3采用**标记清除法**， Go1.5采用**三色标记法**，Go1.8采用**三色标记法+混合写屏障**。

**1.标记清除法**

分为两个阶段：标记和清除

标记阶段：从**根对象**开始迭代遍历所有被引用的对象，对能够通过引用遍历访问到的对象进行标记为“被引用”。

清除阶段：对没有标记过的内存进行回收（回收的同时可能伴有随便整理操作）。

弥补了引用计数的不足(频繁更新引用计数降低了性能+循环引用)，缺点是需要暂停程序STW（每次启动垃圾回收都会暂停当前所有的代码执行，回收使系统响应能力大大减低）。

**2.三色标记法**：

- 初始状态下所有对象都是白色的（首先需要STW，做一些准备工作，比如开启写屏障）。
- 从根节点（包括**全局指针和 goroutine 栈上**的指针）开始遍历所有对象，把遍历到的对象变成灰色对象
- 遍历灰色对象，将灰色对象引用的对象也变成灰色对象，然后将遍历过的灰色对象变成黑色对象。
- 循环步骤3，直到灰色对象全部变黑色。
- 通过写屏障(write-barrier)检测对象有变化，重复以上操作
- 收集所有白色对象（垃圾）。

这种方法有一个缺陷，如果对象的引用被用户修改了，那么之前的标记就无效了。因此Go采用了**写屏障技术**，当对象新增或者更新会将其着色为灰色。

**3.STW （Stop The World）**

- 为了避免在 GC 的过程中，对象之间的引用关系发生新的变更，使得GC的结果发生错误（如GC过程中新增了一个引用，但是由于未扫描到该引用导致将被引用的对象清除了），停止所有正在运行的协程。
- STW对性能有一些影响，Golang目前已经可以做到1ms以下的STW。

**4.写屏障(Write Barrier)**

- 为了避免GC的过程中新修改的引用关系到GC的结果发生错误，我们需要进行STW。但是STW会影响程序的性能，所以我们要**通过写屏障技术尽可能地缩短STW的时间**。

基于插入写屏障和删除写屏障在结束时需要STW来重新扫描栈，带来性能瓶颈。

> 造成引用对象丢失的条件:
>
> 一个黑色的节点A新增了指向白色节点C的引用，并且白色节点C没有除了A之外的其他灰色节点的引用，或者存在但是在GC过程中被删除了。
>
> 以上两个条件需要同时满足：满足条件1时说明节点A已扫描完毕，A指向C的引用无法再被扫描到；满足条件2时说明白色节点C无其他灰色节点的引用了，即扫描结束后会被忽略 。

**混合写屏障**分为以下四步：

1. GC开始时，将栈上的全部对象标记为黑色（不需要二次扫描，无需STW）；
2. GC期间，任何栈上创建的新对象均为黑色
3. 被删除引用的对象标记为灰色
4. 被添加引用的对象标记为灰色

总而言之就是确保黑色对象不能引用白色对象，这个改进直接使得GC时间从 2s降低到2us。

Golang gc 优化的核心就是尽量使得 STW(Stop The World) 的时间越来越短。

### 02 ❤如何知道一个对象是分配在栈上还是堆上？

Go和C++不同，Go局部变量会进行**逃逸分析**。如果**变量离开作用域后没有被引用**，则**优先**分配到栈上，否则分配到堆上。那么如何判断是否发生了逃逸呢？

`go build -gcflags '-m -m -l' xxx.go`.

### 03❤golang的内存管理的原理清楚吗？简述go内存管理机制

golang内存管理基本是参考tcmalloc来进行的。go内存管理本质上是一个内存池，只不过内部做了很多优化：**自动伸缩内存池大小，合理的切割内存块**。

> 一些基本概念：
> 页Page：一块8K大小的内存空间。Go向操作系统申请和释放内存都是以页为单位的。
> span : 内存块，一个或多个连续的 page 组成一个 span 。如果把 page 比喻成工人， span 可看成是小队，工人被分成若干个队伍，不同的队伍干不同的活。
> sizeclass : 空间规格，每个 span 都带有一个 sizeclass ，标记着该 span 中的 page 应该如何使用。使用上面的比喻，就是 sizeclass 标志着 span 是一个什么样的队伍。
> object : 对象，用来存储一个变量数据内存空间，一个 span 在初始化时，会被切割成一堆等大的 object 。假设 object 的大小是 16B ， span 大小是 8K ，那么就会把 span 中的 page 就会被初始化 8K / 16B = 512 个 object 。所谓内存分配，就是分配一个 object 出去。

**内存池mheap**

mheap 将从 OS 那里申请过来的内存初始化成一个大 `span`(sizeclass=0)。然后根据需要从这个大 `span` 中切出小 `span`，放在 mcentral 中来管理。大 `span` 由 `mheap.freelarge` 和 `mheap.busylarge` 等管理。如果 mcentral 中的 `span` 不够用了，会从 `mheap.freelarge` 上再切一块，如果 `mheap.freelarge` 空间不够，会再次从 OS 那里申请内存重复上述步骤。

![img](https://upload-images.jianshu.io/upload_images/11662994-8361f3be115cf456.png?imageMogr2/auto-orient/strip|imageView2/2/w/404/format/webp)

mheap.spans ：用来存储 page 和 span 信息，比如一个 span 的起始地址是多少，有几个 page，已使用了多大等等。

mheap.bitmap 存储着各个 span 中对象的标记信息，比如对象是否可回收等等。

mheap.arena_start : 将要分配给应用程序使用的空间。

**mcentral**

**mcentral是一个span链，用途相同的span会以链表的形式组织在一起存放在mcentral中**。这里用途用**sizeclass**来表示，就是该span存储哪种大小的对象。比如当分配一块大小为 `n` 的内存时，系统计算 `n` 应该使用哪种 `sizeclass`，然后根据 `sizeclass` 的值去找到一个可用的 `span` 来用作分配

找到合适的 span 后，会从中取一个 object 返回给上层使用。



![img](https://upload-images.jianshu.io/upload_images/11662994-730fc9b0a604aea1.png?imageMogr2/auto-orient/strip|imageView2/2/w/551/format/webp)

**mcache**

> 从 mcache 上分配内存空间是不需要加锁的，因为在同一时间里，一个 P 只有一个线程在其上面运行，不可能出现竞争。没有了锁的限制，大大加速了内存分配。

为了提高内存并发申请效率，加入缓存层mcache。每一个mcache和处理器P对应。Go申请内存首先从P的mcache中分配，

![img](https://upload-images.jianshu.io/upload_images/11662994-e6d7200368ec06b6.png?imageMogr2/auto-orient/strip|imageView2/2/w/696/format/webp)

答：

go语言内存管理本质上是一个内存池，只不过做了很多优化：自动伸缩内存池大小，合理的切割内存块等。go语言分配内存首先从处理器P的缓存层mcache中申请。mcache上分配内存空间不需要加锁，因为每一个mcache和一个处理器P对应，而一个P只有一个线程在其上面运行，不可能出现竞争，大大加速了内存分配，提高了内存并发申请效率。

如果mcache缓存中没有资源，则会向内存池mheap申请内存。mheap会将从操作系统申请的内存初始化成一个大span，大span由mheap.freelarge和mheap.busylarge管理，然后根据需要将大span按照sizeclass切割成小span放在mcentral中管理。mcentral是一条span链表，用途相同的span会以链表的形式组织在一起存放在mcentral中。go语言向mheap申请内存时首先会从mcentral中申请，系统计算所需内存应该使用哪种sizeclass，再根据sizeclass找的一个可用的span分配，从中取出一个object返回。如果mcentral中的span不够用了，会从mheap.freelarge上再切一块，如果mheap.freelarge空间不够，会再次从操作系统中申请内存进行分配。

> 参考资料：[Go 语言内存管理（二）：Go 内存管理](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/developer/article/1422392)

### 04goroutine什么情况会发生内存泄漏？如何定位排查内存泄露问题

在Go中内存泄露分为暂时性内存泄露和永久性内存泄露。

**暂时性内存泄露**

- 获取长字符串中的一段导致长字符串未释放
- 获取长slice中的一段导致长slice未释放
- 在长slice新建slice导致泄漏

string相比切片少了一个容量的cap字段，可以把string当成一个只读的切片类型。获取长string或者切片中的一段内容，由于新生成的对象和老的string或者切片共用一个内存空间，会导致老的string和切片资源暂时得不到释放，造成短暂的内存泄漏

**永久性内存泄露**

- goroutine永久阻塞而导致泄漏，互斥锁未释放或者造成死锁会造成内存泄漏
- time.Ticker未关闭导致泄漏
- 不正确使用Finalizer（Go版本的析构函数）导致泄漏

**排查方式：**

一般通过 pprof 是 Go 的性能分析工具，在程序运行过程中，可以记录程序的运行信息，可以是 CPU 使用情况、内存使用情况、goroutine 运行情况等，当需要性能调优或者定位 Bug 时候，这些记录的信息是相当重要。

### 05GC 中 stw 时机，各个阶段是如何解决的？ （百度）

> 1）在开始新的一轮 GC 周期前，需要调用 gcWaitOnMark 方法上一轮 GC 的标记结束（含扫描终止、标记、或标记终止等）。
>
> 2）开始新的一轮 GC 周期，调用 gcStart 方法触发 GC 行为，开始扫描标记阶段。
>
> 3）需要调用 gcWaitOnMark 方法等待，直到当前 GC 周期的扫描、标记、标记终止完成。
>
> 4）需要调用 sweepone 方法，扫描未扫除的堆跨度，并持续扫除，保证清理完成。在等待扫除完毕前的阻塞时间，会调用 Gosched 让出。
>
> 5）在本轮 GC 已经基本完成后，会调用 mProf_PostSweep 方法。以此记录最后一次标记终止时的堆配置文件快照。
>
> 6）结束，释放 M。

- GC开始时进行STW，做一些准备工作，比如开启写屏障，扫描栈等
- 二次扫描：GC 迭代结束时（没有灰色节点），会对栈执行 STW，重新进行扫描清除白色节点。（STW 时间为 10-100ms）。如果开启混合写屏障，无需进行二次扫描

### 06❤GC 的触发时机？（初级必问）

初级必问，分为系统触发和手动触发。

1）gcTriggerHeap：当所分配的堆大小达到阈值（由控制器计算的触发堆的大小）时，将会触发。

2）gcTriggerTime：如果一定时间内没有触发，就会触发新的GC。时间周期以runtime.forcegcperiod 变量为准，默认 2 分钟。

3）gcTriggerCycle：如果没有开启 GC，则启动 GC。

4）手动触发的 runtime.GC 方法。

### 07❤知道 golang 的内存逃逸吗？什么情况下会发生内存逃逸？（必问）

答：

1)**本该分配到栈上的变量，跑到了堆上，这就导致了内存逃逸。**2)栈是高地址到低地址，栈上的变量，函数结束后变量会跟着回收掉，不会有额外性能的开销。3)变量从栈逃逸到堆上，如果要回收掉，需要进行 gc，那么 gc 一定会带来额外的性能开销。编程语言不断优化 gc 算法，主要目的都是为了减少 gc 带来的额外性能开销，变量一旦逃逸会导致性能开销变大。

**内存逃逸的情况如下：**

1）方法内返回局部变量指针，且在方法外被引用。

2）向 channel 发送指针数据，在 slice 或 map 中存储指针。

3）在闭包中引用包外的值。

4）变量大小和类型不确定，变量分配的内存超过用户栈最大值

### 08Channel 分配在栈上还是堆上？哪些对象分配在堆上，哪些对象分配在栈上？

Channel 被设计用来实现协程间通信的组件，其作用域和生命周期不可能仅限于某个函数内部，所以 golang 直接将其分配在**堆**上

准确地说，你并不需要知道。Golang 中的变量只要被引用就一直会存活，存储在堆上还是栈上由内部实现决定而和具体的语法没有关系。

知道变量的存储位置确实和效率编程有关系。如果可能，Golang 编译器会将函数的局部变量分配到函数栈帧（stack frame）上。然而，如果编译器不能确保变量在函数 return 之后不再被引用，编译器就会将变量分配到堆上。而且，如果一个局部变量非常大，那么它也应该被分配到堆上而不是栈上。

当前情况下，如果一个变量被取地址，那么它就有可能被分配到堆上,然而，还要对这些变量做逃逸分析，如果函数 return 之后，变量不再被引用，则将其分配到栈上。

### 10介绍一下大对象小对象，为什么小对象多了会造成 gc 压力？

小于等于 32k 的对象就是小对象，其它都是大对象。一般小对象通过 mspan 分配内存；大对象则直接由 mheap 分配内存。通常小对象过多会导致 GC 三色法消耗过多的 CPU。优化思路是，减少对象分配。

小对象：如果申请小对象时，发现当前内存空间不存在空闲跨度时，将会需要调用 nextFree 方法获取新的可用的对象，可能会触发 GC 行为。

大对象：如果申请大于 32k 以上的大对象时，可能会触发 GC 行为。

## 协程调度GMP模型

### 01❤go如何进行调度的。GMP中状态流转。

> 我们知道，在高并发应用中频繁创建线程会造成不必要的开销，所以有了线程池。线程池中预先保存一定数量的线程，而新任务将不再以创建线程的方式去执行，而是将任务发布到任务队列，线程池中的线程不断地从任务队列中取出任务并执行，可以有效的减少线程创建和销毁所带来的开销。
>
> 下图展示一个典型的线程池：
>
> ![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_b499f154d0135854b725ce27a6b7a009_r.png)
>
> 为了方便下面的叙述，我们把任务队列中的每一个任务称作G，而G往往代表一个函数。
> 线程池中的worker线程不断地从任务队列中取出任务并执行。而worker线程的调度则交给操作系统进行调度。
>
> 如果worker线程执行的G任务中发生系统调用，则操作系统会将该线程置为阻塞状态，也意味着该线程在怠工，也意味着消费任务队列的worker线程变少了，也就是说线程池消费任务队列的能力变弱了。
>
> 如果任务队列中的大部分任务都会进行系统调用，则会让这种状态恶化，大部分worker线程进入阻塞状态，从而任务队列中的任务产生堆积。
>
> 解决这个问题的一个思路就是重新审视线程池中线程的数量，增加线程池中线程数量可以一定程度上提高消费能力，但随着线程数量增多，由于**过多线程争抢CPU**，消费能力会有上限，甚至出现消费能力下降。 如下图所示：
>
> ![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_fa5c2be587f99cf1120b126f0c563055_r.png)
>
> 

- G（Goroutine）：即Go协程，每个go关键字都会创建一个协程。
- M（Machine）：工作线程，在Go中称为Machine，数量对应真实的CPU数（真正干活的对象），M 的数量是不定的，由 Go Runtime 调整，为了防止创建过多 OS 线程导致系统调度不过来，目前默认最大限制为 10000 个。
- P（Processor）：处理器（Go中定义的一个摡念，非CPU），包含运行Go代码的必要资源，用来调度 G 和 M 之间的关联关系，其数量可通过 GOMAXPROCS() 来设置，默认为核心数。但是不论 GOMAXPROCS 设置为多大，P 的数量最大为 256。

M必须拥有P才可以执行G中的代码，P含有一个包含多个G的队列，P可以调度G交由M执行。

![img](https://www.topgoer.cn/uploads/gozhuanjia/images/m_274ee3af62bab4ad8f74a6753d6969cf_r.png)

调度器是M和G之间桥梁。

**go进行调度过程：**

- 某个线程尝试创建一个新的G，那么这个G就会被安排到这个线程的G本地队列LRQ中，如果LRQ满了，就会分配到全局队列GRQ中；


- 队列轮转：P 会周期性的将G调度到M中执行，执行一段时间后，保存上下文，将G放到队列尾部，然后从队列中再取出一个G进行调度。除此之外，P还会周期性的查看全局队列是否有G等待调度到M中执行。
- 系统调用：当G0即将进入系统调用时，M0将释放P，进而某个空闲的M1获取P，继续执行P队列中剩下的G。M1的来源有可能是M的缓存池，也可能是新建的。当G0系统调用结束后，如果有空闲的P，则获取一个P，继续执行G0。如果没有，则将G0放入全局队列，等待被其他的P调度。然后M0将进入缓存池睡眠。

### 02Go什么时候发生阻塞？阻塞时，调度器会怎么做。

- 用于**原子、互斥量或通道**操作导致goroutine阻塞，调度器将把当前阻塞的goroutine从本地运行队列**LRQ换出**，并重新调度其它goroutine；
- 由于**网络请求**和**IO**导致的阻塞，Go提供了网络轮询器（Netpoller）来处理，后台用epoll等技术实现IO多路复用。

其它回答：

- **channel阻塞**：当goroutine读写channel发生阻塞时，会调用gopark函数，该G脱离当前的M和P，调度器将新的G放入当前M。
- **系统调用**：当某个G由于系统调用陷入内核态，该P就会脱离当前M，此时P会更新自己的状态为Psyscall，M与G相互绑定，进行系统调用。结束以后，若该P状态还是Psyscall，则直接关联该M和G，否则使用闲置的处理器处理该G。
- **系统监控**：当某个G在P上运行的时间超过10ms时候，或者P处于Psyscall状态过长等情况就会调用retake函数，触发新的调度。
- **主动让出**：由于是协作式调度，该G会主动让出当前的P（通过GoSched），更新状态为Grunnable，该P会调度队列中的G运行。

> 更多关于netpoller的内容可以参看：[https://strikefreedom.top/go-netpoll-io-multiplexing-reactor](https://link.zhihu.com/?target=https%3A//strikefreedom.top/go-netpoll-io-multiplexing-reactor)



### 03Go中GMP有哪些状态？

![img](https://pic4.zhimg.com/80/v2-87beb4a53dd92ddccef4ecb486dfa213_1440w.webp)

G的状态：

**_Gidle**：刚刚被分配并且还没有被初始化，值为0，为创建goroutine后的默认值

**_Grunnable**： 没有执行代码，没有栈的所有权，存储在运行队列中，可能在某个P的**本地队列或全局队列**中(如上图)。

**_Grunning**： 正在执行代码的goroutine，拥有栈的所有权(如上图)。

**_Gsyscall**：正在执行系统调用，拥有栈的所有权，与P脱离，但是与某个M绑定，会在调用结束后被分配到运行队列(如上图)。

**_Gwaiting**：被阻塞的goroutine，阻塞在某个channel的发送或者接收队列(如上图)。

**_Gdead**： 当前goroutine未被使用，没有执行代码，可能有分配的栈，分布在空闲列表gFree，可能是一个刚刚初始化的goroutine，也可能是执行了goexit退出的goroutine(如上图)。

**_Gcopystac**：栈正在被拷贝，没有执行代码，不在运行队列上，执行权在

**_Gscan** ： GC 正在扫描栈空间，没有执行代码，可以与其他状态同时存在。

P的状态：

**_Pidle** ：处理器没有运行用户代码或者调度器，被空闲队列或者改变其状态的结构持有，运行队列为空

**_Prunning** ：被线程 M 持有，并且正在执行用户代码或者调度器(如上图)

**_Psyscall**：没有执行用户代码，当前线程陷入系统调用(如上图)

**_Pgcstop** ：被线程 M 持有，当前处理器由于垃圾回收被停止

**_Pdead** ：当前处理器已经不被使用

M的状态：

**自旋线程**：处于运行状态但是没有可执行goroutine的线程，数量最多为GOMAXPROC，若是数量大于GOMAXPROC就会进入休眠。

**非自旋线程**：处于运行状态有可执行goroutine的线程。

### 04GMP能不能去掉P层？会怎么样？

P层的作用

- 每个 P 有自己的本地队列，大幅度的减轻了对全局队列的直接依赖，所带来的效果就是锁竞争的减少。而 GM 模型的性能开销大头就是锁竞争。
- 每个 P 相对的平衡上，在 GMP 模型中也实现了 Work Stealing 算法，如果 P 的本地队列为空，则会从全局队列或其他 P 的本地队列中窃取可运行的 G 来运行，减少空转，提高了资源利用率。

参考资料：[https://juejin.cn/post/6968311281220583454](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6968311281220583454)

### 05如果有一个G一直占用资源怎么办？什么是work stealing算法？

如果有个goroutine一直占用资源，那么GMP模型会**从正常模式转变为饥饿模式**（类似于mutex），允许其它goroutine使用work stealing抢占（禁用自旋锁）。

work stealing算法指，如果一个调度器P处于空闲状态，则会尝试从全局队列或其他 P 的本地队列中窃取可运行的 G 来运行，可以极大提高执行效率。

### **06go语言抢占式调度是如何抢占的？**

> 协作式调度依靠被调度方主动弃权；
>
> 抢占式调度则依靠调度器强制将被调度方被动中断。
>
> 参考
>
> https://blog.51cto.com/u_15107299/3935086
>
> https://go-interview.iswbm.com/c02/c02_05.html

**基于协作的抢占式调度**

1. 如果 sysmon 监控线程发现有个协程 A 执行时间太长了（或者 gc 场景，或者 stw 场景），那么会友好的在这个 A 协程的某个字段设置一个抢占标记 ；
2. 协程 A 在 call 一个函数的时候，会复用到扩容栈（morestack）的部分逻辑，检查到抢占标记之后，让出 cpu，切到调度主协程里；

但是这种调度并不完备，比如一个goroutine运行了很久，但是它并没有调用另一个函数，则它不会被抢占

**基于信号量抢占调度**

- M 注册一个 **SIGURG** 信号的处理函数：sighandler。


- sysmon 线程检测到执行时间过长的 goroutine 或者GC stw 时，会向相应的 M（或者说线程，每个线程对应一个 M）发送 SIGURG 信号。


- 收到信号后，内核执行 sighandler 函数，通过 **pushCall** 插入 asyncPreempt 函数调用。


- 回到当前 goroutine 执行 asyncPreempt 函数，通过 mcall 切到 g0 栈执行 gopreempt_m。


- 将当前 goroutine 插入到全局可运行队列，M 则继续寻找其他 goroutine 来运行。


- 被抢占的 goroutine 再次调度过来执行时，会继续原来的执行流。



## 常见数据结构的底层原理

### 01说说 atomic底层怎么实现的.

> 原子操作即是进行过程中不能被中断的操作，针对某个值的原子操作在被进行的过程中，CPU绝不会再去进行其他的针对该值的操作。为了实现这样的严谨性，原子操作仅会由一个**独立的CPU指令**代表和完成。原子操作是无锁的，常常直接通过CPU指令直接实现。 事实上，其它同步技术的实现常常依赖于原子操作。
>
> 具体的原子操作在不同的操作系统中实现是不同的。比如在Intel的CPU架构机器上，主要是使用总线锁的方式实现的。 大致的意思就是当一个CPU需要操作一个内存块的时候，向总线发送一个LOCK信号，所有CPU收到这个信号后就不对这个内存块进行操作了。 等待操作的CPU执行完操作后，发送UNLOCK信号，才结束。
> 在AMD的CPU架构机器上就是使用MESI一致性协议的方式来保证原子操作。 所以我们在看atomic源码的时候，我们看到它针对不同的操作系统有不同汇编语言文件。

atomic源码位于`sync\atomic`。通过阅读源码可知，atomic采用**CAS**（CompareAndSwap）的方式实现的。所谓CAS就是使用了CPU中的原子性操作(**独立的CPU指令**)。在操作共享变量的时候，CAS不需要对其进行加锁，而是通过类似于**乐观锁**的方式进行检测，总是假设被操作的值未曾改变（即与旧值相等），并一旦确认这个假设的真实性就立即进行值替换。

本质上是**不断占用CPU资源来避免加锁的开销**。

**原子操作与互斥锁的区别**

1）、互斥锁是一种数据结构，用来让一个线程执行程序的关键部分，完成互斥的多个操作。

2）、原子操作是针对某个值的单个互斥操作。

> 参考资料：[Go语言的原子操作atomic - 编程猎人](https://link.zhihu.com/?target=https%3A//www.programminghunter.com/article/37392193442/)



### 02channel底层实现？是否线程安全。

channel底层实现在`src/runtime/chan.go`中

```go
type hchan struct {
    qcount   uint           // 当前队列中剩余元素个数
    dataqsiz uint           // 环形队列长度，即可以存放的元素个数
    buf      unsafe.Pointer // 环形队列指针,指向队列的内存
    elemsize uint16         // 每个元素的大小
    closed   uint32            // 标识关闭状态
    elemtype *_type         // 元素类型
    sendx    uint           // 队列下标，指示元素写入时存放到队列中的位置
    recvx    uint           // 队列下标，指示元素从队列的该位置读出
    recvq    waitq          // 等待读消息的goroutine队列
    sendq    waitq          // 等待写消息的goroutine队列
    lock mutex              // 互斥锁，chan不允许并发读写
}
```

channel内部是一个环形队列。内部包含buf, sendx, recvx, lock ,recvq, sendq几个部分；

buf是有缓冲的channel所特有的结构，用来存储缓存数据，是个环形队列（循环链表）； 

![img](https://www.topgoer.cn/uploads/gozhuanjia/images/m_f1b42d200c5d94d02eeacef7c99aa81b_r.png)

- sendx和recvx用于记录buf这个循环链表中的写入或者读取的index；
- lock是个互斥锁；
- recvq和sendq分别是接收(<-channel)或者发送(channel <- xxx)的goroutine抽象出来的结构体(sudog)的等待队列。被阻塞的goroutine将会挂在channel的等待队列中：
  - 因读阻塞的goroutine会被向channel写入数据的goroutine唤醒；
  - 因写阻塞的goroutine会被从channel读数据的goroutine唤醒；

channel是**线程安全**的。

简要回答：channel 的数据结构包含 qccount 当前队列中剩余元素个数，dataqsiz 环形队列长度，即可以存放的元素个数，buf 环形队列指针，elemsize 每个元素的大小，closed 标识关闭状态，elemtype 元素类型，sendx 队列下表，指示元素写入时存放到队列中的位置，recv 队列下表，指示元素从队列的该位置读出。recvq 等待读消息的 goroutine 队列，sendq 等待写消息的 goroutine 队列，lock 互斥锁，chan 不允许并发读写。

**无缓冲和有缓冲区别：** 管道没有缓冲区，从管道读数据会阻塞，直到有协程向管道中写入数据。同样，向管道写入数据也会阻塞，直到有协程从管道读取数据。管道有缓冲区但缓冲区没有数据，从管道读取数据也会阻塞，直到协程写入数据，如果管道满了，写数据也会阻塞，直到协程从缓冲区读取数据。

**channel 的一些特点** 1）、读写值 nil 管道会永久阻塞 2）、关闭的管道读数据仍然可以读数据 3）、往关闭的管道写数据会 panic 4）、关闭为 nil 的管道 panic 5）、关闭已经关闭的管道 panic

**向 channel 写数据的流程：** 

- 如果等待接收队列 recvq 不为空，说明缓冲区中没有数据或者没有缓冲区，此时直接从 recvq 取出 G,并把数据写入，最后把该 G 唤醒，结束发送过程； 

- 如果缓冲区中有空余位置，将数据写入缓冲区，结束发送过程； 

- 如果缓冲区中没有空余位置，将待发送数据写入 G，将当前 G 加入 sendq，进入睡眠，等待被读 goroutine 唤醒；

  ![img](https://www.topgoer.cn/uploads/gozhuanjia/images/m_b235ef1f2c6ac1b5d63ec5660da97bd2_r.png)

**向 channel 读数据的流程：** 

- 如果等待发送队列 sendq 不为空，且没有缓冲区，直接从 sendq 中取出 G，把 G 中数据读出，最后把 G 唤醒，结束读取过程； 
- 如果等待发送队列 sendq 不为空，此时说明缓冲区已满，从缓冲区中首部读出数据，把 G 中数据写入缓冲区尾部，把 G 唤醒，结束读取过程； 
- 如果缓冲区中有数据，则从缓冲区取出数据，结束读取过程；否则将当前 goroutine 加入 recvq，进入睡眠，等待被写 goroutine 唤醒；

![img](https://www.topgoer.cn/uploads/gozhuanjia/images/m_933ca9af4c3ec1db0b94b8b4ec208d4b_r.png)

**使用场景：** **消息传递**、消息过滤，信号广播，**事件订阅与广播**，请求、响应转发，任务分发，结果汇总，**并发控制**，**限流**，**同步与异步**

> 参考资料：[Kitou：Golang 深度剖析 -- channel的底层实现](https://zhuanlan.zhihu.com/p/264305133)

### 03map的底层实现。怎样实现扩容的？

源码位于`src\runtime\map.go` 中。

go的map和C++map不一样，底层实现是哈希表，包括两个部分：**hmap**和**bucket哈希桶**。

```go
type hmap struct {
    count     int // 当前保存的元素个数
    ...
    B         uint8
    ...
    buckets    unsafe.Pointer // bucket数组指针，数组的大小为2^B
    ...
}
//bucket数据结构由runtime/map.go:bmap定义：
type bmap struct {
    tophash [8]uint8 //存储哈希值的高8位
    data    byte[1]  //key value数据:key/key/key/.../value/value/value...
    overflow *bmap   //溢出bucket的地址

}
```

每个bucket可以存储8个键值对。

- tophash是个长度为8的数组，哈希值相同的键（准确的说是哈希值低位相同的键）存入当前bucket时会将哈希值的高位存储在该数组中，以方便后续匹配。
- data区存放的是key-value数据，存放顺序是key/key/key/…value/value/value，如此存放是为了节省字节对齐带来的空间浪费。
- overflow 指针指向的是下一个bucket，据此将所有冲突的键连接起来。

注意：上述中data和overflow并不是在结构体中显示定义的，而是直接通过指针运算进行访问的。

当有两个或以上数量的键被哈希到了同一个bucket时，我们称这些键发生了冲突。Go使用链地址法来解决键冲突。
由于每个bucket可以存放8个键值对，所以同一个bucket存放超过8个键值对时就会再创建一个键值对，用类似链表的方式将bucket连接起来。

![img](https://www.topgoer.cn/uploads/gozhuanjia/images/m_a8b9e5919d9951a71c1c36445dd68521_r.png)

**查找过程**如下：

1. 根据key值算出哈希值
2. 取哈希值低位与hmap.B取模确定bucket位置
3. 取哈希值高位在tophash数组中查询
4. 如果tophash[i]中存储值也哈希值相等，则去找到该bucket中的key值进行比较
5. 当前bucket没有找到，则继续从下个overflow的bucket中查找。
6. 如果当前处于搬迁过程，则优先从oldbuckets查找

注：如果查找不到，也不会返回空值，而是返回相应类型的0值。

**map 扩容**

> 询问map的底层实现的时候，扩容可以少讲或者省略，除非面试官问到扩容
>
> **触发条件**
>
> 1）负载因子超过阈值，源码里定义的阈值是 6.5。
>
> 负载因子用于衡量一个哈希表冲突情况，公式为：
>
> ```
> 负载因子 = 键数量/bucket数量
>
> ```
>
> 2）overflow数量 > 2^15时，也即overflow数量超过32768时。

**增量扩容**

当负载因子过大时，就新建一个bucket，新的bucket长度是原来的2倍，然后旧bucket数据搬迁到新的bucket。
考虑到如果map存储了数以亿计的key-value，一次性搬迁将会造成比较大的延时，Go采用逐步搬迁策略，即每次访问map时都会触发一次搬迁，每次搬迁2个键值对。

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_2f0122f26e5d66ca91e6820ace6b379b_r.png)

hmap数据结构中oldbuckets成员指身原bucket，而buckets指向了新申请的bucket。新的键值对被插入新的bucket中。

**等量扩容**

所谓等量扩容，实际上并不是扩大容量，**buckets数量不变**，重新做一遍类似增量扩容的搬迁动作，把松散的键值对重新排列一次，以使bucket的使用率更高，进而保证更快的存取。
在极端场景下，比如不断地增删，而键值对正好集中在一小部分的bucket，这样会造成overflow的bucket数量增多，但负载因子又不高，从而无法执行增量搬迁的情况，如下图所示：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_f3a5989c90204df9304d5ae246f3db72_r.png)

上图可见，overflow的bucket中大部分是空的，访问效率会很差。此时进行一次等量扩容，即buckets数量不变，经过重新组织后overflow的bucket数量会减少，即节省了空间又会提高访问效率。

### 04select的实现原理？讲讲select的一些特性

select源码位于`src\runtime\select.go`，最重要的`scase` 数据结构为：

```go
type scase struct {
	c    *hchan         // chan
	elem unsafe.Pointer // data element
}
```

scase.c为当前case语句所操作的channel指针，这也说明了一个case语句只能操作一个channel。

scase.elem表示缓冲区地址，根据scase.kind不同，有不同的用途：

- scase.kind == caseRecv ： scase.elem表示读出channel的数据存放地址；
- scase.kind == caseSend ： scase.elem表示将要写入channel的数据存放地址；

**select实现逻辑**

select的主要实现位于：`select.go`函数：

```go
func selectgo(cas0 *scase, order0 *uint16, ncases int) (int, bool)
{ 
  	//1. 锁定scase语句中所有的channel
    //2. 按照随机顺序检测scase中的channel是否ready
    //   2.1 如果case可读，则读取channel中数据，解锁所有的channel，然后返回(case index, true)
    //   2.2 如果case可写，则将数据写入channel，解锁所有的channel，然后返回(case index, false)
    //   2.3 所有case都未ready，则解锁所有的channel，然后返回（default index, false）
    //3. 所有case都未ready，且没有default语句
    //   3.1 将当前协程加入到所有channel的等待队列
    //   3.2 当将协程转入阻塞，等待被唤醒
    //4. 唤醒后返回channel对应的case index
    //   4.1 如果是读操作，解锁所有的channel，然后返回(case index, true)
    //   4.2 如果是写操作，解锁所有的channel，然后返回(case index, false)
}
```

函数参数：

- cas0为scase数组的首地址，selectgo()就是从这些scase中找出一个返回。
- order0为一个两倍cas0数组长度的buffer，保存scase随机序列pollorder和scase中channel地址序列lockorder
  - pollorder：每次selectgo执行都会把scase序列打乱，以达到随机检测case的目的。
  - lockorder：所有case语句中channel序列，以达到去重防止对channel加锁时重复加锁的目的。
- ncases表示scase数组的长度

函数返回值：

1. int： 选中case的编号，这个case编号跟代码一致
2. bool: 是否成功从channle中读取了数据，如果选中的case是从channel中读数据，则该返回值表示是否读取成功。

[^特别说明]: 对于读channel的case来说，如`case elem, ok := <-chan1:`, 如果channel有可能被其他协程关闭的情况下，一定要检测读取是否成功，因为close的channel也有可能返回，此时ok == false。

**select的特性**

1）select 操作至少要有一个 case 语句，出现读写 nil 的 channel 该分支会忽略，在 nil 的 channel 上操作则会报错。

2）select 仅支持管道，而且是单协程操作。

3）每个 case 语句仅能处理一个管道，要么读要么写。

4）多个 case 语句的执行顺序是随机的。

5）存在 default 语句，select 将不会阻塞，但是存在 default 会影响性能。

参考资料：[Go select的使用和实现原理](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/wuyepeng/p/13910678.html%23%3A~%3Atext%3D%25E4%25B8%2580%25E3%2580%2581select%25E7%25AE%2580%25E4%25BB%258B.%25201.Go%25E7%259A%2584select%25E8%25AF%25AD%25E5%258F%25A5%25E6%2598%25AF%25E4%25B8%2580%25E7%25A7%258D%25E4%25BB%2585%25E8%2583%25BD%25E7%2594%25A8%25E4%25BA%258Echannl%25E5%258F%2591%25E9%2580%2581%25E5%2592%258C%25E6%258E%25A5%25E6%2594%25B6%25E6%25B6%2588%25E6%2581%25AF%25E7%259A%2584%25E4%25B8%2593%25E7%2594%25A8%25E8%25AF%25AD%25E5%258F%25A5%25EF%25BC%258C%25E6%25AD%25A4%25E8%25AF%25AD%25E5%258F%25A5%25E8%25BF%2590%25E8%25A1%258C%25E6%259C%259F%25E9%2597%25B4%25E6%2598%25AF%25E9%2598%25BB%25E5%25A1%259E%25E7%259A%2584%25EF%25BC%259B%25E5%25BD%2593select%25E4%25B8%25AD%25E6%25B2%25A1%25E6%259C%2589case%25E8%25AF%25AD%25E5%258F%25A5%25E7%259A%2584%25E6%2597%25B6%25E5%2580%2599%25EF%25BC%258C%25E4%25BC%259A%25E9%2598%25BB%25E5%25A1%259E%25E5%25BD%2593%25E5%2589%258Dgroutine%25E3%2580%2582.%25202.select%25E6%2598%25AFGolang%25E5%259C%25A8%25E8%25AF%25AD%25E8%25A8%2580%25E5%25B1%2582%25E9%259D%25A2%25E6%258F%2590%25E4%25BE%259B%25E7%259A%2584I%252FO%25E5%25A4%259A%25E8%25B7%25AF%25E5%25A4%258D%25E7%2594%25A8%25E7%259A%2584%25E6%259C%25BA%25E5%2588%25B6%25EF%25BC%258C%25E5%2585%25B6%25E4%25B8%2593%25E9%2597%25A8%25E7%2594%25A8%25E6%259D%25A5%25E6%25A3%2580%25E6%25B5%258B%25E5%25A4%259A%25E4%25B8%25AAchannel%25E6%2598%25AF%25E5%2590%25A6%25E5%2587%2586%25E5%25A4%2587%25E5%25AE%258C%25E6%25AF%2595%25EF%25BC%259A%25E5%258F%25AF%25E8%25AF%25BB%25E6%2588%2596%25E5%258F%25AF%25E5%2586%2599%25E3%2580%2582.%2C3.select%25E8%25AF%25AD%25E5%258F%25A5%25E4%25B8%25AD%25E9%2599%25A4default%25E5%25A4%2596%25EF%25BC%258C%25E6%25AF%258F%25E4%25B8%25AAcase%25E6%2593%258D%25E4%25BD%259C%25E4%25B8%2580%25E4%25B8%25AAchannel%25EF%25BC%258C%25E8%25A6%2581%25E4%25B9%2588%25E8%25AF%25BB%25E8%25A6%2581%25E4%25B9%2588%25E5%2586%2599.%25204.select%25E8%25AF%25AD%25E5%258F%25A5%25E4%25B8%25AD%25E9%2599%25A4default%25E5%25A4%2596%25EF%25BC%258C%25E5%2590%2584case%25E6%2589%25A7%25E8%25A1%258C%25E9%25A1%25BA%25E5%25BA%258F%25E6%2598%25AF%25E9%259A%258F%25E6%259C%25BA%25E7%259A%2584.%25205.select%25E8%25AF%25AD%25E5%258F%25A5%25E4%25B8%25AD%25E5%25A6%2582%25E6%259E%259C%25E6%25B2%25A1%25E6%259C%2589default%25E8%25AF%25AD%25E5%258F%25A5%25EF%25BC%258C%25E5%2588%2599%25E4%25BC%259A%25E9%2598%25BB%25E5%25A1%259E%25E7%25AD%2589%25E5%25BE%2585%25E4%25BB%25BB%25E4%25B8%2580case.%25206.select%25E8%25AF%25AD%25E5%258F%25A5%25E4%25B8%25AD%25E8%25AF%25BB%25E6%2593%258D%25E4%25BD%259C%25E8%25A6%2581%25E5%2588%25A4%25E6%2596%25AD%25E6%2598%25AF%25E5%2590%25A6%25E6%2588%2590%25E5%258A%259F%25E8%25AF%25BB%25E5%258F%2596%25EF%25BC%258C%25E5%2585%25B3%25E9%2597%25AD%25E7%259A%2584channel%25E4%25B9%259F%25E5%258F%25AF%25E4%25BB%25A5%25E8%25AF%25BB%25E5%258F%2596).

### 05go的interface怎么实现的？

go interface源码在`runtime\iface.go`中。

go的接口由两种类型实现`iface`和`eface`。iface是包含方法的接口，而eface不包含方法。

- `iface`

对应的数据结构是（位于`src\runtime\runtime2.go`）：

```
type iface struct {
	tab  *itab
	data unsafe.Pointer
}

```

可以简单理解为，tab表示接口的具体结构类型，而data是接口的值。

- itab：

```
type itab struct {
	inter *interfacetype //此属性用于定位到具体interface
	_type *_type //此属性用于定位到具体interface
	hash  uint32 // copy of _type.hash. Used for type switches.
	_     [4]byte
	fun   [1]uintptr // variable sized. fun[0]==0 means _type does not implement inter.
}

```

属性`interfacetype`类似于`_type`，其作用就是interface的公共描述，类似的还有`maptype`、`arraytype`、`chantype`…其都是各个结构的公共描述，可以理解为一种外在的表现信息。interfaetype和type唯一确定了接口类型，而hash用于查询和类型判断。fun表示方法集。

- `eface`

与iface基本一致，但是用`_type`直接表示类型，这样的话就无法使用方法。

```
type eface struct {
	_type *_type
	data  unsafe.Pointer
}

```

这里篇幅有限，深入讨论可以看：[深入研究 Go interface 底层实现](https://link.zhihu.com/?target=https%3A//halfrost.com/go_interface/%23toc-1)

### 06go的reflect 底层实现

go reflect源码位于`src\reflect\`下面，作为一个库独立存在。反射是基于**接口**实现的。

Go反射有三大法则：

- 反射从**接口**映射到**反射对象；**

  给定一个数据对象，可以将数据对象转化为反射对象`Type`和`Value`。

![img](https://pic2.zhimg.com/80/v2-350518add3d5e2757a8bc98f3c6fc15d_1440w.webp)

- 反射从**反射对象**映射到**接口值**；

  给定的反射对象，可以转化为某种类型的数据对象。即法则一的逆向。

![img](https://pic3.zhimg.com/80/v2-c2354d13a1514a482efa60e3d8cff816_1440w.webp)

- 只有**值可以修改**(settable)，才可以**修改**反射对象。

  通过反射对象，可以修改原数据中的内容

Go反射基于上述三点实现。

type用于获取当前值的类型。value用于获取当前的值。

反射的意思是在运行时，能够动态知道给定数据对象的类型和结构，并有机会修改它！
现在一个数据对象，如何判断它是什么结构？
数据interface中保存有结构数据呀，只要想办法拿到该数据对应的内存地址，然后把该数据转成interface，通过查看interface中的类型结构，就可以知道该数据的结构了呀~
其实以上就是Go反射通俗的原理。

> 参考资料：[The Laws of Reflection](https://link.zhihu.com/?target=https%3A//go.dev/blog/laws-of-reflection)， [图解go反射实现原理](https://link.zhihu.com/?target=https%3A//i6448038.github.io/2020/02/15/golang-reflection/)

### 07说说context包的作用？你用过哪些，原理知道吗？

`**context的作用就是在不同的goroutine之间同步请求特定的数据、取消信号以及处理请求的截止日期**。

**原理**

Go 的 Context 的数据结构包含 Deadline，Done，Err，Value，

Deadline 方法返回一个 time.Time，表示当前 Context 应该结束的时间，ok 则表示有结束时间，

Done 方法当 Context 被取消或者超时时候返回的一个 close 的 channel，告诉给 context 相关的函数要停止当前工作然后返回了，

Err 表示 context 被取消的原因，

Value 方法表示 context 实现共享数据存储的地方，是协程安全的。

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

**其主要的应用 ：**

1：上下文控制，

2：多个 goroutine 之间的数据交互等，

3：超时控制：到某个时间点超时，过多久超时。

关于context原理，可以参看：[小白也能看懂的context包详解：从入门到精通](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/developer/article/1900658)

### **08讲讲 Go 的 defer 底层数据结构和一些特性？**

答：每个 defer 语句都对应一个_defer 实例，多个实例使用指针连接起来形成一个单链表，保存在 gotoutine 数据结构中，每次插入_defer 实例，均插入到链表的头部，函数结束再一次从头部取出，从而形成后进先出的效果。

**defer 的规则总结**：

- 延迟函数的参数是 defer 语句出现的时候就已经确定了的。


- 延迟函数执行按照**后进先出**的顺序执行，即先出现的 defer 最后执行。


- 延迟函数可能操作**主函数的返回值**。


- 申请资源后立即使用 defer 关闭资源是个好习惯。

# 并发编程

### 01 ❤无缓冲的 channel 和有缓冲的 channel 的区别？

对于无缓冲区channel：

发送的数据如果没有被接收方接收，那么**发送方阻塞；**如果一直接收不到发送方的数据，**接收方阻塞**；

有缓冲的channel：

缓冲区满的时候发送方阻塞（写阻塞)；缓冲区为空的时候接收方阻塞（读阻塞）。

两者的底层区别在于是否使用hchan.buf这个环形队列

### 02 为什么有协程泄露(Goroutine Leak)？

协程泄漏是指协程创建之后没有得到释放。主要原因有：

1. 缺少接收器，导致发送阻塞
2. 缺少发送器，导致接收阻塞
3. 死锁。多个协程由于竞争资源导致死锁。
4. 创建协程的没有回收。

### 03 Go 可以限制运行时操作系统线程的数量吗？ 常见的goroutine操作函数有哪些？

可以，使用runtime.GOMAXROCS(num int)可以设置线程数目。该值默认为CPU逻辑核数，如果设的太大，会引起频繁的线程切换，降低性能。

runtime.Gosched()，用于让出CPU时间片，让出当前goroutine的执行权限，调度器安排其它等待的任务运行，并在下次某个时候从该位置恢复执行。
runtime.Goexit()，调用此函数会立即使当前的goroutine的运行终止（终止协程），而其它的goroutine并不会受此影响。runtime.Goexit在终止当前goroutine前会先执行此goroutine的还未执行的defer语句。请注意千万别在主函数调用runtime.Goexit，因为会引发panic。

### 04 如何控制协程数目。

> The GOMAXPROCS variable limits the number of operating system threads that can execute user-level Go code simultaneously. There is no limit to the number of threads that can be blocked in system calls on behalf of Go code; those do not count against the GOMAXPROCS limit.

从官方文档的解释可以看到，`GOMAXPROCS` 限制的是同时执行用户态 Go 代码的操作系统线程的数量，但是对于被系统调用阻塞的线程数量是没有限制的。`GOMAXPROCS` 的默认值等于 CPU 的**逻辑核数**，同一时间，一个核只能绑定一个线程，然后运行被调度的协程。因此对于 CPU 密集型的任务，若该值过大，例如设置为 CPU 逻辑核数的 2 倍，会增加线程切换的开销，降低性能。对于 I/O 密集型应用，适当地调大该值，可以提高 I/O 吞吐率。

另外对于协程，可以用带缓冲区的channel来控制，下面的例子是协程数为1024的例子

```go
var wg sync.WaitGroup
ch := make(chan struct{}, 1024)
for i:=0; i<20000; i++{
	wg.Add(1)
	ch<-struct{}{}
	go func(){
		defer wg.Done()
		<-ch
	}
}
wg.Wait()
```

此外还可以用**协程池**：其原理无外乎是将上述代码中通道和协程函数解耦，并封装成单独的结构体。常见第三方协程池库，比如[tunny](https://link.zhihu.com/?target=http%3A//github.com/Jeffail/tunny)等。

### 05❤mutex有几种模式？

> 加锁时，如果当前Locked位为1，说明该锁当前由其他协程持有，尝试加锁的协程并不是马上转入阻塞，而是会持续的探测Locked位是否变为0，这个过程即为自旋过程。
>
> 自旋时间很短，但如果在自旋过程中发现锁已被释放，那么协程可以立即获取锁。此时即便有协程被唤醒也无法获取锁，只能再次阻塞。
>
> 自旋的好处是，当加锁失败时不必立即转入阻塞，有一定机会获取到锁，这样**可以避免协程的切换**。

mutex有两种模式：**normal** 和 **starvation**

**正常模式**

默认情况下，Mutex的模式为normal。

该模式下，协程如果加锁不成功不会立即转入阻塞排队，而是判断是否满足自旋的条件，如果满足则会启动自旋过程，尝试抢锁。公平性：否。

**饥饿模式**

> 自旋过程中能抢到锁，一定意味着同一时刻有协程释放了锁，我们知道释放锁时如果发现有阻塞等待的协程，还会释放一个信号量来唤醒一个等待协程，被唤醒的协程得到CPU后开始运行，此时发现锁已被抢占了，自己只好再次阻塞，不过阻塞前会判断自上次阻塞到本次阻塞经过了多长时间，如果超过1ms的话，会将Mutex标记为"饥饿"模式，然后再阻塞。 

处于饥饿模式下，不会启动自旋过程，也即一旦有协程释放了锁，那么一定会唤醒协程，被唤醒的协程将会成功获取锁，同时也会把等待计数减1。

在饥饿模式下，Mutex 的拥有者将直接把锁交给队列最前面的 waiter。新来的 goroutine 不会尝试获取锁，即使看起来锁没有被持有，它也不会去抢，也不会 spin（自旋），它会乖乖地加入到等待队列的尾部。 如果拥有 Mutex 的 waiter 发现下面两种情况的其中之一，它就会把这个 Mutex 转换成正常模式:

1. 此 waiter 已经是队列中的最后一个 waiter 了，没有其它的等待锁的 goroutine 了；
2. 此 waiter 的等待时间小于 1 毫秒。

公平性：是。

> 参考链接：[Go Mutex 饥饿模式](https://link.zhihu.com/?target=https%3A//blog.csdn.net/qq_37102984/article/details/115322706)，[GO 互斥锁（Mutex）原理](https://link.zhihu.com/?target=https%3A//blog.csdn.net/baolingye/article/details/111357407%23%3A~%3Atext%3D%25E6%25AF%258F%25E4%25B8%25AAMutex%25E9%2583%25BD%2Ctarving%25E3%2580%2582)



### 06go竞态条件了解吗？

所谓竞态竞争，就是当**两个或以上的goroutine访问相同资源时候，对资源进行读/写。**

比如`var a int = 0`，有两个协程分别对a+=1，我们发现最后a不一定为2.这就是竞态竞争。

通常我们可以用`go run -race xx.go`来进行检测。

解决方法是，对临界区资源上**锁**，或者使用**原子操作**(atomics)，原子操作的开销小于上锁。

### 07除了 mutex 以外还有那些方式安全读写共享变量？

- Golang中Goroutine 可以通过 Channel 进行安全读写共享变量。
- 可以用个数为 1 的信号量（semaphore）实现互斥
- 使用sync包，比如sync.Map

### 08悲观锁、乐观锁是什么？

**悲观锁**

悲观锁：当要对数据库中的一条数据进行修改的时候，为了避免同时被其他人修改，最好的办法就是直接对该数据进行加锁以防止并发。这种借助数据库锁机制，在修改数据之前先锁定，再修改的方式被称之为悲观并发控制【Pessimistic Concurrency Control，缩写“PCC”，又名“悲观锁”】。

**乐观锁**

乐观锁是相对悲观锁而言的，乐观锁假设数据一般情况不会造成冲突，所以在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果冲突，则返回给用户异常信息，让用户决定如何去做。乐观锁适用于**读多写少**的场景，这样可以提高程序的吞吐量

### 09goroutine 的自旋占用资源如何解决

自旋锁是指当一个线程在获取锁的时候，如果锁已经被其他线程获取，那么该线程将循环等待，然后不断地判断是否能够被成功获取，直到获取到锁才会退出循环。

**自旋的条件如下：**

1）还没自旋超过 **4** 次,

2）多核处理器，

3）GOMAXPROCS > 1，

4）p 上本地 goroutine 队列为空。

mutex 会让当前的 goroutine 去空转 CPU，在空转完后再次调用 CAS 方法去尝试性的占有锁资源，直到**不满足自旋条件**，则最终会加入到等待队列里。

### 10怎么控制并发数？

**第一，用缓冲通道**

根据通道中没有数据时读取操作陷入阻塞和通道已满时继续写入操作陷入阻塞的特性，正好实现控制并发数量。

```go
func main() {
	count := 10 // 最大支持并发
	sum := 100 // 任务总数
	wg := sync.WaitGroup{} //控制主协程等待所有子协程执行完之后再退出。

	c := make(chan struct{}, count) // 控制任务并发的chan
	defer close(c)

	for i:=0; i<sum;i++{
		wg.Add(1)
		c <- struct{}{} // 作用类似于waitgroup.Add(1)
		go func(j int) {
			defer wg.Done()
			fmt.Println(j)
			<- c // 执行完毕，释放资源
		}(i)
	}
	wg.Wait()
}
```

**第二，第三方库实现的协程池**

panjf2000/ants（比较火）

Jeffail/tunny

```go
import (
	"log"
	"time"

	"github.com/Jeffail/tunny"
)
func main() {
	pool := tunny.NewFunc(10, func(i interface{}) interface{} {
		log.Println(i)
		time.Sleep(time.Second)
		return nil
	})
	defer pool.Close()

	for i := 0; i < 500; i++ {
		go pool.Process(i)
	}
	time.Sleep(time.Second * 4)
}
```

### 11多个 goroutine 对同一个 map 写会 panic，异常是否可以用 defer 捕获？

**可以捕获异常，但是只能捕获一次**，Go语言，可以使用多值返回来返回错误。不要用异常代替错误，更不要用来控制流程。在极个别的情况下，才使用Go中引入的Exception处理：defer, panic, recover 

Go中，对异常处理的原则是：多用error包，少用panic

```go
defer func() {
		if err := recover(); err != nil {
			// 打印异常，关闭资源，退出此函数
			fmt.Println(err)
		}
	}()
```

### 12如何优雅的实现一个 goroutine 池

（百度、手写代码，本人面传音控股被问道：请求数大于消费能力怎么设计协程池）

这一块能啃下来，offer满天飞，这应该是保证高并发系统稳定性、高可用的核心部分之一。

```go
package main
 
import (
	"fmt"
	"time"
)

/* 有关Task任务相关定义及操作 */
//定义任务Task类型,每一个任务Task都可以抽象成一个函数
type Task struct {
	f func() error //一个无参的函数类型
}
 
//通过NewTask来创建一个Task
func NewTask(f func() error) *Task {
	t := Task{
		f: f,
	}
	return &t
}
 
//执行Task任务的方法
func (t *Task) Execute() {
	t.f() //调用任务所绑定的函数
}
 
/* 有关协程池的定义及操作 */
//定义池类型
type Pool struct {
	EntryChannel chan *Task //对外接收Task的入口
	worker_num   int        //协程池最大worker数量,限定Goroutine的个数
	JobsChannel  chan *Task //协程池内部的任务就绪队列
}
 
//创建一个协程池
func NewPool(cap int) *Pool {
	p := Pool{
		EntryChannel: make(chan *Task),
		worker_num:   cap,
		JobsChannel:  make(chan *Task),
	}
	return &p
}
 
//协程池创建一个worker并且开始工作
func (p *Pool) worker(work_ID int) {
	//worker不断的从JobsChannel内部任务队列中拿任务
	for task := range p.JobsChannel {
		//如果拿到任务,则执行task任务
		task.Execute()
		fmt.Println("worker ID ", work_ID, " 执行完毕任务")
	}
}
 
//让协程池Pool开始工作
func (p *Pool) Run() {
	//1,首先根据协程池的worker数量限定,开启固定数量的Worker,
	//  每一个Worker用一个Goroutine承载
	for i := 0; i < p.worker_num; i++ {
		fmt.Println("开启固定数量的Worker:", i)
		go p.worker(i)
	}
 
	//2, 从EntryChannel协程池入口取外界传递过来的任务
	//   并且将任务送进JobsChannel中
	for task := range p.EntryChannel {
		p.JobsChannel <- task
	}
 
	//3, 执行完毕需要关闭JobsChannel
	close(p.JobsChannel)
	fmt.Println("执行完毕需要关闭JobsChannel")
 
	//4, 执行完毕需要关闭EntryChannel
	close(p.EntryChannel)
	fmt.Println("执行完毕需要关闭EntryChannel")
}
 
//主函数
func main() {
	//创建一个Task
	t := NewTask(func() error {
		fmt.Println("创建一个Task:", time.Now().Format("2006-01-02 15:04:05"))
		return nil
	})
 
	//创建一个协程池,最大开启3个协程worker
	p := NewPool(3)
 
	//开一个协程 不断的向 Pool 输送打印一条时间的task任务
	go func() {
		for {
			p.EntryChannel <- t
		}
	}()
 
	//启动协程池p
	p.Run()
}
```



### 13❤go 中有哪些常用的并发模型？

Golang 中常用的并发模型有三种:

- 通过channel通知实现并发控制

  无缓冲的通道指的是通道的大小为0，也就是说，这种类型的通道在接收前没有能力保存任何值，它要求发送 goroutine 和接收 goroutine 同时准备好，才可以完成发送和接收操作。

  从上面无缓冲的通道定义来看，发送 goroutine 和接收 gouroutine 必须是同步的，同时准备后，如果没有同时准备好的话，先执行的操作就会阻塞等待，直到另一个相对应的操作准备好为止。这种无缓冲的通道我们也称之为同步通道。​

  ```go
  func main() {
   //通过无缓冲的channel通知实现并发控制
   //发送 goroutine 和接收 gouroutine 必须是同步的
   //如果没有同时准备好的话，先执行的操作就会阻塞等待，直到另一个相应的操作准备好位置
    
   ch := make(chan struct{})
   go func() {
       fmt.Println(start working)
       time.Sleep(time.Second * 1)
       ch <- struct{}{}
   }()
  // 当主 goroutine 运行到 <-ch 接受 channel 的值的时候，如果该 channel 中没有数据，就会一直阻塞等待，直到有值。这样就可以简单实现并发控制
   <-ch
   
   fmt.Println(finished)
  }
  ```

  ​

- 通过sync包中的WaitGroup实现并发控制

在主 goroutine 中 Add(delta int) 索要等待goroutine 的数量。在每一个 goroutine 完成后 Done() 表示这一个goroutine 已经完成，当所有的 goroutine 都完成后，在主 goroutine 中 WaitGroup 返回返回。

Goroutine是异步执行的，有的时候为了防止在结束mian函数的时候结束掉Goroutine，所以需要同步等待，这个时候就需要用 WaitGroup了，在 sync 包中，提供了 WaitGroup ，它会等待它收集的所有 goroutine 任务全部完成。在WaitGroup里主要有三个方法:

​	Add, 可以添加或减少 goroutine的数量.

​	Done, 相当于Add(-1).

​	Wait, 执行后会堵塞主线程，直到WaitGroup 里的值减至0.

```go
func main(){
   var wg sync.WaitGroup
   var urls = []string{
       http://www.golang.org/,
       http://www.google.com/,
   }
   for _, url := range urls {
       wg.Add(1)
       go func(url string) {
           defer wg.Done()
           http.Get(url)
       }(url)
   }
   wg.Wait()
 }

//会报错，死锁，可以把wg参数改为指针传递
/*
第一个修改方式:将匿名函数中 wg 的传入类型改为 *sync.WaitGrou,这样就能引用到正确的WaitGroup了。

第二个修改方式:将匿名函数中的 wg 的传入参数去掉，因为Go支持闭包类型，在匿名函数中可以直接使用外面的 wg 变量

因此 Wait 就死锁了。

它提示所有的 goroutine 都已经睡眠了，出现了死锁。这是因为 wg 给拷贝传递到了 goroutine 中，导致只有 Add 操作，其实 Done操作是在 wg 的副本执行的。

在Golang官网中对于WaitGroup介绍是A WaitGroup must not be copied after first use,在 WaitGroup 第一次使用后，不能被拷贝

*/
 func main(){
 	 wg := sync.WaitGroup{}
     for i := 0; i < 5; i++ {
         wg.Add(1)
         go func(wg sync.WaitGroup, i int) {
             fmt.Printf(i:%d, i)
             wg.Done()
         }(wg, i)
     }
     wg.Wait()
     fmt.Println(exit)
 }
```

- 通过强大的Context上下文，实现并发控制

  context 包主要是用来处理多个 goroutine 之间共享数据，及多个 goroutine 的管理。

# 其它

### 01你项目有优雅的启停吗？

所谓「优雅」启停就是在启动退出服务时要满足以下几个条件：

- **不可以关闭现有连接**（进程）
- 新的进程启动并「**接管**」旧进程
- 连接要**随时响应用户请求**，不可以出现拒绝请求的情况
- 停止的时候，必须**处理完既有连接**，并且**停止接收新的连接**。

为此我们必须引用**信号**来完成这些目的：

启动：

- 监听**SIGHUP**（在用户终端连接(正常或非正常)结束时发出）；
- 收到信号后将服务监听的文件描述符传递给新的子进程，此时新老进程同时接收请求；

退出：

- 监听**SIGINT和SIGSTP和SIGQUIT**等。
- 父进程停止接收新请求，等待旧请求完成（或超时）；
- 父进程退出。

实现：go1.8采用Http.Server内置的Shutdown方法支持优雅关机。 然后[fvbock/endless](https://link.zhihu.com/?target=http%3A//github.com/fvbock/endless)可以实现优雅重启。

> 参考资料：[gin框架实践连载八 | 如何优雅重启和停止 - 掘金](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6867074626427502600%23heading-3)，[优雅地关闭或重启 go web 项目](https://link.zhihu.com/?target=http%3A//www.phpxs.com/post/7186/)

### 02持久化怎么做的？

所谓持久化就是将要保存的字符串写到硬盘等设备。

- 最简单的方式就是采用ioutil的WriteFile()方法将字符串写到磁盘上，这种方法面临**格式化**方面的问题。
- 更好的做法是将数据按照**固定协议**进行组织再进行读写，比如JSON，XML，Gob，csv等。
- 如果要考虑**高并发**和**高可用**，必须把数据放入到**数据库**中，比如MySQL，PostgreDB，MongoDB等。

参考链接：[Golang 持久化](https://link.zhihu.com/?target=https%3A//www.jianshu.com/p/015aca3e11ae)

### 03GIN怎么做参数校验？

go采用validator包作参数校验。

它具有以下独特功能：

- 使用**验证tag或自定义validator**进行跨字段Field和跨结构体验证。
- 允许切片、数组和哈希表，多维字段的任何或所有级别进行校验。
- 能够对哈希表key和value进行验证
- 通过在验证之前确定它的基础类型来处理类型接口。
- 别名验证标签，允许将多个验证映射到单个标签，以便更轻松地定义结构体上的验证
- **gin web 框架的默认验证器**；

参考资料：[validator package - pkg.go.dev](https://link.zhihu.com/?target=https%3A//pkg.go.dev/github.com/go-playground/validator%23section-readme)

### 04中间件用过吗？

Middleware是Web的重要组成部分，中间件（通常）是一小段代码，它们接受一个请求，对其进行处理，每个中间件只处理一件事情，完成后将其传递给另一个中间件或最终处理程序，这样就做到了程序的解耦。

### 05进程挂了怎么办

> 首先需要问清挂了是指进程阻塞还是被kill了
>
> 当有客户端连接到 HTTP 服务器（或任何 TCP 服务器）并发送需要大量计算的请求时，每个请求都需要执行一些可能需要 “**任意时间才能完成”**代码。如果我们将这个耗时的代码隔离在一个函数中，我们就可以将此函数称为 **阻塞调用。**
>
> **也就是说，调用结果返回之前，当前线程会被挂起。**
>
> 简单的例子是查询数据库的函数或操作大图像文件的函数。在一个连接将由其自己的[专用线](https://www.zhihu.com/search?q=%E4%B8%93%E7%94%A8%E7%BA%BF&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2691425417%7D)程处理的旧模型中，这不是问题。但是在单个线程将处理数千个连接的新反应器模型中，只需要一个连接执行阻塞调用，就会影响和阻塞所有其他连接。
>
> 那么我们如何在不回到专用线程模型的情况下解决这个问题呢？

**解决方案1：线程并发**

你基本上使用CoralQueue将请求的工作（而不是请求本身）分配给固定数量的线程，这些线程同时执行（即并行）。假设您有 1000 个同时连接。除了拥有 1000 个并发线程（即不切实际 *的每个连接一个线程* 模型），您可以分析您的机器有多少可用的 CPU 内核并选择更少的[线程数](https://www.zhihu.com/search?q=%E7%BA%BF%E7%A8%8B%E6%95%B0&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2691425417%7D)，比如说 4。这种架构将为您提供以下优点：

- 处理 HTTP 服务器请求的关键反应器线程永远不会阻塞，因为每个请求所需的工作将简单地添加到队列中，从而释放反应器线程以处理额外的传入 HTTP 请求。
- 即使一两个线程收到一个需要很长时间才能完成的请求，其他线程也可以继续排空队列中的请求。

如果你能提前猜到哪些请求会花费很长时间执行，你甚至可以将队列划分为通道，并为高优先级/快速请求设置一个快速通道，这样它们总能找到一个空闲线程来执行。

**解决方案2：分布式系统**

你可以使用分布式系统架构并利用异步网络调用，而不是在一台机器上使用有限的 CPU 内核做所有事情。这简化了处理请求的 HTTP 服务器，现在不需要任何额外的线程和并发队列。它可以在单个非阻塞反应器线程中完成所有操作。它是这样工作的：

- 您可以将此任务移动到另一个 *节点* （即另一个进程或机器）（通过负载均衡器），而不是在 HTTP 服务器本身上进行繁重的计算。
- 您可以简单地对负责繁重计算任务的节点进行异步网络调用，而不是使用 CoralQueue 跨线程分配工作。
- HTTP 服务器将 **异步等待** 来自繁重计算节点的响应。响应可能需要尽可能长的时间才能通过网络到达，因为 **HTTP 服务器永远不会阻塞**。
- HTTP 服务器只能使用一个线程来处理来自[外部客户](https://www.zhihu.com/search?q=%E5%A4%96%E9%83%A8%E5%AE%A2%E6%88%B7&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2691425417%7D)端的传入 HTTP 连接以及到执行繁重计算工作的[内部节点](https://www.zhihu.com/search?q=%E5%86%85%E9%83%A8%E8%8A%82%E7%82%B9&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2691425417%7D)的传出 HTTP 连接。
- 它的美妙之处在于，您可以通过根据需要简单地添加更多节点来进行扩展。故障转移和[负载平衡](https://www.zhihu.com/search?q=%E8%B4%9F%E8%BD%BD%E5%B9%B3%E8%A1%A1&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2691425417%7D)也变得微不足道。

### 06.nginx配置过吗，有哪些注意的点

> Nginx是一款轻量级的HTTP服务器，采用事件驱动的异步非阻塞处理方式框架，这让其具有极好的IO性能，时常用于服务端的反向代理和负载均衡。

**反向代理**

实现效果：使用 Nginx 反向代理，访问www.123.com直接跳转到127.0.0.1:8080

注意：此处如果要想从www.123.com跳转到本机指定的ip，需要修改本机的hosts文件。此处略过

配置代码

```nginx
server {
    listen       80;
    server_name  www.123.com;

    location / {
        root   html;
        index  index.html index.htm;
        proxy_pass  http://127.0.0.1:8080
    }
}
```

如上配置，`Nginx`监听 `80`端口，访问域名为`www.123.com`（不加端口号时默认为 `80`端口），故访问该域名时会跳转到 `127.0.0.1:8080` 路径上。

反向代理还能解决**跨域**问题

**负载均衡**

![img](https://img-blog.csdnimg.cn/20201102193513560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pwZjE4MTM3NjM2Mzc=,size_16,color_FFFFFF,t_70#pic_center)

几种方式：

- 轮询（默认）
- 加权轮询：weight 代表权重，默认为1,权重越高被分配的客户端越多。指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
- ip_hash：每个请求按访问ip的hash值分配，这样每个访问客户端会固定访问一个后端服务器，可以解决**会话Session丢失的问题**
- 最少连接

### 07.mq消费阻塞怎么办

消费阻塞主要有以下几个原因：

- 消费者的集群数量有问题
- 消费者的consumer group乱用，就是明明你不消费这个topic , 但是你的消费组名称取的跟别人正常消费的一样，这个时候，就会导致一部分队列分配到你这个节点身上，但是这些节点你本身又没有去消费，就会造成消费混乱
- 消费者线程阻塞

基于以上原因排查，基本上第一个可能性几乎没有，第二个需要开发者规范消费者的命令，第三个需要排查并修改代码（可能由于消费消息处理业务的时候，发起http请求，没有设置超时时间，导致在出现网络异常的情况下，程序直接僵死），重启服务

### 08.性能没达到预期，有什么解决方案

| 1 准备阶段

准备阶段是非常关键的一步，不能省略。

首先，需要对我们进行调优的对象进行详尽的了解，所谓知己知彼，百战不殆。

1. **对性能问题进行粗略评估，过滤一些因为低级的业务逻辑导致的性能问题**。譬如，线上应用日志级别不合理，可能会在大流量时导致 CPU 和磁盘的负载飙高，这种情况调整日志级别即可；
2. **了解应用的的总体架构**，比如应用的外部依赖和核心接口有哪些，使用了哪些组件（数据库MYSQL的查询优化和索引优化等的）和框架，哪些接口和模块的**使用率**较高，上下游的**数据链路**是怎么样的等；
3. **了解应用对应的服务器信息**，如服务器所在的**集群信息**、服务器的 **CPU/内存**信息、安装的 Linux 版本信息、服务器是容器还是虚拟机、所在宿主机混部后是否对当前应用有干扰等；

其次，我们需要获取基准数据，然后结合基准数据和当前的一些业务指标，确定此次性能优化的最终目标。

1. **使用基准测试工具获取系统细粒度指标**。可以使用若干 Linux 基准测试工具（eg. jmeter、ab、loadrunnerwrk、wrk等），得到文件系统、**磁盘 I/O**、网络等的性能报告。除此之外，类似 GC、Web 服务器、网卡流量等信息，如有必要也是需要了解记录的；
2. **通过压测工具或者压测平台**（如果有的话），对应用进行压力测试，获取当前应用的**宏观业务指标**，譬如：响应时间、吞吐量、TPS、QPS、消费速率（对于有 MQ 的应用）等。压力测试也可以省略，可以结合当前的实际业务和过往的监控数据，去统计当前的一些核心业务指标，如午高峰的服务 TPS。
3. **通过性能分析工具分析程序运行的性能**。比如使用pporf分析go语言代码执行的CPU和内存占用情况，锁定CPU占用较高的代码进行分析测试调优

| 2 测试阶段

进入到这一阶段，说明我们已经初步确定了应用性能瓶颈的所在，而且已经进行初步的调优了。**检测我们调优是否有效的方式，就是在仿真的条件下，对应用进行压力测试**。注意：由于 Java 有 JIT（just-in-time compilation）过程，因此压力测试时可能需要进行前期预热。

如果压力测试的结果符合了预期的调优目标，或者与基准数据相比，有很大的改善，则我们可以继续通过工具定位下一个瓶颈点，否则，则需要暂时排除这个瓶颈点，继续寻找下一个变量。

**注意事项**

在进行性能优化时，了解下面这些注意事项可以让我们少走一些弯路。

1. 性能瓶颈点通常呈现 **2/8 分布**，即80%的性能问题通常是由20%的性能瓶颈点导致的，2/8 原则也意味着并不是所有的性能问题都值得去优化；
2. 性能优化是一个渐进、迭代的过程，需要逐步、动态地进行。记录基准后，每次改变一个变量，引入多个变量会给我们的观测、优化过程造成干扰；
3. 不要过度追求应用的单机性能，如果单机表现良好，则应该从系统架构的角度去思考; 不要过度追求单一维度上的极致优化，如过度追求 CPU 的性能而忽略了内存方面的瓶颈；
4. 选择合适的性能优化工具，可以使得性能优化取得事半功倍的效果；
5. **整个应用的优化，应该与线上系统隔离，新的代码上线应该有降级方案**。

**瓶颈点分析工具箱**

> `pprof` 是 Go 语言中分析程序运行性能的工具，它能提供各种性能数据：
>
> ![img](https://user-images.githubusercontent.com/7698088/68523507-3ce36500-02f5-11ea-8e8f-438c9ef2b9f8.png)

性能优化其实就是找出应用存在性能瓶颈点，然后设法通过一些调优手段去缓解。性能瓶颈点的定位是较困难的，快速、直接地定位到瓶颈点，需要具备下面两个条件：

1. 恰到好处的工具；
2. 一定的性能优化经验。

工欲善其事，必先利其器，我们该如何选择合适的工具呢？不同的优化场景下，又该选择那些工具呢？

下面给出了一张更为实用的「性能优化工具图谱」，该图分别从系统层、应用层（含组件层）的角度出发，列举了我们在分析性能问题时首先需要关注的各项指标（其中?标注的是最需要关注的），这些点是最有可能出现性能瓶颈的地方。需要注意的是，一些低频的指标或工具，在图中并没有列出来，如 CPU 中断、索引节点使用、I/O事件跟踪等，这些低频点的排查思路较复杂，一般遇到的机会也不多，在这里我们聚焦最常见的一些就可以了。

对比上面的性能工具(Linux Performance Tools-full)图，下图的优势在于：把具体的工具同性能指标结合了起来，同时从不同的层次去描述了性能瓶颈点的分布，实用性和可操作性更强一些。系统层的工具分为CPU、内存、磁盘（含文件系统）、网络四个部分，工具集同性能工具(Linux Performance Tools-full)图中的工具基本一致。组件层和应用层中的工具构成为：JDK 提供的一些工具 + Trace 工具 + dump 分析工具 + Profiling 工具等。

这里就不具体介绍这些工具的具体用法了，我们可以使用 man 命令得到工具详尽的使用说明，除此之外，还有另外一个查询命令手册的方法：info。info 可以理解为 man 的详细版本，如果 man 的输出不太好理解，可以去参考 info 文档，命令太多，记不住也没必要记住。

![img](https://pic2.zhimg.com/80/v2-ab5e2e3da9d497bcc3e91231e7002ed1_1440w.webp)

### 09函数传递指针真的比传值效率高吗？

我们知道传递指针可以减少底层值的拷贝，可以提高效率，但是如果拷贝的数据量小，由于指针传递会产生逃逸，可能会使用堆，也可能会增加GC的负担，所以传递指针不一定是高效的。

### 10go里用过哪些设计模式 ?

> 参考[senghoo/golang-design-pattern: 设计模式 Golang实现－《研磨设计模式》读书笔记 (github.com)](https://github.com/senghoo/golang-design-pattern)
>
> 设计模式的六大原则
>
> 一、单一职责原则（Single Responsibility Principle）
>
> 二、开闭原则(Open-Closed Principle, OCP)
>
> 三、里氏代换原则(Liskov Substitution Principle, LSP)
>
> 四、依赖倒置原则（Dependence Inversion Principle，DIP）
>
> 五、接口隔离原则(Interface  Segregation Principle, ISP)
>
> 六、迪米特法则(Law of  Demeter, LoD)

- 简单工厂模式：`go` 语言没有构造函数一说，所以一般会定义 `NewXXX` 函数来初始化相关类。`NewXXX` 函数返回接口时就是简单工厂模式，也就是说 `Golang` 的一般推荐做法就是简单工厂。

- 单例模式：使用懒惰模式的单例模式（使用sync.Once.Do保证在需要时再加载），使用双重检查加锁保证线程安全

  ```go
  package singleton

  import "sync"

  // Singleton 是单例模式接口，导出的
  // 通过该接口可以避免 GetInstance 返回一个包私有类型的指针
  type Singleton interface {
     foo()
  }
  // singleton 是单例模式类，包私有的
  type singleton struct{}

  func (s singleton) foo() {}

  var (
     instance *singleton
     once     sync.Once
  )

  // GetInstance 用于获取单例模式对象
  func GetInstance() Singleton {
     once.Do(func() {
        instance = &singleton{}
     })

     return instance
  }
  ```

- 装饰者模式：有时候我们需要在一个类的基础上扩展另一个类，例如，一个披萨类，你可以在披萨类的基础上增加番茄披萨类和芝士披萨类。此时就可以使用装饰模式，简单来说，**装饰模式就是将对象封装到另一个对象中，用以为原对象绑定新的行为**。

  ```go
  package decorator

  type pizza interface {
    getPrice() int
  }

  type base struct {}

  func (p *base) getPrice() int {
    return 15
  }

  type tomatoTopping struct {
    pizza pizza
  }

  func (c *tomatoTopping) getPrice() int {
    //装饰者模式
    pizzaPrice := c.pizza.getPrice()
    return pizzaPrice + 10
  }

  type cheeseTopping struct {
    pizza pizza
  }

  func (c *cheeseTopping) getPrice() int {
    //装饰者模式
    pizzaPrice := c.pizza.getPrice()
    return pizzaPrice + 20
  }
  ```


- 代理模式：如果你需要在访问一个对象时，有一个像“代理”一样的角色，她可以在访问对象之前为你进行缓存检查、权限判断等访问控制，在访问对象之后为你进行结果缓存、日志记录等结果处理，那么就可以考虑使用代理模式。

  ```go
  package proxy

  import "fmt"

  type Subject interface {
    Proxy() string
  }

  // 代理

  type Proxy struct {
    real RealSubject
  }

  func (p Proxy) Proxy() string {
    var res string

    // 在调用真实对象之前，检查缓存，判断权限，等等
    p.real.Pre()

    // 调用真实对象
    p.real.Real()

    // 调用之后的操作，如缓存结果，对结果进行处理，等等
    p.real.After()

    return res
  }

  // 真实对象

  type RealSubject struct{}

  func (RealSubject) Real() {
    fmt.Print("real")
  }

  func (RealSubject) Pre() {
    fmt.Print("pre:")
  }

  func (RealSubject) After() {
    fmt.Print(":after")
  }
  ```

- 观察者模式：如果你需要在一个对象的状态被改变时，其他对象能作为其“观察者”而被通知，就可以使用观察者模式。

  我们将自身的状态改变就会通知给其他对象的对象称为“发布者”，关注发布者状态变化的对象则称为“订阅者”。

  ```go
  package observer

  import "fmt"

  // 发布者

  type Subject struct {
    observers []Observer
    content   string
  }

  func NewSubject() *Subject {
    return &Subject{
      observers: make([]Observer, 0),
    }
  }

  // 添加订阅者

  func (s *Subject) AddObserver(o Observer) {
    s.observers = append(s.observers, o)
  }

  // 改变发布者的状态

  func (s *Subject) UpdateContext(content string) {
    s.content = content
    s.notify()
  }

  // 通知订阅者接口

  type Observer interface {
    Do(*Subject)
  }

  func (s *Subject) notify() {
    for _, o := range s.observers {
      o.Do(s)
    }
  }

  // 订阅者

  type Reader struct {
    name string
  }

  func NewReader(name string) *Reader {
    return &Reader{
      name: name,
    }
  }

  func (r *Reader) Do(s *Subject) {
    fmt.Println(r.name + " get " + s.content)
  }
  ```

  ​

### 11go的调试/分析工具用过哪些。

go的自带工具链相当丰富，

- go cover : 测试代码覆盖率；
- godoc: 用于生成go文档；
- pprof：用于性能调优，针对cpu，内存和并发；
- race：用于竞争检测；

### 12grpc为啥好，基本原理是什么，和http比呢

官方介绍：gRPC 是一个现代开源的**高性能远程过程调用** (RPC) 框架，可以在**任何环境**中运行。它可以通过对负载平衡、跟踪、健康检查和身份验证的可插拔支持有效地连接数据中心内和跨数据中心的服务。它也适用于分布式计算的最后一英里，将设备、移动应用程序和浏览器连接到后端服务。

**原理**

- NettyServer 实例创建：gRPC 服务端创建，首先需要初始化 NettyServer，它是 gRPC 基于 Netty 4.1 HTTP/2 协议栈之上封装的 HTTP/2 服务端。NettyServer 实例由 NettyServerBuilder 的 buildTransportServer 方法构建，NettyServer 构建完成之后，监听指定的 Socket 地址，即可实现基于 HTTP/2 协议的请求消息接入。

  ![img](https://pic4.zhimg.com/80/v2-a43c13dc025eba5de4e522a94f365a2b_1440w.webp)


- **绑定 IDL**（IDL是Interface description language的缩写，指接口描述语言，是[CORBA](https://baike.baidu.com/item/CORBA/2776997?fromModule=lemma_inlink)规范的一部分，是跨平台开发的基础） **定义的服务接口实现类**：gRPC 与其它一些 RPC 框架的差异点是服务接口实现类的调用并不是通过动态代理和反射机制，而是通过 proto 工具生成代码，在服务端启动时，将服务接口实现类实例注册到 gRPC 内部的服务注册中心上。请求消息接入之后，可以根据服务名和方法名，直接调用启动时注册的服务实例，而不需要通过反射的方式进行调用，性能更优。


- gRPC 服务实例（ServerImpl）构建：ServerImpl 负责整个 gRPC 服务端消息的调度和处理，创建 ServerImpl 实例过程中，会对服务端依赖的对象进行初始化，例如 Netty 的线程池资源、gRPC 的线程池、内部的服务注册类（InternalHandlerRegistry）等，ServerImpl 初始化完成之后，就可以调用 NettyServer 的 start 方法启动 HTTP/2 服务端，接收 gRPC 客户端的服务调用请求
- 服务端线程模型：**gRPC 的线程由 Netty 线程 + gRPC 应用线程组成**，它们之间的交互和切换比较复杂。gRPC 服务端调度线程为 SerializingExecutor，它实现了 Executor 和 Runnable 接口，通过外部传入的 Executor 对象，调度和处理 Runnable，同时内部又维护了一个任务队列 ConcurrentLinkedQueue，通过 run 方法循环处理队列中存放的 Runnable 。**gRPC 线程模型存在的一个缺点，就是在一次 RPC 调用过程中，做了多次 I/O 线程到应用线程之间的切换，频繁切换会导致性能下降**，这也是为什么 gRPC 性能比一些基于私有协议构建的 RPC 框架性能低的一个原因。尽管 gRPC 的性能已经比较优异，但是仍有一定的优化空间。

**优点**

- **生态好**：背靠Google。还有比如nginx也对grpc提供了支持，[参考链接](https://link.zhihu.com/?target=https%3A//nginx.org/en/docs/http/ngx_http_grpc_module.html)
- **跨语言**：跨语言，且自动生成sdk
- **协议可插拔：**不同的服务可能需要使用不同的消息通信类型和编码机制，例如，JSON、XML 和 Thirft, 所以协议应允许可插拔机制，还有负载均衡，服务发现，日志，监控等都支持可插拔机制
- **性能高**：比如protobuf性能高过json, 比如http2.0性能高过http1.1
- **强类型**：编译器就给你解决了很大一部分问题
- **流式处理（基于http2.0）**：支持客户端流式，服务端流式，双向流式

**区别：**

- rpc是**远程过程调用**，就是本地去调用一个远程的函数，而http是通过 **url**和符合**restful**风格的数据包去发送和获取数据；
- rpc的一般使用的编解码协议更加高效，比如grpc使用**protobuf**编解码。而http的一般使用json进行编解码，数据相比rpc更加直观，但是数据包也更大，效率低下；
- rpc一般用在服务内部的**相互调用**，而http则用于和**用户交互**；

**相似点**：
都有类似的机制，例如grpc的metadata机制和http的头机制作用相似，而且web框架，和rpc框架中都有拦截器的概念。grpc使用的是http2.0协议。
官网：[gRPC](https://link.zhihu.com/?target=https%3A//grpc.io/)

### 13.什么是死锁？如何避免？

死锁是**指两个或者两个以上进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象**。

> 产生死锁的四个必要条件
>
> - 互斥条件：该资源任意一个时刻只由一个线程占用。
> - 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
> - 不剥夺条件:线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
> - 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

如何避免死锁

针对四个必要条件，只要破坏其中一条，即可破坏死锁的产生。最常见的并且可行的就是**使用资源有序分配法，来破环循环等待条件**

- 破坏请求与保持条件 ：**一次性申请所有的资源**。
- 破坏不剥夺条件 ：占用部分资源的线程进一步申请其他资源时，如果申请不到，**可以主动释放它占有的资源。**
- 破坏循环等待条件 ：**靠按序申请资源来预防**。按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。

# 总结

Go面试复习应该有所侧重，关注切片，通道，异常处理，Goroutine，GMP模型，字符串高效拼接，指针，反射，接口，sync。对于比较难懂的部分，GMP模型和GC和内存管理，应该主动去看**源码**，然后慢慢理解。业务代码写多了，自然就有理解了。