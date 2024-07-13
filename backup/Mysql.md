**行记录格式**
- antelope
  - redundant
  - compact
- barracuda
    - dynamic
    - compressed

**compact**
可变长字段长度(1或2个byte)    NULL值(bitmap)    头部(5个byte)

**三个隐藏字段**
- ROWID(6 byte)，在没有指定primary key时才存在
- 事务ID(6 byte)
- 回滚指针(7 byte)

**buffer pool**
由控制块与缓存页组成

Hash
```text
由表空间及页号组成key，控制块引用为value，O(1)的时间判断要查询的数据是否位于buffer pool
```

各链表及其作用
- free链
```text
双向链表，节点存储缓存页等信息
```
- Flush链
```text
双向链表，节点存储缓存页等信息，当数据被更新时，被标记为dirty page，相应的控制块加入flush链，由后台线程异步刷盘
```
- LRU链
```text
双向链表，节点存储缓存页等信息，分为young数据区、old 数据区，当buffer pool满时，淘汰数据
```