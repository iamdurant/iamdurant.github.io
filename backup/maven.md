maven的本质是一个项目管理工具，将项目开发和管理抽象成一个项目对象模型（POM）
POM（Project Object Model）：项目对象模型

**仓库类型**
- 本地仓库
- 私服
- 中央仓库

**坐标**
- groupId
- artifactId
- version

**command**
- mvn clean
- mvn compile
- mvn test
- mvn package
- mvn install

**依赖传递性**
- 直接依赖：直接通过依赖配置建立的依赖关系
- 间接依赖：被依赖的资源的依赖

**依赖传递冲突问题**
- 路劲优先：当依赖中出现冲突，层级越深，优先级越低
- 声明优先：当冲突层级相同，root先声明优先级高

**依赖范围**
- compile 默认
- provided 源码 测试
- test 测试
- runtime 打包

**模块聚合**
```xml
<packaging>pom</packaging>
<modules>
    <module></module>
    <module></module>
</modules>
```

**自定义属性**
```xml
<properties>
    <spring.web.version></spring.web.version>
    <redis.version></redis.version>
</properties>
```

**pom属性解析到配置文件**
```xml
<resources>
    <resource>
        <directory>配置文件目录</directory>
        <filtering>true</filtering>
    <resource>
</resources>
```

**跳过测试**
```text
mvn package -D skipTests
```