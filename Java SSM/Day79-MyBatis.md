# MyBatis
#### 内容摘要

MyBatis 注解，SQL 语句构建器，MyBatis 优化，懒加载，一级缓存，二级缓存，逆向工程

------

### MyBatis 注解

* 不适用于多表查询，**resultMap**
* 使用接口方法的参数作为 **parameterType** ，返回类型作为 **resultType**

**FILED** `@Select`  ==> `<select>`  

**FILED** `@Insert`  ==> `<insert>` 

**FILED** `@Delete`  ==> `<delete>`

**FILED** `@update`  ==> `<update>`

```java
@Insert("insert into user values (null, #{username}, #{password}, #{email}, #{tel})")
void saveUser(User user);

@Delete("delete from user where id = #{id}")
void removeUserById(int id);

@Select("select * from user")
List<User> listUser();

@Update("update user set username=#{username}, password=#{password}, email=#{email}, tel=#{tel} where id = #{id}")
void updateUser(User user);
```

#### SQL语句构建器

配合注解使用，适多条件查询，构建 SQL 语句

```java
public class UserMapperProvider {
	// 多条件查询，匿名内部类，返回拼接后的 sql 语句
    public String findUserByUsername(final String username) {
        return new SQL() {{
            SELECT("*");
            FROM("user");
            if (username != null && !username.isEmpty()) {
                WHERE("username like #{username}");
            }
        }}.toString();
    }
	// 链式调用，返回拼接后的 sql 语句
    public String deleteUserById(final int id) {
        return new SQL().DELETE_FROM("user")
            .WHERE("id = #{id}")
            .toString();
    }
}
```

`@SelectProvider(type, method)`

```java
@SelectProvider(type = UserMapperProvider.class, method = "findUserByUsername")
List<User> findUserByUsername(String username);

@DeleteProvider(type = UserMapperProvider.class, method = "deleteUserById")
void removeUserById(int id);
```

### MyBatis ORM 优化，缓存

#### 懒加载 LazyLoading

* 默认关闭
* 按需加载
* **resultMap** 的 **\<association>** **\<collection>** 可以配置懒加载，分两次 **SQL** 语句查询，需要用到时才查询
* 原理是使用动态代理，**UserMapper** 返回的是代理对象，增强的方法，当调用时再去查询

没有开启懒加载的效果，user - cellphone 案例

```java
@Test
public void lazyLoading() {
    // User 里有 List<Cellphone> cellphones;
    User user = mapper.findUserByUid(2);
    // 场景：当查询 user，仅需使用其部分属性时
    // 即没有使用到与它关联的 cellphones 时，仍会查两次，浪费资源
    System.out.println(user.getUsername());
}
// 日志，发送两次 SQL 语句
DEBUG findUserByUid:159 - ==>  Preparing: SELECT * FROM user_ WHERE uid = ? 
DEBUG findUserByUid:159 - ==> Parameters: 2(Integer)
TRACE findUserByUid:165 - <==    Columns: uid, username
TRACE findUserByUid:165 - <==        Row: 2, Candy
DEBUG findCellphoneByUid:159 - ====>  Preparing: SELECT * FROM cellphone_ WHERE uid = ? 
DEBUG findCellphoneByUid:159 - ====> Parameters: 2(Integer)
TRACE findCellphoneByUid:165 - <====    Columns: cid, cname, uid
TRACE findCellphoneByUid:165 - <====        Row: 2, huawei, 2
DEBUG findCellphoneByUid:159 - <====      Total: 1
DEBUG findUserByUid:159 - <==      Total: 1
```

**mybatis.xml** 开启，全局懒加载配置

```xml
<settings>
    <setting name="lazyLoadingEnabled"  value="true"/>
</settings>
```

懒加载效果

```java
@Test
public void lazyLoading() {
    User user = mapper.findUserByUid(2);
    // 当不需要使用 cellphone 时，不会去查询
    System.out.println(user.getUsername());
    // 只有当使用到 cellphone 时，会再次去查询一次 执行 <collection> 语句
    // System.out.println(user.getCellphones().size());
}
```

日志

```java
// 日志，只发送了一次 SQL 语句，没有去执行 <collection> 查询
DEBUG findUserByUid:159 - ==>  Preparing: SELECT * FROM user_ WHERE uid = ? 
DEBUG findUserByUid:159 - ==> Parameters: 2(Integer)
TRACE findUserByUid:165 - <==    Columns: uid, username
TRACE findUserByUid:165 - <==        Row: 2, Candy
DEBUG findUserByUid:159 - <==      Total: 1
```

**UserMapper.xml** 局部懒加载配置  **fetchType = (eager, lazy)**  积极加载 | 懒加载

```xml
<association property="cellphone"
             javaType="cellphone"
             select="findCellphoneByUid"
             column="uid"
             fetchType="lazy">
```

#### MyBatis 一级缓存

默认开启，每个 **SqlSession**，维护了一个 **Map**，key = sql + statement ，value = result

* 同一个 **sqlSession** 中，同一个条件**查询**
* 当前 **sqlSession** 不能 **close()**，可以 **conmit()**
* 第二次**查询**不会再次发送 **sql** 语句，而是直接从 缓存 **map** 里拿
* 当执行**增删改**操作时，会将 缓存 **map** 里面的数据清空

```java
// 同一个 sqlSession 中，可
@Test
public void firstCache() {
    User user = mapper.findUserByUid(2);
    // sqlSession.commit(); 提交后也可以命中缓存
    User sameUser = mapper.findUserByUid(2);
    // 查询两次相同条件，只发送了一次 SQL 语句
}

// 当前 sqlSession 不能 close()，需要开启二级缓存
```

#### MyBatis 二级缓存

