---
title: 2021春招
category: recruit
tags:
  - 春招
  - 面试
toc: true
---

# 记录下春招

现在有美团、shopee以及keep

京东这些没有消息，再说吧

<!-- more -->

## 一些问得比较多的面试题的总结

### IO 模型

#### BIO

{% asset_img fileInputStream-read.png 报错 %}

实际上 java 提供的从 file 流中读取数据的 read() 方法是调用了底层的 c 方法，在读取数据时会陷入到内核调用中，堵塞的读取文件中的数据到内存中。

#### BIO 改进

在网络编程中的话，由于使用 BIO 的 read 等方法，会堵塞的相应，导致只能进行一对一的通信，而服务端往往对于并发要求更高。所以可以使用子进程 or 子线程的方式，将 堵塞 的操作交给其执行，但是由于需要 进程 or 线程来处理这个请求，会导致的问题就是，当`管理的进程 or 线程较多时无法处理`。

##### 使用进程

{% asset_img bio-sub-process.png 报错 %}

因为 BIO 实际上需要堵塞住当前的主线程，那么为网络等的调用中，为了提高整体的效率，可以考虑使用多进程的方式进行处理，将 `堵塞的调用操作` 交给 fork 的子进程处理。

##### 使用线程

同理，对于进程进行新建和切换的操作会涉及到内核调用以及上下文切换、数据拷贝等一系列的操作，新建线程的话，会更轻量级，所以可以使用 `线程池` 等，将堵塞的 read 等操作交给 `子线程` 执行，提高并发效率。

##### io 多路复用

由于不可能同时处理如此多的线程和进程，所以如果所有的操作都是在一个线程中执行，并且每个 socket 分时的处理，那么从长远看起来所有的操作看起来都是并发的在执行，那么就能避免改进中的问题。

io 多路复用均是采用了以下的思想。通过监听 socket 的事件，在准备就绪的时候，读取 socket 中的数据，拷贝到需要使用的地方。

{% asset_img io-multi.png 报错 %}

均是从系统底层提供的实现，包括 select、poll、epoll。

###### select

> select() allows a program to monitor multiple file descriptors, waiting until one or more of the file descriptors become "ready" for some class of I/O operation (e.g., input possible).  A file descriptor is considered ready if it is possible to perform a corresponding I/O operation (e.g., read(2), or a sufficiently small write(2)) without blocking. <br/><br/>
> select() can monitor only file descriptors numbers that are less than FD_SETSIZE; poll(2) and epoll(7) do not have this limitation.

> select 能够允许程序监听多个 fd，直到某些监听的 fd 在一些 I/O 操作中成为 'ready' 状态。一个 fd 的 ready 状态表示了其能够执行一个相关的 I/O 操作（如，read write 等）。<br/><br/>
> select 只能监听小于 FD_SETSIZE （默认为 1024）个 fd。

select 函数的参数如下

- readfds。关注的能够触发 read 的 fds。
- writefds。write 的 fds。
- exceptfds。关注 "exceptional conditions" fds。
- nfds。需要处理的 fd 的数量。
- timeout。超时时间。

select 函数用 bitmap 的形式来描述 readfds 等参数。每个 fd 会对应 bit 中的一位数字，在事件就绪后，该位会被改为 1。并取消 select 函数的堵塞状态。之后遍历 fds 的 bitmap，找到已经能够使用的 socket，并拷贝数据，`重置 fds 状态`(因为内核态已经把这个位上的数据修改，需要再修改到未置位的状态)。

以上，select 的执行过程中需要：

1. 拷贝 fds 的 bitmap 到内核态中，在置位后，从内核态中拷贝到用户态中
2. 遍历 fds 也需要两次，一次在内核态中遍历，一次在 select 返回后，需要遍历 bitmap 找到已经被置位的 fd。每次都是 o(n) 的开销。
3. bitmap 大小受到限制。
4. bitmap 需要重新置位，在线程处理完拷贝数据后。

###### poll

poll 采用的思路与 select 一致，只是不再使用 bitmap 的方式来描述 fds 的结构。而是使用 array 的形式来表示 bit map，array 中的元素为一个 结构体。

```c
struct pollfd {
    int   fd;         /* file descriptor */
    short events;     /* requested events */
    short revents;    /* returned events */
};
```
其中 
- events `a bit mask specifying the events the application is interested in for the file descriptor fd.` 关注的 请求的时间

