+++
date = '2025-08-06T15:54:00+08:00'
draft = true
title = '2025.08.06'
+++

<!--more-->

工作：


提升：
- leetcode刷刷刷
- 
其他：


好的，没有问题。数组（在 Go 中更常用的是切片 `slice`）是算法题中出现频率最高的数据结构之一。掌握其核心解题方法论至关重要。

下面我为你整理了 LeetCode 数组/切片问题的几种最核心、最常用的解法，并结合 Go 语言的特点和经典题目进行说明。

---

### LeetCode 数组/切片问题核心解法汇总 (Go 版)

数组问题的魅力在于，它看似简单，但其解法往往蕴含着对**空间复杂度**和**时间复杂度**的极致追求。很多最优解都需要在 O(1) 的额外空间内完成。

以下是五种最关键的解题思想：

1.  **双指针 (Two Pointers)**
2.  **前缀和 (Prefix Sum)**
3.  **二分查找 (Binary Search)**
4.  **哈希表 (Hash Table / Map)**
5.  **排序 (Sorting)**

---

### 一、双指针 (Two Pointers)

**核心逻辑**：
使用两个指针（通常是数组的索引）在序列中同向或反向移动，通过指针的移动和元素比较来解决问题。这种方法可以将某些需要 O(n²) 暴力求解的问题优化到 O(n)。

双指针有三种经典模式：

#### 1. 左右指针 (对撞指针)

**模式**：一个指针 `left` 从数组头部开始，一个指针 `right` 从数组尾部开始，两者相向移动，直到相遇或交错。
**适用场景**：常用于**有序数组**中查找数对、反转数组等。

**示例题目：[167. 两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)**
> 在一个升序排列的整数数组 `numbers` 中，找到两个数使得它们相加之和等于一个特定的目标值 `target`。

**Go 代码实现**：
```go
func twoSum(numbers []int, target int) []int {
    // 初始化左右指针
    left, right := 0, len(numbers)-1

    for left < right {
        sum := numbers[left] + numbers[right]

        if sum == target {
            // 题目要求返回的索引是从 1 开始
            return []int{left + 1, right + 1}
        } else if sum < target {
            // 和太小，说明左边的数太小，需要向右移动 left 指针来增大和
            left++
        } else { // sum > target
            // 和太大，说明右边的数太大，需要向左移动 right 指针来减小和
            right--
        }
    }
    return []int{-1, -1} // 未找到
}
```
**代码解析**：
利用数组的有序性。如果 `sum < target`，移动 `left` 指针是唯一可能让 `sum` 变大的方式。反之，如果 `sum > target`，移动 `right` 指针是唯一可能让 `sum` 变小的方式。通过这种方式，我们在一次遍历中就找到了答案。

#### 2. 快慢指针

**模式**：一个快指针 `fast` 在前探路，一个慢指针 `slow` 在后。`fast` 每次移动一步，`slow` 根据一定条件移动。
**适用场景**：原地修改数组，如删除/移动元素、数组去重等。

**示例题目：[26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)**
> 给你一个有序数组 `nums` ，请你**原地**删除重复出现的元素，使每个元素只出现一次，返回删除后数组的新长度。

**Go 代码实现**：
```go
func removeDuplicates(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    // slow 指针指向下一个将要被赋值的唯一元素的位置
    // fast 指针用于遍历整个数组
    slow := 1 
    for fast := 1; fast < len(nums); fast++ {
        // 如果 fast 指向的元素与它前一个元素不同
        // 说明遇到了一个新的、不重复的元素
        if nums[fast] != nums[fast-1] {
            // 将这个新元素放到 slow 指针的位置
            nums[slow] = nums[fast]
            // slow 指针后移，准备接收下一个唯一元素
            slow++
        }
    }
    return slow // slow 的值即为不重复元素的个数
}
```
**代码解析**：
`slow` 指针维护了一个 "无重复元素" 区间的右边界。`fast` 指针负责探索新元素。当 `fast` 发现一个与之前元素不同的新元素时，就把它复制到 `slow` 的位置，然后 `slow` 前进。整个过程只遍历一次数组，且是原地修改。

#### 3. 滑动窗口

**模式**：由 `left` 和 `right` 指针构成一个 "窗口"。`right` 指针负责扩大窗口，`left` 指针负责收缩窗口。
**适用场景**：求解满足特定条件的**连续子数组**问题，如 "最小长度"、"最大长度"、"出现次数" 等。

