---
title: interview-八股
date: '2023-09-17T13:01:23+08:00'
lastmod: '2023-09-17T13:27:30+08:00'
draft: false
tags:
- interview
- network
categories:
- code
url: /zh-cn/2023/09/17/category/interview-八股/
authors:
- yehh
---

## 汇总 {#汇总}

1.  [很多 必看](https://github.com/wolverinn/Waking-Up/tree/master)
2.  [这个很也顶](https://mcg_dev.gitee.io/java-interview/)

### network {#network}

#### 工具： {#工具}

1.  tcpdump
2.  Wireshark

#### 具体一些没背八股前不清晰的问题： {#具体一些没背八股前不清晰的问题}

1.  Cookie&session [你真的了解cookie和session吗](https://www.cnblogs.com/ityouknow/p/10856177.html)

2.  鉴权&Token [鉴权界的后起之秀：Token鉴权方案](https://juejin.cn/post/7017821497980878878) [鉴权必须了解的5个兄弟：cookie、session、token、jwt、单点登录](https://zhuanlan.zhihu.com/p/395273289)

3.  数据从网卡到应用的过程 [计算机网络——数据从网卡到应用的过程](https://blog.csdn.net/JMW1407/article/details/108636657) [网络数据是如何传递给进程的](https://blog.csdn.net/u012796139/article/details/106184401) [很顶](https://zhuanlan.zhihu.com/p/256428917) [个人博客 也很厉害](https://morven.life/posts/networking-1-pkg-snd-rcv/)

4.  ARP协议（不是很记得了） [彻底搞懂系列之：ARP协议](https://zhuanlan.zhihu.com/p/370507243)

5.  http的几个版本 [HTTP 版本区别](https://cloud.tencent.com/developer/article/1857670)

6.  跨域问题 [面试官问我CORS跨域，我直接一套操作斩杀！](https://juejin.cn/post/6983852288091619342)

7.  QUIC [QUIC协议](https://www.infoq.cn/article/lddlsa5f21sty04li3hp)

8.  [QUIC](https://www.cnblogs.com/harrypotterjackson/p/15977660.html)

9.  [tls](https://www.cnblogs.com/sewain/p/14250884.html#ssltls)

10. [http-\>https](https://blog.csdn.net/ffffffff8/article/details/120523524)

11. [汇总](https://testerhome.com/topics/32659)

12. [DNS](https://www.kaspersky.com.cn/resource-center/definitions/dns)

13. [什么是 DNS 劫持？](https://www.kaspersky.com.cn/resource-center/definitions/what-is-dns-hijacking)

14. [DNS劫持的工作原理及应对方法](https://zhuanlan.zhihu.com/p/348711884)

#### 实战 {#实战}

[关于三次握手与四次挥手面试官想考我们什么？— 不看后悔系列](https://juejin.cn/post/6844903834708344840)

#### Reference {#reference}

[帅地玩编程](https://www.iamshuaidi.com/673.html)

小林coding

[字节跳动面试之—计算机网络](https://blog.csdn.net/fighting_tl/article/details/104697207)

[字节计算机网络面试](https://zhuanlan.zhihu.com/p/422455960)

[前端计算机网络面经](https://juejin.cn/post/6939691851746279437)

[计算机网络面经总结](https://segmentfault.com/a/1190000038526729)

### redis {#redis}

#### redis map 扩容 {#redis-map-扩容}

由于redis的map扩容和golang的map扩容其实比较像，go中的map可以看我golang/structure里面的内容

#### redis的数据结构 {#redis的数据结构}

1.  [跳表](https://www.jianshu.com/p/9d8296562806)

#### 链接 {#链接}

1.  [Redis面试题](https://www.nowcoder.com/discuss/353159155588472832)
2.  [Redis 面试题](https://www.cnblogs.com/hechengQAQ/p/17269320.html)
3.  [持久化的内容](https://zhuanlan.zhihu.com/p/340082703)

### Mysql {#mysql}

1.  [存储过程](https://www.cnblogs.com/geaozhang/p/6797357.html)
2.  [也是存储过程](https://segmentfault.com/a/1190000039248897)
3.  [视图](https://cloud.tencent.com/developer/article/1800718)
4.  [窗口函数](https://zhuanlan.zhihu.com/p/132477535)
5.  [分库分表](https://zhuanlan.zhihu.com/p/425398194)
6.  [汇总](https://zhuanlan.zhihu.com/p/164519371)
7.  [主从](https://zhuanlan.zhihu.com/p/164518315)
8.  [主从](https://blog.csdn.net/qq_41786285/article/details/109304126)
9.  [缓存一致性](https://ost.51cto.com/posts/13547)
10. [为什么用rc隔离级别](https://zhuanlan.zhihu.com/p/59061106)

#### binlog {#binlog}

binlog是server层的功能

1.  [腾讯工程师带你深入解析 MySQL binlog](https://zhuanlan.zhihu.com/p/33504555)
2.  [MySQL的binlog日志](https://www.cnblogs.com/martinzhang/p/3454358.html)
3.  [带你了解 MySQL Binlog 不为人知的秘密](https://www.cnblogs.com/rickiyang/p/13841811.html)
4.  [binlog面试](https://m.haicoder.net/note/mysql-interview/mysql-interview-mysql-binlog.html)

#### lock {#lock}

1.  [各种锁 挺不错的](https://zhuanlan.zhihu.com/p/437594231)
2.  [讲解了意向锁，讲得很好](https://juejin.cn/post/6844903666332368909)

#### explain-key_len {#explain-key_len}

1.  [mysql explain中key_len的计算方法](https://blog.csdn.net/fdipzone/article/details/55804684)
2.  [key_len长度计算](https://www.cnblogs.com/xuanzhi201111/p/4554769.html)

#### storage {#storage}

1.  [CMU-pdf1](https://15445.courses.cs.cmu.edu/fall2022/slides/03-storage1.pdf)
2.  [CMU-pdf2](https://15445.courses.cs.cmu.edu/fall2022/slides/04-storage2.pdf)
3.  [MySQL的索引为什么使用B+树而不使用跳表？](https://www.51cto.com/article/706701.html)

### 系统设计 {#系统设计}

1.  [十亿用户的系统设计](https://www.cnblogs.com/javastack/p/17853746.html)
2.  [十亿用户的系统设计](https://posts.careerengine.us/p/621a3d089d168d07394ec913)
3.  [限流](https://juejin.cn/post/6967742960540581918)

### os {#os}

1.  [孤儿进程与僵尸进程](https://www.cnblogs.com/Anker/p/3271773.html)
2.  DMA [dma](https://www.cnblogs.com/wkfvawl/p/10958751.html)
3.  [几种cache](https://forum.huawei.com/enterprise/zh/thread/580900453703630848)
4.  [字节终面：CPU 是如何读写内存的？](https://zhuanlan.zhihu.com/p/522038503)
5.  [Linux 内核的 4 大 IO 调度算法](https://cloud.tencent.com/developer/article/1615744)
6.  [父子进程共享什么](https://www.cnblogs.com/-colin/p/7990053.html)

### 大杂烩 {#大杂烩}

[redis zookeeper](https://zhuanlan.zhihu.com/p/662062549)

### 算法 {#算法}

[100 算法](https://juejin.cn/post/6844904089650724872)
