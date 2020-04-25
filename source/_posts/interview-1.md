---
title: 春招java后端实习岗笔试总结
category:
  - life
tags:
  - 笔试
  - 求职
toc: true
date: 2020-04-24 14:25:57
thumbnail:
---


## 春招实习生投递小结

春招实习生一共投了4家企业，包括字节、美团、阿里和快手，因为两边都相对较多，因此将笔试和面试分开吧

- 字节倒在了二面
- 美团刚刚完成了二面
- 阿里完成了三面(不过我觉得我跟他们需求的方向相对较远，因此希望渺茫)
- 快手，可能是投递得比较晚，至今(4月24日无任何消息)

因为觉得大部分希望渺茫，所以特地记录下春招的笔试和面试情况，希望有所帮助。

### 笔试

有几家是在牛客网提供的平台上笔试的，不过笔试的方式都是大同小异，保证入口class是Main即可，可以先去牛客网上试试

不保证自己的贴出的代码能够全部AC，**仅供参考**

#### 阿里

阿里的笔试是所有人统一时间参加笔试，我是在4月13日参加的晚上的笔试，抽到的笔试题都是跟图相关的

##### 第一题

###### 题目描述

一个数组，表示一组动物（动物的下标从1开始），数组中的数值表示仰慕的*对象*，**这个动物可以给自己投票，或者跟自己仰慕的人投同样的票**，如果数组中数值为0 那么他只能给自己投票，而且数组中的位置代表了这个动物的能力大小 后面的动物越低，他们只能仰慕前面的动物。

问：每个动物可能拿到的最大的投票数为多少

```
例：

输入
[0,1,1,1,1]

输出
4 1 1 1 1
```

解释：第一个动物只能给自己投票，所以他给自己投票，而后面的动物仰慕1号动物，因此全部跟自己仰慕的对象投相同的票，即1投自己后，后面的2 3 4 5号动物全跟1投相同的票，投1，因此1号最多4票，其他的只有自己投自己。

例程（笔试中写的代码没有考虑到递归情况，只对了10%）

###### 可能解法

```java
package com.company;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Scanner;

public class First {

    // 递归的去寻找后面能够投票的是哪些
    public static int dfs(HashMap<Integer, List<Integer>> map, int start, boolean[] memo) {
        List<Integer> temp = map.get(start);
        memo[start] = true;
        int res = temp.size();
        for (Integer admire : temp) {
            if (start < admire && !memo[admire])
                res += dfs(map, admire, memo);
        }
        memo[start] = false;
        return res;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int[] admires = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            admires[i] = scanner.nextInt();
        }
        HashMap<Integer, List<Integer>> map = new HashMap<>();

        // 简历崇拜者图
        for (int i = 1; i < admires.length; i++) {
            if (!map.containsKey(i)) {
                map.put(i, new ArrayList<>());
            }
            if (admires[i] != 0 && admires[i] != i) {
                if (!map.containsKey(admires[i])) {
                    map.put(admires[i], new ArrayList<>());
                }
                List<Integer> temp = map.get(admires[i]);
                temp.add(i);
            }
        }
        System.out.println(map);
        for (int i = 1; i <= n; i++) {
            System.out.println(dfs(map, i, new boolean[n + 1]) + 1);
        }
    }
}
```

##### 第二题

###### 题目描述

给定三个参数，3个参数表示，城市数，路径数和目标城市，然后下面有路径数的输入i,j,k，分别表示从i城市到j城市花费的时间是k。

问：如果每个城市都有一个人，那么这些人，到这个目标城市，然后回来，在路上花费的时间的最小值

(ps:实际上就是个最短路径的问题，但是我当时根本就没咋弄过图算法，只了解基本的dfs，bfs和连通分量啥的，因此笔试的时候是用两边dfs做的，存在问题，最短路算法应该去复习一下)

###### 可能解法

```java
package com.company;

import java.util.*;

public class Second {

    static class Pair {
        public Integer key;
        public Integer value;

        public Pair(Integer key, Integer value) {
            this.key = key;
            this.value = value;
        }
    }

    public static int resolve(HashMap<Integer, List<Pair>> map, int start, int end, int size) {
        boolean[] memo = new boolean[size];
        return dfs(map, start, end, memo, 0);
    }

    public static int dfs(HashMap<Integer, List<Pair>> map, int start, int end, boolean[] memo, int current) {
        if (start == end) return current;
        memo[start] = true;
        List<Pair> temp = map.get(start);
        int res = Integer.MAX_VALUE;
        for (Pair tempPair : temp) {
            if (!memo[tempPair.key]) {
                res = Math.min(res, dfs(map, tempPair.key, end, memo, current + tempPair.value));
            }
        }
        memo[start] = false;
        return res;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt(), m = scanner.nextInt(), x = scanner.nextInt();

        HashMap<Integer, List<Pair>> map = new HashMap<>();
        for (int i = 0; i < m; i++) {
            int a = scanner.nextInt(), b = scanner.nextInt(), l = scanner.nextInt();
            if (!map.containsKey(a)) {
                map.put(a, new ArrayList<>());
            }
            List<Pair> temp = map.get(a);
            temp.add(new Pair(b, l));
        }
        int res = Integer.MIN_VALUE;
        for (int i = 1; i <= n; i++) {
            res = Math.max(res, resolve(map, i, x, n + 1) + resolve(map, x, i, n + 1));
        }
        System.out.println(res);
    }
}
```

#### 美团

美团的笔试，我们同学投的前端和后端是一样的，美团的笔试环境跟牛客网差不多，注意一下即可

##### 第一题

###### 题目描述

给定两个整数n,m。n表示有多少个学生，m表示有多少门课程，
接下来输入n行，每行m个，表示这个学生在这门课的得分是多少。

