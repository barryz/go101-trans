> 译自：[Memory Order Guarantees In Go](https://go101.org/article/memory-model.html)。

# Go中的内存顺序保证

## 关于内存顺序

现代编译器（在编译期间）和CPU处理器（在运行期间）通常针对指令顺序都做了一些优化，所以指令的执行顺序一般和代码中的顺序不一致。

_（指令重排也常被称作[内存排序](https://en.wikipedia.org/wiki/Memory_ordering)。）_

当然， 指令重排不是随便就可以的。在指定的Goroutine内重排的最基本的要求是：如果Goroutine不与其他的Goroutine共享数据，那么Goroutine本身必须不能检测到重排。换句话说，从这样的Goroutine角度来看，它可以认为它的指令执行顺序总是和其代码指定的顺序是一致的，即使它内部确实发生了一些指令重排。

但是，如果一些Goroutines共享了某些数据，发生在这些Goroutines内部的指令重排可能就会被其他的Goroutines观察到，并且会影响到所有这些Goroutines。共享数据在并发编程中非常常见，如果我们忽略了指令重排导致的结果，那么我们的并发程序的行为可能依赖于编译器和CPU，会经常出现异常。

这里有一个不专业的Go程序，该程序没有考虑到指令重排。该程序从[Go1 memory model](https://golang.org/ref/mem)中的一个示例扩展而来。

```go
package main

import "runtime"

var a string
var done bool

func setup() {
	a = "hello, world"
	done = true
}

func main() {
	go setup()

	for !done {
		runtime.Gosched()
	}
	println(a) // expect to print: hello, world
}
```

上面这个程序的行为非常有可能如我们期待的那样，最终打印出`hello, world`。但是，实际上这个程序的行为依赖于具体的编译器额CPU。如果使用不同的编译器来编译这个程序，或者（这个程序）运行在不同架构的平台上，它有可能什么也不打印，或者是一个和`hello, world`不同的文本。原因是不同的编译器或CPU会交换函数`setup`中两行代码的执行顺序，所以对函数`setup`造成的影响可能就是：

```go
func setup() {
	done = true
	a = "hello, world"
}
```

上面程序中的`setup`的Goroutine并不能观察到自身的指令重排，但是，主Goroutine可以观察到这一现象。

除了重排问题之外，该程序中也存在数据竞争的问题。变量`a`和`done`上并没有使用数据同步技术。所以，上面的程序是一个充斥着各种错误的。专业的Go程序员不应该犯这些错误。

## Go的内存模型

有时候，我们需要确保在一个Goroutine中某些代码必须在另一个Goroutine中的某些代码执行之前（或之后）执行，以此保证程序的正确性。在这种情况下，指令重拍可能会导致一些问题。那么我们应该如何避免一些可能的指令重排呢？

不同的CPU架构提供了不同的栅栏指令以防止不同类型的指令重新排序。一些编程语言提供了相应的函数以在代码中插入这些栅栏指令。但是，理解并正确使用这些栅栏指令会提高并发编程的难度。
