# Spring MVC

#### 内容摘要

request 封装形式，自动封装，乱码问题，日期转换问题，常见注解，RESTful，文件上传

---

#### Spring MVC 请求参数封装

* 使用 request 封装形式
* 自动封装
* 其他补充
* 封装中乱码问题
* 封装中日期问题


#### request 封装形式

```java
@RequestMapping("/register")
public String register1(HttpServletRequest request) {
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    String age = request.getParameter("age");
    ...
}
```

#### 自动封装

直接在参数列表出现表单提交同名参数，**表单 name 要与 参数名 对应**

```html
<form action="${pageContext.request.contextPath}/register.do" method="post">
    <h2>username: <input type="text" name="username"></h2>
    <h2>password: <input type="password" name="password"></h2>
    <h2>age: <input type="text" name="age"></h2>
    <input type="submit" value="register">
</form>
```

```java
public String register2(String username, String password, int age, Model model) {
    User user = new User(uesrname, password, age);
    // ... 
    model.addAttribute("user", user);
    return "/user.jsp";
}
```

自动封装 **Java Bean**

* Bean 的成员变量名，要**与表单 name 对应**

```java
@RequestMapping("/register")
public String register3(User user, Model model) {
    model.addAttribute("user", user);
    return "/user.jsp";
}
```

* 如果 Bean 里有引用变量，表单 **name** 带上前缀，并与参数名对应 

```html
<form action="${pageContext.request.contextPath}/register.do" method="post">
    <!-- ... -->
    <h2>teacherName: <input type="text" name="teacher.name"></h2>
    <h2>teacherCourse: <input type="text" name="teacher.course"></h2>
    <input type="submit" value="register">
</form>

<!-- 多级嵌套引用 -->
<h2>teacherName: <input type="text" name="teacher.name"></h2>
<h2>CourseName: <input type="text" name="teacher.course.name"></h2>
```

```java
public class User {
    //...
    private Teacher teacher;
    // ... 
}
public class Teacher {
    private String name;
    private Course course;
    // ...
}
public class Course {
    private String name;
     // ...
}
```

封装 **数组** 类型 **String[]**  int[]

```html
<h2>hobby:
    <input type="checkbox" name="hobby" value="java">Java
    <input type="checkbox" name="hobby" value="C/C++">C/C++
    <input type="checkbox" name="hobby" value="Python">Python
</h2>
```

```java
// 参数列表
@RequestMapping("/register")
public String register4(User user,String[] hobby, Model model) {
	// ...
    System.out.println("hobby = " + Arrays.toString(hobby));
    return "/user.jsp";
}
// 自动封装到 Java Bean
public class User {
    //...
    private String[] hobby;
    // ... 
}
```

封装 **集合** 类型 `${listName}[index].property`

```html
<h2>course 01: <input type="text" name="courseList[0].name"></h2>
<h2>course 02: <input type="text" name="courseList[1].name"></h2>
<h2>course 03: <input type="text" name="courseList[2].name"></h2>
```

```java
public class User {
    //...
    private List<Course> courseList;
    public List<Course> getCourseList() {
        return courseList;
    }
    public void setCourseList(List<Course> courseList) {
        this.courseList = courseList;
    }
    // ... 
}
```

#### 封装中的乱码问题

**POST** 请求乱码，手动实现 **CharacterEncodingFilter** 

```java
@WebFilter(filterName = "CharacterEncoding", value = "/*")
public class CharacterEncoding implements Filter {
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) 
        throws ServletException, IOException {
        req.setCharacterEncoding("utf-8");
        chain.doFilter(req, resp);
    }
    // init() destory()
}
```

**GET** 请求乱码，**Tomcat** 默认 **UTF-8** 解码

**Spring** 解决方案，配置 **web.xml**，不用手动实现

