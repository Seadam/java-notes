# Java JSP

#### 内容摘要

补充，JSP 标签，include，forward，param，映射 JSP，Java Bean，useBean，setProperty，getProperty，

EL，概述，获取数据，执行运算，获取对象，调用 Java 方法，JSTL，概述，标签库，out，set，remove，

catch，if，choose，forEach

---

#### 补充

* JSP 中可以嵌入 Javascript 代码
* out 和 response.getWriter()
  * 可以设置 `<%@ page buffer="none" %>`，让写入顺序按代码逻辑

#### JSP 标签

* JSP Action 元素，避免直接编写 Java 代码，提高可读性和可维护性
* 常用标签
  * `<jsp:include>`
  * `<jsp:forward>`
  * `<jsp:param>`

####` <jsp:include>`

* 把目标组件 Web 资源的响应内容**插入**到原组件 JSP 页面，**动态引入**
* `<jsp:include page="relativeUrl" | <%= expression%> flush="true | false">`
  * page 指定被引入资源的相对路径，也可以通过执行表达式来获得
  * flush 指定在插入该资源的响应内容时，是否将当前 JSP 页面的响应刷新到浏览器，默认 false
* 翻译：
  * `JspRuntimeLibrary.include(request, response, relativePath, out, autoFlush)`
  * 还是 转发器的 `include()`

```Java
		String resourcePath = getContextRelativePath(request, relativePath);
        RequestDispatcher rd = request.getRequestDispatcher(resourcePath);

        rd.include(request,
                   new ServletResponseWrapperInclude(response, out));
```

#### `<jsp:include>`  标签 与 `<%@ include%>` 指令比较

* `<jsp:include>`
  * 动态引入
  * **多个 JSP 页面会被翻译成多个 Servlet，其内容在执行时合并**
* `<%@ include%>`
  * 静态引入
  * **多个 JSP 页面只会被翻译成一个 Servlet，其内容在源码文件合并**
* 都会把 JSP 内容合并输出，要注意不要出现重复的 HTML 全局架构标签，避免混乱 HTML 文档

#### ` <jsp:forward>`

* 用于把请求转发给另一个 Web 资源
* 还是  `pageContext.forward()`
* `<jsp:forward page="relativeUrl" | <%= expression%>>`
  * page 指定被引入资源的相对路径，也可以通过执行表达式来获得

#### ` <jsp:param>`

* 使用  `<jsp:include>`  和  `<jsp:forward>`  转发或包含时，使用该标签传递参数
* 一次请求中的参数拼接，以 **GET 请求参数拼接**
* 可以使用多个标签传递多个参数
* `<jsp:param name="parameterName" value="parameterValue" | <%= expression%>/>`

```jsp
<jsp:forward page="relativeUrl" | <%= expression%>>
    <jsp:param name="parameterName" value="parameterValue" | <%= expression%>/>
  	<jsp:param name="parameterName" value="parameterValue" | <%= expression%>/>  
</jsp:forward>

<jsp:include page="relativeUrl" | <%= expression%>>
    <jsp:param name="parameterName" value="parameterValue" | <%= expression%>/>  
  	<jsp:param name="parameterName" value="parameterValue" | <%= expression%>/>  
</jsp:include>
```

```Java
// 源码 include get
org.apache.jasper.runtime.JspRuntimeLibrary.include(request, response, "index.jsp" + "?" + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("message", request.getCharacterEncoding())+ "=" + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("include hello servlet", request.getCharacterEncoding()), out, false);

// include post 同上

// forward get
 if (true) {
        _jspx_page_context.forward("index.jsp" + "?" + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("message", request.getCharacterEncoding())+ "=" + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("forward hello servlet", request.getCharacterEncoding()) + "&" + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("text", request.getCharacterEncoding())+ "=" + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("forward text 2", request.getCharacterEncoding()));
        return;
      }
// forward post 同上

```

#### 映射 JSP

* IDEA `<web.xml>`  自动创建 `Facets =>  Deployment Descriptors => + web.xml`
* 不能使用注解配置，只能在 `web.xml` 修改 `servlet-mapping`

```xml
	<context-param>
        <param-name>servletMessage</param-name>
        <param-value>messageFromServletContext</param-value>
    </context-param>

	<!-- Jsp 本质上是 Servlet ，支持 Servlet 其他配置 -->
    <servlet>
        <servlet-name>JspMappingDemo</servlet-name>
        <jsp-file>/JspMappingDemo.jsp</jsp-file> <!-- 必须以 ／ 开头 -->
        <init-param>
            <param-name>jspMessage</param-name>
            <param-value>messageFromJspMapping</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>JspMappingDemo</servlet-name>
        <url-pattern>/map.html</url-pattern>
    </servlet-mapping>
```

