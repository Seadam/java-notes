# MySQL

#### 内容摘要

DQL，连接查询，查询过程，子查询，联合查询，报表查询，统计函数，数据库备份与恢复，优化总结

---

#### DQL

复杂查询（多表查询），多表设计

* 连接查询
  * 交叉连接（cross join）
  * 内连接（inner join）
  * 外连接（out join）
    * 左外连接（left out join）
    * 右外连接（right out join）
* 子查询（嵌套查询）
* 联合查询（集合操作）union
* 报表查寻（统计相关）group by

#### 连接查询

* `SELECT * FROM t_name join_type link_name [join_condition][where_condition]`
* **`[join_condition]`**
  * `ON link_name.id = t_name.id`
  * 符合 **[link_name]** 中 **[link_name.id]** 属性 符合 **[t_name.id]** 连接条件结果


* 交叉连接 **CROSS JOIN**

  * `select * from t_name CROSS JOIN link_name`
    * 将 link_name 表与 t_name 表组合
  * 不带 **on** 子句，无条件连接，返回数据行的**笛卡尔积**
  * **笛卡尔积**：综合两张表信息（表结构、数据），两两组合
    * 并不关心数据的正确性，只是简单将表组合
  * **隐式写法：`select * from t_name, link_name;`**

  ```mysql
  -- 交叉链接，笛卡尔积
  select * from stu CROSS JOIN score;
  -- 隐式写法
  select * from stu, score;
  ```


* 内连接 **INNER JOIN**

  * `select * from t_name INNER JOIN link_name [join_condition] [where_condition]`
    * `.` 访问数据表的列名数据
    * 符合 **[join_condition]** 连接条件，符合 **[where_condition]** 过滤条件的结果
  * 带 **on** 子句，返回 交叉链接 中**符合连接条件**的结果
  * **隐式写法：`select * from t_name, link_name WHERE [join_condition]`**

  ```mysql
  -- 符合连接条件
  select * from stu INNER JOIN score ON score.sid = stu.id;
  -- 符合连接条件以及过滤条件
  select * from stu INNER JOIN score ON score.sid = stu.id where english > 200;
  -- 隐式写法
  select * from stu, score WHERE score.sid = stu.id;
  ```

* 外链接  **OUTER JOIN**

  * **含内连接结果**，指定结果显示某一个表（左或右表）**符合查询条件不符合连接条件** 所有数据
  * 左外连接
    * `select * from left_name LEFT OUTER JOIN link_name [join_condi] [where condi]`
    * 内连接结果 + **left_name** 表中 符合 **[where_condition]** 不符合 **[join_condition]** 的全部数据

  ```mysql
  -- 返回 stu 符合查询条件和不符合连接条件的所有数据
  select * from stu LEFT OUTER JOIN score ON score.sid = stu.id;
  ```

  ```mysql
  -- 可以自己跟自己外连接
  -- 对于dept表中，列出所有部门名，部门号，同时列出各部门工作为'CLERK'的员工名与工作
  -- 详见 Day 49 作业 homework 2 9题
  select (select dname from dept where dept.deptno = a.deptno) as dname, a.deptno, b.ename, b.job from emp as a left outer join emp as b on b.job = 'clerk' and a.empno = b.empno order by deptno; 
  ```

  * 右外连接
    * `select * from t_name RIGHT OUTER JOIN right_name ON right_name.id [where condition]`
    * 内连接结果 + **right_name** 表中 符合 **[where_condition]** 不符合 **[join_condition]** 的全部数据

  ```mysql
  -- 返回 score 符合查询条件和不符合连接条件的所有数据
  select * from stu RIGHT OUTER JOIN score ON score.sid = stu.id;
  ```

* 三者比较

  * 交叉连接
    * 结果包含全部笛卡尔积
  * 内连接
    * 符合 **[join_condition]** 符合 **[where_condition]** 交叉连接结果
  * 外连接
    * 内连接结果 + 某表符合 **[join_condition]** 不符合 **[where_condition]** 全部数据

* 连接操作**优化**

  * 连接操作是**关系型数据库中最费时**的操作
  * 两个表连接时，只有一个表需要 **[where_condition]** 过滤，则在 **[join_condition]**  进行过滤效率高###


#### MySQL 查询过程

* **WHERE** 子句
  * 将表中每一行数据看作一个整体，扫描每一行判断 **[where condition]**  true 则加入结果表，否则过滤
* **JOIN** 子句
  * 预处理，Mergesort， 针对两张表先进行排序，再判断 **[join_condition]**

#### 子查询

* 嵌套查询

* 不相关子查询

  * 子查询中查询条件与父查寻没有相关性，**两张不同的表**
  * `select * from table_one where id = (select id from table_two [where_condition]);`
  * 只能查询 table_two 中一列数据
  * 在 **where** 子句或 **from** 子句 再嵌入 **select** 子句，一般写在 where 子句

  ```mysql
  -- 关键点在两张不同的表
  -- 查询没有参加考试的学生
  select * from stu where id not in (select sid from score);
  -- 可以拆分为下面两句
  	-- 先从 score 选出所有有成绩的 sid 信息
  	select sid from score;
  	-- 在判断所有 sid 中不在 成绩表中的学生
  	select * from stu where id not in (...);
  ```

