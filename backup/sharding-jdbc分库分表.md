##### 分库分表

- 垂直分表

  > 将频繁访问的属性与访问率较低的属性,拆分到两张表

- 垂直分库

  > 将不相关的业务实体,拆分到两个数据库

- 水平分库

  > 将表数据划分到不同的数据库

- 水平分表

  > 将表水平拆分成多个表



#####  sharding-jdbc-sql执行过程

1. 解析
2. 改写
3. 路由
4. 执行
5. 归并



##### 水平分表demo

**水平分表**

`sql`准备,创建两张表: `t_order_1`,`t_order_2`

```sql
create table t_order_2(
	`id` bigint primary key comment '订单id',
    `product_id` bigint not null comment '商品id',
    `amount` int not null comment '购买数量',
    `buyer_id` bigint not null comment '购买者id',
    `create_date` datetime not null comment '下单时间',
    `status` int not null default 0 comment '订单状态 0: 未支付, 1: 已支付, 2: 超时, 3: 取消, 4: 其他',
    `pay_time` datetime null comment '支付时间'
) engine=innodb default charset=utf8mb4 comment='订单表2';
```

`maven`依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.7.15</version>
        <exclusions>
            <exclusion>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.34</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.30</version>
    </dependency>

    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.5</version>
        <exclusions>
            <exclusion>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <dependency>
        <groupId>com.github.xiaoymin</groupId>
        <artifactId>knife4j-openapi2-spring-boot-starter</artifactId>
        <version>4.2.0</version>
    </dependency>

    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
        <version>4.1.1</version>
    </dependency>


    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-logging</artifactId>
        <version>2.7.15</version>
    </dependency>
</dependencies>

<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

`yaml`配置

```yaml
server:
  port: 8888

mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true

spring:
  # sharding-jdbc 配置
  shardingsphere:
    datasource:
      names: m1
      m1:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/sharding_jdbc_demo?serverTimezone=UTC
        username: root
        password: pubgM666
    sharding:
      tables:
        t_order:
          actual-data-nodes: m1.t_order_$->{1..2}
          table-strategy:
            inline:
              sharding-column: id
              algorithm-expression: t_order_$->{id % 2 + 1}
          key-generator:
            column: id
            type: SNOWFLAKE
            props:
              worker:
                id: 100
              max:
                tolerate:
                  time:
                    difference:
                      milliseconds: 20
```

**细节**

1. 雪花算法生成的字段,在insert时,需要自定义`sql`,将次字段从`sql`插入字段中去除:

   > 比如:
   >
   > 定义id字段为雪花算法自动生成,则`insert sql`为:
   >
   > ```java
   > @Insert("insert into t_order(product_id, amount, buyer_id, create_date, status, pay_time) values(#{productId}, #{amount}, #{buyerId}, #{createDate}, #{status}, #{payTime})")
   >  int insert(TOrder tOrder);
   > ```

2. 依赖不能导错,`sharding-jdbc`依赖:

   ```xml
   <dependency>
       <groupId>org.apache.shardingsphere</groupId>
       <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
       <version>4.1.1</version>
   </dependency>
   ```


###### 碰到的bug

1. `sharding-jdbc-spring-boot-starter`,无法解析`LocalDateTime`以及`LocalDate`

   > 修复: 
   >
   > - 依赖升级
   >
   > ```xml
   > <dependency>
   >     <groupId>org.apache.shardingsphere</groupId>
   >     <artifactId>shardingsphere-jdbc-core-spring-boot-starter</artifactId>
   >     <version>5.2.1</version>
   >     <exclusions>
   >         <exclusion>
   >             <groupId>org.yaml</groupId>
   >             <artifactId>snakeyaml</artifactId>
   >         </exclusion>
   >     </exclusions>
   > </dependency>
   > ```
   >
   > - `5.x.x`版本yml配置不同,更改为:
   >
   > ```yaml
   > spring:
   >   shardingsphere:
   >     datasource:
   >       names: m1
   >       m1:
   >         type: com.zaxxer.hikari.HikariDataSource
   >         driver-class-name: com.mysql.cj.jdbc.Driver
   >         jdbc-url: jdbc:mysql://localhost:3306/sharding_jdbc_demo?serverTimezone=UTC
   >         username: root
   >         password: pubgM666
   > 
   >     rules:
   >       sharding:
   >         tables:
   >           t_order:
   >             actual-data-nodes: m1.t_order_$->{1..2}
   >             table-strategy:
   >               standard:
   >                 sharding-column: id
   >                 sharding-algorithm-name: t_order_inline
   >             key-generate-strategy:
   >               column: id
   >               key-generator-name: snowflake
   > 
   >         sharding-algorithms:
   >           t_order_inline:
   >             type: INLINE
   >             props:
   >               algorithm-expression: t_order_$->{id % 2 + 1}
   > 
   >         key-generators:
   >           snowflake:
   >             type: SNOWFLAKE
   >             props:
   >               worker-id: 100
   >               max-tolerate-time-difference-milliseconds: 20
   > 
   >     props:
   >       sql-show: true   # 开启SQL打印，方便调试
   > ```

2. `mybatis-plus`与`sharding-jdbc`的不兼容

   >  解决: `sql`都使用`mapper`方式定义,不使用`mybatis-plus`


