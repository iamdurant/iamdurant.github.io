### B树的degree(阶数)

> - 几阶B树即代表最多能有多少个孩子，除叶子节点以外，节点的孩子不少于`阶数/2`
> - 那么除叶子节点以外，节点key的数量范围为`阶数/2 - 1 到 阶数 - 1`

### B树的节点实现，静态内部类，简化设计，不考虑Object，只有key

```java
static class Node {
        int[] keys;             // key for compare
        Node[] children;        // children pointer
        int keyNum;             // key size
        boolean leaf = true;    // leaf or not
        int t;                  // minDegree

        public Node(int minDegree){
            this.t = minDegree;
            keys = new int[2 * t - 1];
            children = new Node[2 * t];
        }

        public Node find(int key){
            int i;
            for (i = 0;i < keyNum;i++){
                if (keys[i] == key)
                    return this;
                if (keys[i] > key)
                    break;
            }

            if (leaf)
                return null;

            return children[i].find(key);
        }

        public void insertKey(int index, int key){
            System.arraycopy(keys, index, keys, index + 1, keyNum - index);
            keys[index] = key;
            keyNum++;
        }

        public void insertNode(int index, Node node){
            System.arraycopy(children, index, children, index + 1, keyNum - index);
            children[index] = node;
        }

        @Override
        public String toString() {
            StringBuilder sb = new StringBuilder();
            sb.append('[');
            for (int i = 0;i < keyNum;i++){
                sb.append(keys[i]);
                if (i != keyNum - 1)
                    sb.append(',');
            }
            sb.append(']');

            return sb.toString();
        }
    }
```

### key的插入

1. 将根节点作为此节点，若找到相同的key，则更新
2. 若此节点不存在此key，则判断是否为叶子节点，若是，则插入，否则此节点更新为chiildren[i]
3. 若插入完成后`满key`了，则**分裂此节点**

#### 什么是满key

> 若degree(度)为4，即最多4个孩子
则t = 2，那么key满的个数为3，若插入新key，达到3个即为满key状态，要进行分裂

#### 节点分裂

1. 创建新的节点，将大>=t索引的key与children移动到此新节点，**特殊情况：**`若分裂节点为根节点，则除了构建一个right节点以外，还需构建一个新根节点`
2. 将索引为t的key移动到父亲节点`若父亲的索引i为此节点`，**特殊情况**：`若分裂节点不是叶子节点 则还需将分裂节点的children复制到right节点`
3. 将新创建的节点作为父亲的 i+1索引节点
4. 最后，记得更新分裂节点的keyNumber

> 从宏观上看节点的分裂过程，就是将索引为t-1的key移动到父节点 将索引为t之后(**包括t**)的key移动给新构建的right节点

#### B树完整的插入代码

```java
public void put(int key){
        doPut(null, 0, root, key);
    }

    private void doPut(Node parent, int index, Node node, int key){
        int i = 0;
        while (i < node.keyNum){
            if (key == node.keys[i])
                return;
            if (key < node.keys[i])
                break;
            i++;
        }

        if (node.leaf){
            // 插入
            node.insertKey(i, key);
        } else {
            // 递归插入
            doPut(node, i, node.children[i], key);
        }

        if (node.keyNum == MAX_KEY_NUM){
            // 节点分裂
            split(node, parent, index);
        }
    }

    private void split(Node node, Node parent, int index){
        if (parent == null){
            Node newRoot = new Node(t);
            newRoot.insertNode(0, node);
            newRoot.leaf = false;
            parent = newRoot;
            root = newRoot;
        }

        Node right = new Node(t);
        right.leaf = node.leaf;
        System.arraycopy(node.keys, t, right.keys, 0, t - 1);
        if (!node.leaf){
            System.arraycopy(node.children, t, right.children, 0, t);
        }

        right.keyNum = t - 1;

        parent.insertKey(index, node.keys[t - 1]);
        parent.insertNode(index + 1, right);
        node.keyNum = t - 1;
    }
```
#### B树插入代码测试
```java
    /**
     * degree: 4
     * t: 2
     *           5            2|5
     *         /   \   ==>   / | \
     *      1|2|3   6       1  3  6
     *               分裂过程
     */
    public static void main(String[] args) {
        BTree bTree = new BTree(2);
        bTree.put(1);
        bTree.put(2);
        bTree.put(5);
        bTree.put(6);
        bTree.put(3);

        System.out.println();
    }
```

---

### key的删除

相对来说，删除key比插入key复杂的多，开始之间，先在Node节点类添加9个辅助方法（直接调用），让删除key的逻辑更加清晰：

>- `删除指定index的key并返回`

```java
public int removeKey(int index){
    int re = keys[index];
    System.arraycopy(keys, index + 1, keys, index, keyNum - index - 1);
    return re;
}
```

>- `移除最左边的key并返回`

```java
public int removeKey(int index){
    int re = keys[index];
    System.arraycopy(keys, index + 1, keys, index, keyNum - index - 1);
    return re;
}
```

>- `移除最右边的key并返回`

```java
public int removeRightestKey(){
    int re = keys[keyNum - 1];
    return re;
}
```

>- `移除指定index的child并返回`

```java
public Node removeChild(int index){
    Node re = children[index];
    System.arraycopy(children, index + 1, children, index, keyNum - index - 1);
    return re;
}
```

>- `移除最左边child并返回`

```java
public Node removeLeftestChild(){
     Node re = children[0];
     System.arraycopy(children, 1, children, 0, keyNum - 1);
     return re;
}
```

>- `移除最右边的child并返回`

```java
public Node removeRightestChild(){
     Node re = children[keyNum - 1];
     return re;
}
```

>- `获取指定index的左兄弟`

```java
public Node leftBrother(int index){
    return index > 0 ? children[index - 1] : null;
}
```

>- `获取指定index的右兄弟`

```java
public Node rightBrother(int index){
    return index == keyNum ? null : children[index + 1];
}
```

>- `将节点的所有key与child移动到指定节点`

```java
public void removeNode(Node target){
    int start = target.keyNum;
    if (keyNum >= 0) System.arraycopy(keys, 0, target.keys, start, keyNum);
    if (!leaf){
        if (keyNum >= 0) System.arraycopy(children, 0, target.children, start, keyNum);
    }
}
```

#### 删除key的各种case

- 若节点为叶子节点
  - 存在指定key，直接返回
  - 不存在指定key，删除操作
- 若节点为非叶子节点
  - 不存在指定key，递归删除
  - 存在指定key，找到后继key，与后继key交换，递归删除后继key（相当于限定了删除操作只能在叶子节点进行）
- 删除上述操作完成后，归向上判断是否需要合并（key数 < t - 1）

#### 合并操作

- 根节点
  - 哈哈
  - 哈哈
- 非根节点
  - 左兄弟或者右兄弟key富余
    - 若存在左兄弟且左兄弟key富余，则右旋
    - 若存在右兄弟且右兄弟key富余，则左旋
  - 无法通过旋转恢复平衡，则合并（都采用左合并）
    - 若不平衡节点存在左兄弟，向左兄弟合并（若`parent.childre[i + 1] == 此不平衡node`，则将`parent.children[i]`以及整个不平衡节点合并到左兄弟）
    - 若不平衡节点不存在左兄弟，则将右兄弟合并到不平衡节点（若`parent.childre[i + 1] == 此不平衡node的右兄弟`，则将`parent.children[i]`以及整个不平衡节点的右兄弟合并到不平衡节点）