*  相关子查询

  * 子查询中查询条件与父查寻有相关性，在**同一张表**嵌套查询

  ```mysql
  -- 操作同一张表，具体看 Day 49 homework 2 10
  -- 对于工资高于本部门平均水平的员工，列出部门号，姓名，工资，按部门号排序
  select deptno, ename, sal as salary from emp as a where sal > (select avg(sal) from emp as b where b.deptno = a.deptno group by deptno) order by deptno;

  -- 11
  -- 对于emp，列出各个部门中工资高于本部门平均工资的员工数和部门号，按部门号排序
  select deptno, count(*) as empCount from emp as a where sal > (select avg(sal) from emp as b where b.deptno = a.deptno group by deptno) group by deptno order by deptno;
  ```


#### 联合查询  UNION

* 操作的对象是表，将表视作集合，**并集**


* 能够合并两条查询语句的查询结果，**合并结果表**

* 去掉其中的重复数据行，**去重结果表**

* 关键字 **UNION**

* `select * | column from t_name UNION select * | column from t_name;`

  * 表结构相同，即同一张表
  * 返回结果列也要一致
  * 并集

  ```mysql
  -- 并集
  select * from stu where age > 18 UNION select * from stu where sex = '男';
  ```

#### 报表查询  GOURP BY

* **分组  与统计函数配合使用** （见下）

* 关键字 **GOURP BY**

* `select * from t_name [where_condition] [GOURP BY column [having]] [order by column];`

  * 单独使用 **group by** ，查询结果**以组为单位**，默认只显示每组中第一条记录

  ```mysql
  -- 根据性别进行分组，统计人数
  select count(*), sex from stu group by sex;
  ```


#### 统计函数

* `count()`
* `sum()`
* `avg()`
* `max()`
* `min()`

#### 合计

* `count(column_name)`

  * 返回某一列，行的总数

  * `select COUNT(*) | COUNT(column_name) from t_name [where_condition]`

  ```mysql
  -- 统计所有学生人数
  select count(*) from stu;
  ```

* `sum(colomn_name)`

  * 返回某一列，行的数据之和
  * 仅对数值列，否则会报错

  * `select SUM(column_name) from t_name [where_condition]`

  ```mysql
  -- 求单列和，并改别名
  select sum(english) english from score
  -- 求多列和
  select sum(english) english, sum(history) history from score;
  -- 求总数两种方式
  select sum(english + chinese + history) from score;
  select sum(english) + sum(chinese) + sum(history) from score;
  ```

* `avg(column_name)`

  * 返回某一列，行的数据的平均值
  * `select AVG(column_name) from t_name [where_condition]`

  ```mysql
  -- 求平均值
  select avg(english) from score;
  -- sum() / count()
  select sum(english) / count(english) from score;
  ```

 * `max(column_name)` / `min(column_name)`

      * 返回某一列，行的数据的最大最小值
      * 可以对字符串进行比较，依赖字符比较集
      * `select max(column) from t_name [where_condition]`

   ```mysql
   select max(english) from score;
   ```

#### 分组／统计

*  **group by** 与 统计函数结合使用

   ```mysql
   -- 根据性别进行分组，统计人数
   select count(*), sex from stu group by sex;
   ```

*  **HAVING** 

   *  分组条件
   *  `select * from t_name [GOURP BY column [HAVING function]]; `
   *  **having** 针对一组数据（多行数据），可使用统计函数
   *  **where** 针对一行数据，不能使用统计函数

   ```mysql
   -- 统计年龄大于 16 且人数大于 1 的男生和女生数量
   select count(*),sex from stu where age >16 group by sex having count(*) >1 ;
   -- 统计函数，可以取别名
   select count(*) total from stu;
   ```

#### 数据库备份与恢复（恢复逻辑数据 sql 语句）

* 备份单位是 数据库 (database)
* 备份
  * **shell 命令行** `$ mysqldump -uroot -p db_name>~/db_name_backup`
* 恢复
  * 方法一
    * 先创建一个空数据库 `db_name` 
    * **shell 命令行** `mysqldump -uroot -p db_name<~/db_name_backup`
  * 方法二
    * 创建一个空数据库 db_name，并进入
    * **mysql 命令行** `mysql> source ~/db_name_backup.sql;`

#### 优化总结

1. 大小写
   * 大写
2. 连接条件，过滤条件
   * 若只有一张表需要过滤，用连接条件替代过滤条件
   * 原理：先判断连接条件，会筛选一遍数据，再连接，效率高一些
     * 否则，先连接，再过滤会，数据太多，效率低
3. select * 与 select 列名
   * **select column**  效率比 **select *** 高

