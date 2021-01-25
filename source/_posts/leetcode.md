---
title: leetcode
category: code
tags:
  - leetcode
toc: true
date: 2021-01-18 16:38:31
thumbnail:
---


# leetcode例题

记录下 leetcode 值得记录的例题

## 二进制题目

### 模拟除法(https://leetcode-cn.com/problems/divide-two-integers/)

不能使用**乘法、除法和 mod 运算符**。

除法的本质，以 10 / 3 为例

10 / 3 = 3 …… 1 (即为 3 个 3 相乘 余 1)

即为 10 - (3 * 2) - (3 * 1) = 1 其结果为 2 + 1 为 3 

也就是说任意一种除法可以用一组除数的 2 的次方的乘积的结果来表示。

如 100 / 15 = 6

100 - (15 * 4) - (15 * 2)

所以可以采用二进制的方法来做，每次用被除数减去最大的一个除数的 2 次方的乘积，循环，直到剩下余数或者 0 

```golang
func divide(dividend int, divisor int) int {
	if divisor == 0 {
		return 1 << 31 - 1
	}
	minus := -1
	if (dividend < 0 && divisor < 0) || (dividend > 0 && divisor > 0) {
		minus = 1
	}

	absDividend, absDivisor := int64(math.Abs(float64(dividend))), int64(math.Abs(float64(divisor)))
	res := 0
	for absDividend >= absDivisor {
		tmp, multi := absDivisor, 1
		for (tmp << 1) < absDividend {
			tmp <<= 1
			multi <<= 1
		}
		absDividend -= tmp
		res += multi
		if minus > 0 && res >= math.MaxInt32 {
			return math.MaxInt32
		} else if minus < 0 && minus * res <= math.MinInt32 {
			return  math.MinInt32
		}
	}
	return minus * res
}
```

```java
class Solution {
    public int divide(int dividend, int divisor) {
        if(divisor == 0 || (dividend == Integer.MIN_VALUE && divisor == -1)) return Integer.MAX_VALUE;

        int sign;
        if ((dividend > 0 && divisor > 0) || (dividend < 0 && divisor < 0)) sign = 1;
        else sign = -1;

        int res = 0;

        long dvd=Math.abs((long)dividend);
        long dvs=Math.abs((long)divisor);
        while (dvd >= dvs) {

            long temp = dvs, m = 1;
            while (temp << 1 < dvd) {
                temp <<= 1;
                m <<= 1;
            }

            res += m;
            dvd -= temp;

        }
        return sign * res;
    }
}
```

## 贪心算法

### [转换罗马字](https://leetcode-cn.com/problems/roman-to-integer/)

```golang
package classic

import (
	"strings"
)

var (
	memo = make(map[int]string)
	keys []int
)

func init() {
	memo[1] = "I"
	memo[4] = "IV"
	memo[5] = "V"
	memo[9] = "IX"
	memo[10] = "X"
	memo[40] = "XL"
	memo[50] = "L"
	memo[90] = "XC"
	memo[100] = "C"
	memo[400] = "CD"
	memo[500] = "D"
	memo[900] = "CM"
	memo[1000] = "M"

	keys = []int{1000,900,500,400,100,90,50,40,10,9,5,4,1}
}

func intToRoman(num int) string {
	res := strings.Builder{}

	for _, key := range keys {
		for num >= key {
			res.WriteString(memo[key])
			num -= key
		}
	}
	return res.String()
}
```

### [jumpGame](https://leetcode-cn.com/problems/jump-game/)

```golang
func canJump(nums []int) bool {
	rightMost := 0
	for i, num := range nums {
		// 如果当前的下标大于 rightMost 说明这个点是无法到达的 直接返回 false 即可
		if i > rightMost {
			return false
		}
		// 维护一个能够到达的最远距离
		rightMost = max(rightMost, i + num)
		// 最远距离大于长度 即可知道能够达到
		if rightMost >= len(nums) - 1 {
			return true
		}
	}
	return false
}
```

## 递归

### [括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

