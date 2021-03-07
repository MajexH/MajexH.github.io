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

```java
public class Edge {

    int from, to;
    int weight;

    public Edge(int from, int to, int weight) {
        this.from = from;
        this.to = to;
        this.weight = weight;
    }

    public int getFrom() {
        return from;
    }

    public void setFrom(int from) {
        this.from = from;
    }

    public int getTo() {
        return to;
    }

    public void setTo(int to) {
        this.to = to;
    }

    public int getWeight() {
        return weight;
    }

    public void setWeight(int weight) {
        this.weight = weight;
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

public class DirectedWeightGraph {

    public int getCapacity() {
        return capacity;
    }

    int capacity;
    List<List<Edge>> nodes;

    public DirectedWeightGraph(int capacity) {
        this.capacity = capacity;
        this.nodes = new ArrayList<>(this.capacity);
        for (int i = 0; i < capacity; i++) {
            this.nodes.add(new LinkedList<>());
        }
    }

    // 有向图只用加入一遍
    public void addEdge(int from, int to, int weight) {
        this.nodes.get(from).add(new Edge(from, to, weight));
    }

    public void addEdge(Edge edge) {
        this.nodes.get(edge.from).add(new Edge(edge.from, edge.to, edge.weight));
    }

    public List<Edge> adj(int node) {
        return this.nodes.get(node);
    }

    // 返回所有的边
    public List<Edge> edges() {
        List<Edge> res = new ArrayList<>();
        for (int i = 0; i < capacity; i++) {
            res.addAll(this.nodes.get(i));
        }
        return res;
    }
}
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

### 无向图连通分量

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

### 成环检测

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

### 拓扑排序

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

逆后续排列。因为要找到 v->w 这种拓扑结果，那么在访问完 V 之后 访问 W 即其连接节点，用 stack 来保存访问顺序，再弹出栈 就可以得到 v -> w 的顺序

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

### 有向图的强连通分量

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

### 最小生成树

最小生成树都是基于贪心的思路和想法。由于需要生成的无向加权图的树（v-1 条边）的路径和最短，所以实际上是一个不断遍历最短路径边的贪心策略。

#### prim 算法

- lazy 版本

在遍历所有的边的时候，不主动删除队列中的无效边，所以为 lazy 实现

```java
public class MinTree {

    // 最小生成树 指的是无向图中，能够生成的边的权重总和最小的树

    // prime 的最小生成树实际上是一个贪心算法
    // 通过 PriorityQueue 不停地生成权重最小的边
    // lazyPrim 指的是在遍历的过程中 pq 中的边 是访问时才失效
    public static Deque<Edge> lazyPrim(UndirectedWeightGraph g) {
        // 存储所有的边
        PriorityQueue<Edge> minQueue = new PriorityQueue<>(Comparator.comparingInt(a -> a.weight));
        // 保存被访问过的节点
        boolean[] marked = new boolean[g.getCapacity()];
        // 保存返回的结果
        Deque<Edge> res = new LinkedList<>();

        // 保证 g 整体是连通的
        // 随意选取一个节点作为开始节点
        addEdgeToPQ(marked, minQueue, 0, g);

        while (!minQueue.isEmpty()) {
            // 抓到的一定是最短的路径
            Edge top = minQueue.poll();

            int from = top.from, to = top.to;
            // from to 两个断点都已经访问过 说明在两个端点之间已经找到最短的了
            if (marked[from] && marked[to]) continue;
            // 否则就找到最短的
            res.add(top);

            // 将两个端点的 edge 加入到 queue 中
            if (!marked[from]) addEdgeToPQ(marked, minQueue, from, g);
            if (!marked[to]) addEdgeToPQ(marked, minQueue, to, g);
        }
        return res;
    }

