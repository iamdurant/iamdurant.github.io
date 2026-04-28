#### tarjan算法

[tarjan学习](https://oi-wiki.org/graph/scc/)

> Robert E. Tarjan（计算机科学家） 发明了很多算法和数据结构，有：
>
> > 已学习：✓
> >
> > 未学习：×
>
> - 并查集（✓）
> - Splay（×）
> - Toptree（×）
> - Tarjan（dfs求强连通分量、dfs求割点、dfs求割边）(✓)

##### 1.在有向图中求强连通分量

**强连通分量（Strongly Connected Components）**：极大的强连通子图

tarjan算法基于dfs，并且维护一个栈，m每次把尚未处理的节点加入栈中

并且，每个节点还维护了以下两个变量：

- dfn：dfs时遍历节点的次序
- low(low link value)：子树中能够回溯到的最早的在stack中的dfn

对于一个连通分量，第一个dfs到的节点的dfn = low，这时将stack里 >= dfn的节点pop

##### 2.在无相图中求割点

基于1进行变形

- 对于根节点，若子树大于等于2，则为割点

- 对于某个节点u的子节点，若存在一个节点v的low值大于等于dfn(u)，则u为割点

  > why？：因为若去掉点u，则点v无法回到祖先节点（因为点v能够回到祖先节点是依赖点u），形成新的连通分量

##### 3.在无向图中求割边

基于1进行变形

- 若某个节点v的dfn == low(v) ，则代表其无法回到祖先节点，也无法通过另一条边回到父亲节点，则其父节点u到v的这条边即为割边

割边例题：[1192.查找集群内的关键连接](https://leetcode.cn/problems/critical-connections-in-a-network/description/?envType=study-plan-v2&envId=graph-theory)

实现：

```java
class Solution {
    // tarjan 割边
    int p = 0;
    List<List<Integer>> ans;
    public List<List<Integer>> criticalConnections(int n, List<List<Integer>> connections) {
        Map<Integer, List<Integer>> map = new HashMap<>();
        for(List<Integer> e : connections) {
            map.computeIfAbsent(e.get(0), k -> new ArrayList<>()).add(e.get(1));
            map.computeIfAbsent(e.get(1), k -> new ArrayList<>()).add(e.get(0));
        }

        int[] dfn = new int[n];
        int[] low = new int[n];
        boolean[] v = new boolean[n];
        Set<Integer> set = new HashSet<>();
        ArrayDeque<Integer> stack = new ArrayDeque<>();
        ans = new ArrayList<>();
        dfs(map, stack, set, v, dfn, low, 0, -1);

        return ans;
    }

    private void dfs(Map<Integer, List<Integer>> map, ArrayDeque<Integer> stack, Set<Integer> set, boolean[] v, int[] dfn, int[] low, int cur, int pare) {
        dfn[cur] = p;
        low[cur] = dfn[cur];
        set.add(cur);
        stack.push(cur);
        v[cur] = true;
        List<Integer> ns = map.get(cur);
        if(ns != null) {
            for(int next : ns) {
                if(!v[next]) {
                    ++p;
                    dfs(map, stack, set, v, dfn, low, next, cur);
                    low[cur] = Math.min(low[cur], low[next]);
                    if(low[next] > dfn[cur]) {
                        // 割边
                        ans.add(List.of(cur, next));
                    }
                } else {
                    if(set.contains(next) && next != pare) {
                        low[cur] = Math.min(low[cur], low[next]);
                    }
                }
            }
        }

        if(dfn[cur] == low[cur]) {
            while(!stack.isEmpty() && stack.peek() != cur) {
                int pop = stack.pop();
                set.remove(pop);
            }
            if(!stack.isEmpty()) {
                int pop = stack.pop();
                set.remove(pop);
            }
        }
    }
}
```

