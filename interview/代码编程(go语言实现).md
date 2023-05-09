### 1.实现使用字符串函数名，调用函数。

思路：采用反射的Call方法实现。

```go
package main
import (
	"fmt"
    "reflect"
)

type Animal struct{ 
}

func (a *Animal) Eat(){
    fmt.Println("Eat")
}

func main(){
    a := Animal{}
    reflect.ValueOf(&a).MethodByName("Eat").Call([]reflect.Value{})
}

```

### 2（Goroutine）有三个函数，分别打印"cat", "fish","dog"要求每一个函数都用一个goroutine，按照顺序打印100次。

此题目考察channel，**用三个无缓冲channel**，如果一个channel收到信号则通知下一个。

```go
package main

import (
	"fmt"
	"time"
)

var cat = make(chan struct{})
var fish = make(chan struct{})
var dog = make(chan struct{})

func Cat() {
	<-cat
	fmt.Println("cat")
	fish <- struct{}{}
}

func Fish() {
	<-fish
	fmt.Println("fish")
	dog <- struct{}{}
}

func Dog() {
	<-dog
	fmt.Println("dog")
	cat <- struct{}{}
}

func main() {
	for i := 0; i < 100; i++ {
		go Cat()
		go Fish()
		go Dog()
	}
	cat <- struct{}{}
	time.Sleep(2 * time.Second)
}
```

### 3两个协程交替打印10个字母和数字

思路：采用channel来协调goroutine之间顺序。

主线程一般要waitGroup等待协程退出，这里简化了一下直接sleep。

```go
package main

import (
	"fmt"
	"time"
)

var word = make(chan struct{}, 1)
var num = make(chan struct{}, 1)

func printNums() {
	for i := 0; i < 10; i++ {
		<-word
		fmt.Println(1)
		num <- struct{}{}
	}
}
func printWords() {
	for i := 0; i < 10; i++ {
		<-num
		fmt.Println("a")
		word <- struct{}{}
	}
}

func main() {
	num <- struct{}{}
	go printNums()
	go printWords()
	time.Sleep(time.Second * 1)
}
```

### 4启动 2个groutine 2秒后取消， 第一个协程1秒执行完，第二个协程3秒执行完。

思路：采用`ctx, _ := context.WithTimeout(context.Background(), time.Second*2)`实现2s取消。协程执行完后通过channel通知，是否超时。

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func f1(in chan struct{}) {

	time.Sleep(1 * time.Second)
	in <- struct{}{}

}

func f2(in chan struct{}) {
	time.Sleep(3 * time.Second)
	in <- struct{}{}
}

func main() {
	ch1 := make(chan struct{})
	ch2 := make(chan struct{})
	ctx, _ := context.WithTimeout(context.Background(), 2*time.Second)

	go func() {
		go f1(ch1)
		select {
		case <-ctx.Done():
			fmt.Println("f1 timeout")
			break
		case <-ch1:
			fmt.Println("f1 done")
		}
	}()

	go func() {
		go f2(ch2)
		select {
		case <-ctx.Done():
			fmt.Println("f2 timeout")
			break
		case <-ch2:
			fmt.Println("f2 done")
		}
	}()
	time.Sleep(time.Second * 5)
}
```

### 5当select监控多个chan同时到达就绪态时，如何先执行某个任务？

可以在子case再加一个for select语句。

```go
func priority_select(ch1, ch2 <-chan string) {
	for {
		select {
		case val := <-ch1:
			fmt.Println(val)
		case val2 := <-ch2:
		priority:
			for {
				select {
				case val1 := <-ch1:
					fmt.Println(val1)

				default:
					break priority
				}
			}
			fmt.Println(val2)
		}
	}
}
```

### 6了解过选项模式吗？能否写一段代码实现一个函数选项模式？

选项模式是 go 语法所特有的，也是 go 语言的创始人所推崇的，**可以做到灵活的给接口提供参数，且参数的数量可以自定义，同时屏蔽了一些不需要对接口使用者的细节**。

```go
 package main

import "fmt"

// 选项设计模式
// 问题：有一个结构体，定义一个函数，给结构体初始化

// 结构体
type Options struct {
    str1 string
    int1 int
}

// 声明一个函数类型的变量，用于传参
type Option func(opts *Options)

func InitOptions(opts ...Option) {
    options := &Options{}
    for _, opt := range opts {
        opt(options)
    }
    fmt.Printf("options:%#v\n", options)
}

func WithString1(str string) Option {
    return func(opts *Options) {
        opts.str1 = str
    }
}

func WithInt1(int1 int) Option {
    return func(opts *Options) {
        opts.int1 = int1
    }
}

func main() {
    InitOptions(WithString1("5lmh.com"),WithInt1(5))
}
```

