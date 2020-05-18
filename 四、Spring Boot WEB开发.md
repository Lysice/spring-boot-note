

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

###### 4.1 简单表达式

| 语法 |              名称              |      描述      |          作用          |
| :--: | :----------------------------: | :------------: | :--------------------: |
| ${…} |      Variable Expressions      |   变量表达式   |   取出上下文变量的值   |
| *{…} | Selection Variable Expressions | 选择变量表达式 | 取出选择的对象的属性值 |
| #{…} |      Message Expressions       |   消息表达式   | 使文字消息国际化，I18N |
| @{…} |      Link URL Expressions      |   链接表达式   | 用于表示各种超链接地址 |
| ~{…} |      Fragment Expressions      |   片段表达式   | 引用一段公共的代码片段 |

####### 4.1.1 ${}  

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

####### 4.1.2 *{}

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

####### 4.1.3 #{} 国际化内容

####### 4.1.4 @{} 定义url

@{http://localhost:80/v1/22(id)}

不写域名则直接默认本项目下。

###### 4.1.5 th:fragment

实验内用。

###### 4.2 字面值

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

###### 4.3 文本操作

拼接文本

```html
<span th:text="'The name of the user is ' + ${user.name}">
```

另外一种写法

```html
<span th:text="'Welcome to our application, ' + ${user.name} + '!'">
```

###### 4.4 算数运算

+, - , * , / , %

###### 4.5 条件运算

```html
<div th:if="${prodStat.count} &gt; 1">
<span th:text="'Execution mode is ' + ( (${execMode} == 'dev')? 'Development' : 'Production')">
```

If-then (if) ? (then)

If-then-else (if) ? (then) : (else)

Default (value) ? (default value)

###### 4.6 布尔运算

 and or ! not

###### 4.7 比较运算

```html
> < >= <= gt lt ge le
      eq ne
```

###### 4.8 无操作

_

###### 4.9 行内写法

可以直接在 p标签内写 [[${user.name}]]

#### 5.SpringMVC自动配置原理


Spring Boot非常适合web应用程序开发。您可以使用嵌入式Tomcat、Jetty、Undertow或Netty创建一个独立的HTTP服务器。大多数web应用程序使用springboot-starter-web模块快速启动和运行。您还可以选择使用springboot-starter-web-flux模块来构建反应式web应用程序。

https://docs.spring.io/spring-boot/docs/2.2.3.RELEASE/reference/htmlsingle/#boot-features-developing-web-applications



官方文档

Spring MVC auto-configuration

Spring Boot provides auto-configuration for Spring MVC that works well with most applications.

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

  - 自动配置了viewResolver(视图解析器: 根据方法返回值得到视图对象 视图对象决定如何渲染(转发 重定向))

  - idea内查询WebMvcAutoConfiguration 查找ContentNegotiatingViewResolver

  - ```
    public class WebMvcAutoConfiguration {
    。。。。
    		@Bean
            @ConditionalOnBean({ViewResolver.class})
            @ConditionalOnMissingBean(
                name = {"viewResolver"},
                value = {ContentNegotiatingViewResolver.class}
            )
            public ContentNegotiatingViewResolver viewResolver(BeanFactory beanFactory) {
                ContentNegotiatingViewResolver resolver = new ContentNegotiatingViewResolver();
                resolver.setContentNegotiationManager((ContentNegotiationManager)beanFactory.getBean(ContentNegotiationManager.class));
                resolver.setOrder(-2147483648);
                return resolver;
            }
            
            从ContentNegotiatingViewResolver点击进去 实现了ViewResolver接口。
            public class ContentNegotiatingViewResolver extends WebApplicationObjectSupport implements ViewResolver
    
    
    
    实现ViewResolver接口的方法resolveViewName
    @Nullable
        public View resolveViewName(String viewName, Locale locale) throws Exception {
            RequestAttributes attrs = RequestContextHolder.getRequestAttributes();
            Assert.state(attrs instanceof ServletRequestAttributes, "No current ServletRequestAttributes");
            List<MediaType> requestedMediaTypes = this.getMediaTypes(((ServletRequestAttributes)attrs).getRequest());
            if (requestedMediaTypes != null) {
            // 获取视图对象
                List<View> candidateViews = this.getCandidateViews(viewName, locale, requestedMediaTypes);
            // 选择最适合的视图对象
                View bestView = this.getBestView(candidateViews, requestedMediaTypes, attrs);
                if (bestView != null) {
                    return bestView;
                }
            }
    
            String mediaTypeInfo = this.logger.isDebugEnabled() && requestedMediaTypes != null ? " given " + requestedMediaTypes.toString() : "";
            if (this.useNotAcceptableStatusCode) {
                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Using 406 NOT_ACCEPTABLE" + mediaTypeInfo);
                }
    
                return NOT_ACCEPTABLE_VIEW;
            } else {
                this.logger.debug("View remains unresolved" + mediaTypeInfo);
                return null;
            }
        }
        
    ```

  - `ContentNegotiatingViewResolver`组合所有视图解析器

    如何定制?o我们可以自己给容器a添加一个视图解析器自动将其添加。

- 

- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.2.3.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-static-content))).  静态资源文件夹路径 webjars

- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans. 

  自动注册了converter等这些东西。

  converter 转换器 类型转换

  Formatter 格式化器 2017/12/12 ===Date

  

- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.2.3.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-message-converters)).

- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.2.3.RELEASE/reference/htmlsingle/#boot-features-spring-message-codes)).

- Static `index.html` support. 静态首页

- Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.2.3.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-favicon)).

- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.2.3.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-web-binding-initializer)).

If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.

If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.

If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.







... todo

6.WEB开发-扩展与全面接管SpringMVC

7.实验

默认访问首页

资源 

7.1 引入资源

7.2 国际化

7.3 登陆 & 拦截器

7.4 Restful实验要求

7.5 


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

2.自动导入bootstrap webjars

pom.xml文件添加

```
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>4.4.1</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.1.1</version>
</dependency>
```

添加完毕

3.使用thymeleaf引擎表达式

```
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```



##### 国际化

Spring-MVC

1.编写国际化文件

2.使用ResourceBundleMessageSource管理国际化资源文件

3.在页面使用fmt:message取出国际化内容



Springboot内只需要编写国际化文件

1)编写国际化配置文件 抽取页面需要显示的国际化配置

(1)resource 新建目录 i18n

(2)添加login.properties

(3)添加login_zh_CN.properties文件

这时候 idea中可以直接操作添加国际化配置文件。

2)Spring Boot自动配置好了管理国际化的组件。

ResourceBundleAutoConfiguration

basename

application.properties

```
spring.messages.basename=i18n.login
```

3)去页面获取

```
<h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}"></h1>
```

全局设置idea文件编码

File>Other Setting>Settings for New Projects>File encodings设置默认utf8并且转换成ascll码。



4)国际化 设置英文

chrome://settings 设置语言 加入英语(美国) 刷新页面。

根据浏览器语言显示

5)点击按钮切换语言



Spring boot国际化原理 

国际化Locale(区域信息对象):LocaleResolver

默认的就是根据请求头带来的区域信息获取LocaleResolver来解析语言。

如果想在Spring Boot内部设置语言 则需要定义一个

```html
<a class="btn btn-sm" th:href="@{/index.html(l=zh_CN)}">中文</a>
<a class="btn btn-sm" th:href="@{/login.html(l=en_US)}">English</a>
```

自定义自己的区域信息解析器

```Java
package com.example.springboot01helloworld.component;

import org.springframework.web.servlet.LocaleResolver;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;

public class MyLocaleResolver implements LocaleResolver {

    @Override
    public Locale resolveLocale(HttpServletRequest httpServletRequest) {
        String l = httpServletRequest.getParameter("l");
        Locale locale = Locale.getDefault();
        if (!StringUtils.isEmpty(l)) {
            String[] split = l.split("_");
            locale = new Locale(split[0], split[1]);
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Locale locale) {

    }
}
```

然后将这个解析器注册成组件。

MyMvcConfig

```
@Bean
public LocaleResolver localeResolver() {
    return new MyLocaleResolver();
}
```



### 

##### 登录

1）指定登录action

```
<form class="form-signin" action="dashboard.html" th:action="@{/user/login}" method="post">
```

2)编写登录接口

LoginController.java

```
@RequestMapping(value="/user/login", method= RequestMethod.POST)
public String login()
{
    return "dashboard";
}
```

