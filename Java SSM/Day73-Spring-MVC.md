# Spring MVC

#### 内容摘要

MVC 回顾，Spring mvc Intro，架构，入门，入门注解配置，URL 路径映射，窄化请求映射，请求方法限定，POJO实现，返回字符串，

---

#### MVC 回顾

> **Model - View - Controller**
>
> **Model** - 模型层
>
> * 数据以及基于数据的操作
>
> **View** - 视图层
>
> * 对数据的显示
>
> **Controller** - 控制层
>
> * 接收请求，获取数据，分发任务，结果转向
> * 对 **Model** 的操作，更新到 **View**
> * 对 **View** 的输入，更新到 **Model**
> * **Model**、**View** 之间可以做到**松散耦合**
>
> **高内聚、低耦合**
>
> **解耦合**
>
> 将控制逻辑和表现层分离

#### Spring MVC Intro

没有使用 Spring MVC 的不足 

* 请求参数封装麻烦
* 结果视图耦合性强
* url 和 controller 的方法，映射不灵活（映射 Servlet）
* controller 的实现需要继承 **HttpServlet** ，不够灵活

Spring MVC 优点

* 请求参数封装简单
* 结果视图配置方便（使用模版生成静态页面）
* url 和 controller 的方法，映射灵活（映射 方法）
* controller 的实现 **POJO** 就可以实现，灵活，不需要继承

MVC vs Spring MVC

* MVC 是一种架构，设计思想
* Spring MVC 是三层架构中 MVC 层的一种框架

#### Spring MVC 架构

接收请求，获取数据，分发任务，结果转向，**Java EE 规范**

1. **DispatcherServlet** 核心处理器
2. **HandlerMapping** 映射处理器
   * **HandlerExecuntionChain**
3. **HandlerAdapter** 处理器适配器
4. 用户 **Controller** 返回 **ModelAndView**
5. **ViewResolver** 视图解析
6. **View** 视图渲染

(tu)

#### Spring MVC 入门

1. 导包

* 5 + 1：core、aop、expression、beans、context、logging
* `spring-web-5.0.4.RELEASE.jar`
* `spring-webmvc-5.0.4.RELEASE.jar`

2. **WEB-INF/web.xml** 配置 **DispatcherServlet**

```xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1" metadata-complete="false"> <!-- 允许使用注解和 xml 并存配置 -->
    
    <servlet>
        <servlet-name>springMVCServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <!-- 配置初始化参数，加载 Spring 配置文件，创建 Bean -->
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:webContext.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <!-- url mapping *.do -->
        <servlet-name>springMVCServlet</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
    
</web-app>
```

3. **webContext.xml** 配置 依赖 **Bean**

* 映射处理器 **BeanNameUrlHandlerMapping** 根据 **Bean** 的 **name** 作 URL 映射
* 处理器适配器 **SimpleControllerHandlerAdapter** 
  * 适配用户自定的 **Handler（Controller）**
  * **要求用户实现 `org.springmvc.web.servlet.mvc.Controller` 接口**
  * `handleRequest()`

```xml
<!-- 根据 Bean name 作 url 映射 -->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
<!-- 适配用户自定的 Handler（Controller） -->
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
```

4. **Controller**  **webContext.xml** 配置，要实现 `org.springframework.web.servlet.mvc.Controller` 接口

* `ModelAndView handleRequest(HttpServletRequest req, HttpServletResponse resp)`

```java
public class MyController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        System.out.println("Hello Spring MVC");
        return null;
    }
}
```

5. **springmvc.xml** 配置 Spring MVC bean 以及 **URL 映射**

```xml
<!-- 配 name ，对应上面 BeanNameUrlHandlerMapping 根据 bean name - url 映射 -->
<!-- url mapping : /demo/hello.do -->
<bean name="/demo/hello.do" class="day73.spring.demo.controller.MyController"/>
```

6. **ModelAndView**  模型和视图

