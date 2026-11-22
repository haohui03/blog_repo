---
title: kafka-interview
draft: false
tags:
- infra
- kafka
- Middleware
categories:
- code
url: /zh-cn/1/01/01/category/kafka-interview/
authors:
- yehh
---

## ISR机制 {#isr机制}

- ISR： 是指同步副本集，我们可以理解为和 leader 保持同步的所有副本的集合。
- AR： 一个分区的所有副本集合叫做 AR（ Assigned Repllicas ）
- OSR：与 leader-replica 未能保持同步的副本集叫做 OSR（ Out-Sync Relipcas ）

## 事务 {#事务}

先来介绍下事务协调者，为了解决分布式事务问题，Kafka 引入了事务协调者这个角色，负责在服务端协调整个事务。这个协调者并不是一个独立的进程，而是 Broker 进程的一部分，协调者和分区一样通过选举来保证自身的可用性。

Kafka 集群中也有一个特殊的用于记录事务日志的主题，里面记录的都是事务的日志。同时会有多个协调者的存在，每个协调者负责管理和使用事务日志中的几个分区。这样能够并行的执行事务，提高性能。

下面看下具体的流程

1、首先在开启事务的时候，生产者会给协调者发送一个开启事务的请求，协调者在事务日志中记录下事务ID;

2、然后生产者开始发送事务消息给协调者，不过需要先发送消息告知协调者在哪个主题和分区，之后就正常的发送事务消息，这些事务消息不像 RocketMQ 会保存在特殊的队列中，Kafka 未提交的事务消息和普通的消息一样，只是在消费的时候依赖客户端进行过滤。

3、消息发送完成，生产者根据自己的执行的状态对协调者进行事务的提交或者回滚；

事务的提交

1、协调者设置事务的状态为PrepareCommit，写入到事务日志中；

2、协调者在每个分区中写入事务结束的标识，然后客户端就能把之前过滤的未提交的事务消息放行给消费端进行消费了；

事务的回滚

1、协调者设置事务的状态为PrepareAbort，写入到事务日志中；

2、协调者在每个分区中写入事务回滚的标识，然后之前未提交的事务消息就能被丢弃了；

## 幂等 {#幂等}

[聊聊幂等设计](https://mp.weixin.qq.com/s?__biz=MzkyMzU5Mzk1NQ==&mid=2247506349&idx=1&sn=47b8ae3a94176440a6e5d5ad3505b4f3&source=41#wechat_redirect)

## exactly once {#exactly-once}

值得注意的是这里的exactly once只能是在流处理中才能做到，如果是在消息队列中，那么只能做到at least once。

[深入解析 Kafka Exactly Once 语义设计 & 实现](https://developer.aliyun.com/article/1172598)
