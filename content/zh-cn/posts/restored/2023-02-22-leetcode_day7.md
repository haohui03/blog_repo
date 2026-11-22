---
title: 二叉树（上）
date: '2023-02-22T13:01:23+08:00'
lastmod: '2023-02-22T13:27:30+08:00'
draft: false
tags:
- algorithm
- jobs
- leetcode
categories:
- code
url: /zh-cn/2023/02/22/category/leetcode_day7/
authors:
- yehh
---

# 二叉树(上) {#二叉树上}

### 二叉树的递归遍历: {#二叉树的递归遍历}

> 这里帮助大家确定下来递归算法的三个要素。每次写递归，都按照这三要素来写，可以保证大家写出正确的递归算法！

> 1.  确定递归函数的参数和返回值： 确定哪些参数是递归的过程中需要处理的，那么就在递归函数里加上这个参数， 并且还要明确每次递归的返回值是什么进而确定递归函数的返回类型。

> 2.  确定终止条件： 写完了递归算法, 运行的时候，经常会遇到栈溢出的错误，就是没写终止条件或者终止条件写的不对，操作系统也是用一个栈的结构来保存每一层递归的信息，如果递归没有终止，操作系统的内存栈必然就会溢出。

> 3.  确定单层递归的逻辑： 确定每一层递归需要处理的信息。在这里也就会重复调用自己来实现递归的过程。

理解下来就是:

- 终止条件
- 返回(其实也是循环入口)
- 单层的处理

#### 前序遍历 {#前序遍历}

递归法

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func preorderTraversal(root *TreeNode) []int {
    if root == nil{
        return nil
    }
    left:=preorderTraversal((*root).Left)
    right:=preorderTraversal((*root).Right)
    ans:=make([]int,len(left)+len(right)+1)
    ans[0]=(*root).Val
    copy(ans[1:len(left)+1],left)
    copy(ans[len(left)+1:],right)
    

    return ans
}
```

迭代法:

```go
func preorderTraversal(root *TreeNode) []int {
    if root==nil{
        return nil
    }
    stack:=make([]*TreeNode,0)
    stack=append(stack,root)
    ans:=make([]int,0)
    for len(stack)!=0{
        pointer:=*stack[len(stack)-1]
        ans=append(ans,pointer.Val)
        stack=stack[:len(stack)-1]
        if pointer.Right!=nil{ stack=append(stack,pointer.Right)}
        if pointer.Left!=nil{ stack= append(stack,pointer.Left)}
    }
    return ans
}
```

对于迭代法来说，重要的是找到如何入栈和出栈。

- 一开始想的是不断把遍历到的节点入栈，但其实只要把右节点和左节点先后入就可以了。这样每次遇到新的节点都是一样的处理逻辑，而不用考虑是新节点还是回溯的节点。
- 代码随想录里面的理解是:插入节点的值（插入ans）和处理节点（记录左右子节点的地址）都可以一起完成。

#### 中序遍历: {#中序遍历}

- 这个就很不一样了，因为他就是不符合上面说的插入值和处理节点同时完成。
- 也像我说的，新遇到一个节点还是回溯遇到节点是不一样的，所以我们要把他的这两步分开处理,而且可能回溯遇到两次同一个节点。

具体来说，也就是一直入栈的时候处理，出栈的时候记录值，并且这个时候把这个根出栈了就不会再遇到了。

```go
func inorderTraversal(root *TreeNode) []int {
    if root==nil{  return nil}
    stack:=make([]*TreeNode,0)
    ans:=make([]int,0)
    pointer:=root
    for len(stack)!=0||pointer!=nil{
        if pointer!=nil{
            stack = append(stack,pointer)
            pointer = (*pointer).Left
        }else{
            
            pointer=stack[len(stack)-1]
            ans=append(ans,pointer.Val)
            fmt.Println((*pointer).Val)
            
            pointer=(*pointer).Right
            
            stack=stack[:len(stack)-1]
            
        }
    }
    return ans 
}
```

#### 后序遍历 {#后序遍历}

后序遍历的话符合我们之前说的要求，所以第一次遇到的时候就可以处理并记录。 但是要注意记录的时候本来应该把它放在最后面的，但是具体实现起来不好从后向前append，所以采用从前向后然后reverse

```go
func postorderTraversal(root *TreeNode) []int {
    if root==nil{
        return nil
    }
    record:=make([]*TreeNode,0)
    record=append(record,root)
    ans:=make([]int,0)
    for len(record)!=0{
        pointer:=record[len(record)-1]
        record= record[:len(record)-1]
        ans=append(ans,(*pointer).Val)
        if (*pointer).Left!=nil{ record=append(record,(*pointer).Left)}
        if (*pointer).Right!=nil{ record=append(record,(*pointer).Right)}
        
    }
    reverse(ans)
    return ans

}

func reverse(in []int){
    i:=0
    j:=len(in)-1
    for i<j{
        in[i],in[j]=in[j],in[i]
        i++
        j--
    }
}
```

#### 三种遍历的统一写法: {#三种遍历的统一写法}

- 核心思想是标记法: 实际上人在遍历的回溯的时候会下意识的辨别出这个节点是新的还是已经见过的。
- 所以在stack中我们就是要把上面的两种操作分开，没有标记的是新节点，前面有个null标记的就是只存值的节点。(和转义的思想差不多)

浅浅模仿写了个中序遍历的，不同的遍历只需要在遇到新节点后改变左中右的插入顺序。

```go
func inorderTraversal(root *TreeNode) []int {
    if root==nil{ return nil}
    record:=make([]*TreeNode,0)
    record = append(record,root)
    ans:=make([]int,0)
    for len(record)!=0{
        pointer:=record[len(record)-1]
        if pointer!=nil{
            record=record[:len(record)-1]
            if (*pointer).Right!=nil{ record = append(record,(*pointer).Right)}
            

            record= append(record,pointer)
            record = append(record,nil)

            if (*pointer).Left!=nil{record=append(record,(*pointer).Left)}
            
        }else{
            pointer=record[len(record)-2]
            ans=append(ans,(*pointer).Val)
            record=record[:len(record)-2]
        }
    }
    return ans
}
```
