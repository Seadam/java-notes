# SQL

#### 内容摘要

数据库概述，数据库架构，常用数据库元件，数据库存储，SQL，DDL，DML，DQL，乱码问题，常见数据类型

---

#### 数据库

* 存储大量数据到硬盘中
* Java 中存储数据到硬盘的方式
  * 序列化（依赖 JVM 反序列化）
  * 文件 （读写需要制定规则，没有强制性）
* 数据库的优势
  * 有**通用的规则**：关系型数据库
  * 不依赖于平台达到数据共享
  * 保证数据的**一致性**和**完整性**（**约束**）
  * 提供了事务处理（数据库提供多线程处理）

####  数据库架构

* 数据库管理软件（DBMS, DataBase Manager Server）：MySQL，SQLServer
  * 服务
    * 对数据的 CRUD 操作的接口
    * 维护数据库的逻辑结构
  * 逻辑结构
    * 数据库
      * 数据表
        * 数据
* 客户端
  * 通过 **Socket** 连接基于 **MySQL** 协议发送操作数据的指令
  * **SQL 语言 （Structure Query Language）**结构化查询语言，只针对关系型数据库
    * **DDL Data Defination Language** 管理数据库和数据表的结构
    * **DML Data Manipulation Language** 操作数据
    * **DCL Data Control Language** 
      * **约束数据，控制数据的正确性**
      * **事务处理**
      * 权限管理，定义存储过程（类似 方法）
    * **DQL Data Query Language**  数据查询（非官方分类）
* 关系型数据库
  * 数据模型就是关系（二维表）
  * 严格的数学（基于**关系模型**的公理系统）
* 常见名词
  * 数据库管理系统：**DBMS**
  * 数据库应用：使用数据的应用，**DBMS**，物理数据，**DBA**(DataBase Administror)
#### 常用数据库软件
* **Oracle** oracle  专注于数据 银行、证券（服务）
* **DB2** (IBM) 数据一致性
* **Informix**  oracle 收购，专注于事物处理
* **Sybase** power designer 设计数据库软件
* **SQL Server** 2000 / 2005 Windows
* **MySQL** 开源／免费  orcle 收购
* **Access** Office
* **SQLite**  小巧 (Android)


#### 数据库存储

数据库 ==> 数据表 ==> 数据 ==> 行 ==> JavaBean

#### SQL

* 数据库：增删改查 **DDL**
* 数据表：增删改查 **DDL**
* 数据：增删改查 **DML**
* **每一行指令必须要有 `;`**
* **不区分大小写**
* **utf8**

#### My SQL DDL 数据库

* 创建数据库和数据表结构

* 关键字：**CREATE** **ALTER** **DROP** **TRUNCATE**  **show**  **use**

* `create database`

  * 创建数据库
  * `CREATE {database | schema} [IF NOT EXISTS] db_name [create_specification];`
    * **create_specification**
      * `CHARACTER SET charset_name`  指定字符集
      * `COLLATE collation_name`  指定数据库**字符集比较方式**
        * 校对规则：字符集中字母比较大小规则

  ```mysql
  # 创建一个默认字符集，默认字符集比较方式的数据库
  mysql> create database first_database;

  # 指定字符集为 gbk
  mysql> create database gbk_database CHARACTER set gbk;
  # show create database gbk_databsse
  | CREATE DATABASE `gbk_database` /*!40100 DEFAULT CHARACTER SET gbk */ |

  # 指定字符集为 utf8，并设置字符集比较方式为 utf8_bin
  mysql> create database second_database character set utf8 collate utf8_bin
  # show create database second_database
  | CREATE DATABASE `second_database` /*!40100 DEFAULT CHARACTER SET utf8 COLLATE utf8_bin */ |

  # show databases
  +----------------------+
  | Database             |
  +----------------------+
  | information_schema   |
  | first_database       |
  | gbk_database         |
  | mysql                |
  | performance_schema   |
  | sys                  |
  +----------------------+
  ```

* `show database`

  * 显示数据库
  * `SHOW databases` 显示所有数据库
  * `SHOW CREATE DATABASE db_name` 查看创建数据库语句
  * `select database()` 显示当前所用数据库

  ```mysql
  show databases;

  mysql> select database();
  +----------------+
  | database()     |
  +----------------+
  | first_database |
  +----------------+
  ```


