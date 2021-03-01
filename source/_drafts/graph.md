---
title: 简单图算法
category:
  - 算法
tags:
  - 图算法
toc: true
---

# 图算法

最简单的 DFS、BFS 一笔带过了，下面先给出几种图的定义，图均使用邻接表表示

## 图的数据结构

以下均使用邻接表标识

- 无向图

无向图设计得有点尴尬 其实没必要

```java
public class UndirectedGraph<T> implements Graph<T> {

    public HashMap<T, UndirectedNode<T>> map;

    public UndirectedGraph() {
        this.map = new HashMap<>();
    }

    public Set<T> keys() {
        return map.keySet();
    }

    // 无向图 两遍都要加
    public void addEdge(T from, T to) {
        if (!map.containsKey(from)) {
            map.put(from, new UndirectedNode<>(to));
        }
        UndirectedNode<T> temp = map.get(from);
        // 说明加入了重复边
        if (!temp.addNode(to)) return;

        if (!map.containsKey(to)) {
            map.put(to, new UndirectedNode<>(from));
        }
        UndirectedNode<T> tempTo = map.get(to);
        tempTo.addNode(from);
    }

    public Node<T> adjacent(T from) {
        return this.map.getOrDefault(from, null);
    }
}

public class UndirectedNode<T> implements Node<T> {

    public T to;
    public UndirectedNode<T> next;

    public UndirectedNode(T to) {
        this.to = to;
        this.next = null;
    }

    /**
     *
     * @param to
     * @return boolean 表示是否加入成功
     */
    public boolean addNode(T to) {
        // 加入第一个节点的时候的判断
        if (this.to == to) return true;
        UndirectedNode<T> temp = this.next;
        // 加入第二个节点时的判断
        if (temp == null) {
            this.next = new UndirectedNode<>(to);
            return true;
        }
        while (temp.next != null) {
            if (temp.to == this.to) return false;
            temp = temp.next;
        }
        temp.next = new UndirectedNode<>(to);
        return true;
    }
}
```

- 有向图

```java
public class DirectedGraph {

    public int getCapacity() {
        return capacity;
    }

    private int capacity;
    private List<LinkedList<Integer>> nodes;

    public DirectedGraph(int capacity) {
        this.capacity = capacity;
        this.nodes = new ArrayList<>(capacity);
        for (int i = 0; i < capacity; i++) {
            nodes.add(new LinkedList<>());
        }
    }

    public LinkedList<Integer> adj(int node) {
        return this.nodes.get(node);
    }

    public void addEdge(int from, int to) {
        this.nodes.get(from).add(to);
    }

    public DirectedGraph reverse() {
        DirectedGraph reversed = new DirectedGraph(this.capacity);
        for (int from = 0; from < this.capacity; from++) {
            for (Integer to : adj(from)) {
                if (to != null) {
                    reversed.addEdge(to, from);
                }
            }
        }
        return reversed;
    }
}
```

- 加权无向图

```java
public class UndirectedWeightGraph {

    public int getCapacity() {
        return capacity;
    }

    private int capacity;
    // 邻接表
    private List<LinkedList<Edge>> nodes;

    public UndirectedWeightGraph(int capacity) {
        this.capacity = capacity;
        this.nodes = new ArrayList<>(capacity);
        for (int i = 0; i < capacity; i++) {
            this.nodes.add(new LinkedList<>());
        }
    }

    public LinkedList<Edge> adj(int node) {
        return this.nodes.get(node);
    }

    public void addEdge(int from, int to, int weight) {
        Edge edge = new Edge(from, to, weight);
        this.nodes.get(from).add(edge);
        this.nodes.get(to).add(edge);
    }

}

public class Edge {
    public int from;
    public int to;
    public int weight;

    public Edge(int from, int to, int weight) {
        this.from = from;
        this.to = to;
        this.weight = weight;
    }

    // 根据传入的值 返回不同的 from to
    public int other(int in) {
        // 根据传入的端点 找到链接的另外一个断点
        if (in == from) {
            return to;
        }
        return from;
    }

    @Override
    public String toString() {
        return "Edge{" +
                "from=" + from +
                ", to=" + to +
                ", weight=" + weight +
                '}';
    }
}
```

- 加权有向图

TODO:

```java
```
## 图的遍历方法

以无线图为例，有向图和无向图的遍历基本一样

- DFS

DFS 即使用栈的思想

