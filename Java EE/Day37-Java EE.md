# Java EE

#### 内容摘要

概述，常见 Web 服务器程序，结构，Java EE 规范，Tomcat，安装，Web app，URL，Tomcat 组件，Tomcat 体系结构，Context，Web.xml，软件开发架构，HTTP 事务工作流程，域名解析，TCP 三次握手，HTTP 协议，请求，响应，HTTP1.1

---

#### 概述

* 供外界访问的资源，数据
  * 静态：
    * 数据始终不变
    * HTML ／ CSS ／JavaScript
  * 动态：
    * 数据由程序产生
    * **JSP / Servlet（Java）**，JAP，PHP
* Web
  1. Socket 通信（进程与进程之间的通信）
  2. HTTP 请求的接收和解析
  3. **实现应用业务逻辑**
* **Web 服务器（Web 应用程序）** 核心功能
  * 实现 Socket 通信（进程与进程之间的通信），并发多线程
  * 实现 HTTP 请求的接收和解析
  * 实现 HTTP 响应的封装和发送
* Web 服务器区分
  * 硬件：一台电脑，运行 web 服务器程序，如 dns， rip，os
  * 软件工程师：Web 服务器程序

#### 常见 Web 服务器（程序）

* WebLogic
  * BEA 公司
  * 支持 Java EE 规范
* WebSphereAS   
  * IBM 公司
  * 支持 Java EE 规范
* JBossAS  
  * redhat
  * 支持 Java EE 规范
* Tomcat
  * 支持**全部 Java EE 规范**（Servlet ／ JSP）
  * 免费
  * 开源

#### 结构

* 一流公司（Sun 公司）：制定标准（Java EE 规范）
* 二流公司（web 服务器公司）：实现标准
* 三流公司：做产品
  * 基于服务器提供的接口，实现具体的业务逻辑

  #### Java EE 规范

* Java 服务器程序**接口规范**
  * `java servlet`
  * `java servlet.jsp`

#### Tomcat

* Sun 公司测试 Servlet ／ JSP 规范的时候使用的，**Servlet 容器**
* 目前是 Apcache 的一个开源项目
* 纯 Java 编写

| Tomcat | Servlet / JSP 规范 | JDK   |
| ------ | ---------------- | ----- |
| 8.0    | 3.1 / 2.3        | JDK 7 |
| 7.0    | 3.0 / 2.2        | JDK 6 |

#### 安装之后

* `localhost:8080`  检查是否启动服务器成功
* 默认使用端口 8080

```java
// Tomcat 目录结构
Tomcat
|- bin // 二进制执行文件，启动、关闭 .sh .exe .dat
|- conf // 服务器配置文件 .xml
|- lib // 服务器运行依赖包 .jar
|- logs // 日志文件 .txt
|- temp // 运行时临时文件
|- webapps // 部署的 Web 应用程序，一个文件夹代表一个应用，供外界访问的 Web 数据资源
	|- myapp // Web 应用程序
		|- .html, .css, .js // 根目录下的文件／文件夹 外界可以直接访问
		|- images // 通常会创建不同文件夹管理
		|- style
		|- script
		|- WEB-INF // 该目录下的文件外界无法访问，由 Web 服务器调用
			|- classes // 编译后 class 文件目录 .class
			|- lib // java 类运行所需 jar 包 .jar
			|- web.xml // 配置文件，配置 Web 资源
|- work // Tomcat 工作目录，Tomcat 由 Java 写的
```

#### Web 应用

* 开放式目录，如上
* `war` 打包，Tomcat 会自动解压到 webapps 路径下

#### URL

* 定位资源
* Web 应用默认 URL 入口是 Web 应用根目录名，如本地： `http://localhost:8080/Myapp`
* `http://localhost:8080/Myapp/index.html`
  * `http://127.0.0.1:8080/Myapp/index.html ` 等价

#### Tomcat 组件