#####  水平分库demo

**`yml`配置:**

```yaml
spring:
  shardingsphere:
    datasource:
      names: m1, m2
      m1:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/sharding_jdbc_demo?serverTimezone=UTC
        username: root
        password: pubgM666
      m2:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/sharding_jdbc_demo2?serverTimezone=UTC
        username: root
        password: pubgM666

    rules:
      sharding:
        tables:
          t_order:
            actual-data-nodes: m$->{1..2}.t_order_$->{1..2}
            database-strategy:
              standard:
                sharding-column: product_id
                sharding-algorithm-name: t_order_database_inline
            table-strategy:
              standard:
                sharding-column: id
                sharding-algorithm-name: t_order_inline
            key-generate-strategy:
              column: id
              key-generator-name: snowflake

        sharding-algorithms:
          t_order_inline:
            type: INLINE
            props:
              algorithm-expression: t_order_$->{id % 2 + 1}
          t_order_database_inline:
            type: INLINE
            props:
              algorithm-expression: m$->{product_id % 2 + 1}

        key-generators:
          snowflake:
            type: SNOWFLAKE
            props:
              worker-id: 100
              max-tolerate-time-difference-milliseconds: 20

    props:
      sql-show: true   # 开启SQL打印，方便调试
```



##### 垂直分库

**数据库准备**

```sql
create database sharding_jdbc_userdb;

create table `user`(
	`id` bigint auto_increment primary key comment 'id,主键',
    `username` varchar(16) not null comment '用户名',
    `encoded_password` varchar(256) not null comment '加密后的密码',
    `salt` char(16) not null comment '随机盐',
    `create_date` datetime not null comment '创建时间'
)engine=innodb default charset=utf8mb4 comment='用户表';
```



##### 广播表

若垂直分库后,部分库都需要对某个表join,并且此表数据读u多写少,可以将其作为为公告表

```sql
create table common(
	`id` bigint primary key auto_increment comment '主键',
    `name` varchar(32) null comment '名字'
)engine=innodb default charset=utf8mb4 comment='公共表';
```

**`yml`配置:**

```yaml
spring:
  shardingsphere:
    rules:
      sharding:
        # 广播表
        broadcast-tables:
          common
```

##### sharding-jdbc读写分离

> `sharding-jdbc`根据`sql`语义,将写操作路由至`master`,将读操作路由至`slave`.提供透明化读写操作,像使用一个数据库一样使用主从数据库集群.
>
> ![image-20250429231159360](http://192.168.1.101:9000/kijjqvbsoduu/imgs/image-20250429231159431.png)

`yaml`配置

```yaml
    rules:
      readwrite-splitting:
        data-sources:
          m0_group:
            static-strategy:
              write-data-source-name: m0
              read-data-source-names:
                - s0
            load-balancer-name: round_robin
        load-balancers:
          round_robin:
            type: ROUND_ROBIN
      sharding:
        # 广播表
        broadcast-tables:
          common
        tables:
          user:
            actual-data-nodes: m0_group.user
```

##### 实践

**描述:**

以商品为例,进行实践.

###### 数据库设计

>  *entity:*
>
>  - 商品
>  - 店铺
>  - 地理区域
>
>  *垂直分库:*
>
>  将商品以及店铺划分到product_db,store_db
>
>  *垂直分表:*
>
>  将商品信息划分为product_info,product_desc
>
>  *广播表:*
>
>  region
>
>  *水平分库:*
>
>  将product_db根据store_id分库
>
>  *水平分表:*
>
>  将product_info根据product_id分表

###### sql准备

product_info

```sql
create database product_db_1;
create database product_db_2;

create table product_info_1(
	`id` bigint primary key comment '商品id',
    `store_id` bigint not null comment '所属店铺id',
    `name` varchar(64) not null comment '商品名称',
    `specification` varchar(8) not null comment '规格',
    `source_place` varchar(16) null comment '产地',
    `price` decimal not null comment '价格',
    `pic_url` varchar(200) not null comment '图片url'
) engine=innodb default charset=utf8mb4 comment='商品信息表';
```

product_desc

```sql
create table product_desc_1(
	`id` bigint auto_increment primary key comment 'id',
    `product_id` bigint not null comment '商品id',
    `store_id` bigint not null comment '店铺id',
    `desc` text not null comment '商品描述'
) engine=innodb default charset=utf8mb4 comment '商品描述表';
```

store

```sql
create databse store_db;

create table store(
	`id` bigint primary key auto_increment comment '店铺id',
    `name` varchar(32) not null comment '店铺名称',
    `score` float null default 0 comment '店铺评分',
    `source_place_id` bigint not null comment '店铺所在地'
) engine=innodb default charset=utf8mb4 comment '店铺信息表';
```

region

```sql
create table region(
	`id` bigint primary key auto_increment comment 'id',
    `code` bigint not null comment '地域编码',
    `name` varchar(32) not null comment '地理区域名称',
    `level` int not null comment '级别,省:1 市:2 县:3 镇:4',
    `parent` bigint null comment '上级地理区域'
) engine=innodb default charset=utf8mb4 comment '地域信息表';
```

