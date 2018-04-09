# Java Filter

#### 内容摘要

概述，工作原理，流程，使用，流程，步骤，FilterChain，生命周期，配置，uri-parttern，装饰者设计模式，

Listener，概述，三大类型接口，拿到服务器中文件路径

---

#### 概述

* Servlet 2.3 才引入，是一个 Web app 组件
* 由 **Servlet 容器**调用，用来**拦截和处理请求、响应**
* 不能生成请求和响应对象，但是可以对请求和响应对象进行**检查和修改**

#### 工作原理

* 过滤器介于浏览器与 **Servlet/JSP** 等相关的资源之间
* 关联了 Filter 的 Servlet，在 Servlet 被调用之前检查并且修改 **request** 对象
* 关联了 Filter 的 Servlet，在 Servlet 调用之后检查并修改 **response** 对象

#### 工作流程

1. 浏览器请求发送给 Web 容器
2. Web 容器生成 request 和 response 对象
3. 调用绑定了 Filter 的 Servlet 的 service 方法时，会先将 request 和 response 交给 Filter
4. Filter 先对 request 进行处理
5. 再把处理过的 request 和空的 response 对象传递给 service 方法
6. service 方法结束后，再通过 Filter 对 response 进行处理
7. Filter 把处理过的 response 对象传递给 Web 容器
8. Web 容器将响应结果返回到浏览器

#### Filter 使用

* Servlet 实现 `javax.servlet.Filter` 接口

* `void init(FilterConfig config)`
  * 初始化过滤器，当容器装载并初始化过滤器时调用
  * Web 容器为此方法传递一个 **FilterConfig** 对象
  * **FilterConfig** 对象可以获取 **web.xml** 文件中过滤器初始化参数的配置
  * **IDEA** 通过注解配置初始化参数
  * 利用 **FilterConfig** 对象可以获取当前 **Filter** 的名称以及相关联的 **ServletContext** 对象

* `void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)`

  * **ServletRequest** 对象为请求对象，包括表单数据、**Cookie**以及**HTTP**请求头等信息
  * **ServletResponse** 对象为响应对象，用于响应使用 **ServletRequest** 对象访问的信息
  * **FilterChain** 用来调用过滤器链中的下一个资源
  * `chain.doFilter(req, res)`
    * 将 **req** 和 **res** 对象传递给**下一个过滤器**或者是其它的 **Servlet/JSP** HTML
    * filter1 => filter2 => filter3 => servlet / jsp

* `void destroy()`
  * 当容器要销毁过滤器实例时调用此方法，**Filter** 占用的资源会被释放


* 拦截：不将请求转发给接收 Servlet，即不调用 `chain.doFilter`
* 处理：可以直接操作 **requset** 和 **response** 对象
* 移交给下一个过滤器处理或转发给 Servlet 处理 `chain.doFilter(req, res)`

#### Filter 流程控制

* Filter 处理请求 => Servlet 处理请求 => Servlet 处理响应 => Filter 处理响应
* 普通的方法调用，过滤器链式方法调用 `chain.doFilter(req, resp);`

```Java
 public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
   		// 预处理 request, response
        chain.doFilter(req, resp);
   		// 后处理 response
}
```

#### 使用步骤

1. 创建一个实现 `Filter` 接口的 Java 类
2. 实现 `init()` 方法，读取过滤器初始化参数
3. 实现 `doFilter()` 检查和修改操作
4. `doFilter()` 调用 **FilterChain** `chain.doFilter()` ，将过滤器**传递给后续过滤器**，即放行
5. 在 web.xml 注册过滤器，设置参数以及过滤器要过滤的资源

#### FilterChain

* **对于同一个 URL 请求，会调用多个符合路径匹配的 Filter，形成一个过滤链**
* 过滤器使用 FilterChain 调用链中的下一个过滤器，直到最后一个过滤器
* 多个 Filter 过滤，执行完所有 `chain.doFilter(request,response)` 才会交给 web 资源
* 多个 Filter 形成了 Filter 链执行顺序取决于 `<filter-mapping>` 在 **web.xml** 文件中配置的先后顺序
* **IDEA** 由于使用注解来配置 urlParttren ，**根据 FIlter 类名， 字典顺序来决定**

#### Filter 生命周期

* **服务器启动**，会创建 Filter 对象，并调用 `init()`，只调用一次
* 访问资源时，与 Filter 的路径匹配时，会执行 Filter 中的 `doFilter()`
* 服务器关闭，会调用 Filter 的 `destroy()`
  * Tomcat Manager 关闭 Web app 也会调用  `destroy()`

#### Filter 配置

