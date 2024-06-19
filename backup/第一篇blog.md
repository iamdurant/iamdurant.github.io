# 借助github的博客，用于记录生活或者学习
`int a = 1;`
```java
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
```