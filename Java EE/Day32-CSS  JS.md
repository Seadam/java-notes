# CSS / JS

#### 内容摘要

补充，概述，CSS，样式优先级，代码规范，CSS选择器，框模型，定位

---

#### html 补充

* 块元素
  * 默认宽度与屏幕等宽
  * 换行，浏览器会自动地在块元素的前后添加空行
  * `display="block"`
* 内联元素
  * 不换行
  * 如 `<span>`
  * `display="inline"`
* `em `   W3C 推荐
  * 抽象单位，没有固定值
  * 相对于浏览器默认字体大小（16px）
  *  `h1:  40px/16=2.5em`   `h2:  30px/16=1.875em`   `p:  14px/16=0.875em`

#### 概述

* CSS：*Cascading style sheet*
* 层叠样式表
* 将网页内容和样式进行分离

#### CSS 与 HTML 结合

基于复用和可维护性考虑

* Style 属性（行内样式）
  * `style=""`
  * `<a style="text-align: center"> </a>`
  * 比较灵活，对于多个标签定义同一属性时麻烦，适合局部修改
* Style 标签（内联样式）
  * **在 `<head>` 标签内**写 `<style>` 标签
  * `<style type="text/css"></style>`
  * 对单个页面的样式进行统一配置，局部不够灵活
* 链接方式
  * 通过 `<link>` 标签链接外部样式表
  * `<link ref="stylesheet" type="text/css" href="style.css">`
  * 重复样式以最后链接的 CSS 文件为准
* 导入方式
  * 在 `<style>` 标签引入外部样式表
  * 若与本页样式重叠，以本页为主
  * `<style type="text/css"> @import url(style.css); </style>`

#### 样式优先级

* 由上到下，由外到内，优先级由低到高
* **就近原则**
* `行内样式 > id样式 > class样式 > 标签名样式`

```css
　　<div id="ID" class="CLASS" style="color:black;"></div>
   /* 行内样式优先级最高，其他 */
　　div < .class < div.class < #id < div#id < #id.class < div#id.class
```

#### 代码规范

* 选择器 ` { 属性名：属性值；  属性名：属性值；}`

#### CSS 选择器

指明该样式作用域，规定目标样式

* html 标签名选择器
  * 标签名
* class 选择器
  * 标签中 class 属性
  * `<a class="btn"> </a>`
  * ` .btn { color: white; }`
* id 选择器
  * 标签中 id 属性
    * 唯一性，针对个别元素定义样式
  * `<a id="btn"> </a>`
  * ` #btn { color: white; }`
* 关联选择器（上下文选择器）
  * 相同标签中的不同标签显示不同样式
  * `<p> <a href="#"> change style </a> </p>`
  * `p a { color: white}`
* 组合选择器
  * 对多个不同选择器使用相同样式
  * 用 `,` 隔开
  * `.btn,#btn,p { color: white}`
* 伪元素选择器
  * html 预先定义的选择器
  * **顺序不能变：link、visited、hover、active**
    * `a:link`
    * `a:visited`  
    * `a:hover`  
    * `a:active`
* 属性选择器
  * `[att*=val] {}`
    * 如果元素的 att 属性的值包含 val 值则使用该属性
    * 如：`<a name="href" href="#"> </a>`
    * `[name*=e] { }`  name 属性中包含 e 这个字符串，使用该属性
  * `[att^=val] {}`
    * 如果元素的 att 属性的值开头以 val 值则使用该属性
    * 如：`<a name="href" href="#"> </a>`
    * `[name^=hr] { }`  name 属性中包含 hr 这个字符串，使用该属性
  * `[att$=val] {}`
    * 如果元素的 att 属性的值结尾是 val 值则使用该属性
    * 如：`<a name="href" href="#"> </a>`
    * `[name$=ef] { }`  name 属性中包含 ef 这个字符串，使用该属性

#### 框模型

* content 元素实际内容所占空间
  * `background-color` 作用于 content + padding 内
* padding 内边距
* border 边框
* margin 外边距

> * **Content** - The content of the box, where text and images appear
> * **Padding** - Clears an area around the content. The padding is transparent
> * **Border** - A border that goes around the padding and content
> * **Margin** - Clears an area outside the border. The margin is transparent

#### 定位

块级框 `display: block;` 从上到下依次排列

行内框 `display: inline;`

* static  默认排序方式

  * 文档流中按元素框正常排列

* relative  相对定位

  > 元素框偏移某个距离。元素仍保持其未定位前的形状，它**原本所占的空间仍保留**。

  * 相对于该元素自己正常定位位置进行移位

* absolute  绝对定位

  > 元素框从文档流完全删除，并**相对于其包含块定位**。包含块可能是文档中的另一个元素或者是初始包含块。元素原先在正常文档流中所占的空间会关闭，就好像元素原来不存在一样。元素定位后生成一个块级框，而不论原来它在正常流中生成何种类型的框。

  > if an absolute positioned element has no positioned **ancestors**, it uses the **document body**, and moves along with page scrolling.

  * 相对于其包含块：左上角基准点（大部分相对于网页正文左上角基准点）

