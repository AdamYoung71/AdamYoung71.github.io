---
layout:	post
title:  "Leetcode数组专题"
date: 2022-2-7 10:00
author: "Adam"
header-img: "img/post-bg-cyberwar.jpg"
catalog: true
tags:
   - Leetcode
---

### 1. 二分查找

#### [704.二分查找](https://leetcode-cn.com/problems/binary-search/)

给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。提示：必须以`log(n)`时间复杂度完成。

条件：

- 所有num都是唯一的。
- Nums 升序排列

思路：先考虑边界，如果第一个元素比target大，或最后一个元素比target小，直接返回-1。使用二分法，先初始化le ft和right为列表左右边界，找到中间值mid与target比大小，如果mid>target则说明target在左区间，right更新为mid左边第一个。如果mid<target说明其在右区间，left更新为mid右边第一个。循环条件为left小于等于right，等于时left，right，mid都是最终结果，代码如下：

```c++
int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;
        int mid = 0;
        if(target>nums[right] || target < nums[0]) return -1;
        while (left<=right)
        {
            mid = left + (right - left) / 2; //或 >> 1循环右移
            if(nums[mid]>target) {
                right = mid - 1;
            } else if(nums[mid]<target)
            {
                left = mid + 1;
            } else {
                return mid;
            }
        }
        return -1;
    }
```