#### JSP 小结

* 原理：java => class => servlet
* 最佳实践： Servlet 处理数据，JSP 显示
* out 和 response.getWriter()
* PageEncoding 和 contentType
* errorPage 和 isErrorPage
* 九大隐式对象
  * PageContext 可以设置获取其他域对象的属性
  * 四大域对象 ServletContext > HttpSession > HttpRequst > PageContext

#### Java Bean

遵循特定写法的 Java 类

* 必须有一个无参构造方法（反射用）
* 成员变量必须私有化
* 私有成员变量必须通过 public 方法暴露出来（getter and setter)
* 并且方法的命名必须**遵循一定的命名规范** (getName,  setName)
  * **第三方框架处理 Java Bean 问题**，通过反射，在成员变量名前拼接前缀 get/set
    1. 直接将成员变量名首字母，调用 API 转化为大写字母，安全
    2. 直接将成员变量名首字母，-32 通过计算 ascii 转化为 大写字母，不太安全
       * 若命名规范不一致，就会出错

#### JSP 标签

* `<jsp: useBean>`
* `<jsp: setProperty>`
* `<jsp: getProperty>`
* 使用 JavaBean
  1. 导包 JavaBean 类
     * `<%@ page import="" %>`
  2. 声明 JavaBean 对象
     * `<jsp: useBean>`
  3. 访问 JavaBean 属性
     * `<jsp: setProperty>`
     * `<jsp: getProperty>`

#### `<jsp: useBean>`

* 用于在**指定的域范围内查找或实例化**指定名称的 JavaBean 对象
  * 如果存在，则返回该对象的引用
  * 如果不存在，就实例化一个新的对象，并存储到指定的域范围内
* `<jsp:useBean id="user" class="day44.servlet.bean.User" scope="page"/>`
  * `id`  指定实例对象的引用名称和存储域范围名称
  * `class`  指定 JavaBean 完整类名，带包名
  * `scope`  指定域范围，默认值为 `page`，只能制定一个 `page` / `requset` / `session` / `application`

```Java
// 翻译后
	  day44.servlet.bean.User user = null;
      user = (day44.servlet.bean.User) _jspx_page_context.getAttribute("user", javax.servlet.jsp.PageContext.PAGE_SCOPE);
      if (user == null){
        user = new day44.servlet.bean.User();
        _jspx_page_context.setAttribute("user", user, javax.servlet.jsp.PageContext.PAGE_SCOPE);
      }
```

#### `<jsp: setProperty>`

* 设置 JavaBean 对象的属性，调用 set 方法
* `<jsp:setProperty name="user" property="username" value="admin">`
  * `name`  JavaBean 对象的引用名，即上面的 id
  * `property`  属性名
  * `value`  属性值或 JSP 表达式
    * 属性值：接收字符串，自动转化为相应类型
    * JSP 表达式：表达式返回结果类型要与属性值类型一致
* `<jsp:setProperty name="user" property="password" param="password">`
  * `param`  接收请求参数值，不要求成员变量名要与请求参数名一致
* `<jsp:setProperty name="user" property="*">`
  * `property="*"`  设置 JavaBean 中所有符合请求参数名的成员变量值
  * **JavaBean 类的成员变量名要与请求参数名一致，才会赋值**

```java
// 翻译后
org.apache.jasper.runtime.JspRuntimeLibrary.introspecthelper(_jspx_page_context.findAttribute("user"), "username", "admin", null, null, false);
```

#### `<jsp: getProperty>`

* 读取 JavaBean 对象属性，调用 get 方法，直接写入到 out 流中
* `<jsp:getProperty name="user" property="username"/>`
  * `name`  JavaBean 对象的引用名，即上面的 id
  * `property`  属性名
* 如果某个属性值为 `null`，该标签返回 `"null"` 字符串

```java
// 翻译后
out.write(org.apache.jasper.runtime.JspRuntimeLibrary.toString((((day44.servlet.bean.User)_jspx_page_context.findAttribute("user")).getUsername())));
```

## EL

#### 概述

* Expression language
* 依赖于 JSP，不是一门单独的语言
* 主要功能
  * 获取数据
    * 用于替换 JSP 页面的脚本表达式，更方便使用 JSP
  * 执行运算
    * 关系、逻辑、算术运算
  * 获取 Web 开发常用对象
    * 获取隐式对象
  * 调用 Java 方法
    * 允许使用自定义 EL 函数
