---
title: sort
category:
  - algorithm
tags:
  - code
  - sort
toc: true
---

# 排序算法

## 希尔排序

希尔排序实际上就是带步长的插入排序。不同于插入排序按照步长为1进行排序，希尔排序采用了缩进的步长。每次只在相同步长的一组数据中进行排序。

参考维基百科的解释，例如对于数组 []int{1,5,4,2,7,45,75,3,4,87} 排序，选取`步长`为 len() / 2，且每次步长`缩进一半`

其排序的过程如下

- 第一次分组 （步长为 5）

排序前

<pre>
1   5   4  2  7 
45  75  3  4 87
</pre>

调用插入排序 对竖着的数组进行排序

<pre>
1   5   3  2  7 
45  75  4  4 87
</pre>

- 第二次分组 (步长为 2)

排序前的数组为 {1,5,3,2,7,45,75,4,4,87}，因此分组为

<pre>
1   5
3   2
7   45
75  4
4   87
</pre>

插入排序

<pre>
1   2
3   4
4   5
7   45
75  87
</pre>

- 第三次分组 （步长为1）

步长为1 相当于插入排序 直接排序即可

```golang
package sort

// shell 排序是一个按照步长的排序
// 其本质是一个优化了步长的插入排序
func shellSort(nums []int) {
	if nums == nil || len(nums) == 0 {
		return
	}

	step := len(nums) / 2
	// 步长不停的缩短 直到最后成为插入排序（但是插入排序这个时候已经基本有序 所以跟完全乱序的相比 会有比较大的性能提升）
	for step >= 1 {
		// 插入排序
		for i := step; i < len(nums); i++ {
			exchange := nums[i]
			j := i - step
			// 将 i 位置的数 插入到 以 step 为步长的数组中间
			for ; j >= 0 && exchange < nums[j]; j -= step {
				nums[j + step] = nums[j]
			}
			// 上面已经把数据迁移完毕 只需要在 j+step 即结束迁移的位置 把需要插入的数据插入即可
			nums[j + step] = exchange
		}

		step /= 2
	}
}
```
