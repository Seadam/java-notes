# DBUtils

#### 内容摘要

DBUtils 概述，DBUtils，QueruRunner， ResultSetHandler，XML，概述，语法细节，与 HTML 比较，DTD，DTD 语法细节

---

#### DBUtils 概述

Apache DBUtils

* `commons.dbutils.jar` 
* 对 **JDBC** 简单的封装，轻量级，不会影响程序性能
* 线程安全

#### DBUtils

* `closeQuiety(Connection conn)`
* `loadDriver(String driverClassName)`
* `commitAndCloseQuietly(Connection conn)`
* 简单封装

#### QueryRunner

* 简化查询代码量
* 无参构造
  * `QueryRunner()`
  * `query(Connection conn, String sql, ResultSetHandler rsh,Object... param)`
* 有参构造
  * `QueryRunner(DataSource dataSource)`  传入连接池，构造方法
  * `query(String sql, ResultSetHandler rsh,Object... param)`
* 会自动处理 **prepareStatement** 和 **resultSet** 关闭
* **Object... param**  参数中可变参数类型
  * 内部使用 **PrepareStatement** 进行查询的，该参数是指查询参数
* 增、删、改
  * `update(String sql)`
* 查
  * `query(String sql, ResultSetHandler rsh)`
* 批处理
  * `batch(String sql, Obejct[][] param)`
    * 二维数组，每一行是 SQL 语句 `?` 替换值
* **Bean** 类注意事项
  * **Bean** 要有无参构造
  * **Bean** 成员变量名要与数据表列名一致
  * **Bean** 要有 **get / set**

```java
// 使用 QueryRunner
public void test() throws SQLException {
  QueryRunner queryRunner = new QueryRunner(C3P0DataSource.getDataSource());
  List<User> lists = queryRunner.query("SELECT * FROM t_user", new BeanListHandler<>(User.class));
  System.out.println("lists = " + lists);
}
```

#### ResultSetHandler

* **BeanHandler<T>(t.class)**
  * 返回结果集**第一行**转化为 **Bean**


* **BeanListHandler<T>(t.class)**
  * 返回结果集**所有行**转化为 **List<Bean>** 列表
  * **Bean** 要有无参构造
  * **Bean** 成员变量名要与数据表列名一致
  * **Bean** 要有 **get / set**

* `ArrayHandler`

  * 返回结果集**第一行**数据放在 **Object[]** 数组
  * 通过排序语句，可以拿到结果集最大、最小值

* `ArrayListHandler`

  * 返回结果集所有数据放在 **List<Object[]>** 列表

* `ColumnListHandler`

  * 返回结果集某一列 **List<Object>** 列表

* `KeyedHandler`

  * `Map<key, Map<String, Object>>`
  * 第一列的值作为为 **key**，对应行所有数据作为为 **Map<String, Object>**
  * **Map** 中，列名为 **key**，值为 **Object** 

* `MapHandler`

  * 返回结果集**第一行**数据放在 **Map<String, Object>**

* `MapListHandler`

  * 返回结果集每一行作为 **Map<String, Objcet>** 放到 `List<Map<String, Object>>` 列表

* `ScalarHandler`

  * 返回结果集第一行第一列 **Object**
  * 应用：拿到统计函数结果

  ```java
  // 返回统计函数结果
  Object query = queryRunner.query("select count(*) from t_user", new ScalarHandler());
  ```



## XML

#### 概述

* **XML：eXtensible Markup Language**
* **标记语言，宗旨是传输数据，用户自定义标签**
* 遵循 **W3C** 于 2000 年发布 **XML1.0** 规范
* 可以描述**有关系的数据**，通用的数据交换格式
* 目前用于配置文件
  * 传统配置文件 `.properties`
  * 比如描述程序模块间关系

#### XML 语法

* 文档声明
  * **必须出现在文档第一行**
  * `<?xml version='1.0' encoding='utf-8'?>`
  * **声明编码格式要与文件保存编码一致**
* 元素
  * 开始标签和结束标签不能省略
    * `<tag>Something</tag>`
    * `<tag/>`  没有内容的标签
  * 不允许交叉嵌套
  * **必须有且仅有一个根标签**
  * **不会忽略主体内容中出现的空格和换行**  当作文本节点
  * 命名规范，标签名
    * 字母，数字，减号，下划线，英文句点
    * **严格区分大小写**
    * 只能以字母或下划线开头
    * 不能以 xml 开头，保留字段
    * 字符之间不能有空格或字表符
    * 字符之间不能使用冒号 `:` 
