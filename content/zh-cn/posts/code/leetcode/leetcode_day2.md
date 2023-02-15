---
title: "模拟，链表(上)"
date: 2023-02-07T13:01:23+08:00
lastmod: 2023-02-07T13:27:30+08:00
draft: false
tags: ["leetcode", "jobs", "algorithm"]
categories: ["code"]
slug: leetcode_day2
authors:
- "yehh"
---

## 模拟，链表(上) 

### 模拟

以leetcode 59.螺旋矩阵为例:

>相信很多同学刚开始做这种题目的时候，上来就是一波判断猛如虎。

>结果运行的时候各种问题，然后开始各种修修补补，最后发现改了这里那里有问题，改了那里这里又跑不起来了。


说的就是我了(捂脸)


题目：

>给你一个正整数 n ，生成一个包含 1 到 n2 所有元素，且元素按顺时针顺序螺旋排列的 n x n 正方形矩阵 matrix 。

我的answer

值得注意的是二维切片的创建与初始化
```go
//golang
func generateMatrix(n int) [][]int {
    number:=1
    out:=make([][]int,n)
    for i:=range(out){
        out[i]=make([]int,n)
    }

    terms:=(n+1)/2
    for t:=0;t<terms;t++{
        for i:=t;i<n-t;i++{
           out[t][i]=number
           number++
           
        }
        for j:=t+1;j<n-1-t;j++{
            out[j][n-1-t]=number
            number++
        }
        for i:=n-t-1;i>=t;i--{
            if number==n*n+1{
                break
            }
            out[n-1-t][i]=number
            number++
           
        }
        for j:=n-2-t;j>=t+1;j--{
            out[j][t]=number
            number++
        }

    }
    return out
}
```
和我的思路一样但是更清晰的代码：

```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> res(n, vector<int>(n, 0)); // 使用vector定义一个二维数组
        int startx = 0, starty = 0; // 定义每循环一个圈的起始位置
        int loop = n / 2; // 每个圈循环几次，例如n为奇数3，那么loop = 1 只是循环一圈，矩阵中间的值需要单独处理
        int mid = n / 2; // 矩阵中间的位置，例如：n为3， 中间的位置就是(1，1)，n为5，中间位置为(2, 2)
        int count = 1; // 用来给矩阵中每一个空格赋值
        int offset = 1; // 需要控制每一条边遍历的长度，每次循环右边界收缩一位
        int i,j;
        while (loop --) {
            i = startx;
            j = starty;

            // 下面开始的四个for就是模拟转了一圈
            // 模拟填充上行从左到右(左闭右开)
            for (j = starty; j < n - offset; j++) {
                res[startx][j] = count++;
            }
            // 模拟填充右列从上到下(左闭右开)
            for (i = startx; i < n - offset; i++) {
                res[i][j] = count++;
            }
            // 模拟填充下行从右到左(左闭右开)
            for (; j > starty; j--) {
                res[i][j] = count++;
            }
            // 模拟填充左列从下到上(左闭右开)
            for (; i > startx; i--) {
                res[i][j] = count++;
            }

            // 第二圈开始的时候，起始位置要各自加1， 例如：第一圈起始位置是(0, 0)，第二圈起始位置是(1, 1)
            startx++;
            starty++;

            // offset 控制每一圈里每一条边遍历的长度
            offset += 1;
        }

        // 如果n为奇数的话，需要单独给矩阵最中间的位置赋值
        if (n % 2) {
            res[mid][mid] = count;
        }
        return res;
    }
};
```
---

从这个可以学到的：
- 对角线遍历来等价一圈一圈。
- 起点和终点都有收缩，对应到代码就是startx,starty, offset。
- 利用左闭右开，上下和左右等价对待。


顶级c++
```c++
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        int t = 0;      // top
        int b = n-1;    // bottom
        int l = 0;      // left
        int r = n-1;    // right
        vector<vector<int>> ans(n,vector<int>(n));
        int k=1;
        while(k<=n*n){
            for(int i=l;i<=r;++i,++k) ans[t][i] = k;
            ++t;
            for(int i=t;i<=b;++i,++k) ans[i][r] = k;
            --r;
            for(int i=r;i>=l;--i,++k) ans[b][i] = k;
            --b;
            for(int i=b;i>=t;--i,++k) ans[i][l] = k;
            ++l;
        }
        return ans;
    }
};
```


### 链表

套路：

构造单链表
```cpp
//cpp
// 单链表
struct ListNode {
    int val;  // 节点上存储的元素
    ListNode *next;  // 指向下一个节点的指针
    ListNode(int x) : val(x), next(NULL) {}  // 节点的构造函数
};
```

```go
//golang
type ListNode struct {
    Val int
    Next *ListNode
}
```

删除的话一般需要知道这个节点的前一个节点添加节点的话几乎也是

所以遇到添加或者删除一般要提前一个删除，但是这样的话head没有判断，所以最后判断一次就ok

然而后面又学到一个骚方法

设置一个虚假的头节点，那么真head就是中间节点了(亏贼)

**一般要先判断链表是否没有元素，没有元素不能next，而就算有一个也能next**


leetcode 707.

>设计链表的实现。您可以选择使用单链表或双链表。单链表中的节点应该具有两个属性：val 和 next。val 是当前节点的值，next 是指向下一个节点的指针/引用。如果要使用双向链表，则还需要一个属性 prev 以指示链表中的上一个节点。假设链表中的所有节点都是 0-index 的。

>在链表类中实现这些功能：

