---
title: 贪心
date: '2023-03-27T08:01:23+08:00'
lastmod: '2023-03-27T09:27:30+08:00'
draft: false
tags:
- algorithm
- jobs
- leetcode
categories:
- code
url: /zh-cn/2023/03/27/category/leetcode_day11/
authors:
- yehh
---

## 贪心算法 {#贪心算法}

#### [摆动序列](https://leetcode.cn/problems/wiggle-subsequence/) {#摆动序列httpsleetcodecnproblemswiggle-subsequence}

如果连续数字之间的差严格地在正数和负数之间交替，则数字序列称为 摆动序列 。第一个差（如果存在的话）可能是正数或负数。仅有一个元素或者含两个不等元素的序列也视作摆动序列。

例如， `[1, 7, 4, 9, 2, 5]` 是一个 摆动序列 ，因为差值 `(6, -3, 5, -7, 3)` 是正负交替出现的。

相反，`[1, 4, 7, 2, 5]` 和 `[1, 7, 4, 5, 5]` 不是摆动序列，第一个序列是因为它的前两个差值都是正数，第二个序列是因为它的最后一个差值为零。 子序列 可以通过从原始序列中删除一些（也可以不删除）元素来获得，剩下的元素保持其原始顺序。

给你一个整数数组 nums ，返回 nums 中作为 摆动序列 的 最长子序列的长度 。

> 不得不说我这个实现实在是太优美

```go
func wiggleMaxLength(nums []int) int {
    if len(nums)==1{
        return 1
    }
    length:=0
    left :=0
    middle:=1
    right :=2
    for right<len(nums){
        if (nums[middle]-nums[left])*(nums[right]-nums[middle])<0{
            right++
            middle=right-1
            left=middle-1
            length++
            continue
        }
        middle++
        right++    
    }
    if nums[left]==nums[middle]{
        return length+1
    }

    return length+2


}
```

1.  充分理解题目摆动序列的思想: 判断摆动至少需要三个数字，一旦不符合就把中间的删除(具体操作是middle和right右移，left不动)
2.  贪心思想: 把连续增加的剔除掉，剩下的就都是连续的。
3.  滑动窗口思想: 窗口大小是三，每次移动右边，只有条件满足才移动左边的。这样每次窗口左边移动都是判断最左边是不是满足条件的。 而且在移动左侧的时候不要思维定势，每次加一，移动会要跳过剔除的，所以直接移动到middle左边。
