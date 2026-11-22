---
title: golang-pprof
date: '2023-11-09T13:01:23+08:00'
lastmod: '2023-11-09T12:27:30+08:00'
draft: false
tags:
- basic
- golang
- language
categories:
- code
url: /zh-cn/2023/11/09/category/golang-pprof/
authors:
- yehh
---

## 简单介绍 {#简单介绍}

> 是什么 pprof 是用于可视化和分析性能分析数据的工具

> pprof 以 profile.proto 读取分析样本的集合，并生成报告以可视化并帮助分析数据（支持文本和图形报告）

> profile.proto 是一个 Protocol Buffer v3 的描述文件，它描述了一组 callstack 和 symbolization 信息， 作用是表示统计分析的一组采样的调用栈，是很常见的 stacktrace 配置文件格式

> 支持什么使用模式

> - Report generation：报告生成
> - nteractive terminal use：交互式终端使用
> - Web interface：Web 界面

> 可以做什么
>
> - CPU Profiling：CPU 分析，按照一定的频率采集所监听的应用程序 CPU（含寄存器）的使用情况，可确定应用程序在主动消耗 CPU 周期时花费时间的位置
> - Memory Profiling：内存分析，在应用程序进行堆分配时记录堆栈跟踪，用于监视当前和历史内存使用情况，以及检查内存泄漏
> - Block Profiling：阻塞分析，记录 goroutine 阻塞等待同步（包括定时器通道）的位置
> - Mutex Profiling：互斥锁分析，报告互斥锁的竞争情况

## 参考 {#参考}

1.  [pprof实战](https://blog.wolfogre.com/posts/go-ppof-practice/)
2.  [pprof](https://howieyuen.github.io/docs/golang/tools/pprof/)
3.  [煎鱼大佬的博客](https://github.com/eddycjy/blog/blob/master/content/posts/go/tools/2018-09-15-go-tool-pprof.md)