* 返回字符串，直接输出到浏览器

```Java
// 翻译后 EL 引擎翻译 转化为字符串 再写入到 out 
out.write((java.lang.String) org.apache.jasper.runtime.PageContextImpl.proprietaryEvaluate("${\"Hello world\"}", java.lang.String.class, (javax.servlet.jsp.PageContext)_jspx_page_context, null));

      out.write((java.lang.String) org.apache.jasper.runtime.PageContextImpl.proprietaryEvaluate("${user}", java.lang.String.class, (javax.servlet.jsp.PageContext)_jspx_page_context, null));
```

#### 获取数据

* `${标识符}`
* **！！！数据必须是在四大域对象中！！！**
* 原理：
  * 翻译后 `pageContext.findAttribute(String attributeName)` 
  * 从四大域中查找，**找不到则 EL 表达式返回 `""` 空串**
  * **当在 EL 表达式内部查找对象时，如果对象不存在，返回 `null`**
  * 如 `${user}` 
  * **注意标识符**， `${"user"}`  加了引号，会直接输出 字符串
* 获取 JavaBean 属性，数组，**Collection，Map**类型集合的数据
* 前提是，**集合对象必须在域对象中**，理由见上翻译后，如
  * `${user.username}`  实际是**调用 get 方法**
  * `${list[0]}`  返回有序集合的某个下标所对应元素，也能调用集合的方法 `${list.get(3)}`
  * `${map.key}`  返回 Map 中某个 key 映射的值

#### 执行运算

* `${运算表达式}`
* 关系运算符
  * `==`  eq
  * `!=`  ne
  * `<` lt `>` gt `<=` le `>=` ge
* 逻辑运算符
  * && 或 and
  * || 或 or
  * ! 或 not
* 常用运算符
  * empty
    * 检查对象是否为 null 或 ""
    * `${empty user}`
  * 三目运算符
    * `${user != null ? user.name : ""}`
  * `[]`  访问集合，返回有序集合的某个下标所对应元素
  * `.`  
    * 访问 JavaBean 对象成员，调用 get 方法
    * 返回 Map 的 key 所映射的 value 值

#### 获取 Web 开发常用对象

* 11 个隐式对象，`${隐式对象名}`，返回对象的引用
* pageContext
* pageScope
  * page 域中保存属性的 Map 对象
* requestScope
  * request 域中保存属性的 Map 对象
* sessionScope
  * session 域中保存属性的 Map 对象
* applicationScope
  * application 域中保存属性的 Map 对象
* param
  * 所有请求参数的 Map 对象 (String, String)
* paramValues
  * 所有请求参数的 Map 对象 (String, String[])
  * 对于某个参数值不止一个的，返回 String[] 数组
  * `request.getParamterMap()`
* header
  * 所有请求头字段的 Map 对象 (String, String)
* headerValues
  * 所有请求头字段的 Map 对象 (String, String[])
  * 对于某个请求头字段的值不止一个的，返回 String[] 数组
  * 如果头字段有 `-` ，如 `Accept-Encoding` ，要用字符串当作标识符
  * 如：`${headerValues["Accept-Encoding"]}`
* cookie
  * 所有 cookie 的 Map 对象
  * 如 `${cookie.myCookie}` 返回的是 Cookie 对象
  * 访问属性要 `${cookie.myCookie.name}`  `${cookie.myCookie.value}`
* initParam
  * 所有 Web app 初始化参数的 Map 对象

#### 调用 Java 方法

* 允许开发人员自定义 Java 方法，调用 `${prefix: method(param)}`
* 只能调用 Java 类的静态方法，该方法需要在 TLD (*Tag Library Descriptor*) 文件中描述
* 开发步骤
  1. 导库 `javax.servlet.jsp.jstl.jar`  `javax.servlet.jsp.jstl.api.jar`
  2. 编写一个 Java 类静态方法
  3. 编写标签库描述符（TLD）文件，描述自定义方法
     * 放到 WEB-INF 目录中除 classes lib 目录或子目录下
  4. JSP  taglib 指令导入 `<%@ taglib prefix="tool" uri="http://www.sample.com" %>`
  5. 使用

```Java
// Tool.java 
// 用 String 参数类型，在 JSP 中可以直接传入，不用 "" 包裹
public class Tool {
    public static int add(String x, String y) {
        return Integer.parseInt(x) + Integer.parseInt(y);
    }
}
```

