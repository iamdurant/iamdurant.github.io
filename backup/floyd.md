#### [原理](https://blog.csdn.net/qq_35644234/article/details/60875818)

#### 实现

- 数据

  ```text
  12
  1 2 12
  1 6 16
  1 7 14
  2 3 10
  2 6 7
  3 4 3
  3 5 5
  3 6 6
  4 5 4
  5 6 2
  5 7 8
  6 7 9
  ```

```java
public static void floyd() {
    Scanner sc = new Scanner(System.in);
    System.out.print("请输入图的边数量: ");
    int edgesAmount = sc.nextInt();
    List<int[]> edges = new ArrayList<>();
    sc.nextLine();

    System.out.println("请输入所有边（格式为: 点x 点y 权重）: ");
    Set<Integer> points = new HashSet<>();
    for(int i = 0;i < edgesAmount;i++) {
        String line = sc.nextLine();
        String[] edge = line.split(" ");
        edges.add(new int[]{Integer.parseInt(edge[0]), Integer.parseInt(edge[1]), Integer.parseInt(edge[2])});

        points.add(Integer.parseInt(edge[0]));
        points.add(Integer.parseInt(edge[1]));
    }
    int pointSize = points.size() + 1;
    int[][] d = new int[pointSize][pointSize];
    int[][] p = new int[pointSize][pointSize];
    // 初始化d数组
    for(int i = 0;i < pointSize;i++) {
        Arrays.fill(d[i], Integer.MAX_VALUE);
    }
    for(int[] e : edges) {
        d[e[0]][e[1]] = e[2];
        d[e[1]][e[0]] = e[2];
    }

    // 初始化path数组，假设全都可达
    for(int i = 0;i < pointSize;i++) {
        for(int j = 0;j < pointSize;j++) {
            p[i][j] = j;
        }
    }

    // O(n^3)
    for(int k = 0;k < pointSize;k++) {
        for(int i = 0;i < pointSize;i++) {
            for(int j = 0;j < pointSize;j++) {
                if(d[i][k] == Integer.MAX_VALUE || d[k][j] == Integer.MAX_VALUE) continue;
                int dis = d[i][k] + d[k][j];
                if(dis < d[i][j]) {
                    d[i][j] = dis;
                    p[i][j] = k;
                }
            }
        }
    }

    // 输出
    for(int i = 0;i < pointSize;i++) {
        for(int j = 0;j < pointSize;j++) {
            if(i == j || d[i][j] == Integer.MAX_VALUE) continue;
            System.out.println(i + "->" + j + "的最短路径值: " + d[i][j]);
            List<Integer> paths = new ArrayList<>();
            path(p, i, j, -1, paths);
            System.out.println("路径为: " + buildPath(paths));
        }
    }
}

private static void path(int[][] p, int s, int t, int lr, List<Integer> paths) {
    int m = p[s][t];

    if(lr != 1) paths.add(s);
    // 处理左边
    if(!(m == s || m == t)) path(p, s, m, 1, paths);
    // 中点
    if(!(m == s || m == t)) paths.add(m);
    // 处理右边
    if(!(m == s || m == t)) path(p, m, t, 1, paths);
    if(lr != 1) paths.add(t);
}

private static String buildPath(List<Integer> paths) {
    StringBuilder sb = new StringBuilder();
    for(int i = 0;i < paths.size();i++) {
        sb.append(paths.get(i));
        if(i != paths.size() - 1) sb.append("->");
    }

    return sb.toString();
}
```

