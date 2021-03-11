---
title: sort
category:
  - algorithm
tags:
  - code
  - sort
toc: true
date: 2021-02-24 22:21:07
thumbnail:
---
# 排序算法

简单实现了一下几种常见的内排序算法

- 希尔排序
- 归并排序
- 快速排序
- 堆排序
- 基数排序
- 桶排序

<!-- more -->

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
## 归并排序

```golang
package sort

func mergeSort(nums []int, i, j int) {
	if i >= j {
		return
	}
	mid := (i + j) / 2
	mergeSort(nums, i, mid)
	mergeSort(nums, mid+1, j)
	// 这一步 i -> mid mid + 1 -> j 已经是有序的了
	merge(nums, i, mid, j)
}

func merge(nums []int, i, mid, j int) {
	tmp, index := make([]int, j-i+1), 0
	iStart, jStart := i, mid+1
	for iStart <= mid && jStart <= j {
		if nums[iStart] > nums[jStart] {
			tmp[index] = nums[jStart]
			jStart++
		} else {
			tmp[index] = nums[iStart]
			iStart++
		}
		index++
	}

	for iStart <= mid {
		tmp[index] = nums[iStart]
		index++
		iStart++
	}

	for jStart <= j {
		tmp[index] = nums[jStart]
		index++
		jStart++
	}

	for m, n := 0, i; m < len(tmp); m, n = m+1, n+1 {
		nums[n] = tmp[m]
	}
}

```

### [链表的归并排序](https://leetcode-cn.com/problems/sort-list/submissions/)

其实思想跟普通的归并排序基本一样，但是需要注意的是。归并分链表的时候，要直接**截断**链表

```golang
func sortList(head *ListNode) *ListNode {
	return mergeSort(head)
}

func mergeSort(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	mid := divideList(head)
	return mergeTwoListNode(mergeSort(head), mergeSort(mid))
}

func mergeTwoListNode(left, right *ListNode) *ListNode {
	res := new(ListNode)
	mov := res

	for left != nil && right != nil {
		if left.Val > right.Val {
			mov.Next = right
			right = right.Next
		} else {
			mov.Next = left
			left = left.Next
		}
		mov = mov.Next
	}

	if left != nil {
		mov.Next = left
	}
	if right != nil {
		mov.Next = right
	}
	return res.Next
}

// divideList 会把 list 分为两个 list
// 会截断原来的 node
func divideList(node *ListNode) *ListNode {
	if node == nil || node.Next == nil {
		return node
	}
	fast := node.Next

	for fast != nil && fast.Next != nil {
		fast = fast.Next.Next
		node = node.Next
	}
	mid := node.Next
	node.Next = nil
	return mid
}
```

## 快速排序

```golang
package sort

func QuickSort(nums []int, i, j int) {
	if i >= j {
		return
	}
	p := partition(nums, i, j)
	QuickSort(nums, i, p-1)
	QuickSort(nums, p+1, j)
}

func partition(nums []int, start, end int) int {
	base := nums[start]

	i, j := start, end+1

	for true {
		i++
		for i < len(nums) && nums[i] < base {
			i++
		}

		j--
		for j >= 0 && nums[j] > base {
			j--
		}

		if i >= j {
			break
		}

		swap(nums, i, j)
	}
	swap(nums, start, j)
	return j
}

func swap(nums []int, i, j int) {
	if nums[i] == nums[j] {
		return
	}
	nums[i] = nums[i] ^ nums[j]
	nums[j] = nums[i] ^ nums[j]
	nums[i] = nums[i] ^ nums[j]
}

```

## 堆排序

