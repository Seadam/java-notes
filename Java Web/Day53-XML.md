# XML

#### 内容摘要

Schema 约束，Schema 与 DTD 相比，Schema 声明语法，XML 引入 Schema，XML 解析概述，JAXP，DOM 解析流程，更新 XML，SAX 解析，SAX 解析流程，DOM 模型回忆

---

概念、语法、DTD 约束、语法见前

#### XML Schema 约束

* 克服 DTD 约束的局限性
* `.xsd`  *XML Schema Definition*
* 引入命名空间：前缀，约束

#### Schema 比 DTD 优势

* **符合 XML 语法结构**，可以使用 XML 解析器进行解析（DTD 不符合 XML 语法，需要额外解析器）
* **支持命名空间** （DTD 不支持命名空间（不支持 `:` ））
* **支持更多类型** （DTD 只有 PCDATA、ID、枚举类型）

#### Schema 声明语法

* `.xsd`  *XML Schema Definition*


* 必须有一个根节点，且该节点名为 **schema**
* 需要把 XML **Schema** 文档声明绑定到一个**命名空间**（**URI**）
* **xmlns** *XML namespace*;
* `xmlns:xs` 简写命名空间为 xs
* `xmlns:xs="http://www.w3.org/2001/XMLSchemea"`   约束的约束
* `targetNamespace="http://www.sample.com"`  绑定 命名空间，用于外部引用

```scheme
<?xml version="1.0" encoding="utf-8"?>
<xs:schema 
	xmlns:xs="http://www.w3.org/2001/XMLSchemea" // 标准命名空间
    targetNamespace="http://www.sample.com"> // 绑定到 http://www.sample.com URI 命名空间 
```
```dtd
<?xml version="1.0" encoding="UTF-8" ?>
<test:schema xmlns:test="http://www.w3.org/2001/XMLSchema"  // 约束的约束
             targetNamespace="http://www.test.com" // 命名空间，外部引用 uri
             elementFormDefault="qualified"> // 任意使用该 Schema 的 XML 文档要验证命名空间
    <test:element name="books">
        <test:complexType>
            <test:sequence maxOccurs="unbounded">
                <test:element name="book">
                    <test:complexType>
                        <test:sequence>
                            <test:element name="name" type="test:string"/>
                            <test:element name="author" type="test:string"/>
                            <test:element name="price" type="test:float"/>
                        </test:sequence>
                    </test:complexType>
                </test:element>
            </test:sequence>
        </test:complexType>
    </test:element>
</test:schema>
```

> **elementFormDefault="qualified"**
>
> indicates that any elements used by the XML instance document which were declared in this schema must be namespace qualified.

#### XML 引入 Schema

* `xmlns:sample="http://www.sample.com"`  引入命名空间，并取别名

* `xmlns:xsi="http://www.w3.org/2001/XMLSchemea-instance"`   
* `xsi:schemaLocation="http://www.sample.com sample.xsd"`   要引入的约束文件位置
  * `http://www.sample.com` 命名空间
  * `sample.xsd`  对应约束文件位置，也可以是网路位置，会自动去下载
* `xmlm="http://www.sample.com"` 没有取别名，使用默认命名空间

```xml
<?xml version="1.0" encoding="utf-8" ?>
<sample:tag 
    xmlns:sample="http://www.sample.com" // 引入命名空间并取别名
    xmlns:xsi="http://www.w3.org/2001/XMLSchemea-instance" // 引入另一个命名空间，用来指定
    xsi:schemaLocation="http://www.sample.com sample.xsd"> // xsd 文件位置
</sample:tag>
```
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<books xmlns="http://www.test.com" // 引入命名空间，默认命名空间，没有取别名
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" // 引入 w3c Schema 实例
       xsi:schemaLocation="http://www.test.com test.xsd"> // 本地当前命名空间文件位置
    <book>
        <name>BookName</name>
        <author>BookAuthor</author>
        <price>9.9</price>
    </book>
    <book>
        <name>BookNameTwo</name>
        <author>BookAuthor</author>
        <price>18.9</price>
    </book>
