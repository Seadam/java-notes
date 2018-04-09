# Java Servlet Response

#### 内容摘要

复习，RequestDispatcher，request 转发 包含，转发处理流程，转发特点，包含处理流程，包含特点，转发与包含的区别，请求范围，小结，Response，HttpServletResponse API，Response 应用，乱码问题，重定向，重定向流程，重定向特点，Response，路径总结

---

#### 复习

* 乱码问题
  * POST，GET，**依赖网页编码**，例如  `<meta charset="gbk">` 
  * POST 通过 `request.setCharacterEncoding("gbk")`   解决
    * 指定 Tomcat 对请求正文的请求方式
  * GET 通过 Tomcat 服务器 `server.xml` 中添加属性 `<Connector URIEncoding="GBK">`

  ``` XML
  <Connector port="80" protocol="HTTP/1.1"
                 connectionTimeout="20000"
                 redirectPort="8443"
                 URIEncoding="GBK"/>
  ```

* Tomcat 实例问题
  * IDEA 会开启新一个 Tomcat 服务器实例来运行 Web app
  * IDEA 中不勾选 `deploy applications configurations in tomcat instance`
    * 无法访问到 Tomcat 的默认 ROOT 、manager、example 应用
    * 独立的配置文件
    * 修改 Tomcat 编码，仅需要改变实例编码
  * IDEA 中勾选 `deploy applications configurations in tomcat instance`
    * 可以访问到 Tomcat 的默认 ROOT 、manager、example 应用
    * 会采用 Tomcat 本体的配置文件，每次运行服务器是会拷贝本体配置运行
    * 修改 Tomcat 编码，需要改变本体编码

* 路径问题（见 day39）
  * action 可以提交给任意服务器的任意应用
  * RequestDispatcher 转发包含 当前应用

* ServletRequest 对象是一个域对象，同 ServletContext
  * ServletContext：可以存数据，也可以拿数据，**作用域是整个 Web app**
  * ServletRequest：可以存数据，也可以拿数据，**作用域是同一个请求**

#### RequestDispatcher

* 请求分发器
* `RequestDispatcher requestDispatcher = request.getRequestDispatcher()`
* `RequestDispatcher(String pathname)` ，pathname 见下
* 方法
  * `public void forward(ServletRequest request,ServletResponse response)`
    * 把请求转发给**目标组件 （Servlet、JSP、HTML）**
  * `public void include(ServletRequest request,ServletResponse response)`
    * 包含目标组件的响应结果 （Servlet、JSP、HTML）
* 拿到 RequestDispatcher 对象
  * `ServletContext`  对象 调用 `RequestDispather(String path1)`
    * path1 必须以 `/` 开头，不写会抛出异常 `IllegalArgumentException`
  * `ServletRequest`  对象 调用 `RequestDispather(String path2)`
    * path2 可以用绝对路径，也可以用相对路径

#### request 转发包含

* Servlet 规范实现多个 Servlet 通信
  * 转发：Servlet（源组件）对客户端请求预处理操作，再把请求转发给其他 Web 组件（目标组件）
  * 包含：Servlet （源组件）把其他 Web 组件（目标组件）生成的**响应结果包含到自身响应结果**中
* 转发与包含的共同点
  * 源组件和目标组件处理的都是同一个客户请求，共享一个 `ServletRequest`  `ServletResponse`  对象
  * 目标组件可以是：`Servlet` / `JSP` / `HTML`
  * 依赖 `javax.servlet.RequestDispatcher` 接口

#### 转发处理流程

1. **清空响应用于存放响应正文的缓存区**
2. 如果是 Servlet、JSP，Tomcat 直接调用，响应结果发送给客户端
3. 如果是 HTML，Tomcat 读取**文档中的内容**，发送给客户端

#### 转发特点

* **源组件的生成的响应结果，无论转发前后，不会发送到客户端，只有目标组件的响应结果发送到客户端**
* 源组件在转发前，提交了响应结果（即调用 response 流的  `flush()` `close()` 方法），再 `forward()` ，会抛出异常 `IllegalStateException`

#### 包含处理流程

1. 如果是 Servlet、JSP，Tomcat 直接调用，响应结果添加到源组件响应结果

