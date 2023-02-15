---
title: "链表(下)"
date: 2023-02-09T13:01:23+08:00
lastmod: 2023-02-09T13:27:30+08:00
draft: false
tags: ["leetcode", "jobs", "algorithm"]
categories: ["code"]
slug: leetcode_day3
authors:
- "yehh"
---

## 链表(下)

leetcode 206. 简单一遍过
>给你单链表的头节点 head ，请你反转链表，并返回反转后的链表。

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
    if head==nil || head.Next==nil{
        return head
    }
    var first *ListNode
    second := head
    third := second.Next

    for third!=nil{
        second.Next=first
        first=second
        second=third
        third=second.Next
    }
    second.Next=first

    return second
    
}
```


[leetcode 24](https://leetcode.cn/problems/swap-nodes-in-pairs/)
>给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。

```go

/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func swapPairs(head *ListNode) *ListNode {
    dommy := &ListNode{Val:0,Next:head}
    first:= dommy
    second := head
    for second!=nil && second.Next!=nil{
        first.Next=second.Next
        second.Next=first.Next.Next
        first.Next.Next=second

        first=second
        second=second.Next

    }
    return dommy.Next
    
}
```

不算难，用虚拟头节点模拟就行。


[leetcode 19.](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)
>给你一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    
    nodeB:=head
    dommy := &ListNode{Val:0,Next:head}
    nodeA:=dommy
    for i :=0;i<n;i++{
        nodeB=nodeB.Next
    }
    for nodeB!=nil{
        nodeB=nodeB.Next
        nodeA=nodeA.Next
    }
    nodeA.Next=nodeA.Next.Next
    return dommy.Next
}
```


[leetcode 面试题 02.07](https://leetcode.cn/problems/intersection-of-two-linked-lists-lcci/)

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func getIntersectionNode(headA, headB *ListNode) *ListNode {
    if headA==nil || headB ==nil{
        return nil
    }

    pa ,pb:= headA,headB

    for pa!=pb{
        if pa!=nil{
            pa=pa.Next
        }else{
            pa = headB
        }

        if pb!=nil{
            pb=pb.Next
        }else{
            pb = headA
        }
    }
    return pa
}
```
本来的想法也是，双指针一起移动，但是两个的长度不一样，可能移动的过程中并不能相遇到一起，但是利用等差数列求和的思想，补齐就可以。详情见[题解](https://leetcode.cn/problems/intersection-of-two-linked-lists-lcci/solution/lian-biao-xiang-jiao-by-leetcode-solutio-2kne/)


[leetcode 142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)

*Crazy 的一题*

>给定一个链表的头节点  head ，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。

```go

/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func detectCycle(head *ListNode) *ListNode {
    slow, fast := head,head

    for {
        for i:=0;i<2;i++{
            if fast!=nil{
            fast=fast.Next
            }else{
                return nil
            }
        }
        slow= slow.Next
        if slow==fast{
            break
        }
    }

    cur:= head
    for cur!=slow{
        cur=cur.Next
        slow=slow.Next
    }
    return cur
}
```

建议直接看[神级题解](https://leetcode.cn/problems/linked-list-cycle-ii/solution/linked-list-cycle-ii-kuai-man-zhi-zhen-shuang-zhi-/)

看完题解之后的感受是: 
- 这类链表题目一般都是使用双指针法解决的，例如寻找距离尾部第 K 个节点、寻找环入口、寻找公共尾部入口等。
- 要对数量关系进行建模理解

