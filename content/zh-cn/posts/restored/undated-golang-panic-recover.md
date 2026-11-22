---
title: golang-panic-recover
draft: false
tags:
- basic
- golang
- language
categories:
- code
url: /zh-cn/1/01/01/category/golang-panic-recover/
authors:
- yehh
---

## panic和recover {#panic和recover}

问题来源于一个面试问题：

父协程能否recover子协程的panic

经过实验：是不能的

而且子协程的panic如果没有被处理是会使程序退出的

### panic {#panic}

首先看下panic的接口

```go
func panic(v any)
```

也就是panic可以传递一个接口

通过汇编可以看到其实是调用了runtime.gopanic函数于是可以通过这个进去看

### ref {#ref}

[附录 B：Goroutine 与 panic、recover 的小问题](https://golang2.eddycjy.com/posts/appendix/02-goroutine-panic/)