2. 如果是 HTML，Tomcat 读取**文档中的内容**，添加到源组件响应结果

#### 包含特点

* 源组件与目标组件的输出数据都会添加到响应结果中
* 在目标组件中对**响应状态代码或相应头**所做的修改都会被忽略
* 源组件包含目标组件后，再向响应正文写入响应内容，会添加到响应结果中

#### 转发与包含区别

* **最终响应结果是否包含源组件响应结果** forward（不包含） include（包含）
* 不管是转发还是包含，其后的代码都会执行到

#### 请求范围

* **Web 应用范围内**的共享数据作为 `ServeltContext` 对象的属性而存在（setAttribute）
  * 只要共享 ServletContext 对象也就共享了其数据
  * 域对象
* **同一个请求范围内**的共享数据作为 `ServletRequest` 对象的属性而存在（setAttribute）
  * 只要共享 ServletRequest 对象也就共享了其数据
  * 域对象


#### 小结

* 从请求中拿到响应信息
  * 请求行： `getMethod()` `getUri()` `getProtocol()`
  * 请求头： `getHeader(String name)`  键值对
  * 请求参数／请求正文／表单数据：`getParameter(String key)` `getParametervalues(String key)`
* 乱码问题
  * POST
  * GET
* 路径问题
  * action 路径： 绝对路径，以 / 开头相对路径，不以 / 开头相对路径
  * dispatcher 路径： 以 / 开头相对路径，不以 / 开头相对路径
* 转发包含
  * 目标组件：Servlet、JSP、HTML
  * 范围：只限于当前应用
  * 转发器
    * `request.getRequestDispatcher()`
    * `foward()`
    * `include()`
    * 区别：**最终响应结果是否包含源组件响应结果**

## Response

#### 简介

Servlet 容器会收到客户端 HTTP 请求，针对**每一次**请求一个创建 request 对象，和空的 response 对象

#### HttpServletResponse API

* `setStatus(int sc)`
  * 状态码，响应行
* `setHeader(String name, String value)`
  * 响应头，键值对
* `PrintWriter getWriter()`
  * 字符流，响应数据
* `ServletOutputStream getOutputStream()`
  * 字节流，响应数据

#### response 应用

* 向客户端输入中文数据
  * `response.setContentType("text/html;charset=UTF-8")`
* 发送响应头，控制浏览器定时刷新网页
  * 随机数
    * 通过 HTTP 协议告诉浏览器，以 2 秒的频率刷新页面（重新发送请求，获取相应）
    * `response.setHeader("Refresh", "2")`  单位 秒
  * 注册响应
    * `response.setHeader("Refresh", "2;URL=www.baidu.com")`  URL 可以省略
      * 路径：可以是绝对路径，可以是相对路径
      * 相对路径：以 `/` 开头，相对于 Tomcat 服务器，不以 `/` 开头，相对于父目录
      * `setHeader("Refresh", "2;/response/login.html")`
      * **小技巧：根据请求提交范围，提交对象**
        * 任意服务器任意应用：相对于 Tomcat 服务器根路径
        * 当前应用：相对于 当前 Web app 根路径
* 发送响应头，控制浏览器缓存当前文档内容
  * `response.setDateHeader("Expires", System.currentTimeMillis() + 60 * 1000)`
  * 从 Epoch 开始，毫秒数，显示的时间是国际时间，东八区需要再加 8 小时
* 请求重定向，见下

#### 乱码问题

* `getCharsetEncoding()`  获取响应正文编码字符集，若不指定 Tomcat 默认为  `ISO-8859-1`

* 方式一（字符流）

  * `setCharacterEncoding(String charset)`  指定相应正文**编码字符集**
  * 还需要看浏览器的**解码字符集**
  * 例如：`setCharacterEncoding("UTF-8")` ，浏览器解码字符集为 `GBK`，会出现乱码

* 方式二（字符流）

  * `response.setContentType("text/html;charset=UTF-8")`  `;` 不可省略
  * 指明响应正文编码方式以及浏览器解码方式
  * `response.setHeader("Content-Type", "text/html;charset=UTF-8")`   

