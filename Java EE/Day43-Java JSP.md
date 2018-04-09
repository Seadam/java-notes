# Java JSP

#### 内容摘要

Servlet 补充，概述，JSP 访问原理、最佳实践、模板元素、表达式、脚本片段、声明、注释、指令、九大隐式对象， out， pageContext，四大域对象，`<base>`

---

#### Servlet 补充

* Servlet 线程安全问题

  * 单实例多线程
  * 多次请求同一个 Servlet 时，Tomcat 会创建多个线程调用 `service` 方法
  * 此时，**多个线程间共享同一个 Servlet 成员变量**，所以会出现成员变量覆盖问题

* servlet 初始化参数

  * 在单个 Servlet 中通过注解配置的 `@WebInitparam` 初始化参数，通过 `getInitParameter` 在` init()` 方法拿到，作用范围仅在当前 Servlet
  * 配置全局初始化参数，需要在 `Web.xml` 中，配置 `<Context-param>`，通过 `getServletContext()` 拿到 Context ，在拿 `getInitParameter` ，作用范围所有 Servelt

  ```xml
  <!-- 单个 Servlet 作用域 -->
  <!-- 也可以通过注解配置 -->
  <Servlet>
    <init-param>
      <param-name>name</param-name>
      <param-value>value</param-value>
    </init-param>
  </Servlet>

  <!-- 全局 ServletContext 作用域 -->
  <!-- 只能通过 Web.xml 配置 -->
  <Context-param>
  	<param-name>name</param-name>
      <param-value>value</param-value>
  </Context-param>
  ```

#### 概述

* **JSP：*Java Server Pages*， 实际上就是 Servlet**

* 允许在 HTML 页面里面嵌套 Java 代码，提供动态数据

* JSP 只允许 POST GET 或 HEAD 方式提交的请求

  > JSPs only permit POST GET or HEAD

#### JSP 访问原理

1. URL 找到 JSP 页面
2. Tomcat 生成 JSP 对应的 Java 文件 和 class 文件
   * Tomcat 实例中 work 目录下 Web app 目录下能找到
   * `/Users/clixin/Library/Caches/IntelliJIdea2017.3/tomcat/Tomcat_9_0_4_JavaWeb`
3. 加载 class 文件，调用 JSP 的 `_jspService` 方法
4. 把 HTML 代码通过 out 流（字符流）打印到浏览器，并把动态结果写回 浏览器

#### JSP 最佳实践

* **Servlet：Web app 中 控制器组件，负责响应请求数据，把数据转发给 JSP**
* **JSP：Web app 中 数据显示模板，接收转发数据，动态渲染界面**

#### JSP 模版元素

* HTML 代码

#### JSP 表达式

* 单行，**没有分号**，将程序数据输出到浏览器
* 如 拿到时间 `<%= new java.util.Date() %>`
* JSP 引擎在翻译脚本表达式时，将程序数据**转化为字符串**，再在相应位置使用 `out.print` 输出到浏览器

```jsp
<%= 表达式或变量%>
```

#### JSP 脚本片段

* 多行 Java 代码，可以有多个脚本片段，可以互相访问
* **单个脚本片段的 Java 代码可以是不完整的，但所有组合后的 Java 代码必须是完整的**
* 只能写 Java 代码，**不能出现其他模版元素，遵循 Java 语法，必须有分号**
* **JSP 引擎在翻译脚本片段时，将 Java 代码原封不动的放到 Servlet 的 `_jspService` 中**

```jsp
<%
	多行 Java 代码
%>
```

#### JSP 声明

* **JSP 引擎在翻译声明时，将 Java 代码放到 Servlet 的 `_jspService` 外面**
* **单个声明的 Java 代码可以是不完整的，但所有组合后的 Java 代码必须是完整的**
* 用于定义 JSP 转为 Servlet 的静态代码块、成员变量、成员方法
* **JSP 隐式对象作用范围为 `_jspService` 方法内，在 JSP 声明中不能使用**

```jsp
<%!
 	多行 Java 声明代码
%>
<!-- 比如 -->
<%! 
    static  {
      System.out.println("Loading Servlet.")
    }
	private int id = 0;
    public void jspInit() {
		System.out.println("JSP init()")
    }
%>
<%! 
    public void jspDestroy() {
		System.out.println("JSP destroy()")
    }
%>
```

#### JSP 注释

* **JSP 引擎在翻译注释时，忽略注释内容**

