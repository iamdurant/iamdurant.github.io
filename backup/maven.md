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