```xml
<filter>
    <filter-name>SpringCharacterEncoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>SpringCharacterEncoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

#### 封装中的日期转换问题

对于不支持的封装类型，如 Date ，报错 `HTTP Status 400 – Bad Request`，需要转换器来手动转换

实现 `Converter<String, Data>` 接口，自定义日期格式转换器

```java
public class DateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String s) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        Date birthday = null;
        try {
            birthday = dateFormat.parse(s);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return birthday;
    }
}
```

```xml
<mvc:annotation-driven conversion-service="converterService"/>
<bean id="converterService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="day74.spring.demo.util.DateConverter"/>
        </set>
    </property>
</bean>
```

#### 封装中的常见注解 @RequestParam

`@RequestParam` 等同 `@RequestMapping(params)`，表示接受具体的参数，否则报错

```java
@RequestMapping("/login")
public String login(@RequestParam String username, String password) {
    // ...
    return "/user.jsp";
}
```

`String value() default "";` 绑定请求参数给形式参数赋值 

```java
@RequestMapping("/login")
public String login(@RequestParam("username") String username,
                    @RequestParam("password") String password) {
    // ...绑定请求参数
    return "/user.jsp";
}
```

`String defaultValue() default "\n\t\t\n\t\t\n\ue000\ue001\ue002\n\t\t\t\t\n";`

```java
// /hello.do?username=Tom
// 设置请求参数默认值
@RequestMapping("/hello")
public String hello(@RequestParam(defaultValue = "World") String username, Model model) {
    model.addAttribute("username", username);
    return "/index.jsp";
}
```

#### 封装中的常见注解 @PathVariable

从请求路径取请求参数， **RESTful** 风格  **Representational State Transfer**

```java
@RequestMapping("/restful/{user}/{id}")
public void get(@PathVariable("user") String user,
                @PathVariable("id") int id) {
    System.out.println("user = " + user);
    System.out.println("id = " + id);
}
@GetMapping @PostMapping  @PutMapping  @PatchMapping  @DeleteMapping 
```

#### 封装中的常见注解 @RequestHeader

获取指定请求头参数

```java
@RequestMapping("/header")
public String header(@RequestHeader("host") String host,
                     @RequestHeader("Accept") String[] accept) {
    System.out.println("host = " + host);
    System.out.println("accept = " + Arrays.toString(accept));
    return "/index.jsp";
}
```

#### 封装中的常见注解 @CookieValue

获取指定名的 Cookie 的值

```java
@RequestMapping("/cookie")
public String cookie(@CookieValue("JSESSIONID") String id) {
    System.out.println("id = " + id);
    return "/index.jsp";
}
```

#### 封装中的常见注解 @SessionAttributes

**CLASS** `@SessionAttributes`  指定 **Model** 中的属性，放到 **Session** 域中

```java
// @SessionAttributes(types = User.class) 通过 Model 将指定类型放入 Session
@SessionAttribute("user")
@Controller
class UserController {
   // 指定 Model 将 user 属性名的域属性，放到 Session 中
    @RequestMapping("/login")
    public String login(String username, String password, Model model) {
        User user = new User();
        user.setUsername(username);
        user.setPassword(password);
        model.addAttribute("user", user);
        return "/index.jsp";
    }
}
```

#### 封装中的常见注解 @SessionAttribute

获取  **Session** 域中指定属性名的属性值

```java
// 从 Session 中拿到 user 属性值
@RequestMapping("/show")
public String show(@SessionAttribute("user") User user) {
    System.out.println("user = " + user);
    return "/user.jsp";
}
```

将 Model 中 Session 域中东西移除

```java
public String invalidate (SessionStatus sessionStatus) {
    sessionStatus.setComplete();
}
```

#### 封装中的常见注解 @ModelAttribute 

**METHOD**  `@ModelAttribute`  将方法的返回值绑定到 **Model** 中

* 每次 `@RequestMapping` 方法前调用
* 如果没有指定属性名，默认是返回值类型首字母小写，User ==> `@ModelAttribute("user")` 
* 可以获取请求参数

```java
// key = "user" 
// value = 方法的返回值
@ModelAttribute
public User initUser() {
    return new User("root", "root");
}