```jsp
<%-- JSP 注释：最终会被翻译忽略，不能在 JSP 脚本片段中使用 --%>
<!-- HTML 注释：最终会被翻译写回到相应报文 -->
<% // Java 注释：最终会被翻译到 Servlet Java 源码中 %>
```

#### JSP 指令

* **为 JSP 引擎设计，不直接产生输出**，告诉引擎如何处理 JSP 页面

* `<%@ 指令 属性名="属性值" %>`

* 一个指令可以有多个属性，多个属性可以写到一条指令内

* 如：`<%@ page contentType="text/html;charset=utf-8" %>`

* page

  * 定义 JSP 页面属性，作用范围整个页面，可以放在任意位置，**最好放在整个页面起始位置**

  ```jsp
  <%-- 常见 page 指令属性 --%>
  <%@ page 
  	[ language="java" ] // 支持语言，仅 Java
  	[ extends="package.class" ] // Servlet 继承
  	[ import="{package.class | package.*}, ..." ] // 导包
  	[ session=“true | false” ] // true 创建 Session 对象 （默认为 true）
  	[ buffer="none | 8kb | sizekb" ] 
    	// JSP Writer 缓存大小，默认 8192 字节（8kb），满了就自动刷新到 response.getWriter()
  	[ autoFlush="true | false" ] 
    	// 默认 true，当缓存区满了后，自动清空缓存，写入到 response.getWriter()
    	// false ：不自动清空缓存，会出现缓存溢出异常
  	[ info="text" ] // 生成一个方法 getServletInfo() 获取服务器描述信息
  	[ errorPage="relative_url" ] // 当抛出异常时 500，跳转到异常友好提示界面
  	[ isErrorPage="true | false" ] 
    	// true 翻译后的 Java 文件可以拿到抛出的异常对象， JSP 中能使用该隐式对象 exception
    	// 通常用于 记录异常 日志
  	[ contentType="mimeType;text/html;charset=utf-8" ]
    	// 指定 response.setContentType
  	[ pageEncoding="charset | ISO-8859-1" ] 
    	// 指定 JSP 文件编码格式
    	// 指定 响应报文 charset 即 contenType="text/html;charset=utf-8"
    	// 与 contentType 同时存在时，以 contentType 为准
  %>
  ```

  ```Java
  	// 通过 page 指令配置 JSP PageContext
  	// [ session=“true | false” ] 
  	// [ buffer="none | 8kb | sizekb" ] 
  	// [ autoFlush="true | false" ] 
  	// [ errorPage="relative_url" ] 

  	/* @param servlet      the requesting servlet
       * @param request      the current request pending on the servlet
       * @param response     the current response pending on the servlet
       * @param errorPageURL the URL of the error page for the requesting JSP, or
       *                         null
       * @param needsSession true if the JSP participates in a session
       * @param buffer       size of buffer in bytes, {@link JspWriter#NO_BUFFER}
       *                         if no buffer, {@link JspWriter#DEFAULT_BUFFER}
       *                         if implementation default.
       * @param autoflush    should the buffer autoflush to the output stream on
       *                         buffer overflow, or throw an IOException?
       *
       * @return the page context
       *
       * @see javax.servlet.jsp.PageContext
       */
      public abstract PageContext getPageContext(Servlet servlet,
              ServletRequest request, ServletResponse response,
              String errorPageURL, boolean needsSession, int buffer,
              boolean autoflush);

  	// 翻译后的 jsp.java
  	pageContext = _jspxFactory.getPageContext(this, request, response,
        			null, true, 8192, true);

  	// [ isErrorPage="true" ] 翻译后 jsp.java 此处省略了包名
  	Throwable exception = JspRuntimeLibrary.getThrowable(request);
      if (exception != null) {
        response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
      }
  	
  ```

* include

  * 引入其他 **JSP 页面**
  * **静态引入**：如果引入多个 JSP 页面，**JSP 引擎会将多个 JSP 页面翻译成一个 Servlet**
  * 静态引入：源码层面引入 JSP 页面
  * `<%@ include file="组件的 URL" %>`
    * 路径：以 `/` 相对于当前应用，即引用范围只能引用本应用内
  * 如 header ，footer，include 引用即可
  * 注意事项
    * 被引入文件必须遵循 JSP 语法
    * 被引入的文件可以是**任意扩展名**，即使是 `.html`，JSP 引擎会按照处理 JSP 方式处理
      * 可以在 HTML 页面写 JSP 代码
    * **JSP 规范建议静态引入文件扩展名** `.jspf`  （*JSP fragments*）
    * 由于会翻译为一个 Servlet，多个引用 JSP 页面指令不能冲突（除 pageEncoding 和 impor）

