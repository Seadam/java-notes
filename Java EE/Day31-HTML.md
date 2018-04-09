# HTML

#### 内容摘要

概述，编写规范，head 部分，body 部分，RGB，图片插入，列表，超链接，表格，表单

nbsp

---

#### 概述

* 万维网 ： WWW （*World Wild Web*）
  * 基于 *Internet*，将所有互联网中的**资源**，统一的表示，
  * 资源：统一的表示形式，网页，HTML（Hyper text markup language)
  * 定位资源：URL（*Universal resource location*）  统一资源定位
  * 传输资源；Http（*Hyper text transport protocal*)
* HTML（*Hyper text markup protocal*)
  * 超文本标记语言，描述性的标记语言
  * 用来描述**超文本**内容显示方式
    * 纯文本：可见字符组成的，没有结构
    * 超文本：**超链接**，还可以表示图片、视频、音频等，内容有结构
  * HTML 5
    * 更加规范化
      * 将内容与内容的显示形式分离开，推荐使用 CSS 来表示内容显示形式
    * 效率提高
    * 增加功能强大的标签
    * 移动端可以通过 HTML 5 跨平台
    * 处理兼容性问题
    * 声明改进
      * `<!DOCTYPE html>`
      * `<meta charset="UTF-8">`
  * w3c 万维网联盟
    * 早期 HTML 标签可以自定义，导致浏览器的兼容问题大
    * 统一 HTML 规范
* http 传输协议

#### 编写规范

* 后缀扩展名：html 或 htm

* 书写规范

  * 标签
    * 可以是自定义的（浏览器可能无法解析）
    * 成对出现的标签
      * `<div>  </div>`
      * `<a>  </a>`
    * 单个出现的标签
      * `<br>`
  * 元素
    * 成对出现的标签，包含包裹的文本
    * 属性
      * `<font size="7"> this is a element</font> `  // html 5 已不推荐此标签

* 文档的组成部分

  * `<html> <head> </head> <body> </body> </html>`

  ```html
  <!DOCTYPE html>  <!-- 声明该文件是一个 html 文件 -->
  <html lang="en">  <!-- 声明网页所使用的语言 -->

      <head>
          <meta charset="UTF-8">
          <title>Document</title>
      </head>

      <body>
          <!-- 显示在网页中正文部分 -->
      </body>

  </html>
  ```

#### head 部分

* 不会显示在网页正文中的内容


* `<title>Web title</title>` 网页的标题，显示在选项卡中
* `<meta>` 元数据：描述整个 html 特定属性的元数据
  * `<meta charset="UTF-8">`  指定网页的编码
  * `<meta name="keywords" content="HTML,CSS\">`  指明搜索引擎抓取关键字
  * `<meta name="description" content="免费在线教程">`  指明该网页描述
* `<link> </link>` 引用外部文件 `stylesheet`  `script`
  * `<link rel="stylesheet" type="text/css" href="style.css">`  引入外部 style.css 文件

#### body 部分

* 注释标记
  * `<!-- 注释 -->`
* 段落标记
  * `<p> new paragraph </p>`
  * 块级元素
  * **浏览器会自动地在段落的前后添加空行**
  * **连续的空行（换行）也被显示为一个空格**
* 换行标记
  * `<br>`
  * 不产生一个新段落的情况下进行换行
  * **连续的空行（换行）也被显示为一个空格**
* 水平线标记
  * `<hr>`
  * 属性
    * 默认单位为像素，可以省去
    * `align`
    * `size`
    * `color`
    * `width`  默认值为整个网页屏幕
      * 可以指定百分比
      * 规避屏幕适配问题，**推荐使用百分比**
    * `noshade="noshade"`
      * 无立体效果
* 字体标记，**不推荐**
  * `<font>` 
  * 属性
    * `size`
      * 范围 1 ~ 7
      * 基准值是 3，+2 = 5
    * `color`
    * `face`
      * 字体
* 标题标记
  * `<h1> </h1>`  ….  `<h6> </h6>`
  * 一级标题最大
* 特殊符号
  * `&lt;  =   <`
  * `&gt;  =   >`
  * `&nbsp;   =   空格`
  * `&copy;   =   C 版权`
  * `&trade;  =   TM 商标`

#### RGB

