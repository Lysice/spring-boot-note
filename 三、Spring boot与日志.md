### 三、Spring boot与日志

#### 1.日志框架

开发调试

1. System.out.println
2. 框架来记录系统的运行时信息 Logger.jar
3. 高大上功能 异步模式/自动归档/...
4. 将以前的框架卸下来 换新框架需要修改api
5. JDBC-数据库驱动模式 面向接口编程   

1. 接口层 日志门面
2. 项目中导入具体的日志实现 我们之前的日志框架都是实现的抽象层的东西。

JUL

JCL

Jboss-logging

logback

log4j

log4j2

slf4j



| 日志门面                                                     | 日志实现                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ~~JCL(2014年更新)~~        SLF4j        ~~Jboss-logging(生来就不是普通程序员用)~~ | Log4j JUL(Java Util Log) Log4j2(借了Log4j的名字 太好了?? 框架没适配) Logback |

左边选择一个门面(SLF4j) 右边选择一个实现Logback

Spring Boot底层是Spring框架 Spring框架默认JCL

Spring Boot选用SLF4j和Logback



#### 2.SLF4j使用

1.如何在系统中使用SLF4j

以后开发 不应该直接调用实现 而应该调用门面方法。

系统导入slf4j jar和logback的实现jar

```Java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

<img src="/home/zhao/notes/images/concrete-bindings.png" style="zoom:75%;" />



每个日志的实现框架都有自己的配置文件 使用slf4j之后 配置文件还是做成日志实现框架配置文件。



##### 遗留问题

a sjf4j+logback: Spring(commons-logging) Hibernate(Jboss-logging) MyBatis xxx

统一日志记录 即使是别的框架和我i一起使用slf4j进行输出。

slf4j有专门的解决方案。

<img src="/home/zhao/notes/images/legacy.png" style="zoom:75%;" />

如何让系统中的所有日志都统一到slf4j

1.将系统中的其他日志框架先排除出去

2.用中间包来替换原有的日志框架

3.导入slf4j其他的实现



#### 3.Spring Boot日志的关系

4.Spring Booth 日志默认配置

5.指定日志文件和日志profile功能

6.切换日志框架