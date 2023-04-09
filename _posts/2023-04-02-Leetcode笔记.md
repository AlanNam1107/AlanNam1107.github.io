---
layout:     post
title:      "Leetcode"
subtitle:   "更新中"
author:     "Alan Nam"
header-img: "img/in-post/post-bg-hello.jpg"
tags:
    - Leetcode
    - Greedy Algorithm
    - Dynamic Programming
    - Divide and conquer
---



# Leetcode

## ACM数据读取

```c++
ios::sync_with_stdio(false); // 让 cout cin 比 scanf 和 printf 更快
std::cin.tie(0);
//sync_with_stdio是输入同步开关
//cin.tie是控制输出同步
```

### C++ 中数据读取

C++ 输入过程中，是把输入加载到缓冲区中，然后对缓冲区中的字符进行读取。`cin`、`cin.get()`、 `cin.getline()`、`geline()` 四个函数虽然都能进行数据读取，但是它们对缓冲区内数据的处理方法是不同的。下面会介绍它们之间的区别。

#### cin

结束条件：`[enter]` , `[space]` , `[tab]`

处理方法：`cin` 输入从非 `[enter]` ,`[sapce]`, `[tab]`开始（丢弃），遇到 `[enter]` ,`[space]`,`[tab]`表示当前输入结束。

#### cin.get()

用法: `char a = cin.get()`

处理方法：从当前位置读取一个字符，存入 `a` 中。

#### 读取一行数组

```cpp
/**
 * Sample Input
 * 1 3 5 7 9
 */

// C
while (scanf("%d", &a) == 1)
{
    /* code */
}

// C++
while (cin >> a)
{
    /* code */
}
```

#### 每次读取两个数

```cpp
/**
 * Sample Input
 * 1 3 5 7
 * Output
 * -2
 * -2
 */

int a, b;

// C
while(scanf("%d %d", &a, &b) == 2) printf("%d\n", a-b);

// C++
while(cin >> a >> b) cout << a-b << endl;
```

#### 限定输入数量

```cpp
/**
 * Sample Input
 * 2
 * 1 5
 * 10 20
 * Output:
 * 6
 * 30
*/

ios::sync_with_stdio(false); // 让 cout cin 比 scanf 和 printf 更快
int n, a, b;
cin >> n;
while(n--) {
    cin >> a >> b;
    cout << a - b << endl;
}

#include<iostream>
#include<vector>
using namespace std;
int main() {
    int n;
    while (cin >> n) {
        vector<int> gym(n);
        vector<int> work(n);
        for (int i = 0; i < n; i++) cin >> work[i];
        for (int i = 0; i < n; i++) cin >> gym[i];
        int result = 0;

        // 处理逻辑

        cout << result << endl;
    }
    return 0;
}
```

#### 以特定输入为结束标志

例如：以 `0 0` 作为结束标志

```cpp
/**
 * Sample Input
 * 1 5
 * 10 20
 * 0 0
 * Output:
 * 6
 * 30
 */

// C
int a, b;
while(scanf("%d %d", &a, &b) && (a !=0 && b != 0)) {
    printf("%d\n", a+b);
}

// C++
while(cin >> a >> b && (a != 0 && b != 0)) {
    cout << a + b << endl;
}
```

#### 读取整行字符串

读取整行字符串不能使用 `cin`， `cin` 会忽略 空格、回车符、制表符 等。

```cpp
// 使用字符串
string buf;
getline(cin, buf);

// 使用字符数组
char buff[255];
cin.getline(buff, 255);
```

#### 以空格为分割的数组

```cpp
/** 数组以空格分割
 * Sample Input
 * 1 2 3 4 5
 * 11 12 13 14 15
 * 21 22 23 24 25
 */

// 只知道有三行，每列未知
vector<vector<int>> res(3);

for(int i = 0; i < 3; i++) {
    int a;
    while(cin >> a) {
        res[i].push_back(a);
        if(cin.get() == '\n')
            break;
    }
}
```

#### 数组以 ',' 分割

```cpp
/**
 * 未知数组长度，用 ',' 隔开
 * Sample Input
 * 1,2,3,4,5,6,7
 * 0,2,3,4,5,7,3
 */

vector<int> w;
vector<int> v;

string s1, s2;
cin >> s1 >> s2;

string tmp;
stringstream ss;

ss << s1;

while (getline(ss, tmp, ','))
{
    w.push_back(stoi(tmp));
}

ss.clear();

ss << s2;
while (getline(ss, tmp, ','))
{
    v.push_back(stoi(tmp));
}
```

### c++类型范围

| 类型      | 16位系统/字节 | 32位系统/字节 | 64位系统/字节 |
| --------- | ------------- | ------------- | ------------- |
| char      | 1             | 1             | 1             |
| char*     | 2             | 4             | 8             |
| short     | 2             | 2             | 2             |
| int       | 2             | 4             | 4             |
| long      | 4             | 4             | 8             |
| long long | 8             | 8             | 8             |

一些int不够要用longlong，注意

LONG_MIN, LONG_MAX



## 线性表

### 数组

#### [704. 二分查找](https://leetcode.cn/problems/binary-search/)

给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。

```c++
// 版本一
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1; // 定义target在左闭右闭的区间里，[left, right]
        while (left <= right) { // 当left==right，区间[left, right]依然有效，所以用 <=
            int middle = left + ((right - left) / 2);// 防止溢出 等同于(left + right)/2
            if (nums[middle] > target) {
                right = middle - 1; // target 在左区间，所以[left, middle - 1]
            } else if (nums[middle] < target) {
                left = middle + 1; // target 在右区间，所以[middle + 1, right]
            } else { // nums[middle] == target
                return middle; // 数组中找到目标值，直接返回下标
            }
        }
        // 未找到目标值
        return -1;
    }
};

// 版本二
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size(); // 定义target在左闭右开的区间里，即：[left, right)
        while (left < right) { // 因为left == right的时候，在[left, right)是无效的空间，所以使用 <
            int middle = left + ((right - left) >> 1);
            if (nums[middle] > target) {
                right = middle; // target 在左区间，在[left, middle)中
            } else if (nums[middle] < target) {
                left = middle + 1; // target 在右区间，在[middle + 1, right)中
            } else { // nums[middle] == target
                return middle; // 数组中找到目标值，直接返回下标
            }
        }
        // 未找到目标值
        return -1;
    }
};
```

- 时间复杂度：O(log n)
- 空间复杂度：O(1)

#### [27. 移除元素](https://leetcode.cn/problems/remove-element/)——双指针

