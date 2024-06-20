### B树的degree(阶数)

> - 几阶B树即代表最多能有多少个孩子，除叶子节点以外，节点的孩子不少于`阶数/2`
> - 那么除叶子节点以外，节点key的数量范围为`阶数/2 - 1 到 阶数 - 1`

### B树的节点实现，静态内部类，`简单设计，并不考虑Object，只有key`

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
3. 若插入完成后满key了，则**分裂此节点**

#### 节点分裂

1. 创建新的节点，将大>=t索引的key与children移动到此新节点，**特殊情况：**`若分裂节点为根节点，则除了构建一个right节点以外，还需构建一个新根节点`
2. 将索引为t的key移动到父亲节点`若父亲的索引i为此节点`，**特殊情况**：`若分裂节点不是叶子节点 则还需将分裂节点的children复制到right节点`
3. 将新创建的节点作为父亲的 i+1索引节点
4. 最后，记得更新分裂节点的keyNumber

> 从宏观上看节点的分裂过程，就是将索引为t-1的key移动到父节点 将索引为t之后(**包括t**)的key移动给新构建的right节点

### B树完整的插入代码

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

            if (node.keyNum == MAX_KEY_NUM){
                // 节点分裂
                split(parent, node, index);
            }
        } else {
            // 递归插入
            doPut(node, i, node.children[i], key);
        }
    }

    private void split(Node node, Node parent, int index){
        if (parent == null){
            // 分裂节点为root 需要构造新的root节点
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
            // 分裂节点为非叶子节点 需要移动children
            System.arraycopy(node.children, t, right.children, 0, t);
        }

        right.keyNum = t - 1;

        parent.insertKey(index, node.keys[t - 1]);
        parent.insertNode(index + 1, right);
        node.keyNum = t - 1;
    }
```