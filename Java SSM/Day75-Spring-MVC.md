# Spring MVC

#### 内容摘要

Json 简介，JavaScript 对象， Json 优势，Java Json，Spring MVC Json，拦截器 Interceptor，、Spring 异常处理

------

#### Json 简介

***JavaScript Object Notation***  JavaScript 对象表示法，轻量级数据交换格式，是一个种**数据格式**

#### JavaScript 对象

解析方便

```javascript
/*
class Person {
	String name;
	int age;
}
*/
var person = {
    "name": "Tom",
    "age": 18
}
// 使用，解析方便
alert(person.name);
```

对象引用  `{ }`

```javascript
/*
class Person {
	String name;
	ing age;
	Info info;
}
class Info {
	String hobby;
}
*/
var person = {
    "name": "Tom",
    "age": 18,
    "info": {
    	"hobby": "java"
	}
}
alert(person.info.hobby);
```

数组  `[ ]`

```javascript
/*
class Person {
	String name;
	ing age;
	Info[] infos;
}
class Info {
	String hobby;
}
*/
var person = {
    "name": "Tom",
    "age": 18,
    "infos": [
        {"hobby": "java"},
        {"hobby": "python"}
    ]
}
alert(person.infos[0].hobby);
```

#### Json 优势

可读性，可扩展性，解析难度（要预先知道 Json 结构），数据效率（操作 JS 对象）

不足？没有约束文件

#### Java Json

导包

* `jackson-annotations-2.9.4.jar`
* `jackson-core-2.9.4.jar`
* `jackson-databind-2.9.4.jar`

```java
// Servlet
protected void doGet(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {
    // ... 假设去数据库里查到了 Weather weather

    ObjectMapper objectMapper = new ObjectMapper();
    String json = objectMapper.writeValueAsString(weather);
    response.getWriter().write(json);
}
```

#### Spring MVC Json 发送

导包，同上

**METHOD** `@ResponseBody`  把方法返回值解析为 **Json** 文本写入响应正文

```java
@Controller
public class WeatherController {

    @RequestMapping("/weather")
    @ResponseBody
    public Weather getWeather() {
        // ... 假设去数据库里查到了 Weather weather
        return weather;
    }
}
```

返回值可以是 **JavaBean，List ，Map**

```java
// 当 User 有很多属性字段，只要求返回 User 部分信息时，使用 Map 手动设置，灵活
public Map<String, String> findPartUser() {
   // ... 假设去数据库里查到了 User user
    Map<String, String> map = new HashMap<>();
    map.put("username", user.getUsername());
    map.put("password", user.getPassword());
    return map;
}
```

#### Spring MVC Json 解析

引入 **Jquery** 库

```html
<!-- 导包不能写单标签 -->
<script type="text/javascript" src="js/jquery-3.3.1.js"></script>
```

```javascript
// Jquery 发送 Ajax 异步请求，请求数据
$.ajax({
    type: "post",
    url: "${pageContext.request.contextPath}/findUser.do",
    contentType: "application/json;charset=utf-8", // 定义接受 json 数据的响应报文
    success: function (data) { 
        // data 是已经解析好了的 JS Json 对象
        console.log(data);
    }
});

// 向服务器提交数据
function createUser() {
    $.ajax({
        type: "post",
        url: "${pageContext.request.contextPath}/createUser.do",
        contentType: "application/json;charset=utf-8",
        // data json 数据格式，必须是单引号开头，用双引号标示数据
        data: '{"username": "admin", "password": "admin123"}'
    });
}
```

**FIELD** `@RequestBody`  将 **POST** 请求正文  **Json** 数据封装为请求参数 **JavaBean** 传入

```java
// 向请求正文封装为 json 数据写入
@RequestMapping("/findUser")
@ResponseBody
public User findUser() {
    User user = new User();
    user.setUsername("root");
    user.setPassword("root");
    return user;
}

// 接收请求正文 json 数据封装为参数
@RequestMapping("/createUser")
@ResponseBody
public User createUser(@RequestBody User user) {
    System.out.println("user = " + user);
    return user;
}
```

### Spring MVC 拦截器 Interceptor

