# 第二天 管理员管理&商品管理

### 功能实现

* Admin 增、删、改、查管理
* Product 增（文件上传）
* 列表添加分页

### jar 包

* `commons-fileupload-1.3.1.jar`
* `commons-io-2.4.jar`

### 问题

* JSP 中表单 name 命名要与 product 中成员变量名一致，并于数据库中列名一致
  * JSP 中表单 name 命名没有按照驼峰命名规则，强迫症不能忍

### 额外

* AJAX 异步判断
  * 添加管理员账号是否重复
  * 添加的分类名是否存在
  * 修改的分类名是否存在
* 添加返回按钮
  * 修改管理员账号密码页面添加返回按钮
  * 修改商品页面添加返回按钮

### Bug

* 分页页面跳转 BUG
  * BUG 描述：在分页输入跳转页码进行跳转时，按回车键导致表单提交，而不是进行页码跳转

  * 解决方法一，不够完善

    * 屏蔽回车提交，
      1. 设置提交为 **type="button"**
      2. 设置一个隐藏的 **type="text"**


    * 为输入页码框注册回车按键监听，如果按下回车键，进行跳转
    * 缺陷：
      * 点击 删除 按钮后，不是直接提交表单，需要给 **button** 添加一个 **onclick** 事件
      * **onclick** 再调用表单的 **submit** 方法，麻烦

    ```javascript
    // 屏蔽回车提交
    // 设置 button
    <input type="button" value="删除" />
    // 设置隐藏 text
    <input type="text" style="display: none">
      
    // onkeypress 监听回车按键
    <input type="text" name="num" onkeypress="pressEnterJump()">
    function pressEnterJump() {
      // 13 回车键 键码
      if(13 === event.keyCode) { 
        jump(); 
      }
    }

    // 简洁版
    <input type="text" name="num"  onkeypress="if(13 === event.keyCode)jump();"/>
    ```

  * 解决方法二，不灵活

    * 给表单添加 **enter** 按键监听事件，当按下 enter 键时页面跳转

    ```html
    <form action="#" method="post"
     	onsubmit="return checkSubmit()" 
    	onkeypress="if(13 === event.keyCode){jump();return false;}">
    ```


### 小结

* 重复代码很多
  * 分页查询中，**AdminService** 和 **CategoryService** 仅仅是数据不同，但是还是要写重复代码
    * 有没有一种方式、只写一次填充数据的代码，数据通过不同的配置文件填充