</books>
```

#### Schema 语法 

查文档，见上

## XML 解析

#### 概述

* **DOM** 方式
  * **Document Object Model**
  * **W3C** 推荐
* **SAX** 方式
  * **Simple API for XML**
  * 开源社区 **XML-DEV**
* 开发包
  * **JSAP** ：Sum 公司推出
  * **DOM4J**：开源组织推出

#### JAXP

* **Java API for XML Processing**
* `org.w3c.dom`  **DOM** 方式解析标准接口
* `org.xml.sax`  **SAX** 方式解析标准接口
* `javax.xml`  解析 **XML** 文档实现类
  * `javax.xml.parsers`
  * **DocumentBuilderFactory**
  * **SAXParserFactory**

####DOM 解析流程 DocumentBuilderFactory 

`javax.xml.parsers.DocumentBuilder`

`javax.xml.parsers.DocumentBuilderFactory`  

`org.w3c.dom.Document`

1. 创建 **DOM** 解析器工厂 `DocumentBuilderFactory.newInstance()`
2. 拿到 **DOM** 解析器对象 `factory.newDocumentBuilder()`
3. `parse()` 解析 **XML** 文档，返回 **Document** `parse(new File("books.xml"))`
4. 操作 **DOM** 的方式操作 **Document** 对象

* `getChildNodes()`   **换行、空格被当作文本节点**

```Java
// 创建 DOM 解析器工厂
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
// 拿到 DOM 解析器对象
DocumentBuilder documentBuilder = factory.newDocumentBuilder();
// 解析 XML 文档，拿到 Document 对象
Document document = documentBuilder.parse(new File("books.xml"));
```

#### 更新 XML

`javax.xml.transform.TransformerFactory`

`javax.xml.transform.Transformer`

* 将内存中代表 XML 的 DOM 树保存为 XML 写回文件
* **DOMSource**  要转换的 **document** 对象
* **StreamResult**  要保存的目的地

```java
// 拿到 Transformer 工厂
TransformerFactory factory = TransformerFactory.newInstance();
// 创建 Transformer 对象
Transformer transformer = factory.newTransformer();
// 将 DOM 树写回文件
transformer.transform(new DOMSource(document), new StreamResult("books.xml"));
```

#### SAX 解析

* DOM 解析时，要读取整个 XML 文档，会占很多内存
* SAX 解析，在解析文档时就会对文档进行处理，**只能读，不能增删改**
* **解析器与事件处理器**，使用回调（**CallBack**）来处理文档
* 将解析出来的数据，放到 **Bean** 类集合中，可以灵活读取

#### SAX 解析流程 SAXParserFactory

1. 创建 **SAX** 解析器工厂 `SAXParserFactory.newInstance()`
2. 拿到 **SAX** 解析器对象 `factory.newSAXParser()`
3. 拿到 **XML** 读取器 `saxParser.getXMLReader()`
4. 设置读取器的事件处理器，即回调函数，需要自己实现 **ConntentHandler** 或继承 **DefaultHandler**
5. `parse()` 解析 **XML** 文档

* **换行、空格被当作文本节点**

```java
// 创建 SAX 解析器工厂
SAXParserFactory factory = SAXParserFactory.newInstance();
// 创建 SAX 解析器对象
SAXParser saxParser = factory.newSAXParser();
// 拿到 XML 读取器
XMLReader xmlReader = saxParser.getXMLReader();
// 设置读取器的事件处理器
xmlReader.setContentHandler(new MyContentHandler());
// 解析 XML
xmlReader.parse("books.xml");
```

```java
class MyContentHandler extends DefaultHandler {
    @Override
    public void startDocument() throws SAXException {
        // 开始解析时调用
        super.startDocument();
    }

