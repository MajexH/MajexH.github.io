---
title: 春招java后端面试总结
category:
  - life
tags:
  - 面试
  - 求职
toc: true
---

# 春招java后端面试总结

面试了3家企业，快手现在知道直接简历没有通过，可能还是做得东西太少，投的太晚了，看来金三银四好像有点道理，哈哈。

面试的步骤一般都是

1. 自我介绍（我是按照自己的学习状况，自己的项目经历，会什么东西来组织的）
2. 基础问答（一般前几面这个东西比较多，java就是jvm、aqs、数据库、锁这些）
3. 项目介绍以及项目深入问（这个后面几面相对较多）
4. 学习方向，以后的打算（一般终面会有）

## 字节跳动面试

字节跳动实习没有笔试，直接从面试开始，每轮面试基本都是问答+算法题1-2道（我在二面没了，不过三面听同学说也差不多）。

### 字节一面

字节一面的面试，很多都是偏向基础的，大致整理问题如下

#### 数据库（我写的mysql，因此从mysql出发问的）

1. Myisam 是否支持聚合索引
   （TODO 待查）
2. Innodb 的索引是什么做的
   （b+树）
3. 聚簇索引 和 非聚簇索引 有什么区别
   1. myisam采用的非聚簇索引，在其b+树的结构中，其叶子节点保存的索引和data域，data域中是存储的具体文件的位置的指针，因此是非聚簇的
   2. innodb是聚簇索引，在b+树的叶子节点中的data域存放的是真正的数据
4. Myisam 与 innodb有什么区别
   1. myisam只支持表级锁，而innodb支持行级锁和表级锁
   2. myisam不支持事务，innodb支持
   3. myisam没有奔溃后的安全恢复，而innodb支持
   4. innodb使用了mvcc（Multiversion concurrency control 多版本并发控制），并发访问（读或者写）数据库时，对正在事务内处理的数据做多版本的管理，用来避免由于写操作的堵塞，而引发读操作失败的并发问题。

#### jvm 

1. 内存区域
   图
   
2. Jdk1.6 1.8的内存区域的区别是什么 为什么要把方法去 放到元数据空间？

3. 垃圾收集的过程

4. cms 是什么
   
5. full gc  是什么 就是哪些gc是可以不做的


#### Java

1. Java的内存使用异常的 比如说上面的爆堆 有哪些情况
   OOM的话 首先要看 OOM出现在哪儿
   1. java的虚拟机栈也是会OOM的，因此虚拟机栈也可以动态的分配空间，当采用这种模式的时候，可能会OOM，那么这个时候可能是，栈帧递归过深
   2. OOM会发生在堆里面，比如老年代现在满了，但是GC的时候无法回收到足够的空间，因此会有问题，可能存在的OOM情况
      - 最简单的情况就是jvm启动的过程中，堆开小了，比如XXX参数给了1M
      - 大家都是强引用，什么无法回收 
      - 内存泄漏，比如ThreadLocal中的ThreadMap就可能内存泄漏，因此ThreadLocalMap的key是**弱引用**，有可能key回收了但是map的value是强引用，没有回收，就直接发生了内存泄漏。 

2. Java的cpu使用异常的话 比如说cpu占用异常高 有哪些情况
   1. 这个我只想到了死循环，因为遇到过 

#### Redis
Redis 相比内存有什么优势
Redis的自动保存数据是怎么做的

#### spring

为什么 spring 采用单例
怎么用多例
使用多例的情况是什么

#### web基础
一个网站从输入到输出页面经历了

#### 项目相关

简单问了问项目

#### 笔试题 

