---
title: 贪心(2)
date: '2023-04-23T08:01:23+08:00'
lastmod: '2023-04-23T09:27:30+08:00'
draft: false
tags:
- algorithm
- jobs
- leetcode
categories:
- code
url: /zh-cn/2023/04/23/category/leetcode_day12/
authors:
- yehh
---

## 贪心算法(2) {#贪心算法2}

[leetcode 135 分发糖果](https://leetcode.cn/problems/candy/)

这题我本来根据题目意思，三个三个来看一步一步来赋值，但是还是不对。原因在于:上一个山，你从左边上山是不知道山顶距离右边的山底多高，只有都走一遍才知道山顶相对于谁来说最高。

根据代码随想录写了一版:

```go
func candy(ratings []int) int {
    sum:=0
    record := make([]int,len(ratings))
    for i:=0;i<len(ratings);i++{
        record[i]=1
    }
    for i:=1;i<len(ratings);i++{
        if ratings[i]>ratings[i-1]{
            record[i]=record[i-1]+1
        }
    }
    i:=len(ratings)-2
    for ;i>=0;i--{
        sum+=record[i+1]
        if ratings[i]>ratings[i+1]{
            record[i]=max(record[i+1]+1,record[i])
        }
    }
    sum+=record[i+1]
    return sum

}

func max(a,b int)int{
    if a>b{
        return a
    }
    return b
}
```
