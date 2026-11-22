---
title: 二叉树（下）
date: '2023-02-28T08:01:23+08:00'
lastmod: '2023-02-28T09:27:30+08:00'
draft: false
tags:
- algorithm
- jobs
- leetcode
categories:
- code
url: /zh-cn/2023/02/28/category/leetcode_day9/
authors:
- yehh
---

# 二叉树(下) {#二叉树下}

## 二叉搜索树 {#二叉搜索树}

二叉搜索树的话最重要就是中序遍历，普通的中序遍历用stack存值，然后用pointer遍历，每次pop的时候存入然后push右子树。但是在二叉搜索树很多时候会涉及到比较，如果像上面那样的话，没有办法比较，所以一般会有一个pre指针，这个如其名就是存上一个pointer，这样的话才不会在回溯的时候丢弃父节点。

#### [98. 验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/) {#98-验证二叉搜索树httpsleetcodecnproblemsvalidate-binary-search-tree}

方法一: 统一写法的中序遍历，然后判断这个数组是不是有序。

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func isValidBST(root *TreeNode) bool {
    stack:=[]*TreeNode{root}
    list:=[]int{}
    for len(stack)!=0{
        pointer:=stack[len(stack)-1]
        stack=stack[:len(stack)-1]
        if pointer!=nil{
            if (*pointer).Right!=nil{stack=append(stack,(*pointer).Right)}
            stack=append(stack,pointer)
            stack=append(stack,nil)
            if (*pointer).Left!=nil{stack=append(stack,(*pointer).Left)}
        }else{
            pointer=stack[len(stack)-1]
            stack=stack[:len(stack)-1]
            list=append(list,(*pointer).Val)
        }
        
        
    }
    return isAscend(list)

}

func isAscend(slice []int) bool{
    for i:=1;i<len(slice);i++{
        if slice[i]<=slice[i-1]{
            return false
        }
    }
    return true
}
```

而实际上中序遍历本来就是一个判断前后的过程，可以在这个过程中加上判断逻辑。

**方法二**:

```c++
class Solution {
public:
    TreeNode* pre = NULL; // 用来记录前一个节点
    bool isValidBST(TreeNode* root) {
        if (root == NULL) return true;
        bool left = isValidBST(root->left);

        if (pre != NULL && pre->val >= root->val) return false;
        pre = root; // 记录前一个节点

        bool right = isValidBST(root->right);
        return left && right;
    }
};
```

迭代法:

```c++
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        stack<TreeNode*> st;
        TreeNode* cur = root;
        TreeNode* pre = NULL; // 记录前一个节点
        while (cur != NULL || !st.empty()) {
            if (cur != NULL) {
                st.push(cur);
                cur = cur->left;                // 左
            } else {
                cur = st.top();                 // 中
                st.pop();
                if (pre != NULL && cur->val <= pre->val)
                return false;
                pre = cur; //保存前一个访问的结点

                cur = cur->right;               // 右
            }
        }
        return true;
    }
};
```

#### [501. 二叉搜索树中的众数](https://leetcode.cn/problems/find-mode-in-binary-search-tree/) {#501-二叉搜索树中的众数httpsleetcodecnproblemsfind-mode-in-binary-search-tree}

其实也就是中序遍历BST，但是要注意的是，在处理单个的逻辑的时候，要分成pre==nil 和else的情况，分别对应首个节点和后续节点。

然后对于这题的话，要把不同情况分开操作，然后再统一判断

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
 //递归
var MaxTimes int =0
var curTimes int=0
var pre *TreeNode=nil
var ans []int= []int{}
func findMode(root *TreeNode) []int {
    findLastMode(root)
    return ans
}

func findLastMode(root *TreeNode){
    if root==nil{return }
    findLastMode((*root).Left)

    val:=(*root).Val
    if pre!=nil &&(*pre).Val==val{
        curTimes+=1
    }else{
        curTimes=1
    }
    if curTimes>MaxTimes{
            ans =[]int{}
            MaxTimes=curTimes
            ans=append(ans,val)
    }else{
            if curTimes==MaxTimes{
                MaxTimes=curTimes
                ans=append(ans,val)
            }
    }
    pre=root
    findLastMode((*root).Right)
}
```

#### [236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/) {#236-二叉树的最近公共祖先httpsleetcodecnproblemslowest-common-ancestor-of-a-binary-tree}

这题最重要就是回溯，也就是找到了一个要的点之后不断回溯寻找下一个点。

