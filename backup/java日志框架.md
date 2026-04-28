#### java现有的日志框架有哪些

- JUL(java util logging)
- logback
- log4j
- log4j2
- JCL(jakarta commons logging)
- slf4j(simple logging facade for java)



#### 日志门面技术的优点

1. 面向接口开发，不再依赖具体的实现类
2. 通过导入不同的日志实现类，可以灵活的切换日志框架
3. 统一api，方便开发者学习和使用
4. 统一配置便于项目日志的管理



#### JUL(java util logging)

JUL，java原生日志框架，不需要引入第三方依赖包，使用简单方便，一般在小型应用中使用，主流项目中很少使用

##### JUL架构

- Application：java应用程序
- loggers：记录器，application通过获取logger对象，调用其api来发布日志消息
- appenders：也称为handlers，每个logger都会关联一组handlers，logger会将日志交给handlers处理handlers是一个抽象，其具体的实现决定了日志记录的位置可以是控制台、文件、网络上的其他日志服务或操作系统日志等
- layouts：也称为formatters，负责对日志进行转换或者格式化
- level：每条日志都有一个关联的日志级别
- filters：过滤器，根据需要自定义哪些消息会被记录，哪些消息会被放过

##### 日志级别(由高到低)

1. severe
2. warning
3. info(默认的日志级别)
4. config
5. fine
6. finer
7. finest

##### 为什么要对日志进行分级

无论是将日志输出到`console`还是文件，都会带来消耗，降低程序的运行效率，但是，日志又是调试或者了解程序运行的不可或缺的手段，通过对日志进行分级处理，在一个配置文件中进行统一的管理，这样代码中日志部分就可以不用删除，又可以控制输出哪些日志

```java
public static void main(String[] args) {
    Logger logger = Logger.getLogger("multi");

    logger.setUseParentHandlers(false);

    ConsoleHandler consoleHandler = new ConsoleHandler();
    consoleHandler.setLevel(Level.ALL);
    SimpleFormatter simpleFormatter = new SimpleFormatter();
    consoleHandler.setFormatter(simpleFormatter);
    logger.addHandler(consoleHandler);

    logger.setLevel(Level.ALL);

    logger.severe(">>>> severe");
    logger.warning(">>>> warning");
    logger.info(">>>> info");
    logger.config(">>>> config");
    logger.fine(">>>> fine");
    logger.finer(">>>> finer");
    logger.finest(">>>> finest");

    logger.log(Level.WARNING, ">>>> waring!!!!");

    String name = "wwb";
    int age = 23;
    logger.log(Level.INFO, ">>>> name: {0}, age: {1}", new Object[]{name, age});

    //logger.log(Level.SEVERE, "NPE: ", new NullPointerException());
}
```

##### logger的继承关系

创建`logger`时，若未指定父`logger`，则默认继承`RootLogger`。子代`logger`会继承`fulogger`的配置，例如日志级别、`handlers`、日志格式等等。

```java
Logger l1 = Logger.getLogger("com.niuma");

Logger l2 = Logger.getLogger("com.niuma.fuck");

System.out.println(l1.getParent() +  " name: " + l1.getParent().getName());
System.out.println(l2.getParent() + " name: " + l2.getParent().getName());

输出：
java.util.logging.LogManager$RootLogger@7cd84586 name: 
java.util.logging.Logger@30dae81 name: com.niuma
```

##### 日志配置

我们可以通过2种方式配置`JUL`的日志行为(设置日志级别、日志输出地、日志格式等等)

1. 代码中编码
2. 配置文件

```java
// 编码：配置两个handler
Logger logger = Logger.getLogger("kevin");
logger.setUseParentHandlers(false);

ConsoleHandler consoleHandler = new ConsoleHandler();
FileHandler fileHandler = new FileHandler("./logging.txt");

SimpleFormatter formatter = new SimpleFormatter();
fileHandler.setFormatter(formatter);
consoleHandler.setFormatter(formatter);

logger.addHandler(consoleHandler);
logger.addHandler(fileHandler);

logger.info("有人访问");
logger.warning("有人偷窃!!!!!");
```

```java
// 读取配置文件
FileInputStream properties = new FileInputStream("A:\\logging.properties");
LogManager logManager = LogManager.getLogManager();
logManager.readConfiguration(properties);

Logger logger = Logger.getLogger("test");
logger.info("fuck you bitch");
logger.info("FBI, open the door!!!!");
```

```properties
# 全局配置
# logger配置handler 用,分隔
handlers = java.util.logging.FileHandler,java.util.logging.ConsoleHandler

# 全局level级别
.level = ALL

# FileHandler配置
java.util.logging.FileHandler.pattern = ./log_file.log
# 数量限制
java.util.logging.FileHandler.count = 4
# 大小限制 64MB
java.util.logging.FileHandler.limit = 64 * 1024 * 1024
# 日志级别
java.util.logging.FileHandler.level = INFO
# 是否追加
java.util.logging.FileHandler.append = true
# 编码
java.util.logging.FileHandler.encoding = UTF-8
# formatter
java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter

# ConsoleHandler配置
java.util.logging.ConsoleHandler.level = INFO
# 编码
java.util.logging.ConsoleHandler.encoding = UTF-8
# formatter
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter

# Logger配置
# 设置com.niuma.ak logger的日志级别
com.niuma.ak.level = SEVERE
# 关联handler
com.niuma.ak.handlers = java.util.logging.FileHandler,java.util.logging.ConsoleHandler
# 不继承RootHandler
com.niuma.ak.useParentHandlers = false
```

##### 日志过滤器(filter)

`Filter`，用来对输出的日志记录进行过滤。我们可以根据多个维度进行过滤，例如`message`中包含某段信息的日志、只输出某个方法的日志，只输出某个级别的日志等等。

`Logger`对象将日志封装为一个`LogRecord`对象，然后将该对象交给handler处理。每个`Logger`和`handler`都可以关联一个`filter`。`LogRecord`中包含了日志的文本消息、日志生成的时间戳、日志来源于哪个类，日志来源于哪个方法、日志来源于哪个线程等等。

只需实现Filter接口即可自定义如何过滤日志。

```java
public static void main(String[] args) throws IOException {
    Logger logger = Logger.getLogger("kevin");
    logger.setFilter(new CustomFilter());

    logger.info("fuck");
    logger.info("bitch");
}

static class CustomFilter implements Filter {
    @Override
    public boolean isLoggable(LogRecord record) {
        String message = record.getMessage();
        return !message.contains("fuck");
    }
}
```



