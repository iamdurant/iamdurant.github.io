#### 前缀函数（kmp）

##### [原理](https://oi-wiki.org/string/kmp/#__tabbed_3_3)

##### 1.求前缀函数，普通写法
   - 时间复杂度：O(n^3)

```java
/**
     *
     * 时间复杂度: O(n^3)
     * @param s 字符串
     * @return  pai
     */
public int[] pf(String s) {
    char[] c = s.toCharArray();
    int n = c.length;
    int[] pi = new int[n];
    for(int i = 1;i < n;i++) {
        for(int j = 1;j <= i;j++) {
            int l = 0;
            int r = j;
            while(r < n && c[l] == c[r]) {
                l++;
                r++;
            }
            if(r > i) {
                pi[i] = i - j + 1;
                break;
            }
        }
    }

    return pi;
}
```

##### 2.求前缀函数，第一个优化

   - 时间复杂度：O(n^2)
   - 原理：

   > 观察相邻的`pai[i]`、`pai[i + 1]`，发现`pai[i + 1] <= pai[i] + 1`，那么对于每个`pai[i]`都能跳过`i - pai[i + 1]`个前缀，大大提高效率

```java
/**
     *
     * 时间复杂度: O(n^2)
     * @param s 字符串
     * @return  pi
     */
public int[] pf(String s) {
    char[] c = s.toCharArray();
    int n = c.length;
    int[] pi = new int[n];
    for(int i = 1;i < n;i++) {
        if(c[i] == c[pi[i - 1]]) {
            pi[i] = pi[i - 1] + 1;
            continue;
        }
        for(int j = i - pi[i - 1] + 1;j <= i;j++) {
            int l = 0;
            int r = j;
            while(r < n && c[l] == c[r]) {
                l++;
                r++;
            }
            if(r > i) {
                pi[i] = i - j + 1;
                break;
            }
        }
    }

    return pi;
}
```

##### 3.求前缀函数，第二个优化

- 时间复杂度：O(n)

- 原理：

  > 求`pi[i]`，`len = pi[i - 1]`，`len`为`c[i - 1]`的相等真前后缀长度，若`s[len] != s[i]`，则减少，直到`0`。那么如何快速求得小于`len`的最长相等真前后缀长度？如图所示：
  >
  > 
  > ![kmp](https://github.com/user-attachments/assets/d114d604-c78f-4e12-9b7c-a44852d7e2ad)
  >
  > 因为：`s[0...j-1] = s[i-j+1...i] = s[n[i]-j...n[i]-1]`
  >
  > 所以：下一个最长相等真前后缀 = `pi[len - 1]`

```java
/**
     *
     * 时间复杂度: O(n)
     * @param s 字符串
     * @return  pi
     */
public int[] pf(String s) {
    char[] c = s.toCharArray();
    int n = c.length;
    int[] pi = new int[n];
    for(int i = 1;i < n;i++) {
        int secondLen = pi[i - 1];
        while(secondLen > 0 && c[i] != c[secondLen]) {
            secondLen = pi[secondLen - 1];
        }
        if(c[i] == c[secondLen]) secondLen++;
        pi[i] = secondLen;
    }

    return pi;
}
```

