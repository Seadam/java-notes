# MVC

#### 内容摘要

概述，三层架构，框架

---

#### MVC 概述

* **Model - View - Controller**
* **Model** - 模型层
  * 数据以及基于数据的操作
* **View** - 视图层
  * 对数据的显示
* **Controller** - 控制层
  * 对 **Model** 的操作，更新到 **View**
  * 对 **View** 的输入，更新到 **Model**
  * **Model**、**View** 之间可以做到**松散耦合**
* **高内聚、低耦合**
* **解耦合**
* 将控制逻辑和表现层分离

#### 三层架构

* **MVC**  视图层
  * 获取请求参数
  * 分发、转向
  * Struts
  * SpringMVC
* **service**  业务层
* **dao**  数据访问层 / 持久化层
  * Mybatis
  * Hibernate

#### 框架

* **SSH**
  * Struts
  * Spring
  * Hibernate
* **SSM**
  * StringMVC
  * Spring
  * Mybatis











