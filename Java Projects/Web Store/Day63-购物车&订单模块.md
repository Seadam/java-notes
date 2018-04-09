# 第六天 购物车&订单模块

### 功能实现

* 购物车
* 订单
* 下单事务
  * 订单存数据库
  * 订单项存数据库
  * 清空购物车
  * 减商品库存

### jar 包

* 无

### 问题

* 将购物车放在 Session 中，添加或删除商品后，更新 Session 中数据

* **User**  表 **Order**  表时间问题、每次更新数据库会重置时间

  ```mysql
  ALTER TABLE `order` MODIFY `orderTime` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP;
  ```

### 额外

* 用户管理，如果用户没有登录，不能进个人中心、购物车、订单，强行跳转登录

### Bug

* ​

### 小结

* ​