问：每门课获得最高成绩的人（可能有**多个**）一共有多少个，重复的人在不同的课程中拿第一门，算同一个人。

###### 可能解法（笔试时AC）

```java
import java.util.*;

public class MFirst {

    public static class Pair {
        public Integer index;
        public Integer credit;

        public Pair(int i, int i1) {
            this.index = i;
            this.credit = i1;
        }
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        int n = scanner.nextInt(), m = scanner.nextInt();

        HashSet<Integer> res = new HashSet<>();
        int[][] credits = new int[n][m];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                credits[i][j] = scanner.nextInt();
            }
        }

        PriorityQueue<Pair> queue = new PriorityQueue<>((a, b) -> b.credit - a.credit);
        for (int j = 0; j < m; j++) {

            for (int i = 0; i < n; i++) {
                queue.add(new Pair(i, credits[i][j]));
            }
            if (queue.size() > 0) {
                int max = queue.peek().credit;
                Pair temp;
                while (queue.size() > 0 && (temp = queue.poll()).credit == max) {
                    res.add(temp.index);
                }
            }
            queue.clear();
        }
        System.out.println(res.size());
    }
}
```

##### 第二题

###### 问题描述

给定一个输入，a、b、m和x。采用一个循环算法，如下

```
while (true)
  x = (a * x + b) % m;
```

问循环几次，这个算法中x值会跟最开始一样

###### 可能解法（因为没有注意到题目的数据范围，爆了int，AC了65%）

```java
import java.util.HashSet;
import java.util.Scanner;
import java.util.Set;

public class MSecond {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int a = scanner.nextInt(), b = scanner.nextInt(),
                m = scanner.nextInt();
        long x = scanner.nextLong();

        int counter = 0;
        Set<Long> set = new HashSet<>();
        while (true) {
            x = (a * x + b) % m;
            if (!set.contains(x)) {
                set.add(x);
                counter++;
            } else {
                break;
            }
        }
        System.out.println(counter);
    }
}
```

##### 第三题

###### 问题描述

给定一个输入n和m，接下来输入n个数。

问：如果这个n个数（这n个数有可能重复）进行排列组合会产生n*n的组合（自己可以和自己组合），那么如果将这些组合从大到小排序（排序的规则是对于一个组合[i,j] 如果i相同，则按照j的大小排序，如果i不同，i大的排在前面），问第m小的一个组合是什么

```
例

输入：
3 4
1 2 3

输出
(2,1)
```

###### 可能解法(当时AC了)

```java
import java.util.Arrays;
import java.util.Scanner;

public class MThird {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt(), m = scanner.nextInt();
        int[] nums = new int[n];
        for (int i = 0; i < n; i++) {
            nums[i] = scanner.nextInt();
        }

        Arrays.sort(nums);
        int divide;

        if (m % n == 0) {
            divide = m / n - 1;
        } else {
            divide = m / n;
        }
        int counter = 0, counter1 = 0;
        for (int num : nums) {
            if (nums[divide] > num) counter1++;
            if (nums[divide] == num) counter++;
        }
        int temp = (m - counter1 * n) / counter;
        System.out.format("(%d,%d)\n", nums[divide], nums[temp - 1]);
    }
}
```

##### 第四题

###### 问题描述

给定一个输入n，k。n表示要输入的数组的数的数量，k表示其中的某个数，然后输入n个数。

问：如果要让k这个数成为中位数，那么需要再往这个数组里面添加几位数才行

```
例

输入
4 1
1 2 2 3

输出
2
```

解释：要让1成为中位数，那么就需要再在1前面添加两个数即可成为新数组的中位数

###### 可能的解法（TODO）

我当时以为整个数组的输入顺序，包括现在的一切都不能改变，因此我只找了离现在的中心最近的k的位置，然后看看它需要几位数才能到中心（AC 10%）

实际上这道题是一个桶排序，只需要统计小于K的有多少个数，等于K的有多少个数，大于K的多少个数，然后再去跟mid比较，最后得到答案（感谢镇宇dalao）

我的解法
```java
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class MFourth {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt(), k = scanner.nextInt();
        int mid = (n + 1) / 2;
        int[] nums = new int[n];
        int remember = Integer.MAX_VALUE;

        for (int i = 0; i < n; i++) {
            nums[i] = scanner.nextInt();
            if (nums[i] == k)
                remember = Math.min(remember, Math.abs(mid - i));
        }
        int res = Integer.MAX_VALUE;

        System.out.println(res);
    }
}
```

正确的解法(模仿镇宇dalao的思路来的 TODO)

```java

```

##### 第五题

###### 问题描述

给定两个字符串s1,s2

问：
s1子串与s2的的子序列的匹配个数（子串就是说是连续的，子序列就是说是不连续的）

###### 解法

方法当时第一反应就是dp，可是没有找到递推公式，普航julao AC了，请教了一下，大致理解了思路，就是用dp[i][j] 表示以s1[i]结尾子串和s2[j]结尾的子序列能够匹配的数量

转移方程的话就是需要考虑s1[i] == s[j],相等就说明我们只需要看dp[i - 1][k](k = 0 ………… j-1)的和+1即可 ，因此最后一位是匹配的，而不等，就直接赋值为0即可

讨论原文：dp[i][j]是s[i]为尾的的子串与t[j]为尾的的子序列的匹配个数 如果s[i] != t[j]那直接就是0了 如果不是的话 那就是以s[i-1]以及所有t[k]（0<=k<=j-1）的匹配个数的和（因为这些匹配的接上s[i] t[j]还是匹配的）再+1（只有这一个位置匹配前面全部不匹配的情况）


代码（TODO）

```java
```