// 可以获取请求参数
@ModelAttribute
public User initUser(User user) {
    return user;
}

@RequestMapping("/show")
public String show(){
    return "/user.jsp";
}
```

**FIELD**  `@ModelAttribute`  数据绑定

* 从 **Model** 中去拿 **指定属性名** 或者 **与参数类型相同** 的属性
* 如果没有拿到，会先自动注入请求参数，再放入 **Model** 中

```java
@RequestMapping("/show")
public String show(@ModelAttribute User user){
    return "/user.jsp";
}
// 等同于
@RequestMapping("/show")
public String show(User user, Model model){
    model.add(user);
    return "/user.jsp";
}
```

**FIELD** 和 **METHOD** 都出现时

*  `@RequestMapping` 方法前调用 `@ModelAttribute` 的方法，将返回值放入 **Model** 中
  * 如果没有指定属性名，会按照 [返回值类型] 首字母小写 放入 **Model** 中
* 在  `@RequestMapping` 方法参数上，去 **Model** 中查找，通过 **指定属性名** 或 **参数类型** 自动注入到参数
  * 如果没有指定属性名，会按照 [参数类型] 首字母小写 去 **Model** 查
* 若请求参数有相同参数名，会覆盖 **Model** 中同名属性值

```java
@ModelAttribute
public User initUser() {
    return new User("root", "root");
}
// 此时，user{username = "root", password = "root"}
@RequestMapping("/show")
public String show(@ModelAttribute User user){
    return "/user.jsp";
}
// 如果请求参数有相同参数名，会覆盖
```

#### RESTful

**Representational State Transfer ** 表现层状态转化

> 1. 每一个URI代表一种资源；
> 2. 客户端和服务器之间，传递这种资源的某种表现层；
> 3. 客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

> * GET（SELECT）：从服务器取出资源（一项或多项）。
> * POST（CREATE）：在服务器新建一个资源。
> * PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
> * PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
> * DELETE（DELETE）：从服务器删除资源。

> * ?limit=10：指定返回记录的数量
> * ?offset=10：指定返回记录的开始位置。
> * ?page=2&per_page=100：指定第几页，以及每页的记录数。
> * ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
> * ?animal_type_id=1：指定筛选条件

#### Spring MVC 文件上传

导包

* `commons-fileupload-1.3.1.jar`
* `commons-io-2.4.jar`

**webContext.xml** 配置 **MultiPartResolver** 解析器

```xml
<!-- id  -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"/>
```

**MultipartFile**

* `transferTo(File file);`  保存到服务器

```java
@RequestMapping("/upload")
public String upload(HttpServletRequest request,
                     MultipartFile uploadFile, Model model) {
    String message = "您还没有上传文件";
    if (!uploadFile.isEmpty()) {
        // 如果有上传文件
        // 拿到上传文件路径
        String path = request.getServletContext().getRealPath("/img");
        // 创建目录
        File parent = new File(path);
        parent.mkdirs();

        // 拿到文件名 ， UUID
        String filename = uploadFile.getOriginalFilename();
        filename = UUID.randomUUID() + filename;

        // 创建文件
        File file = new File(parent, filename);

        // 写入文件
        try {
            uploadFile.transferTo(file);
            message = "上传文件成功";
            model.addAttribute("imgUrl", filename);
        } catch (IOException e) {
            e.printStackTrace();
            message = "上传文件失败";
        }
    }

    model.addAttribute("message", message);
    return "/index.jsp";
}
```

限制文件上传大小

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="1024000"/> <!-- 1MB -->
</bean>
```

限制文件上传类型

```html
<input type="file" name="img" accept="image/jpeg,image/png,image/jpg">
```

















