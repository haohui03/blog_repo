---
title: leetcode_record1
date: '2023-08-13T08:01:23+08:00'
lastmod: '2023-08-13T09:27:30+08:00'
draft: false
tags:
- algorithm
- jobs
- leetcode
categories:
- code
url: /zh-cn/2023/08/13/category/leetcode_record1/
authors:
- yehh
---

## 剑指offer {#剑指offer}

### [剑指 Offer 12. 矩阵中的路径](https://leetcode.cn/problems/ju-zhen-zhong-de-lu-jing-lcof/description/) {#剑指-offer-12-矩阵中的路径httpsleetcodecnproblemsju-zhen-zhong-de-lu-jing-lcofdescription}

一眼看出是回溯，其实还是挺简单的，但是要主要一开始遍历来找到起点。

### [剪绳子II](https://leetcode.cn/problems/jian-sheng-zi-ii-lcof/description/) {#剪绳子iihttpsleetcodecnproblemsjian-sheng-zi-ii-lcofdescription}

不能用动态规划，只能贪心，但是贪心的话要用到数学知识，比如说3越多越好，因为3*3\>2*2*2，所以尽量分成3，但是要注意3*2\>2*2，所以如果最后剩下4的话，要分成2*2，而不能分成3\*1

我之前动态规划是看4以及以上的都是乘积比自身大，所以只考虑将绳子剪掉二和三的情况(剪掉一显然小)所以贪心可以从中得到启发就是分成3越多越好，但是要注意3*1\<2*2，所以如果最后剩下4的话，要分成2*2，而不能分成3*1.

### [剑指 Offer 16. 数值的整数次方](https://leetcode.cn/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/description/) {#剑指-offer-16-数值的整数次方httpsleetcodecnproblemsshu-zhi-de-zheng-shu-ci-fang-lcofdescription}

快速指数幂

```c++
class Solution {
public:
    double myPow(double x, int n) {
        //避免-2147483648(2的-31次方)溢出，负数还是等于自己
        long long z = n;
        //处理负指数
        if(n<0){
            z = -z;
            x = 1/x;
        }
        return calculate(x,z);
    }
    double calculate(double x,long long z){
        //每次缩小范围是通过指数的log来下降，所以时间复杂度是logn
        double res =1.0;
        while(z>0){
            if((z&1)==1){
                res*=x;
            }
            z = z>>1;

            x = x*x;
        }
        return res;
    }
};
```

### [正则表达式匹配](https://leetcode.cn/problems/zheng-ze-biao-da-shi-pi-pei-lcof/description/) {#正则表达式匹配httpsleetcodecnproblemszheng-ze-biao-da-shi-pi-pei-lcofdescription}

我对这题的评价是: 生不出人，我很抱歉

```c++
class Solution {
public:
    bool isMatch(string s, string p) {
        int m = s.size();
        int n = p.size();

        auto matches = [&](int i, int j) {
            if (i == 0) {
                return false;
            }
            if (p[j - 1] == '.') {
                return true;
            }
            return s[i - 1] == p[j - 1];
        };

        vector<vector<int>> f(m + 1, vector<int>(n + 1));
        f[0][0] = true;
        for (int i = 0; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                if (p[j - 1] == '*') {
                    f[i][j] |= f[i][j - 2];
                    if (matches(i, j - 1)) {
                        f[i][j] |= f[i - 1][j];
                    }
                }
                else {
                    if (matches(i, j)) {
                        f[i][j] |= f[i - 1][j - 1];
                    }
                }
            }
        }
        return f[m][n];
    }
};
```

### [剑指 Offer 27. 二叉树的镜像](https://leetcode.cn/problems/er-cha-shu-de-jing-xiang-lcof/description/) {#剑指-offer-27-二叉树的镜像httpsleetcodecnproblemser-cha-shu-de-jing-xiang-lcofdescription}

### [剑指 Offer 28. 对称的二叉树](https://leetcode.cn/problems/dui-cheng-de-er-cha-shu-lcof/) {#剑指-offer-28-对称的二叉树httpsleetcodecnproblemsdui-cheng-de-er-cha-shu-lcof}