>- get(index)：获取链表中第 index 个节点的值。如果索引无效，则返回-1。
>- addAtHead(val)：在链表的第一个元素之前添加一个值为 val 的节点。插入后，新节点将成为链表的第一个节点。
>- addAtTail(val)：将值为 val 的节点追加到链表的最后一个元素。
>- addAtIndex(index,val)：在链表中的第 index 个节点之前添加值为 val  的节点。如果 index 等于链表的长度，则该节点将附加到链表的末尾。如果 index 大于链表长度，则不会插入节点。如果index小于0，则在头部插入节点。
>- deleteAtIndex(index)：如果索引 index 有效，则删除链表中的第 index 个节点。

```go
type MyLinkedList struct {
    Length int
    Head   *Node
    Tail   *Node
}

type Node struct{
    Val int
    Next *Node
    Prev *Node
}


func Constructor() MyLinkedList {
    list := MyLinkedList{Length:0,Head:nil,Tail:nil,}
    return list
}

func (this *MyLinkedList) GetOne(index int) *Node{
    var cursor *Node
        
        if index>=this.Length/2{
            gap:= this.Length-1-index
            cursor=this.Tail
            for i:=0;i<gap;i++{
                cursor=cursor.Prev
            }
        }else{
            gap:= index-0
            cursor=this.Head
            for i:=0;i<gap;i++{
                cursor=cursor.Next
            }
        }
        return cursor
    }



func (this *MyLinkedList) Get(index int) int {
    fmt.Println(this.Length)
    if index>=this.Length{
        return -1
    }else{
        cursor := this.GetOne(index)
        return cursor.Val
    }
}


func (this *MyLinkedList) AddAtHead(val int)  {
    if this.Length == 0{
        nodeP := &Node{Val:val,Next:nil,Prev:nil}
        this.Head=nodeP
        this.Tail=nodeP
        this.Length++
        return
    }
    node := Node{Val:val,Next:this.Head,Prev:nil,}
    
    this.Head.Prev=&node
    this.Head=&node
    this.Length++
}


func (this *MyLinkedList) AddAtTail(val int)  {
    if this.Length == 0{
        nodeP := &Node{Val:val,Next:nil,Prev:nil}
        this.Head=nodeP
        this.Tail=nodeP
        this.Length++
        return 
    }
    node := Node{Val:val,Next:nil,Prev:this.Tail,}
    this.Tail.Next= &node
    this.Tail = &node
    this.Length++
}


func (this *MyLinkedList) AddAtIndex(index int, val int)  {
    if index>this.Length{
        return
    }else{
        if index==this.Length{
            this.AddAtTail(val)
        }else{
            if index>0{
                nodeP := this.GetOne(index)
                prevP := nodeP.Prev
                newNode := Node{Val:val,Next:nodeP,Prev:nodeP.Prev,}
                nodeP.Prev=&newNode
                prevP.Next=&newNode
                this.Length++
            }else{
                this.AddAtHead(val)
            }
        }
    }

    

}


func (this *MyLinkedList) DeleteAtIndex(index int)  {
    fmt.Println(this.Length)
    if index<this.Length&& index>=0{
        if index==0{
            this.Head=this.Head.Next
            this.Length--
            if this.Length!=0{
                this.Head.Prev=nil
            }

            
        }else{
            if index==this.Length-1{
                this.Tail=this.Tail.Prev
                this.Length--
                if this.Length!=0{
                    this.Tail.Next=nil
                }
            }else{
                nodeP := this.GetOne(index)
                prevP:=nodeP.Prev
                nextP:=nodeP.Next
                prevP.Next=nextP
                nextP.Prev=prevP
                this.Length--
            }
        }
    }else{
        return
    }
}


/**
 * Your MyLinkedList object will be instantiated and called as such:
 * obj := Constructor();
 * param_1 := obj.Get(index);
 * obj.AddAtHead(val);
 * obj.AddAtTail(val);
 * obj.AddAtIndex(index,val);
 * obj.DeleteAtIndex(index);
 */

```


官方实现:
```go
type MyLinkedList struct {
    head *ListNode
    size int
}

func Constructor() MyLinkedList {
    return MyLinkedList{&ListNode{}, 0}
}

func (l *MyLinkedList) Get(index int) int {
    if index < 0 || index >= l.size {
        return -1
    }
    cur := l.head
    for i := 0; i <= index; i++ {
        cur = cur.Next
    }
    return cur.Val
}

func (l *MyLinkedList) AddAtHead(val int) {
    l.AddAtIndex(0, val)
}

func (l *MyLinkedList) AddAtTail(val int) {
    l.AddAtIndex(l.size, val)
}

func (l *MyLinkedList) AddAtIndex(index, val int) {
    if index > l.size {
        return
    }
    index = max(index, 0)
    l.size++
    pred := l.head
    for i := 0; i < index; i++ {
        pred = pred.Next
    }
    toAdd := &ListNode{val, pred.Next}
    pred.Next = toAdd
}

func (l *MyLinkedList) DeleteAtIndex(index int) {
    if index < 0 || index >= l.size {
        return
    }
    l.size--
    pred := l.head
    for i := 0; i < index; i++ {
        pred = pred.Next
    }
    pred.Next = pred.Next.Next
}

func max(a, b int) int {
    if b > a {
        return b
    }
    return a
}

```

总结起来(虽然一个是单链表一个是双链表):
1. 构造函数给后面的处理带来了麻烦,没有真正用到虚拟头节点(虚拟尾节点)
2. 虚拟头节点不是你在add的时候创建一个，而是这个链表一直就有的。在这里就是

    head->index[0]->...->inde [length-1]->tail
3. 而且这样遍历的时候可以左闭右开，也就是遍历的时候可以从第一个开始遍历。