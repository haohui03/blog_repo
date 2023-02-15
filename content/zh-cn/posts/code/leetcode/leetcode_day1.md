---
title: "二分查找，双指针，滑动窗口"
date: 2023-02-05T16:01:23+08:00
lastmod: 2023-02-06T16:27:30+08:00
draft: false
tags: ["leetcode", "jobs", "algorithm"]
categories: ["code"]
slug: leetcode_day1
authors:
- "yehh"
---

## 23-02-05 刷题记录
基本是跟着[代码随想录](https://programmercarl.com/)刷的，day1刷了二分查找，滑窗还有双指针。记录一下当时的思路。

### 二分查找
二分查找突出的是一个**熟练**，针对已有的两种写法: 开区间与闭区间我选择的是闭区间的写法。抄录我的写法如下:
>第一种写法，我们定义 target 是在一个在左闭右闭的区间里，也就是[left, right] （这个很重要非常重要）。

>区间的定义这就决定了二分法的代码应该如何写，因为定义target在[left, right]区间，所以有如下两点：

>- while (left <= right) 要使用 <= ，因为left == right是有意义的，所以使用 <= 
>- if (nums[middle] > target) right 要赋值为 middle - 1，因为当前这个nums[middle]一定不是target，那么接下来要查找的左区间结束下标位置就是 middle - 1

例题都比较简单，不做展示。值得注意的是，边界条件为left==right，遇到变体的时候直接考虑这个以及之后的情况作为边界条件。

### 双指针

双指针法（快慢指针法）： 通过一个快指针和慢指针在一个for循环下完成两个for循环的工作。

- 快指针: 遍历原始数组，根据条件寻找新数组的元素。
- 慢指针: 按序更新新数组

#### 例题
> leetcode 27.
>给你一个数组 nums 和一个值 val，你需要*原地* 移除所有数值等于 val 的元素，并返回移除后数组的新长度。不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

```go
func removeElement(nums []int, val int) int {
    j :=0
    for i:=0; i<len(nums);i++{
        if nums[i]==val{
            continue
        }else{
            nums[j]=nums[i]
            j++
        }
    }
    nums=nums[:j]
    return j
}
```

类似题目：
- 26.删除排序数组中的重复项
- 283.移动零
- 844.比较含退格的字符串
- 977.有序数组的平方

### 滑动窗口

这类问题基本都和连续最短/最长子串相关，这种问题可以看成以数组中的任一个结尾满足条件的最短/最长。滑动窗口的思想就是利用连续的思想，窗口右侧在向右移动的过程中，左侧也不断移动，这样一般是O(2n)既O(n)而不是 ${O(n^2)}$。 而且一般会要在移动过程中记录下一min/max。

在使用滑窗时要注意:
- 窗口内是什么？
- 如何移动窗口的起始位置？
- 如何移动窗口的结束位置？

以leetcode 209. 为例:
>给定一个含有 n 个正整数的数组和一个正整数 target 。

>找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。


```go
func minSubArrayLen(target int, nums []int) int {
    i:=0
    j:=0
    sum:=0
    min:=len(nums)
    flag:=false
    for ;j<len(nums);j++{
        sum+=nums[j]
        if sum>=target{
            for (sum-nums[i])>= target{
                sum-=nums[i]
                i++
            }
            if j-i+1<min{
            min = j-i+1
            }
            flag=true
        }
    }
    if flag{
        return min
    }else{
        return 0
    }

}
```
- 窗口内是以窗口最右为最末时当前的最短的和超过target的子串(如果能超过)
- 移动起始位置一般是比较难的，需要根据条件移动(比如这个里的如果少了这个还能大于target那就是可以移动)
- 末尾移动还是比较简单，暂时没有遇到什么困难的。

leetcode 76.最小覆盖子串:
>给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。

```go
func minWindow(s string, t string) string {
    i:=0
    j:=0
    out:=s
    target:=0
    dict := map[byte] int{}
    targetDict := map[byte] int{}
    for _,ch:= range([]byte(t)){
        if _,ok:=targetDict[ch];ok{
            targetDict[ch]++
        }else{
            targetDict[ch]=1
        }
    }


    for ;j<len(s);j++{
        if _,ok:=dict[s[j]];ok{
            dict[s[j]]++
        }else{
            dict[s[j]]=1
        }
        if t,ok:=targetDict[s[j]];ok&&dict[s[j]]==t{
            target++
        }
        if target==len(targetDict){
            for i<=j{
                if t,ok:=targetDict[s[i]];!ok||dict[s[i]]-t>=1{
                    dict[s[i]]--
                    i++
                    }else{
                        break
                    }
                }
            if j-i+1<len(out){
            out=s[i:j+1]
            }
        }

    }
    
    if target!=len(targetDict){
        return ""
    }
    return out

}
```

注意：在每次判断左边界是否右移的时候，本来是需要比较旧dict和targetDict是否相等(每个元素都比较),但是其实只需要把满足条件的次数记录下来就可，就是需要出现的字符刚好相等的时候(target++)，这样到target等于len(targetdict)就必定可以满足条件。而且滑窗一般都会有设置flag这种操作。


相关例题：
- 904.水果成篮
- 76.最小覆盖子串