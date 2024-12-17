[原理](https://oi-wiki.org/string/z-func/#__tabbed_2_1)

```java
public int[] zFunction(String s) {
        int n = s.length();
        char[] c = s.toCharArray();
        int[] z = new int[n];
        int l = 0;
        int r = 0;
        for(int i = 1;i < n;i++) {
            if(i > r) {
                // 暴力匹配
                while(i + z[i] < n && c[i + z[i]] == c[z[i]]) z[i]++;
            } else {
                if(z[i - l] < r - i + 1) z[i] = z[i - l];
                else {
                    // 跳过前r - i + 1个后继续匹配
                    while(r + z[i] + 1 < n && c[r + z[i] + 1] == c[r - i + 1 + z[i]]) z[i]++;
                    z[i] += r - i + 1;
                }
            }
            if(i + z[i] > r && z[i] != 0) {
                // 更新l、r
                l = i;
                r = i + z[i] - 1;
            }
        }

        return z;
    }
```