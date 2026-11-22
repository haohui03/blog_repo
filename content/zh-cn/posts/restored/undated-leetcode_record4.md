---
title: leetcode_record4-随机算法
draft: false
tags:
- algorithm
- jobs
- leetcode
categories:
- code
url: /zh-cn/1/01/01/category/leetcode_record4/
authors:
- yehh
---

记几个网站：

- [1](https://github.com/hongdayu/Algorithm_Interview_Notes-Chinese/tree/master/C-%E7%AE%97%E6%B3%95)
- [2](https://github.com/hongdayu/Algorithm_Interview_Notes-Chinese/tree/master/D-%E7%AC%94%E8%AF%95%E9%9D%A2%E7%BB%8F)

## 随机算法 {#随机算法}

### 打乱（Shuffle） {#打乱shuffle}

#### Fisher–Yates shuffle {#fisheryates-shuffle}

```text
-- To shuffle an array a of n elements (indices 0..n-1):
for i from n−1 downto 1 do
     j ← random integer such that 0 ≤ j ≤ i
     exchange a[j] and a[i]
```

　　第一步 即从0到N-1个元素中随机选择一个与第N-1个替换

　　第二步 从0到N-2个元素中随机选择一个与第N-2个替换

　　第k步 从0到N-k个元素中随机选择一个与第N-K个替换

这样得到的结果就是一个随机打乱的数组，每个元素出现在某个位置上的概率就是1/N.

### 数位dp {#数位dp}

#### [统计整数数目](https://leetcode.cn/problems/count-of-integers/description/?envType=daily-question&envId=2024-01-16) {#统计整数数目httpsleetcodecnproblemscount-of-integersdescriptionenvtypedaily-questionenvid2024-01-16}

> 大意就是找到两个数之间，位数和在某个区间的个数

这其实会用到很多(如果是3位数，他们位数和为x的个数是多少)这种，而很多都是重复的，又是可以通过前面的推算出来的，所以很容易想到dp。

但是没写过的话，写起来还是比较麻烦，所以记一下模板。

这里的话，用全局变量加记忆化搜索会比较好

这里的f代表从左往右数的第几位而且都是任选的可以满足(min_sum,max_sum)条件的（注意初始化：基本都要+1，而且初始化为-1标识未遇见过）

dfs：

1.  入口：包括特定的num，当前位置，当前位数和，是否限制
2.  递归：取决于是否限制，限制的话就要把限制传递下去(这里也有trick，就是用&来传递)并且不能更新f，不限制的话就要更新f
3.  出口：当当前位置超过的时候就返回

字符串减一的操作也可以记一记

```go
func count(num1 string, num2 string, min_sum int, max_sum int) int {
	const mod = 1e9 + 7
	f := [23][220]int{}
	for i := range f {
		for j := range f[i] {
			f[i][j] = -1
		}
	}
	num := num2
	var dfs func(int, int, bool) int
	dfs = func(pos, s int, limit bool) int {
		if pos >= len(num) {
			if s >= min_sum && s <= max_sum {
				return 1
			}
			return 0
		}
		if !limit && f[pos][s] != -1 {
			return f[pos][s]
		}
		var ans int
		up := 9
		if limit {
			up = int(num[pos] - '0')
		}
		for i := 0; i <= up; i++ {
			ans = (ans + dfs(pos+1, s+i, limit && i == up)) % mod
		}
		if !limit {
			f[pos][s] = ans
		}
		return ans
	}
	a := dfs(0, 0, true)
	t := []byte(num1)
	for i := len(t) - 1; i >= 0; i-- {
		if t[i] != '0' {
			t[i]--
			break
		}
		t[i] = '9'
	}
	num = string(t)
	// f = [23][220]int{} 其实是不用这一条的
	for i := range f {
		for j := range f[i] {
			f[i][j] = -1
		}
	}
	b := dfs(0, 0, true)
	return (a - b + mod) % mod
}
```
