#### 四、Web开发

##### 1.使用Spring Boot

- 创建SpringBoot应用 选中需要模块
- Spring Boot已经默认配置好了很多东西 只需要在配置文件中配置少量就可以运行。

##### 2.自动配置原理

这个场景 SpringBoot帮我们配置了什么 如何配置 如何扩展

@xxxAutoconfiguration 帮我们在容器中自动配置

@EnableConfigurationProperties可以配置

xxxProperties配置类来封装配置文件的内容

##### 3.Spring Boot对静态资源的映射规则

```Java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
    } else {
        Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
        CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
        if (!registry.hasMappingForPattern("/webjars/**")) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }

        String staticPathPattern = this.mvcProperties.getStaticPathPattern();
        if (!registry.hasMappingForPattern(staticPathPattern)) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
        }

    }
}
```

###### 1)webjars目录下 -> classpath:/META-INF/resources/webjars/ 找资源

以jar包的形式导入a静态资源

https://webjars.com

```pom
<!--       引入jquery 访问时u只需要写webjars下面相关资源-->
      <dependency>
         <groupId>org.webjars</groupId>
         <artifactId>jquery</artifactId>
         <version>3.3.1</version>
      </dependency>
```

mvn package

![截图_2020-01-16_15-41-50](/home/zhao/notes/images/截图_2020-01-16_15-41-50.png)

localhost:8080/webjars/jquery/3.3.1/jquery.js

###### 2).自己的资源放在哪里?

```Java
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties {
     可以设置和资源有关的参数 缓存时间
         
```



```
this.staticPathPattern = "/**";
```

```Java
String staticPathPattern = this.mvcProperties.getStaticPathPattern();
if (!registry.hasMappingForPattern(staticPathPattern)) {
    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
}
```

```java
public String[] getStaticLocations() {
    return this.staticLocations;
}
```

```java
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
private String[] staticLocations;
private boolean addMappings;
private final ResourceProperties.Chain chain;
private final ResourceProperties.Cache cache;

public ResourceProperties() {
    this.staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
    this.addMappings = true;
    this.chain = new ResourceProperties.Chain();
    this.cache = new ResourceProperties.Cache();
}
```

###### 3）访问资源下任意资源

"classpath:/META-INF/resources/"

"classpath:/resources/",

"classpath:/static/", 

"classpath:/public/"



其中 资源类路径

classpath=resources

一开始没有找到  重新启动服务可以了。

###### 4)静态资源文件夹下 的所有index.html页面 被/**映射

public->index.html 访问localhost:8080

###### 5)图标



##### 4.模板引擎

JSP/Velocity Freemarker Thymeleaf

Spring Boot推荐 Thymeleaf

1 加入thymeleaf依赖 mvn package

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

2 Thymeleaf使用

2-1 页面渲染

```Java
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
    只要我们把html页面放在classpath:templates下 就能自动渲染。
```

新建html文件 resources/templates/success.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>success</title>
</head>
<body>
    success11111
</body>
</html>
```



```
@Controller
public class HelloWorldController {

        @RequestMapping("/success")
        public String success()
        {
            return "success";
        }
}
```

2-2 

```Java
@RequestMapping("/success")
public String success(Map<String, Object> map)
{
    map.put("hello", "你好");
    return "success";
}
```

thymeleaf语法

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>success</title>
</head>
<body>
    <h1>成功</h1>
    <div th:text="${hello}"></div>
</body>
</html>
```

2-3 语法规则

