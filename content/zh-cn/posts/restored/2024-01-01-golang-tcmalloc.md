---
title: golang-tcmalloc
date: '2024-01-01T13:01:23+08:00'
lastmod: '2024-01-01T12:27:30+08:00'
draft: false
tags:
- basic
- golang
- language
categories:
- code
url: /zh-cn/2024/01/01/category/golang-tcmalloc/
authors:
- yehh
---

## TCmalloc {#tcmalloc}

![tomalloc](/images/restored/DWdNm4x8Ig-d1e1801280a5.png)

为了解决中大对象的内存申请，TCmalloc有一个全局共享内存堆

### Page {#page}

每个 Page 默认是 8KB。这里的page就像是虚拟内存的page

### Span {#span}

Span是包含了不同数量的page的单位 ![span](/images/restored/ivmjaMfAeU-9a5f3533a90a.png)

### Size Class {#size-class}

![](/images/restored/TxIGJR26fZ-81d5bc8f52ee.png)

### ThreadCache {#threadcache}

每个线程有单独的缓存 ![](/images/restored/V85EFusAJ5-4d55db101be4.png)

### 参考 {#参考}
