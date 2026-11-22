---
title: 二叉树（中）
date: '2023-02-23T08:01:23+08:00'
lastmod: '2023-02-23T09:27:30+08:00'
draft: false
tags:
- algorithm
- jobs
- leetcode
categories:
- code
url: /zh-cn/2023/02/23/category/leetcode_day8/
authors:
- yehh
---

# 二叉树(中) {#二叉树中}

#### 层序遍历 {#层序遍历}

和广度优先差不多

需要注意的是：要判断这层什么时候结束 我的方法是通过置一个nil作为结束，其实可以一趟一趟地看，先获取queue目前的长度这就是这一层的数量，然后按照这个size遍历就好。

```go
func levelOrder(root *TreeNode) [][]int {
    if root==nil{return nil}
    
    ans := make([][]int,0)
    record:=make([]*TreeNode,0)
    record = append(record , []*TreeNode{root,nil}...)
    subAns:=make([]int,0)
    for len(record)!=0{
        pointer:=record[0]
        record=record[1:]
        if pointer!=nil{
   
            subAns=append(subAns,(*pointer).Val)
            
            if (*pointer).Left!=nil{record=append(record,(*pointer).Left)}
            if (*pointer).Right!=nil{record=append(record,(*pointer).Right)}
        }else{
    
            ans=append(ans,subAns)
            if len(record)==0{
                return ans
            }else{
                subAns=make([]int,0)
                record=append(record,nil)
            }     
        }
    }
    return ans

}
```

相关题目：

- 102.二叉树的层序遍历
- 107.二叉树的层次遍历II
- 199.二叉树的右视图
- 637.二叉树的层平均值
- 429.N叉树的层序遍历
- 515.在每个树行中找最大值
- 116.填充每个节点的下一个右侧节点指针
- 117.填充每个节点的下一个右侧节点指针II
- 104.二叉树的最大深度
- 111.二叉树的最小深度

#### 反转二叉树 {#反转二叉树}

- 一开始的想法是用层序遍历然后把每一层内的值reverse，然后发现有可能有不存在左子树但是有右子树的情况。 后改成把遍历到的每一个点的左节点和右节点交换，后pass。
- 问题是为什么可以，其实可以自己画图，最下层叶子节点是否成功可以转化成一个个小一点规模的问题，最后得以解决。

#### [101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/) {#101-对称二叉树httpsleetcodecnproblemssymmetric-tree}

这题的话其实一开始想的是通过中序遍历遍历左子树，用先遍历右子树的中序遍历遍历右子树然后判断两个slice相等不相等的，但是理论证明还是比较困难。

这种题还是按照正常的流程来一个个遍历然后判断，其实递归的写法倒是不能写出来主要是要往那个方向去想。

```go
func isSymmetric(root *TreeNode) bool {
    if root == nil{return true}
    return ifSymmetric((*root).Left,(*root).Right)
}

func ifSymmetric(left,right *TreeNode) bool{
    if left==nil&&right!=nil || left!=nil&&right==nil{
        return false
    }
    if left==nil&&right==nil{
        return true
    }
    if (*left).Val!=(*right).Val{
        return false
    }
    outside:=ifSymmetric((*left).Left,(*right).Right)
    inside:=ifSymmetric((*left).Right,(*right).Left)
    return outside&&inside
}
```

#### [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/) {#104-二叉树的最大深度httpsleetcodecnproblemsmaximum-depth-of-binary-tree}

这题的话，一下就能感觉出来可以层序遍历这比较符合直觉。 但是，还是需要学习树的深度和高度怎么求

深度： 一般是前序遍历

高度： 通过后序遍历，主要的代码就是：

```c++
class solution {
public:
    int getdepth(treenode* node) {
        if (node == NULL) return 0;
        int leftdepth = getdepth(node->left);       // 左
        int rightdepth = getdepth(node->right);     // 右
        int depth = 1 + max(leftdepth, rightdepth); // 中
        return depth;
    }
    int maxdepth(treenode* root) {
        return getdepth(root);
    }
};
```

求深度的话一般来说是会和层序遍历有关的，所以要多想层序遍历。

#### [222. 完全二叉树的节点个数](https://leetcode.cn/problems/count-complete-tree-nodes/) {#222-完全二叉树的节点个数httpsleetcodecnproblemscount-complete-tree-nodes}

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func countNodes(root *TreeNode) int {
    if root==nil{return 0}
    h:=0
    for pointer:=root;pointer!=nil;{
        h++
        pointer=(*pointer).Left
    }
    ans,_:= getNodes(root,h)
    return ans
    
}

func getNodes(root *TreeNode, height int) (int,int){
    if root==nil{return 0,0}
    h:=0
    pointer:=root
    for pointer!=nil{
        h++
        pointer=(*pointer).Left
    }
    if h<height{return getCompleteNodes(h),h}
    
    right,rightH:=getNodes((*root).Right,height-1)
    left:=0
    if rightH>height-2{left=getCompleteNodes(height-1)}else{
        left,_=getNodes((*root).Left,height-1)
    }
    return left+right+1,h
}

func getCompleteNodes(height int) int{
    return int(math.Pow(2,float64(height))-1)
}
```

这个题的话，自己的这个方法我感觉还是很不错的。 利用了完全二叉树的性质，从右边开始算高度，如果高度匹配，说明左子树一定是满二叉树，如果高度不匹配那么自己肯定是满二叉树。利用这个性质就能很快写出。

#### [110. 平衡二叉树](https://leetcode.cn/problems/balanced-binary-tree/) {#110-平衡二叉树httpsleetcodecnproblemsbalanced-binary-tree}

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func isBalanced(root *TreeNode) bool {
    _,balance:=getHeightNBalance(root)
    return balance
}

func getHeightNBalance(root *TreeNode) (int,bool){
    if root==nil{return 0,true}
    leftHeight,leftBalance:=getHeightNBalance((*root).Left)
    rightHeight,rightBalance:=getHeightNBalance((*root).Right)
    if !(leftBalance&&rightBalance){return 0,false}
    balance:=true
    if int(math.Abs(float64 (leftHeight-rightHeight)))>1 {balance=false}
    return max(leftHeight,rightHeight)+1,balance
}

func max(a,b int) int{
    if a>b{return a}
    return b
}
```

这题不难，注意求高度，后序遍历注意一下递归的三条件就ok