最近重新做了两道对称二叉树或者说镜像二叉树的题目，对对称二叉树的理解更深了。先来看如何获取镜像：

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root) {
        if(!root){
            return root;
        }
        deque<TreeNode*> q;
        q.push_back(root);
        int layerSize =1;
        while(q.size()!=0){
            for(int i =0;i<layerSize;i++){
                if(!q[0]){
                    q.pop_front();
                    continue;
                }
                swap(q[0]->left,q[0]->right);
                
                q.push_back(q[0]->left);
                q.push_back(q[0]->right);
                
                    
                
                q.pop_front();
            }
            layerSize = q.size();
            
        }
        return root;
    }
};
```

这里我使用了中序遍历，把一棵二叉树的轴对称这个操作看成层序遍历下的左右节点交换，这样问题范围就缩小成左右子树的镜像，然后一直。。。就行

### [剑指 Offer 37. 序列化二叉树](https://leetcode.cn/problems/xu-lie-hua-er-cha-shu-lcof/description/) {#剑指-offer-37-序列化二叉树httpsleetcodecnproblemsxu-lie-hua-er-cha-shu-lcofdescription}

序列化二叉树，我这里用的是层序遍历，要注意的是，如果是空节点，也要加入到序列化字符串中，这样才能保证反序列化的时候，空节点也能被还原出来。

### [剑指 Offer 39. 数组中出现次数超过一半的数字](https://leetcode.cn/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/description/) {#剑指-offer-39-数组中出现次数超过一半的数字httpsleetcodecnproblemsshu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcofdescription}

虽然是一道简单题，可以用hash表解决，但是有个最好的方法还是可以记一记，就是众数的性质，众数的个数大于一半，所以如果我们把众数记为1，其他数记为-1，那么最后的和一定大于0，这样就可以用一个变量来记录当前和，如果和为0，那么就把当前数记为众数，最后返回的时候，还要判断一下这个数是不是众数，因为有可能没有众数，比如\[1,2,3,4,5,6,7,8,9\]，这样的数组就没有众数。

两个数比较

1.  都是众数，那么num++；
2.  一个众数一个其他，num–
3.  两个都不是众数那–(因为去掉两个非众数，众数还是在剩下的数组里面是众数)

代码:

```c++
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int candidate = -1;
        int count = 0;
        for (int num : nums) {
            if (num == candidate)
                ++count;
            else if (--count < 0) {
                candidate = num;
                count = 1;
            }
        }
        return candidate;
    }
};
```

### [1444. 切披萨的方案数](https://leetcode.cn/problems/number-of-ways-of-cutting-a-pizza/description/) {#1444-切披萨的方案数httpsleetcodecnproblemsnumber-of-ways-of-cutting-a-pizzadescription}

属于是顶级的逆天题。

1.  前缀和减少重复计算，这个我见得比较少，但是实际上应该是比较常见的，所以要记住。这道题是二维前缀和，也就是说，我们可以用一个二维数组来记录到当前位置的所有苹果的数量，这样我们就可以在O(1)的时间内计算出任意一个矩形区域的苹果数量。
2.  dp，其实dp这个思路不难想到，难的是状态定义和状态方程的定义。

直接看代码吧

```c++
class Solution {
public:
    int ways(vector<string>& pizza, int k) {
        int m = pizza.size(), n = pizza[0].size(), mod = 1e9 + 7;
        vector<vector<int>> apples(m + 1, vector<int>(n + 1));
        vector<vector<vector<int>>> dp(k + 1, vector<vector<int>>(m + 1, vector<int>(n + 1)));

        // 预处理
        for (int i = m - 1; i >= 0; i--) {
            for (int j = n - 1; j >= 0; j--) {
                apples[i][j] = apples[i][j + 1] + apples[i + 1][j] - apples[i + 1][j + 1] + (pizza[i][j] == 'A');
                dp[1][i][j] = apples[i][j] > 0;
            }
        }

        for (int ki = 2; ki <= k; ki++) {
            for (int i = 0; i < m; i++) {
                for (int j = 0; j < n; j++) {
                    // 水平方向切
                    for (int i2 = i + 1; i2 < m; i2++) {
                        if (apples[i][j] > apples[i2][j]) {
                            dp[ki][i][j] = (dp[ki][i][j] + dp[ki - 1][i2][j]) % mod;
                        }
                    }
                    // 垂直方向切
                    for (int j2 = j + 1; j2 < n; j2++) {
                        if (apples[i][j] > apples[i][j2]) {
                            dp[ki][i][j] = (dp[ki][i][j] + dp[ki - 1][i][j2]) % mod;
                        }
                    }
                }
            }
        }
        return dp[k][0][0];
    }
};