- revents `filled by the kernel with the events that actually occurred.  The bits returned in revents can include any of those specified in events, or one of the values POLLERR, POLLHUP, or POLLNVAL` 讷河设置的返回的事件类型。

因此，在使用中

1. poll 仍需要遍历 pollfd 的数组，仍然是 o(n) 的开销
2. poll 仍然需要重置 events 的状态位

###### epoll

{% asset_img epoll.png 报错%}

epoll 优化了 poll、select 需要遍历所有的 fd 找到已经被置位的 fd 的过程。

- 使用了红黑树来组织所有的 ctl 上去的 fd
- 在 epoll_wait 等待 fd 中有状态被更改后，会返回触发的 fd 数量，并在 fd 数组中将被触发的 fd 移到数组的前端，这样就可以在 fd 的数组中直接访问到被触发的 fd

epoll 经历的阶段

- epoll_create: 创建 epoll 返回对应的 epoll_fd
- epoll_ctl: 将对应的 socket 格式化成 epoll_event 并加入到 epoll_fd 对应的 epoll 中
- epoll_wait: 直到 epoll_wait 返回对应数量的 fd

下面是直接复制参考的：

epoll 支持两种事件触发模式，分别是边缘触发（edge-triggered，ET）和水平触发（level-triggered，LT）。

这两个术语还挺抽象的，其实它们的区别还是很好理解的。

使用边缘触发模式时，当被监控的 Socket 描述符上有可读事件发生时，服务器端只会从 epoll_wait 中苏醒一次，即使进程没有调用 read 函数从内核读取数据，也依然只苏醒一次，因此我们程序要保证一次性将内核缓冲区的数据读取完；

使用水平触发模式时，当被监控的 Socket 上有可读事件发生时，服务器端不断地从 epoll_wait 中苏醒，直到内核缓冲区数据被 read 函数读完才结束，目的是告诉我们有数据需要读取；

#### 参考

- https://mp.weixin.qq.com/s/Qpa0qXxuIM8jrBqDaXmVNA

## 美团

### 笔试

美团的春招笔试分为5道题，本次做的难度均适中。

#### 第一道

> 给定一个字符串，问根据以下的规则，调整**一次**字符串后(即更改一个字符，能够形成更满足要求的字符串)，会更满足要求 <br/>
> 1. 如果是一个回文串，会更满足要求，或者修改一次字符串后能够形成回文串
> 2. 更改后的字符串更小能够满足要求，例如 `1212` 比 `2212` 更满足要求

**做的时候忽略了一种情况，即形成的本来就是回文串，但是如果是奇数的回文串需要把中间的数字改成 `0` 这样能够形成更小的回文串，满足条件2**

所以只过了 0.18

```java
package meituan;

import java.io.BufferedWriter;
import java.io.IOException;
import java.io.OutputStreamWriter;
import java.util.Scanner;

public class First {

    public static void main(String[] args) throws IOException {
        Scanner sc = new Scanner(System.in);
        BufferedWriter out = new BufferedWriter(new OutputStreamWriter(System.out));
        int T = sc.nextInt();

        for (; T > 0; T--) {
            int n = sc.nextInt();

            String num = sc.next();
            if (n == 1) {
                out.write(String.valueOf(0));
                out.newLine();
                continue;
            }
            char[] nums = num.toCharArray();

            if (isPalindrome(nums)) {
                out.write(new String(nums));
                out.newLine();
                continue;
            }

            nums = num.toCharArray();

            for (int i = 0; i < n; i++) {
                if (nums[i] != '0') {
                    nums[i] = '0';
                    break;
                }
            }
            out.write(new String(nums));
            out.newLine();
        }
        out.flush();
    }

    public static boolean isPalindrome(char[] nums) {
        boolean change = false;
        int i = 0, j = nums.length - 1;
        for (; i < j; i++, j--) {
            if (nums[i] == nums[j]) continue;
            if (!change) {
                change = true;
                nums[i] = getMin(nums[i], nums[j]);
                nums[j] = getMin(nums[i], nums[j]);
            } else {
                return false;
            }
        }
        if (i == j) {
            // 奇数要找到更小 需要把中间一个变为 0
            nums[i] = '0';
        }
        return true;
    }

    public static char getMin(char a, char b) {
        if (a > b) return b;
        return a;
    }
}

```

