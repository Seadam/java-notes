# 第三天 用户管理&后台商品搜索

### 功能实现

* User 增、删、改、查管理
* 商品搜索
* 后台登录

### jar 包

* 无

### 问题

* 分类类名随处都需要用到，可以放到内存中
  * 当第一次加载时，读取放入 **ServletContext** 域中 `loadOnStartup = 1`
  * 加一个过滤器，每次进行增、删、该、改操作后，更新域中数组
* 多条件 SQL 查询
  * SQL 语句拼接
    * `String sql = "select * from product where true ";`
    * 商品名采用 **like** 模糊查询
    * `sql += "and pname like ? ";param.add("%" + condition.getPname() + "%");`
  * SQL 语句参数 
    * 可变参数列表，构造一个 **List<Object> param**
    * 再通过 **param.toArray()** 转化为数组传入参数重

### 额外

* 将查找条件参数封装到 **SearchCondition** 类
* AJAX 异步判断
  * 添加注册用户用户名是否重复

### Bug

* 修改商品详情时，如果不上传新图片，更新旧图片路径出错
  * 提交请求时，传入旧图片路径，如果没有上传新图片，保存旧图片路径

* 点击编辑商品，商品分类的下拉菜单从第一个开始显示，没有正确显示当前商品的分类

  ```jsp
  <option value="${category.cid}" 
          <c:if test="${category.cid == product.cid}"> 
  			selected
  		</c:if>
  >${category.cname}</option>
  ```

* 搜索列表编辑或删除后跳转出错，没有跳转回操作前页面
  * 未解决

* 删除分类，若该分类下还有商品导致不能删除，提示信息还未完善

  * 未解决

### 小结

* 重复代码
* **Jsp** 要特别注意，小心 **domain** 类成员变量名称不同的问题









