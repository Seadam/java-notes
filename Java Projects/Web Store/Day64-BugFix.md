# 第七天 Bug Fix

### 功能实现

* 订单支付业务

### jar 包

* 无

### Bug

* 前台，搜索商品页面搜索的商品为空时，点击尾页，页面异常

  * 原因：商品为空时，`<a href="javascript:searchPageSplit(${page.totalPageNum })">尾页</a>`
    * `page.totalPageNum` 为0，跳转到第 0 页，出错
  * 解决方案：
    * 前台
      * **searchProducts.jsp** 判断，如果 `page.totalPageNum == 0`  ==> **page.totalPageNum = 1**
    * 后台
      * 如果 `num == 0`  ==>  **num = 1**

* 下单后，返回上一页，还能再次下单

  * 防止表单重新提交

* 局域网测试 **localhost** 会导致 404

  * 全部 **localhost** 要变成实际的 **IP** 地址
  * 或者 `${pageContext.request.contextPath }/index.jsp`

* **商品库存问题（严重）**

  * 商品库存不够时，还能加入购物车，还能下单

    * 在下单时，事务处理的时候进行判断，如果库存不足，抛异常

    ```java
    if (item.getSnum() < 0) {
      throw new IllegalStateException("请输入正确商品数量...");
    }
    ```

  * 商品详情页，如果购买的商品为 **负数**，还能下单，商品库存还会增加

    * 在加入购物车页面判断，如果加入负数值，抛异常

    ```java
    if (buynum > product.getPnum()) {
      // 如果下单数量大于商品库存，无法下单
      throw new IllegalStateException("商品库存不足，请调整商品数量或者等待补充库存...");
    }
    ```

    ​