```
@GetMapping
@PostMapping
@DeleteMapping
@PutMapping
public String login()
{
    return "dashboard";
}
```

3)修改login name

4)禁用thymeleaf缓存

application.properties中 设置spring.thymeleaf.cache=false

idea中 ctrl+F9重新编译

5)显示错误消息

```
<p style="color:red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```

成功后先来到我们的成功页面。如果在页面F5刷新 会出现 确认重新提交。

防止重复提交:重定向。

重定向需要用到视图模板解析 加视图映射。

MyMvcConfig中

```
registry.addViewController("/dashboard").setViewName("dashboard");
```



##### 拦截器-登录检查

SpringMvc中的Interceptor

自定义组件

```Java
package com.example.springboot01helloworld.component;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 登录检查
 */
public class LoginHandlerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 目标方法执行之前
        Object user = request.getSession().getAttribute("loginUser");
        if (user == null) {
            // 未登录
            request.setAttribute("msg", "没有权限 请先登录");
            request.getRequestDispatcher("/index.html").forward(request, response);
            return false;
        } else {
            // 已登录
            return true;
        }
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```



配置组件 MyMvcConfig

```Java
    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        super.addInterceptors(registry);

        // 注册拦截器 拦截多层请求 排除路由
        // 拦截其无需处理静态
        registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**")
                .excludePathPatterns("/index.html", "/", "/login", "/user/login");
    }

```

最左上角展示用户名

```
<a class="navbar-brand col-sm-3 col-md-2 mr-0" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
   [[${session.loginUser}]]
</a>
```



##### 员工列表

要求

1)Restful curd

uri: /资源名称/资源标识 Http请求方式 区分对资源curd的操作

|      | 普通curd  | restful curd    |
| ---- | --------- | --------------- |
| 查询 | getEmp    | GET emp         |
| 添加 | addEmp    | POST emp        |
| 修改 | updateEmp | PUT emp/{id}    |
| 删除 | deleteEmp | DELETE emp/{id} |

2)实验的请求路径

|                              | 请求uri   | 请求方式 |
| ---------------------------- | --------- | -------- |
| 查询所有员工                 | emps      | GET      |
| 查询某个                     | emps/{id} | GET      |
| 添加的页面                   | emp       | GET      |
| 添加员工                     | emp       | POST     |
| 来到修改页面(查员工信息回显) | emp/{id}  | GET      |
| 修改员工                     | emp/{id}  | PUT      |
| 删除员工                     | emp/{id}  | DELETE   |
|                              |           |          |

员工列表

抽取公共片段

```Java
<nav class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0" th:fragment="topbar">
   <a class="navbar-brand col-sm-3 col-md-2 mr-0" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
      [[${session.loginUser}]]
   </a>
   <input class="form-control form-control-dark w-100" type="text" placeholder="Search" aria-label="Search">
   <ul class="navbar-nav px-3">
      <li class="nav-item text-nowrap">
         <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Sign out</a>
      </li>
   </ul>
</nav>
```

```Java
<!--       引入topbar-->
      <div th:insert="~{dashboard::topbar}"></div>
```

三种引入功能片段的th属性

th:insert  

th:replace

th:include

效果:

th:insert 给当前元素插入公共片段

th:replace 将当前元素替换成当前公共片段

th:include 将被引入



1)高亮

引入片段时候传入参数

```
<div th:replace="common/bar::#sidebar(activeUri='main.html')"></div>
```

```
th:class="${activeUri == 'emps' ? 'nav-link active' : 'nav-link'}"
```

2)列表展示

```
<tr th:each="emp:${emps} ">
    <td th:text="${emp.getId()}"></td>
   <td th:text="${emp.getLastName()}"></td>
   <td th:text="${emp.getGender() == 0}? '女': '男'"></td>
   <td th:text="${emp.getDepartment().getDepartmentName()}"></td>
   <td th:text="${emp.getBirth()}"></td>
</tr>
```



#####  错误处理

1）Sb的默认处理机制

(1)返回一个默认的错误页面 浏览器

(2)如果是其他客户端 默认响应json

客户端切换成postman之后  

{

  "timestamp": 1585661753050,

  "status": 404,

  "error": "Not Found",

  "message": "No message available",

  "path": "/aaaaa"

}



