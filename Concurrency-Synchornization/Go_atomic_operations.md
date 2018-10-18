> 译自：[Atomic Operations Provided In The sync/atomic Standard Package](https://go101.org/article/concurrent-atomic-operation.html) :book:

# 标准库`sync/atomic`提供的原子操作

原子操作比其他同步机器更加原始。它们是无锁的，且通常实现在硬件层面。实际上，它们经常用来实现其他的同步技术。

请注意，下面将要介绍的很多例子都不是并发程序，它们仅是为了演示的目的，为了展示如何使用`sync/atomic`标准库下提供的一系列的原子函数。

##  Go中的原子操作概览

`sync/atomic`标准库为一个整型类型`T`提供了下列5种原子方法。这里的`T`必须是类型`int32`， `int64`，`uint32`，`uint64`和`uintptr`之一。

```go
func AddT(addr *T, delta T)(new T)
func LoadT(addr *T) (val T)
func StoreT(addr *T, val T)
func SwapT(addr *T, new T) (old T)
func CompareAndSwapT(addr *T, old, new T) (swapped bool)
```

例如，下面有5个提供给类型`int32`的原子操作函数的例子。

```go
func AddInt32(addr *int32, delta int32)(new int32)
func LoadInt32(addr *int32) (val int32)
func StoreInt32(addr *int32, val int32)
func SwapInt32(addr *int32, new int32) (old int32)
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)
```
