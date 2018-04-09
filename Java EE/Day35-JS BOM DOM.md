# JavaScript

#### 内容摘要

事件，鼠标移动，鼠标点击，加载与卸载，聚焦与离焦，键盘，提交与重制，选择与改变，DOM，XML，HTML DOM，获取 Element 节点，添加节点，删除节点

---

#### 事件

发生时机，回调函数，`on` 前缀

* 鼠标移动
* 鼠标点击
* 加载与卸载
* 聚焦与离焦
* 键盘
* 提交与重制
* 选择与改变

```html
<div onmousemove="myAction()">   </div>
<!-- 属性名即函数名，要带上括号 -->
```

#### 鼠标移动

* `mousemove `  鼠标在目标区域移动
* `mouseout`  鼠标移出目标区域
* `mouseover`  鼠标进入目标区域

#### 鼠标点击

* `click`  鼠标单击（或者按下回车键），会触发一次 mousedown 和 mouseup
* `dblclick `  鼠标双击，会触发两次 click


* `mousedown `  鼠标按下
* `mouseup `  鼠标抬起
* `contextmenu`  点击鼠标右键时触发，或者按下“上下文菜单”键时触发

#### 加载与卸载

* `<body>` 针对标签使用，或者 `window.onload = myFunciton`
  * 属于进度事件，不仅发生在 document 对象，还发生在各种外部资源上面。浏览网页就是一个加载各种资源的过程，图像（image）、样式表（style sheet）、脚本（script）、视频（video）、音频（audio）、Ajax请求（XMLHttpRequest）等等
* `load`  在页面加载成功时触发，页面从浏览器缓存加载，不会触发 load 事件，刷新会触发
* `unload `  在窗口关闭或者 document 对象将要卸载时触发
  * unload 事件只在页面没有被浏览器缓存时才会触发，换言之，如果通过按下“前进/后退”导致页面卸载，并不会触发 unload 事件

####聚焦与离焦

* `focus`  聚焦
* `blur`  离焦
* 比如，使用正则表达式来验证输入的邮箱或者注册信息是否正确

#### 键盘

* `keydown`  按下
* `keypress `  按下键，包括 keydown 和 keyup，不包括 Ctrl、Alt、Shift 和 Meta
* `keyup`  抬起

#### 提交与重制

```html
<!-- 发生在表单对象上，check() 返回 true 可以提交，返回 false 无法提交 -->
<form action="#" onsubmit="return check()">
  <!-- 验证表单提交的信息是否符合规范 -->
</form>

<!-- 发生在表单对象上，check() 返回 true 可以重置，返回 false 无法重置 -->
<form action="#" onrest="return confirmReset()">
  <!-- window.confirm() -->
</form>
```

* `submit`  表单提交时触发
* `reset`  表单重制时触发，可以让用户确认，避免数据丢失

#### 选择与改变

* `select`  input textarea 中选中文本时触发

* `change`  input select textarea 的值发生变化时触发

  *  二级表单联动

  > 激活单选框（radio）或复选框（checkbox）时触发。
  >
  > 用户提交时触发。比如，从下列列表（select）完成选择，在日期或文件输入框完成选择。
  >
  > 当文本框或textarea元素的值发生改变，并且丧失焦点时触发

## DOM

* 文档对象模型（Document Object Model）
* 访问 XML 和 HTML 文档的一套标准，允许脚本动态访问和修改其内容、结构和样式
* 分类
  * DOM core （XML HTML 共有 基于树的操作）
  * XML DOM 
  * HTML DOM

#### XML *extensive markup language*

* 可扩展标记语言
* 描述的数据具有树形的逻辑结构
* 标签需要自定义
* 通用的数据传输格式

| HTML           | XML            |
| -------------- | -------------- |
| 不可扩展标记语言       | 可扩展标记语言        |
| 标签已自定义         | 标签需要自定义        |
| 显示数据           | 通用的数据传输格式      |
| 描述的数据具有树形的逻辑结构 | 描述的数据具有树形的逻辑结构 |

## HTML DOM

* Node

  * Element 每个 HTML 标签
  * Attribute 每个 HTML 标签的属性
  * Text 包含在 HTML 元素中的纯文本
    * **换行被当作文本节点**
  * Document 整个文档
  * <comment> 

  > `Document `：整个文档树的顶层节点
  >
  > `DocumentType` ：`doctype` 标签（比如 `<!DOCTYPE html>` ）
  >
  > `Element` ：网页的各种HTML标签（比如 `<body>`、`<a>` 等）
  >
  > `Attribute `：网页元素的属性（比如 `class="right"` ）
  >
  > `Text` ：标签之间或标签包含的文本
  >
  > `Comment` ：注释
  >
  > `DocumentFragment` ：文档的片段

* HTML DOM 是对 XML DOM 扩展

* JS 中可以同时使用 HTML DOM 和 XML DOM

* 三种关系

  * 父节点关系（parentNode）：直接的那个上级节点
  * 子节点关系（childNodes）：直接的下级节点， `firstChild`  `lastChild`
  * 同级节点关系（sibling）：拥有同一个父节点的节点，`nextSibling`  `previousSibling`

|           |  nodeName   | nodeValue | nodeType |
| :-------: | :---------: | :-------: | :------: |
|  Element  |    标签名称     |   null    |    1     |
| Attribute |    属性名称     |    属性值    |    2     |
|   Text    |   `#text`   |   包含的文本   |    3     |
|  Comment  | `#comment`  |   注释文本    |    8     |
| Document  | `#document` |   null    |    9     |

#### 获取 Element 节点

* `getElementById(String)`  返回指定 id 的第一个节点的引用
* `getElementByName(String)`  返回指定 name 的节点集合
* `getElementByTagName(String)`  返回指定 TagName 的 节点集合

#### 创建节点

* `creatElementNode()`  创建元素节点
* `creatAttributeNode(String)`  创建属性节点
* `creatTextNode()`  创建文本节点

#### 添加节点

* `parent.appendChild(node)`  将 node 插入到父节点的最后一个子节点位置
* `parent.insertBefore(node, before node) `  将 node 插入到父节点的 before 节点前的位置

#### 删除节点

* `parent.removeChild(node)`  在父节点上删除子节点 node 



