```c++
// 时间复杂度：O(n)
// 空间复杂度：O(1)
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int slowIndex = 0;
        for (int fastIndex = 0; fastIndex < nums.size(); fastIndex++) {
            if (val != nums[fastIndex]) {
                nums[slowIndex++] = nums[fastIndex];
            }
        }
        return slowIndex;
    }
    
/**
* 相向双指针方法，基于元素顺序可以改变的题目描述改变了元素相对位置，确保了移动最少元素
* 时间复杂度：O(n)
* 空间复杂度：O(1)
*/
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int leftIndex = 0;
        int rightIndex = nums.size() - 1;
        while (leftIndex <= rightIndex) {
            // 找左边等于val的元素
            while (leftIndex <= rightIndex && nums[leftIndex] != val){
                ++leftIndex;
            }
            // 找右边不等于val的元素
            while (leftIndex <= rightIndex && nums[rightIndex] == val) {
                -- rightIndex;
            }
            // 将右边不等于val的元素覆盖左边等于val的元素
            if (leftIndex < rightIndex) {
                nums[leftIndex++] = nums[rightIndex--];
            }
        }
        return leftIndex;   // leftIndex一定指向了最终数组末尾的下一个元素
    }
};
    
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int left = 0, right = nums.size();
        while (left < right) {
            if (nums[left] == val) {
                nums[left] = nums[right - 1];
                right--;
            } else {
                left++;
            }
        }
        return left;
    }
};

```

#### [977. 有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/)——双指针

```c++
class Solution {
public:
    vector<int> sortedSquares(vector<int>& A) {
        int k = A.size() - 1;
        vector<int> result(A.size(), 0);
        for (int i = 0, j = A.size() - 1; i <= j;) { // 注意这里要i <= j，因为最后要处理两个元素
            if (A[i] * A[i] < A[j] * A[j])  {
                result[k--] = A[j] * A[j];
                j--;
            }
            else {
                result[k--] = A[i] * A[i];
                i++;
            }
        }
        return result;
    }
};
此时的时间复杂度为O(n)，相对于暴力排序的解法O(n + nlog n)
class Solution {
public:
    vector<int> sortedSquares(vector<int>& A) {
        for (int i = 0; i < A.size(); i++) {
            A[i] *= A[i];
        }
        sort(A.begin(), A.end()); // 快速排序
        return A;
    }
};   
    
```

#### [209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)——双指针/滑动窗口

```c++
时间复杂度：O(n)
空间复杂度：O(1)
class Solution {
public:
    int minSubArrayLen(int s, vector<int>& nums) {
        int n = nums.size();
        if (n == 0) {
            return 0;
        }
        int ans = INT_MAX;
        int start = 0, end = 0;
        int sum = 0;
        while (end < n) {
            sum += nums[end];
            while (sum >= s) {
                ans = min(ans, end - start + 1);
                sum -= nums[start++];
            }
            end++;
        }
        return ans == INT_MAX ? 0 : ans;
    }
};
```



#### [59. 螺旋矩阵 II](https://leetcode.cn/problems/spiral-matrix-ii/)——模拟

```c++
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        int num = 1;
        int left = 0, top = 0, right = n - 1, bottom = n - 1;

        //初始化数组
        vector<vector<int>> res(n,vector<int>(n));
        while (num <= n*n ) {

            //left to right
            for (int i = left; i <= right; ++i) res[top][i] = num++;
            ++top;

            //top to bottom
            for (int i = top; i <= bottom; ++i) res[i][right] = num++;
            --right;

            //right to left
            for (int i = right; i >= left; --i) res[bottom][i] = num++;
            --bottom;

            //bottom to top
            for (int i = bottom; i >= top; --i) res[i][left] = num++;
            ++left;
        }
        return res;
    }
};
```



#### [31. 下一个排列](https://leetcode.cn/problems/next-permutation/)

##### **算法描述**

对于长度为 n 的排列 a：

首先**从后向前**查找第一个顺序对 (i,i+1)，满足 a[i] < a[i+1]。这样「较小数」即为 a[i]。此时 [i+1,n) 必然是下降序列。

如果找到了顺序对，那么在区间 [i+1,n)中**从后向前**查找第一个元素 j 满足 a[i] < a[j]。这样「较大数」即为 a[j]。

交换 a[i] 与 a[j]，此时可以证明区间 [i+1,n)必为降序。我们可以直接使用双指针反转区间 [i+1,n) 使其变为升序，而无需对该区间进行排序。

##### **注意**

如果在步骤 1 找不到顺序对，说明当前序列已经是一个降序序列，即最大的序列，我们直接跳过步骤 2 执行步骤 3，即可得到最小的升序序列。

该方法支持序列中存在重复元素，且在 C++ 的标准库函数 next_permutation 中被采用。

##### **代码**

```c++
//官解答
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int i = nums.size() - 2;
        while (i >= 0 && nums[i] >= nums[i + 1]) {
            i--;
        }
        if (i >= 0) {
            int j = nums.size() - 1;
            while (j >= 0 && nums[i] >= nums[j]) {
                j--;
            }
            swap(nums[i], nums[j]);
        }
        reverse(nums.begin() + i + 1, nums.end());
    }
};

//模板
//Next Permutation
class Solution {
 public:
  void nextPermutation(vector<int> &nums) {
    next_permutation(nums.begin(), nums.end());
  }
  template <typename BidiIt>
  bool next_permutation(BidiIt first, BidiIt last) {
    // Get a reversed range to simplify reversed traversal.
    const auto rfirst = reverse_iterator<BidiIt>(last);
    const auto rlast = reverse_iterator<BidiIt>(first);
    // Begin from the second last element to the first element.
    auto pivot = next(rfirst);
    // Find `pivot`, which is the first element that is no less than its
    // successor. `Prev` is used since `pivort` is a `reversed_iterator`.
    while (pivot != rlast && *pivot >= *prev(pivot)) ++pivot;
    // No such elemenet found, current sequence is already the largest
    // permutation, then rearrange to the first permutation and return false.
    if (pivot == rlast) {
      reverse(rfirst, rlast);
      return false;
    }
    // Scan from right to left, find the first element that is greater than
    // `pivot`.
    auto change = find_if(rfirst, pivot, bind1st(less<int>(), *pivot));
    swap(*change, *pivot);
    reverse(rfirst, pivot);
    return true;
  }
};

```

##### **复杂度分析**

时间复杂度：O(N)，其中 N 为给定序列的长度。我们至多只需要扫描两次序列，以及进行一次反转操作。

空间复杂度：O(1)，只需要常数的空间存放若干变量。



#### [60. 排列序列](https://leetcode.cn/problems/permutation-sequence/)

Given n and k, return the k~th~ permutation sequence. 

Note: Given n will be between 1 and 9 inclusive.

##### 算法描述

方案一：暴力枚举法，调用k-1次next_permutation()

方案二：康托编码的解码

##### **康托编码**

**康托展开式**

X = a[n]*(n-1)! + a[n-1]*(n-2)! + … + a[i]*(i-1)! + … + a[2]*1! + a[1]*0!

康托展开可以实现按字典序排序的序列与自然数之间相互对应的双射。

**康托编码**

给出一个全排列数字，求出这个数是第几个全排列。

规则：

a[n] 代表比处在第n位的数字小，并且在第n位前面没有出现过的数字的个数。（个位是第1位）
X 代表比这个全排列的数字小的数的个数，意味着这个全排列排在第 X+1 位。
例子： 
集合{1,2,3,4,5}，按照字典序排好它的全排列，问 45231 这个数字在排列中的序号是多少？

1、比数字4小的数有3个； 
2、比数字5小的数有4个，但是4已经出现在前面了，所以只有3个； 
3、比数字2小的数有1个； 
4、比数字3小的数有2个，但是2已经出现在前面了，所以只有1个； 
5、比数字1小的数有0个；