* 核心组件： **`Servlet` 容器**
* 组件：`Servlet` `Service` `Connector` `Engine` `Host` `Context`
* 每个组件可以在 `/conf/server.xml` 配置

```xml
<!-- 组件结构 -->
<Servlet>
  <!-- 最顶层元素，可以包含多个 <Service> 元素 -->
  <Service>
     <!-- 包含一个或多个 <Connector> 共享 一个 <Engine>， -->
    <Connector/> <!-- 和客户程序实际交互的组件，负责接收客户请求，向客户返回响应 -->
    <Engine>
      <!-- 处理同一个 <Service> 中所有 <Connector> 接收到的客户请求 -->
      <!-- 包含一个或多个 <Host> -->
      <Host>
        <!-- 代表一个虚拟主机（网络） -->
        <!-- 包含一个或多个 <Context> -->
        <Context/> <!-- 运行在虚拟主机上单个 Web 应用 -->
      </Host>
    </Engine>
  </Service>
</Servlet>
```

#### 体系结构

* 图

#### Context

* 虚拟目录映射，可以不用放到 `/usr/local/Cellar/tomcat/9.0.4/libexec/webapps/` 目录下
* `<Context path="/myapp" docBase="/Users/clixin/TomcatApps/myapp" reloadable="true"/>`
* 映射 myapp 默认启动，即打开 `localhost` 就显示该应用，而不用附加 url
  * 方法一：path 配置为空，则匹配任何context path的请求，达到默认启动
    * `<Context path="" docBase="/Users/clixin/TomcatApps/myapp" reloadable="true"/>`
  * 方法二：指定 `ROOT.xml`
    * 放在  `/usr/local/Cellar/tomcat/9.0.4/libexec/conf/Catalina/localhost` 目录下
    * 内容  `<Context docBase="/Users/clixin/TomcatApps/myapp" reloadable="true"/>`
* 参数说明
  * `path`
    * 该 web 应用 URL 入口，如果为空，匹配任何 context path 的请求
  * `docBase`
    * web 应用文件根目录的 **绝对路径**，也可以给相对于 `appBase` 的相对路径
  * `reloadable`
    * true，当 web.xml 或者 class 有改动的时候都会自动重新加载不需要从新启动服务
    * 调试阶段方便调试，产品上线关闭降低服务器负荷
  * `className`
    * 实现 Context 组件的 java 类的名字，这个类需要实现 `org.apache.catalina.Context` 接口

#### Web.xml

* 配置 web app 首页
  * `<welcome-file-list> <welcome-file>index.html</welcome-file></welcome-file-list>`
* 使用注解 + web.xml 配置
  * `<metadata-complete="false"/>`

#### 软件开发架构

* Client／Server
* Browser／Server

## HTTP 事务过程

#### HTTP 事务 工作流程

**HTTP 事务 = 请求命令（request） + 响应结果（response）**

1. 域名解析
2. 发起 TCP 三次握手
3. 发起 HTTP 请求
4. 服务器响应 HTTP 请求
5. 浏览器解析 HTTP 代码，并请求相关资源（CSS、JS、Images）
6. 浏览器渲染页面

* 图

#### 域名解析

* DNS （Domain Name Server）服务器
* 域名解析过程（顺序搜索）
  1. 搜索浏览器自身 DNS 缓存
  2. 搜索自身操作系统 DNS 缓存
  3. 读取 host 文件
  4. 向本地配置的 DNS 服务器发起域名解析请求

#### TCP 三次握手

* 第一次：client ===> server
  * `SYN = 1 ACK = 0 seq = x`
