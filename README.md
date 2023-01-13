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