// 作者：力扣官方题解
// 链接：https://leetcode.cn/problems/number-of-ways-of-cutting-a-pizza/solutions/2387392/qie-pi-sa-de-fang-an-shu-by-leetcode-sol-7ik7/
// 来源：力扣（LeetCode）
// 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

下面这个是优化的

```c++
class Solution {
public:
    int ways(vector<string> &pizza, int k) {
        const int MOD = 1e9 + 7;
        int m = pizza.size(), n = pizza[0].length();
        vector<vector<int>> sum(m + 1, vector<int>(n + 1)); // 二维后缀和
        vector<vector<int>> f(m + 1, vector<int>(n + 1));
        for (int i = m - 1; i >= 0; i--) {
            for (int j = n - 1; j >= 0; j--) {
                sum[i][j] = sum[i][j + 1] + sum[i + 1][j] - sum[i + 1][j + 1] + (pizza[i][j] & 1);
                if (sum[i][j]) f[i][j] = 1; // 初始值
            }
        }

        while (--k) {
            vector<int> col_s(n); // colS[j] 表示 f 第 j 列的后缀和
            for (int i = m - 1; i >= 0; i--) {
                int row_s = 0; // f[i] 的后缀和
                for (int j = n - 1; j >= 0; j--) {
                    int tmp = f[i][j];
                    if (sum[i][j] == sum[i][j + 1]) // 左边界没有苹果
                        f[i][j] = f[i][j + 1];
                    else if (sum[i][j] == sum[i + 1][j]) // 上边界没有苹果
                        f[i][j] = f[i + 1][j];
                    else // 左边界上边界都有苹果，那么无论怎么切都有苹果
                        f[i][j] = (row_s + col_s[j]) % MOD;
                    row_s = (row_s + tmp) % MOD;
                    col_s[j] = (col_s[j] + tmp) % MOD;
                }
            }
        }
        return f[0][0];
    }
};

// 作者：灵茶山艾府
// 链接：https://leetcode.cn/problems/number-of-ways-of-cutting-a-pizza/solutions/1/ji-bai-100cong-di-gui-dao-di-tui-dao-you-dxz5/
// 来源：力扣（LeetCode）
// 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

### [剑指 Offer 41. 数据流中的中位数](https://leetcode.cn/problems/shu-ju-liu-zhong-de-zhong-wei-shu-lcof/description/) {#剑指-offer-41-数据流中的中位数httpsleetcodecnproblemsshu-ju-liu-zhong-de-zhong-wei-shu-lcofdescription}

这题主要是要想到用两个堆来实现，然后每次添加的情况要考虑清除，总共有三个区间。一步步来就不会忘

### [剑指 Offer 43. 1～n 整数中 1 出现的次数](https://leetcode.cn/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/description/) {#剑指-offer-43-1n-整数中-1-出现的次数httpsleetcodecnproblems1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcofdescription}

这题思路对了，而且时间上击败100%

一开始老是出错有几个原因：

1.  在递归的时候处理首位情况没有考虑清楚，也就是首位大于一，等于1，小于1(等于0)的情况没有想清楚。
2.  递归结束条件也没有想清楚

```c++
class Solution {
public:
    int countDigitOne(int n) {
        long long i =1;
        while(n/i){
            i*=10;
        }
        i/=10;
        return recursive(n,i);
        

        
    }
    int recursive(int n ,long long i){
        if(n==0){
            return 0;
        }
        if(i==1){
            return 1;
        }
        int first  =n/i;
        int rest = n%i;
        
        if(first==0){
            return recursive(rest,i/10);
        }

        // cout<<first<<rest<<endl;
        if(first>1){
            return (first)*recursive(i-1,i/10)+recursive(rest,i/10)+i;
        }
        return rest+1+(first)*recursive(i-1,i/10)+recursive(rest,i/10);
        
        
    }
};
```

### [剑指 Offer 44. 数字序列中某一位的数字](https://leetcode.cn/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/description/) {#剑指-offer-44-数字序列中某一位的数字httpsleetcodecnproblemsshu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcofdescription}

这个其实也是数位找规律的问题，要问某一位的数是什么，就得知道这位在哪个数上，并且在这个数的哪一位。 首先就是按照 0-9 10-99来给位数分组，然后就能知道这个位是在该组的哪个数

```c++
class Solution {
public:
    int findNthDigit(int n) {
        if(n<10){
            return n;
        }
        long long nn = n;
        long long i=1;
        long long max_ = 10;
        long long min_ =0;
        long long count = 0;
        while(count+(max_-min_)*i<nn+1){
            count+=(max_-min_)*i;
            i+=1;
            min_=max_;
            max_*=10;
            
        }
        long long newPos = nn-count;
        long long first = newPos/i;
        long long rest = newPos%i;
        long long num = min_+first;
        // cout<<newPos<<" "<<first<<" "<<rest<<" "<<num<<" "<<i;
        long long  shift = (long long)(pow(10,(i-1-rest)));
        return  (num/shift)%10;
        

    }
};
```

### [剑指 Offer 46. 把数字翻译成字符串](https://leetcode.cn/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/description/) {#剑指-offer-46-把数字翻译成字符串httpsleetcodecnproblemsba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcofdescription}

其实也是数位计算，有几个需要改进的地方

1.  计算掩码位的时候一开始写成

```c++
    int i =1;
    while(num/i>10){
        i*=10;
    }