因此 X = 3* 4! + 3* 3! + 1* 2! + 1* 1! + 0* 0! 
　　　 = 72 + 18 + 2 + 1 +0 
　　　 = 93 
　　　

**康托解码**

给出一个序号y，求出对应这个序号的全排列是什么数字。

规则： 
解码是编码的逆运算，假如这个数字的序号是y，那么比它小的数就有X=y-1个，然后分别求出每个位置的数字是多少。

例子： 
集合{1,2,3,4,5}，按照字典序排好它的全排列，问第94个全排列是什么数？

首先 X = 94 - 1 = 93 
1、93 / 4! = 3，余21。有3个数比它小的数字是4，所以排在第5位的数字是4； 
2、21 / 3! = 3，余 3 。有3个数比它小的数字是4，但4已经有了，因此排在第4位的数字是5； 
3、 3 / 2! = 1，余 1 。有1个数比它小的数字是2，所以排在第3位的数字是2； 
4、 1 / 1! = 1，余 0 。有1个数比它小的数字是2，但2已经有了，因此排在第2位的数字是3； 
5、 0 / 0! = 0，余 0 。有0个数比它小的数字是1，所以排在第1位的数字是1；

所以排在第94位的全排列是 45231 。

##### 代码

```c++
// LeetCode, Permutation Sequence
class Solution {
 public:
  string getPermutation(int n, int k) {
    string s(n, '0');
    for (int i = 0; i < n; ++i) s[i] += i + 1;
    for (int i = 0; i < k - 1; ++i) next_permutation(s.begin(), s.end());
    return s;
  }
  template <typename BidiIt>
  bool next_permutation(BidiIt first, BidiIt last) {
    // Next Permutation
  }
};
// LeetCode, Permutation Sequence
class Solution {
 public:
  string getPermutation(int n, int k) {
    string s(n, '0');
    string result;
    for (int i = 0; i < n; ++i) s[i] += i + 1;
    return kth_permutation(s, k);
  }

 private:
  int factorial(int n) {
    int result = 1;
    for (int i = 1; i <= n; ++i) result *= i;
    return result;
  }
  // seq 已排好序，是第一个排列
  template <typename Sequence>
  Sequence kth_permutation(const Sequence &seq, int k) {
    const int n = seq.size();
    Sequence S(seq);
    Sequence result;
    int base = factorial(n - 1);
    --k;  //康托编码从0开始
    for (int i = n - 1; i > 0; k %= base, base /= i, --i) {
      auto a = next(S.begin(), k / base);
      result.push_back(*a);
      S.erase(a);
    }
    result.push_back(S[0]);  //最后一个
    return result;
  }
};
```

##### 复杂度分析

方案一：暴力枚举法，调用k-1次next_permutation()

时间复杂度：O(n^2^)

方案二：康托编码的解码

时间复杂度：O(n)，空间复杂度：O(1)

#### [421. 数组中两个数的最大异或值](https://leetcode.cn/problems/maximum-xor-of-two-numbers-in-an-array/)

```c++
class Solution {
private:
    // 最高位的二进制位编号为 30
    static constexpr int HIGH_BIT = 30;

public:
    int findMaximumXOR(vector<int>& nums) {
        int x = 0;
        for (int k = HIGH_BIT; k >= 0; --k) {
            unordered_set<int> seen;
            // 将所有的 pre^k(a_j) 放入哈希表中
            for (int num: nums) {
                // 如果只想保留从最高位开始到第 k 个二进制位为止的部分
                // 只需将其右移 k 位
                seen.insert(num >> k);
            }

            // 目前 x 包含从最高位开始到第 k+1 个二进制位为止的部分
            // 我们将 x 的第 k 个二进制位置为 1，即为 x = x*2+1
            int x_next = x * 2 + 1;
            bool found = false;
            
            // 枚举 i
            for (int num: nums) {
                if (seen.count(x_next ^ (num >> k))) {
                    found = true;
                    break;
                }
            }

            if (found) {
                x = x_next;
            }
            else {
                // 如果没有找到满足等式的 a_i 和 a_j，那么 x 的第 k 个二进制位只能为 0
                // 即为 x = x*2
                x = x_next - 1;
            }
        }
        return x;
    }
};
```



#### [1863. 找出所有子集的异或总和再求和](https://leetcode.cn/problems/sum-of-all-subset-xor-totals/)

```c++
class Solution {
public:
    int subsetXORSum(vector<int>& nums) {
        int res = 0;
        int n = nums.size();
        for (auto num: nums){
            res |= num;
        }
        return res << (n - 1);
    }
};
```



#### [15. 三数之和](https://leetcode.cn/problems/3sum/)

```c++
//先排序，然后左右夹逼，注意跳过重复的数 O(n^2)
class Solution {
public:
  vector<vector<int>> threeSum(vector<int> &nums) {
    vector<vector<int>> result;
    if (nums.size() < 3)
      return result;
    sort(nums.begin(), nums.end());
    const int target = 0;
    auto last = nums.end();
    for (auto i = nums.begin(); i < last - 2; ++i) {
      auto j = i + 1;
      if (i > nums.begin() && *i == *(i - 1))
        continue;
      auto k = last - 1;
      while (j < k) {
        if (*i + *j + *k < target) {
          ++j;
          while (*j == *(j - 1) && j < k)
            ++j;
        } else if (*i + *j + *k > target) {
          --k;
          while (*k == *(k + 1) && j < k)
            --k;
        } else {
          result.push_back({*i, *j, *k});
          ++j;
          --k;
          while (*j == *(j - 1) && *k == *(k + 1) && j < k) {
            ++j;
            --k;
          }
        }
      }
    }
    return result;
  }
};
```

#### [16. 最接近的三数之和](https://leetcode.cn/problems/3sum-closest/)

```c++
class Solution {
public:
  int threeSumClosest(vector<int> &nums, int target) {
    int result = 0;
    int min_gap = INT_MAX;
    sort(nums.begin(), nums.end());
    for (auto a = nums.begin(); a != prev(nums.end(), 2); ++a) {
      auto b = next(a);
      auto c = prev(nums.end());
      while (b < c) {
        const int sum = *a + *b + *c;
        const int gap = abs(sum - target);
        if (gap < min_gap) {
          result = sum;
          min_gap = gap;
        }
        if (sum < target)
          ++b;
        else
          --c;
      }
    }
    return result;
  }
};
```

#### [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/) 双指针/栈

```c++
class Solution {
public:
    int trap(vector<int>& height) {
        int ans = 0;
        int left = 0, right = height.size() - 1;
        int leftMax = 0, rightMax = 0;
        while (left < right) {
            leftMax = max(leftMax, height[left]);
            rightMax = max(rightMax, height[right]);
            if (height[left] < height[right]) {
                ans += leftMax - height[left];
                ++left;
            } else {
                ans += rightMax - height[right];
                --right;
            }
        }
        return ans;
    }
};
class Solution {
public:
  int trap(const vector<int> &A) {
    const int n = A.size();
    int max = 0; //最高柱子分两半
    for (int i = 0; i < n; i++)
      if (A[i] > A[max])
        max = i;
    int water = 0;
    for (int i = 0, peak = 0; i < max; i++)
      if (A[i] > peak)
        peak = A[i];
      else
        water += peak - A[i];
    for (int i = n - 1, top = 0; i > max; i--)
      if (A[i] > top)
        top = A[i];
      else
        water += top - A[i];
    return water;
  }
};
class Solution {
public:
  int trap(const vector<int> &A) {
    const int n = A.size();
    stack<pair<int, int>> s;
    int water = 0;
    for (int i = 0; i < n; ++i) {
      int height = 0;
      while (
          !s.empty()) { // 将栈里比当前元素矮或等高的元素全部处理掉
        int bar = s.top().first;
        int pos = s.top().second;
        // bar, height, A[i]三者夹成一个凹槽
        water += (min(bar, A[i]) - height) * (i - pos - 1);
        height = bar;
        if (A[i] < bar) // 碰到比当前元素高的元素就
          break;
        else
          s.pop(); // 否则就弹出栈顶元素 该元素处理完不需要了
      }
      s.push(make_pair(A[i], i));
    }
    return water;
  }
};

```