* taglib

  * 引入标签库文件，详细见 Day44 JSTL

#### JSP 九大隐式对象

* 隐式对象：由 JSP 引擎创建，可以使用对象的引用，**局部变量**
* **作用范围仅限于 `_jspServise()` 方法中，隐式对象在 JSP 声明中 `<%! %>` 不能使用**


* `request: HttpServletRequest`
* `response: HttpServletResponse`
* `config:  ServletConfig`
* `application :ServletContext`
* `exception: Throwable`
* `session: HttpSession`
* `page: this Object`
* `out: JspWriter`
* `pageContext`

```Java
public void _jspService(final javax.servlet.http.HttpServletRequest request, final javax.servlet.http.HttpServletResponse response)
      throws java.io.IOException, javax.servlet.ServletException {

  	java.lang.Throwable exception = org.apache.jasper.runtime.JspRuntimeLibrary.getThrowable(request);
	final javax.servlet.jsp.PageContext pageContext;
    javax.servlet.http.HttpSession session = null;
    final javax.servlet.ServletContext application;
    final javax.servlet.ServletConfig config;
    javax.servlet.jsp.JspWriter out = null;
    final java.lang.Object page = this;
	

    try {
      pageContext = _jspxFactory.getPageContext(this, request, response,
      			null, true, 8192, true);
      application = pageContext.getServletContext();
      config = pageContext.getServletConfig();
      session = pageContext.getSession();
      out = pageContext.getOut();
    }
```

#### out 

* 用于向客户端发送文本数据，**字符流，所以在 JSP 中不能使用字节流**
* `pageContext.getOut()`    `JspWriter`
* 维护一个缓存，通过 page 指令的 `buffer` 、 `autoFlush` 指令配置缓存
* 只有当满足一下条件，out 才会写入到 `ServletResponse.getWriter()` 流
  * `<%@ page buffer="0" %>`  缓存设为 0，每次直接写入不缓存
  * out 的缓存已满，会 autoFlush 写入，若 `autoFlush="false"`  ，缓存满后再写入会溢出异常
  * 整个 JSP 页面结束

#### pageContext

* JSP 页面上下文环境，封装其他 8 个隐式对象的引用
  * `pageContext.getServletContext()`
  * `pageContext.getException()`
  * `pageContext.getPage()`
  * `pageContext.getRequest()`
  * `pageContext.getResponse()`
  * `pageContext.getServletConfig()`
  * `pageContext.getSession()`
  * `pageContext.getOut()`
* **域对象，作用范围当前页面**
* 封装常用操作
  * 简化 `RequestDispatcher.forward()`  `include()`
  * `pageContext.forward(String relativUrlPath)`
  * `pageContext.include(String relativUrlPath)`
  * 此处路径同 `request.getRequestDispatcher(String relativUrlPath)`  `/`  相对于当前应用
* 检索其他域对象的属性 API
  * 自身域对象属性
    * `setAttribute(String name, Object value)`
    * `getAttribute(String name)`
    * `removeAttribute(String name)`
  * 其他域对象属性
    * `getAttribute(String name,int scope)`
    * `setAttribute(String name, Object value,int scope)`
    * `removeAttribute(String name,int scope)`
  * scope 常量
    * `PageContext.APPLICATION_SCOPE`  **ServletContext**
    * `PageContext.SESSION_SCOPE`  **HttpSession**
    * `PageContext.REQUEST_SCOPE`  **HttpRequest**
    * `PageContext.PAGE_SCOPE`  **PageContext ** 
  * `findAttribute(String name)`
    * 从小往大找：`PAGE_SCOPE` => `REQUEST_SCOPE` => `SESSION_SCOPE` => `APPLICATION_SCOPE`

#### 四大域对象

作用范围，大小之分

* **ServletContext** 整个 Web app
* **HttpSession** 一次会话过程
* **HttpRequest** 一次请求过程
* **PageContext ** 一个 JSP 页面

#### `<Base>`

* `<base href="">`  改变当前 HTML 页面的相对路径，默认值为 **basePath**
* `String path = request.getContextPath()`  Web app 入口
* `String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request .getServerPort() + request.getContextPath() + "/"`
  * `request.getScheme()`  获得协议名 http
  * `request.getServerName()`  获得服务器名 localhost
  * `request .getServerPort()`  获得服务器端口 8080
  * `request.getContextPath()`  获得 Web app 入口 /myApp
  * `<Base>`  默认值为 **basePath**









