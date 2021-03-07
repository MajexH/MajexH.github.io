---
title: kmp
category:
  - algorithm
tags:
  - kmp
  - 字符串
toc: true
date: 2021-03-07 20:51:36
thumbnail:
---


算法书上使用的是 有限状态机 来描述的 kmp 算法。

之前学习的是通过前后缀匹配的方式来做的 kmp 算法。

现在记录一下。

参考：[阮一峰 blog](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)

## 前后缀匹配

实际上 kmp 的思想就是在字符串失配的时候，跳过一定的距离，就可以不用重置模式串的指针到开头。

如

<pre>
字符串 a b a b a b <span style="color: red">a</span> b c a
模式串 a b <span style="color: green">a b a b</span> <span style="color: red">c</span> a
kmp        <span style="color: green">a b a b</span> <span style="color: red">a</span> b c a
</pre>

如上所示，在失配的时候(红色所示)，会找到前面字符串中相同长度的前缀和后缀(绿色所示)，这样在匹配的时候，就可以直接跳过这些长度的前后缀，向右移动 6 - 4 = 2 位长度即可。

那么实际上 kmp 的原始运算算法就可以得到了

```java
public int kmp(String origin, String pattern) {
  // 获得 kmp 数组
  int[] next = getNext(String pattern);

  int i = 0, j = 0;

  while (i < origin.length() && j < pattern.length()) {
      if (origin.charAt(i) == pattern.charAt(j)) {
        i++;
        j++;
      } else {
        // 因为有相同的前后缀 所以直接赋值 就找到模式串相等的前缀的位置
        j = next[j];
      }
  }
  // 返回开始的 index
  if (j == pattern.length()) return i - j;
  // 没有找到匹配
  return -1;
}
```

而 next 数组的求法，实际上是 pattern 字符串自己跟自己匹配，找到相等的前后缀的长度

以 `a b a b a b c a` 为例

| 字符串  | 前缀               | 后缀               | 匹配的最大长度   |
| ------- | ------------------ | ------------------ | ---------------- |
| `a`     | []                 | []                 | 0                |
| `a b`   | [a]                | [b]                | 0                |
| `aba`   | [a, ab]            | [a, ba]            | 1（a 与 a 匹配） |
| `abab`  | [a, ab, aba]       | [bab, ab, b]       | 2 (ab 匹配)      |
| `ababa` | [a, ab, aba, abab] | [baba, aba, ba, a] | 3(aba 匹配)      |
| ……      | ……                 | ……                 | ……               |

上面的表格表示了前后缀的匹配情况，下一步就是求得这个前后缀匹配长度的 next 数组

```java
public int[] getNext(String str) {
  int n = str.length();

  int[] next = new int[n];
  // 让模式串错位一个开始 kmp 的匹配过程
  int k = -1, j = 0;
  while (j < n)) {
    if (k == -1 || str.charAt(k) == str.charAt(j)) {
      k++;
      j++;
      next[j] = k;
    } else {
      // 根据已经匹配的长度 重新定位
      k = next[k];
    }
  }
  return next;
}
```