我的方法是符合直觉的中序遍历，因为这样的话找到第一个，那么现在栈里面的就是祖先。

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
 func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
     stack:=[]*TreeNode{}
     cur:=root
     oneDone:=false
     ancestor:=root
     numAncestir:=-1
     for len(stack)!=0||cur!=nil{
         if cur!=nil{
             stack=append(stack,cur)
             cur=cur.Left
         }else{
             cur=stack[len(stack)-1]
             stack=stack[:len(stack)-1]
             
             if !oneDone{
                 if cur==p||cur==q{
                     if cur==q{
                         q=p
                     }
                    oneDone=true
                    numAncestir=len(stack)
                    ancestor=cur
                 }
             }else{
                 if len(stack)==numAncestir-1{
                 ancestor=cur
                 numAncestir--
                }
                 if cur==q{
                     return ancestor
                 }
             }
             
             cur=(*cur).Right
             
         }
     }
     return ancestor
}
```

而代码随想录使用的是后序遍历的方法，这首先需要你了解一个前提: 第一次遇到左右节点都找到某个值的时候那这个节点就是就深的祖先节点。

他是在返回和处理的时候有一些特别的方法，就像上面的一旦左右都返回！nil 那这个就是target 就一路返回(回溯) 单边有那就是那边找到了一路回溯上来的。

```c++
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (root == q || root == p || root == NULL) return root;
        TreeNode* left = lowestCommonAncestor(root->left, p, q);
        TreeNode* right = lowestCommonAncestor(root->right, p, q);
        if (left != NULL && right != NULL) return root;

        if (left == NULL && right != NULL) return right;
        else if (left != NULL && right == NULL) return left;
        else  { //  (left == NULL && right == NULL)
            return NULL;
        }

    }
};
```

#### [235. 二叉搜索树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-search-tree/) {#235-二叉搜索树的最近公共祖先httpsleetcodecnproblemslowest-common-ancestor-of-a-binary-search-tree}

这题简单不少，原因是可以利用二叉树的性质，基本不需要回溯。不断自顶向下遍历再根据根节点判断（利用上一题说的那个前提，很快就能写出正确的代码）

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val   int
 *     Left  *TreeNode
 *     Right *TreeNode
 * }
 */

func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
	min,max:=minMax(p,q)
    return findAncestor(root,min,max)
}

func findAncestor(root ,min,max *TreeNode) *TreeNode{
    if root==nil{return nil}
    
    val :=(*root).Val
    maxVal:=(*max).Val
    minVal:=(*min).Val
    if maxVal==val||minVal==val||maxVal>val&&minVal<val{return root}
    if maxVal<val{return findAncestor((*root).Left,min,max) }
    return findAncestor((*root).Right,min,max)
}

func minMax(p,q *TreeNode) ( *TreeNode,*TreeNode){
    if (*p).Val<(*q).Val{return p ,q}else{
        return q,p
    }
}
```

#### [450. 删除二叉搜索树中的节点](https://leetcode.cn/problems/delete-node-in-a-bst/s) {#450-删除二叉搜索树中的节点httpsleetcodecnproblemsdelete-node-in-a-bsts}

这个题算是BST中比较难的题目了，难点在于递归还有要分析的条件很多

- 首先，题目只给了一个值，所以要先find，同时记录pre
- 找到tar 和 pre之后正式进入delete的逻辑
- 条件判断
  - 没有右孩子:把左孩子提上来（根据parent 判断如何提)
  - 有右孩子:初始化 pre和min 递归删除min然后把这个min提到tar的位置

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
var pre *TreeNode =nil
func deleteNode(root *TreeNode, key int) *TreeNode {
    pre=nil
    tar:=find(root,key)
    if tar!=root{
        deleteNodeWithPre(pre,tar)
        return root
    }else{
        return deleteNodeWithPre(pre,tar)
    }
}

func deleteNodeWithPre(pre,tar *TreeNode) *TreeNode {
    parent:=pre
    if tar==nil{return parent}
    
    //without right son
    if (*tar).Right==nil{
        if parent!=nil{
            //left or right son of parent
            if (*parent).Val>(*tar).Val{
                (*parent).Left=(*tar).Left
            }else{
                (*parent).Right=(*tar).Left
            }
            return parent
        }else{
            return (*tar).Left
        }
    }
    //with right son
    
    min:=(*tar).Right
    pre=tar
    for (*min).Left!=nil{
        pre=min
        min=(*min).Left
    }
    //delete right min
    deleteNodeWithPre(pre,min)
    (*min).Left=(*tar).Left
    (*min).Right=(*tar).Right
    if parent!=nil{
        if (*parent).Val>(*tar).Val{(*parent).Left=min}else{
            (*parent).Right=min
        }
        return parent
    }
    return min
}

func find(root *TreeNode,key int) *TreeNode{
    if root==nil{ return nil}
    if (*root).Val==key{return root}
    pre=root
    if (*root).Val<key{return find((*root).Right,key)}
    return find((*root).Left,key)
}
```

#### [669. 修剪二叉搜索树](https://leetcode.cn/problems/trim-a-binary-search-tree/) {#669-修剪二叉搜索树httpsleetcodecnproblemstrim-a-binary-search-tree}

这题也比较简单,判断然后递归自上而下就行

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func trimBST(root *TreeNode, low int, high int) *TreeNode {
    return trim(root,low,high)
}

func trim(root *TreeNode,low,high int) *TreeNode{
    if root==nil{return nil}
    if (*root).Val<low{return trim((*root).Right,low,high)}
    if (*root).Val>high{return trim((*root).Left,low,high)}
    (*root).Left=trim((*root).Left,low,high)
    (*root).Right=trim((*root).Right,low,high)
    return root
}
```

#### [538.把二叉搜索树转换为累加树](https://leetcode.cn/problems/convert-bst-to-greater-tree/) {#538把二叉搜索树转换为累加树httpsleetcodecnproblemsconvert-bst-to-greater-tree}

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
 //reversed inorder travesal
 // recursion
 var cur int = 0
func convertBST(root *TreeNode) *TreeNode {
    cur = 0
    reversedInorder(root)
    return root
}
func reversedInorder(root *TreeNode){
    if root==nil{return }
    reversedInorder((*root).Right)
    (*root).Val+=cur
    cur =(*root).Val
    reversedInorder((*root).Left)

}
```

其实就是从大到小遍历，注意要么参数和返回值构造回溯，要么使用全局变量。
