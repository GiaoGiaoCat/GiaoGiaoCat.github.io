---
layout: post
title:  "图解算法 3"
date:   2018-03-13 19:45:00

categories: other
tags: learningnote algorithms
author: "Victor"
---

## 查找算法
### 二分法查找 Binary Search

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
```

### 广度优先搜索 breadth-first search

能够找出两样东西之间的最短距离。主要用来解决最短路径问题 shortest-path problem，用于图的算法。

* 场景：编写国际跳棋AI，计算最少走多少步就可以获胜
* 场景：编写拼写检查器，计算最少编辑多少个地方就可以将错误的单词改正

它解决两类问题。

1. 从节点 A 出发，有前往节点 B 的路径吗
2. 从节点 A 出发，前往节点 B 的哪条路径最短

时间复杂度：O(V+E)，V vertice 为顶点，E 为边数

1. 你从起点沿着每条边前行，运行时间至少为 O（边数）
2. 你还使用了一个队列，其中包含要检查的每个邻居，将每个邻居添加到队列需要的时间是固定的，O(1)
3. 你需要对每个邻居节点都这样做，需要的时间我 O（节点数）

**你需要按加入顺序检查搜索列表中的节点，否则找到的就不是最短路径，因此搜索列表必须是队列。**

**另外，对于检查过的人，务必不要再去检查，否则可能导致无限循环。**

```ruby
def search(name)
  search_queue = []
  search_queue += graph[name]
  searched = [] # 用于记录检查过的人
  while search_queue.any?
    person = search_queue.shift # remove first element
    next if searched.include?(person)
    if person_is_seller(person)
      return true
    else
      search_queue += graph[person]
      searched << person
    end
    false
  end
end
```

更好的写法在这里 [Breadth-first search (BFS)](https://github.com/brianstorti/ruby-graph-algorithms/tree/master/breadth_first_search)

## 排序算法
排序算法有很多，包括插入排序，冒泡排序，堆排序，归并排序，选择排序，计数排序，基数排序，桶排序，快速排序等。

插入排序，堆排序，选择排序，归并排序和快速排序，冒泡排序都是比较排序，它们通过对数组中的元素进行比较来实现排序，其他排序算法则是利用非比较的其他方法来获得有关输入数组的排序信息。

### 选择排序 Selection Sort

* 是对 定位比较交换法（也就是冒泡排序法）的一种改进。
* 时间复杂度：![](https://wikimedia.org/api/rest_v1/media/math/render/svg/b4a9cde84a808a1c8b6658032611f99e7fa0bb13)

算法过程：

1. 每一次从待排序的数据元素中选出最小（或最大）的一个元素，存放在序列的起始位置，直到全部待排序的数据元素排完。


```ruby
def find_smallest(arr)
  smallest = arr[0]
  smallest_index = 0
  arr.each_with_index do |n, i|
    if n < smallest
      smallest = n
      smallest_index = i
    end
  end
  smallest_index
end

def selection_sort(arr)
  new_arr = []
  until arr.size.zero? do
    smallest_index = find_smallest(arr)
    new_arr << arr[smallest_index]
    arr.delete_at smallest_index
  end
  new_arr
end
```

```ruby
# 使用 Ruby 提供的辅助方法
def selection_sort(arr)
  new_arr = []
  arr.size.times do
    min = arr.min
    new_arr << arr.min
    arr.delete_at(arr.index(min))
  end
  new_arr
end
```

### 快速排序

* 速度依赖于基准值
* 时间复杂度：![](https://wikimedia.org/api/rest_v1/media/math/render/svg/4d532063b672f55f3d9d24f9950d47278b837b22)

算法过程：

1. 选择基准值
2. 将数组分成两个子数组：小于基准值的元素和大于基准值的元素
3. 对这两个子数组进行快速排序
4. 小于基准值的数组 + [基准值] + 大于基准值的数组

```ruby
def quick_sort(arr)
  return arr if arr.size < 2 # 基线条件：数组为空或只包含一个元素的数组是“有序”的
  pivot = arr[0]
  less = arr.select { |x| x < pivot }
  greater = arr.select { |x| x > pivot }
  quick_sort(less) + [pivot] + quick_sort(greater)
end
arr = [617, 902, 159, 7, 579, 693, 525, 805, 871, 1002, 748, 473, 161, 271, 129, 632, 546, 894, 162, 637, 313]
pp quick_sort(arr)
```

```ruby
def quick_sort(a)  
  (x=a.pop) ? quick_sort(a.select{|i| i <= x}) + [x] + quick_sort(a.select{|i| i > x}) : []  
end  
```

## 相关阅读

* [Ruby algorithms and data structures](https://github.com/kanwei/algorithms/tree/master)
* [各种排序的Ruby实现](http://hideto.iteye.com/blog/280891)
* [经典排序算法资料及 ruby 实现](https://ruby-china.org/topics/20569)
* [Ruby 实现各种排序查找算法](http://liuzxc.github.io/blog/sorting-algorithm/)
