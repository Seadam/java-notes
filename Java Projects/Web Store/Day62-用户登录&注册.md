# 第五天 用户登录&注册

### 功能实现

* 用户登录、注册完善
* 邮箱验证登录
* 验证码验证登录
* 密码 MD5 存储

### jar 包

* `javax.mail.jar`
* 工具类
  * `SendJMail.java`
  * `VerifyCodeServlet.java`

### 问题

* 更新用户信息时，回显没有及时更新

  * 如果更新成功，要更新 **Session** 中 **User**
  * 拿出数据先看看 **User** 是否为空

* 邮箱验证登录

  * **ClassNotFound** 导包 `activation.jar`
  * 如果是 QQ 邮箱，需要加上如下代码，不然会导致连不上服务器

  ```java
  // 设置 QQ 邮箱 开启 SSL 加密传输，必须
  props.setProperty("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
  props.setProperty("mail.smtp.port", "465");
  props.setProperty("mail.smtp.socketFactory.port", "465");
  ```

* 密码 MD5储存时需要修改三个地方

  * 登录时，将输入密码 MD5 后与数据库比较
  * 注册时，将输入密码 MD5 后存入数据库
  * 更改时，将输入密码 MD5 后存入数据库

### 额外

* AJAX 异步请求判断注册用户名是否重复
* **UserFormBean**
  * `BeanUtils.copyProperties()`  如果类型不一致要先使用下面的方法转换
  * ` ConvertUtils.register(new DateConverter(), Date.class);`
* 用户详情页面 可以看到用户是否验证邮箱，如果没有，可以点击发送邮箱验证
  * 问题：在用户详情页点击发送邮箱验证，用户在邮箱内点击验证链接后，回显问题
    * 解决方法：当用户在点击链接跳提交验证请求后，更新 **Session** 里面的 **User** 的 **Status** 
    * 提交验证请求要在同一个浏览器，同一次会话中提交才可以

### Bug

* 输入 `书` 能查出来，输入 `书` 加空格／空串，就查不出来
  * 服务端将空格去掉
  * `String pname = "%" + condition.getPname().trim() + "%";`

### 小结

* 数据库的建表语句，以及对应的实体类问题