**示例题目：[209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)**
> 给定一个含有 `n` 个正整数的数组和一个正整数 `target` 。找出该数组中满足其和 `≥ target` 的长度最小的 **连续子数组**。

**Go 代码实现**：
```go
import "math"

func minSubArrayLen(target int, nums []int) int {
    left, sum := 0, 0
    minLength := math.MaxInt32

    for right := 0; right < len(nums); right++ {
        // 1. right 指针向右移动，扩大窗口，累加窗口内的和
        sum += nums[right]

        // 2. 当窗口内的和满足条件时，开始收缩窗口
        for sum >= target {
            // 更新最小长度
            currentLength := right - left + 1
            if currentLength < minLength {
                minLength = currentLength
            }
            // 3. left 指针向右移动，收缩窗口
            sum -= nums[left]
            left++
        }
    }

    if minLength == math.MaxInt32 {
        return 0 // 未找到满足条件的子数组
    }
    return minLength
}
```
**代码解析**：
这是一个典型的滑动窗口模板。外层循环控制 `right` 指针，不断扩大窗口。内层循环检查当前窗口是否满足条件，如果满足，就更新结果，并移动 `left` 指针收缩窗口，以寻找可能存在的更短的满足条件的窗口。

---

### 二、前缀和 (Prefix Sum)

**核心逻辑**：
通过预处理，创建一个 `prefixSum` 数组，其中 `prefixSum[i]` 存储原数组从 `0` 到 `i-1` 的元素之和。这样，任意区间 `[i, j]` 的和就可以在 O(1) 时间内通过 `prefixSum[j+1] - prefixSum[i]` 计算得出，避免了重复计算。

**适用场景**：需要频繁计算数组某个区间的和。

**示例题目：[560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)**
> 给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 该数组中和为 `k` 的**连续子数组**的个数。

**Go 代码实现** (前缀和 + 哈希表):
```go
func subarraySum(nums []int, k int) int {
    // map[prefixSum]count: 存储某个前缀和出现的次数
    prefixSumCount := make(map[int]int)
    // base case: 前缀和为 0 出现了 1 次 (表示空数组)
    prefixSumCount[0] = 1

    count := 0
    currentSum := 0 // 当前从 0 到 i 的前缀和

    for _, num := range nums {
        currentSum += num
        
        // 我们要找的是一个前缀和 `prevSum`，使得 `currentSum - prevSum = k`
        // 这等价于 `prevSum = currentSum - k`
        // 我们在哈希表中查找之前出现过多少次 `currentSum - k`
        if times, found := prefixSumCount[currentSum-k]; found {
            count += times
        }
        
        // 将当前的前缀和存入哈希表，或更新其出现次数
        prefixSumCount[currentSum]++
    }
    
    return count
}
```
**代码解析**：
这个问题是前缀和思想的绝佳应用。我们想找 `sum(nums[i:j]) == k`。这等价于 `prefixSum[j] - prefixSum[i-1] == k`。
我们变换一下公式：`prefixSum[i-1] = prefixSum[j] - k`。
当我们遍历数组，计算到索引 `j` 的当前前缀和 `currentSum` (`prefixSum[j]`) 时，我们只需要查找在 `j` 之前，有多少个 `i-1` 满足它们的前缀和等于 `currentSum - k`。使用哈希表可以 O(1) 地完成这个查找。

---

### 三、二分查找 (Binary Search)

**核心逻辑**：
在一个**有序**或具有**单调性**的数据集合中进行查找。每次将查找空间缩小一半，时间复杂度为 O(log n)。

**适用场景**：
1.  有序数组的查找。
2.  问题答案具有单调性，可以对 "答案" 进行二分查找。

