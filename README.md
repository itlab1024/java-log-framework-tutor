# Java日志框架讲解

Java的日志框架有很多，常用的有`log4j1.x`，`log4j2.x`，`logback`，`jul`，`jcl`等等，这些框架如何使用？他们如何与`slf4j-api`搭配使用？不搭配`slf4j-api`能否单独使用？使用过程中需要注意哪些事项？

实际开发中我们也会遇到很多问题，本篇文章就是为了讲解下Java的日志体系。

我会尽可能的讲解清楚，如有问题，欢迎指正！！！



# 日志体系讲解
下图是SlF4j日志的体系图。

![](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131057384.png)

理解了上图，其实就理解了Slf4j日志框架的体系。

查看上图时，特别注意颜色的区分，不同颜色有不同的说明，记下来我具体讲解下。

![](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131107122.png)：代表的应用，这没有什么可说的。

![](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131108796.png)：代表的是Slf4j Api（slf4j-api.jar）,这个jar是日志框架的门面，里面是对日志的抽象（接口），它将暴漏给应用来使用。

![](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131110892.png)：这些日志的实现（provider），是直接实现slf4j-api.jar相关接口实现的。

![](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131112670.png)：代表适配层，是因为有些日志的实现（provider）并没有直接使用`slf4j-api`而是有自己的接口，此时就需要使用适配将其和`slf4j-api`打通。

![](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131113484.png)：没有原生实现`sfl4j-api`的日志框架，这些框架需要使用适配层与``sfl4j-api`接口适配。

纵向看，每个application都对应一种实现方式。接下来一一来说明下具体实现方式。

[SLF4J官网文档地址](https://www.slf4j.org/)



# 说明

本文使用的sfl4j-api版本是2.0.6版本，JDK是17

# 未绑定实现
未绑定实现肯定是不能使用的，也就是说只有`slf4j-api`的接口定义，没有具体地实现。

示例项目：[slf4j-unbound](slf4j-unbound)

## 依赖说明
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itlab1024.log</groupId>
    <artifactId>slf4j-unbound</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>2.0.6</version>
        </dependency>
    </dependencies>
</project>
```
只引入了`slf4j-api`!
## 测试类

```java
package com.itlab1024.log;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class com.itlab1024.log.Main {
    public static final Logger logger = LoggerFactory.getLogger(com.itlab1024.log.Main.class);
    public static void main(String[] args) {
        logger.debug("debug");
        logger.info("info");
        logger.warn("warn");
        logger.error("error");
    }
}
```
## 运行结果
![](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131123344.png)

日志并没有正常输出，从红色文字提示可知道，没有提供Slf4j的实现框架（provider）。



# 绑定logback实现

通常使用logback只需要引入`logback-classic`即可，它内部的pom依赖了`sfl4j-api`和`logback-core`，会自动下载下来响应的依赖。

示例项目：[slf4j-logback-classic](slf4j-logback-classic)