### 链表

#### [92. 反转链表 II](https://leetcode.cn/problems/reverse-linked-list-ii/)

```c++
class Solution {
public:
  ListNode *reverseBetween(ListNode *head, int m, int n) {
    ListNode dummy(-1);
    dummy.next = head;
    ListNode *prev = &dummy;
    for (int i = 0; i < m - 1; ++i)
      prev = prev->next;
    ListNode *const head2 = prev;
    prev = head2->next;
    ListNode *cur = prev->next;
    for (int i = m; i < n; ++i) {
      prev->next = cur->next;
      cur->next = head2->next;
      head2->next = cur; // 头插法
      cur = prev->next;
    }
    return dummy.next;
  }
};

```



#### [203. 移除链表元素](https://leetcode.cn/problems/remove-linked-list-elements/)

```c++
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        ListNode dummyHead = ListNode(0); // 设置一个虚拟头结点
        dummyHead->next = head; // 将虚拟头结点指向head，这样方面后面做删除操作
        ListNode* cur = &dummyHead;
        while (cur->next != NULL) {
            if(cur->next->val == val) {
                ListNode* tmp = cur->next;
                cur->next = cur->next->next;
                delete tmp;
            } else {
                cur = cur->next;
            }
        }
        return dummyHead->next;
    }
};
```

#### [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)

```c++
class Solution {
public:
  ListNode *swapPairs(ListNode *head) {
    if (head == nullptr || head->next == nullptr)
      return head;
    ListNode dummy(-1);
    dummy.next = head;
    for (ListNode *prev = &dummy, *cur = prev->next, *next = cur->next; next;
         prev = cur, cur = cur->next, next = cur ? cur->next : nullptr) {
      prev->next = next;
      cur->next = next->next;
      next->next = cur;
    }
    return dummy.next;
  }
};
//交换元素 
class Solution {
public:
  ListNode *swapPairs(ListNode *head) {
    ListNode *p = head;
    while (p && p->next) {
      swap(p->val, p->next->val);
      p = p->next->next;
    }
    return head;
  }
};
```

- 时间复杂度：O(n)
- 空间复杂度：O(1)

#### [19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)——双指针

```cpp
class Solution {
public:
  ListNode *removeNthFromEnd(ListNode *head, int n) {
    ListNode dummy{-1, head};
    ListNode *p = &dummy, *q = &dummy;
    for (int i = 0; i < n; i++) // q 先走n步
      q = q->next;
    while (q->next) { // ̯一起走
      p = p->next;
      q = q->next;
    }
    ListNode *tmp = p->next;
    p->next = p->next->next;
    delete tmp;
    return dummy.next;
  }
};

```



#### [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)

复杂度分析：
时间复杂度 O(a+b) ： 最差情况下
O(1) ： 节点指针 A , B 使用常数大小的额外空间。

```c++
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode *A = headA, *B = headB;
        while (A != B) {
            A = A != nullptr ? A->next : headB;
            B = B != nullptr ? B->next : headA;
        }
        return A;
    }
};


```

#### [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)

```c++
class Solution {
public:
  ListNode *detectCycle(ListNode *head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
      slow = slow->next;
      fast = fast->next->next;
      if (slow == fast) {
        ListNode *slow2 = head;
        while (slow2 != slow) {
          slow2 = slow2->next;
          slow = slow->next;
        }
        return slow2;
      }
    }
    return nullptr;
  }
};

```



#### [146. LRU 缓存](https://leetcode.cn/problems/lru-cache/)

```c++
// LeetCode, LRU Cache
//时间O(logn)空间O(n)
class LRUCache {
private:
  struct CacheNode {
    int key;
    int value;
    CacheNode(int k, int v) : key(k), value(v) {}
  };

public:
  LRUCache(int capacity) { this->capacity = capacity; }
  int get(int key) {
    if (cacheMap.find(key) == cacheMap.end())
      return -1;
    //把当前访问的节点移到链表头部，并更新map中的位置
    cacheList.splice(cacheList.begin(), cacheList, cacheMap[key]);
    cacheMap[key] = cacheList.begin();
    return cacheMap[key]->value;
  }
  void set(int key, int value) {
    if (cacheMap.find(key) == cacheMap.end()) {
      if (cacheList.size() == capacity) { //删除链表尾部节点（最少访问的节点）
        cacheMap.erase(cacheList.back().key);
        cacheList.pop_back();
      }
      // 插入新节点到链表头部，并更新map中的位置
      cacheList.push_front(CacheNode(key, value));
      cacheMap[key] = cacheList.begin();
    } else {
      //更新节点值，并移到链表头部，更新map中的位置
      cacheMap[key]->value = value;
      cacheList.splice(cacheList.begin(), cacheList, cacheMap[key]);
      cacheMap[key] = cacheList.begin();
    }
  }

private:
  list<CacheNode> cacheList;
  unordered_map<int, list<CacheNode>::iterator> cacheMap;
  int capacity;
};

```



## 哈希表

#### [454. 四数相加 II](https://leetcode.cn/problems/4sum-ii/)

```c++
class Solution {
public:
    int fourSumCount(vector<int>& A, vector<int>& B, vector<int>& C, vector<int>& D) {
        unordered_map<int, int> umap; //key:a+b的数值，value:a+b数值出现的次数
        // 遍历大A和大B数组，统计两个数组元素之和，和出现的次数，放到map中
        for (int a : A) {
            for (int b : B) {
                umap[a + b]++;
            }
        }
        int count = 0; // 统计a+b+c+d = 0 出现的次数
        // 在遍历大C和大D数组，找到如果 0-(c+d) 在map中出现过的话，就把map中key对应的value也就是出现次数统计出来。
        for (int c : C) {
            for (int d : D) {
                if (umap.find(0 - (c + d)) != umap.end()) {
                    count += umap[0 - (c + d)];
                }
            }
        }
        return count;
    }
};
```

#### [383. 赎金信](https://leetcode.cn/problems/ransom-note/)

```cpp
// 时间复杂度: O(n)
// 空间复杂度：O(1)
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        int record[26] = {0};
        //add
        if (ransomNote.size() > magazine.size()) {
            return false;
        }
        for (int i = 0; i < magazine.length(); i++) {
            // 通过recode数据记录 magazine里各个字符出现次数
            record[magazine[i]-'a'] ++;
        }
        for (int j = 0; j < ransomNote.length(); j++) {
            // 遍历ransomNote，在record里对应的字符个数做--操作
            record[ransomNote[j]-'a']--;
            // 如果小于零说明ransomNote里出现的字符，magazine没有
            if(record[ransomNote[j]-'a'] < 0) {
                return false;
            }
        }
        return true;
    }
};
```