类似 **Filter**，是 **DispatcherServlet** 内部对匹配到的 **RequestMapping** 预处理、后处理以及处理完成后回调

常见使用场景

* 日志记录
* 权限检查
* 性能监控
* 通用行为，如乱码问题
* **OpenSessionInView**，**Hibernate** 缓存

**接口 `HandlerInterceptor`** 

* `boolean preHander(request, response, Object handler)`  
  * 预处理，执行 **handler** 方法前调用
  * **handler** 当前 响应**URL** 的处理器，即 **Controller** 
  * 返回 **true** 放行， **false** 拦截，不再调用下一个拦截器，也不会调用 **Controller** 
  *  **false** 拦截后，要用 **response** 返回响应
* `void postHander(request, response, Object handler, ModleAndView mav)`  
  * 后处理，执行 **handler** 方法后调用，视图解析后，视图渲染之前
  * **handler** 当前 响应**URL** 的处理器，即 **Controller** 
  * **mav** 视图结果，可以对结果进行修改
* `void afterCompletion(request, response, Object handler, Exception e)`
  * 整个请求完毕，视图渲染完毕，返回响应之前
  * 类似于 **finally** ，对资源进行清理或处理异常
  * 仅对 **preHandler** 返回 **true**，放行才能执行到

配置

```xml
<mvc:interceptors>
	<!-- 默认拦截所有 DispatcherServlet 配置的映射 *.do -->
	<bean class="day75.spring.demo.interceptor.FirstInterceptor"/>
</mvc:interceptors>
```

实现 **HandlerInterceptor** 接口

```java
public class FirstInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response, 
                             Object handler) throws Exception {
        System.out.println("FirstInterceptor.preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler, 
                           ModelAndView modelAndView) throws Exception {
        System.out.println("FirstInterceptor.postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, 
                                HttpServletResponse response, 
                                Object handler, 
                                Exception ex) throws Exception {
        System.out.println("FirstInterceptor.afterCompletion");
    }
}
```

#### 多个 Interceptor

* 都放行，同 **Filter** ，递归调用，递归返回
* 一旦拦截
  * 不会执行 **Controller** 
  * 不会调用 **postHandler**
  * 仅对对放行的 **preHandler** 返回 **true**，执行 **afterCompletion**

```xml
<!-- 拦截顺序同声明顺序 -->
<mvc:interceptors>
    <bean class="day75.spring.demo.interceptor.FirstInterceptor"/>
    <bean class="day75.spring.demo.interceptor.SecondInterceptor"/>
</mvc:interceptors>
```

#### 多个 Interceptor 过滤 URL

```xml
<mvc:interceptors>
    <bean class="day75.spring.demo.interceptor.FirstInterceptor"/>
    <bean class="day75.spring.demo.interceptor.SecondInterceptor"/>
    <mvc:interceptor>
        <mvc:mapping path="/user/*"/>
        <bean class="day75.spring.demo.interceptor.ThirdInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 拦截流程

1. 先找 Handler（RequestMapping）
   1. 没有找到 Handler 直接返回 404， 不会执行拦截
2. 根据 URL 找匹配的 Interceptor ，进行过滤操作

### Spring MVC 异常处理

**dao**，**service**，**controller** 都抛异常，交给**异常处理器**处理，解耦

先自定义异常

```java
public class MyException extends Exception {
    public MyException(String message) {
        super(message);
    }
}
```

自定义异常处理视图，实现接口 **HandlerExceptionResolver**

```java
public class CustomExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest,
                                         HttpServletResponse httpServletResponse,
                                         Object o,
                                         Exception e) {
        ModelAndView mav = new ModelAndView("/error.jsp");
        if (e instanceof MyException) {
            // 如果匹配到自定义异常
            mav.addObject("massage", e.getMessage());
        } else {
            mav.addObject("massage", "unknown exception");
        }
        return mav;
    }
}
```

**webContext.xml**  配置

```xml
<!-- 自定义异常视图解析，命名首字母小写 -->
<bean id="handlerExceptionResolver" class="day75.spring.demo.exception.CustomExceptionResolver"/>
```