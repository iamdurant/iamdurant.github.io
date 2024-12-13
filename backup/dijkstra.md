1. 普通dijkstra

```java
import java.util.*;

public class Dijkstra {
    static class Node implements Comparable<Node> {
        int vertex;
        int distance;

        public Node(int vertex, int distance) {
            this.vertex = vertex;
            this.distance = distance;
        }

        @Override
        public int compareTo(Node other) {
            return Integer.compare(this.distance, other.distance);
        }
    }

    public static int[] dijkstra(int[][] graph, int start) {
        int n = graph.length;
        int[] distances = new int[n];
        boolean[] visited = new boolean[n];

        // 初始化距离数组
        Arrays.fill(distances, Integer.MAX_VALUE);
        distances[start] = 0;

        // 使用优先队列优化
        PriorityQueue<Node> pq = new PriorityQueue<>();
        pq.offer(new Node(start, 0));

        while (!pq.isEmpty()) {
            Node current = pq.poll();
            int u = current.vertex;

            // 如果已经访问过，跳过
            if (visited[u]) continue;
            visited[u] = true;

            // 更新相邻节点的距离
            for (int v = 0; v < n; v++) {
                if (graph[u][v] != 0) {  // 存在边
                    int newDist = distances[u] + graph[u][v];
                    if (newDist < distances[v]) {
                        distances[v] = newDist;
                        pq.offer(new Node(v, distances[v]));
                    }
                }
            }
        }

        return distances;
    }

    // 测试代码
    public static void main(String[] args) {
        int[][] graph = {
            {0, 4, 2, 0, 0},
            {4, 0, 1, 5, 0},
            {2, 1, 0, 8, 10},
            {0, 5, 8, 0, 2},
            {0, 0, 10, 2, 0}
        };

        int start = 0;  // 起点为A（0号顶点）
        int[] distances = dijkstra(graph, start);

        // 打印结果
        System.out.println("从顶点 " + start + " 到各个顶点的最短距离：");
        for (int i = 0; i < distances.length; i++) {
            System.out.println("到顶点 " + i + " 的距离: " + distances[i]);
        }
    }
}
```

2. 带路径记录的dijkstra

```java
import java.util.*;

public class DijkstraWithPath {
    static class Node implements Comparable<Node> {
        int vertex;
        int distance;

        public Node(int vertex, int distance) {
            this.vertex = vertex;
            this.distance = distance;
        }

        @Override
        public int compareTo(Node other) {
            return Integer.compare(this.distance, other.distance);
        }
    }

    static class Result {
        int[] distances;
        int[] previousVertices;

        public Result(int[] distances, int[] previousVertices) {
            this.distances = distances;
            this.previousVertices = previousVertices;
        }

        // 获取从起点到目标点的路径
        public List<Integer> getPath(int target) {
            List<Integer> path = new ArrayList<>();
            int current = target;

            // 如果没有路径到达目标点
            if (distances[target] == Integer.MAX_VALUE) {
                return path;
            }

            while (current != -1) {
                path.add(0, current);
                current = previousVertices[current];
            }

            return path;
        }
    }

    public static Result dijkstra(int[][] graph, int start) {
        int n = graph.length;
        int[] distances = new int[n];
        int[] previousVertices = new int[n];
        boolean[] visited = new boolean[n];

        // 初始化
        Arrays.fill(distances, Integer.MAX_VALUE);
        Arrays.fill(previousVertices, -1);
        distances[start] = 0;

        PriorityQueue<Node> pq = new PriorityQueue<>();
        pq.offer(new Node(start, 0));

        while (!pq.isEmpty()) {
            Node current = pq.poll();
            int u = current.vertex;

            if (visited[u]) continue;
            visited[u] = true;

            for (int v = 0; v < n; v++) {
                if (graph[u][v] != 0) {
                    int newDist = distances[u] + graph[u][v];
                    if (newDist < distances[v]) {
                        distances[v] = newDist;
                        previousVertices[v] = u;
                        pq.offer(new Node(v, distances[v]));
                    }
                }
            }
        }

        return new Result(distances, previousVertices);
    }

    public static void main(String[] args) {
        // 示例图（邻接矩阵表示）
        int[][] graph = {
            {0, 4, 2, 0, 0},
            {4, 0, 1, 5, 0},
            {2, 1, 0, 8, 10},
            {0, 5, 8, 0, 2},
            {0, 0, 10, 2, 0}
        };

        int start = 0;
        Result result = dijkstra(graph, start);

        // 打印所有最短距离
        System.out.println("从顶点 " + start + " 到各个顶点的最短距离：");
        for (int i = 0; i < result.distances.length; i++) {
            System.out.println("到顶点 " + i + " 的距离: " + result.distances[i]);

            // 打印路径
            List<Integer> path = result.getPath(i);
            System.out.println("路径: " + path);
        }
    }
}
```

3. 解释
   1. Node类
      - 实现了Comparator接口，用于优先队列的排序
      - 存储顶点编号和到起点的距离
   2. 数据结构
      - `distances[]`：存储从起点到各个顶点的最短距离
      - `visited[]`：记录顶点是否被访问过
      - `previousVertices[]`：记录最短路径中每个顶点的前驱顶点
      - `PriorityQueue`：优先队列，用于获取当前最短距离的顶点
   3. 核心步骤
      - 初始化距离数组，起点距离为0，其他为无穷大
      - 使用优先队列选择当前最短距离的顶点
      - 更新选中顶点的邻接点的距离
      - 记录路径信息
4. 性能优化建议
   1. 对于稀疏图，使用邻接表代替邻接矩阵
   2. 使用索引优先队列可以进一步优化性能
   3. 如果只需要找到到特定目标的最短路径，可以在找到后提前终止
5. 使用示例

```java
// 创建图（邻接矩阵）
int[][] graph = new int[5][5];
// 添加边
graph[0][1] = 4;  // A到B的距离为4
graph[1][0] = 4;  // B到A的距离为4
// ... 添加其他边

// 计算最短路径
Result result = dijkstra(graph, 0);  // 从顶点0开始

// 获取到某个顶点的最短路径
List<Integer> pathTo3 = result.getPath(3);
System.out.println("到顶点3的路径: " + pathTo3);
System.out.println("距离: " + result.distances[3]);
```