```golang
// generateParenthesis 入口函数
func generateParenthesis(n int) []string {
	res := make([]string, 0)
	recursionGenerate(&res, "", n, n)
	return res
}

// 递归生成
func recursionGenerate(res *[]string, tmp string, left, right int) {
	if left == 0 && right == 0 {
		*res = append(*res, tmp)
		return
  }
  // 由于左括号可以直接放到结果上，因此左括号不用判断其他的
	if left > 0 {
		recursionGenerate(res, tmp+"(", left-1, right)
  }
  // 而有括号需要跟左括号匹配，所以有括号遍历的时候 必须已经有左括号被放到了结果中
  // 所以需要判断一下 right > left
	if right > left {
		recursionGenerate(res, tmp+")", left, right-1)
	}
}
```

```java
public List<String> generateParenthesis(int n) {
    List<String> res = new ArrayList<>();
    recursion(res, n, n, "");
    return res;
}

public void recursion(List<String> res, int left, int right, String tmp) {
    if (left == 0 && right == 0) {
        res.add(tmp);
        return;
    }

    if (left > 0) {
        recursion(res, left - 1, right, tmp + "(");
    }
    if (right > left) {
        recursion(res, left, right - 1, tmp + ")");
    }
}
```

### [正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)

这道题可以用递归的思想去做，也可以采用 dp 的方法。实际上递归就是从上向下的 dp

```golang
func isMatch(s string, p string) bool {
	return recursionIsMatch(s, p, 0, 0)
}

func recursionIsMatch(s, p string, sIndex, pIndex int) bool {
	// 完全匹配
	if sIndex == len(s) && pIndex == len(p) {
		return true
	}
	// 越界 pattern 匹配完了一定有问题
	if pIndex == len(p) && sIndex != len(s) {
		return false
	}

	if pIndex < len(p)-1 && p[pIndex+1] == '*' {
		// 匹配
		if pIndex < len(p) && sIndex < len(s) && (p[pIndex] == s[sIndex] || p[pIndex] == '.') {
			return recursionIsMatch(s, p, sIndex, pIndex+2) || // 匹配0次 因为 * 代表 0 -> 多次
				recursionIsMatch(s, p, sIndex+1, pIndex+2) || // 匹配1次
				recursionIsMatch(s, p, sIndex+1, pIndex) // 匹配多次
		} else {
			// 如果不匹配 则跳过
			return recursionIsMatch(s, p, sIndex, pIndex+2)
		}
	}
	// 现在的字符是匹配的
	if pIndex < len(p) && sIndex < len(s) && (p[pIndex] == s[sIndex] || p[pIndex] == '.') {
		return recursionIsMatch(s, p, sIndex+1, pIndex+1)
	}
	return false
}
```

```java
public boolean isMatch(String s, String p) {
    return recursion(s, p, 0, 0);
}

public boolean recursion(String s, String p, int sIndex, int pIndex) {
    if (pIndex == p.length() && sIndex != s.length()) {
        return false;
    }

    if (sIndex == s.length() && pIndex == p.length()) {
        return true;
    }

    if (pIndex < p.length() - 1 && p.charAt(pIndex + 1) == '*') {
        if (sIndex < s.length() && (p.charAt(pIndex) == '.' || p.charAt(pIndex) == s.charAt(sIndex))) {
            return recursion(s, p, sIndex, pIndex + 2) || // 匹配0次
                    recursion(s, p, sIndex + 1, pIndex + 2) || // 匹配1次
                    recursion(s, p, sIndex + 1, pIndex); // 匹配多次
        } else {
            return recursion(s, p, sIndex, pIndex + 2);
        }
    }
    if (sIndex < s.length() && pIndex < p.length() && (p.charAt(pIndex) == '.' || p.charAt(pIndex) == s.charAt(sIndex))) {
        return recursion(s, p, sIndex + 1, pIndex + 1);
    }
    return false;
}
```

## 数据结构

### 链表

#### [删除倒数的第N个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

```golang
func removeNthFromEnd(head *ListNode, n int) *ListNode {
	// 因为可能删除头结点 所有加了一个
	newHead := new(ListNode)
	newHead.Next = head
	// fast 是先走的一个节点 pre 是后走的
	pre, fast := newHead, head
	for n > 0 {
		// 有问题 数量不够
		if fast == nil {
			return nil
		}
		fast = fast.Next
		n--
	}
	// 两个指针开始走
	for fast != nil {
		fast = fast.Next
		pre = pre.Next
	}

	pre.Next = pre.Next.Next
	return newHead.Next
}
```