#### 第二道题

> 给定一个只包含字符 `T` 和 `F` 的字符串，问如果不能出现连续的 3 个以及以上的 `F`，需要多小的代价去修改其中的 `F` 成为 `T`，保障不会出现连续的 3 个`F`，给定更改一个 `F` 成为 `T` 的代价为 c1 和 c2 <br/>
> 例如 `TFFFTT` 需要将中间的 `FFF` 中的一个更改为 `T`，才能保证不会出现连续的 3 个 `F`

所以其实就是直接比较 min(c1, c2) 然后看要变至少多少个 `F`，所以需要统计`连续三个F`的len，每次只需要更改 `len / 3` 个 F 即可

```java
package meituan;

import java.util.Arrays;
import java.util.Scanner;

public class Second {

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt(), c1 = sc.nextInt(), c2 = sc.nextInt();

        int c = Math.min(c1, c2);

        String str = sc.next();

        long res = 0;
        // 只要超过三条的才加入
        // 其实可以直接 split 更方便
        int preIndex = -1;
        for (int i = 0; i < n; i++) {
            if (str.charAt(i) == 'F') {
                if (preIndex == -1) preIndex = i;
            } else {
                if (preIndex != -1) {
                    int len = i - preIndex;
                    if (len >= 3) {
                        res += (long) c * (len / 3);
                    }
                    preIndex = -1;
                }
            }
        }

        if (preIndex != -1) {
            int len = n - preIndex;
            if (len >= 3) {
                res += (long) c * (len / 3);
            }
        }
        System.out.println(res);
    }
}
```

#### 第三道题

> 给定一组输入数组，然后给定一组输入数字，返回这个数字在数组中第一次出现和最后一次出现的下标，不存在则输出 0。<br/>
> 例如: [1,1,2,2,3,3] 1, 2, 3, 4<br/>
> 输出: 1, 2<br/>
>      3, 4<br/>
>      5, 6<br/>
>      0<br/>

没啥好说的 map 完事儿

```java
package meituan;

import java.io.BufferedWriter;
import java.io.IOException;
import java.io.OutputStreamWriter;
import java.util.*;

public class Third {

    public static void main(String[] args) throws IOException {
        Scanner sc = new Scanner(System.in);
        BufferedWriter out = new BufferedWriter(new OutputStreamWriter(System.out));

        int n = sc.nextInt(), m = sc.nextInt();
        Map<Integer, int[]> map = new HashMap<>();
        for (int i = 0; i < n; i++) {
            int val = sc.nextInt();

            if (!map.containsKey(val)) {
                map.put(val, new int[]{i + 1, i + 1});
            } else {
                int[] tmp = map.get(val);
                tmp[1] = i + 1;
                map.put(val, tmp);
            }
        }

        for (int i = 0; i < m; i++) {
            int find = sc.nextInt();
            if (!map.containsKey(find)) {
                out.write(String.valueOf(0));
                out.newLine();
                continue;
            }
            int[] indexes = map.get(find);
            out.write(String.format("%d %d", indexes[0], indexes[1]));
            out.newLine();
        }
        out.flush();
    }
}

```

#### 第四道题

第四道题是个模拟题，具体题目记不清楚了，大概是求一个数组中的一个数及其以后的 K 个数，形成的 异或 结果最大。

```java
package meituan;

import java.util.Scanner;

public class Four {

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        int n = sc.nextInt(), k = sc.nextInt();

        int[] nums = new int[n];

        for (int i = 0; i < n; i++) {
            nums[i] = sc.nextInt();
        }

        int res = 0;

        for (int i = 0; i < n; i++) {
            int tmp = 0;
            for (int j = i; j < Math.min(n, i + k); j++) {
                tmp ^= nums[j];
                res = Math.max(res, tmp);
            }
        }

        System.out.println(res);
    }
}
```

#### 第五道题

> 爬树问题，问如果要去爬树，给定每个树需要消耗的 cost 组成一个数组。现在有一个人选择其中的一棵树开始爬，然后剩下的树的序号为 偶数 的一队人爬，为 奇数 的一队人爬，那么这个人选择哪棵树爬，能够使得爬奇数树和偶数树的人的 cost 是一样的。
> 注：这个人选择爬了这棵树后，后面的树的编号要减一，所以后面的树的奇偶性会变化