**示例题目：[704. 二分查找](https://leetcode.cn/problems/binary-search/)**
> 给定一个 `n` 个元素有序的（升序）整型数组 `nums` 和一个目标值 `target` ，写一个函数搜索 `nums` 中的 `target`，如果目标值存在返回下标，否则返回 -1。

**Go 代码实现**：
```go
func search(nums []int, target int) int {
    left, right := 0, len(nums)-1

    for left <= right {
        // mid := (left + right) / 2 // 可能溢出
        mid := left + (right-left)/2 // 更安全的写法

        if nums[mid] == target {
            return mid
        } else if nums[mid] < target {
            // 中间值太小，目标在右侧
            left = mid + 1
        } else { // nums[mid] > target
            // 中间值太大，目标在左侧
            right = mid - 1
        }
    }
    
    return -1 // 未找到
}
```
**代码解析**：
这是最标准的二分查找模板。关键在于 `while left <= right` 循环条件和每次循环后如何收缩 `left` 和 `right` 的边界。`mid` 的计算方式 `left + (right-left)/2` 是为了防止 `left + right` 在某些语言中可能出现的整数溢出，是最佳实践。

---

### 四、哈希表 (Hash Table / Map)

**核心逻辑**：
利用键值对 (key-value) 存储信息，提供平均 O(1) 时间复杂度的插入和查找操作。这是一种典型的 "空间换时间" 策略。

**适用场景**：需要快速查找一个元素是否存在、或需要统计元素出现的频率。

**示例题目：[1. 两数之和](https://leetcode.cn/problems/two-sum/)**
> 给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 和为目标值 `target` 的那两个整数，并返回它们的数组下标。

**Go 代码实现**：
```go
func twoSum(nums []int, target int) []int {
    // map[value]index: 存储数组中值和其对应的索引
    valMap := make(map[int]int)

    for i, num := range nums {
        // 计算需要找到的另一个数
        complement := target - num
        
        // 在 map 中查找 complement 是否存在
        if j, found := valMap[complement]; found {
            // 如果存在，说明找到了答案
            return []int{j, i}
        }
        
        // 如果不存在，将当前数字和它的索引存入 map
        // 以便后续的数字进行匹配
        valMap[num] = i
    }
    
    return nil // 未找到
}
```
**代码解析**：
我们遍历数组，对于每个元素 `num`，我们计算出它的 "补数" `complement`。然后，我们不去数组里查找 `complement`（这将是 O(n) 操作），而是去哈希表里查找。如果哈希表里有，说明 `complement` 在当前元素 `num` 之前已经出现过，直接返回结果。如果没找到，就把当前 `num` 和它的索引 `i` 存入哈希表，供后面的元素查询。这样，总时间复杂度就是 O(n)。

---

### 五、排序 (Sorting)

**核心逻辑**：
排序本身不是目的，而是手段。将无序的数组变得有序后，往往能解锁更高效的解法，比如双指针、二分查找等。

**适用场景**：当元素的相对顺序不重要，或者问题的解决依赖于元素的有序性时。

**示例题目：[56. 合并区间](https://leetcode.cn/problems/merge-intervals/)**
> 以数组 `intervals` 表示若干个区间的集合，其中 `intervals[i] = [start_i, end_i]`。请你合并所有重叠的区间，并返回一个不重叠的区间数组。

**Go 代码实现**：
```go
import "sort"

func merge(intervals [][]int) [][]int {
    if len(intervals) <= 1 {
        return intervals
    }

    // 1. 关键第一步：按区间的起始位置排序
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })

    merged := [][]int{intervals[0]} // 初始化结果，放入第一个区间

    for i := 1; i < len(intervals); i++ {
        lastMerged := merged[len(merged)-1]
        current := intervals[i]

        // 2. 比较当前区间与已合并的最后一个区间
        if current[0] <= lastMerged[1] {
            // 有重叠，合并区间。更新最后一个合并区间的 end
            // end 取两者 end 的最大值
            if current[1] > lastMerged[1] {
                merged[len(merged)-1][1] = current[1]
            }
        } else {
            // 无重叠，将当前区间直接加入结果集
            merged = append(merged, current)
        }
    }
    
    return merged
}
```
**代码解析**：
这个问题的核心在于，如果不排序，判断任意两个区间是否重叠非常麻烦。一旦我们按区间的起始位置 `start` 进行了排序，问题就变得简单了：一个区间 `i` 只可能与它前面紧邻的那个已合并好的区间 `i-1` 发生重叠。我们只需要遍历排序后的数组，逐个比较和合并即可。排序将一个复杂的问题转化为了一个简单的线性扫描问题。

---

### 总结

-   遇到**有序数组**，优先考虑**二分查找**和**双指针**。
-   遇到**连续子数组**问题（求和、求最值），优先考虑**滑动窗口**和**前缀和**。
-   遇到查找数对、检查元素是否存在/频率，优先考虑**哈希表**。
-   遇到原地修改数组，优先考虑**双指针**（尤其是快慢指针）。
-   如果问题不依赖元素原始顺序，或者排序后能大大简化问题，**排序**就是你的第一步。

在实际解题中，这些方法经常会组合使用，例如 "前缀和 + 哈希表"、"排序 + 双指针"。理解每种方法的底层逻辑和适用场景，是高效解决数组问题的关键。