* `use database` 

  * 使用数据库
  * `USE database db_name` 进入数据库

  ```mysql
  use database first_database;
  ```

* `drop database`

  * 删除数据库
  * `DORP DATABASE [IF EXISTS] db_name`

  ```mysql
  drop database first_database;
  ```

* `alter database`

  * 修改数据库
  * `ALERT DATABASE [IF NOT EXISTS] db_name  [create_specification];`
    * 修改指定数据库 字符集或字符集比较方式
    * **create_specification**
      * `CHARACTER SET charset_name`  指定字符集
      * `COLLATE collation_name`  指定数据库**字符集比较方式**

  ```mysql
  alter database gbk_database character set utf8;
  ```

#### DDL 数据表

* `create table`
  * 创建数据表
  * `create table t_name (filed type, filed type, filed type) [create_specification];`
  * 最好写在一个 txt ，写好再复制进去
  * 根据数据类型定义相应的列类型
  * 数据表命名推荐 `t_` 开头
  * **注意事项：**
    * 以 `( )` 包裹建表语句
    * 以 `,` 分隔建表语句项
    * 最后一句不需要 `,`
    * 以 `;` 结尾

  ```mysql
  create table employee (
      id int,
      name varchar(20),
      gender char(4),
      birthday date,
      Entry_data date,
      job varchar(30),
      salary float,
      resume text
  );

  mysql> show create table employee;
  | employee | CREATE TABLE `employee` (
    `id` int(11) DEFAULT NULL,
    `name` varchar(20) DEFAULT NULL,
    `gender` char(4) DEFAULT NULL,
    `birthday` date DEFAULT NULL,
    `Entry_data` date DEFAULT NULL,
    `job` varchar(30) DEFAULT NULL,
    `salary` float DEFAULT NULL,
    `resume` text
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
  ```

* `des`
  * 查看表数据类型详情
  * `desc t_name`

  ```mysql
  mysql> desc employee;
  +------------+-------------+------+-----+---------+-------+
  | Field      | Type        | Null | Key | Default | Extra |
  +------------+-------------+------+-----+---------+-------+
  | id         | int(11)     | YES  |     | NULL    |       |
  | name       | varchar(20) | YES  |     | NULL    |       |
  | gender     | char(4)     | YES  |     | NULL    |       |
  | birthday   | date        | YES  |     | NULL    |       |
  | Entry_data | date        | YES  |     | NULL    |       |
  | job        | varchar(30) | YES  |     | NULL    |       |
  | salary     | float       | YES  |     | NULL    |       |
  | resume     | text        | YES  |     | NULL    |       |
  +------------+-------------+------+-----+---------+-------+
  ```

* `alter table`
  * 修改表

  * **ADD**
    * 添加新数据列
    * `alter table t_name ADD column_name column_type`

    ```mysql
    mysql> alter table employee age int;
    | age        | int(11)     | YES  |     | NULL    |       |
    ```

  * **CHANGE**
    * 修改某一列数据类型，还可以修改列名
    * `alter table t_name CHANGE old_column_name new_column_name new_column_type`

    ```mysql
    # 改数据类型 + 改列名
    mysql> alter table employee change gender sex boolean;
    | sex        | tinyint(1)  | YES  |     | NULL    |       |
    ```

  * **MODIFY**
    * 修改某一列数据类型
    * `alter table t_name MODIFY column_name new_column_type`

    ```mysql
    mysql> alter table employee modify id tinyint;
    | id         | tinyint(4)  | YES  |     | NULL    |       |
    ```

  * **DROP**
    * 删除表中的一列
    * `alter table t_name DROP column_name`

    ```mysql
    mysql> alter table employee drop age;
    ```

  * **CHARACTER SET**
    * 修改表的字符集
    * `alter table t_name CHARACTER SET charset_name`

    ```mysql
    mysql> alter table employee character set gbk;

    # show create table employee;
    | employee | CREATE TABLE `employee` (
      `id` tinyint(4) DEFAULT NULL,
      `name` varchar(20) CHARACTER SET utf8 DEFAULT NULL,
      `sex` tinyint(1) DEFAULT NULL,
      `birthday` date DEFAULT NULL,
      `Entry_data` date DEFAULT NULL,
      `job` varchar(30) CHARACTER SET utf8 DEFAULT NULL,
      `salary` float DEFAULT NULL,
      `resume` text CHARACTER SET utf8
    ) ENGINE=InnoDB DEFAULT CHARSET=gbk |
    ```

