# Java Servlet Request

#### 内容摘要

Setvlet 补充，简介，HttpServletRequest API，乱码问题，表单提交，BeanUtils，request 请求转发，RequestDispatcher，路径，表单 Action 路径

---

**@WebServlet  value 与 urlPartterns  等价**

#### Setvlet 补充

* URL 匹配顺序 如 `http://localhost:8080/firstapp/myservlet`
  1. Tomcat 根据 URI 请求资源路径  `/firstapp/myservlet`
  2. Tomcat 在 `Service.xml` 找 `<Context url>` （即 Web app 名）入口 `firstapp`
  3. Tomcat 进入 Web app 在 `web.xml` 在 `<servlet-url>` 找到  `/myservlet`  映射的 Servlet 名
  4. 通过 Servlet 名找到 classes 源文件
  5. 注：在 IDEA 中使用的是注解来配置
     * urlPartterns ，name
* URL 匹配顺序
  * 若设置 `value = “/”`  Servlet，就相当于当前 Web app 中默认 Servlet 
    * 当其他 url 都不匹配时，匹配默认
    * 在配置了 Web app 默认 Servlet 前提下，访问 Tomcat 中不存在的应用时， Tomcat 的默认 Servlet 会接受并处理
  * `/*`  优先级比  `/`  高
* 真实路径
  * （真实路径）运行路径：资源和 classes 运行在 Tomcat 服务器上
  * 源码路径：本地编写源码路径
  * `WEB-INF`  内的文件，外部不能访问，内部可以通过流来访问，实现控制访问
    * 此处**文件路径需要填写真实运行路径**
  * 拿到真实路径
    * 假设 `pic.img` 在 `WEB-INF` 根目录下
    * `getContext().getRealPath("pic.img")`   
    * `MyServlet.class.getClassLoader().getResourceAsStream("pic.img")`
    * 返回该文件在 war 包中真实路径
* ServletContext
  * 作用域
    * 所有Servlet 共享同一个 **ServletContext** 对象
  * 生命周期
    * Servlet 不用时卸载
    * ServletContext 是 Web app 创建时创建，销毁时销毁，等同于整个 Web app
    * 可以通过 Tomcat 管理工具手动结束一个 Web app，ServletContext

#### 简介

Servlet 容器会收到客户端 HTTP 请求，针对**每一次**请求一个创建 request 对象，和空的 response 对象

#### HttpServletRequest API

* `getRequestURL` 返回客户端发出**请求的完整 URL**  `http://localhost:8080/firstapp/myservlet`

* `getRequestURI` 返回请求行中的**资源路径**  `/firstapp/myservlet`

  * **`URI` : Universal Resource Identification  统一资源定位，资源在服务器上的路径**
  * **`URL`: Universal Resource Location**
    * **`http` ：协议**
    * **`localhost:8080`  ：主机、端口**
    * **`/firstapp/myservlet` ： 资源路径，即 URI**
  * **二者均不包括请求参数** 
    * 例如：`http://localhost:8080/firstapp/login.html?name=Tony`
    * URL : `http://localhost:8080/firstapp/login.html`
    * URI :  `/firstapp/login.html`

* `getQueryString` 返回请求行中的参数部分

* `getRemoteAddr` 返回发出请求的客户端的IP地址

* `getRemoteHost` 返回发出请求的客户端的完整主机名

* `getRemotePort` 返回客户机所使用的网络端口号

* `getLocalAddr` 返回WEB服务器的IP地址

* `getLocalName` 返回WEB服务器的主机名

* `getMethod` 得到客户端请求方式

* `setCharacterEncoding`

  * 设置**请求正文**的解码方式，必须在获取数据前设置

* 获取请求头
  * `getHeader(name)` 
    * GET POST 方法提交的都能拿到
    * 根据 name 返回请求头值，键值对
  * `getHeaders(String name)`
  * `getHeaderNames`

* 获得客户端**请求参数（提交的数据）**  键值对
  * `String getParameter(name)`
  * `String getParameterValues（String name）`
    * 如果参数的值不止一个，用该方法，比如复选框的参数，等等
  * `getParameterNames`
  * `Map getParamterMap()`
    * 返回参数 键值对 map

  > If the parameter data was sent in the request body, such as occurs with an HTTP POST request, then reading the body directly via `getInputStream()`or `getReader()` can interfere with the execution of this method.

* 常见应用：表单输入数据项获取

#### 乱码问题

* POST 乱码
  * **请求正文**
  * 编码**依赖 HTML 网页**，  `<meta charset="utf-8">`  (一个汉字占 3 个字节) 放入 HTTP 请求正文
    * 如   `<meta charset="gbk">`  
    * `request.setCharacterEncoding("gbk")`  
  * 服务器获取请求参数式没有指定解码方式，则会导致乱码
  * **POST 提交请求正文 Tomcat 默认编码字符集为 `iso-8859-1`(or Latin1)**
  * 依赖浏览器编码，**浏览器默认编码字符集为 `UTF-8`**
  * 解决设置 Tomcat 编码方式为 UTF-8 ：`request.setCharacterEncoding("UTF-8")`  