#### [合并k个已经排序的链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

类似归并排序

```golang
package classic

// mergeKLists 合并k个已经按照升序排列的数组
func mergeKLists(lists []*ListNode) *ListNode {
	return merge(lists,0, len(lists) - 1)
}

func merge(lists []*ListNode, start, end int) *ListNode {
	if start > end {
		return nil
	}
	if start == end {
		return lists[start]
	}
	mid := (start + end) / 2
	left, right := merge(lists, start, mid), merge(lists, mid + 1, end)
	return mergeTwoList(left, right)
}

func mergeTwoList(list1 *ListNode, list2 *ListNode) *ListNode {
	res := new(ListNode)
	rem := res

	for list1 != nil && list2 != nil {
		if list1.Val > list2.Val {
			res.Next = list2
			list2 = list2.Next
		} else {
			res.Next = list1
			list1 = list1.Next
		}
		res = res.Next
	}
	if list1 != nil {
		res.Next = list1
	}
	if list2 != nil {
		res.Next = list2
	}
	return rem.Next
}
```

#### 翻转链表一系列

##### [反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

最简单的反转链表的思路肯定是直接用一个 stack，FILO 的机制来反转，而不采用额外的空间可以用一下的方法

```golang
func reverseList(head *ListNode) *ListNode {
    // 反转后的头节点
	var pre *ListNode = nil

	for head != nil {
        // 用一个 局部变量 来保存下一个节点
        next := head.Next
        
        // 反转当前遍历的 head 节点，指向已经反转完毕的头结点
		head.Next = pre
        pre = head
        // 重新设置 head 头
		head = next
	}
	return pre
}
```

##### [反转链表II](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

反转链表II是反转链表下表从 m -> n 的一个链表，实际上采用上述的反转的操作，即可反转 m -> n 之间的节点

```golang
// 主函数
func reverseBetween(head *ListNode, m int, n int) *ListNode {
    // 添加一个新的头结点，保障原来的头结点被反转时的结果
	newHead := new(ListNode)
	newHead.Next = head
    cp := newHead
    // 分别保存需要反转的节点之前的一个节点以及最后需要反转的一个节点
	var preStartNode, endNode *ListNode = nil, nil
	for i := 0; cp != nil; cp, i = cp.Next, i + 1 {
		if i == m - 1 {
			preStartNode = cp
		}
		if i == n {
			endNode = cp
		}
    }
    // 保存反转完毕后的链表需要连接到的下一个节点
	afterEndNode := endNode.Next
	reverseHead, reverseEnd := reverseBetweenNodes(preStartNode.Next, afterEndNode)
	preStartNode.Next = reverseHead
	reverseEnd.Next = afterEndNode

	return newHead.Next
}

// reverseBetweenNodes reverse两个node之间的链表
// 其中 startNode 为开始翻转的节点 endNodeNext 为结束翻转的节点的后一个节点
func reverseBetweenNodes(startNode, endNodeNext *ListNode) (*ListNode, *ListNode) {
	var newHead *ListNode = nil
	newEndNode := startNode
	for startNode != endNode {
		next := startNode.Next
		startNode.Next = newHead
		newHead = startNode
		startNode = next
	}
	return newHead, newEndNode
}
```

##### [reverse K group](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

reverse K group 的更进一步，在上面一题的基础上，每K个节点反转一次