```golang
package sort

func defaultCompare(a, b int) bool {
	return a > b
}

// Heap 根据输入的 compare 构建不同的堆
// 默认小顶堆 -> 从大到小排列
type Heap struct {
	nums    []int
	compare func(a, b int) bool
}

// shiftDown 移动顶层的向下
func (h *Heap) shiftDown(k, n int) {
	for 2*k+1 <= n {
		j := 2*k + 1
		// 注意这个地方 大顶堆的时候 因为需要把小的东西往下沉 所以需要选择的是 子节点中 的较大值
		// 小顶堆的时候 由于需要把大的东西往下沉 所以需要选取的是较小值 （因为比较小值小 这个节点一定比两个节点都小）
		if j+1 <= n && h.compare(h.nums[j],h.nums[j+1]) {
			j++
		}
		if h.compare(h.nums[j], h.nums[k]) {
			break
		}
		swap(h.nums, k, j)
		k = j
	}
}

// popUp 最下面的浮动到最上面
func (h *Heap) popUp(k int) {
	for k >= 0 {
		var father int
		if k%2 == 1 {
			father = k / 2
		} else {
			father = k/2 - 1
		}
		if h.compare(h.nums[k], h.nums[father]) {
			break
		}
		swap(h.nums, father, k)
		k = father
	}
}

func NewHeap(nums []int) *Heap {
	return &Heap{nums: nums, compare: defaultCompare}
}

func NewHeapWithCompare(nums []int, compare func(a, b int) bool) *Heap {
	return &Heap{nums: nums, compare: compare}
}

func (h *Heap) Sort() {
	n := len(h.nums) - 1

	// 首先将输入构造成堆
	for i := n / 2; i >= 0; i-- {
		h.shiftDown(i, n)
	}
	// 这样排序是吧最小的排在后面
	for n >= 0 {
		swap(h.nums, 0, n)
		n--
		h.shiftDown(0, n)
	}
}

func swap(nums []int, i, j int) {
	if nums[i] == nums[j] {
		return
	}
	nums[i] = nums[i] ^ nums[j]
	nums[j] = nums[i] ^ nums[j]
	nums[i] = nums[i] ^ nums[j]
}
```

### 基数排序

基数排序的思维很简单，就是根据数字的每一位来排序

首先排序个位，根据个位数字 分别放到 编号 0-9 的桶里面

然后再排序十位，最后直到最大的数字也为 0 即可停止

```golang
func maximumGap(nums []int) int {
	if len(nums) < 2 {
		return 0
	}

	biggest := nums[0]
	for _, num := range nums {
		biggest = max(biggest, num)
	}

	// 基数排序的桶 分为 0 - 9
	lists := make([][]int, 10)

	base := 1
	for biggest > 0 {
		// 每次循环前都重置
		for i := 0; i < 10; i++ {
			lists[i] = make([]int, 0)
		}
		// 排序数组
		for _, num := range nums {
			i := num / base % 10
			lists[i] = append(lists[i], num)
		}

		// 根据每轮的顺序 重新赋值 nums 数组
		for i, index := 0, 0; i < 10; i++ {
			for j := 0; j < len(lists[i]); j++ {
				nums[index] = lists[i][j]
				index++
			}
		}

		biggest /= 10
		base *= 10
	}

	res := 0

	for i := 0; i < len(nums)-1; i++ {
		res = max(res, nums[i+1]-nums[i])
	}
	return res
}
```

### 桶排序

桶排序也很简单 直接根据最大小值分桶 然后根据其数值放到不同的桶里面

排序 只需要遍历桶的 下标即可

```golang
func maximumGapWithBucket(nums []int) int {
	if len(nums) < 2 {
		return 0
	}

	smallest, biggest := math.MaxInt32, math.MinInt32

	for _, num := range nums {
		smallest = min(smallest, num)
		biggest = max(biggest, num)
	}

	// 桶排序
	counts := make([]int, biggest - smallest + 1)

	for _, num := range nums {
		counts[num-smallest]++
	}
	res := 0
	var pre *int

	for i, num := range counts {
		// 表示没有数字
		if num == 0 {
			continue
		}
		if pre == nil {
			tmp := i
			pre = &tmp
			continue
		}
		// 比较
		res = max(res, i - *pre)
		tmp := i
		pre = &tmp
	}
	return res
}
```