---
title: go-redis-note
date: '2023-10-08T13:01:23+08:00'
lastmod: '2023-10-08T13:27:30+08:00'
draft: false
tags:
- infra
- Middleware
- Redis
categories:
- code
url: /zh-cn/2023/10/08/category/go-redis-note/
authors:
- yehh
---

## 记录go-redis使用过程中的一些事项 {#记录go-redis使用过程中的一些事项}

### 分布式锁(一上来就是大菜) {#分布式锁一上来就是大菜}

最近在阅读两个项目的时候都发现了分布式锁：

1.  osproxy： 分片上传时抢锁
2.  redis-tool： 本来就是教你redis操作的。。。。。。

redis的事务和原子性可以看[这里记录的一些文章](/zh-cn/2023/09/13/category/redis_basic/)

分布式锁的应用场景: [分布式锁有哪些应用场景和实现？](https://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/%E5%88%86%E5%B8%83%E5%BC%8F%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E4%B8%8E%E5%AE%9E%E6%88%9845%E8%AE%B2-%E5%AE%8C/11%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E6%9C%89%E5%93%AA%E4%BA%9B%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%E5%92%8C%E5%AE%9E%E7%8E%B0%EF%BC%9F.md)

于是记录一下go-redis中的实践：

要注意的几个点：

1.  是否可重入
2.  释放锁的方式

#### 首先来看看osproxy中是怎么做的 {#首先来看看osproxy中是怎么做的}

首先是调用接口的例子，这里是一个分片上传的例子，需要抢锁，然后判断是否已经上传过了，如果上传过了就直接返回成功，如果没有上传过就去上传，上传成功后再去释放锁，因为实际上Web 服务器在处理请求的时候是并发的，所以需要抢锁，防止多个请求同时上传同一个文件的同一个分片，这样就会导致文件上传失败。

```go
var lgRedis = new(plugins.LangGoRedis).NewRedis()
	ctx := context.Background()
	createLock := base.NewRedisLock(&ctx, lgRedis, fmt.Sprintf("multi-part-%d-%d-%s", uid, chunkNum, md5))
	if flag, err := createLock.Acquire(); err != nil || !flag {
		lgLogger.WithContext(c).Error("上传多文件抢锁失败")
		web.InternalError(c, "上传多文件抢锁失败")
		return
	}
	partInfo, err := repo.NewMultiPartInfoRepo().GetPartInfo(lgDB, uid, chunkNum, md5)
	if err != nil {
		lgLogger.WithContext(c).Error("多文件上传，查询分片信息失败")
		web.InternalError(c, "内部异常")
		return
	}
	if len(partInfo) != 0 {
		web.Success(c, "")
		return
	}
	_, _ = createLock.Release()
```

接下来看一下` RedisLock`的实现

创建RedisLock，获取锁其实就是通过Eval命令去执行lua脚本。

关键就是这个lua脚本通过获取key的值判断是否与自己线程的唯一id对应，如果对应就返回OK，如果不对应就尝试set(不存在才set)，然后通过判断返回值来判断是否获取到锁。

注意这里还可以完成可重入锁的功能。

释放锁的时候，通过判断key的值是否与自己线程的唯一id对应，如果对应就删除key，如果不对应(过期或者被别的占了)就返回0。

```go

const (
	letters     = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
	lockCommand = `if redis.call("GET", KEYS[1]) == ARGV[1] then
                      redis.call("SET", KEYS[1], ARGV[1], "PX", ARGV[2])
                      return "OK"
                   else
                       return redis.call("SET", KEYS[1], ARGV[1], "NX", "PX", ARGV[2])
                   end`
	delCommand = `if redis.call("GET", KEYS[1]) == ARGV[1] then
                      return redis.call("DEL", KEYS[1])
                  else
                      return 0
                  end`
	randomLen = 16
	// 默认超时时间，防止死锁
	tolerance       = 500 // milliseconds
	millisPerSecond = 1000
)



// NewRedisLock returns a RedisLock.
func NewRedisLock(c *context.Context, store *redis.Client, key string) *RedisLock {
	return &RedisLock{
		ctx:   c,
		store: store,
		key:   key,
		// 获取锁时，锁的值通过随机字符串生成
		// 实际上go-zero提供更加高效的随机字符串生成方式
		// 见core/stringx/random.go：Randn
		id: randomStr(randomLen),
	}
}

// Acquire acquires the lock.
// 加锁
func (rl *RedisLock) Acquire() (bool, error) {
	// 获取过期时间
	seconds := atomic.LoadUint32(&rl.seconds)
	// 默认锁过期时间为500ms，防止死锁
	res := rl.store.Eval(*rl.ctx, lockCommand, []string{rl.key}, []string{
		rl.id, strconv.Itoa(int(seconds)*millisPerSecond + tolerance),
	})
	resp, err := res.Result()
	if err == redis.Nil {
		return false, nil
	} else if err != nil {
		fmt.Printf("lock failed %s", err)
		return false, err
	}
	if resp == nil {
		return false, nil
	}

	reply, ok := (resp).(string)
	if ok && reply == "OK" {
		return true, nil
	} else {
		fmt.Printf("Unknown reply when acquiring lock for %s: %v", rl.key, resp)
		return false, nil
	}
}
// Release releases the lock.
// 释放锁
func (rl *RedisLock) Release() (bool, error) {
	res := rl.store.Eval(*rl.ctx, delCommand, []string{rl.key}, []string{rl.id})
	resp, err := res.Result()
	if err != nil {
		return false, err
	}

	reply, ok := (resp).(int64)
	if !ok {
		return false, nil
	} else {
		return reply == 1, nil
	}
}
```

#### 接下来看看redis-tool中是怎么做的 {#接下来看看redis-tool中是怎么做的}

首先看调用基本也是传一个client和key。

```go
func main() {
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6378",
		Password: "123456", // no password set
		DB:       0,        // use default DB
	})

	disLock, err := tools.NewRedisLock(client, "lock resource")
	if err != nil {
		log.Fatal(err)
	}

	succ, err := disLock.TryLock(context.Background())
	if err != nil {
		log.Println(err)
		return
	}

	if succ {
		defer disLock.Unlock(context.Background())
	}

}
```

实现起来,unlock方法其实也是基于上面的那个lua脚本(compare and delete)

```go
func (rl *RedisLock) TryLock(ctx context.Context) (bool, error) {
	succ, err := rl.Client.SetNX(ctx, rl.Key, rl.uuid, defaultExp).Result()
	if err != nil || !succ {
		return false, err
	}
	c, cancel := context.WithCancel(ctx)
	rl.cancelFunc = cancel
	rl.refresh(c)
	return succ, nil
}

func (rl *RedisLock) Unlock(ctx context.Context) (bool, error) {
	resp, err := NewTools(rl.Client).Cad(ctx, rl.Key, rl.uuid)
	if err != nil {
		return false, err
	}

	if resp {
		rl.cancelFunc()
	}
	return resp, nil
}
```