```java
public class DFS {

    public static <T> void DFSWithRecursion(Graph<T> graph) {
        Set<T> memo = new HashSet<>();
        for (T start : graph.keys()) {
            if (!memo.contains(start))
                DFSWithRecursion(graph, start, memo);
        }
    }

    public static <T> void DFSWithoutRecursion(Graph<T> graph) {
        Set<T> memo = new HashSet<>();
        for (T start : graph.keys()) {
            if (!memo.contains(start))
                DFSWithOutRecursion(graph, start, memo);
        }
    }

    public static <T> void DFSWithRecursion(Graph<T> graph, T start, Set<T> memo) {
        System.out.println(start);
        UndirectedNode<T> node = (UndirectedNode<T>) graph.adjacent(start);
        memo.add(start);
        while (node != null) {
            if (!memo.contains(node.to)) DFS.DFSWithRecursion(graph, node.to, memo);
            node = node.next;
        }
    }

    public static <T> void DFSWithOutRecursion(Graph<T> graph, T start, Set<T> memo) {
        LinkedList<T> stack = new LinkedList<>();
        stack.add(start);
        memo.add(start);
        while (stack.size() != 0) {
            T top = stack.removeLast();
            System.out.println(top);

            UndirectedNode<T> temp = (UndirectedNode<T>) graph.adjacent(top);
            while (temp != null) {
                if (!memo.contains(temp.to)) {
                    stack.add(temp.to);
                    memo.add(temp.to);
                }
                temp = temp.next;
            }
        }
    }

}
```

- BFS

bfs 使用队列的思想

```java
public class BFS {

    public static <T> void BFS(Graph<T> graph) {
        Set<T> memo = new HashSet<>();
        for (T start : graph.keys()) {
            if (!memo.contains(start))
                BFS(graph, start, memo);
        }
    }

    public static <T> void BFS(Graph<T> graph, T start, Set<T> memo) {
        LinkedList<T> queue = new LinkedList<>();
        queue.add(start);
        memo.add(start);
        while (!queue.isEmpty()) {
            T first = queue.removeFirst();
            System.out.println(first);
            UndirectedNode<T> temp = (UndirectedNode<T>) graph.adjacent(first);
            while (temp != null) {
                if (!memo.contains(temp.to)) {
                    queue.add(temp.to);
                    memo.add(temp.to);
                }
                temp = temp.next;
            }
        }
    }
}
```

## 图遍历方法的应用

- 连通分量 

```java
// 连通分量
public class Connected<T> {

    // 如果连个端点属于一个连通分量
    // 他们的id应该是一样的
    public HashMap<T, Integer> ids;
    public int count = 0;
    public Graph<T> graph;
    public Set<T> memo;

    public Connected(Graph<T> graph) {
        this.ids = new HashMap<>();
        this.graph = graph;
        this.memo = new HashSet<>();

        for (T start : graph.keys()) {
            if (!this.memo.contains(start)) {
                dfs(start);
                count++;
            }
        }
    }

    public void dfs(T start) {
        this.memo.add(start);
        ids.put(start, this.count);
        UndirectedNode<T> temp = (UndirectedNode<T>) this.graph.adjacent(start);
        while (temp != null) {
            if (!memo.contains(temp.to)) dfs(temp.to);
            temp = temp.next;
        }
    }

    public boolean connected(T from, T to) {
        return ids.get(from).equals(ids.get(to));
    }
}
```

- 成环检测

```java
public class CheckCycle<T> {

    // memo 记录在dfs的过程中的节点
    public Set<T> memo;

    public Graph<T> graph;

    public boolean hasCycle = false;

    public CheckCycle(Graph<T> graph) {
        this.graph = graph;
        this.memo = new HashSet<>();
        for (T start : graph.keys()) {
            if (!this.memo.contains(start)) {
                dfs(start, start);
            }
        }
    }

    public void dfs(T start, T parent) {
        memo.add(start);
        UndirectedNode<T> temp = (UndirectedNode<T>) graph.adjacent(start);
        while (temp != null) {
            if (!memo.contains(temp.to))
                dfs(temp.to, start);
            // 因为是无向图 因此在访问的时候 会在子节点上 重新访问父节点过来的那条边 因此这样记录父节点即可
            else if(temp.to != parent)
                hasCycle = true;
            temp = temp.next;
        }
    }
}
```

- 拓扑排序

拓扑排序对于排队、课程安排之类的有帮助，其基于有向图实现

首先要做的就是有向图成环检测，因此成环是没有 拓扑排序 的

1. 有向图成环检测