    public static void addEdgeToPQ(boolean[] marked, PriorityQueue<Edge> minQueue, int start, UndirectedWeightGraph g) {
        marked[start] = true;

        for (Edge e : g.adj(start)) {
            if (!marked[e.other(start)]) {
                // 另外一个断点没有访问过
                minQueue.add(e);
            }
        }
    }
}
```

- 及时版本

在遍历的时候，不再以边作为 优先队列 的遍历对象，而是采用对点进行遍历，在遍历的图中，不停的更新 优先队列 中点对应的最短边，以此减少调整堆的时间。

```java
private static class Pair {
    int node;
    int weight;

    public Pair(int node, int weight) {
        this.node = node;
        this.weight = weight;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Pair pair = (Pair) o;
        return node == pair.node;
    }

    @Override
    public int hashCode() {
        return Objects.hash(node);
    }
}

// 实时的 prim 算法 与 lazy 不同的是
// 在 v 这个节点 加入 pq 的时候 其余的非树阶段 应该只加入 到 树中最短的边
// 相对来说 可以减少时间 因为 一个图的话 一般是 边比点多
// 这个算法只用在 优先队列中保存点
public static Edge[] prim(UndirectedWeightGraph g) {
    // 仍然使用一个 pq 保存最短的点 （到这个点的最短距离）
    PriorityQueue<Pair> pq = new PriorityQueue<>(Comparator.comparingInt(e -> e.weight));

    // edge[i] 保存 i 到 edge.to 的最短边长
    Edge[] edgeTo = new Edge[g.getCapacity()];
    boolean[] marked = new boolean[g.getCapacity()];

    pq.add(new Pair(0, 0));

    while (!pq.isEmpty()) {
        Pair top = pq.poll();
        inTimeAddEdgeToPQ(g, top.node, marked, edgeTo, pq);
    }
    return edgeTo;
}

private static void inTimeAddEdgeToPQ(UndirectedWeightGraph g, int node, boolean[] marked, Edge[] edgeTo, PriorityQueue<Pair> pq) {
    marked[node] = true;

    for (Edge adj : g.adj(node)) {
        int otherNode = adj.other(node);
        // 已经找到了
        if (marked[otherNode]) continue;
        // 说明还没有找到到这个点的最短距离 或者
        // 现在的 edge 的 weight 更短 更新
        if (edgeTo[otherNode] == null || adj.weight < edgeTo[otherNode].weight) {
            edgeTo[otherNode] = adj;
            Pair p = new Pair(otherNode, adj.weight);
            for (Pair tmp : pq) {
                // remove 掉已经失效的边
                if (tmp.node == otherNode) {
                    pq.remove(tmp);
                }
                break;
            }
            pq.add(p);
        }
    }
}
```

- krusal 算法

与 lazy prim 算法类似，其也是遍历所有的边并加入到 优先队列 中，但是遍历的时候采用的方法是通过 并查集 判断点是否已经找到了最短的距离，在找到最短距离后，会判断两个点相连，知道结果边集合大小扩展到 v-1。

```java
public class MinTreeKruskal {

    // 使用 并查集 来判断加入的边是否成环
    public static List<Edge> kruskalUseUnion(UndirectedWeightGraph g) {
        List<Edge> res = new ArrayList<>();
        Union uf = new Union(g.getCapacity());

        PriorityQueue<Edge> pq = new PriorityQueue<>(Comparator.comparingInt((e) -> e.weight));
        pq.addAll(g.getEdges());

        // 最小生成树 只能有 V - 1 个 （V 为 node 数）
        while (!pq.isEmpty() && (res.size() < g.getCapacity() - 1)) {
            Edge e = pq.poll();
            assert e != null;
            int from = e.from, to = e.to;
            // 之前已经连接了最短的边 所以不用再连接了
            if (uf.connected(from, to)) continue;
            // 连接两条边
            uf.union(from, to);
            res.add(e);
        }
        return res;
    }
}

public class Union {
    // 并查集
    int[] parents;