```golang


// reverseKGroup k个一组翻转
func reverseKGroup(head *ListNode, k int) *ListNode {
	newHead := new(ListNode)
	newHead.Next = head

	var preStartNode, endNode *ListNode = newHead, nil
	counter := 0
	mov := newHead

	for mov != nil {
		if counter == k {
			endNode = mov
            afterEnd := endNode.Next
            // 调用二题中所述的函数进行反转
			reverseHead, reverseEnd := reverseBetweenNodes(preStartNode.Next, endNode.Next)
			preStartNode.Next = reverseHead
			reverseEnd.Next = afterEnd
            // 因为反转之后要重新记录 preStartNode
            counter = 0
            // 反转之后的需要再次反转的头是上次反转的尾结点
			preStartNode = reverseEnd
			// 重新定位移标
			mov = reverseEnd
		}
		mov = mov.Next
		counter++
	}

	return newHead.Next
}

// reverseBetweenNodes reverse两个node之间的链表
// 其中 startNode 为开始翻转的节点 endNodeNext 为结束翻转的节点的后一个节点
func reverseBetweenNodes(startNode, endNodeNext *ListNode) (*ListNode, *ListNode) {
	var newHead *ListNode = nil
	newEndNode := startNode
	for startNode != endNodeNext {
		next := startNode.Next
		startNode.Next = newHead
		newHead = startNode
		startNode = next
	}
	return newHead, newEndNode
}
```

### 图

#### 并查集的数据结构

并查集，表示的是一个树形的结构

{% asset_img union.png 并查集示意 %}

如图所示，针对图的一个极大连通分量，会形成一个对应的树结构（并查集只关注一个连通分量有多少连接点，不关注内部的其他的细节）

所以针对查找连通分量有哪些，以及连同量间的关系有作用

##### 并查集存储数据的结构

```golnag
type Union struct {
	parents []int // 存储树的数据结构 parents[i] 表示连接到该节点的父节点的索引 如果不能用 int 来表示 可以考虑 map 类的数据结构
}
```
##### 并查集的操作

- union (联合，关联两个点)
- find  (查找，找到当前点的最终的父节点)

所以，实际上 如果 r1 r2 之间有连接线的话，要关联 r1 r2 的操作就是。

- 就是通过 `find` 找到分别的根节点 r1Root r2Root
- 在通过 `union` 方法关联两个根节点，实际上就是将 r2Root 作为一个子节点，挂载到 r1Root 下

所以整体的数据结构为

```golang
type unionFind struct {
	Parents []int
	Count   int     // 表示连通分量的多少
}

func NewUnionFind(size int) *unionFind {
	res := &unionFind{
		Parents: make([]int, size),
		Count:   size,
	}
	// 初始化并查集中的每个元素的父节点都是自己
	for i := 0; i < size; i++ {
		res.Parents[i] = i
	}
	return res
}

func (u *unionFind) union(i, j int) {
	iRoot := u.find(i)
	jRoot := u.find(j)
	if iRoot != jRoot {
		u.Parents[jRoot] = iRoot
		// 每次连接一个之后 最大连通分量就要 --
		u.Count--
	}
}

func (u *unionFind) find(i int) int {
	if u.Parents[i] == i {
		return i
	}
	return u.find(u.Parents[i])
}

func (u *unionFind) GetCount() int {
	return u.Count
}
```

#### [连通网络的操作次数](https://leetcode-cn.com/problems/number-of-operations-to-make-network-connected/)

题目所述，给定一个图，找到将其所有最大连通分量连通所需更改的最少的边的数量为多少。

> 图的所有最小连通为一个树，即需要 n 个节点有 n - 1 条边。

所以题目其实是要找到这个图里面有多少独立的连通分量，然后判断其是否可以连接

- 第一种思路就是直接 dfs 遍历，找到所有的连通分量。首先判断边的数量是否足够 n - 1 这个时候，如果有多个连通分量，说明某个连通分量一定有多的边，随意选取其中的边即可

```golang
package classic

// makeConnected 方法查看
func makeConnected(n int, connections [][]int) int {
	// 最短的话肯定是形成一棵树 才能联通所有
	// 所以 边 至少要达到 n - 1 的数量
	// 这个时候不能连通
	if len(connections) < n-1 {
		return -1
	}
	// map 的邻接表表示
	cMap := make(map[int][]int)
	for _, connection := range connections {
		if _, ok := cMap[connection[0]]; !ok {
			cMap[connection[0]] = make([]int, 0)
		}
		cMap[connection[0]] = append(cMap[connection[0]], connection[1])

		if _, ok := cMap[connection[1]]; !ok {
			cMap[connection[1]] = make([]int, 0)
		}
		cMap[connection[1]] = append(cMap[connection[1]], connection[0])
	}

	// 到这里的时候 由于边的数量够 所以一定是可以连通的
	// 这个时候 只需要知道有 m 块是不相连的 然后就知道需要连接的次数就为 m - 1
	memo := make([]bool, n)
	res := 0
	for i := 0; i < n; i++ {
		if !memo[i] {
			res++
			dfs(memo, cMap, i)
		}
	}
	return res - 1
}

func dfs(memo []bool, cMap map[int][]int, start int) {
	memo[start] = true
	for _, next := range cMap[start] {
		if !memo[next] {
			dfs(memo, cMap, next)
		}
	}
}
```