#### [18. 四数之和](https://leetcode.cn/problems/4sum/)

若用三数之和思路夹逼为O(n^ 3),哈希表缓存两数之和为O(n^ 2)

```c++
class Solution {
public:
  vector<vector<int>> fourSum(vector<int> &nums, int target) {
    vector<vector<int>> result;
    if (nums.size() < 4)
      return result;
    sort(nums.begin(), nums.end());
    unordered_multimap<int, pair<int, int>> cache;
    for (int i = 0; i + 1 < nums.size(); ++i)
      for (int j = i + 1; j < nums.size(); ++j)
        cache.insert(make_pair(nums[i] + nums[j], make_pair(i, j)));
    for (auto i = cache.begin(); i != cache.end(); ++i) {
      int x = target - i->first;
      auto range = cache.equal_range(x);
      for (auto j = range.first; j != range.second; ++j) {
        auto a = i->second.first;
        auto b = i->second.second;
        auto c = j->second.first;
        auto d = j->second.second;
        if (a != c && a != d && b != c && b != d) {
          vector<int> vec = {nums[a], nums[b], nums[c], nums[d]};
          sort(vec.begin(), vec.end());
          result.push_back(vec);
        }
      }
    }
    sort(result.begin(), result.end());
    result.erase(unique(result.begin(), result.end()), result.end());
    return result;
  }
};
//夹逼
class Solution {
public:
  vector<vector<int>> fourSum(vector<int> &nums, int target) {
    vector<vector<int>> result;
    if (nums.size() < 4)
      return result;
    sort(nums.begin(), nums.end());
    auto last = nums.end();
    for (auto a = nums.begin(); a < prev(last, 3); ++a) {
      for (auto b = next(a); b < prev(last, 2); ++b) {
        auto c = next(b);
        auto d = prev(last);
        while (c < d) {
          if (*a + *b + *c + *d < target) {
            ++c;
          } else if (*a + *b + *c + *d > target) {
            --d;
          } else {
            result.push_back({*a, *b, *c, *d});
            ++c;
            --d;
          }
        }
      }
    }
    sort(result.begin(), result.end());
    result.erase(unique(result.begin(), result.end()), result.end());
    return result;
  }
};

```



## 字符串

#### [28. 找出字符串中第一个匹配项的下标](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/) KMP

```c++
class Solution {
public:
    void getNext(int* next, const string& s) {
        int j = 0;
        next[0] = 0;
        for(int i = 1; i < s.size(); i++) {
            while (j > 0 && s[i] != s[j]) {
                j = next[j - 1];
            }
            if (s[i] == s[j]) {
                j++;
            }
            next[i] = j;
        }
    }
    int strStr(string haystack, string needle) {
        if (needle.size() == 0) {
            return 0;
        }
        int next[needle.size()];
        getNext(next, needle);
        int j = 0;
        for (int i = 0; i < haystack.size(); i++) {
            while(j > 0 && haystack[i] != needle[j]) {
                j = next[j - 1];
            }
            if (haystack[i] == needle[j]) {
                j++;
            }
            if (j == needle.size() ) {
                return (i - needle.size() + 1);
            }
        }
        return -1;
    }
};

```

#### [459. 重复的子字符串](https://leetcode.cn/problems/repeated-substring-pattern/)

```c++
class Solution {
public:
    bool repeatedSubstringPattern(string s) {
        return (s + s).find(s, 1) != s.size();
    }
};
```

#### [71. 简化路径](https://leetcode.cn/problems/simplify-path/)

给你一个字符串 path ，表示指向某一文件或目录的 Unix 风格 绝对路径 （以 '/' 开头），请你将其转化为更加简洁的规范路径。

在 Unix 风格的文件系统中，一个点（.）表示当前目录本身；此外，两个点 （..） 表示将目录切换到上一级（指向父目录）；两者都可以是复杂相对路径的组成部分。任意多个连续的斜杠（即，'//'）都被视为单个斜杠 '/' 。 对于此问题，任何其他格式的点（例如，'...'）均被视为文件/目录名称。

请注意，返回的 规范路径 必须遵循下述格式：

始终以斜杠 '/' 开头。
两个目录名之间必须只有一个斜杠 '/' 。
最后一个目录名（如果存在）不能 以 '/' 结尾。
此外，路径仅包含从根目录到目标文件或目录的路径上的目录（即，不含 '.' 或 '..'）。
返回简化后得到的 规范路径 。

```c++
class Solution {
public:
  string simplifyPath(const string &path) {
    vector<string> dirs; // 栈
    for (auto i = path.begin(); i != path.end();) {
      ++i;
      auto j = find(i, path.end(), '/');
      auto dir = string(i, j);
      if (!dir.empty() && dir != ".") { //当有连续'///'时，dir为空
        if (dir == "..") {
          if (!dirs.empty())
            dirs.pop_back();
        } else
          dirs.push_back(dir);
      }
      i = j;
    }
    stringstream out;
    if (dirs.empty()) {
      out << "/";
    } else {
      for (auto dir : dirs)
        out << '/' << dir;
    }
    return out.str();
  }
};
```



## 栈与队列

### 栈

#### [150. 逆波兰表达式求值](https://leetcode.cn/problems/evaluate-reverse-polish-notation/)

```c++
class Solution {
public:
    int evalRPN(vector<string>& tokens) {
        // 力扣修改了后台测试数据，需要用longlong
        stack<long long> st; 
        for (int i = 0; i < tokens.size(); i++) {
            if (tokens[i] == "+" || tokens[i] == "-" || tokens[i] == "*" || tokens[i] == "/") {
                long long num1 = st.top();
                st.pop();
                long long num2 = st.top();
                st.pop();
                if (tokens[i] == "+") st.push(num2 + num1);
                if (tokens[i] == "-") st.push(num2 - num1);
                if (tokens[i] == "*") st.push(num2 * num1);
                if (tokens[i] == "/") st.push(num2 / num1);
            } else {
                st.push(stoll(tokens[i]));
            }
        }

        int result = st.top();
        st.pop(); // 把栈里最后一个元素弹出（其实不弹出也没事）
        return result;
    }
};

```

#### [32. 最长有效括号](https://leetcode.cn/problems/longest-valid-parentheses/)

```c++
class Solution {
public:
  int longestValidParentheses(const string &s) {
    int max_len = 0, last = -1; // the position of the last ')'
    stack<int> lefts; // keep track of the positions of non-matching '('s
    for (int i = 0; i < s.size(); ++i) {
      if (s[i] == '(') {
        lefts.push(i);
      } else {
        if (lefts.empty()) {
          // no matching left
          last = i;
        } else {
          // find a matching pair
          lefts.pop();
          if (lefts.empty()) {
            max_len = max(max_len, i - last);
          } else {
            max_len = max(max_len, i - lefts.top());
          }
        }
      }
    }
    return max_len;
  }
};

```

#### [84. 柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/)

