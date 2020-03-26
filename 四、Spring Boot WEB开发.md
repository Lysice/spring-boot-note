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
如果想接口和页面写在同一个控制器内
    @Controller 
    
    @ResponseBody
    @RequestMapping("/success")
```

thymeleaf语法

要使用thymeleaf 首先需要加入thymeleaf的命名空间。

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

`th:text`用于处理`p`标签体的文本内容。该模板文件直接在任何浏览器中正确显示，浏览器会自动忽略它们不能理解的属性`th:text`。但这不是一个真正有效的 HTML5 文档，因为 HTML5 规范是不允许使用`th:*`这些非标准属性的。我们可以切换到 Thymeleaf 的`data-th-*`语法，以此来替换`th:*`语法：

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Index Page</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <p data-th-text="${message}">Welcome to BeiJing!</p>
</body>
</html>
```

HTML5 规范是允许`data-*`这样自定义的属性的。`th:*`和`data-th-*`这两个符号是完全等效且可以互换的。但为了简单直观和代码的紧凑性，本文采用`th:*`的表示形式。

2-3 语法规则

![](/home/zhao/notes/images/截图_2020-01-17_14-42-40.png)

th:insert 片段包含 jsp:include

th:each 遍历

th:if 判断

th:unless 判断

th:object 变量声明

th:attr 属性修改

th:attrprepend 任意属性修改 支持prepend append

th:attrappend

th:value th:text th:id 等 修改指定i属性。

th:text 标签体内容修改 utext 不转义

th:fragment 声明片段

th:remove 移除

##### 2.1 简单表达式

| 语法 |              名称              |      描述      |          作用          |
| :--: | :----------------------------: | :------------: | :--------------------: |
| ${…} |      Variable Expressions      |   变量表达式   |   取出上下文变量的值   |
| *{…} | Selection Variable Expressions | 选择变量表达式 | 取出选择的对象的属性值 |
| #{…} |      Message Expressions       |   消息表达式   | 使文字消息国际化，I18N |
| @{…} |      Link URL Expressions      |   链接表达式   | 用于表示各种超链接地址 |
| ~{…} |      Fragment Expressions      |   片段表达式   | 引用一段公共的代码片段 |

###### 2.1.1 ${}  

可以取值

```html
<div id="div01" class="div01" th:id="div02" th:class="div-2" th:text="${hello}"></div>
<p th:text="${world}"></p>
```

可以使用内置对象

#ctx : the context object.
#vars: the context variables.
#locale : the context locale.
#request : (only in Web Contexts) the HttpServletRequest object.
#response : (only in Web Contexts) the HttpServletResponse object.
#session : (only in Web Contexts) the HttpSession object.
#servletContext : (only in Web Contexts) the ServletContext object

可以使用工具对象

#execInfo : information about the template being processed.
#messages : methods for obtaining externalized messages inside variables expressions, in the same way as they
would be obtained using #{…} syntax.
#uris : methods for escaping parts of URLs/URIs
Page 20 of 106
#conversions : methods for executing the configured conversion service (if any).
#dates : methods for java.util.Date objects: formatting, component extraction, etc.
#calendars : analogous to #dates , but for java.util.Calendar objects.
#numbers : methods for formatting numeric objects.
#strings : methods for String objects: contains, startsWith, prepending/appending, etc.
#objects : methods for objects in general.
#bools : methods for boolean evaluation.
#arrays : methods for arrays.
#lists : methods for lists.
#sets : methods for sets.
#maps : methods for maps.
#aggregates : methods for creating aggregates on arrays or collections.
#ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration)

###### 2.1.2 *{}

```Java
Dog dog = new Dog("dog1", 2);
System.out.println(dog.toString());

Map<String, String> hashMap = new HashMap<String, String>();
hashMap.put("0", "0");
hashMap.put("1", "1");
hashMap.put("2", "2");

List<String> list = new ArrayList<String>();
list.add("list1");
list.add("list2");

Person person = new Person("last", 2, false,
        "2019-01-01",  hashMap, list, dog);

map.put("person", person);
System.out.println(person.toString());
return "success";
```

模板写法