* 第二次：server ===> client
  * ``SYN = 1 ACK = 1 seq = y ack = x + 1`
* 第三次：client ===> server
  * `ACK = 1 seq = x + 1 ack = y + 1`

> seq(Sequence Number) : 用来标识从TCP发端向TCP收端发送的数据字节流，它表示在这个报文段中的的第一个数据字节在数据流中的序号；主要用来解决网络报乱序的问题
>
> ack(Acknowledgment Number) : 32位确认序列号包含发送确认的一端所期望收到的下一个序号，因此，确认序号应当是上次已成功收到数据字节序号加1。不过，只有当标志位中的ACK标志为1时该确认序列号的字段才有效。主要用来解决不丢包的问题
>
> **ACK ： TCP协议规定，只有ACK=1时有效，也规定连接建立后所有发送的报文的ACK必须为1**
>
> **SYN(SYNchronization) ： 在连接建立时用来同步序号。当SYN=1而ACK=0时，表明这是一个连接请求报文。对方若同意建立连接，则应在响应报文中使SYN=1和ACK=1. 因此, SYN置1就表示这是一个连接请求或连接接受报文。**
>
> [source](https://github.com/jawil/blog/issues/14)

![tcp](https://camo.githubusercontent.com/36cf7d4e1598683fe72a5e1c3e837b16840f4085/687474703a2f2f6f6f327239726e7a702e626b742e636c6f7564646e2e636f6d2f6a656c6c797468696e6b544350342e6a7067)

## HTTP 协议

* HTTP 协议：Web 浏览器与 Web 服务器之间的请求和响应交互过程遵循标准
* HTTP: **Hyper Text Transfer Protocol**
  * **HTTPS** 数据加密，身份认真
* TCP／IP 协议，**应用层**的协议
* 版本：HTTP1.0   **HTTP 1.1** （区别在状态）


* **HTTP 事务 = 请求命令（request） + 响应结果（response）**
* 报文流：流入（流向）服务器，流出（流向）客户端

#### HTTP 请求消息 request 

* 请求行
  * 请求方法  POST／GET
  * 请求资源 URL
  * HTTP 版本
* 请求首部
  *  `信息名: 值`
  *  键值对
* **一个空行** （分隔）
* 请求正文

```http
GET / HTTP/1.1
Host: www.acfun.cn
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Referer: http://www.acfun.cn/search/
Accept-Encoding: gzip, deflate
Accept-Language: zh-TW,zh;q=0.9,en-US;q=0.8,en;q=0.7,zh-CN;q=0.6
Cookie: uuid=11f1021cdab27e9a2be99fefbce9dabf

