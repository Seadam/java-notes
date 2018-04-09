# Java Servlet

#### 内容摘要

Servlet 简介，实现 Servlet，Servlet 生命周期，Servlet 执行流程，Servlet 第二种实现方式，URL映射，DefaultServlet，ServletConfig，ServletContext，IDEA 创建 Web app

---

#### Servlet 简介

* 动态 web 资源
* 通过 HTTP 协议接收和响应请求

#### 实现 Servlet

1. 编写 Java Servlet，继承 `java.servlet.GenticServlet` 

   * 依赖包： `ext/`


   * 重写 `public void service(ServletRequest req, ServletResponse res)` 方法

   * 在该方法写处理 HTTP 请求与响应的业务逻辑

     ```java
     import javax.servlet.*;
     public class MyServlet {
       public void service(ServletRequest req, ServletResponse res) 
       		throws IOException {
         // 处理 HTTP 请求与响应
         // res.getWriter.write("Hello, servlet.");
       }
     }
     ```

2. 将开发好的 Java Servlet 部署到 web 服务器中

   1. 编译 java 文件，将 class 文件放到 `WEB-INF/classes` 目录下

   2. 配置 web.xml

      ```xml
      <Servlet> <!-- class 全类名映射为一个 Servlet 名 -->
      	<Servlet-name>servlet</Servlet-name>
      	<Servlet-class>MyServlet</Servlet-class>
      </Servlet>

      <!-- 一个 url 可以映射多个 Servlet -->
      <Sevlet-mapping> <!-- url 映射到一个 Servlet 名 -->
      	<Servlet-name>servlet</Servlet-name>
      	<url-pattern>/servlets/myfirstservlet</url-pattern>
      </Sevlet-mapping>

      <!-- url ==> servlet-name ==> servlet-class -->
      ```

3. 部署成功，输入 url 进行访问 `localhost:8080/servlets/myfirstservlet`

#### Servlet 生命周期

* `init()`   Servlet 容器指明要访问某一个 Servlet 时创建该 Servlet，并调用 *init()* 方法，**懒加载**

  > Called by the servlet container to indicate to a servlet that the servlet is being placed into service.

* `service()`  处理 HTTP 请求时调用，通过 url 指定请求调用 Servlet ，见上映射

  > Called by the servlet container to allow the servlet to respond to a request.
  >
  > This method is only called after the servlet's `init()` method has completed successfully.


* `destroy()`  Servlet 被移除 service 时调用，处理资源释放

  * undeploy 取消部署会调用，会删除 web 整个目录
  * redeploy 重新部署也会调用

  > Called by the servlet container to indicate to a servlet that the servlet is being taken out of service. This method is only called once all threads within the servlet's `service` method have exited or after a timeout period has passed. After the servlet container calls this method, it will not call the `service` method again on this servlet.

#### Servlet 执行流程

1. 客户端发出请求 `localhost:8080/servlets/myfirstservlet`
2. 服务端通过 URL 映射，找到 Servlet 名，通过 Servlet 名映射，找到 class 文件， 创建对象，`init()`
3. 同时，服务端，解析 HTTP 请求，创建 Servlet 请求对象 ServletRequest 和响应对象 ServletResponse
4. 服务端调用 Servlet 的 `service（）` 方法，传入参数 ServletRequest 和 ServletResponse，处理请求与响应
5. Servlet 不再使用，服务器卸载， `destory()`

* url ==> servlet name ==> servlet class ==> `init()` ==> `service(req, res)` ==> `destroy()`
* 图

#### Servlet 第二种实现方式

* 继承 HttpServlet
* 对 `ServletRequest` `ServletResponse` 包装，区分出不同请求方式，要求至少覆盖一个方法
* `doGet(HttpServletRequest request, HttpServletResponse response)`
  * HTTP 默认 GET 提交请求，请求内容拼接在 url 后，参数大小 1k
* `doPost(HttpServletRequest request, HttpServletResponse response)`

```Java
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "MyFirstServlet", urlPatterns = "/servlet/first-servlet")
public class MyFirstServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        // System.out.println("Hello, Servlet.");
    }
}

// 源码
public abstract class HttpServlet extends GenericServlet {
  	@Override
    public void service(ServletRequest req, ServletResponse res)
        throws ServletException, IOException {

        HttpServletRequest  request;
        HttpServletResponse response;
      	// 创建对象
        try {
            request = (HttpServletRequest) req;
            response = (HttpServletResponse) res;
        } catch (ClassCastException e) {
            throw new ServletException("non-HTTP request or response");
        }
        service(request, response);
    }
  
  // 区分请求方式
  protected void service(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {

        String method = req.getMethod();

        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                // servlet doesn't support if-modified-since, no reason
                // to go through further expensive logic
                doGet(req, resp);
            } else {
                long ifModifiedSince;
                try {
                    ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                } catch (IllegalArgumentException iae) {
                    // Invalid date header - proceed as if none was set
                    ifModifiedSince = -1;
                }
                if (ifModifiedSince < (lastModified / 1000 * 1000)) {
                    // If the servlet mod time is later, call doGet()
                    // Round down to the nearest second for a proper compare
                    // A ifModifiedSince of -1 will always be less
                    maybeSetLastModified(resp, lastModified);
                    doGet(req, resp);
                } else {
                    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                }
            }

        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);

        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);

        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);

        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);

        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);

        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);

        } else {
            //
            // Note that this means NO servlet supports whatever
            // method was requested, anywhere on this server.
            //

            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);

            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }
}
```

#### URL 映射