* GET 乱码 （URL乱码）<_通常 HTML 编码都为 UTF-8>
  * **请求参数**

  * 编码**依赖 HTML 网页**，  `<meta charset="gbk">`  ，放入 HTTP 请求参数，导致 URL 乱码

  * **Tomcat 服务器对 GET 提交方式默认解码字符集为 `UTF-8`**

  * 解决：Tomcat 服务器 `service.xml` 中添加属性 `<Connector URIEncoding="GBK">`
    * 即，让 Tomcat HTTP 请求参数解码格式为 GBK
    * IDEA 中，若设置了 Tomcat 实例配置，只需要**修改 Tomcat 本体的编码** （见 day40）
    * 改实例 Service.xml 编码，不会生效，实例会在每次运行时拷贝一份 Tomcat 本体 Service.xml 配置

    ```XML
    <Connector port="80" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443"
                   URIEncoding="GBK"/>
    ```
* IDEA Tomcat Console 乱码
  * Tomcat Configurations => Startup/Connection => pass environment variables 
    * `JAVA_TOOL_OPTIONS = -Dfile.encoding=UTF-8`

#### 表单提交

* 表单中的所有元素，必须有 `name` 属性才能拿到其值
* 表单数据通过 `submit` 元素，进行提交
* 表单的 `action` 属性：指明要将数据提交给谁，见下看 action 属性的路径
* 要实现单选效果，保证同一组单选框，`name` 属性的值相同
* 获取表单单个数据 `String getParameter(name)`
* 获取复选框的值 `String getParameterValues（String name）`
* **将数据放入 Bean 类对象中，在内存中存储**

#### 第一个框架 BeanUtils

* 依赖两个包
  * `commons-beanutils-1.8.3`
  * `commons-logging-1.1.1`
* 使用方法
  * `BeanUtils.populate(Object obj, Map data)`
  * `BeanUtils.populate(bean, request.getParameterMap())`
* 注意事项
  * Bean 类必须有无参的构造方法
  * Bean 类必须有 get set 方法
  * Bean 类被填充的成员变量名称，必须与填充的数据 Map 中的 `键 key` 完全一致

#### request 请求转发

* 请求转发：一个 Web 资源收到客户端请求后，通知服务器调用另一个 Web 资源进行处理
* `RequestDispatcher requestDispatcher = request.getRequestDispatcher()`
* `requestDispatcher.forward()`  实现转发请求
* **`request`  对象，是一个域对象**，可以实现数据通信
  * `setAttribute()`
  * `getAttribute()`
  * `removeAttribute()`
  * `getAttributeNames()`

####RequestDispatcher

* 请求分发器
* `RequestDispatcher requestDispatcher = request.getRequestDispatcher()`
* `RequestDispatcher(String pathname)` ，pathname 见下
* 方法

  * `public void forward(ServletRequest request,ServletResponse response)`

    * 把请求转发给目标组件 （Servlet、JSP、HTML）
  * `public void include(ServletRequest request,ServletResponse response)`
    * 包含目标组件的响应结果 （Servlet、JSP、HTML）
* 拿到 RequestDispatcher 对象
  * `ServletContext`  对象 调用 `RequestDispather(String path1)`
    * path1 必须以 `/` 开头，不写会抛出异常 `IllegalArgumentException`
  * `ServletRequest`  对象 调用 `RequestDispather(String path2)`
    * path2 可以用绝对路径，也可以用相对路径
* **请求转发时的提交方式和原始请求提交方式相同**


#### RequestDispatcher 路径

只能提交给当前服务器当前应用

只能是相对路径，例如 ServletA 路径 `http://localhost:8080/myapp/servlet/servletA`

* 以 `/` 开头，相对于当前 Web app 的根路径 `http://localhost:8080/myapp`
  * `request.getRequestDispatcher("/servlet/servletB");`
* 不以 `/` 开头，相对于当前 ServletA 父路径 `http://localhost:8080/myapp/servlet/`
  * `request.getRequestDispatcher("servletB");`

#### 表单中 Action 路径

可以提交给任意服务器的任意应用

例如表单 `login.html` 路径 `http://localhost:8080/myapp/login.html`

* 绝对路径
  * `http://localhost:8080/myapp/servlet/handleLoginServlet`
    * `action="http://localhost:8080/myapp/servlet/handleLoginServlet"`
* 相对路径
  * 以 `/` 开头，相对于当前 Tomcat 服务器的根路径 `http://localhost:8080/`
    * `action="/myapp/myservlet/handleLoginServlet"`
  * 不以 `/` 开头，相对于当前 `login.html`  父路径 `http://localhost:8080/servlet/`
    * `action="servlet/handleLoginServlet"`


#### 猜想：两个 Servlet 互相转发

`StackOverflowError` 会出错！

#### 猜想：Request 请求参数

* POST 提交能通过在 URL 后面拼接参数传递参数再转发或包含，能再次增加参数
* GET 则只能修改 GET 参数
* 内部转发或包含，安全性能高