请求正文...
```

#### 请求行

* 请求方式：GET／POST   （HEAD、OPTIONS、DELETE、PUT）
  * 默认是 GET 请求，如地址栏访问，超链接访问
  * 可以通过更改表单提交方式把请求方式改为 POST
* GET 方式
  * 请求数据以 `?` 带上数据，多个数据以 `&` 分隔
  * 如：`www.example.com?username=tony&age=12`
  * **URL 地址附带的参数数据容量不能超过 1K**
* POST 方式
  * 数据是放在请求正文中向服务器发送请求的
  * **数据量无限制**
* 负载均衡，缓存技术，结合使用

#### 请求头部

* 传递附加信息，格式为 `信息名: 值`  （有空格）
* 不区分大小写，习惯每个单词第一个字母大写
* 各行消息头可以**按任意顺序排列**
* 分为：信息头、请求头、响应头、实体头
* 多个可选选项用 `,` 分隔
* **有些消息头可以出现多次**

#### 常见请求头部如下

* `Accept: text/html,*/*`  浏览器可接受的 MIME 类型，大类型／小类型
* `Accept-CharSet: UTF-8`  浏览器支持的字符集
* `Accept-Encoding: gzip, deflate`  浏览器能够进行解码的数据编码方式
* `Accept-Language: zh-CN`  浏览器期望返回的语言种类
* `Host: www.acfun.cn`  初始 URL 中的主机和端口（默认为80）
* `Referer: http://www.acfun.cn/search/`  用户从该 URL 访问到当前请求页面
  * 可以防止盗链
* `User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36`  浏览器 navigator 信息
* `If-Modified-Since: Mon, 12 Feb 2018 11:40:58 GMT`  服务器利用该信息与服务器中文件对比，如果一致，则告诉浏览器从 缓存中直接读取文件不需要
* `Cookie: uuid=11f1021cdab27e9a2be99fefbce9dabf`  缓存
* `Connection: keep-alive`  持久链接，详见下 HTTP 1.1 特征


* `Content-Type`  请求消息内容类型
* `Content-Length`  请求消息内容长度，字节
* `Date`  发送请求时间

#### HTTP 响应消息 response

* 响应行
  * 协议版本 状态码 状态码描述
* 响应头部
  *  `信息名: 值`
* **一个空行**（分隔）
* 响应正文

```http
HTTP/1.1 200 OK
Server: Tengine
Content-Type: image/jpeg
Content-Length: 15323
Connection: keep-alive
Date: Tue, 30 Jan 2018 02:28:31 GMT
Last-Modified: Tue, 30 Jan 2018 02:22:22 GMT
ETag: "5a6fd6de-3bdb"
Expires: Thu, 27 Sep 2018 02:28:31 GMT
Cache-Control: max-age=20736000
Access-Control-Allow-Origin: *
Accept-Ranges: bytes
Age: 1158688
X-Cache: HIT TCP_MEM_HIT dirn:8:246033459 mlen:-1
X-Swift-SaveTime: Tue, 30 Jan 2018 02:28:31 GMT
X-Swift-CacheTime: 31104000
Timing-Allow-Origin: *
EagleId: b73df1a815184379998654605e

响应正文...
```

#### 状态行

常见状态码

* 200 正常
  * 表示一切正常，返回正常请求结果
* 301、302/307（重定向）
  * 301：`Moved Permanently`  永久重定向
    * 比如 `acfun.cn`  重定向到 `www.acfun.cn`
  * 302:   `Found`  重定向
  * 307：`Temporary Redirect`  临时重定向
    * 与 302 区别，浏览器收到 307 后，应保持请求方法不变，并向新的地址发送请求
* 304 未修改 `Not Modified`
  * 直接读取浏览器缓存内容
* 404 `Not Found`
* 500 服务器内部错误
  * 服务器应用程序抛异常

#### 常见响应头部 

* `Location: http://www.acfun.cn`  指定重定向的 URL 地址
* `Server: Tengine`  服务器的类型
* `Content-Encoding: gzip` 服务器发送的数据采用的编码类型
* `Content-Length: 80` 告诉浏览器响应正文的长度
* `Content-Language: zh-cn`  服务发送的响应正文的语言
* `Content-Type: text/html`  服务器发送的响应正文的 MIME 类型
* `Last-Modified: Tue, 30 Jan 2018 02:22:22 GMT` 文件的最后修改时间
* `Refresh: 1,url=http://www.bilibili.tv` 指示浏览器刷新频率，重定向到新 URL 单位是秒
  * 通常用于提醒用户转到外部链接
* `Content-Disposition: attachment; filename=aaa.zip`  **指示客户端保存文件，浏览器下载文件**
* `Set-Cookie: SS=Q0=5Lb_nQ; path=/search`  服务器端发送的 Cookie
* `Expires: 0`  缓存过期时间
* `Cache-Control: no-cache (1.1)`  缓存控制
* `Connection: close/Keep-Alive`  链接类型，断开链接 发送 close
* `Date: Tue, 11 Jul 2000 18:23:51 GMT`  服务端响应时间

#### HTTP 1.0

客户端建立连接 ==> 客户端发送请求 ==> 服务端返回响应 ==> 客户端关闭连接

* 无状态，没有记录链接的状态，不记录客户端链接到的服务器
* 每次发送请求都要重新建立连接

#### HTTP 1.1

* 有状态，记录链接的状态，记录客户端链接到的服务器
* **一次链接可以发送多个请求和响应**
* 省去了每次请求重新建立链接的时间
* 添加更多请求头和响应头