    @Override
    public void endDocument() throws SAXException {
        // 解析完成时调用
        super.endDocument();
    }

    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        // 解析到元素节点开始标签时调用
        super.startElement(uri, localName, qName, attributes);
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        // 解析到元素节点结束标签时调用
        super.endElement(uri, localName, qName);
    }

    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        // 解析到文本节点时调用，包括空格、换行等
        super.characters(ch, start, length);
    }
}

```

#### DOM 模型

> * Node
>
>   * Element 每个 HTML 标签
>   * Attribute 每个 HTML 标签的属性
>   * Text 包含在 HTML 元素中的纯文本
>     * **换行被当作文本节点**
>   * Document 整个文档
>   * <comment> 
>
>   > `Document `：整个文档树的顶层节点
>   >
>   > `DocumentType` ：`doctype` 标签（比如 `<!DOCTYPE html>` ）
>   >
>   > `Element` ：网页的各种HTML标签（比如 `<body>`、`<a>` 等）
>   >
>   > `Attribute `：网页元素的属性（比如 `class="right"` ）
>   >
>   > `Text` ：标签之间或标签包含的文本
>   >
>   > `Comment` ：注释
>   >
>   > `DocumentFragment` ：文档的片段
>
> * HTML DOM 是对 XML DOM 扩展
>
> * JS 中可以同时使用 HTML DOM 和 XML DOM
>
> * 三种关系
>
>   * 父节点关系（parentNode）：直接的那个上级节点
>   * 子节点关系（childNodes）：直接的下级节点， `firstChild`  `lastChild`
>   * 同级节点关系（sibling）：拥有同一个父节点的节点，`nextSibling`  `previousSibling`
>
> |           |  nodeName   | nodeValue | nodeType |
> | :-------: | :---------: | :-------: | :------: |
> |  Element  |    标签名称     |   null    |    1     |
> | Attribute |    属性名称     |    属性值    |    2     |
> |   Text    |   `#text`   |   包含的文本   |    3     |
> |  Comment  | `#comment`  |   注释文本    |    8     |
> | Document  | `#document` |   null    |    9     |
>
> #### 获取 Element 节点
>
> * `getElementById(String)`  返回指定 id 的第一个节点的引用
> * `getElementByName(String)`  返回指定 name 的节点集合，数组
> * `getElementByTagName(String)`  返回指定 TagName 的 节点集合，数组
> * `parent.childNodes`  当前 parent 节点所有孩子节点集合，数组
>   * **换行被当作文本节点**
> * `parentNode`  当前节点的父节点
> * `firstNode`  当前节点的第一个子节点
> * `lastNode`  当前节点的最后一个节点
>
> #### 创建节点
>
> * `creatElementNode()`  创建元素节点
> * `creatAttributeNode(String)`  创建属性节点
> * `creatTextNode()`  创建文本节点
>
> #### 添加节点
>
> * `parent.appendChild(node)`  将 node 插入到父节点的最后一个子节点位置
> * `parent.insertBefore(node, before node) `  将 node 插入到父节点的 before 节点前的位置
>
> #### 删除节点
>
> * `parent.removeChild(node)`  在父节点上删除子节点 node 
> * `textNode.deleteData(offset, length)`  删除 文本 节点，从 offset 开始，删除 len 个字符长度
> * `elementNode.removeAttribute(String)`  删除元素 属性名为 String 的属性值
>
> #### 修改节点
>
> * `parent.replaceChild(newNode, oldNeode)`   父节点上用 newNode 替换 oldNode
> * `textNode.replaceData(offset, length, String)`  文本节点从 offset 开始 length 长度替换为 String
> * `elementNode.setAttribute(name value)`  元素节点属性名为 name 的属性值设置为 value
>
> #### 拿到节点的值
>
> * `innerText`
> * `innerHtml`
> * `nodeValue`  // text attribute 
> * **换行被当作文本节点**
>
> #### 复制节点
>
> * `node.cloneNode(deep)`  返回一个新克隆节点，deep : boolean
>   * true 深度克隆，当前节点和属性，及其子节点
>   * false 浅度克隆，当前节点及其属性