* 坐标系

  * 原点（基准点）：左上角

# JavaScript

#### 内容摘要

概述，与 Java 对比，语言组成，与 HTML 组合，基本语法，数据类型，运算符，流程控制，Array 数组对象

---

#### 概述

* NetScape   ECMAScript
* 基于**对象**和**事件驱动**的**脚本语言**
* 无须编译，由浏览器解释运行
* **弱类型语言**
* 特点
  * 交互型
    * 与 html 交互
  * 安全性
    * 不允许直接访问本地磁盘
  * 跨平台性
    * 只需浏览器解释

#### JavaScript VS Java

| JavaScript          | Java         |
| ------------------- | ------------ |
| NetScape，LiveScript | Oracle，Sun   |
| 基于对象                | 面向对象         |
| 浏览器解释执行             | 编译器编译字节码，再执行 |
| 弱类型                 | 强类型          |

#### 语言组成

* 核心语法 **ECMAScript**
* 文档对象模型（**DOM** *document object model*）
* 浏览器对象模型（**BOM** *browser object model*）

#### JS 与 HTML 结合

* 内部 JS
  * `<script type="text/javascript"> alert("hello world.") </script>`
* 外部 JS
  * `<script src="script.js" type="text/javascript"> </script>`
  * 在引入外部 JS 的`<script>` 间不能写 JS 代码，无效果

#### 基本语法

* 变量、函数名、运算符 区分大小写
* 定义变量：弱类型
  * `var a = 1;`  `var b = true;`  `var c = "hello";`
  * `;` 可以省略，但是不推荐
* **先定义后使用**

#### 数据类型

* 原始数据类型

  * **伪对象** ：虽然是基础数据类型，但仍然可以调用方法
  * `undefined`  `null`  `boolean`  `number`  `string`
  * `number`
    * 包括整数和小数
  * `string`
    * 字符串
  * `boolean`
    * 布尔，值 true false
  * `Undefined`
    * 未定义，声明变量，没有赋值， `var a; ` 此时 a 未定义
  * `null`
    * 表示一个空对象指针
    * `typeof(null)  = object`


  * 字符串是原始数据类型

* 引用数据类型

  * 所有 object 都是引用类型

* `typeof`

  * 判断变量类型

*  `instanceof`

  * 判断对象类型

* 如果某变量的值可以转化为数值类型，那么该变量可以参与运算

  * `1 + "1" = 11` 字符串拼接 
  * `10 - "5" = 5` 减法

#### 运算符

* 算术
  * 算术运算不关心类型，只关心值，转换为数值类型再计算
* 比较
  * `==`  只比较值，不会比较类型 ` 1 == "1" // true`
  * `===`  不仅比较值，还比较类型 `1 === "1" // false`
  * ​
* 三目
  * 同 Java

#### 流程控制

大体上同 Java

* 顺序
* 选择
* 循环

#### 常用对象

* Array 数组对象
* String 字符串对象
* Number 数值对象
* Boolean 布尔对象
* Math
* Date

#### Array 数组对象

* 同一个数组可以放入任意类型，没有数组越界错误

* `var a = new array();` 

  > * 当不写参数时，初始化一个空数组
  > * 当参数为一个整数时，表示一个有初始化大小的数组
  > * 当参数不止一个时，每个参数表示数组元素

  ```javascript
  // 无参数时，返回一个空数组
  new Array() // []

  // 单个正整数参数，表示返回的新数组的长度
  new Array(1) // [ empty ]
  new Array(2) // [ empty x 2 ]

  // 非正整数的数值作为参数，会报错
  new Array(3.2) // RangeError: Invalid array length
  new Array(-3) // RangeError: Invalid array length

  // 单个非数值（比如字符串、布尔值、对象等）作为参数，
  // 则该参数是返回的新数组的成员
  new Array('abc') // ['abc']
  new Array([1]) // [Array[1]]

  // 多参数时，所有参数都是返回的新数组的成员
  new Array(1, 2) // [1, 2]
  new Array('a', 'b', 'c') // ['a', 'b', 'c']
  ```

* `var b = [1, 2]; `  指明数组元素

* 方法

  * `join(seperator: String)` 把数组所有元素拼接到一个字符串，可以添加 分隔符

  * `push()` 向数组末尾添加一个或多个元素，返回新数组长度

  * `pop()` 删除并返回数组的最后一个元素

  * `shift()` 删除并返回数组第一个元素

  * `reverse()` 反转数组

  * `sort()` 排序数组

    * 字符串比较结果 字典序
    * 也可以传入 比较器
    * `sort(function(a, b) {// 类似于 Comparable 的匿名方法，可以指明比较规则});`
      * 若 a > b ，返回正数
      * 若 a = b ，返回零
      * 若 a < b ，返回负数

    #### 





