    public Union(int capacity) {
        this.parents = new int[capacity];
        for (int i = 0; i < capacity; i++) {
            // 初始化
            this.parents[i] = i;
        }
    }

    public void union(int n1, int n2) {
        int rootOfN1 = find(n1);
        int rootOfN2 = find(n2);

        if (rootOfN1 == rootOfN2) return;
        // n1 root 连接到 n2 root 上
        this.parents[rootOfN1] = rootOfN2;
    }

    // 找到跟节点
    private int find(int node) {
        if (this.parents[node] == node) {
            return node;
        }
        return find(this.parents[node]);
    }

    public boolean connected(int i, int j) {
        // 判断两个 root 是否相等
        int rootOfI = find(i);
        int rootOfJ = find(j);

        return rootOfI == rootOfJ;
    }
}
```

### 最短路径

最短路径其实跟上述的算法类似，也是一个类似贪心的策略，但是在遍历最短边的时候，或同时使用 relax 的操作，保障一个点经过一个中间点可能比直接到目标点的距离短这个问题。

- Dijkstra 算法

基本与 prim 算法一样，只是加入了 relax 的操作

其只能处理非负的有向图

```java
public static class Pair {
    int node;
    int weight;

    public Pair(int node, int weight) {
        this.node = node;
        this.weight = weight;
    }
}

// dijkstra 与 in time 的 prim 算法类似
// 这些是找到单源最短路的
public static Edge[] dijkstraMinPath(DirectedWeightGraph g, int start) {
    // 保存最短路径的边
    Edge[] edgeTo = new Edge[g.getCapacity()];
    // 保存最短路径的长度
    int[] distTo = new int[g.getCapacity()];
    for (int i = 0; i < g.getCapacity(); i++) {
        distTo[i] = Integer.MAX_VALUE;
    }
    distTo[start] = 0;

    PriorityQueue<Pair> pq = new PriorityQueue<>(Comparator.comparingInt((e) -> e.weight));

    pq.add(new Pair(start, 0));
    // 每次都找现在最短的路径 然后 relax 路径
    while (!pq.isEmpty()) {
        Pair min = pq.poll();
        relax(g, min.node, pq, edgeTo, distTo);
    }
    return edgeTo;
}

// relax 节点
private static void relax(DirectedWeightGraph g, int node, PriorityQueue<Pair> pq, Edge[] edgeTo, int[] distTo) {
    for (Edge adj : g.adj(node)) {
        // 因为 edgeTo 保存的是之前遍历的最短的路径
        // 所以 如果通过现在这个点 + adj.weight 的距离 比 edgeTo 的短 就需要更新
        if (distTo[adj.to] > distTo[node] + adj.weight) {
            edgeTo[adj.to] = adj;
            distTo[adj.to] = distTo[node] + adj.weight;
            // 添加新的 或者 更新原来的节点的最小值
            Pair p = new Pair(adj.to, distTo[adj.to]);
            // 更新 pq
            for (Pair tmp : pq) {
                if (tmp.node == adj.to) {
                    pq.remove(tmp);
                }
                break;
            }
            pq.add(p);
        }
    }
}
```

- 拓扑排序处理无环图

由于拓扑排序的性质，是从入度为 0 的点不断向外延伸，所以，如果根据 拓扑排序 的顺序访问图中的点，那么后面的点是一定不会再访问已经访问过的点，所以不会出现 relax。

```java
// 采用 dfs 做的 topologySort
// 保证无环
public static int[] topologySort(DirectedWeightGraph g) {
    boolean[] marked = new boolean[g.getCapacity()];

    Deque<Integer> stack = new LinkedList<>();

    for (int i = 0; i < g.getCapacity(); i++) {
        // marked 标识标记过的点
        if (!marked[i]) {
            dfs(g, marked, i, stack);
        }
    }
    int[] res = new int[stack.size()];
    int i = 0;
    while (!stack.isEmpty()) {
        res[i++] = stack.removeLast();
    }
    return res;
}

