# XML

#### 内容摘要

DOM4J，API，更新 XML， XPath， XPath + DOM4J

---

#### 补充

* **IDEA** Java 模块工作路径 `work location = %MODULE DIR%`
* 查看工作路径 `system.getPropertie("user.dir")`

#### DOM4J

**不包含回车换行文本子节点**

* 性能优异，极易使用，功能强大，很多框架都是用该工具，如 **Hibernate**
* 解析 XML 配置文件
* 基于 DOM 解析
* `dom4j.jar`
* `jaxen.jar`

#### DOM4J API

* 读取 XML，获取 **Document** 对象

  ```java
  SAXReader saxReader = new SAXReader();
  Document document = saxReader.read(new File("books.xml"));
  ```

* 创建 Document 对象，可以生成 XML 文件

  * `Document document = DocumentHelper.createDocument()`

* `document.getRootElement()`

  * 首先要获取根结点

* `parentNode.elements()`

  * 获取所有子节点
  * **不包含回车换行文本子节点**

* `parentNode.element(String tagName)`

  * 返回 **parentNode** 中第一个标签名为 **tagName** 的子节点，支持命名空间

* `element.getText()`

  * 返回文本子节点的值

* `parentNode.addElement(String tagName)`

  * 在 **parentNode** 下添加子节点标签名为 **tagName**

* `parentNode.remove(childNode)`

  * 删除 **parentNode** 下子节点 **childNode**

* `element.addCDATA(String s)`

  * 为 **element** 节点创建一个**CDATA** 区

* `element.attribute(String name)`

  * 返回该节点属性名为 **name** 的 **Attribute** 对象

* `elment.remove(Attribute attr)`

  * 删除某个属性

#### 更新 XML

```java
 OutputFormat format = OutputFormat.createPrettyPrint();
// 指定编码形式，针对中文，英文则不需要指定
format.setEncoding("utf-8");
XMLWriter writer = new XMLWriter(new FileOutputStream("output.xml"));
writer.write(document);
writer.close();
```

#### XPath

* 在 XML 文档中定位标签
* `/` 绝对路径，从根节点开始匹配
* `//` 相对路径，匹配所有符合元素
* `/*` 匹配元素中所有子元素，
* `//*` 所有元素
* `[1]` 匹配第一个元素
* `[i]` 匹配第 i 个元素，下标从 1 开始
* `[last()]` 匹配最后一个元素
* `//@id` 匹配属性名的所有元素

#### XPath + DOM4J

**DOM4J 与 XPath 结合使用，查找方便**

* 从根节点开始匹配


* `selectSingleNode(String xPath)`
  * 返回匹配到的 第一个 元素
* `selectNodes(String xPath)`