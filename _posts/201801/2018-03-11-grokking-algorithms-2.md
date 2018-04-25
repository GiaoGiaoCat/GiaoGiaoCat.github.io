---
layout: post
title:  "图解算法 2"
date:   2018-03-12 19:45:00

categories: other
tags: learningnote algorithms
author: "Victor"
---

## 思路

### 分而治之 divide and conquer, D&C

D&C 并非算法，而是解决问题的思路。

**编写涉及数组的递归函数时，基线条件通常都是数组为空或只包含一个元素。**

1. 找出基线条件，这种条件必须尽可能简单
2. 不断将问题分解或者说缩小规模，直到符合基线条件（每次递归调用都必须缩小问题的规模）

```ruby
# 计算数组之合
def sum(ary)
  a = ary.shift
  return a if ary.size.zero?
  a + sum(ary)
end
puts sum([1,2,3])

# 找出数组中的最大值
def find_biggest(arr)
  return arr[0] if arr.size <= 1
  if arr[0] < arr[1]
    arr.delete_at(0)
  else
    arr.delete_at(1)
  end
  find_biggest(arr)
end
arr = [617, 902, 159, 7, 579, 693, 525, 805, 871, 1002, 748, 473, 161, 271, 129, 632, 546, 894, 162, 637, 313]
puts find_biggest(arr)

# 计算数组元素个数
def calculate_count(arr)
  return 0 if arr.size.zero?
  arr.shift
  1 + calculate_count(arr)
end
arr = [1,2,3]
puts calculate_count(arr)
```

### 递归 recursive

* 程序调用自身的编程技巧称为递归
* 递归只是让解决方案更清晰，并没有性能上的优势。有时，循环的性能可能更好。
* 每个递归函数都有两部分：
  * 基线条件 base case 函数何时不再调用自己
  * 递归条件 recursive case 函数何时调用自己

```ruby
# 递归实现二分法查找
def binary_search(array, key, low=0, high=array.size-1)
  return -1 if low > high
  mid = (low + high) / 2
  return mid if array[mid]==key
  if array[mid] > key
    high = mid - 1
  else
    low = mid + 1
  end
  binary_search(array, key, low, high)
end
```

### 贪婪

每次都选择局部最优解，最终得到的就是全局最优解。

在有些情况下，完美是优秀的敌人。有时候，只要找到一个大致解决问题的算法，此时贪婪算法正好可派上用场，因为实现容易，得到的结果又与正确结果相当接近。