* **一个 URL 可以映射多个 Servlet**
* URL 支持两种通配符
  * `*.扩展名`  如  `.html`  `*.do`
  * `/some dir/*`  如  `/action/*`
  * 不允许 `/abc/*.do`  两种通配符混用会出错
  * 优先级
    * 越精确越优先匹配
    * `/*`   高于 `*.do`
* IDEA 用注解替代 web.xml 配置   `@urlpatterns = {"url1", "url2", ...}`
* `/` => `/*` => `/abc/*` => `*.do`
  * 优先着找精确路径名下的 Servlet  `/abc/*`
  * 当没有路径名匹配时，找符合后缀的 Servlet `.html`
  * 最后找默认路径 Servlet `/`
  * 注意：当出现 `/*` 时，会覆盖 `/` 以及 `*.html`

####DefaultServlet

* 若 Servlet `urlPattern = "/"`  ，那么这个 Servlet 就是该 Web 应用的默认 Servlet
* 默认 Servlet：找不到匹配的 URL 就会交给默认 Servlet 处理
* Tomcat 中 `conf/web.xml` ，注册了 `DefaultServlet`
  * 当访问 Tomcat 服务器中静态文件或图片时，就是访问的是这个 `DefaultServlet`
  * 404，错误代码网页也是由 DefaultServlet 处理

```xml
 <servlet>
   	<servlet-name>default</servlet-name>
   	<servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
   	<!-- ... -->
</servlet>

<servlet-mapping>
  	<servlet-name>default</servlet-name>
  	<url-pattern>/</url-pattern>
</servlet-mapping>
```

####ServletConfig

* `<load-on-startup>` 参数 >0 表示 web **应用部署后就创建 servlet 对象，执行 init() 方法**
  * 应用：InitServlet，配置为启动时装载，为整个 Web 应用创建必要的数据库表和数据
* `<init-param>`  创建 Servlet 时传入的参数，可以在创建后通过 `ServletConfig` 拿到
  * 应用：获得字符串编码集，获得数据库连接信息
* **配置文件 web.xml，但 IDEA 中可以通过 Java Annonation 来实现**

``` xml
<servlet>
		<init-param>
            <param-name>listings</param-name>
            <param-value>false</param-value>
        </init-param> <!-- 顺序必须在 load-on-startup 之前 -->
        <load-on-startup>1</load-on-startup>
</servlet>
```

```java
@WebServlet(
        name = "MyFirstServlet",
        urlPatterns = "/servlet/first-servlet",
        loadOnStartup = 1,
        initParams = {
                @WebInitParam(name = "name", value = "Tony"),
                @WebInitParam(name = "age", value = "18")
            }
        )
public class MyFirstServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        ServletConfig servletConfig = getServletConfig();
        String name = servletConfig.getInitParameter("name");
        String age = servletConfig.getInitParameter("age");
        System.out.println("name = " + name);
        System.out.println("age = " + age);
    }
}
// name = Tony
// age = 18
```

#### ServletContext

* Web app 对应的 `ServletContext` ，**Context 域对象** ，代表整个 Web app
  * 一个 `<Context>` 对应一个 Webapp 对应一个 `ServletContext`
* 获取方式
  * `ServletContext servletContext = getServletConfig().getServletContext();`
    * `ServletConfig`  维护该引用
  * `ServletContext servletContext = getServletContext();`
    * 也是从 `ServletConfig` 拿出，只是简化了书写
* 所有Servlet 共享同一个 **ServletContext** 对象
  * 可以通过 `ServletContext` 进行 Servlet 之间通信，间接通信
* 应用：
  * 获取 Web app 初始化参数，多个 Servlet 获取同一个参数
  * 多个 Servlet 通过 `ServletContext` 实现数据共享
  * **实现 Servlet 转发（request）**

```java
// 通信

@WebServlet(name = "MyFirstServlet", urlPatterns = "/servlet/first-servlet")
public class MyFirstServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response {
        System.out.println("ServletContext:");
        ServletContext servletContext = getServletContext();
        servletContext.setAttribute("message", "Hello, second servlet.");
        System.out.println("message set by first servlet");
    }
}

@WebServlet(name = "MySecondServlet", urlPatterns = "/servlet/second-servlet")
public class MySecondServlet extends HttpServlet {
     protected void doGet(HttpServletRequest request, HttpServletResponse response) {
        ServletContext servletContext = getServletContext();
        Object message = servletContext.getAttribute("message");
        System.out.println("ServletContext:");
        System.out.println("message from servlet context: " + message);
    }
}
/*
ServletContext:
message set by first servlet
ServletContext:
message from servlet context: Hello, second servlet.
*/
```

#### IDEA 创建 Web app

1. Java Enterprise Web Application

   * Java 9.0
   * Java ee8
   * Tomcat 9.0.4

2. web 目录下，创建

   * WEB-INF
     * classes
     * lib
     * `<web.xml>` 不需要，使用注解来进行配置
       * `<web.xml>`  自动创建 `Facets =>  Deployment Descriptors => + web.xml`

3. Project Construction

   1. modules—path—out => classes
   2. modules—dependencies—lib => Jar Directory
      * 此处可以为 Tomcat lib 配置源码

4. edit Configuration ，配置 Tomcat 服务器

   * 可以选择通过创建 Tomcat 实例的方式启动服务器
     * 原理：不同的 Tomcat 实例，修改 `service.xml` 中 `<Host appBase="~/myapp">`


   * Name   port
   * Configure: Tomcat path 
   * Deployment :   `+`  artifact (打包成 war 包部署到服务器)
     * Application context  配置 url
     * 可以修改 web app 名字

#### IDEA 热部署

* 开启服务器 两个参数
  * `On update action`  `Update classes and resources`
* DEBUG 模式启动

