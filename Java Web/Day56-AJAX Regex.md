# Regex

#### 内容摘要

Regex 概述，常用 Regex，Rexgex 规则，AJAX 概述，使用流程，XmlHttpRequest

---

#### Regex 概述

* **正则表达式 Regular Expression** **RE**  **RegExp**
* 用来**判断**、检索、替换符合**规则**的字符串
* **Java**  `matches(String regex)`   `"\\d{6}"`
* **JavaScript** `test(String: regex)`   `/^\d{6}$/.test(string)`
  * `/^       $/.test()` 必要

#### 常用 Regex

* 邮政编码： `\\d{6}`
* 汉字：`[\u4E00-\u9FA5]` 

#### Regex 规则

匹配符合规则的一模一样的字符串

* 次数限定符，**前一个字符或括号内字符串**
  * `*` ：任意次
  * `+` ：至少一次或多次
  * `?` ：零次或一次
  * `{n}` ：**n** 是一个非负整数， **n** 次
  * `{n,}` ：**n** 是一个非负整数，至少 **n** 次
  * `{n,m}` ：**n、m** 是一个非负整数，**n<m**，至少 **n** 次，至多 **m** 次，`,` 之间没有空格
* 字符集
  * `x|y` ：**左右两边字符串，x 或 y**
  * `[xyz]` ：**一个字符，匹配 xyz 中任一字符**
  * `[^xyz]` ：**一个字符，匹配不包含的 xyz 的任一字符**
  * `[a-z]` ：**一个字符，匹配 a 到 z 范围内的任一字符**  *a、z 可以为数字*
  * `[^a-z]` ：**一个字符，匹配不在 a 到 z 范围内的任意字符**
* 特殊字符
  * `^` `$` ：正则首位开始 `^` 、末位结束 `$`
  * `\d` ：一个字符，匹配数字字符，`[0-9]`
  * `\D` ：一个字符，匹配非数字字符，`[^0-9]`
  * `\w` ：一个字符，匹配任何子类字符，包括下划线，`[A-Za-z0-9_]`
  * `\W` ：一个字符，匹配任何非单词字符，`[^A-Za-z0-9_]`
* 其他
  * `\` ：转义字符
  * `s` ：匹配任何空白字符，包括空格、制表符、换页符，`[\f\n\r\t\v]`
  * `S` ：匹配非任何空白字符
  * `.` ：匹配除 **\n** 之外的任意字符，要匹配 `.`  需要 `\.` 或  `[.]`

## AJAX

#### AJAX 概述

* **AJAX Asynchronous JavaScript And XML**
* 基于服务器的页面异步处理技术
* 应用场景：地图、搜索建议

#### 使用流程 JavaScript

1. 创建 **XMLHttpRequest** 对象
2. 注册状态监听回调函数 **onreadystatechange**
   * **readyState  4**
   * **status  200**
   * **responseText**  拿到响应正文
3. 建立与服务器的异步连接 **open(String method, String url, boolean async)**
   * JSP 中可以使用相对路径 `${pageContext.request.contexPath}/check?username=?`
4. 发出异步请求 **send()**

```javascript
// 1. 创建 XMLHttpRequest 对象
var xmlHttpRequest = new XMLHttpRequest();

// 2. 注册状态监听回调函数
xmlHttpRequest.onreadystatechange = function () {
  if (xmlHttpRequest.readyState == 4 && xmlHttpRequest.status == 200) {
    	// 在此处理服务器响应报文  
  }
};

// 3. 建立与服务器的异步连接
var url = "${pageContext.request.contextPath}/check?username=" + username;
xmlHttpRequest.open("GET", url, true);

// 4. 发出异步请求
xmlHttpRequest.send();
```

#### XmlHttpRequest

* ReadyState
  * **0 XMLHttpRequest** 初始化
  * **1 XMLHttpRequest open**  
  * **2 XMLHttpRequest send**
  * **3 XMLHttpRequest 收到响应头**
  * **4 XMLHttpRequest 响应接收完毕**


* API 
  * `open(String method, String url, boolean async)`
  * `send(String post)` 可以发送 POST 请求参数
  * `sendRequestHeader()`
  * `onreadystatechange`
  * `readyState`  
  * `responseText`
  * `status`
    * 200