```java
public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
    ModleAndView mav = new ModleAndView();
    // 放入模型
    mav.addObject("message", "Hello Spring MVC");
    // 放入视图
    mav.setViewName("/index.jsp");
    return mav;
}
```

#### Spring MVC 入门 注解配置

1. 导包，同上
2. **WEB-INF/web.xml** 配置 **DispatcherServlet** 同上
3. **webContext.xml** 配置 
   * 映射处理器 **BeanNameUrlHandlerMapping**
   * 处理器适配器 **SimpleControllerHandlerAdapter** 

```xml
<!-- 引入约束 -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" 
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd 
        http://www.springframework.org/schema/mvc 
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
```

```xml
<context:component-scan base-package="day73.spring"/>    
<!-- 会加载所有映射处理器和处理器适配器，根据注解自动匹配对应的 映射处理器和处理器适配器 -->
<mvc:annotation-driven/>
```

4. **Controller** 注解配置，对应上面第 4 ，第 5 步
   * `@Controller` ==> `<bean class="MyController"/>`
   * `@RequsetMapping("/demo/hello.do")` ==> `<bean name="/demo/hello.do"/>`

```java
@Controller
public class MyController implements org.springframework.web.servlet.mvc.Controller {

    @RequestMapping("/demo/hello.do")
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
       // ...
    }
}
```

5. **ModelAndView**  模型和视图，同上

   ​


#### URL 路径映射 @RequestMapping

`@RequestMapping()` 可以配置多个路径， 字符串数组  

```java
@RequestMapping(value = {"/info.do", "/user.do"}) // value() = path()
```

可以省略后缀，`.do`， `/` 可以省略，实质上是加不加都相当于加 `/` 

* 如果 **DispatcherServlet** 匹配 `<url-pattern>*.do</url-pattern>`

```java
// 三者等价，如果 DispatcherServlet 匹配 <url-pattern>*.do</url-pattern> 
@RequestMapping("/hello"})  // localhost/hello.do
@RequestMapping("/hello.do"}) // localhost/hello.do
@RequestMapping("hello"}) // localhost/hello.do
```

URL 映射可以使用通配

* `*` 匹配一级目录，或字符串匹配

```java
// 匹配一级目录
@RequestMapping("/*/user.do") // localhost/a/user.do
// 匹配字符串
@RequestMapping("/user*.do") // localhost/useruser.do
@RequestMapping("/*user*.do") // localhost/prefixuseruser.do
// 一起使用
@RequestMapping("/*/user*.do") // localhost/abc/userabc.do
```

* `**` 匹配多级目录

```java
@RequestMapping("/**/user.do") // localhost/abc/efg/user.do
```

* `?` 匹配一个字符

```java
@RequestMapping("/user?.do") // localhost/user1.do
```

```java
// 一起使用
@RequestMapping("/*/**/?user?.do") // localhost/abc/def/gh/@user!.do
```

#### 窄化请求映射 @RequestMapping

在**类上**声明 `@RequestMapping("/user")` ，限制该类下的所有 `@RequestMapping()`  以 `/user` **前缀**开头

```java
@RequestMapping("/user")
public class UserController {
     @RequestMapping("/user.do")
    public ModelAndView showUser() {
        // localhost/user/user.do
    }
}
```

#### 请求方法限定 @RequestMapping

**RequestMethod[] method**，匹配具体请求方式

```java
@RequestMapping(value = "/get.do", method = RequestMethod.GET)
// 如果请求方式不匹配 RequestMethod.POST
// HTTP Status 405 – Method Not Allowed
```

**String[] prams**，匹配具体的请求参数

```java
// 如果请求参数不匹配 localhost/get.do
// Parameter conditions "id" not met for actual request parameters:

// 匹配请求参数 localhost/get.do?id
@RequestMapping(value = "/get.do", params = "id") 
// 匹配请求参数 localhost/get.do?id=2
@RequestMapping(value = "/get.do", params = "id=2") 
// 匹配请求参数 localhost/get.do?id=3
@RequestMapping(value = "/get.do", params = "id!=2") 

// 匹配请求参数 localhost/get.do?id>2  id>2 只是参数名， 不支持 > < 比较
@RequestMapping(value = "/get.do", params = "id>2") 
```