[官网地址]([Logback Home (qos.ch)](https://logback.qos.ch/))

## 依赖说明

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itlab1024.log</groupId>
    <artifactId>slf4j-logback-classic</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.4.5</version>
        </dependency>
    </dependencies>
</project>
```

上面pom中我只依赖了`logback-classic`。通过idea工具可以看到该项目的所有依赖情况：

![image-20230113114548311](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131145409.png)

可以看到，自动下载了`logback-core:1.4.5`和`slf4j-api:2.0.4`，当然也可以手动引入这两个依赖（不同版本）。

## 测试类

```java
package com.itlab1024.log;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class com.itlab1024.log.Main {
    public static final Logger logger = LoggerFactory.getLogger(com.itlab1024.log.Main.class);
    public static void main(String[] args) {
        logger.debug("debug");
        logger.info("info");
        logger.warn("warn");
        logger.error("error");
    }
}
```

## 运行结果

![image-20230113114827808](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131148890.png)

可以看到能够正常打印日志信息。

## 配置文件

logback支持使用xml的方式配置日志的相关信息，需要在classpath下（maven项目的resources下），创建`logback.xml`文件。

```xml
<configuration  xmlns="http://ch.qos.logback/xml/ns/logback"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://ch.qos.logback/xml/ns/logback
                https://raw.githubusercontent.com/enricopulatzo/logback-XSD/master/src/main/xsd/logback.xsd">

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- encoders are assigned the type
             ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
        <encoder>
            <pattern>ITLab1024---%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} -%kvp- %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

特别注意config的配置，此配置会让你在xml中编辑的时候，自动提示！！！，可以去看我的另一个博客[IDEA下Logback.xml自动提示功能配置 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/398558799)

再次运行程序查看运行结果：

![](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131209551.png)

可以看到，配置文件已经生效。

# 绑定reload4j（log4j 1.x升级版）

`log4j`的`1.x`版本是一个通用版本，但是由于2022年的log4j漏洞原因，`slf4j-log4j`模块在`build`时，会自动重定向至`slf4j-reload4j`模块。也就是说如果想用`log4j`的话，就直接使用哦个`reload4j`吧。

示例项目：[slf4j-reload4j](slf4j-reload4j)

[官网地址](https://reload4j.qos.ch/)

## 依赖说明

通常只需要引入`slf4j-reload4j`即可，会自动下载其他依赖项`slf4j-api`和`reload4j`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itlab1024.log</groupId>
    <artifactId>slf4j-reload4j</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-reload4j</artifactId>
            <version>2.0.6</version>
        </dependency>
    </dependencies>
</project>
```

看下下载下来的依赖项

![image-20230113132913298](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131329400.png)

## 测试类

```java
package com.itlab1024.log;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class com.itlab1024.log.Main {
    public static final Logger logger = LoggerFactory.getLogger(com.itlab1024.log.Main.class);
    public static void main(String[] args) {
        logger.debug("debug");
        logger.info("info");
        logger.warn("warn");
        logger.error("error");
    }
}
```

## 测试结果

初次运行，很遗憾，出问题了。

![image-20230113133247685](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131332758.png)

为什么呢？这是因为log4j 环境的配置通常在 应用程序初始化。首选方法是通过读取 配置文件。所以要在`classpath`下添加`log4j.properties(xml)`文件。

增加`log4j.properties`文件内容如下：

```properties
log4j.rootLogger=debug, stdout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout

# Pattern to output the caller's file name and line number.
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] (%F:%L) - %m%n
```

再次运行：

![image-20230113133950011](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131339083.png)

日志正常输出。

# 绑定juc（java.util.logging）

这是JDK自带的日志框架，官方地址是：[Java Logging Overview (oracle.com)](https://docs.oracle.com/en/java/javase/17/core/java-logging-overview.html#GUID-B83B652C-17EA-48D9-93D2-563AE1FF8EDA)

## 依赖说明

只需要引入依赖`slf4j-jdk14`即可，会将其依赖的`slf4j-api`自动下载。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itlab1024.log</groupId>
    <artifactId>sfl4j-jul</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-jdk14</artifactId>
            <version>2.0.6</version>
        </dependency>
    </dependencies>
</project>
```

依赖截图如下：

![image-20230113135202950](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131352035.png)

## 测试类

```java
package com.itlab1024.log;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class com.itlab1024.log.Main {
    public static final Logger logger = LoggerFactory.getLogger(com.itlab1024.log.Main.class);
    public static void main(String[] args) {
        logger.debug("debug");
        logger.info("info");
        logger.warn("warn");
        logger.error("error");
    }
}
```

## 运行结果

![image-20230113135356194](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131353266.png)

## 配置文件

juc的配置文件名称是`logging.properties`。

官方提供了默认的文件配置。在JDK目录下的`conf`文件夹下能够找到。

```properties
############################################################
#  	Default Logging Configuration File
#
# You can use a different file by specifying a filename
# with the java.util.logging.config.file system property.
# For example, java -Djava.util.logging.config.file=myfile
############################################################

############################################################
#  	Global properties
############################################################

# "handlers" specifies a comma-separated list of log Handler
# classes.  These handlers will be installed during VM startup.
# Note that these classes must be on the system classpath.
# By default we only configure a ConsoleHandler, which will only
# show messages at the INFO and above levels.
handlers= java.util.logging.ConsoleHandler

# To also add the FileHandler, use the following line instead.
#handlers= java.util.logging.FileHandler, java.util.logging.ConsoleHandler

# Default global logging level.
# This specifies which kinds of events are logged across
# all loggers.  For any given facility this global level
# can be overridden by a facility-specific level
# Note that the ConsoleHandler also has a separate level
# setting to limit messages printed to the console.
.level= INFO

############################################################
# Handler specific properties.
# Describes specific configuration info for Handlers.
############################################################

# default file output is in user's home directory.
java.util.logging.FileHandler.pattern = %h/java%u.log
java.util.logging.FileHandler.limit = 50000
java.util.logging.FileHandler.count = 1
# Default number of locks FileHandler can obtain synchronously.
# This specifies maximum number of attempts to obtain lock file by FileHandler
# implemented by incrementing the unique field %u as per FileHandler API documentation.
java.util.logging.FileHandler.maxLocks = 100
java.util.logging.FileHandler.formatter = java.util.logging.XMLFormatter

# Limit the messages that are printed on the console to INFO and above.
java.util.logging.ConsoleHandler.level = INFO
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter

# Example to customize the SimpleFormatter output format
# to print one-line log message like this:
#     <level>: <log message> [<date/time>]
#
# java.util.logging.SimpleFormatter.format=%4$s: %5$s [%1$tc]%n

############################################################
# Facility-specific properties.
# Provides extra control for each logger.
############################################################

# For example, set the com.xyz.foo logger to only log SEVERE
# messages:
# com.xyz.foo.level = SEVERE
```

# 绑定slf4j-simple

## 依赖说明

只要引入`sfl4j-simple`即可，会自动下载`slf4j-api`依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itlab1024</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>2.0.6</version>
        </dependency>
    </dependencies>

</project>
```

依赖情况

![image-20230113140203234](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131402325.png)

## 测试类

```java
package com.itlab1024.log;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class com.itlab1024.log.Main {
    public static final Logger logger = LoggerFactory.getLogger(com.itlab1024.log.Main.class);
    public static void main(String[] args) {
        logger.debug("debug");
        logger.info("info");
        logger.warn("warn");
        logger.error("error");
    }
}
```

## 运行结果

![image-20230113140715105](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131407171.png)

未打印出来debug的日志，是因为默认的日志级别是`info`。

## 配置文件

配置文件名称是`simplelogger.properties`，比如配置如下配置文件。

```properties
org.slf4j.simpleLogger.defaultLogLevel=DEBUG
org.slf4j.simpleLogger.showDateTime=true
org.slf4j.simpleLogger.dateTimeFormat=yyyy-MM-dd HH:mm:ss
org.slf4j.simpleLogger.showLogName=true
```

默认日志级别我修改为了DEBUG

运行结果：

![image-20230113141549920](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131415002.png)

可以看到debug级别的日志被打印了出来。

# 绑定log4j2

`Apache Log4j 2` 是对 `Log4j `的升级，对其前身 `Log4j 1.x `和 提供了 `Logback` 中可用的许多改进，同时修复了 `Logback `架构中的一些固有问题。

`log4j2`并未在上图中展示，但是并不是说起不能与`slf4j-api`融合，相反，官方已经给了解决方案。

[Log4j – Apache Log4j 2](https://logging.apache.org/log4j/2.x/)

## 依赖说明

log4j也不是直接实现的`slf4j-api`，而是实现的`log4j-api`，如果搭配`slf4j-api`，则需要引入桥接jar`log4j-slf4j-impl`。当依赖下载的时候会自动下载

`slf4j-api`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itlab1024</groupId>
    <artifactId>slf4j-log4j2</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.19.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.19.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>2.19.0</version>
        </dependency>
    </dependencies>
</project>
```

上面引入了三个依赖，这三个是必须要引入的。看下最终下载的所有依赖。

![image-20230113142930317](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131429413.png)

## 测试类

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Main {
    public static final Logger logger = LoggerFactory.getLogger(Main.class);
    public static void main(String[] args) {
        logger.debug("debug");
        logger.info("info");
        logger.warn("warn");
        logger.error("error");
    }
}
```

## 运行结果

![image-20230113143053539](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131430620.png)



## 配置文件

log4j2支持4中文件格式，`XML`, `JSON`, `YAML`, `Properties`。名称默认是log4j2

比如我增加配置文件`log4j2.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="debug">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

再次执行程序查看结果：

![image-20230113143624118](https://itlab1024-1256529903.cos.ap-beijing.myqcloud.com/202301131436192.png)

因为我修改了日志级别，所以日志都打印出来了。



---

> 本篇文章主要是介绍几种日志框架的基本使用，并未深入讲解比如配置文件如何配置，如何修改默认配置等等内容。



---



> 博主信息

* github: https://github.com/itlab1024