```html
<div id="div01" class="div01" th:id="div02" th:class="div-2" th:text="${hello}"></div>
<p th:text="${world}"></p>
<p>person</p>
<div th:text="${person.lastName}"></div>
<div th:text="${person.age}"></div>
<div th:text="${person.boss}"></div>
<div th:text="${person.birth}"></div>
<div th:text="${person.lastName}"></div>
<div th:object="${person.dog}">
    <a th:text="*{name}"></a>
    <a th:text="*{age}"></a>
</div>
```

若对象未被选中 则 *{}与 ${}效果同。

```html
<div th:text="*{person.dog.name}"></div>
<div th:text="${person.dog.name}"></div>
```

配合th:object获取对象的值。

###### 2.1.3 #{} 国际化内容

###### 2.1.4 @{} 定义url

@{http://localhost:80/v1/22(id)}

不写域名则直接默认本项目下。

###### 2.1.5 th:fragment

实验内用。

#####  2.2 字面值

文本 th:text="work string"

数字 th:text="2013 + 2"

布尔 

```html
<div th:if="${user.isAdmin()} == false"> 
```

Null

```java
<div th:if="${variable.something} == null"> .
```

##### 2.3 文本操作

拼接文本

```html
<span th:text="'The name of the user is ' + ${user.name}">
```

另外一种写法

```html
<span th:text="'Welcome to our application, ' + ${user.name} + '!'">
```

##### 2.4 算数运算

+, - , * , / , %

##### 2.5 条件运算

```html
<div th:if="${prodStat.count} &gt; 1">
<span th:text="'Execution mode is ' + ( (${execMode} == 'dev')? 'Development' : 'Production')">
```

If-then (if) ? (then)

If-then-else (if) ? (then) : (else)

Default (value) ? (default value)

##### 2.6 布尔运算

 and or ! not

##### 2.7 比较运算

```html
> < >= <= gt lt ge le
      eq ne
```

##### 2.8 无操作

_

###### 2.9 行内写法

可以直接在 p标签内写 [[${user.name}]]

#### 5.SpringMVC自动配置原理





#### 实验

1.展示首页 templates

空方法 返回页面

自定义mvcConfigureAdapter接管SpringMVC

```Java
@Configuration
public class MyMvcConfig extends WebMvcConfigurationSupport {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        super.addViewControllers(registry);
        registry.addViewController("/a").setViewName("/index");
        registry.addViewController("/index.html").setViewName("/index");
    }
}
```

Spring Boot自动配置了classpath:/static/下面的资源为静态资源，后来网上找了很多的方法都试过了，解决不了。

  于是我重新写了一个项目，把这个旧项目的配置一个一个的移动过去，最后发现是我配置的拦截器的问题。因为我配置拦截器继承的类是：WebMvcConfigurationSupport这个类，它会让spring boot的自动配置失效。

怎么解决呢？

  第一种可以继承WebMvcConfigurerAdapter，当然如果是1.8+WebMvcConfigurerAdapter这个类以及过时了，可以直接实现WebMvcConfigurer接口，然后重写addInterceptors来添加拦截器：

```
@Configuration
public` `class` `InterceptorConfig ``implements` `WebMvcConfigurer {
```

 

```
  ``@Override
  ``public` `void` `addInterceptors(InterceptorRegistry registry) {
    ``registry.addInterceptor(``new` `UserInterceptor()).addPathPatterns(``"/user/**"``);
    ``WebMvcConfigurer.``super``.addInterceptors(registry);
  ``}
  }
```

 或者还是继承WebMvcConfigurationSupport，然后重写addResourceHandlers方法：

```
@Configuration
public` `class` `InterceptorConfig ``extends` `WebMvcConfigurationSupport {
```

 

```Java
  ``@Override
  ``protected` `void` `addInterceptors(InterceptorRegistry registry) {
    ``registry.addInterceptor(``new` `UserInterceptor()).addPathPatterns(``"/user/**"``);
    ``super``.addInterceptors(registry);
  ``}
  
  ``@Override
  ``protected` `void` `addResourceHandlers(ResourceHandlerRegistry registry) {
    ``registry.addResourceHandler(``"/**"``).addResourceLocations(``"classpath:/static/"``);
    ``super``.addResourceHandlers(registry);
  ``}
  
}
```

``

```
@Configuration
public class MyMvcConfig extends WebMvcConfigurationSupport {
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("classpath:/static/");
        super.addResourceHandlers(registry);
    }
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/a").setViewName("login");
        registry.addViewController("/index.html").setViewName("login");
    }
}
```

2.