* 计算机中的颜色表示，红 绿 蓝
* 每一个种颜色，分为 256 个程度
* 8 位 表示一种颜色的取值，用十六进制表示
* RGB（红，绿，蓝）==>（0xFF，0x00，0x00）
* `red : #ff0000`  `green : #00ff00`  `blue : #0000ff`


* `red : rgb(255, 0, 0)`  `green : rgb(0, 255, 0)`  `blue : rgb(0, 0, 255)`
* ARGB 表示法
  * RGBA (不透明度，红，绿，蓝)  ==> (0xFF, 0xFF, 0x00, 0x00)
  * 数值越大，越不透明
* RGBA
  * (red, green, blue, alpha)
  * 0 不透明， 1 透明
  * alpha通道值可以用百分比、整数或者像RGB参数那样用0到1的实数表示

#### 图片插入

* `<img src="img.jpg" width=100 height=100 border=0 alt="image">
* `src` 图片路径一般是相对路径
* `alt` 当图片加载失败时，显示的提示性文字

#### 列表

* 有序列表
  * `<ol> </ol>` 有序元素
  * `<li> </li>` 列表元素
  * 属性
    * `type`
      * `A`   `a`  `i`  `I`  `1`  `一`  默认为 `1`
    * `start`
      * 编号起始位置，默认为 1
* 无序列表
  * `<ul> </ul>` 无序元素
  * `<li> </li>` 列表元素
  * 属性
    * `type`
      * `circle`  `disc`  `square`  默认为 `disc`
* 定义列表
  * `<dl> </dl>`  自定义列表
  * `<dd> </dd>`  列表元素

#### 超链接 *Hyper Reference*

* `herf`
  * 文字超链接 `<a herf="https:\\baidu.com">Click me</a>`
  * 图片超链接 `<a herf="https:\\baidu.com"> <img src="img.jpg"> </a>`
* 目标地址
  * 相对于本网站目录地址
    * 可以不用加 `https:\\` `http:\\`
  * 本网页书签锚点地址
    * `<a name="leap"> head </a>`
    * `<a herf="#leap"> leap </a>`
  * 外部网站地址
* `target`
  * `_self` 当前窗口访问
  * `_blank` 开启新窗口

#### 表格

* `<table> </table>`
  * `width`   `height`
  * `border`  `bordercolor`
  * `align`  
  * `cellspacing` 单元格外边距
  * `cellpadding ` 单元格内边距
* `<caption> title </caption>`  表格标题
* `<thead> </thead>`  表头
* `<tr> row </tr> `  行
* ` <td> </td> ` 行中的元素
* `rowspan`  行向右扩展单元格
* `colspan`  列向下扩展单元格

#### 表单

`<form> </from>`

* **用于与服务器交互**
* **`name` 属性**，表单中所有元素必须有 `name` 属性才能拿到值
* **`action` 属性**，指明要讲数据提交给谁，默认为自己


* `<input type="">`  用户输入
  * `text` 明文
  * `password`  密文
  * `radio`  单选框
    * 要实现单选效果，需要保证同一组单选框 `name` 属性要相同，才能实现单选效果
    * `<input type="radio" name="sex">男  <input type="radio" name="sex">女`
  * `checkbox`  复选框
    * 要实现一个值传到服务器，`name` 属性要相同
  * `file`  上传文件
  * `button`  按钮
    * 按钮名 value
    * `<input type="button" value="click me">`
  * `submit`
    * 按钮名 value
    * 点击 `submit` 这个按钮元素，进行提交
    * 如果没有指定 `method` ，默认是 GET 提交方式
    * `<input type="submit" method="post">`
  * `reset` 
  * `hidden` 
* `<select> </select>  ` 下拉列表
  * `<option> </option>`  下拉列表元素
  * 默认选中 `<option selected="selected"> </>`
* `<textarea rows="20" cols="20"> </textarea>`  文本框
  * rows  行数
  * cols  列数
* 提交方式
  * GET
    * HTTP 请求参数的形式，将代提交数据拼接到 url 后进行提交 
    * `?username=java&pwd=1234`
    * 数据安全性差，地址栏可以看到提交数据
    * 数据拼接到**地址栏** `?username=java&pwd=1234`
  * POST
    * 代提交数据通过 HTTP 请求正文中方式进行提交
    * 地址栏不可见，安全性高
* 语义化表单
  * `<fieldset> </fieldset> ` 包裹表单内元素
    * `<legend> </legend>` 小标题

