* `rename table`
  * 修改表名
  * `rename t_name to new_name`

  ```mysql
  mysql> alter table employee rename to user;
  # mysql> show tables;
  +--------------------------+
  | Tables_in_first_database |
  +--------------------------+
  | user                     |
  +--------------------------+
  ```

* `show create table t_name`
  * 查看建表语句
  * `show tables` 查看已有表

  ```mysql
  show create table employee;
  show tables;
  +--------------------------+
  | Tables_in_first_database |
  +--------------------------+
  | user                     |
  +--------------------------+
  ```

* `drop table`
  * 删除表
  * `drop table t_name`

  ```mysql
  mysql> drop table user;
  # mysql> show tables;
  Empty set (0.00 sec)
  ```

#### DML

* 用于向数据库表中，插入，修改，删除

* 常用关键字： **INSERT** **UPDATE** **DELETE**

* `insert into`
  * 向表中插入一行数据
  * `()` 不能省略
  * `insert into t_name(column, column...) values (value, value...);`
    * 数据类型要相同
    * 待插入数据大小应在列的规定范围内
    * 字符串和日期要用单引号 `''` ，日期不足两位要补零 `'1992-2-3' ==> '1992-02-03'`
    * 插入空值，不指定或者 `null`
  * `insert into t_name values(value, value...);`
    * 向所有列赋值
  * 若在命令行使用 MySQL 插入中文数据，Microsoft 下命令行 中文 编码方式为 GBK，而数据库默认编码位 utf-8，会导致插入乱码，需要 `set names gbk;` 解决，具体见下乱码问题

  ```mysql
  # 向所有列填充一行数据
  mysql> insert into user values(0,'中文名','男','1998-01-02','2018-2-27','java developer',9999.99,'你好关系型数据库');
  # 向指定列填充一行数据
  mysql> insert into user(id,name,salary) values(1,'tony',123.01);
  ```

* `update ... set`
  * 修改表中列的值
  * `update t_name SET column_name=value `
    * 修改 列名为 column_name 整列的数据 为 value
  * `update t_name SET column_name=value WHERE [where-condition] `
    * 修改 列名为 column_name `where-condition` 定位的所在行 的数据 为 value
  * `update t_name SET column_name=value,column_name=value,... WHERE [where-condition] `
    * 可以修改多列的值
  * `update t_name SET column_name=column+value WHERE [where-condition]`
    * 修改，其值在原有基础上增加 value

  ```mysql
  mysql> update user set gender='男' where id=1;
  mysql> update user set birthday='2000-12-31',Entry_data='2018-02-28' where name='tony';
  mysql> update user set salary = salary + 2000 where name = 'tony';
  ```

* `delete form`
  * 删除表中行
  * `delete from t_name`
    * 删除表中所有数据，只能删除记录，表还在
  * `delete from t_name WHERE [where-condition]`
    * 删除表中符合条件的某一行

  ```mysql
  mysql> delete from user where id = 0;
  ```

* `select`

  * 见下 DQL

#### DQL

* **所有查询结果都是一张表，查询结果实质上是源数据的副本**
* 参与表达式运算的数据有 null 参与则结果必为 null
  * 解决方法：
    1. 插入数据前为表中的列设置默认值
    2. `ifnull(column,0)` 函数，若为空，则为 列名为 column 的数据设置为 0，仅 MySQL 才有


* 单表查询 `select ... form`
  * `select * from t_name`
    * 查询所有列
  * `select column1, column2,… FROM t_name WHERE [where-condition]`
    * t_name 表中查询列名为 column 的数据，也可以定位 [where-condition] 行 
  * `select DISTINCT * from t_name`
    * **DISTINCT** 指定，过滤重复行
  * `select colomn1+value, colomn2+value c from t_name`
    * 可以对查询结果使用表达式运算
    * colomn 列查询结果加 value 值再显示，**并没有修改源数据**
  * `select column as name from t_name`
    * **as 可以省略**
    * 取别名再显示

