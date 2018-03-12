---
layout: post
title:  "图解算法"
date:   2018-03-11 19:45:00

categories: other
tags: learningnote
author: "Victor"
---

## 基础知识
### 大O表示法

我们谈论算法速度时候，说的是随着输入的增加，其运行时间以什么样的速度增加。

大O表示法被用来描述一个算法的性能或复杂度。指出了在最糟糕情况下算法的运行时间。

从快到慢依次排列：

* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/7b8d37856ce892bd7ee9a9f58ffea0febee5e9bc) 常量时间
* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/6c35c6c21c30a5643d3100b7993f907b58cf79cf) 对数时间
* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/8bc936ac28af050e96d7262b89eb11af36bcc958) 线性时间
* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/b4a9cde84a808a1c8b6658032611f99e7fa0bb13) 二次方时间
* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/a830bd28003689e75eb85a330c2017413dcceb98) 指数时间
* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/f5a7957bb5d704181437f3fcf22b257ecbe699a7) 阶乘时间

[大O小抄](http://bigocheatsheet.com/) 提供了常用的算法时间复杂度，并以图表的形式呈现。可以看一下 [维基百科](https://zh.wikipedia.org/wiki/%E5%A4%A7O%E7%AC%A6%E5%8F%B7)。

## 数据结构

## 算法
### 二分法查找 binary search


* 场景：当数据量很大适宜采用该方法
* 前提：数据需是排好序的
* 时间复杂度：![](https://wikimedia.org/api/rest_v1/media/math/render/svg/6c35c6c21c30a5643d3100b7993f907b58cf79cf)

算法过程：

1. 确定查找范围 `front = 0，end = N-1`，计算中项 `mid = (front+end)/2`
2. 若 `a[mid] == x` 或 `front >= end` 则结束查找；否则，向下继续
3. 若 `a[mid] < x` ,说明待查找的元素值只可能在比中项元素大的范围内，则把 `mid+1` 的值赋给 front，并重新计算 mid，转去执行步骤2；若 `a[mid] > x`，说明待查找的元素值只可能在比中项元素小的范围内，则把 `mid-1` 的值赋给 end，并重新计算 mid，转去执行步骤2。

```ruby
def binary_search(arr, i)
  low, high = 0, arr.size - 1
  while (low < high) do
    mid = (low + high)/2
    if arr[mid] < i
      low = mid + 1
    elsif arr[mid] > i
      high = mid - 1
    else
      return mid
    end
  end
end

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
