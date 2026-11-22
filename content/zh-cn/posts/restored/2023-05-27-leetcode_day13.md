---
title: 动态规划
date: '2023-05-27T08:01:23+08:00'
lastmod: '2023-05-27T09:27:30+08:00'
draft: false
tags:
- algorithm
- jobs
- leetcode
categories:
- code
url: /zh-cn/2023/05/27/category/leetcode_day13/
authors:
- yehh
---

# 发现好久没有写刷题记录了(博客也好久没有更新了) {#发现好久没有写刷题记录了博客也好久没有更新了}

[718. 最长重复子数组](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/)

> `输入：nums1 = [1,2,3,2,1], nums2 = [3,2,1,4,7]`
>
> `输出：3`
>
> `解释：长度最长的公共子数组是 [3,2,1] 。`

虽然说一开始就知道用DP，但是死活想不出来该怎么做。 仔细琢磨了一下发现应该把nums1\[i\]和nums\[j\]的匹配结果保存起来。但是后面的使用没有优化好，我这里是先用二维数组保存所有的匹配情况，再根据连续的定义也就是i，j都取一个数然后对角线连续出现一这样。但是我这样的话复杂度是O(2\*M\*N)

代码如下:

```go
class Solution {
public:
    int findLength(vector<int>& nums1, vector<int>& nums2) {
        vector<vector<int>> pack(nums1.size());
        for(int i =0 ; i<nums1.size();i++){
            pack[i]=vector<int>(nums2.size(),0);
        }
        for(int i =0;i<nums1.size();i++){
            for(int j =0 ;j<nums2.size();j++){
                if(nums1[i]==nums2[j]){
                    pack[i][j]=1;
                }
                cout<<pack[i][j]<<" ";
            }
            cout<<endl;
        }
        int record =0;
        int max = 0;
        int j =0;
        int i =0;
        int ii=0;
        int jj=0;
        for(;j<nums2.size();j++){
            ii = 0;
            jj=j;
            record=0;
            while(ii<nums1.size()&&jj<nums2.size()){
                if(pack[ii][jj]==1){
                    record++;
                }else{
                   
                                     if(record>max){
                    max = record;
                    }
                    record=0;
                             ii++;
                jj++;
                    continue;
                }
                ii++;
                jj++;
            }
            if(record>max){
                max = record;
            }
            
        }

        for(i=1;i<nums1.size();i++){
            jj=0;
            record=0;
            ii =i;
            while(ii<nums1.size()&&jj<nums2.size()){
                if(pack[ii][jj]==1){
                    record++;
                }else{
                 if(record>max){
                    max = record;
                    }
                    record=0;
                             ii++;
                jj++;
                    continue;
                }
                ii++;
                jj++;
            }
            if(record>max){
                max = record;
            }
        }
        return max;
    }
};
```

题解好的地方就是在生成pack的时候已经在记录最大的了，我没有注意到这个递推关系所以烂了。

题解:

```go
class Solution {
public:
    int findLength(vector<int>& A, vector<int>& B) {
        int n = A.size(), m = B.size();
        vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
        int ans = 0;
        for (int i = n - 1; i >= 0; i--) {
            for (int j = m - 1; j >= 0; j--) {
                dp[i][j] = A[i] == B[j] ? dp[i + 1][j + 1] + 1 : 0;
                ans = max(ans, dp[i][j]);
            }
        }
        return ans;
    }
};
```

[647. 回文子串](https://leetcode.cn/problems/palindromic-substrings/)

> 给你一个字符串 s ，请你统计并返回这个字符串中 回文子串 的数目。

> 回文字符串 是正着读和倒过来读一样的字符串。

> 子字符串 是字符串中的由连续字符组成的一个序列。

> 具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被视作不同的子串。

dp的做法基本能写出来，但是这道题给双指针带来的一些启发可以记录下来。

首先给出题解:

```go
class Solution {
public:
    int countSubstrings(string s) {
        int result = 0;
        for (int i = 0; i < s.size(); i++) {
            result += extend(s, i, i, s.size()); // 以i为中心
            result += extend(s, i, i + 1, s.size()); // 以i和i+1为中心
        }
        return result;
    }
    int extend(const string& s, int i, int j, int n) {
        int res = 0;
        while (i >= 0 && j < n && s[i] == s[j]) {
            i--;
            j++;
            res++;
        }
        return res;
    }
};
```

实际上，暴力判断是O(n\^3)。也就是遍历每个开始点和结束点n\^2。并且在判断每一次是否是回文的。 那我们最好是降到O(n\^2),dp的思想就是用上次的判断为下次所用。

而双指针的想法则是将所有的区间分组，分成中心相同的组，然后在这些组内进行从中心开始的双指针，这样上次判断的结果也可以为下一次所用，而且这样把所有的区间全都遍历了。值得注意的是，中心点可以是一个具体的点(奇数的情况)，也可以是点之间的间隔（偶数的情况），这样才可以把所有区间都遍历。
