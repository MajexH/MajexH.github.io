---
title: hash-code
category:
  - code
tags:
  - hash
toc: false
date: 2021-03-13 15:09:08
thumbnail:
password:
---

## 简单实现了 hash 容器

分别采用拉链法和开放寻址法

<!-- more -->

### hashset

leetcode 例题，设计一个 [hash 集合](https://leetcode-cn.com/problems/design-hashset/)，不能使用 built-in 的 hash 类

这边使用了开放寻址法简单是实现了一个 hashset，但是没有实现整理以及清除。

开放寻址法，实际上是一个使用数组，根据 key 的 hashcode 进行定位存储。

针对 hash 碰撞，会线性的寻找下一个能够放入的地址放入，而如果 删除其中一个元素，直接将其引用删除的话，会造成碰撞链的断裂，因此删除是使用`特殊的标识位`来表示的。

```java
    // 不能使用任何 built-in 的 hash 库
    private static class Pair {
        Object val;
        boolean isDeleted;
        int hashcode;

        public Pair(Object val, boolean isDeleted, int hashcode) {
            this.val = val;
            this.isDeleted = isDeleted;
            this.hashcode = hashcode;
        }
    }

    private Pair[] objs;
    private int size;

    /**
     * Initialize your data structure here.
     */
    public MyHashSet() {
        this.objs = new Pair[16];
    }

    public void add(int key) {
        if (++size >= this.objs.length) {
            // 先判断增长
            grow();
        }
        int addedIndex = getAddedIndex(key);
        if (addedIndex != this.objs.length) {
            this.objs[addedIndex] = new Pair(key, false, Integer.hashCode(key));
            return;
        }
        // 这个时候也应该 grow 因为没有地方放置了
        // 按道理说 这个时候应该去扫描前后 去除无用的引用
        grow();
        add(key);
    }

    private int getAddedIndex(int key) {
        int i = Integer.hashCode(key) & (this.objs.length - 1);
        for (; i < this.objs.length; i++) {
            if (this.objs[i] == null || (int) this.objs[i].val == key || this.objs[i].isDeleted) {
                return i;
            }
        }
        return i;
    }

    private void grow() {
        int newLen = this.objs.length << 1;
        Pair[] objs = new Pair[newLen];

        for (Pair old : this.objs) {
            if (old == null) continue;
            // 删除的就不管了
            if (old.isDeleted) continue;
            int j = old.hashcode & (newLen - 1);
            for (; j < newLen; j++) {
                if (objs[j] == null) {
                    objs[j] = old;
                    break;
                }
            }
        }
        this.objs = objs;
    }

    // 获取 remove contains 的 index
    // 其实可以用 lambda 表达式来传入不同的函数 将两个 index 整合
    private int getOtherIndex(int key) {
        int j = Integer.hashCode(key) & (this.objs.length - 1);
        // 开放寻址法 继续往后找
        for (; j < this.objs.length; j++) {
            if (this.objs[j] != null && (int) this.objs[j].val == key) {
                return j;
            }
        }
        return j;
    }

    public void remove(int key) {
        int j = getOtherIndex(key);
        if (j < this.objs.length) {
            this.objs[j].isDeleted = true;
            --size;
        }
    }

    /**
     * Returns true if this set contains the specified element
     */
    public boolean contains(int key) {
        int j = getOtherIndex(key);
        if (j < this.objs.length) {
            return !this.objs[j].isDeleted;
        }
        return false;
    }
}
```

### hashMap

[leet code 例题](https://leetcode-cn.com/problems/design-hashmap/)

采用拉链法处理，hash 碰撞连接到原来的链上。

同时现在是采用的头插法来处理的，头插法会在多线程情况下GG，最好不用这样用。

```java
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.OutputStreamWriter;
import java.io.Writer;
import java.util.Arrays;
import java.util.HashMap;

public class MyHashMap {

    // 拉链法
    private static class Node {
        int key, val;
        int hashcode;
        Node next;

        public Node(int key, int val, int hashcode) {
            this.key = key;
            this.val = val;
            this.hashcode = hashcode;
            this.next = null;
        }

        @Override
        public String toString() {
            return "Node{" +
                    "key=" + key +
                    ", val=" + val +
                    ", hashcode=" + hashcode +
                    ", next=" + next +
                    '}';
        }
    }

    private int DEFAULT_CAPACITY = 16;

    private Node[] nodes;
    private int size;
    // 负载因子
    private double factor = 0.75;
    private int threshold = (int) (DEFAULT_CAPACITY * factor);


    public MyHashMap() {
        this.nodes = new Node[16];

    }

    /**
     * value will always be non-negative.
     */
    public void put(int key, int value) {
        int hashcode = Integer.hashCode(key);
        int index = hashcode & (this.nodes.length - 1);
        if (!headPut(index, hashcode, key, value, nodes)) return;
        if (++size >= threshold) grow();
    }

    // 返回加入成功或者失败
    private boolean headPut(int index, int hashcode, int key, int value, Node[] nodes) {
        // 头插法插入
        Node tmp = nodes[index];
        while (tmp != null) {
            // 找到重复 key 跳出
            if (tmp.key == key) {
                tmp.val = value;
                return false;
            }
            tmp = tmp.next;
        }
        Node newHead = new Node(key, value, hashcode);
        newHead.next = nodes[index];
        nodes[index] = newHead;
        return true;
    }

    // 重新定位
    private void grow() {
        int newCapacity = this.nodes.length << 1;
        int newThreshold = (int) (newCapacity * this.factor);
        int oldCapacity = this.nodes.length;

        Node[] newNodes = new Node[newCapacity];
        this.threshold = newThreshold;
        // 头插法比较简单 但是会有问题
        // 实际上用一个 NodeHolder 抓住整个 node 保存 头尾 更方便
        for (int i = 0; i < oldCapacity; i++) {
            Node old = this.nodes[i];
            while (old != null) {
                // 根据原来的 hashcode 进去在新数组的位置 下面这个是由数学特性得到的
                int newIndex = (old.hashcode & oldCapacity) == 0 ? i : i + oldCapacity;
                headPut(newIndex, old.hashcode, old.key, old.val, newNodes);
                old = old.next;
            }
            this.nodes[i] = null;
        }
        this.nodes = newNodes;
    }

    /**
     * Returns the value to which the specified key is mapped, or -1 if this map contains no mapping for the key
     */
    public int get(int key) {
        int hashcode = Integer.hashCode(key);
        int index = hashcode & (this.nodes.length - 1);

        Node tmp = this.nodes[index];

        while (tmp != null) {
            if (tmp.key == key) return tmp.val;
            tmp = tmp.next;
        }
        return -1;
    }

    /**
     * Removes the mapping of the specified value key if this map contains a mapping for the key
     */
    public void remove(int key) {
        int hashcode = Integer.hashCode(key);
        int index = hashcode & (this.nodes.length - 1);

        Node tmp = this.nodes[index];

        if (tmp == null) return;
        // 头是 key
        if (tmp.key == key) {
            this.nodes[index] = tmp.next;
            --size;
            return;
        }

        // 开始删除
        Node mv = tmp.next;
        while (mv != null) {
            if (mv.key == key) {
                tmp.next = mv.next;
                --size;
                return;
            }
            mv = mv.next;
            tmp = tmp.next;
        }
        // 没有找到
    }


    public static void main(String[] args) throws IOException {
        MyHashMap map = new MyHashMap();
        Writer out = new BufferedWriter(new OutputStreamWriter(System.out));
        out.write("[null,");
        map.put(504, 155);
        map.remove(89);
        out.write("," + String.valueOf(map.get(334)));
        out.flush();
    }
}

```