默认关闭，每个 **Mapper** 代理，维护了一个 **Map**，key = sql + statement ，value = result

* 仅出现在一次事务中，**sqlSession** 没有提交情况下
* 不同 **sqlSession** 中，同一个条件查询
* 前一个 **sqlSession** 需要 **close()** 才会将查询结果放入缓存中，可以 **conmit()**
* 通过**序列化**，将查询结果对象写到某个地方缓存，因此 Bean 要实现  `Serializable` 接口
  * 否则报错，`PersistenceException` 不能序列化
* `Cache hit Ratio` 缓存命中率：使用缓存次数 ／ 查询次数（SQL 语句）

```xml
<!-- mybatis.xml 开启二级缓存总开关 -->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>

<!-- UserMapper.xml 开启二级缓存 -->
<cache/>
```

```java
// 不同 sqlSession 中，可用
@Test
public void firstCache() {
    User user = mapper.findUserByUid(2);
    // 关闭当前
    sqlSession.close();

    // 前一个 sqlSession 需要 close() 才会将查询结果放入缓存中
    sqlSession = factory.openSession();
    mapper = sqlSession.getMapper(UserMapper.class);
    User sameUser = this.mapper.findUserByUid(2);
}
```

### MyBatis 逆向工程 Generator

* 正向：数据库表 => JavaBean => Mapper.java => Mapper.xml


* 逆向：数据库表 自动生成 **JavaBean / Mapper.java / Mapper.xml**

建表，填充数据

```mysql
CREATE TABLE user
(
    id       INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(32) NULL,
    password VARCHAR(32) NULL,
    email    VARCHAR(32) NULL,
    tel      INT         NULL
)
```

新建一个空项目，导包

* `mybatis-generator-core-1.3.6.jar`
* `mysql-connector-java-5.1.44-bin.jar`

新建包结构

```java
src
 |- mybatis
 	 |- generator
 	 		|- bean
 	 		|- mapper
```

**generator-config.xml** 配置文件 ，配置连接数据库，放在项目根目录下

```xml
<generatorConfiguration>
    <context id="testTables" targetRuntime="MyBatis3">
        <!-- 数据库连接的信息 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis"
                        userId="root"
                        password="root123456"/>
        <!-- ... -->
    </context>
</generatorConfiguration>
```

**Generator.java** 执行代码

```java
public class Generator {
    public static void main(String[] args) throws Exception {
        new Generator().generator();
    }
    private void generator() throws Exception {
        List<String> warnings = new ArrayList<>();
        boolean overwrite = true;
        //指向逆向工程配置文件
        File configFile = new File("generator-config.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator =
            new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
    }
}
```

**generator-config.xml** 完整配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
                PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
                "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
   <!-- targetRuntime:
        1，MyBatis3：默认的值，生成基于MyBatis3.x以上版本的内容，包括XXXBySample；
        2，MyBatis3Simple：类似MyBatis3，不生成XXXBySample，<Base_Column_list> -->
    <context id="testTables" targetRuntime="MyBatis3">
        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis"
                        userId="root"
                        password="root123456"/>

        <!--默认false，
                    false 把 JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，
                    true 把 JDBC DECIMAL 和 NUMERIC 类型解析为 java.math.BigDecimal -->
        <!--<javaTypeResolver>
                    <property name="forceBigDecimals" value="false" />
                </javaTypeResolver>-->

        <!-- javaModelGenerator javaBean生成的配置信息
                     targetProject:生成PO类的位置
                     targetPackage：生成PO类的类名-->
        <javaModelGenerator targetPackage="mybatis.generator.bean"
                            targetProject="./src">
            <!-- 从数据库返回的值是否清理前后的空格 -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!-- sqlMapGenerator Mapper映射文件的配置信息
                    targetProject:mapper映射文件生成的位置
                    targetPackage:生成mapper映射文件放在哪个包下-->
        <sqlMapGenerator targetPackage="mybatis.generator.mapper"
                         targetProject="./src"/>
        <!--
           javaClientGenerator 生成 Model对象(JavaBean)和 mapper XML配置文件 对应的Dao代码
           targetProject:mapper接口生成的位置
           targetPackage:生成mapper接口放在哪个包下
           type：选择生成mapper接口（在MyBatis3/MyBatis3Simple下）：
           1. ANNOTATEDMAPPER：生成 Mapper 接口 + Annotation，
				不会生成对应的XML（SQL生成在annotation中）
           2. MIXEDMAPPER：使用混合配置，会生成 Mapper 接口，
				并适当添加合适的 Annotation，同时生成对应的XML
           3. XMLMAPPER：会生成 Mapper 接口，接口完全依赖 XML 配置
        -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="mybatis.generator.mapper"
                             targetProject="./src"/>

        <!-- 指定数据库表 -->

        <!-- 指定所有数据库表 -->

        <!--<table tableName="%"
                       enableCountByExample="false"
                       enableUpdateByExample="false"
                       enableDeleteByExample="false"
                       enableSelectByExample="false"
                       enableInsert="false"
                       enableDeleteByPrimaryKey="true"
                       enableSelectByPrimaryKey="true"
                       selectByExampleQueryId="false" />-->

        <!-- 指定数据库表，要生成哪些表，就写哪些表，要和数据库中对应，不能写错！ -->
        <table tableName="user"
               enableCountByExample="false"
               enableUpdateByExample="false"
               enableDeleteByExample="false"
               enableSelectByExample="false"/>
        
        <!--      <table schema="" tableName="orders"></table>
                     <table schema="" tableName="items"></table>
                     <table schema="" tableName="orderdetail"></table>
              -->
        <!-- 有些表的字段需要指定java类型
                 <table schema="" tableName="">
                    <columnOverride column="" javaType="" />
                </table> -->
    </context>
</generatorConfiguration>
```























