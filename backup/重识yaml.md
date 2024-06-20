### spring将yaml格式作为配置文件格式，必须得好好熟悉一下

### 基本语法

- 大小写敏感
- 使用缩进表示层级关系
- 缩进值允许使用空格，不允许使用tab
- 缩进的空格数不重要，只要保证相同层级的左对齐即可
- 使用`#`表示注释

### yaml对象

以键值对的方式表示 中间用`: `分隔key & value，`:`号后面带**一个**空格

```yaml
key: 
  value1
  value2
```

### yaml数组

yaml数组用`- `号开头，`:`号后面带**一个**空格

```yaml
arr: 
 - a
 - b
 - c
```

也可以表示多维数组，需控制层级

```yaml
array: 
  -
    - a
    - b
    - c
  -
    - x
    - y
    - z
```

对象数组，就像java中的`Person p1 = new Person("薛大炮", 18);Person p2 = new Person("蔡徐坤", 26);Person[] persons = new Person[]{p1, p2};`

```yaml
persons
  -
    name: 薛大炮
    age: 18
  -
    name: 蔡徐坤
    age: 26
```

### 复合结构

对象和数组可以构成复杂的结构关系

```yaml
languages: 
  - Java
  - C++
  - Go
WebSites: 
  baidu: https://www.baidu.com
  google: https:www.google.com
  bing: https:www.bing.com
```

对应到json为

```json
{
  "languages": ["Java", "C++", "Go"],
  "WebSites": {
    "baidu": "https://www.baidu.com",
    "google": "https:www.google.com",
    "bing": "https:www.bing.com"
  }
}
```

### 纯量

纯量是最基本的，不可再分的值
- 字符串
- 整数
- 浮点数
- 布尔值
- null
- 时间（date）
- 日期（datetime）

以一个例子来看各纯量的使用

```yaml
string: 
  - 哈哈            # 字符串可直接写
  - 'good'         # 或者单引号引表示
  - "西海岸"        # 或者双引号表示
  - you jump       # 字符串可写多行，每行被转化为空格即为` you jump i jump！`
    i jump！

int: 
  - 123
  - 0100_1101     # 二进制表示
 
float: 
  - 66.6
  - 45.1231402+e5      # 可以使用科学计数法表示

boolean: 
  - TRUE
  - FALSE
  - True
  - False
  - true
  - false        # 其实 true false 全小写最常用

null: 
  - a
  - ~            # 使用~表示null

date: 2024-06-20       # 使用ISO-8601标准：yyyy-MM-dd

datetime: 2024-06-20T16:34:20+09:00     # 使用ISO-8601标准：yyyy-MM-ddTHH:mm:ss+时区
```

### 引用

`&`表示锚点，`*`表示引用锚点，`<<: `表示合并到当前数据流

```yaml
database: &database
  host: 127.0.0.1
  port: 3306
  user: root
  password: 123456

mysql: 
  <<: *database             # 若未生效 则mardown解析器不兼容
```