```c++
class Solution {
public:
  int largestRectangleArea(vector<int> &height) {
    stack<int> s;
    height.push_back(0);
    int result = 0;
    for (int i = 0; i < height.size();) {
      if (s.empty() || height[i] > height[s.top()])
        s.push(i++);
      else {
        int tmp = s.top();
        s.pop();
        result = max(result, height[tmp] * (s.empty() ? i : i - s.top() - 1));
      }
    }
    return result;
  }
};
```

### 队列

#### [239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)

```c++
//优先级队列 O(nlogn)
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int n = nums.size();
        priority_queue<pair<int, int>> q;
        for (int i = 0; i < k; ++i) {
            q.emplace(nums[i], i);
        }
        vector<int> ans = {q.top().first};
        for (int i = k; i < n; ++i) {
            q.emplace(nums[i], i);
            while (q.top().second <= i - k) {
                q.pop();
            }
            ans.push_back(q.top().first);
        }
        return ans;
    }
};

//单调队列 O(n)
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int n = nums.size();
        deque<int> q;
        for (int i = 0; i < k; ++i) {
            while (!q.empty() && nums[i] >= nums[q.back()]) {
                q.pop_back();
            }
            q.push_back(i);
        }

        vector<int> ans = {nums[q.front()]};
        for (int i = k; i < n; ++i) {
            while (!q.empty() && nums[i] >= nums[q.back()]) {
                q.pop_back();
            }
            q.push_back(i);
            while (q.front() <= i - k) {
                q.pop_front();
            }
            ans.push_back(nums[q.front()]);
        }
        return ans;
    }
};
```

#### [347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)

```c++
class Solution {
public:
  vector<int> topKFrequent(vector<int> &nums, int k) {
    unordered_map<int, int> occurrences;
    for (auto &v : nums) {
      occurrences[v]++;
    }

    // pair 第一个元素代表了该值出现的次数 第二个元素代表数组的值
    priority_queue<pair<int, int>> q;
    for (auto &[num,count] : occurrences) {
      q.emplace(count, num);
    }
    vector<int> ret;
    while (k--) {
      ret.emplace_back(q.top().second);
      q.pop();
    }
    return ret;
  }
};
```



## 二叉树

### 构造

#### 数组构造二叉树

```c++
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

//节点
struct TreeNode {
  int val;
  TreeNode *left;
  TreeNode *right;
  TreeNode() : val(0), left(nullptr), right(nullptr) {}
  TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
  TreeNode(int x, TreeNode *left, TreeNode *right)
      : val(x), left(left), right(right) {}
};

// 根据数组构造二叉树
TreeNode *construct_binary_tree(const vector<int> &vec) {
  vector<TreeNode *> vecTree(vec.size(), nullptr);
  TreeNode *root = nullptr;
  for (int i = 0; i < vec.size(); i++) {
    TreeNode *node = nullptr;
    if (vec[i] != -1)
      node = new TreeNode(vec[i]);
    vecTree[i] = node;
    if (i == 0)
      root = node;
  }
  // 遍历一遍，根据规则左右孩子赋值就可以了
  // 注意这里 结束规则是 i * 2 + 1 < vec.size()，避免空指针
  // 为什么结束规则不能是i * 2 + 2 < arr.length呢?
  // 如果i * 2 + 2 < arr.length 是结束条件
  // 那么i * 2 + 1这个符合条件的节点就被忽略掉了
  // 例如[2,7,9,-1,1,9,6,-1,-1,10] 这样的一个二叉树,最后的10就会被忽略掉
  // 遍历一遍，根据规则左右孩子赋值就可以了
  for (int i = 0; i * 2 + 1 < vec.size(); i++) {
    if (vecTree[i] != nullptr) {
      vecTree[i]->left = vecTree[i * 2 + 1];
      if (i * 2 + 2 < vec.size())
        vecTree[i]->right = vecTree[i * 2 + 2];
    }
  }
  return root;
}

// 层序打印打印二叉树
void print_binary_tree(TreeNode *root) {
  queue<TreeNode *> que;
  if (root != nullptr)
    que.push(root);
  vector<vector<int>> result;
  while (!que.empty()) {
    int size = que.size();
    vector<int> vec;
    for (int i = 0; i < size; i++) {
      TreeNode *node = que.front();
      que.pop();
      if (node != NULL) {
        vec.push_back(node->val);
        que.push(node->left);
        que.push(node->right);
      }
      // 这里的处理逻辑是为了把null节点打印出来，用-1 表示null
      else
        vec.push_back(-1);
    }
    result.push_back(vec);
  }
  for (int i = 0; i < result.size(); i++) {
    for (int j = 0; j < result[i].size(); j++) {
      cout << result[i][j] << " ";
    }
    cout << endl;
  }
}

int main() {
  // 注意本代码没有考虑输入异常数据的情况
  // 用 -1 来表示null
  vector<int> vec = {4, 1, 6, 0, 2, 5, 7, -1, -1, -1, 3, -1, -1, -1, 8};
  TreeNode *root = construct_binary_tree(vec);
  print_binary_tree(root);
}

```

#### 从前序、中序、后序遍历组合中构造二叉树（三种通解）

**不同之处一 寻找当前根节点**
这一部分总的来说是在寻找可以将左右子树划分开的根节点

前+后
首先我们可以显然知道当前根节点为pre[pre_start],并且它在后序中的位置为post_end，因此这里我们需要找到能区分左右子树的节点。
我们知道左子树的根节点为pre[pre_start+1]，因此只要找到它在后序中的位置就可以分开左右子树（index的含义）
前+中
首先我们可以显然知道当前根节点为pre[pre_start],只用找出它在中序中的位置，就可以把左右子树分开（index的含义）
中+后
首先我们可以显然知道当前根节点为post[post_end]，只用找出它在中序中的位置，就可以把左右子树分开（index的含义）
**不同之处二 左右遍历范围**
这一部分运用了一个技巧是 “两种遍历中，同一子树的节点数目是相同的”
需要说明的是在"前+后","前+中"我们都运用到了“右子树起始位置为左子树终止位置+1”，其实这个也可以运用上面这个技巧来计算出起始位置

前+后
后序遍历中，我们知道 左子树：[post_start,index], 右子树：[index+1, post_end-1]
在前序遍历中，左子树起始位置为pre_start+1,左子树个数一共有(index - post_start)个，因此左子树：[pre_start+1, pre_start+1 + (index - post_start)]
右子树起始位置为左子树终止位置+1，终止位置为pre_end，因此右子树：[ pre_start+1 + (index - post_start) + 1, pre_end]
前+中
中序遍历中，我们知道 左子树：[inorder_start,index-1], 右子树：[index+1, inorder_end]
在前序遍历中，左子树起始位置为pre_start+1,左子树一共有(index-1 - inorder_start)个，因此左子树：[pre_start+1, pre_start+1 + (index-1 - inorder_start)]
右子树起始位置为左子树终止位置+1，终止位置为pre_end，因此右子树：[ pre_start+1 + (index-1 - inorder_start) + 1, pre_end]
中+后
中序遍历中，我们知道 左子树：[inorder_start,index-1], 右子树：[index+1, inorder_end]
在后序遍历中，左子树起始位置为post_start，左子树一共有(index-1 - inorder_start)个，因此左子树：[post_start, post_start + (index-1 - inorder_start)]
右子树的终止位置为post_end - 1,右子树一共有(inorder_end - (index+1))个,因此右子树:[post_end - 1 - (inorder_end - (index+1)), post_end - 1]

