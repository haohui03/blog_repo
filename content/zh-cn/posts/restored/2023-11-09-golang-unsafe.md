---
title: golang-unsafe
date: '2023-11-09T13:01:23+08:00'
lastmod: '2023-11-09T12:27:30+08:00'
draft: false
tags:
- golang
- language
categories:
- code
url: /zh-cn/2023/11/09/category/golang-unsafe/
authors:
- yehh
---

## unsafe包介绍 {#unsafe包介绍}

首先，unsafe正如其名是不安全的。

> 来自go语言官方文档的警告

> unsafe包的文档是这么说的：

> 导入unsafe的软件包可能不可移植，并且不受Go 1兼容性指南的保护。

> Go 1 兼容性指南这么说：

> 导入unsafe软件包可能取决于Go实现的内部属性。 我们保留对可能导致程序崩溃的实现进行更改的权利。

golang里面的三个指针类型

- \*类型:普通指针类型，用于传递对象地址，不能进行指针运算。
- unsafe.Pointer:通用指针类型，用于转换不同类型的指针，不能进行指针运算，不能读取内存存储的值（必须转换到某一类型的普通指针）。
- uintptr:用于指针运算，GC 不把 uintptr 当指针，uintptr 无法持有对象。uintptr 类型的目标会被回收。

### unsafe包里面 {#unsafe包里面}

函数：

func Alignof（variable ArbitraryType）uintptr

func Offsetof（selector ArbitraryType）uintptr

func Sizeof（variable ArbitraryType）uintptr

这里，ArbitraryType不是一个真正的类型，它只是一个占位符。与Golang中的大多数函数不同，上述三个函数的调用将始终在编译时求值，而不是运行时。 这意味着它们的返回结果可以分配给常量。

func String(ptr \*byte, len IntegerType) string //(go 1.20)

func StringData(str string) \*byte //(go 1.20)

这里只讲后两个新加的函数，StringData用于从string中获取地址，String函数用于从特定的字符串地址中截取len长度的子字符串。

类型：

```go
type ArbitraryType
    //其实和上面String的两个函数一样
    func Slice(ptr *ArbitraryType, len IntegerType) []ArbitraryType

    func SliceData(slice []ArbitraryType) *ArbitraryType
    //SliceData returns a pointer to the underlying array of the argument slice.

    //If cap(slice) > 0, SliceData returns &slice[:1][0].
    //If slice == nil, SliceData returns nil.
    //Otherwise, SliceData returns a non-nil pointer to an unspecified memory address.


type IntegerType
type Pointer
    //其实就是封装了地址偏移函数，以前是要转成uintptr然后计算再转回来
    func Add(ptr Pointer, len IntegerType) Pointer
```

## 参考 {#参考}

1.  [Go里面的unsafe包详解](https://cloud.tencent.com/developer/article/1192566)
2.  [pointer和uintptr](https://www.cnblogs.com/-wenli/p/12682477.html)
3.  [如何得到一个对象的内存大小](https://www.cnblogs.com/-wenli/p/12681044.html)
