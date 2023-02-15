---
title: "哈希表"
date: 2023-02-12T13:01:23+08:00
lastmod: 2023-02-12T13:27:30+08:00
draft: false
tags: ["leetcode", "jobs", "algorithm"]
categories: ["code"]
slug: leetcode_day3
authors:
- "yehh"
---

## 哈希表

哈希碰撞的两个解决方法:

1. 拉链法
![](https://img-blog.csdnimg.cn/20210104235015226.png)
2. 线性探测
![](https://img-blog.csdnimg.cn/20210104235109950.png)


常见的哈希表:
1. 数组

2. set

![23-02-12-set.JPG](https://s2.loli.net/2023/02/12/y6YRtcNObwFrE3j.jpg)

3. map
![xiaz.png](https://s2.loli.net/2023/02/13/LYFdkshWVyco84P.png)

有关红黑树，见[红黑树深入剖析及Java实现](https://tech.meituan.com/2016/12/02/redblack-tree.html)

RBTree的定义
RBTree的定义如下:

- 任何一个节点都有颜色，黑色或者红色。
- 根节点是黑色的。
- 父子节点之间不能出现两个连续的红节点。
- 任何一个节点向下遍历到其子孙的叶子节点，所经过的黑节点个数必须相等。
- 空节点被认为是黑色的。

[golang 字典的底层](https://juejin.cn/post/6972535873971847204)

### 例题
#### [leetcode 242 easy](https://leetcode.cn/problems/valid-anagram/submissions/)

>给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的字母异位词。

>注意：若 s 和 t 中每个字符出现的次数都相同，则称 s 和 t 互为字母异位词。
my version
```go
func isAnagram(s string, t string) bool {
    sMap:= make(map[rune]int)
    tMap := make(map[rune]int)
    for _,i:=range(s){
        sMap[i]++
    }
    for _,i:=range(t){
        tMap[i]++
    }
    if len(sMap)!=len(tMap){
        return false
    }
    for k,v:=range(sMap){
        if v==tMap[k]{
            continue
        }else{
            return false
        }
    }
    return true
}
```

better version 

```go 
func isAnagram(s string, t string) bool {
    record := [26]int{}
    
    for _, r := range s {
        record[r-rune('a')]++
    }
    for _, r := range t {
        record[r-rune('a')]--
    }

    return record == [26]int{}   // record数组如果有的元素不为零0，说明字符串s和t 一定是谁多了字符或者谁少了字符。
}
```
把数组看出hash map，同时只用一个hash表就完成比较，最后的判断比较零也很巧妙(因为数组的每个元素都是可以比较的所以可以直接用等于。)

相关题目:
- 383.赎金信
- 49.字母异位词分组
- 438.找到字符串中所有字母异位词

#### [383 赎金信.](https://leetcode.cn/problems/ransom-note/) 
也是需要优化，不能直接用map(要维护一个哈希表或者红黑树),面对这种单词的出现的哈希，要注意用26字母的数组来代替。

>给你两个字符串：ransomNote 和 magazine ，判断 ransomNote 能不能由 magazine 里面的字符构成。

>如果可以，返回 true ；否则返回 false 。

>magazine 中的每个字符只能在 ransomNote 中使用一次。

```go

func canConstruct(ransomNote string, magazine string) bool {
    record := make([]int, 26)
    for _, v := range magazine {   // 通过recode数据记录 magazine里各个字符出现次数
        record[v-'a']++
    }
    for _, v := range ransomNote { // 遍历ransomNote，在record里对应的字符个数做--操作
        record[v-'a']--
        if record[v-'a'] < 0 {     // 如果小于零说明ransomNote里出现的字符，magazine没有
            return false
        }
    }
    return true
}
```

从优秀的解法中可以学到:
- 利用固定数组代替map(set)
- 首先把有的用数组记录下来, 然后遍历需要的然后一个个减，这样的话小于零就是不满足。我的是反过来，这样的话要记录目标==0，然后记录次数与length相比。这样是比较麻烦的。


#### [leetcode 49.字母异位词分组](https://leetcode.cn/problems/group-anagrams/)
>给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。

>字母异位词 是由重新排列源单词的字母得到的一个新单词，所有源单词中的字母通常恰好只用一次。

这题也是有点难

```go
func groupAnagrams(strs []string) [][]string {
    record := map[[26]int][]string{}
    for _,v:=range(strs){
        cnt:= [26]int{}
        for _,i:=range(v){
            cnt[i-'a']++
        }
        record[cnt]=append(record[cnt],v)
    }
    ans:= make([][]string,0,len(record))
    for _,v :=range(record){
        ans=append(ans,v)
    }
    return ans

}
```
总结:
- 这个哈希我其实有想到，但是对于key和value的数据结构还是不是很了解，key: [26]int value []string, 所以没有写出来

#### [leetcode 438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

>给定两个字符串 s 和 p，找到 s 中所有 p 的 异位词 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

>异位词 指由相同字母重排列形成的字符串（包括相同的字符串）。

```go
func findAnagrams(s string, p string) []int {
    ans:=[]int{}
    target:= [26]int{}
    for _,v:=range(p){
        target[v-'a']++
    }
    source:= [26]int{}
    left:=0

    right:=0
    for right!=len(s){

        source[s[right]-'a']++
        if right-left+1==len(p){
            break
        }
        right++
    }
    for {
            if source==target{
                ans=append(ans,left)
            }
            right++
            if right<len(s){
                source[s[right]-'a']++
                source[s[left]-'a']--
                left++
            }else{
                return ans
            }
        }
    return ans
    
}
```

总结:
- 第一次错误: 写了一步特殊的 source[s[left]-'a']这样不具有普遍性,应该把添加节点的操作放进for 用right来加入。
- 第二次错误: 在第一个for 的if 里面处理第二个for，没有考虑第一个*for*退出的时候也需要处理第二个for，正确的做法是：第一个*for*+在第二个*for*中统一考虑第一个for结束的两种情况。




#### [leetcode 15. 三数之和](https://leetcode.cn/problems/3sum/)

>给你一个整数数组 nums ，判断是否存在三元组 [nums[i], nums[j], nums[k]] 满足 i != j、i != k 且 j != k ，同时还满足 nums[i] + nums[j] + nums[k] == 0 。请

>你返回所有和为 0 且不重复的三元组。

>注意：答案中不可以包含重复的三元组。

- 确实很难，如果用2sum的哈希表方法去重逻辑非常复杂，就是不会去重。
- 注意到是可以排序的，不管用什么方法都要排序。可见在去重中，排序的很重要的手段。

```go
func threeSum(nums []int) [][]int {
    sort.Ints(nums)
    fmt.Println(nums)
    ans:= [][]int{}
    for i:=0;i<len(nums)-2;i++{
        if i>0 && nums[i]==nums[i-1]{
            continue
        }

        j:=i+1
        k:=len(nums)-1
        for k>j{
            if nums[i]+nums[j]+nums[k]>0{
                k--
                continue
            }else{
                if nums[i]+nums[j]+nums[k]<0{
                    j++
                    continue
                }else{
                    ans=append(ans,[]int{nums[i],nums[j],nums[k]})
                    for {
                        j++
                        if k>j&&nums[j]==nums[j-1]{
                            continue
                        }else{
                            break
                        }
                    }
                    for {
                        k--
                        if k>j&&nums[k]==nums[k+1]{
                            continue
                        }else{
                            break
                        }
                    }
                }
            }
        }
    }
    return ans
}
```
四数之和就是建立在三数之和的基础上的了。
还要注意的是，开头可以剪枝，就是排序之后判断第一个。