结果：![image-20220207105826044](https://tva1.sinaimg.cn/large/008i3skNgy1gz55dwxoh8j30r005ydge.jpg)

#### [35. 查找插入位置](https://leetcode-cn.com/problems/search-insert-position/description/)

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 `O(log n)` 的算法。

- nums是无重复元素的升序数组。

思路：在元素存在的情况下与上一题相同，所以只需要再考虑元素不存在的情况。考虑上一题的程序，最后的`else`就不能用了，不然会返回答案两边。最后的`return -1`也需要修改，最终运行到`left=right`时，如果目标值存在，左右都是目标，如果目标不存在，`mid=left=right`如果目标大于`nums[left]`则`left=left+1=right+1`就是`left`右侧第一个，这时`left>right`推出循环，返回`left`；如果目标小于`nums[left]`则`right=right-1=left-1`就是left左边一个，此时插入位置仍为`left`所以同样返回`left`。代码如下：

```c++
 int searchInsert(vector<int>& nums, int target) {
        short n = nums.size();
        short l=0,r=n-1;
        while(l<=r){
            short mid=l+(r-l)/2;
            if(nums[mid]<target)
                l=mid+1;
            else r=mid-1;
        }
        return l;
    }
```



结果：时间复杂度：`O(logn)`，空间复杂度：`O(1)`![image-20220207164239519](https://tva1.sinaimg.cn/large/008i3skNgy1gz5fc5ldp8j30ra06o0tb.jpg)

#### 34. [在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 target，返回 [-1, -1]。

进阶：

你可以设计并实现时间复杂度为 O(log n) 的算法解决此问题吗？

思路一：遍历法，找到后开始计数。时间复杂度`O(n)`，最坏情况为都是target。

```c++
vector<int> searchRange(vector<int>& nums, int target) {
        int first, last;
        if(nums.empty()) return {-1, -1};
        for(int iter = 0;iter<nums.size();iter++) {
            if(nums[iter]==target) {//找到target
                first = iter;
                last = iter;
                while((last+1<=nums.size() - 1) && (nums[last+1] == nums[first])) {
                    last++; //这里要注意条件不要使得数组越界
                }
                return {first, last};
            }
        }
        return {-1, -1}; //没找到
    }
```



结果：竟然还不错![image-20220207171954179](https://tva1.sinaimg.cn/large/008i3skNgy1gz5gervovzj30rg06yq3g.jpg)

思路二：二分查找，想法是二分的`left`和`right`分别定位到target出现的第一个和最后一个的位置，可以在每次产生新的`mid`后，分别在其左右再进行二分查找，最终找到最边上的两个target，即为`left, right`。





#### [69.x 的平方根](https://leetcode-cn.com/problems/sqrtx/)

给你一个非负整数 x ，计算并返回 x 的 算术平方根 。

由于返回类型是整数，结果只保留整数部分 ，小数部分将被舍去 。

注意：不允许使用任何内置指数函数和算符，例如 pow(x, 0.5) 或者 x ** 0.5 。



简单思路：循环。时间复杂度：`O(n)`

```c++
int mySqrt(int x) {
        if(x == 0) {return 0;}
        if(x == 1) {return 1;}
        int num = 1;
        while(x / num >= num){
            if(x/(num+1) < (num+1))
            {return num;}
            num++;
        }
        return num;
    }
```

二分法：。

```c++
int mySqrt(int x) {
        if(x == 0) return 0;
        if(x == 1) return 1;	//单独处理，否则会出现除数为0
        int left = 0;
        int right = x ;
        while (left<=right) 
        {
             int mid = left + ((right - left) / 2);
             if(x/mid > mid) { //避免溢出
                 left = mid + 1;
             } else if(x/mid < mid) {
                 right = mid - 1;
             } else {
                 return mid; //正好是mid的平方
             }
        }
        return left - 1; //left是找到的第一个大于的sqrt（x）的值，所以要返回left-1
        
    }
```

结果：时间复杂度：`O(logn)`，空间复杂度：`O(1)`![image-20220207214842960](https://tva1.sinaimg.cn/large/008i3skNgy1gz5o6i6xxzj30ry05q3z1.jpg)

#### [367. 有效的完全平方数](https://leetcode-cn.com/problems/valid-perfect-square/)

给定一个 正整数 num ，编写一个函数，如果 num 是一个完全平方数，则返回 true ，否则返回 false 。

进阶：不要 使用任何内置的库函数，如  sqrt 。

这一题和上一题基本一样，只需要根据题意做一些小小的修改即可。需要注意的点是，由于为了避免溢出判断时候使用除法而非乘法，就导致可能会出现5/2=2这种情况，所以需要多判断一下。

```c++
bool isPerfectSquare(int num) {
        if(num == 0) return false;
        if(num == 1) return true;	//单独处理，否则会出现除数为0
        int left = 0;
        int right = num ;
        while (left<=right) 
        {
             int mid = left + ((right - left) / 2);
             if(num/mid > mid) { //避免溢出
                 left = mid + 1;
             } else if(num/mid < mid) {
                 right = mid - 1;
             } else {
                 if(num/mid != (num-1)/mid) {	//这里要注意，由于整数除法向下取整，不判断会有问题
                    return true; //正好是mid的平方
                 }
                 
                 return false;
             }
        }
        return false;
    }
```

结果：时间复杂度：`O(logn)`，空间复杂度：`O(1)`

![image-20220207223140439](https://tva1.sinaimg.cn/large/008i3skNgy1gz5pf6xgsuj30ty05umxo.jpg)



### 2. 移除元素

#### [27. 移除元素](https://leetcode-cn.com/problems/remove-element/)

给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。

不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。



思路：实际上不用改变数组的长度，只需要把要移除的元素放到最后即可。

方法一：暴力算法，双循环，循环数组，找到target后循环将其之后的数都往前移一位。`O(n^2)`

方法二：快慢指针，是常用的一种用一个循环就可以完成双循环工作的思路。快指针完整遍历数组，慢指针为最终剩余元素数。开始时，快慢指针都指向数组头，在没有遇到目标时，快慢指针同时增加；当遇到目标时，快指针继续向前，慢指针不动，并将自己所指元素指向快指针所指元素。继续向后，如果没有遇到目标，快慢指针同时增加，且若快慢指针不想等，就将慢指针值改为快指针。若遇到目标，慢指针不动，快指针增加，以此类推，当快指针遍历数组后，慢指针指向元素即为新的数组边界。

```c++
int removeElement(vector<int>& nums, int val) {
        int slowIndex = 0;
        for (int fastIndex = 0; fastIndex<nums.size(); fastIndex++) {
            if(val != nums[fastIndex]) {	//只有快指针指向的不是目标时慢指针增加。
                nums[slowIndex] = nums[fastIndex];
                slowIndex++;
            }
        }
        return slowIndex;
    }
```

结果：时间复杂度：O(n)，空间复杂度：O(1)

![image-20220208224316153](https://tva1.sinaimg.cn/large/008i3skNgy1gz6vdldovwj30rk05omxp.jpg)

#### [26.删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/description/)

给你一个有序数组 nums ，请你 原地 删除重复出现的元素，使每个元素 只出现一次 ，返回删除后数组的新长度。

不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 `O(1)` 额外空间的条件下完成。

- nums升序排列

思路：重点就是nums按升序排列，不然一定会需要`O(n)`的空间来存放每一个出现过的元素。利用上一题的快慢指针法，初始状态：快指针指向第二个元素，慢指针指向第一个元素。如果快指针的值等于慢指针，快指针右移，慢指针不动。若不等，慢指针右移，所指等于快指针，快指针右移。缺点：需要单独处理数组长度为0和1。

```c++
int removeDuplicates(vector<int>& nums) {
        int slowIndex = 0;
        if(nums.size() == 1) return 1;
        if(nums.empty()) return 0;
        for (int fastIndex = 1;fastIndex<nums.size();fastIndex++) {
            if(nums[slowIndex] != nums[fastIndex]) {
                slowIndex++;
                nums[slowIndex] = nums[fastIndex];
            } 
        }
        return slowIndex + 1;
    }
```

结果：时间复杂度`O(n)`，空间复杂度`O(1)`。

![image-20220208232309980](https://tva1.sinaimg.cn/large/008i3skNgy1gz6wj30eqhj30se05ujrz.jpg)

#### [283.移动零](https://leetcode-cn.com/problems/move-zeroes/)

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。



思路：由于要把0移动到最后，一个想法是使用两个指针，一个从前遍历数组，一个在后面做置换。当左指针指向0时，如果右指针不为零则可以直接交换，如果不为0则右指针向左移一位，直到左指针等于右指针时整个数组都被遍历且0都被转移到最右侧。但是结果需要保持顺序所以此想法不成立，那就继续用快慢指针。由于需要移动0，所以将非0数移到左边后，只需要把剩下的元素赋值为0即可。

```c++
void moveZeroes(vector<int>& nums) {
        int slow = 0;
        for(int fast=0;fast<nums.size();fast++) {
            if(nums[fast] != 0) {
                nums[slow] = nums[fast];
                slow++;
            }
        }
        for(int iter = slow;iter <nums.size();iter++) {
            nums[iter] =0;
        }
    }
```

结果：复杂度：`O(n)`,`O(1)`

![image-20220208235807303](https://tva1.sinaimg.cn/large/008i3skNgy1gz6xjgfufzj30s205eq3h.jpg)

#### [844.比较含退格的字符串](https://leetcode-cn.com/problems/backspace-string-compare/description/)

给定 s 和 t 两个字符串，当它们分别被输入到空白的文本编辑器后，请你判断二者是否相等。# 代表退格字符。

如果相等，返回 true ；否则，返回 false 。

注意：如果对空文本输入退格字符，文本继续为空。



思路：这一题看起来很新颖，仔细想一下其实就是上面题的变型，思路也很简单，仍然使用快慢指针法，快指针遍历字符串，慢指针遇到#就回退（注意越界）。最终得到manzhizhen

```c++
string procString(string s) {
        int left = 0;
        for(int right=0;right<s.size();right++) {
           if(s[right] == '#') {
               s.erase(right,1);
               if(right == 0) right--;
               if(right-1>=0) {
                   s.erase(right-1,1);
                   right = right  - 2;
               }
           }
        }
        return s;
}

    bool backspaceCompare(string s, string t) {
        s = procString(s);
        t = procString(t);
        return s==t;

        }
```

结果：复杂度：`O(n),O(1)`

![image-20220209191953891](https://tva1.sinaimg.cn/large/008i3skNgy1gz7v4bt6wfj30qy068jrw.jpg)

### 4. 有序数组的平方

#### [977.有序数组的平方 Easy](https://leetcode-cn.com/problems/squares-of-a-sorted-array/description/)

给你一个按 **非递减顺序** 排序的整数数组 `nums`，返回 **每个数字的平方** 组成的新数组，要求也按 **非递减顺序** 排序。

思路：最简单的想法就是先每个数平方，然后排序。这样的复杂度大约是`O(n+nlogn)`（如果用快排）。由于nums是有序的，所以最大值只能出现在数组的两边，利用这个信息我们就可以使用双指针法来解决。如果左右的任意一边的平方大于另一边，则大的那边往中间移动，小的那边不动，将大的那边的平方放入新数组右侧。

```c++
vector<int> sortedSquares(vector<int>& nums) {
        vector<int> sorted(nums.size());
        int n = nums.size() - 1;
        int left = 0;
        int right = nums.size() - 1;
        while(left<=right) {//相等也需要考虑，相等时为最后一个数
            int leftSquare = nums[left]*nums[left];
            int rightSquare = nums[right]*nums[right];
            if(leftSquare>rightSquare) {
                sorted[n] = leftSquare;
                left++;
                n--;
            } else {
                sorted[n] = rightSquare;
                right--;
                n--;
            }
        }
        return sorted;
    }
```

结果：复杂度：`O(n),O(n)`



### 5. 长度最小的数组

#### [209.长度最小的子数组 Medium](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

给定一个含有 n 个正整数的数组和一个正整数 target 。

找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。



思路：这一题又是一个重要思路：滑动窗口法，看起来很像双指针，**就是不断的调节子序列的起始位置和终止位置，从而得出我们要想的结果**。当窗口满足要求时，记录子串长度和

```c++
int minSubArrayLen(int target, vector<int>& nums) {
        int result = __INT32_MAX__;
        int i=0; //窗口左侧
        int subSum = 0; //子串和
        int subLen = 0; //子串长度
        for(int j=0;j<nums.size();j++) {
            subSum += nums[j];
            while(subSum >= target) {
                subLen = (j - i + 1); //获取当前子串长度
                result = result < subLen ? result : subLen;
                subSum -= nums[i]; //去掉窗口左侧元素
                i++; //左侧右移
                //可以写成一句：subSum -= nums[i++];
            }
        }
        //如果result没有被改变则返回0
        return result == __INT32_MAX__ ? 0 : result;
    }
```

本题收获：

- 滑动窗口思想。
- 使用MAX赋值进行最后的输出判断。
- 命令组合（其实也没必要，可读性更重要）

#### [904. 水果成篮](https://leetcode-cn.com/problems/fruit-into-baskets/)

你正在探访一家农场，农场从左到右种植了一排果树。这些树用一个整数数组 fruits 表示，其中 fruits[i] 是第 i 棵树上的水果 种类 。

你想要尽可能多地收集水果。然而，农场的主人设定了一些严格的规矩，你必须按照要求采摘水果：

你只有 两个 篮子，并且每个篮子只能装 单一类型 的水果。每个篮子能够装的水果总量没有限制。
你可以选择任意一棵树开始采摘，你必须从 每棵 树（包括开始采摘的树）上 恰好摘一个水果 。采摘的水果应当符合篮子中的水果类型。每采摘一次，你将会向右移动到下一棵树，并继续采摘。
一旦你走到某棵树前，但水果不符合篮子的水果类型，那么就必须停止采摘。
给你一个整数数组 fruits ，返回你可以收集的水果的 最大 数目。



思路：这题描述地花里胡哨，但看完例子仔细想一下就可以发现这一题就是上一题的变体，目标是要找出fruits中最长的只含有两种元素的子串长度。依然使用滑动窗口，如果窗口右侧元素不是两个篮子之一，则更新长度，将两个篮子分别改为右侧-1和右侧。

- 考虑右边一颗不在篮子里
- 如果当前只用了一个篮子（前面的都为一种），则左侧不变，第二个篮子为右侧。
- 如果两个篮子都用了，第二个篮子变为窗口右侧，第一个篮子变为第二个篮子，窗口左侧移动到原第二个篮子第一次出现的位置

```c++
int totalFruit(vector<int>& tree) {
        if (tree.empty()) return 0;
        int n = tree.size();
        int head{0},tail{0};
        unordered_map<int,int>table;
        int ans{0};
        while (tail<n){
            table[tree[tail++]]++;
            while (table.size()>2){
                if (table[tree[head]]==1){
                    table.erase(tree[head]);
                }else{
                    table[tree[head]]--;
                }
                head++;
            }
            ans = max(ans,tail-head);
        }
        return ans;
    }
```



#### [76.最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)[](https://leetcode-cn.com/problems/minimum-window-substring/)

给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。

注意：

    对于 t 中重复字符，我们寻找的子字符串中该字符数量必须不少于 t 中该字符数量。
    如果 s 中存在这样的子串，我们保证它是唯一的答案。



思路：



### 6. 螺旋矩阵（出现率较高）

#### [59.螺旋矩阵](https://leetcode-cn.com/problems/spiral-matrix-ii/)

给你一个正整数 `n` ，生成一个包含 `1` 到 `n2` 所有元素，且元素按顺时针顺序螺旋排列的 `n x n` 正方形矩阵 `matrix` 。

<img src="/Users/tingyi/Library/Application Support/typora-user-images/image-20220218205853415.png" alt="image-20220218205853415" style="zoom:50%;" />

思路：这种类型的题目不涉及什么算法，只要能正确描述流程即可。难点在于正确分析边界

```c++
 vector<vector<int>> res(n, vector<int>(n, 0)); // 使用vector定义一个二维数组
        int startx = 0, starty = 0; // 定义每循环一个圈的起始位置
        int loop = n / 2; // 每个圈循环几次，例如n为奇数3，那么loop = 1 只是循环一圈，矩阵中间的值需要单独处理
        int mid = n / 2; // 矩阵中间的位置，例如：n为3， 中间的位置就是(1，1)，n为5，中间位置为(2, 2)
        int count = 1; // 用来给矩阵中每一个空格赋值
        int offset = 1; // 每一圈循环，需要控制每一条边遍历的长度
        int i,j;
        while (loop --) {
            i = startx;
            j = starty;

            // 下面开始的四个for就是模拟转了一圈
            // 模拟填充上行从左到右(左闭右开)
            for (j = starty; j < starty + n - offset; j++) {
                res[startx][j] = count++;
            }
            // 模拟填充右列从上到下(左闭右开)
            for (i = startx; i < startx + n - offset; i++) {
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
            offset += 2;
        }

        // 如果n为奇数的话，需要单独给矩阵最中间的位置赋值
        if (n % 2) {
            res[mid][mid] = count;
        }
        return res;
```

