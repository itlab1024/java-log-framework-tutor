# java日志框架讲解
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

# 绑定reload4j

`log4j`的`1.2`版本是一个通用版本，但是由于2022年的log4j漏洞原因，`slf4j-log4j`模块在`build`时，会自动重定向至`slf4j-reload4j`模块。也就是说如果想用`log4j`的话，就直接使用哦个`reload4j`吧。

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