* 属性
  * 属性名命名规范同元素的命名规
  * 属性值要以引号（单引号或双引号）包裹起来
  * 属性不允许重复
  * 元素的属性所代表的信息，也可以改成子元素的形式
* 注释
  * `<!-- comment -->`
  * 不能嵌套注释
* CDATA 区
  * `<![CDATA[<tag>会忽略解析<tag>标签</tag>]]>`
* 特殊字符
  * `&  &amp;`
  * `<  &lt;`
  * `>  &gt;`
  * `"  &quot;`
  * `'  &apos;`
* 处理指令，类似文档声明 `<?xml version='1.0' encoding='utf-8'?>`


```xml
<?xml version='1.0' encoding='utf-8'?>
<Tag>Something</Tag>
<Tag/>

<MyTag name="hell"></MyTag>

<!-- 元素的属性所代表的信息，也可以改成子元素的形式 -->
<user username="root" password="root"/>
<user>
	<username>root</username>
  	<password>root</password>
</user>

<![CDATA[<tag>会忽略解析标签</tag>]]>
```

#### XML 与 HTML

|          | XML             | HTML              |
| -------- | --------------- | ----------------- |
| **数据**   | 传输数据，描述数据，配置文件  | 显示数据              |
| **大小写**  | 区分大小写           | 不区分大小写            |
| **空格**   | 不会忽略空格          | 会过滤空格             |
| **语法严格** | 属性值必须加引号，标签必须闭合 | 属性值可以不加引号，标签可以不闭合 |
| **标签**   | 没有预定义标签，用户自定义   | 只能使用预定义标签         |
| **根结点**  | 有且只有一个          | 可以有多个             |

#### XML 约束

* 编写文档约束 XML 书写规范
* 遵循约束约束文档的 XML（语法符合规范，符合约束）
* 规定了元素名称、属性、及出现的顺序
* 常见约束
  * **XML DTD**  W3C 
  * **Schema**  W3C 
  * XDR, SOX

#### DTD

* `.dtd`  必须以 **utf-8** 编码保存

* 引入外部 DTD 文档

  * `<!DOCTYPE 根元素 SYSTEM "DTD 文档路径">`
  * `<!DOCTYPE 根元素 PUBLIC "DTD 文档名称" "DTD 文档 URL 路径">`

  ```xml
  <!DOCTYPE books SYSTEM "books.dtd">

  <!DOCTYPE html PUBLIC "-/w3c/dtd html 4.01/en" "http://www.w3.org/tr/html4/strict.dtd">
  ```

#### DTD 语法细节

* 定义元素

  * `<!ELEMENT 元素名称 使用规则>`
  * 使用规则
    * **(#PCDATA)**
      * (*Parsed Character Data*)
      * 元素的主体内容只能是普通的文本
    * **EMPTY**
      * 指示元素的主体为空  比如 `<br/>`
    * **ANY**
      * 指示元素的主体内容为任意类型
    * **(子元素)**
      * 指示元素中包含的子元素
      * 定义子元素及其关系
        * `,` 分开，必须按照声明顺序添加子元素
        * `|` 分开，任选其一， 只能选一个
        * `+`  `*`  `?`  表示元素出现的次数
          * 如果没有声明，默认必须且只能出现一次
          * `+`  至少出现一次，一次或多次
          * `*`  任意次，零次，一次或多次
          * `?`  只能一次或零次

  ```dtd
  <!ELEMENT file (titile,author,email)>
  <!ELEMENT file (titile|author|email)>
  <!ELEMENT myfile ((title*, author?, email)* | comment)>
  ```

* 定义属性

  * `<!ATTLIST 元素名 属性名1 属性值类型 设置说明...>`
    * 属性值类型
      * **#CDATA**：表示属性的取值为普通的文本字符串
      * 枚举：只能从枚举列表中任选其一，如(鸡肉|牛肉|猪肉|鱼肉)
      * **#ID**：表示 属性的取值不能重复
    * 设置说明
      * **#REQUIRED**：**必须出现**
      * **#IMPLIED**：**可有可无**
      * **#FIXED**：**固定值**，语法：#FIXED "固定值"
      * **直接值**：**默认值**

  ```dtd
  <!ATTLIST file name #CDATA #REQUIRED>
  ```

  ​