原理参照

SpringBoot org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration 错误处理的自动配置

1.DefaultErrorAttributes

（1）帮我们在页面共享信息 timestamp status error exception message errors JSR303错误

(2)

2.BasicErrorController

处理默认的/error请求

public ModelAndView errorHtml(){

ResolverErrorView 去哪个页面 是由DefaultErrorViewResolver决定。

默认去找error/404这样的页面

}

@RequestMapping

public function error()

3.ErrorPageCustomizer

系统出现错误4xx/5xx等错误 ErrorPageCustomizer生效(定制错误响应规则) ErrorPage->path(配置文件中error.path:/error)

4.DefaultErrorViewResolver



2)如何定制错误响应

(1)如何定制错误页面

1）有模板引擎情况下 error/状态码.html 将错误页面命名为错误状态码 放在模板引擎文件夹下 error文件夹下/状态码.html

可以使用4xx.html作为这种类型的所有错误。(优先精确查找)

2)无模板引擎情况下 (模板引擎下找不到错误页面)

默认在静态资源文件夹下查找static

3)以上都没有 就默认来到sb的默认错误提示页面



(2)如何定制json错误数据

exception/UserNotExistException

1.创建自己的异常类

```Java
package com.example.springboot01helloworld.exception;

public class UserNotExistException extends RuntimeException {
    public UserNotExistException() {
        super("用户不存在");
    }
}
```

2.在控制器内使用

```
@RequestMapping("/hello")
public String hello(@RequestParam("user") String user) {
    if (user.equals("aaa")) {
        throw new UserNotExistException();
    }
    return "Hello world";
}
```



1.自定义错误处理器

```Java
package com.example.springboot01helloworld.Controller;

import com.example.springboot01helloworld.exception.UserNotExistException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
public class MyExceptionHandler {
    @ResponseBody
    @ExceptionHandler(UserNotExistException.class)
    public Map<String, Object>  handleException(Exception ex) {
        Map<String, Object> map = new HashMap<>();
        map.put("code", "user.notexist");
        map.put("message", ex.getMessage());
        return map;
    }
}
```

缺点:没有自适应效果。。。

2.转发到error进行自适应响应效果

```Java
    @ExceptionHandler(Exception.class)
    public String  handleException(Exception ex, HttpServletRequest request) {
        Map<String, Object> map = new HashMap<>();
        request.setAttribute("javax.servlet.error.status_code", 500);
        map.put("code", "user.notexist");
        map.put("message", ex.getMessage());
        return "forward:/error";
    }
```

3.将我们的定制数据携带出去

出现错误以后会来到/error请求 会被BasicErrorController处理 响应出去的数据是由

getErrorAttributes得到的。(AbstractErrorController 规定的方法)。

​    (1)完全编写一个errorController的实现类放入容器中。 或者编写一个子类 重写某个方法。

​    (2)页面上能用的数据 或者json返回的数据都是通过errorAttributes.getErrorAttributes获得。

​       容器中DefaultErrorAttributes.getErrorAttributes



```Java
package com.example.springboot01helloworld.component;

import org.springframework.boot.web.servlet.error.DefaultErrorAttributes;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.WebRequest;

import java.util.Map;

@Component 添加入sb的组建
public class MyErrorAttributes extends DefaultErrorAttributes {
    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
        Map<String, Object> map = super.getErrorAttributes(webRequest, includeStackTrace);
        map.put("company", "mmm");
        Map<String, Object> ext = (Map<String, Object>)webRequest.getAttribute("ext", 0);
        map.put("ext", ext);
        return map;
    }
}
```

```
@ExceptionHandler(Exception.class)
    public String  handleException(Exception ex, HttpServletRequest request) {
        Map<String, Object> map = new HashMap<>();
        request.setAttribute("javax.servlet.error.status_code", 500);
        map.put("code", "user.notexist");
        map.put("message", ex.getMessage());
        request.setAttribute("ext", map);
        return "forward:/error";
    }
```



#### 配置嵌入式Servlet容器

SpringBoot默认使用Tomcat作为嵌入式的Servlet容器。

1)如何定制和修改Servlet容器的相关配置

2)能否切换其他的Servlet容器
>>>>>>> c7a898dee5bc19da54ef35f359e07dd508d1633f