```c++
class Solution {
public:
  TreeNode *buildTree(vector<int> &preorder, vector<int> &postorder) {
    return buildTree(preorder, postorder, 0, preorder.size() - 1, 0,
                     postorder.size() - 1);
  }

private:
  TreeNode *buildTree(vector<int> &preorder, vector<int> &postorder,
                      int preStart, int preEnd, int postStart, int postEnd) {
    if (preStart > preEnd || postStart > postEnd) {
      return nullptr;
    }
    TreeNode *root = new TreeNode(preorder[preStart]);
    if (preStart == preEnd) {
      return root;
    }
    //中+后
    // TreeNode *root = new TreeNode(postorder[postEnd]);
    // if (postStart == postEnd) {
    //   return root;
    // }
    int index = postStart;
    while (postorder[index] != preorder[preStart + 1]) {
      index++;
    }

    //前+中
    // int index = inStart;
    // while (inorder[index] != preorder[preStart]) {
    //   index++;
    // }
    //中+后
    // int index = inStart;
    // while (inorder[index] != postorder[postEnd]) {
    //   index++;
    // }

    root->left =
        buildTree(preorder, postorder, preStart + 1,
                  preStart + (index - postStart + 1), postStart, index);

    root->right =
        buildTree(preorder, postorder, preStart + (index - postStart + 1) + 1,
                  preEnd, index + 1, postEnd - 1);
    //前+中
    // root->left = buildTree(preorder, inorder, preStart + 1,
    //                        preStart + index - inStart, inStart, index - 1);
    // root->right = buildTree(preorder, inorder, preStart + (index - inStart) + 1,
    //                         preEnd, index + 1, inEnd);
    //中+后
    // root->left = buildTree(inorder, postorder, inStart, index - 1, postStart,
    //                        postStart + (index - inStart) - 1);
    // root->right = buildTree(inorder, postorder, index + 1, inEnd,
    //                         postStart + (index - inStart), postEnd - 1);

    return root;
  }
};
```



### 遍历

#### 递归遍历

只记录了递归用作平常使用，其他方法可用栈迭代或者**Morris遍历**

```C++
class Solution {
public:
    //前序
    void traverse(TreeNode* cur, vector<int>& vec) {
        if (cur == NULL) return;
        vec.push_back(cur->val);    // 中
        traverse(cur->left, vec);  // 左
        traverse(cur->right, vec); // 右
    }
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> result;
        traverse(root, result);
        return result;
    }
};
//中序
void traverse(TreeNode* cur, vector<int>& vec) {
    if (cur == NULL) return;
    traverse(cur->left, vec);  // 左
    vec.push_back(cur->val);    // 中
    traverse(cur->right, vec); // 右
}
//后序
void traverse(TreeNode* cur, vector<int>& vec) {
    if (cur == NULL) return;
    traverse(cur->left, vec);  // 左
    traverse(cur->right, vec); // 右
    vec.push_back(cur->val);    // 中
}
//层序
class Solution {
public:
    void order(TreeNode* cur, vector<vector<int>>& result, int depth)
    {
        if (cur == nullptr) return;
        if (result.size() == depth) result.push_back(vector<int>());
        result[depth].push_back(cur->val);
        order(cur->left, result, depth + 1);
        order(cur->right, result, depth + 1);
    }
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> result;
        order(root, result, 0);
        return result;
    }
};
```

#### [114. 二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/)

```c++
class Solution {
public:
  void flatten(TreeNode *root) { flatten(root, NULL); }

private:
  TreeNode *flatten(TreeNode *root, TreeNode *tail) {
    if (NULL == root)
      return tail;
    root->right = flatten(root->left, flatten(root->right, tail));
    root->left = NULL;
    return root;
  }
};

```

#### [117. 填充每个节点的下一个右侧节点指针 II](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node-ii/)

```c++
class Solution {
public:
  void connect(TreeLinkNode *root) {
    if (root == nullptr)
      return;
    //下一层链表头
    TreeLinkNode dummy(-1);
    //层序遍历
    for (TreeLinkNode *curr = root, *prev = &dummy; curr; curr = curr->next) {
      if (curr->left != nullptr) {
        prev->next = curr->left;
        prev = prev->next;
      }
      if (curr->right != nullptr) {
        prev->next = curr->right;
        prev = prev->next;
      }
    }
    connect(dummy.next);
  }
};

```

### 二叉搜索树(BST)

#### [96. 不同的二叉搜索树](https://leetcode.cn/problems/unique-binary-search-trees/)——一维DP

![image-20230408163159544](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230408163159544.png)

```c++
class Solution {
public:
  int numTrees(int n) {
    vector<int> f(n + 1, 0);
    f[0] = 1;
    f[1] = 1;
    for (int i = 2; i <= n; ++i) {
      for (int k = 1; k <= i; ++k)
        f[i] += f[k - 1] * f[i - k];
    }
    return f[n];
  }
};
```

#### [95. 不同的二叉搜索树 II](https://leetcode.cn/problems/unique-binary-search-trees-ii/)

```c++
class Solution {
public:
  vector<TreeNode *> generateTrees(int n) {
    if (n == 0)
      return generate(1, 0);
    return generate(1, n);
  }

private:
  vector<TreeNode *> generate(int start, int end) {
    vector<TreeNode *> subTree;
    if (start > end) {
      subTree.push_back(nullptr);
      return subTree;
    }
    for (int k = start; k <= end; k++) {
      vector<TreeNode *> leftSubs = generate(start, k - 1);
      vector<TreeNode *> rightSubs = generate(k + 1, end);
      for (auto i : leftSubs) {
        for (auto j : rightSubs) {
          TreeNode *node = new TreeNode(k);
          node->left = i;
          node->right = j;
          subTree.push_back(node);
        }
      }
    }
    return subTree;
  }
};
```

#### [98. 验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/)

```c++
class Solution {
public:
  bool isValidBST(TreeNode *root) {
    return isValidBST(root, LONG_MIN, LONG_MAX);
  }
  bool isValidBST(TreeNode *root, long long lower, long long upper) {
    if (root == nullptr)
      return true;
    return root->val > lower && root->val < upper &&
           isValidBST(root->left, lower, root->val) &&
           isValidBST(root->right, root->val, upper);
  }
};
```

#### [109. 有序链表转换二叉搜索树](https://leetcode.cn/problems/convert-sorted-list-to-binary-search-tree/)

```c++
//自底向上 O(n) 空间O(logn)
class Solution {
public:
  TreeNode *sortedListToBST(ListNode *head) {
    int len = 0;
    ListNode *p = head;
    while (p) {
      len++;
      p = p->next;
    }
    return sortedListToBST(head, 0, len - 1);
  }

private:
  TreeNode *sortedListToBST(ListNode *&list, int start, int end) {
    if (start > end)
      return nullptr;
    int mid = start + (end - start) / 2;
    TreeNode *leftChild = sortedListToBST(list, start, mid - 1);
    TreeNode *parent = new TreeNode(list->val);
    parent->left = leftChild;
    list = list->next;
    parent->right = sortedListToBST(list, mid + 1, end);
    return parent;
  }
};
```