* 方式三（字节流）

  * `response.getOutputStream.write("中文".getBytes())`  将字符串转化为字节流需要进行编码
  * 默认 JVM 字符串编码为 `UTF-8` ，`Charset.defaultCharset()`
  * 浏览器收到字节数据后，需要解码，默认解码方式为 `GBK` ，出现乱码
  * 解法一
    * 字节流编码方式制定同浏览器 `response.getOutputStream.write("中文".getBytes("GBK"))`
  * 解法二
    * `response.setContentType("text/html;charset=UTF-8")`
    * 指明响应正文编码方式以及浏览器解码方式

* 方式四

  * `write("<meta charset=utf-8/>"` 指定浏览器解码方式
  * 前提是指定的编码方式相同，`setCharacterEncoding("UTF-8")`

  ```Java
  		response.setCharacterEncoding("utf-8");
          PrintWriter writer = response.getWriter();
          writer.write("<html lang=\"en\">\n" +
                  "<head>\n" +
                  "    <meta charset=\"utf-8\">\n" +
                  "    <title>Document</title>\n" +
                  "</head>\n" +
                  "<body>\n" +
                  "    <h1>你好</h1>\n" +
                  "</body>\n" +
                  "</html>");
  ```

* **编解码一致**

  * 编码
    * `response.setCharacterEncoding("UTF-8")`
    * `response.getOutputStream.write("中文".getBytes("GBK"))`
    * `response.setContentType("text/html;charset=UTF-8")`  一条代码解决
  * 解码：
    * `response.setContentType("text/html;charset=UTF-8")`  一条代码解决
    * `write("<meta charset=utf-8/>"` 

#### 重定向 sendRedirect()

* 概念：一个 Web 资源收到浏览器请求之后，**通知浏览器**去访问另一个 Web 资源
* 特征：地址栏为会变，会请求两次，增加服务器负担，重定向到本服务器时，建议使用转发或包含
* 实现：`response.sendRedirect(String location)` 
* 原理
  * 模拟 重定向
  * 状态码：302 ／ 307  `response.setStatus(302)`
  * 响应头：Location  `response.setHeader("Location", "www.baidu.com")`

#### 重定向与 Expires

* Expires ：直接跳转到目标网页，不用浏览器重新发送请求
* 重定向：向浏览器指明下一个请求目标，浏览器重新构建请求，重新向目标发出 HTTP 请求

#### 重定向流程

1. 在浏览器输入 URL，访问服务端某个 Web 组件
2. 该组件返回状态码为 302 的响应结果
3. 浏览器接收到响应结果，立即自动请求另一个 Web 组件
4. 浏览器接收到目标 Web 组件的响应结果

#### 重定向特点 

* Servlet 源组件的响应结果不会发送到浏览器
* `response.sendRedirect(String location)`  一律返回状态码 302 的响应结果
* 重定向前，提交了响应结果（即调用 response 流的  `flush()` `close()` 方法），调用 `sendRedirect()` ，会抛出异常 `IllegalStateException`
  * 模拟的重定向，不会抛异常，也不会跳转到重定向网页
* 源组件 `sendRedirect()` 后面的代码会执行
* **源组件和目标组件不共享同一个 `ServletRequest`**
* `response.sendRedirect(String location)` 路径问题，同任意服务器任意应用

#### Response 流

* `getWriter()`  字符流
* `getOutputStream()`  字节流
* **相互排斥**，同一个 response 对象，只能使用一种流，否则会抛异常 `IllegalStateException`
* Servlet 引擎自动调用 `close()` 关闭流

#### 路径总结

* 表单提交，超链接，重定向，refresh
  * 绝对路径
  * 以 `/` 开头相对路径：相对于 Tomcat 服务器
  * 不以 `/` 开头相对路径：相对于 父目录
* servlet 转发以及包含，context.getRealPath()
  * 以 `/` 开头相对路径：相对于当前 Web app
  * 不以 `/` 开头相对路径：相对于 父目录
* **小技巧：根据请求提交范围，提交对象**
  * 任意服务器任意应用：相对于 Tomcat 服务器根路径
  * 当前应用：相对于 当前 Web app 根路径
  * 不以 `/` 开头，相对于父目录









