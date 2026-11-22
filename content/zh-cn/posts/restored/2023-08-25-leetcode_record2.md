---
title: leetcode_record2
date: '2023-08-25T08:01:23+08:00'
lastmod: '2023-10-10T09:27:30+08:00'
draft: false
tags:
- algorithm
- jobs
- leetcode
categories:
- code
url: /zh-cn/2023/08/25/category/leetcode_record2/
authors:
- yehh
---

### 力扣记录2 {#力扣记录2}

#### [剑指 Offer 51. 数组中的逆序对](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/description/) {#剑指-offer-51-数组中的逆序对httpsleetcodecnproblemsshu-zu-zhong-de-ni-xu-dui-lcofdescription}

我对这题的评价是逆天

```c++
class Solution {
public:
    int mergeSort(vector<int>& nums, vector<int>& tmp, int l, int r) {
        if (l >= r) {
            return 0;
        }

        int mid = (l + r) / 2;
        int inv_count = mergeSort(nums, tmp, l, mid) + mergeSort(nums, tmp, mid + 1, r);
        int i = l, j = mid + 1, pos = l;
        while (i <= mid && j <= r) {
            if (nums[i] <= nums[j]) {
                tmp[pos] = nums[i];
                ++i;
                inv_count += (j - (mid + 1));
            }
            else {
                tmp[pos] = nums[j];
                ++j;
            }
            ++pos;
        }
        for (int k = i; k <= mid; ++k) {
            tmp[pos++] = nums[k];
            inv_count += (j - (mid + 1));
        }
        for (int k = j; k <= r; ++k) {
            tmp[pos++] = nums[k];
        }
        copy(tmp.begin() + l, tmp.begin() + r + 1, nums.begin() + l);
        return inv_count;
    }

    int reversePairs(vector<int>& nums) {
        int n = nums.size();
        vector<int> tmp(n);
        return mergeSort(nums, tmp, 0, n - 1);
    }
};
```

1.  先讲讲这题，递归归并排序后，左右两个数组是有序，然后在并的时候统计左侧比右侧大的数量，如何统计呢：就是在左边的归并的时候统计这个时候右侧数组左边有多少已经被归并，这些就是比左侧这个小的。

2.  其实这道题还可以学到归并排序的写法：

```c++
void mergeSort(vector<int>& nums, vector<int>& tmp, int l, int r){
    //递归结束条件，可以变成l-r小于一定值时用插入排序
    if (l >= r) {
            return 0;
    }
    // 递归归并子数组
    int mid = (l + r) / 2;
    mergeSort(nums, tmp, l, mid);
    mergeSort(nums, tmp, mid + 1, r);

    // 合并两个有序数组

    int i = l, j = mid + 1, pos = l;
    while (i <= mid && j <= r) {
        if (nums[i] <= nums[j]) {
            tmp[pos] = nums[i];
            ++i;
        }
        else {
            tmp[pos] = nums[j];
            ++j;
        }
        ++pos;
    }
    // 统一对待剩下的左(右)数组
    for (int k = i; k <= mid; ++k) {
        tmp[pos++] = nums[k];
    }
    for (int k = j; k <= r; ++k) {
        tmp[pos++] = nums[k];
    }
    // 将tmp数组的值复制到nums数组
    copy(tmp.begin() + l, tmp.begin() + r + 1, nums.begin() + l);
    return ;
}
```

### [剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/) {#剑指-offer-56---i-数组中数字出现的次数httpsleetcodecnproblemsshu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof}

先贴代码，感觉这种题就是见过就知道……

```c++
class Solution {
public:
    vector<int> singleNumbers(vector<int>& nums) {
        int ret = 0;
        for (int n : nums)
            ret ^= n;
        int div = 1;
        while ((div & ret) == 0)
            div <<= 1;
        int a = 0, b = 0;
        for (int n : nums)
            if (div & n)
                a ^= n;
            else
                b ^= n;
        return vector<int>{a, b};
    }
};
```

首先一个前置知识是：异或运算的性质：两个相同的数异或为0，一个数和0异或还是它本身。所以当只有一个数字只出现一次时，我们可以将所有的数异或，最后得到的结果就是那个只出现一次的数。