public static void dfs(DirectedWeightGraph g, boolean[] marked, int start, Deque<Integer> stack) {
    marked[start] = true;

    for (Edge adj : g.adj(start)) {
        if (!marked[adj.to]) {
            dfs(g, marked, adj.to, stack);
        }
    }
    stack.addLast(start);
}

// 使用拓扑排序的单源最短路经
// 只能处理无环的情况
// 拓扑排序只能针对无环图
// 由于 拓扑排序是从 无入度的点开始
// 如果找最短路径从这儿开始的话 这个点 一定不会再次被访问到 所以只放松一次
// 同理 解决单点无环图的最长路径 可以把 weight 取负 再来最短路径即可
public static Edge[] topologyMinPath(DirectedWeightGraph g, int start) {

    int[] topologyPath = TopologySort.topologySort(g);
    Edge[] res = new Edge[g.getCapacity()];
    // 初始化距离
    int[] dst = new int[g.getCapacity()];
    Arrays.fill(dst, Integer.MAX_VALUE);
    dst[start] = 0;

    for (int node : topologyPath) {
        relax(g, res, dst, node);
    }
    return res;
}

private static void relax(DirectedWeightGraph g, Edge[] res, int[] dst, int node) {

    for (Edge adj : g.adj(node)) {
        // 如果之前遍历的 到 adj.to 的距离 比从 node 经过 adj 到达 adj.to 的距离长 说明该更新了
        if (dst[adj.to] > dst[node] + adj.weight) {
            res[adj.to] = adj;
            dst[adj.to] = dst[node] + adj.weight;
        }
    }
}
```

- BellmanFord 算法

能过处理负数的图，但是不能处理负数环（因为负数环能够达到任意短的负数）。

其核心思想是遍历 V 次 图，这样保障每个点都被遍历 V 次，找到最短的路径。

但是可以优化的点是，只有在上次被更改了长度的点才能加入队列中。

```java // 这个算法只能处理没有负权重环的有向图
// 因为带有负权重环的图可以得到任意短的权重 是无效的
public static Edge[] bellmanFordMinPath(DirectedWeightGraph g, int start) {
    // 所以 如果遍历所有的点 同时 relax 所有的 边 就可以得到一个结果
    // 其效率为 o(v + e)
    // 但是 可以考虑一个问题 就是只有在上轮循环中更新过的点 才有可能使 距离更短，所以 用一个 queue 来保存这样的点

    // 转上轮对 dst 数组有贡献的点
    Queue<Integer> queue = new LinkedList<>();
    // 结果
    Edge[] edgeTo = new Edge[g.getCapacity()];
    // 距离初始化
    int[] dst = new int[g.getCapacity()];
    Arrays.fill(dst, Integer.MAX_VALUE);
    dst[start] = 0;
    // 因为 queue 里面不能有重复节点 所以用这个来判断
    boolean[] inQueue = new boolean[g.getCapacity()];
    // 执行 relax 的次数
    int cost = 0;

    // 最开始的节点
    queue.add(start);
    // TODO 这个时候还要检查负权重环
    // 这个地方只需要检查是否是成环即可
    while (!queue.isEmpty()) {
        relax(g, queue.remove(), dst, edgeTo, queue, inQueue);
    }
    return edgeTo;
}

// 在放松的时候同时更新 queue
private static void relax(DirectedWeightGraph g, int node, int[] dst, Edge[] edgeTo, Queue<Integer> queue, boolean[] inQueue) {
    for (Edge adj : g.adj(node)) {
        int to = adj.to;

        // 更新
        if (dst[to] > dst[node] + adj.weight) {
            edgeTo[to] = adj;
            dst[to] = dst[node] + adj.weight;
            if (!inQueue[node]) {
                inQueue[node] = true;
                // 这有这轮已经更新过的 到 to 的更短距离 其他才可能更短
                queue.add(to);
            }
        }
        // TODO 检查负权重环
    }
}

```