其实就是一个遍历的问题，遍历能够作为这个人爬的树的编号，然后分别计算去掉这棵树后，所有的奇数和和偶数和是否一样即可。

但是这个就涉及到，如果快速得到奇数和和偶数和的结果。

所以就用两个数组:

- left 保存从左向右的 奇数树 和 偶数树 的和
- right 保存从右向左的 奇数树 和 偶数树 的和

这样就可以快速得到和的结果

```java
package meituan;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Scanner;

public class Five {

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        int n = sc.nextInt();

        int[] trees = new int[n + 1];

        for (int i = 1; i <= n; i++) {
            trees[i] = sc.nextInt();
        }

        List<Integer> res = new ArrayList<>();

        int[] left = new int[n + 1];
        // 只有编号重新来过
        for (int i = 1; i <= n; i++) {
            if ((i & 1) == 1) {
                left[i] = (i - 2 < 0 ? 0 : left[i - 2]) + trees[i];
            } else {
                left[i] = left[i - 2] + trees[i];
            }
        }
        int[] right = new int[n + 1];

        for (int i = n; i >= 1; i--) {
            right[i] = (i + 2 > n ? 0 : right[i + 2]) + trees[i];
        }

        // 遍历
        for (int i = 1; i <= n; i++) {
            // 用 i 来爬
            int num = trees[i];

            int l = left[i] - num + (i == n ? 0 : right[i + 1]);
            int r = left[i - 1] + right[i] - num;
            if (l == r) res.add(i);
        }

        if (res.size() == 0) {
            System.out.println(0);
        } else {
            System.out.println(res.size());
            StringBuilder builder = new StringBuilder();
            for (int a : res) {
                builder.append(a);
                builder.append(" ");
            }
            System.out.println(builder.toString().trim());
        }
    }
}
```

### 面试

唉，春招成都没 HC 了，是美团优选的，社区团购实在是勾不起兴趣

#### 一面

- 自我介绍
- 项目介绍
  - 在项目接受完了之后，根据项目问了几个问题
  - select poll epoll 的区别
    - select 使用一个 bitmap 来保存 fd 的，其中的每一位表示一个 fd 是否有数据，在调用后，其会被拷贝到内核态，堵塞的等待 bitmap 被更改，也就是意味着数据被读取到。缺点： 1. bitmap 大小 2. fd 要置为，而且不可重用，因为内核会更改数据，每次需要重新置位 3. 拷贝需要用户态到内核态的开销 4. 遍历 bitmap 需要一个 o(n) 的开销
    - poll。不用 bitmap，而是使用 fd 数组，其中保存了 fd 以及关注的读写事件。所以拷贝到内核后，被置位的是关注的`事件`，所以只需要重置这个触发事件即可。
    - epoll。创建一个 epoll_fd，然后在 ctl 放放中在 epoll_fd 上的创建 fd_events 的关注的数据。然后等待数据到达，数据到达之后，会通过重排，将已经就绪的 fd 放到队列的前端。然后 epoll_wait 会返回被出发的 fd 的数量，这样就可以遍历 这个数量 长度即可
  - 责任链模式的实现
- 操作系统
  - 进程、线程、协程的区别
  - 内核态、用户态的切换
- 洋葱架构
- domain模型
- spring
  - spring aop 的实现
    - 动态代理
    - cgclib
    - AspectJ
- mysql
  - 锁
  - next-key 锁
  - 间歇锁 gap
  - b+ 树与 平衡查找树、红黑树的区别
  - 死锁的条件和怎么破坏死锁

## shopee

### 笔试

shopee 的笔试贼难，基本没咋做出来

1. 图问题

> 给了一个二维的矩阵 告诉中间有些矩阵块是黑色的，问如果你从第一行出发 把所有的黑色块遍历完 最少需要几步

思考的方向是贪心，一行一行的遍历，直到遍历完所有的黑色节点

2. 字符串输入问题

> 给定一个输入的字符串 格式如 {名字}\t{数量}\n   （\t \n 都是字符串 即 \\t， 数量是 10进制 or 16进制）  把他format 成 {名字},{数量} 换行符 的格式 （不保证输入一定是上述的格式 需要自己 read 一个 char 进来判断）

3. 跳格子问题

> 跳格子 每次能跳 1 个 或者 2个。但是会输入一组 index 表示这个块不能走，还会输入一组index，这个index可以直接转移到另外一个 index 问达到 n 个格子 有多少种走法