* **IDEA** 通过注解来配置
* `@WebFilter(filterName = "name", value = "/url", servletNames = "servletA")`

```xml
<filter>
	<filter-name> filter名称 </filter-name>
	<filter-class> filter类全名 </filter-class>
    <servlet-name> 指定拦截 Servlet </servlet-name>
</filter>
<filter-mapping>
	<filter-name> filter名称 </filter-name>
	<url-pattern> 映射路径：格式同 Servlet </url-pattern>
</filter-mapping>
```

#### Filter uri-pattern

> * URL 支持两种通配符
>   * `*.扩展名`  如  `.html`  `*.do`
>   * `/some dir/*`  如  `/action/*`
>   * 不允许 `/abc/*.do`  两种通配符混用会出错
>   * 优先级
>     * 越精确越优先匹配
>     * `/*`   高于 `*.do`
> * IDEA 用注解替代 web.xml 配置   `@urlpatterns = {"url1", "url2", ...}`
> * `/` => `/*` => `/abc/*` => `*.do`
>   * 优先着找精确路径名下的 Servlet  `/abc/*`
>   * 当没有路径名匹配时，找符合后缀的 Servlet `.html`
>   * 最后找默认路径 Servlet `/`
>   * 注意：当出现 `/*` 时，会覆盖 `/` 以及 `*.html`

* 对 urlPattern 匹配请求进行过滤
* 完全匹配，以 `/` 开始
* 目录匹配，以 `/` 开始，以 * 结束
* 扩展名匹配，以 `*.xxx` 结束

#### 装饰者设计模式应用

* 问题描述：用 Filter 来修改敏感词汇，预处理请求参数
  1. 替换请求对象
  2. 自定义一个请求对象， **extends HttpServletRequest**
  3. 重写 **getParameter(String name)** 方法
     * 在此方法中对请求参数进行修改
* 使用装饰者设计模式：不修改一个类已有代码情况下，增加新功能

```Java
public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        // 替换请求参数，装饰者设计模式
        // 1. 自定义 request 对象，继承自 HttpServletRequestWrapper
        MyRequest myRequest = new MyRequest(req);
        // 2. 重写 getParameter(String name) 方法
        // 3. 传递新的 request 对象
        chain.doFilter(myRequest, resp);
}

class MyRequest extends HttpServletRequestWrapper {

    public MyRequest(ServletRequest request) {
        super((HttpServletRequest) request);
    }

    @Override
    public String getParameter(String name) {
        if ("message".equals(name)) {
            String s = super.getParameter(name);
            return s.replace("TMD", "挺萌的");
        }
        return super.getParameter(name);
    }
}

```



## Listener

#### 概述

* 监听当前整个 Web app，需要在 web.xml 配置 `<listener></listener>`
* **IDEA** 配置 `@WebListener()` 没有参数
* 回调

#### 三大类型 Listener 接口

* 监听 Web 域对象的创建与销毁
  * `ServletContextListener`
    * `contextInitialnized(ServletContextEvent sce)`
    * `contextDestroyed(ServletContextEvent sce)`
  * `HttpSessionListener`
  * `ServletRequestListener`
* `sce.getSource()`  返回当前监听事件发生的对象源
* 监听域对象属性的变化
  * `ServletContextAttributionListener`
  * `HttpSessionAttributionListener`
  * `ServletRequestAttributionListener`
* 监听 Session 与 JavaBean **由 JavaBean 实现该接口**
  * `HttpSessionActivationListener`  Session 中 JavaBean 激活（反序列化）和钝化（序列化）
    * `sessionWillPassivate(HttpSessionEvent se)`
    * `sessionDidActivate(HttpSessionEvent se)`
  * `HttpSessionBindingListener`  Session 绑定 JavaBean
    * `valueBound(HttpSessionBindingEvent event)`
    * `valueUnbound(HttpSessionBindingEvent event)`

## 拿到服务器中文件的路径

> 真实路径
>
> * （真实路径）运行路径：资源和 classes 运行在 Tomcat 服务器上
> * 源码路径：本地编写源码路径
> * `WEB-INF`  内的文件，外部不能访问，内部可以通过流来访问，实现控制访问
>   * 此处**文件路径需要填写真实运行路径**
> * 拿到真实路径
>   * 假设 `pic.img` 在 `WEB-INF` 根目录下
>   * `getContext().getRealPath("pic.img")`   
>   * `MyServlet.class.getClassLoader().getResource("pic.img")`
>   * 返回该文件在 war 包中真实路径

* `getContext().getRealPath("pic.img")`    这个目录是 /web 根目录
* `MyServlet.class.getClassLoader().getResource("pic.img")`  这个目录是 classes 目录











