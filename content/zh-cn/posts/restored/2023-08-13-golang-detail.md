---
title: golang-detail
date: '2023-08-13T08:01:23+08:00'
lastmod: '2023-08-13T09:27:30+08:00'
draft: false
tags:
- golang
categories:
- code
url: /zh-cn/2023/08/13/category/golang-detail/
authors:
- yehh
---

### 空结构体 {#空结构体}

```go

type EST stuct{

}

func main(){
    var b EST
    var c struct{};
    fmt.Printf("b address %P size %d\n",&b,unsafe.size(b))
    fmt.Printf("c address %P size %d\n",&c,unsafe.size(c))
    //两者地址相同，空间占用都是0
}
```

1.  自定义的和struct{}是等价
2.  用法set可以用值为空结构体的map来实现
3.  可以用值为空结构体的channel