用带 memo 的做的 但是没过多少，估计是 % 算法的时候有点不对，应该是加起来的时候越界了。

```java
package shopee;

import java.util.Arrays;
import java.util.Scanner;

public class Third {

    private static class Stair {
        int index, to;
        boolean isD;

        public Stair(int index) {
            this.index = index;
            this.to = -1;
            this.isD = false;
        }

        public Stair(int index, boolean isD) {
            this.index = index;
            this.to = -1;
            this.isD = isD;
        }

        public Stair(int index, int to) {
            this.index = index;
            this.to = to;
            this.isD = false;
        }
    }

    private static int recursion(Stair[] stairs, int index, int n, int[] memo) {
        if (index == n) {
            // 到达
            return 1;
        }
        if (index > n) return 0;
        if (memo[index] != -1) return memo[index];
        int res = 0;
        if (index + 1 < stairs.length) {
            if (!stairs[index + 1].isD) {
                res = (res + recursion(stairs, index + 1, n, memo)) % 20212021;
                if (stairs[index + 1].to != -1) {
                    res =(res + recursion(stairs, stairs[index + 1].to, n, memo)) % 20212021;
                }
            }
        } else {
            res = (res + recursion(stairs, index + 1, n, memo)) % 20212021;
        }

        if (index + 2 < stairs.length) {
            if (!stairs[index + 2].isD) {
                res = (res + recursion(stairs, index + 2, n, memo)) % 20212021;
                if (stairs[index + 2].to != -1) {
                    res = (res + recursion(stairs, stairs[index + 2].to, n, memo)) % 20212021;
                }
            }
        } else {
            res = (res + recursion(stairs, index + 2, n, memo)) % 20212021;
        }
        memo[index] = res;
        return memo[index];
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        int n = sc.nextInt(), d = sc.nextInt(), m = sc.nextInt();

        Stair[] stairs = new Stair[n];
        for (int i = 0; i < n; i++) {
            stairs[i] = new Stair(i);
        }

        for (int i = 0; i < d; i++) {
            int index = sc.nextInt();
            stairs[index] = new Stair(index, true);
        }

        for (int j = 0; j < m; j++) {
            int from = sc.nextInt(), to = sc.nextInt();
            stairs[from] = new Stair(from, to);
        }
        int[] memo = new int[n];
        Arrays.fill(memo, -1);
        System.out.println(recursion(stairs, 0, n, memo));
    }
}

```

### 面试

只记录了一下二面，一面是在没啥好记的，一面是基础中的基础，特别简单，但是二面很多分布式和中间件，可能和他们做的东西相关，这些就不太了解了

- 消息队列
- 分布式事务
- mysql
  - InnoDB 事务怎么实现的
  - ACID 特性怎么实现的
- mysql 
  - 索引失效的情况
  - 乐观锁 悲观锁
  - mysql 怎么分库分表 什么情况下分库分表
  - 怎么根据用户 ID 进行分库分表，并且如何查找
- 外排序
- 计网
  - 拥塞控制是什么
  - tcp time-wait 是什么阶段
  - linux 查看系统占用
- redis
  - 主从
  - 哨兵
  - 集群
- nginx
  - 找到 top10 访问 IP
- awk 用过嘛

## keep

### 笔试

笔试都很简单，最难的是第三道题，[最短回文串](https://leetcode-cn.com/problems/shortest-palindrome/)

### 面试 

#### 一面

1. 排序算法
2. java 线程池怎么实现的，参数是什么
3. go 协程实现
4. https 握手过程
5. 如果要实现一个类似微博的系统，实现关注、发微博、查看关注的微博等几种操作，怎么设计数据库
6. 如何实现对于一个（忘了 原题是差不多是对于一个有 increase decrease inc des 等四种操作的数据如何实现所有操作都是 o(1)）

#### 二面

1. 项目
2. 场景题：帖子 ID 从 1 -> 一亿，问如何实现推荐的功能
  - 考虑布隆过滤器
  - 减少随机次数
3. 中间件
  - redis redis的跳表（这个跳表就是为了优化链表的查找 o(n) 的问题）
  - redis 的哨兵、redlock
4. 分布式锁
  - 如何保障原子性
  - 如何实现可重入的操作
5. 秒杀系统