```xml
<!-- tool.tld -->
<!-- IDEA => new => XML Configuration File => JSP Tag Library Descriptor -->
<?xml version="1.0" encoding="ISO-8859-1"?>

<taglib xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 		http://java.sun.com/xml/ns/javaee/web-jsptaglibrary_2_1.xsd"
        version="2.1">

    <tlib-version>1.0</tlib-version>
    <short-name>tool</short-name>
    <!-- uri 指定使用 taglib 指定时引入路径  -->
    <uri>http://www.sample.com</uri>


    <!-- Invoke 'Generate' action to add tags or functions -->
    <function>
        <!-- EL 方法名 -->
        <name>add</name>
        <!-- 完整 Java 类名 -->
        <function-class>day44.servlet.demo.Tool</function-class>
        <!-- 方法签名，包括返回值类型 -->
        <function-signature>
            int add(java.lang.String,java.lang.String)
        </function-signature>
    </function>
</taglib>
```

```jsp
<!-- text.jsp -->
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="tool" uri="http://www.sample.com" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${tool: add(88, 20)}
</body>
</html>
```

#### EL 注意事项

* Tomcat 不能使用 EL 表达式解决方法
  * 升级 Tomcat，支持 Servlet 2.4 ／ JSP 2.0 以上版本
  * 在 JSP 中加入 `<% @page isELIgnored="false" %>`

## JSTL

#### 概述

* JSTL： JSP Standard Tag Library
* 与 EL 结合，取代 Java 脚本片段
* 功能：
  * 标准通用标签函数库
  * 提高 JSP 可读性，可维护性，方便性
* Tomcat，需要支持 Servlet 2.4 ／ JSP 2.0 以上版本
* `<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>` 引入

#### 标签库

* **核心标签库（core）  c:**
* XML（操作 XML 标签库）x: 
* SQL（标签库）sql: 
* FMT（国际化标准库）fmt: 
* JSTL 函数（EL 函数）el:

#### 笔记声明

* `${el}` 支持 EL 表达式语言
* `String`  `boolean` 不支持表达式语言，声明类型

#### `<c:out>`

* `<c:out value="Object ${el}" escapeXml="boolean ${el}" default="Object ${el}">`
* value：指定输出内容  
* escapeXml: 是否将 Xml 语法特殊字符进行转义，默认为 true  
* default：当 value 值为 null 或者 "" 时，输出的默认值
* `<c:out value="${null}" default="Hello default."/>`

#### `<c:set>`

* 用法一
  * `<c:set value="Object ${el}" var="String" scope="String">`
  * var：指定要设置的域属性名称
  * scope：指定属性所在域对象 `(page, request, session, application)`
* 用法二
  * `<c:set value="Object ${el}" target="Object ${el}" property="String ${el}">`
  * target：指定要设置属性的对象 `(JavaBean, Map)`
  * property：指定对象要设置的属性名


* value：指定属性值

####`<c:remove>`

* `<c:remove var="String" scope="String">`
* var：指定要删除的域属性名称
* scope：指定删除哪一个域对象中的域属性，默认为 page `(page, request, session, application)`

#### `<c:catch>`

* `<c:catch var="String"> 异常代码 </c:catch>`
* var：捕获到的异常对象引用名，捕获后保存在 page 域中
  * `${e.message}` 类似这样操作异常对象

####`<c:if>`

构造简单的 if - then 条件结构

* `<c:if test="boolean ${el}" var="String" scope="String">`
* test：条件表达式
* var：指定将 test 执行结果保存为的域属性名称
* scope：指定保存属性所在域对象 `(page, request, session, application)`

#### `<c:choose>`

构造 if else-if else 条件结构

* `<c:choose>` `<c:when>` `<c:otherwise>`  三个标签一起使用

```xml
<c:choose>
	<c:when test="boolean ${el}">
  	</c:when>
  	<c:otherwise test="boolean ${el}">
  	</c:otherwise>
</c:choose>
```

#### `<c:forEach>`

* `<c:forEach items="Object ${el}" var="String" varstatus="String">`
* items：要遍历的集合对象
* var：当前遍历到的元素保存到 page 域属性名，可以通过 `${String}` 方便访问
* varstatus：记录当前遍历到的元素的属性，保存到 page 域属性
  * int index：当前元素在集合中下标
  * int count：已遍历元素个数，即元素计数器
  * boolean first：返回是否是第一个元素
  * boolean last：返回是否是最后一个元素
* `begin="int ${el}"`  从第 begin 个元素开始遍历，默认为 0
* `end="int ${el}"`  遍历到第 end 个元素，默认为 元素个数
* **开区间：`[begin, end]`**
* `step="int ${el}"` 指定遍历步长，默认为 1，即 i++