- 并查集，找到每个群组的数据的一个代表点

```golang
// findRoot 找到根节点
func findRoot(parents []int, index int) int {
	if parents[index] == -1 {
		return index
	}
	return findRoot(parents, parents[index])
}

// makeConnected 并查集
func makeConnected(n int, connections [][]int) int {
	if len(connections) < n - 1 {
		return -1
	}

	// 并查集的 parents 数组，标识 当前索引的 节点的父节点的索引是谁
	// 相同的数最后均能找到同样的父节点
	parents := make([]int, n)
	// 初始化所有的为 -1
	for i := 0; i < n; i++ {
		parents[i] = -1
	}

	// 执行 union 的操作
	for _, connection := range connections {
		sRoot := findRoot(parents, connection[0])
		eRoot := findRoot(parents, connection[1])

		// 两个点上有连接线 但是现在还没有连接起来
		// 让其根节点相连
		if sRoot != eRoot {
			// 将 e 节点连接到 s 上
			parents[eRoot] = sRoot
		}
	}
	// 剩下的还是 -1 的就一定是整个群里面的代表节点
	res := 0
	for _, val := range parents {
		if val == -1 {
			res++
		}
	}
	return res - 1
}
```

#### [由斜杠划分区域](https://leetcode-cn.com/problems/regions-cut-by-slashes/)

采用并查集，但是这道题有特殊的地方。

题目中所示，针对一个方格有 **/** 和 **\\** 两种，如下

<pre>
----    ----
|\ |    | /|
| \|    |/ |
----    ----
</pre>

总之，针对一个 方格 ，可以把他看成四个部分

{% asset_img 959-1.png 一个单元格分块 %}

那么，也就是说，

- 如果当前 char == ' ' 表示 0 1 2 3 都是联通的
- 如果 char == '\\' 表示 01 23 分别连通
- char == '/' 表示 03 12 分别连通

内部的连通完毕后，
- 还可以知道 1 一定跟下一个 3 连通
- 2 一定跟下一行的 0 连通

```golang
// regionsBySlashes 通过斜杠划分
func regionsBySlashes(grid []string) int {
	// n * n 的矩阵的长度
	length := len(grid)
	// 为了使用 并查集 将一个1*1 的正方形，即 一个 grid[i] 标识的区域分成 四个地方
	// 然后再根据 / \\ 两个符号的位置进行合并 最后看有几个节点
	unionSize := 4 * length * length
	u := NewUnionFind(unionSize)

	for i, str := range grid {
		for j, char := range str {
			// 0 号位置
			uIndex := 4 * (i*length + j)

			// 同一个单元格里面的连接起来
			switch char {
			// 0 1 2 3 都要连接起来
			case ' ':
				u.union(uIndex, uIndex + 1)
				u.union(uIndex + 1, uIndex + 2)
				u.union(uIndex + 2, uIndex + 3)
			case '\\':
				// 反斜杠的话 01 23 分别连接
				u.union(uIndex, uIndex + 1)
				u.union(uIndex + 2, uIndex + 3)
			case '/':
				// 斜杠的话 03 12 分贝连接
				u.union(uIndex, uIndex + 3)
				u.union(uIndex + 1, uIndex + 2)
			}
			// 单元格外面的连接起来
			// 不管是 \\ 还是 / 这个区域的1一定可以和右边下一个区域( j+ 1)的 3 连接
			// 这个区域的 2 一定可以和下边(i + 1)下一个区域的 0 连接
			if j + 1 < length {
				u.union(uIndex+1, 4*(i*length+j+1)+3)
			}
			if i + 1 < length {
				u.union(uIndex+2, 4*((i+1)*length+j))
			}
		}
	}

	return u.GetCount()
}
```

