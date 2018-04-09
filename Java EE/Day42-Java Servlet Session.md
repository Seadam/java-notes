# Java Servlet Session

#### 内容摘要

请求转发提交方式，Cookie 中文 值 浏览器兼容问题，购物车 Demo，Session 生命周期，IE 禁用 Cookie 后 Session 处理，

---

#### 请求转发提交方式

* **请求转发时的提交方式和原始请求提交方式相同**

#### Cookie 中文 value 浏览器兼容问题

Chrome 支持中文 value

* `URLEncoder.encode(String value, String charset)`  将中文编码后写入 value 中
* `URLDecoder.encode(String value, String charset)`  从 value 中解码拿到中文

#### 购物车 Demo

1. 动态生成首页信息 `ServletIndexPage`
2. 点击商品跳转到商品详情页面（超链接携带 id 参数） `ServletProductDetail`
3. 点击加入购物车，跳转到添加到购物车（超链接携带 id 参数）`ServletAddToCart`
   1. 检查 Session 第一次创建
      1. 创建 Session
      2. 创建 ArrayList，添加到 Session 中
   2. 不是第一次创建
      1. 拿到 Session
      2. 拿出 ArrayList
         * 若 ArrayList 为空，创建新 ArrayList，添加到 Session 中
   3. 将商品加入到 购物车
4. 查看购物车，显示购物车中已加入商品 `ServletShowCart`

#### Session 生命周期

* 产生：由 Tomcat 产生， `request.getSession()`

* 运行：当内存紧张时，Tomcat 会将用户不常用的 Session 序列化到硬盘，释放内存空间

* 死亡：

  * Session 失效： `invalidate()`  立即失效
  * Session 空闲过期死亡
    * 在 Tomcat Manager App 里可以设置 `Expire sessions` 空闲回收
    * 也可以在 Tomcat 本体或实例 `Web.xml` 设置 Session 空闲过期时间，单位秒

  ```xml
  <!-- web.xml -->
  <!-- Session 过期时间 -->
  <session-config>
  	<session-timeout>30</session-timeout>
  </session-config>
  ```

* 序列化与反序列化

  * 当内存紧张时，Tomcat 会将用户不常用的 Session  序列化到硬盘，释放内存空间
  * 通过 Tomcat Manager App 关闭或重启 Web App 也会使其序列化与反序列化
  * 关闭或重启 Tomcat ，Tomcat 会序列化反序列化 Session
  * Tomcat 默认将 Session 序列化保存在硬盘中，可以在 Tomcat 实例里面找到序列化的文件
    * 实例 Session 位置，见下
    * 序列化及反序列化需要 **Session 中存储的内容都实现 `java.io.Serializable` 接口**
    * 即 **Bean 类实现 `java.io.Serializable` 接口**
  * IDEA 中需要选中 `Preserve sessions across restarts and redeploys` 
  * `/Users/clixin/Library/Caches/IntelliJIdea2017.3/tomcat/Tomcat_9_0_4_JavaWeb/sessions`

#### IE 禁用 Cookie 后 Session 处理

禁用 Cookie 后，导致无法拿到 `JSessionID`，解决思路，将 `JSessionId` 拼接到请求 URL 中

* 解决方案：URL 重写
  * `response.encodeRedirectURL(String url)`
    * 重写 重定向的 URL 地址
  * `response.encodeURL(String url)`
    * 如果有 JSessionID ，会加密**拼接到请求 URL 中**
    * 从 URL 拿到 JSessionID，拿到指定 Session
    * 重写 **表单 action 和超链接 herf** 的 URL 地址