### 完全二叉树

#### [222. 完全二叉树的节点个数](https://leetcode.cn/problems/count-complete-tree-nodes/)

```cpp
//O(n) O(n)
class Solution {
public:
    int countNodes(TreeNode* root) {
        if (root == NULL) return 0;
        return 1 + countNodes(root->left) + countNodes(root->right);
    }
};
//时间复杂度O(log n × log n)
//空间复杂度O(log n)
class Solution {
public:
    int countNodes(TreeNode* root) {
        if (root == nullptr) return 0;
        TreeNode* left = root->left;
        TreeNode* right = root->right;
        int leftDepth = 0, rightDepth = 0; // 这里初始为0是有目的的，为了下面求指数方便
        while (left) {  // 求左子树深度
            left = left->left;
            leftDepth++;
        }
        while (right) { // 求右子树深度
            right = right->right;
            rightDepth++;
        }
        if (leftDepth == rightDepth) {
            return (2 << leftDepth) - 1; // 注意(2<<1) 相当于2^2，所以leftDepth初始为0
        }
        return countNodes(root->left) + countNodes(root->right) + 1;
    }
};
```

## 回溯算法

## 贪心算法

#### [253.会议室2](https://leetcode-cn.com/problems/meeting-rooms-ii)

实际上求最大重叠数。这里使用扫描线技巧。

把start和end分别排序。当遇到start时count++，当遇到end时count--。记录过程中count的最大值。就是最大重叠数。

```c++
class Solution {
public:
    int minMeetingRooms(vector<vector<int>>& intervals) {
        map<int,int> mp;
        for(vector<int>& vec:intervals){
            mp[vec[0]]++;//会议开始，说明新增一个会议
            mp[vec[1]]--;//会议结束，说明减少一个会议
        }
        int res=0;
        int cur=0;
        //遍历map，统计所有的起始和结束会议的和
        for(auto it=mp.begin();it!=mp.end();++it){
            cur+=it->second;
            res=max(res,cur);
        }
        return res;
    }
};
	
```

#### 活动安排

与上题类似  求最大相容活动子集合，但是是求最大不重叠数。

**必须以结束时间升序排序**，若以开始时间排序，考虑一个时间段很长的情况

```c++
template <class Type> 
void GreedySelector(int n, Type s[], Type f[], bool A[]) {
  A[1] = true;
  int j = 1;
  for (int i = 2; i <= n; i++) {
    if (s[i] >= f[j]) {
      A[i] = true;
      j = i;
    } else
      A[i] = false;
  }
}
```



## 动态规划

### 0-1背包问题

```cpp
void test_1_wei_bag_problem() {
    vector<int> weight = {1, 3, 4};
    vector<int> value = {15, 20, 30};
    int bagWeight = 4;

    // 初始化
    vector<int> dp(bagWeight + 1, 0);
    for(int i = 0; i < weight.size(); i++) { // 遍历物品
        for(int j = bagWeight; j >= weight[i]; j--) { // 遍历背包容量
            dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
        }
    }
    cout << dp[bagWeight] << endl;
}

int main() {
    test_1_wei_bag_problem();
}
```

### 零钱问题

当零钱面值不小于次面值二倍时，可以直接贪心，否则只能用DP

```c++
class Solution {
    public int coinChange(int[] coins, int amount) {
        // 初始一个比较大的值
        int max = amount + 1;
        int []dp = new int[amount + 1];
        //将dp数组填充最大值，因为我们求的是最小硬币数
        Arrays.fill(dp, max);
        //初始值
        dp[0] = 0;
        //穷举每一个目标数的最小硬币数
        for(int i = 1; i <= amount; i++) {
            //对每种子问题都穷举
            for(int j = 0; j < coins.length; j++) {
                if(coins[j] <= i)
                    dp[i] = Math.min(dp[i], dp[i - coins[j]] + 1);
            }
        }
        //没有符合的情况就返回 - 1
        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```

问题变为有多少种方案

```c++
class Solution {
    public int change(int amount, int[] coins) {
        int []dp = new int[amount + 1];
        //当目标数为 0 的时候，组合数为1
        dp[0] = 1;
        //对硬币种类进行穷举
        for(int co : coins) {
            //穷举每个目标数的情况，直至所有硬币种类都列举完，最后更新的就是最终的组合数
            for(int i = co; i <= amount; i++) {
                if(i >= co) {
                    //组合数等于当前的组合数加上子问题的组合数
                    dp[i] += dp[i - co];
                }
            }
        }
        return dp[amount];
    }
}
```

## 其他

#### [137. 只出现一次的数字 II](https://leetcode.cn/problems/single-number-ii/)

```c++
class Solution {
public:
  int singleNumber(vector<int> &nums) {
    int one = 0, two = 0, three = 0;
    for (auto i : nums) {
      two |= (one & i);
      one ^= i;
      three = ~(one & two);
      one &= three;
      two &= three;
    }
    return one;
  }
};

```



#### [44. 通配符匹配](https://leetcode.cn/problems/wildcard-matching/)

```c++
class Solution {
public:
  bool isMatch(const string &s, const string &p) {
    return isMatch(s.c_str(), p.c_str());
  }

private:
  bool isMatch(const char *s, const char *p) {
    bool star = false;
    const char *str, *ptr;
    for (str = s, ptr = p; *str != '\0'; str++, ptr++) {
      switch (*ptr) {
      case '?':
        break;
      case '*':
        star = true;
        s = str, p = ptr;
        while (*p == '*')
          p++; // skip continuous '*'
        if (*p == '\0')
          return true;
        str = s - 1;
        ptr = p - 1;
        break;
      default:
        if (*str != *ptr) {
          ߎ᜿̼䙼ࡨ݈喌⇐᰸ '*'䲑ݼ᳋ັ //
              if (!star) return false;
          s++;
          str = s - 1;
          ptr = p - 1;
        }
      }
    }
    while (*ptr == '*')
      ptr++;
    return (*ptr == '\0');
  }
};
```



#### [260. 只出现一次的数字 III](https://leetcode.cn/problems/single-number-iii/)

x & -x取出x的二进制表示中最低位那个 1

```c++
class Solution {
public:
    vector<int> singleNumber(vector<int>& nums) {
        unsigned int diff = accumulate(nums.begin(), nums.end(), 0, bit_xor<int>());
        diff &= -diff;
        vector<int> res(2, 0);
        for (auto &a : nums) {
            if (a & diff) res[0] ^= a;
            else res[1] ^= a;
        }
        return res;
    }
};
```

#### [10. 正则表达式匹配](https://leetcode.cn/problems/regular-expression-matching/)

```c++
class Solution {
public:
    bool isMatch(string s, string p) {
        return isMatch(s.c_str(), p.c_str());
    }
    
    bool isMatch(const char* s, const char* p) {
        if(*p == 0) return *s == 0;
        
        auto first_match = *s && (*s == *p || *p == '.');
        
        if(*(p+1) == '*'){
            return isMatch(s, p+2) || (first_match && isMatch(++s, p));
        }
        else{
            return first_match && isMatch(++s, ++p);
        }
    }
};

```

