---
title: 栈与队列
date: '2023-02-17T13:01:23+08:00'
lastmod: '2023-02-17T13:27:30+08:00'
draft: false
tags:
- algorithm
- jobs
- leetcode
categories:
- code
url: /zh-cn/2023/02/17/category/leetcode_day6/
authors:
- yehh
---

## 栈与队列 {#栈与队列}

#### 用栈实现队列 {#用栈实现队列}

> 用两个栈实现队列 比较简单，就是一个输入一个输出。可以理解为负负得正(先进后出\*2=先进先出)

#### 用队列实现栈 {#用队列实现栈}

> 用队列实现栈 这个的话复杂一点，我的方法会比较麻烦，是在每次push后把之前的push进这个队列，然后交换。这样的话每次push都会O(n)，时间复杂度比较高。

优化的话应该是每次pop的时候更新。

其实这样还是会pop时间复杂度O(n)，但是一般pop的次数比push少

那最简化就是，不作任何存储上的简化，只用一个队列并且一直pop再push到最后一个然后pop出去就行。

#### [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/) {#20-有效的括号httpsleetcodecnproblemsvalid-parentheses}

也比较简单，就是用栈就行了。一遍过

#### [1047. 删除字符串中的所有相邻重复项](https://leetcode.cn/problems/remove-all-adjacent-duplicates-in-string/) {#1047-删除字符串中的所有相邻重复项httpsleetcodecnproblemsremove-all-adjacent-duplicates-in-string}

也比较简单，和上面的题目差不多。

这种题主要是要辨别出使用栈

- 需要记录最近使用过的元素，并且带有删除现在的，找之前可能是最近使用过的。

#### [逆波兰表达式](https://leetcode.cn/problems/evaluate-reverse-polish-notation/) {#逆波兰表达式httpsleetcodecnproblemsevaluate-reverse-polish-notation}

也比较简单吧，可能是以前做过，但是字符串和int转换的库函数不熟悉。 趁这个题多了解了解。

学习一下别人的写法 我的:

```go
func evalRPN(tokens []string) int {
    number:=[]string{}
    for i:=0;i<len(tokens);i++{
        switch tokens[i]{
            case "+": fallthrough
            case "-": fallthrough
            case "*": fallthrough
            case "/": 
            operandA,_:=strconv.Atoi(number[len(number)-1])
            number=number[:len(number)-1]
            operandB,_:=strconv.Atoi(number[len(number)-1])
            number=number[:len(number)-1]
            number=append(number,calculate(tokens[i],operandA,operandB))
            default:
            number = append(number,tokens[i])
        }
    }
    ans,_:=strconv.Atoi(number[0])
    return ans
}

func calculate(operator string,operandA int,operandB int) string{
    var ans string
    switch operator{
        case "+":
        ans= strconv.Itoa(operandB+operandA)
        case "-":
        ans= strconv.Itoa(operandB-operandA)
        case "*":
        ans= strconv.Itoa(operandB*operandA)
        case "/":
        ans= strconv.Itoa(int(operandB/operandA))
    }
    return ans
}
```

好的写法

```go
func evalRPN(tokens []string) int {
	stack := []int{}
	for _, token := range tokens {
		val, err := strconv.Atoi(token)
		if err == nil {
			stack = append(stack, val)
		} else {   // 如果err不为nil说明不是数字
			num1, num2 := stack[len(stack)-2], stack[(len(stack))-1]
			stack = stack[:len(stack)-2]
			switch token {
			case "+":
				stack = append(stack, num1+num2)
			case "-":
				stack = append(stack, num1-num2)
			case "*":
				stack = append(stack, num1*num2)
			case "/":
				stack = append(stack, num1/num2)
			}
		}
	}
	return stack[0]
}
```

学到的写法:

- go出栈可以一次出两个
- 可以利用strconv.Atoi的err判断是否为数字
- 存在栈的最好是int

[239. 滑动窗口最大值 hard](https://leetcode.cn/problems/sliding-window-maximum/)

> 给你一个整数数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 k 个数字。滑动窗口每次只向右移动一位。

> 返回 滑动窗口中的最大值 。

```go
func maxSlidingWindow(nums []int, k int) []int {
    window:= make([]int,0,k)
    
    for i:=0;i<k;i++{
        for len(window)!=0&&window[len(window)-1]<nums[i]{
            window=window[:len(window)-1]
        }
        window=append(window,nums[i])
    }
    ans:=[]int{}
    ans=append(ans,window[0])
    for i:=0;i<len(nums)-k;i++{
        if nums[i]==window[0]{
            window=window[1:]
        }
        for len(window)!=0&&window[len(window)-1]<nums[i+k]{
            window=window[:len(window)-1]
        }
        window=append(window,nums[i+k])
        ans=append(ans,window[0])
    }
    return ans
}
```

这题还是比较难的，主要是每次移动窗口的时候保存需要的信息不知道用什么数据结构。 这题引入了单调队列，非常适合。

单调队列： 队列中的数据有序排列，这主要通过push时候的不断检查来达到。

#### [347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/) {#347-前-k-个高频元素httpsleetcodecnproblemstop-k-frequent-elements}

> 给你一个整数数组 nums 和一个整数 k ，请你返回其中出现频率前 k 高的元素。你可以按 任意顺序 返回答案。

```go
type Iheap [][2]int

func ( h Iheap ) Len() int{return len(h)}

func (h Iheap) Less(i,j int) bool{ return h[i][1]<h[j][1] }

func (h Iheap) Swap(i, j int){h[i],h[j]=h[j],h[i]}

func (h *Iheap) Pop() interface{} {
    x:= (*h)[len((*h))-1]
    *h=(*h)[:len((*h))-1]
    return x
}

func (h *Iheap) Push(i interface{}) {
    *h = append(*h,i.([2]int))
}

func topKFrequent(nums []int, k int) []int {
    record:=make(map[int]int)
    for i:=0;i<len(nums);i++{
        record[nums[i]]++
    }
    iHeap:=&Iheap{}
    heap.Init(iHeap)
    for key,value:=range(record){
        heap.Push(iHeap,[2]int{key,value})
        if iHeap.Len()>k{
            heap.Pop(iHeap)
        }
    }
    ans :=[]int{}
    for i:=0;i<k;i++{
        ans=append(ans,heap.Pop(iHeap).([2]int)[0])
    }
    return ans
}
```

这题也是比较经典的题目了:

- 考察优先队列(堆)
- golang实现(实现sort.Interface和heap.Interface)注意的是Push和Pop需要pointer receiver。
- 求最大的k个需要建立最小堆，vice versa。
