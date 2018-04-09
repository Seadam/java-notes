# JavaScript

#### 内容摘要

Array，String，Number，Boolean，Math，Date，方法，BOM，window，navigator，history，screen，    location，网页跳转

---

#### Array 数组对象

* 当数中的某一个元素被删除后，元素下标和数组长度都会变
* 定义方式
  * `new Array()` 
    * 当不写参数时，初始化一个空数组
    * 当参数为一个整数时，表示一个有初始化大小的数组
    * 当参数不止一个时，每个参数表示数组元素
  * `[]`  推荐
* 方法：
  * `sort()` 排序数组
    * 字符串比较结果 字典序
    * 也可以传入 比较器
    * `sort(function(a, b) {// 类似于 Comparable 的匿名方法，可以指明比较规则});`
      * 若 a > b ，返回正数
      * 若 a = b ，返回零
      * 若 a < b ，返回负数

#### String 字符串对象 

* 同 Java，字符串是不可变的，任何改变字符串操作都会返回新串
* 定义：`var s = new String("hello");`
* 方法
  * `big()`  用大号字体显示字符串
  * `charAt(index)` 返回 index 下标字符
  * `concat()` 拼接字符串，返回一个新串
  * `substr(index, length)` 从 index 开始 提取 length 长度字符，返回一个新串
  * `substring(start, end)`  [start, end)  返回两个下标范围之间的字符串

#### Number 数值对象

* `0b`  开头 二进制，`0x1111`
* ` 0x`  开头十六进制，`0xF`


* `toString()`  返回原始类型 string 字符串
* `valueOf()`  返回原始类型 number 数值
* `NaN` 表示非数值 *not a number*
  *  `new Number("hello").valueOf();`  输出 NaN
  * `new Number("undefined").valueOf();`  输出 NaN
  * **`new Number("null").valueOf();`  输出 0**

#### Boolean 布尔对象

* `toString()`  返回原始类型 string 字符串
* `valueOf()`  返回原始类型 boolean 值
  * 对于字符串，非空可以转化为 true ，空串为 false
    * `new Boolean("hello").valueOf(); // true`  
    *  `new Boolean("").valueOf(); // false`  
  * 对于数值型，非零可以转化为 true ，零为 false
    * `new Boolean(1).valueOf(); // true`  
    *  `new Boolean(0).valueOf(); // false`  
  * `new Boolean(undefined).valueOf(); // false`  
  * `new Boolean(null).valueOf(); // false`  

#### Math 

* `Math.ceil(x)` 向上取整，返回大于等于 x 的最小整数
* `Math.floor(x) ` 向下取整，返回小于等于 x 的最大整数
* `Math.random()` 返回 0 ～1 随机数，小数
  * 产生 [0, x) 范围随机数 `Math.floor(Math.random() * x)`
* `Math.round(x)` 将 x 四舍五入，返回最近整数

#### Date

* `Date()`  返回当前日期、时间对象
* `getYear()` 返回 Date 对象 距离1900的年数
* `getFullYear()` 返回 Date 对象 四位的年份
* `getDate()` 返回 Date 对象 一个月中的某一天 (1 ~ 31)
* `getDay()` 返回 Date 对象 星期 (0 ~ 6) 星期日为0
* `getMonth()` 返回 Date 对象 月份  (0 ~ 11)
* `getHours()` 返回 Date 对象 小时 （0～23）
* `getMinutes()` 返回 Date 对象 分钟（0～59）
* `getSeconds()` 返回 Date 对象 秒（0～59）
* `getMilliseconds()` 返回  Date 对象 毫秒（0-999）
* `getTime()` 返回 Date 对象 距离1970年1月1日00:00:00 的秒数，等同于 `valueOf ` 方法
* `epoch` ：纪元时间 Unix 1970 年 1 月 1 日 00:00:00 
  * Java 中 1970 年 1 月 1 日 08:00:00 东八区

> * 分钟和秒：0 到 59
> * 小时：0 到 23
> * 星期：0（星期天）到 6（星期六）
> * 日期：1 到 31
> * 月份：0（一月）到 11（十二月）
> * 年份：距离1900年的年数

#### 方法 Function