所以这题就是如何分组使得每组只有一个只出现一次的数，那么我们可以先将所有的数异或，得到的结果就是那两个只出现一次的数异或的结果，然后我们找到这个结果中第一个为1的位，因为这个位是两个数不同的位，所以我们可以根据这个位将所有的数分成两组，这样每组就只有一个只出现一次的数了。

### [剑指 Offer 58 - I. 翻转单词顺序](https://leetcode.cn/problems/fan-zhuan-dan-ci-shun-xu-lcof/) {#剑指-offer-58---i-翻转单词顺序httpsleetcodecnproblemsfan-zhuan-dan-ci-shun-xu-lcof}

之前用go写发现挺简单，但是用c++写inplace的就有点难了。

方法我是知道的，先全翻转然后单词翻转，主要就是要注意多个空格怎么办，实际上还是用双指针的思想把空格或者说不用的挪到最后面去，然后erase掉就行了。

上代码：

```c++
class Solution {
public:
    string reverseWords(string s) {
        reverse(s.begin(),s.end());
        int idx = 0;
        int n = s.size();
        for(int start =0;start<n;start++){
            //  cout<<s<<endl;
            if(s[start]!=' '){
                if(idx!=0) s[idx++]=' ';

                int end =start;
                while(end<n&&s[end]!=' '){
                    s[idx++]=s[end++];
                }
                reverse(s.begin()+idx-(end-start),s.begin()+idx);
                // idx+=end-start;
                cout<<"idx:"<<idx<<endl;
                start=end;
            }
            
        }
       
        s.erase(s.begin()+idx,s.end());

        
        return s;
    }
    
};
```

### [岛屿数量](https://leetcode.cn/problems/number-of-islands/description/) {#岛屿数量httpsleetcodecnproblemsnumber-of-islandsdescription}

### [数组第k大](https://leetcode.cn/problems/kth-largest-element-in-an-array/) {#数组第k大httpsleetcodecnproblemskth-largest-element-in-an-array}

主要是要记住两个

1.  这题的最优解法：快速选择

首先给出快排模板：

一个令人惊叹的事实是：这几乎是唯一的模板，他可以用来实现快排，快速选择等分治类型的问题。

```c++
void quick_sort(int q[], int l, int r)
{
    if (l >= r) return;

    int i = l - 1, j = r + 1, x = q[l + r >> 1];
    while (i < j)
    {
        do i++; while (q[i] < x);
        do j--; while (q[j] > x);
        if (i < j) swap(q[i], q[j]);
    }

    quick_sort(q, l, j);
    quick_sort(q, j + 1, r);
}
```

说回这道题：

在while 循环结束之后只能说begin到j是小于等于partition，j+1到end是大于等于partition，但是内部仍然不保证排序，所以要根据k和j的大小关系来决定下一步递归的范围。最后结束是begin==end的时候。

```c++
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        return fastSelect(nums,nums.size()-k,0,nums.size()-1);
    }

    int fastSelect(vector<int>& nums,int k,int begin,int end){
      if(begin==end){
        return nums[k];
      }
      int i = begin;
      int j =end;
      int partition = nums[i];
      i--;
      j++;
      while(i<j){
        do i++;while(nums[i]<partition);
        do j--;while(nums[j]>partition);
        if(i<j){
          swap(nums[i],nums[j]);
        }
      }
      
      if(k<=j){
        return fastSelect(nums,k,begin,j);
      }else{
        return fastSelect(nums,k,j+1,end);
      }
    }

};
```

2.  堆的实现，(c++ 泛型)

学ds的时候学过，建堆有两种第一种是给定一个数组，然后一个一个插入，这样的时间复杂度是O(nlogn)，第二种是给定一个数组，然后从最后一个非叶子节点开始，从下往上调整，这样的时间复杂度是O(n)。

下面给出我的模板:

有几个要注意的点：

1.  两种初始化方式对应上面说的两个
2.  percolate_up对应insert，parent节点的index计算方式：(i-1)/2（也是通过0，1，2）来记忆。
3.  percolate_down对应delete_top，left和right节点的index计算方式：2*i+1,2*i+2（可以用0，1，2来记忆）。而且在建堆的时候是从最后一个非叶子节点开始的，所以这里的i是从heap\_.size()/2开始的。也是一个重要结论。

