##### 一主多从

主库my.cnf配置:

```mysql
[mysqld]
bind-address = 0.0.0.0
port = 3306

server-id = 1
log-bin = mysql-bin

binlog-do-db = sharding_jdbc_demo
binlog-do-db = sharding_jdbc_demo2
binlog-do-db = sharding_jdbc_userdb
```



从库my.cnf配置

```sql
[mysqld]
# 基础配置
basedir=/home/jasper/桌面/apps/mysql/mysql-8.0.42-linux-glibc2.28-x86_64         # 修改为你的MySQL解压目录（即bin的上级）
datadir=/home/jasper/桌面/apps/mysql/mysql-8.0.42-linux-glibc2.28-x86_64/mysql_data
socket=/home/jasper/桌面/apps/mysql/mysql-8.0.42-linux-glibc2.28-x86_64/mysql.sock
pid-file=/home/jasper/桌面/apps/mysql/mysql-8.0.42-linux-glibc2.28-x86_64/mysql.pid
bind-address = 0.0.0.0
port=3307                           # 避免与主库冲突

# 日志配置
log-error=/home/jasper/桌面/apps/mysql/mysql-8.0.42-linux-glibc2.28-x86_64/mysql.err
log-bin=/home/jasper/桌面/apps/mysql/mysql-8.0.42-linux-glibc2.28-x86_64/mysql-bin.log

# server id 必须唯一
server-id=2

# 复制配置
relay-log=/home/jasper/桌面/apps/mysql/mysql-8.0.42-linux-glibc2.28-x86_64/mysql-relay-bin
read_only=1

# 只复制主库中的某个数据库（可选）
replicate-do-db=sharding_jdbc_demo
replicate-do-db=sharding_jdbc_demo2
replicate-do-db=sharding_jdbc_userdb
```



初始化从库

```shell
./mysqld --defaults-file=./my.cnf --initialize-insecure
```

> **error:**
>
> error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
>
> **解决:**
>
> 安装并创建软链接
>
> ```shell
> sudo apt install libaio-dev
> ldconfig -p | grep libaio # 找到此lib
> sudo ln -s /lib/x86_64-linux-gnu/libaio.so.1t64 /lib/x86_64-linux-gnu/libaio.so.1
> ```
>
> 

启动从库

```shell
./mysqld_safe --defaults-file=./my.cnf
```

配置master信息并启动replica

```sql
# 在主库中获取LOG_File以及LOG_POS
# 低版本: show master status;
# 高版本: show binary log status;

CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='127.0.0.1',
  SOURCE_PORT=3306,
  SOURCE_USER='root',
  SOURCE_PASSWORD='pubgM666',
  SOURCE_LOG_FILE='mysql-bin.000006',
  SOURCE_LOG_POS=158;
  
start replica;
```

