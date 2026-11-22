---
title: golang-blog
date: '2023-02-17T13:01:23+08:00'
lastmod: '2023-07-11T12:27:30+08:00'
draft: false
tags:
- algorithm
- golang
- language
categories:
- code
url: /zh-cn/2023/02/17/category/golang-blog/
authors:
- yehh
---

# 阅读golang的官方blog做的一些笔记 {#阅读golang的官方blog做的一些笔记}

## [pipeline](https://go.dev/blog/pipelines) {#pipelinehttpsgodevblogpipelines}

### fan-in fan-out {#fan-in-fan-out}

> Multiple functions can read from the same channel until that channel is closed; this is called fan-out. This provides a way to distribute work amongst a group of workers to parallelize CPU use and I/O.

> A function can read from multiple inputs and proceed until all are closed by multiplexing the input channels onto a single channel that’s closed when all the inputs are closed. This is called fan-in.

example:

```go

func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}

func sq(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}

func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(in)
    c2 := sq(in)

    // Consume the merged output from c1 and c2.
    for n := range merge(c1, c2) {
        fmt.Println(n) // 4 then 9, or 9 then 4
    }
}

//merge 是一个 fan-in 的函数
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed, then calls wg.Done.
    output := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // Start a goroutine to close out once all the output goroutines are
    // done.  This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

### better pattern {#better-pattern}

> There is a pattern to our pipeline functions:

> - stages close their outbound channels when all the send operations are done.
> - stages keep receiving values from inbound channels until those channels are closed.

有时候下游stage不需要上游的所有数据，但是上游stage不知道下游需要多少，所以下游需要一个办法来阻止上游继续发送

1.  第一个就是close channel，上游使用select，如果done被关闭就变成可以receive 到zero type 再在case语句中处理不继续发送的东西然后关闭goroutine（其实这个在mit6.824 lab中还算常用）。

## Reference {#reference}

[pipeline](https://go.dev/blog/pipelines)
