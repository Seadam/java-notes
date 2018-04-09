# 第一天 分类管理

### 功能实现

* **Category** 增、删、改、查、批量删除操作

### jar 包

* `c3p0-0.9.1.2-jdk1.3.jar`
* `commons-beanutils-1.8.3.jar`
* `commons-dbutils-1.4.jar`
* `commons-logging-1.1.1.jar`
* `javax.servlet.jsp.jstl-1.2.1.jar`
* `javax.servlet.jsp.jstl-api-1.2.1.jar`
* `mysql-connector-java-5.1.44-bin.jar`

### 问题

* **Category** 的 **id** 字段是自增项，删掉其中某 **Category** 后，再添加新项，编号会出现空缺
  * `TRUNCATE table category;`  重置表自增字段

### 额外

* **Category** 提交前判断
  * 是否为空，空不提交到服务器，提示用户 “没有输入”
  * 是否和原值相同，相同不提交到服务器，提示用户 “没有修改”
* **Category** 页面提供返回按钮，回到 **categoryList.jsp**
  * `onclick="history.go(-1)"`
* **categoryList.jsp** 复选框全选问题
  * 当每个复选框（除全选）全部选中时，全选应该被选中
* **categoryList.jsp** 当没有选择 批量删除项 而提交删除时，可能引发空指针
  * 前端判断是否选择复选框
  * 后端验证请求参数是否为空

### Bug

* **categoryList.jsp** 页面中，全选删除时，由于全选复选框在 **form** 表单中，会被当作参数传递到服务器，会造成数据库删除到脏数据（或者是数据库没有的 **cid** 字段），解决如下
  * 第一种：后端
    * 遍历请求参数时，过滤全选复选框，不通过数据库删除该字段
  * 第二种：前端
    *  **form** 表单不要包含全选复选框
  * 第三种：前端（采用这个方法最佳）
    * 修改 jsp 页面中 复选框（除全选框） name 为 cid

### 小结

* 重复代码很多
  * **CategoryDao** 层数据库操作，**QueryRunner** 的创建重复，仅 SQL 语句和返回值不同
  * **CategoryServlet** 拿请求参数，向浏览器写入反馈信息重复