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

分析依赖关系

1.maven查看依赖

2.pom文件中右键找到diagrams->show dependences

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter</artifactId>
</dependency>
```

Spring Boot内部依赖spring-boot-starter-logging来做日志

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-logging</artifactId>
  <version>2.2.2.RELEASE</version>
  <scope>compile</scope>
</dependency>
```

spring-boot-starter-logging内部依赖以下jar包 目的在于将其他日志转为slf4j的相关依赖。

```xml
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.2.3</version>
  <scope>compile</scope>
</dependency>
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-to-slf4j</artifactId>
  <version>2.12.1</version>
  <scope>compile</scope>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>jul-to-slf4j</artifactId>
  <version>1.7.29</version>
  <scope>compile</scope>
</dependency>
```

最后在三个jar包内依赖slf4j-api

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
</dependency>
```

总结:

SpringBoot底层也是使用slf4j+logback的方式进行日志记录

SpringBoot把其他日志都替换成了slf4j

如果我们要引入其他框架 也需要先把框架的默认日志依赖排除掉。

SpringBoot内部排除

```xml
<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>httpclient</artifactId>
  <version>4.5.10</version>
  <scope>compile</scope>
  <exclusions>
    <exclusion>
      <artifactId>commons-logging</artifactId>
      <groupId>commons-logging</groupId>
    </exclusion>
  </exclusions>
  <optional>true</optional>
</dependency>
```

SpringBoot能自动适配所有日志 而且底层使用slf4j+logback的方式记录日志 引入其他框架的时候 只需要把这个框架依赖的日志框架排除掉。



#### 4.Spring Booth 日志默认配置



```Java
Logger logger = LoggerFactory.getLogger(getClass());
@Test
public void testLog()
{
   // 日志的级别 由低到高 可以调整输出日志级别
   this.logger.trace("trace日志");
   this.logger.debug("debug日志");
   this.logger.info("info日志");
   this.logger.warn("警告日志");
   this.logger.error("错误日志");
}
```

SpringBoot默认输出的是info级别的日志。

##### 配置log的级别

```XML
logging.level.com.example.demo=trace
```

##### 配置log文件

```xml
logging.file=/home/lysice/java/demo/demo.log
```

##### 配置日志文件目录在目录下生成日志spring.log

```
logging.path=/home/lysice/java/demo
```

```xml
# 控制台输出日志格式
logging.pattern.console=
# 文件输出日志格式
logging.pattern.file=
```

##### 日志输出格式 

- %d           日期时间
- %thread  线程名
- %-5level  级别从左显示5个字符宽度
- %logger[50] 表示logger名字最长50个字符 否则按照句点分割。
- %msg 日志消息
- %n       换行符

5.指定日志文件和日志profile功能

logback.xml可以直接被日志框架识别

如果xml名称为 logback-spring 日志框架就不直接加载 配置 由Spring Boot加载 可以使用profile。在profile内部可以设置不同环境下的日志打印。

<springProfile name="staging">    ** </springProfile> <springProfile name="dev | staging">    ** </springProfile> 

<springProfile name="!production">    ** </springProfile>

6.切换日志框架