```java
public void DFS(DirectedGraph g) {
      boolean[] memo = new boolean[g.getCapacity()];
      for (int i = 0; i < g.getCapacity(); i++) {
          if (!memo[i]) {
              DFSRecursion(g, i, memo, new boolean[g.getCapacity()]);
          }
      }
  }

private void DFSRecursion(DirectedGraph g, int start, boolean[] memo, boolean[] marked) {
    memo[start] = true;
    System.out.printf("%s ", start);
    marked[start] = true;
    for (int i : g.adj(start)) {
        if (!marked[i]) {
            DFSRecursion(g, i, memo, marked);
        } else {
          // 已经成环
          this.cycle = true;
        }
    }
    marked[start] = false;
}
```

2. 拓扑排序

有两种方法

逆后续排列。因为要找到 v->w 这种拓扑结果，那么在访问完 V 之后 访问 W 即其连接节点，用stack 来保存访问顺序，再弹出栈 就可以得到 v -> w 的顺序

```java
public static List<Integer> topologySortWithRecursion(DirectedGraph g) {
    // 作为一个栈
    List<Integer> stack = new ArrayList<>();
    boolean[] memo = new boolean[g.getCapacity()];
    for (int i = 0; i < g.getCapacity(); i++) {
        if (!memo[i]) {
            // 根据一个出发点 找到其 拓扑排序
            topologySortRecursion(g, stack, i, memo);
        }
    }

    List<Integer> res = new ArrayList<>();
    // 输出 stack
    for (int i = stack.size() - 1; i >= 0; i--) {
        res.add(stack.get(i));
    }
    return res;
}

private static void topologySortRecursion(DirectedGraph g, List<Integer> res, int start, boolean[] memo) {
    memo[start] = true;
    for (int adj : g.adj(start)) {
        if (!memo[adj]) {
            topologySortRecursion(g, res, adj, memo);
        }
    }
    res.add(start);
}
```

遍历入度为 0 的点，因为能够作为开始节点的点，一定入度为 0，那么不停的遍历，删除边，维护一个入度为 0 的点的 collection 既可找到访问顺序

```java
// 2. 不停地遍历入度为 0 的点 然后删除
public static List<Integer> topologySortIteration(DirectedGraph g) {

    int[] inDegree = new int[g.getCapacity()];

    for (int i = 0; i < g.getCapacity(); i++) {
        for (int adj : g.adj(i)) {
            inDegree[adj] += 1;
        }
    }

    Deque<Integer> inDegreeEqualsZero = new LinkedList<>();

    // 找到为 0 的点
    for (int i = 0; i < g.getCapacity(); i++) {
        if (inDegree[i] == 0) {
            inDegreeEqualsZero.offer(i);
        }
    }

    List<Integer> res = new ArrayList<>();

    while (!inDegreeEqualsZero.isEmpty()) {
        int top = inDegreeEqualsZero.poll();
        res.add(top);
        for (int adj : g.adj(top)) {
            inDegree[adj] -= 1;
            if (inDegree[adj] == 0) {
                inDegreeEqualsZero.offer(adj);
            }
        }
    }

    return res;

}
```

- 有向图的强连通分量 

```java
public class DirectedGraphStrongConnected {

    public static List<Integer> KosarajuConnected(DirectedGraph g) {

        // 保存的强连通分量的 id
        // ids[i] 表示 i 节点属于哪个 强连通分量 id
        List<Integer> ids = new ArrayList<>(g.getCapacity());
        for (int i = 0; i < g.getCapacity(); i++) {
            ids.add(-1);
        }
        // 遍历的时候的强连通分量 id
        int count = 0;
        // 先得到 反图 的拓扑排序
        List<Integer> order = TopologySort.topologySortWithRecursion(g.reverse());
        boolean[] memo = new boolean[g.getCapacity()];
        for (int node : order) {
            if (!memo[node]) {
                recursionDFS(g, node, count, memo, ids);
                count++;
            }
        }
        // 根据拓扑排序 DFS
        return ids;
    }

    private static void recursionDFS(DirectedGraph g, int start, int count, boolean[] memo, List<Integer> ids) {
        memo[start] = true;
        ids.set(start, count);
        for (int adj : g.adj(start)) {
            if (!memo[adj]) {
                recursionDFS(g, adj, count, memo, ids);
            }
        }
    }

    public static void main(String[] args) {
        DirectedGraph g = new DirectedGraph(13);
        g.addEdge(1, 2);
        g.addEdge(3, 1);
        g.addEdge(6, 3);
        g.addEdge(4, 7);
        g.addEdge(2, 0);
        g.addEdge(11, 8);
        g.addEdge(10, 1);
        g.addEdge(0, 7);
        g.addEdge(0, 6);


        System.out.println(KosarajuConnected(g));
    }
}

```

## 加权图

### 无向加权图

结果如上述，无向加权图可以用作 电线 呀之类的规划。

#### 最小生成树

最小生成树都是基于贪心的思路和想法

- prim 算法