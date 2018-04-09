# MySQL

#### 内容摘要

数据完整性，约束，多表设计

---

#### 补充

* sql 语句，表名，列名，不区分大小写
  * sql 语句，尽量大写
* 表中数据，区分大小写

#### 数据完整性

* 保证插入到数据库中的数据的正确性
* 实体完整性
  * 唯一性
  * 规定表中一行（每一条记录）是表中唯一的实体，独一无二的标识
    * 不能为 null
    * 不能重复
  * **表的主键** 来实现
* 域完整性
  * 规定表中一列（每条字段）必须符合特定的**数据类型或约束**
  * 比如： **NOT NULL**   **UNIQUE**
* 参照完整性
  * 在某一个表中引入的数据必须是某一个表中已经存在的数据
  * **一个表的外键与另一个表的主键对应**
  * 存储数据以及数据之间的关系
#### 约束

* 主键
  * `primary key`
  * 主键列的数据不能为 null
  * 主键列的数据不能 重复
  * 删除：`alter table t_name drop promary key;`
* 主键自动增长
  * `primary key auto_increment`
  * 只有 `key` 列才能定义 自动增长
  * 自增的列 **不显式指明插入的值， 插入 null** 会自动赋值
  * 可以显式指明插入的值
  * 不显式指明插入时，自增依赖于**该表中最大的 主键值**   （主键值 ++ ）
  * 当删除（delete from t_name）表中数据，再插入自增项时，自增依赖于删除前**该表中最大的 主键值** 
  * 当删除（drop table t_name）表，再创建时，**自增从 1 开始**
  * 自增越界问题

```mysql
create table t_user (
	id primary key auto_increment,
    name varchar(10)
);
```

* 唯一 `unique`
* 非空 `not null`
* `primary key = unique not null`
* 外键
  * **一个表的外键与另一个表的主键对应**
  * 当外键关系表中有数据时，禁止删除引用主键的数据或者引用主键所在表
  * 关键字： **constraint**   **foreign key**   **references** 
  * `CONSTRAINT fk_name FOREIGN KEY (filed_name) REFERENCES t_name (pk_name);`
    * **fk_name ** 外键名
    * **filed_name**  本表中引用外键列
    * **pk_name**  引用其他表的主键名

```mysql
create table t_refer (
    sid int,
    cid int,
    constraint student_id_fk foreign key (sid) references t_student (sid),
    constraint course_cid_fk foreign key (cid) references t_course (cid)
);
```

#### 多表设计

* 避免数据的冗余
  * 若不适用外键作为引用
    * 插入异常
    * 更新异常
    * 删除异常
* 一对一
  * 存储在同一张表中
* 一对多
  * 存储在两张或三张表中
* 多对多
  * 存储在多张表中

