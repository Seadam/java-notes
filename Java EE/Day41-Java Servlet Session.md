# Java Servlet Session

#### 内容摘要

会话概述，Cookie， Cookie api，Cookie 应用，浏览器携带 Cookie 条件，Cookie 细节，Cookie 容量限制，Session， Session API， Session & Cookie 比较

---

在 Tomcat 多线程中，Servlet 都是单实例，对于多个请求，会对每一个请求创建一个新的线程，在新的线程中调用 Servlet 的 `service()` ，称为单实例多线程

#### 会话概述

* 会话：用户打开**一个浏览器**，点击多个超链接，访问**同一个服务器的多个 Web 资源**，再**关闭浏览器**的过程
* 会话解决的问题： **一个会话过程中的数据存储**

#### Cookie

* Cookie：**浏览器**保存技术，服务器把每个用户的数据以 Cookie 的形式通过响应头 `Set-Cookie` 写到浏览器，当浏览器再次访问服务器中 Web 资源时，自动带上所有 Cookie 放在 `Cookie` 请求头
* 一个 Cookie 只能存储一个 键值对

>Creates a cookie, a small amount of information sent by a **servlet** to a Web browser, saved by the browser, and later sent back to the server. A cookie's value can uniquely identify a client, so cookies are commonly used for session management.

#### Cookie API

* `javax.servlet.http.Cookie`
  * `public Cookie(String name, String value)`
  * `setValue()` `getValue()`
  * `setMaxAge()` `getMaxAge()` （单位秒），声明周期
    * 若不显式设置，浏览器会默认设置，`request.cookies()` 返回的不会封装浏览器默认属性
    * 默认是存储在浏览器内存中，浏览器关闭当前 Cookie 删除
    * `setMaxAge(int expiry)` 
      * 正数，存在 expiry 秒 后过期
      * 负数，浏览器关闭后删除
      * 0，立马过期，删除
    * **删除操作**
      1. `request.getCookies()` 找到要删除的 Cookie cookie
      2. **`cookie.setPath(uri)` 指明 cookie 的 path，要与浏览器中待删除 Cookie 的 path 一致**
      3. `cookie.setMaxAge(0)` 设置过期时间为 0
      4. `response.addCookie(cookie)`  写回浏览器，让浏览器删除该 cookie
  * `getName()`
  * `setPath(String uri)` `getPath()`
    * `URI` 
    * 若不显式设置，浏览器会默认设置，`request.cookies()` 返回的不会封装浏览器默认属性
    * **默认路径是创建该 Cookie 的 Servlet 父目录 URI**
    * 路径设置：以 `/` 开头，相对于 Tomcat 服务器
  * `setDomain(String domain)` `getDomain()`
    * `domain`
    * 若不显式设置，浏览器会默认设置，`request.cookies()` 返回的不会封装浏览器默认属性
    * **默认值是创建该 Cookie 的 Servlet 所在服务器域名**
* `response.addCookie(Cookie cookie)` 响应头增加 `Set-Cookie` 字段，可以发送多个 Cookie
* `request.getCookies()` 拿到请求头 `Cookie` 字段所有 Cookie 数组
  * 如果浏览器没有携带 Cookie 信息，返回 null

#### Cookie 应用

* 显示用户上次访问时间
* 保存登陆用户名

#### 浏览器请求头携带 Cookie 条件

* 为节省网络带宽，浏览器携带 Cookie 需要条件限制，所有符合 Cookie 放在 `Cookie` 请求头
* 根据本次请求的 URL
  * `domain == cookie.getDomain()`
  * `URI.startWith(cookie.getPath())`  默认路径是创建该 Cookie 的 Servlet 父目录 URI
* 例如，请求 URL `localhost/myApp/servlet/cookie`  
  * 域名：`localhost`  
  * URI： `/myApp/servlet/cookie` 
  * cookie path 为 `/myApp/servlet`
  * 符合条件，将会携带在请求头
* 浏览器默认设置的 domain 和 path 不会发送到服务器，若不显式设置，`getDomain()` `getPath()` 都为 null
* 即使显式设置了 domain 和 path，浏览器也不会发送回服务器，即请求头 Cookie 不会携带

#### Cookie 细节

* 一个 Cookie 只能标识一种信息，键值对，String
* 服务器可以给一个浏览器发送多个 Cookie，浏览器可以存储多个 Cookie
* 浏览器一般允许存放 300 个 Cookie，每个站点最多存放 20-30 个，每个 Cookie 大小限制 4kb
* 默认 Cookie 时是一个**会话级别的 Cookie（存储在浏览器内存中）**，用户退出浏览器删除
  * 如果希望 Cookie 存储在硬盘上，需要声明 `maxAge` 属性（单位秒）
  * `cookie.setMaxAge(0)` 命令浏览器删除该 Cookie
  * 删除 Cookie 时，`path` 必须一致，否则不能删除
* Cookie 中文 value 浏览器兼容问题
  * Chrome 支持中文 value
  * `URLEncoder.encode(String value, String charset)`  将中文编码后写入 value 中
  * `URLDecoder.encode(String value, String charset)`  从 value 中解码拿到中文

#### Cookie 容量限制

| 浏览器     | Cookie 数量限制 | Cookie 总大小限制 |
| ------- | ----------- | ------------ |
| IE8     | 50个／每个域名    | 4095 字节      |
| IE9     | 50个／每个域名    | 4095 字节      |
| Chrome  | 50个／每个域名    | 大于 80000     |
| FireFox | 50个／每个域名    | 4097 字节      |

#### Session

* Session：**服务器**保存技术，服务器为**每一个浏览器**创建一个**独享的 HttpSession 对象**，用户各自的数据放在各自的 Session 中，当用户访问服务器中的其他 Web 资源时，从 Session 中取出各自数据
* **一个浏览器独占一个 HttpSession 对象**，HttpSession 对象由**服务器创建**
* `JsessionID` 唯一标识浏览器独享的 HttpSession 对象
* **`HttpSession` 是个域对象**
  * `setAttribute(String name, Object value)`
  * `getAttribute(String name)`

#### Session API

* `request.getSession()`
  * 返回当前浏览器请求 Cookie  `JsessionID` 相关联的 HttpSession 对象，
  * 若没有创建 HttpSession 对象，则创建
* `request.getSession(boolean create)`
  * 返回当前浏览器请求 Cookie  `JsessionID` 相关联的 HttpSession 对象
  * 若 create 为 true，若没有创建 HttpSession 对象，则创建
  * 若 create 为 false，若没有创建 HttpSession 对象，则不创建

#### Session & Cookie

* Cookie
  * 用户数据写到用户浏览器
  * 浏览器保存
* Session
  * 用户数据写到用户独占的 Session 中
  * 服务器保存