```c++
template <typename t> class Heap {
public:
  Heap(std::vector<t> h, std::function<bool(t, t)> func)
      : func_(func), heap_(std::move(h)) {
    for (int i = heap_.size() / 2; i >= 0; i--) {
      percolate_down(i);
    }
  }
  Heap(std::function<bool(t, t)> func) : func_(func) {}
  void percolate_up(int i) {
    int parent = (i - 1) / 2;
    if (parent >= 0 && func_(heap_[i], heap_[parent])) {
      std::swap(heap_[i], heap_[parent]);
      percolate_up(parent);
    }
  }

  void insert(t val) {
    heap_.push_back(val);
    percolate_up(heap_.size() - 1);
  }
  int size() { return heap_.size(); }
  void percolate_down(int i) {
    int left = 2 * i + 1;
    int right = 2 * i + 2;
    int smallest = i;
    if (left < heap_.size() && func_(heap_[left], heap_[smallest])) {
      smallest = left;
    }
    if (right < heap_.size() && func_(heap_[right], heap_[smallest])) {
      smallest = right;
    }
    if (smallest != i) {
      std::swap(heap_[i], heap_[smallest]);
      percolate_down(smallest);
    }
  }

  t delete_top() {
    t top = heap_[0];
    heap_[0] = heap_[heap_.size() - 1];
    heap_.pop_back();
    percolate_down(0);
    return top;
  }

  t top() { return heap_[0]; }

private:
  std::vector<t> heap_;
  std::function<bool(t, t)> func_;
};
```

### 处理溢出 {#处理溢出}

一般来说是乘法会出现溢出错误，所以可以用INT_MAX 来先除判断结果是否大于10，加法是不会产生异常的。

### [2875. 无限数组的最短子数组](https://leetcode.cn/problems/minimum-size-subarray-in-infinite-array/description/) {#2875-无限数组的最短子数组httpsleetcodecnproblemsminimum-size-subarray-in-infinite-arraydescription}

> 给你一个下标从 0 开始的数组 nums 和一个整数 target 。

> 下标从 0 开始的数组 infinite_nums 是通过无限地将 nums 的元素追加到自己之后生成的。

> 请你从 infinite_nums 中找出满足 元素和 等于 target 的 最短 子数组，并返回该子数组的长度。如果不存在满足条件的子数组，返回 -1 。

这题我思路全对，但是还是被滑动窗口的细节困住了，选择了直接copy insert一段数组的做法。

先给出比较好的滑窗做法：

其实主要思路就是left，right正常从0到2\*n，但是在计算sum要去num里面找数据的时候可以取模来找对应的数据。 这样是可以达到题目要求的。也可以作为环形数组滑动窗口的解法。

```c++
class Solution {
public:
    int minSizeSubarray(vector<int> &nums, int target) {
        long long total = accumulate(nums.begin(), nums.end(), 0LL);
        int n = nums.size();
        int ans = INT_MAX;
        int left = 0;
        long long sum = 0;
        for (int right = 0; right < n * 2; right++) {
            sum += nums[right % n];
            while (sum > target % total) {
                sum -= nums[left++ % n];
            }
            if (sum == target % total) {
                ans = min(ans, right - left + 1);
            }
        }
        return ans == INT_MAX ? -1 : ans + target / total * n;
    }
};

作者：灵茶山艾府
链接：https://leetcode.cn/problems/minimum-size-subarray-in-infinite-array/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

没这么好的做法：

```c++
class Solution {
public:
   int minSizeSubarray(vector<int>& nums, int target) {
       long long sum = accumulate(nums.begin(), nums.end(), 0L);
       int circle = (int)(target / sum * (int)nums.size());
       target %= sum;
       sum = 0;
       vector<int> infinite_nums = nums;
       infinite_nums.insert(infinite_nums.end(), nums.begin(), nums.end());
       int size = infinite_nums.size(), left = 0, right = 0, minSize = INT_MAX;

       for (; left < size; ++left) {
           while (right < size && sum < target) {
               sum += infinite_nums[right++];
           }

           if (sum == target) {
               minSize = min(minSize, right - left);
           }

           sum -= infinite_nums[left];
       }

       return minSize == INT_MAX ? -1 : minSize + circle;
   }
};
```