**String[] headers**，匹配具体的请求头

```java
// 匹配 localhost:8080/get.do
// http://127.0.0.1:8080/get.do 匹配不到
@RequestMapping(value = "/get.do", headers = "Host=localhost:8080")
```

**String[] consumes**，匹配指定处理请求提交内容类型（**response** Content-Type）

```java
// 匹配失败 HTTP Status 415 – Unsupported Media Type
// 处理请求的内容为 text/html application/json
@RequestMapping(value = "/get.do", consumes = "text/html")
```

**String[] producers**，匹配指定响应接受返回内容类型（**request**  Accept)

```java
// 处理响应的内容为 text/html application/json
@RequestMapping(value = "/get.do", produces = "text/html")
```

#### Controller POJO 实现

实现方式一：实现 `org.springframework.web.servlet.mvc.Controller` 接口

实现方式二：纯 POJO 实现

```java
@Controller
public class UserController {
   // 不用继承 Controller 接口，使用注解
   // @RequestMapping
}
```

返回 **ModleAndView**

```java
@RequestMapping("/show.do")
public ModelAndView showUser() {
    ModelAndView mav = new ModelAndView("/user.jsp");
    User user = new User("root", "root", 22);
    mav.addObject(user);
    return mav;
}
```

返回 **void**

* 没有返回结果视图，但是可以拿 Request Response HttpSession 等

```java
@RequestMapping("/get.do")
public void getUser(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    User user = new User("root", "root", 22);
    request.setAttribute("user", user);
    request.getRequestDispatcher("/user.jsp").forward(request, response);
}
```

####返回字符串

返回物理视图，对应 web 下物理文件路径

* 可以返回 `WEB-INF` 下的资源，原理是内部转发

```java
@RequestMapping("/home.do")
public String view() {
    return "/index.jsp";
}

// 可以返回 WEB-INF 下的资源，原理是内部转发
@RequestMapping("/user.do")
public String view() {
    return "/WEB-INF/user.jsp";
}
```

`/index.jsp`

* 以 `/` 开始，以 **web** 为根目录的相对路径，推荐使用
* 不以 `/` 开始，以当前请求 URL 的父路径的相对路径

返回字符串，返回逻辑视图

* 返回物理视图的部分，页面解析器拼接


* 使用页面解析器 **InternalResourceViewResolver**，配置 **webContext.xml**

```xml
<!-- 配置页面解析器，返回逻辑视图 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/"/>
    <property name="suffix" value=".jsp"/>
</bean> 
```

```java
// 物理视图 localhost/WEB-INF/user.jsp
@RequestMapping("/web.do")
public String logicView() {
    return "user";
}
```

**Model** 传递

```java
// 方式一 
@RequestMapping("/show")
public String show(HttpServletRequest request) {
    User user = new User("request", "request", 22);
    request.setAttribute("user", user);
    return "/user.jsp";
};
```

```java
// 方式二 Model
@RequestMapping("/show")
public String show(Model model) {
    User user = new User("model", "model", 22);
    model.addAttribute(user);
    return "/user.jsp";
}
```

默认转发，也可以显式声明  `forword:` ，转发路径同 **RequestDispatcher**

```java
@RequestMapping("/forward")
public String forward() {
    return "forward:/index.jsp";
}
```

重定向，`redirect:  ` ，重定向路径同 **forward** 因为**会自动拼接应用名 contentPath**

```java
@RequestMapping("/redirect")
public String redirect() {
    return "redirect:/index.jsp";
}
```

#### 动态加载 DispatcherServlet 不用配置 web.xml

```java
public class InitDispatcherServlet implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = servletContext.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("*.do");
    }
}
```

### Spring 运行结构图