* 定义，**先定义后使用**

  * 无须定义返回值类型，可以直接返回，也可以不返回

    ```javascript
    function add(a, b) {
      return a + b;
    }
    ```

  * 无须定义返回值类型，函数指针，指向方法的引用

    ```javascript
    var add = function (a, b) {
      return a + b;
    }
    ```

  * 不推荐，最后一个 `''` 内是方法体，前面是可变参数类型

    ```javascript
    var add = new Function('a', 'b', 'return a + b;');
    ```

* `setInterval(function, milliseconds)`

  * 回调，事件驱动
  * 间隔 milliseconds 毫秒，执行 function
  * `clearInterval()` 或关闭窗口，才能结束循环
  * `clearInterval(id)`  由 setInterval 返回的 id 来控制结束

* `setTimeout(function, milliseconds)`

  * 延迟 milliseconds 毫秒，执行 function

## BOM

JS 和网页交互就是动态操作文档对象 DOM（HTML 元素和 CSS 元素）（*Document object model*）

JS 也可以和浏览器操作 BOM （*Browser object model*）

* BOM：将浏览器一些功能抽象成 JS 对象，提供 JS 与浏览器交互的接口
* `window`
* `navigator`
* `screen`
* `history`
* `location`

#### window

* **区分大小写**，Window 指对象，window 指浏览器已创建的对象实例

* 浏览器打开的窗口，是顶层对象，全局对象

* 代表一个浏览器窗口或框架（iframe），为 HTML 文档创建一个 Window 对象，为每一个框架额外创建

* `window.frames`  属性，是 window 中所有 iframe 的 Window 对象数组

* `top` 最顶级窗口 `parent` 父窗口 `self` 当前窗口引用（等价于 window）

* 顶级窗口 `parent = top = self`

* `window.opener` 属性返回打开当前窗口的父窗口。如果当前窗口没有父窗口，则返回 `null`

  * 通过 `opener` 属性，可以获得父窗口的的全局变量和方法
  * 比如 `window.opener.propertyName` 和 `window.opener.functionName()`

* `window.open` 方法一共可以接受四个参数。

  * 第一个参数：字符串，表示新窗口的网址。如果省略，默认网址就是`about:blank`。
  * 第二个参数：字符串，表示新窗口的名字。如果该名字的窗口已经存在，则跳到该窗口，不再新建窗口。如果省略，就默认使用 `_blank`，表示新建一个没有名字的窗口。`_self` 当前窗口打开
  * 第三个参数：字符串，内容为逗号分隔的键值对，表示新窗口的参数，比如有没有提示栏、工具条等等。如果省略，则默认打开一个完整UI的新窗口。
  * 第四个参数：布尔值，表示第一个参数指定的网址，是否应该替换`history`对象之中的当前网址记录，默认值为`false`。显然，这个参数只有在第二个参数指向已经存在的窗口时，才有意义。

  下面是一个例子。

  ```javascript
  var popup = window.open(
    'somepage.html',
    'DefinitionsWindows',
    'height=200,width=200,location=no,status=yes,resizable=yes,scrollbars=yes'
  );
  ```

* `alert()`  提示消息

* `comfirm()`  指定消息， OK 及 取消 按钮，返回 true or false

  * `prompt()`  提示用户输入，Ok 及 取s消 按钮，返回 string or null

#### navigator

* 客户机浏览器信息
* `navigator.appName()`  浏览器名称  `Netscape`
* `navigator.userAgent()`

> Strange enough, "Netscape" is the application name for both IE11, Chrome, Firefox, and Safari.

> "Mozilla" is the application code name for both Chrome, Firefox, IE, Safari, and Opera.

> Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36

#### history

* 用户在浏览器访问过的 url
* `history.back()`  加载 history 列表前一个 url
* `history.forward()`  加载 history 列表后一个 url
* `history.go(number)`  -1 后退， 1 前进 

#### location

* 地址栏 url 信息，操作地址栏
* **`location.href()`**  类似超链接效果
* **`widow.open()`**

#### screen

* `screen.height`
* `scrren.width`

#### 网页跳转

* **`location.href()`** 
* **`widow.open()`**
* **`history.go()`**
* **`<a href=""></a>`**