```

但是应该是\>=

2.  又是考虑每次的结束和分支

    - 结束，i\<=1的时候退出返回1.
    - 分支：
      - 首位0:意味着第一位和前两位的编码是一样的所以直接返回后面的编码方案数
      - 首位非零: 就是比较直观的两种情况:单个数字编码和可能两位数的编码

### [剑指 Offer 49. 丑数](https://leetcode.cn/problems/chou-shu-lcof/description/) {#剑指-offer-49-丑数httpsleetcodecnproblemschou-shu-lcofdescription}

其实很容易看出来是DP，但是状态转移不好想，主要是顺序很难保证。

基于这样一种思想，后面的丑数总是从前面的生成-\>前面的数总是会乘2，3，5来得到后面的数，所以用三个指针来从左向右跟踪这三个序列中最小的每次取最小的就可以保证顺序了。

### [剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode.cn/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/description/) {#剑指-offer-53---i-在排序数组中查找数字-ihttpsleetcodecnproblemszai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcofdescription}

其实就是二分查找，但是这个是二分查找的变体，就是会有重复的数字，所以需要找是target的第一个和最后一个

可以总结下来的写法: 可以发现，如果想找第一个那就是在相等的时候j=middle，注意这个时候的middle是(i+j)/2也就是说会偏向小的，那把right赋小值也是可以跳出i\<j的循环的。

那么最后一个也相似，就是把左边界赋middle，同时middle的算法是(i+j+1)/2。

需要注意的是，当找不到的时候得到的两个结果有可能越界找第一个值的时候j=-1,找最后一个值的时候可能i=num.size()。

```c++
class Solution {
public:
    int search(vector<int>& nums, int target) {
        if(nums.size()==0){
            return 0;
        }
        //binary search 

        int i =0;
        int j = nums.size()-1;
        while(i<j){
            //  cout<<i<<" "<<j<<endl;
            int middle = (i+j)/2;
            if(nums[middle]>target){
                j=middle-1;
            }else{
                if(nums[middle]<target){
                    i=middle+1;
                }else{
                    j=middle;
                }
            }
        }
       
        if(j<0||nums[j]!=target){
            return 0;
        }
        
        //j leftmost target
        int ii=j;
        int jj = nums.size()-1;
        cout<<j<<" "<<ii;
        while(ii<jj){
            int middle = (ii+jj+1)/2;
            if(nums[middle]>target){
                jj=middle-1;
            }else{
                if(nums[middle]>target){
                    ii=middle+1;
                }else{
                    ii=middle;
                }
            }
        }
        
        return ii-j+1;
    }
};
```