* `where`
  | 运算符                 | 说明                              |
  | ------------------- | ------------------------------- |
  | <, >, <=, >=, =, <> | 小于，大于，小于等于，大于等于，等于，不等于          |
  | BETWEEN…AND         | 某个区间，闭区间                        |
  | IN(set)             | 符合某个列表中的值， 如 in(999, 997)       |
  | LIKE % / _          | 模糊匹配， % 代表 0 个或多个任意字符， _ 代表一个字符 |
  | IS NULL             | 判断是否为空                          |
  | AND                 | 与                               |
  | OR                  | 或                               |
  | NOT                 | 非                               |

* `order by`
  * `select * from t_name order by column;`
    * column 可以是列名 也可以是**别名**
  * **asc** 升序 **desc** 降序，默认升序
  * 对查询结果进行排序

  ```mysql
  mysql> select math + chinese + english as score from student order by score desc;

  # 可以设置多种排序方式，按顺序匹配
  # 下面语句：查询全体学生情况，查询结果按所在系的系号升序排列，同一系中的学生按年龄降序排列
  mysql> select * from S order by Depa, age desc;
  ```

#### MySQL 中文乱码问题

* **client**
  * 服务器端认为客户端所使用的字符集
* **connection**
  * 连接数据库的字符集设置
* **database**
  * 某个逻辑数据库所用字符集
* **result**
  * 数据库返回给客户端时所用字符集
* **server**
  * 服务器安装时指定默认字符集
* **system**
  * 数据库系统使用的字符集
* 一次数据库请求响应流程
  * **`client`** => **`connection`** => (Server、System、database) => **`connection`** => **`result`**
  * 乱码问题容易出现在 **client** 、**connection** 、**result**
  * `set names gbk;` 可以直接设置 **client** 、**connection** 、**result** 三者字符集
  * 故可以根据实际需要 使用 `set names` 来解决

#### SQL 常见数据类型

* int 类型：**tinyint**, **smallint**, **int**, **bigint**
* float 类型: **float**, **double**
* 字符串类型：**char**, **varchar**, **blog**, **longblog**, **text**, **longtext**
* 注意：
  * 字符串和日期必须要用**单引号包裹**
  * bit 类型的数据，保存的是二进制值，但是当显示在表中数据项时，会显示为 具体字符集所对应的字符
    * 比如默认为 utf8 的表中，存入 97 bit 的数据，显示在表中的是 小写字母 a
    * bit 可以 通过 SQL 函数计算返回具体数值
      * bin(bit) 返回二进制  
      * oct(bit) 返回八进制
      * hex(bit) 返回十六进制
      * `select bit + 0 from t_name`   bit + 0  返回十进制

| 分类   | 数据类型           | 说明                                       |
| ---- | -------------- | ---------------------------------------- |
| 数值   | bit(m)         | 比特，位，m 指定位数，默认值为1，范围 1～ 64               |
|      | bool, boolean  | 使用 0 或 1 表示假或真                           |
|      | tinyint        | 1字节，8位，有符号范围 -128～127，无符号 0 ～ 255        |
|      | smallint       | 2字节，16位                                  |
|      | mediumint      | 3字节，                                     |
|      | int            | 4字节，32位                                  |
|      | bigint         | 8字节，64位                                  |
|      | float(m, d)    | 4字节，m 指定显示长度，d 指定小数位数                    |
|      | double(m, d)   | 8字节，m 总个数，d 小数位，经过多次计算会有精度丢失             |
|      | decimal        | 同 double，进过多次计算，精度不会丢失，把数据当作字符串存储        |
| 字符串  | char (size)    | 固定长度字符串，size **固定**字符长度，即使长度小于 size 也会占用 size 大小 |
|      | varchar (size) | 可变长度字符串，size **最大**字符长度，只会占用实际字符串长度大小    |
|      | blog           | 二进制数据                                    |
|      | longblog       | 二进制数据                                    |
|      | text           | 大文本数据                                    |
|      | longtext       | 大文本数据                                    |
| 时间   | year           | 1字节，年份，'2018'                            |
|      | date           | 3字节，日期，'2018-02-27'                      |
|      | time           | 3字节，时间，'20:47:33'                        |
|      | datetime       | 8字节，日期时间，'2018-02-27 20:47:33'           |
|      | timestamp      | 4字节，自动存储记录修改的时间                          |