找到股票的最大利润 （不是最大利润是多少 是最大利润的下标是多少,即找到`买入和卖出`的点是什么）[leetcode 股票的最大利润](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

例程如下

```java
public class Main {

   public static void main(String[] args) {
        // 这个获取最大利润的下标就是 1 4 (在下标为 1 买入 下标为 4 卖出)
        int[] inputs = new int[]{ 7,4,5,3,6,1 };

        // minIndex 保存当前最小的值的位置
        int minIndex = -1;
        // maxIndex 保存当前最大的值得位置
        int maxIndex = -1;
        // 保存当前访问到的最大值
        int max = 0;
        // 保存返回的最大小值得位置
        int resMinIndex = -1, resMaxIndex = -1;
        for (int i = 0; i < inputs.length; i++) {
            if (minIndex == -1 || inputs[minIndex] > inputs[i]) {
                minIndex = i;
            }
            if (inputs[i] - inputs[minIndex] > max) {
                maxIndex = i;
                max = inputs[i] - inputs[minIndex];
                resMaxIndex = maxIndex;
                resMinIndex = minIndex;
            }
        }
        System.out.println(resMinIndex + " " + resMaxIndex);
   }
}
```

### 字节二面

字节二面主要问了些项目的问题

因为我写的项目里面有docker，就问了一些docker相关的问题

1. 为什么选择docker，不采用其他的方式
2. docker的隔离是怎么做的（namespace和cgroups）
   1. namespace（命名空间）可以隔离哪些
      1. 文件系统需要是被隔离的
      2. 网络也是需要被隔离的
      3. 进程间的通信也要被隔离
      4. 针对权限，用户和用户组也需要隔离
      5. 进程内的PID也需要与宿主机中的PID进行隔离
      6. 容器也要有自己的主机名
   2. 使用Namespace进行容器的隔离有什么缺点呢？
      1. 最大的缺点就是隔离不彻底
      2. 容器知识运行在宿主机上的一种特殊的进程，那么多个容器之间使用的就还是同一个宿主机的操作系统内核
      3. 在Linux内核中，有很多资源和对象是不能被Namespace化的，最典型的例子是：时间即如果某个容器修改了时间，那整个宿主机的时间都会随之修改
   3. cgroups，cgroups是Linux的另外一个强大的内核工具，有了cgroups，不仅可以限制被namespace隔离起来的资源，`还可以为资源设置权重、计算使用量、操控任务（进程或县城）启停等`。说白了就是：cgroups可以限制、记录任务组所使用的`物理资源`（包括`CPU`，`Memory`，`IO`等）


#### 算法题

算法题是一个典型的二叉树的 zigzagLevelOrder, [leetcode地址](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/) 

```java
public class Second {

   public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        LinkedList<TreeNode> queue = new LinkedList<>();
        LinkedList<Integer> temp = new LinkedList<>();
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) return res;
        boolean left = true;
        queue.add(root);
        queue.add(null);
        
        while (queue.size() > 0) {
            TreeNode top = queue.pollFirst();
            if (top == null) {
                res.add(new ArrayList<>(temp));
                temp.clear();
                if (queue.size() == 0) break;
                queue.add(null);
                left = !left;
                continue;
            }
            if (left) {
                temp.add(top.val);
            } else {
                temp.addFirst(top.val);
            }
            if (top.left != null) queue.add(top.left);
            if (top.right != null) queue.add(top.right);
        }
        return res;
    }
}
```

## 美团面试

### 美团一面

#### servlet是什么、servlet的生命周期、servlet是否单例

##### 是什么

{% blockquote 百度百科%}
Servlet（Server Applet）是Java Servlet的简称，称为小服务程序或服务连接器，用Java编写的服务器端程序，具有独立于平台和协议的特性，主要功能在于交互式地浏览和生成数据，生成动态Web内容。
狭义的Servlet是指Java语言实现的一个接口，广义的Servlet是指任何实现了这个Servlet接口的类，一般情况下，人们将Servlet理解为后者。Servlet运行于支持Java的应用服务器中。从原理上讲，Servlet可以响应任何类型的请求，但绝大多数情况下Servlet只用来扩展基于HTTP协议的Web服务器。
{% endblockquote %}

对于servlet我的了解相对较少，只是在最开始学习的时候了解了动态web项目的构建，学习了一下。当时编写的话，需要编写 servlet （相当于controller），并且需要在 web.xml 里面将请求路径与 servlet 关联，`因此，servlet在其生命周期中是一个单例的`。

{% asset_img servlet-structure.png 哦豁~图没了 %}

其实编写的业务代码位于整体的 servlet 层次，即最后面，如 tomcat 等容器，其不仅作为一个http服务器，能够正常的解析 http 协议，同时，能够提供 servlet 容器的功能，将编写的逻辑 servlet 组织起来，并且能够生成动态的响应。

##### 生命周期

{% asset_img servlet-interface.png 哦豁~图没了 %}

servlet 接口的函数定义如上，其中 init() destroy() 分别在 servlet 容器初始化和销毁的时候调用。

service() 方法作为处理请求的入口，实现该方法，会在请求到来时调用。


#### 如何监控java程序

因为我提到了 jdk 自带的监控命令，因此这儿主要是问的 jdk 的监控命令，如 jps, jstack等

1. jps

{% asset_img jps.png 哦豁~图没了 %}

jps 可以放出现在这个 jvm 正在运行的进程和其 vmid 号

2. jinfo {pid}

可以查看这个进程的 Java System Properties、VM Flags 和 VM Arguments。

3. jstat

{% asset_img jstat.png 哦豁~图没了 %}

可以查看当前进程的一些 gc 情况等

4. jmap / jhat
   
{% asset_img jmap.png 哦豁~图没了 %}

主要是针对当前进程的内存进行分析，可以查看当前进程占用的内存是多少，包括heap的概要信息，GC使用的算法，heap的配置及wise heap的使用情况等

jhat这个工具是用来分析jmap dump出来的文件

5. jstack

java堆栈跟踪工具，线程快照就是当前虚拟机内**每一条线程正在执行的方法堆栈的集合**(包括GC线程)。生成线程快照的主要目的是定位线程出现长时间停顿的原因，`入线程间死锁、死循环、请求外部资源导致的长时间等待`都是导致线程长时间停顿的常见原因。

#### minor gc 和 full gc 触发的条件

{% blockquote %}
当准备要触发一次 young GC时，如果发现统计数据说之前 young GC的平均晋升大小比目前的 old gen剩余的空间大，则不会触发young GC而是转为触发 full GC (因为HotSpot VM的GC里，除了垃圾回收器 CMS的concurrent collection 之外，其他能收集old gen的GC都会同时收集整个GC堆，包括young gen，所以不需要事先准备一次单独的young GC)
{% endblockquote %}

{% blockquote %}
当准备要触发一次 young GC时，如果发现统计数据说之前 young GC的平均晋升大小比目前的 old gen剩余的空间大，则不会触发young GC而是转为触发 full GC (因为HotSpot VM的GC里，除了垃圾回收器 CMS的concurrent collection 之外，其他能收集old gen的GC都会同时收集整个GC堆，包括young gen，所以不需要事先准备一次单独的young GC)
或者当需要在老年代上新开一个对象时，其超过了现在的老年代的剩余的空间，也会触发full gc
{% endblockquote %}

{% blockquote %}
如果有永久代(perm gen),要在永久代分配空间但已经没有足够空间时，也要触发一次 full GC
{% endblockquote %}

{% blockquote %}
System.gc()，heap dump带GC,其默认都是触发 full GC.
(System.gc() 不一定保证能够立即执行 gc 与 Thread.yield 相似)
{% endblockquote %}


#### OOM（outofmemory 是否可以捕获）

`OutOfMemoryError extends VirtualMachineError extends Error`

所以是不可以处理的异常，因此是`无法捕获`的

OOM可能发生的原因

1. 分配的少了：比如虚拟机本身可使用的内存（一般通过启动时的VM参数指定）太少。
	 
2. 应用用的太多，并且用完没释放，浪费了。此时就会造成内存泄露或者内存溢出。
	
	内存泄露：申请使用完的内存没有释放，导致虚拟机不能再次使用该内存，此时这段内存就泄露了，因为申请者不用了，而又不能被虚拟机分配给别人用。
	（如 ThreadLocal的内存泄漏问题）

   内存溢出：申请的内存超出了JVM能提供的内存大小，此时称之为溢出。

3. java的虚拟机栈如果采用自动分配，如果超过大小，也会产生OOM

### 笔试题

数字转换成字符串有多少种转换方式

其中转换规则为 1 = a , 2 = b, …… , 26 = z

实际上这就是个 dp 问题

```java
public class First {

    public int resolve(String input) {
        // dp 表示以 input[i] 开始的能够形成多少个转换
        int[] dp = new int[input.length() + 1];
        // 这个地方为 1 其实表示了 这种划分 能够把前面的数字正好全部划分完 是一种可能的转换
        dp[input.length()] = 1;
        for (int i = dp.length - 1; i >= 0; i--) {
            // 因为字母最大到 26 因此最多只需要找两位数字即可
            for (int j = 1; j <= 2; j++) {
                if (i + j <= input.length() && valid(input.substring(i, i + j)))
                    dp[i] += dp[i + j];
            }
        }
        return dp[0];
    }

    public boolean valid(String str) {
        // 可能存在 101 这个时候 1 0 1 单独分开是不行的 而且 1 01 也不行
        if (str.length() == 0 || str.startsWith("0")) return false;
        return Integer.parseInt(str) >= 1 && Integer.parseInt(str) <= 26;
    }
    public static void main(String[] args) {
        System.out.println(new Second().resolve("101"));
    }
}

```