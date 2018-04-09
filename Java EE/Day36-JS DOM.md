# JavaScript

#### 内容摘要

获取 Element 节点，创建节点，添加节点，删除节点，删除节点，拿到节点的值，复制节点，Array 补充

---

#### 获取 Element 节点

* `getElementById(String)`  返回指定 id 的第一个节点的引用
* `getElementByName(String)`  返回指定 name 的节点集合，数组
* `getElementByTagName(String)`  返回指定 TagName 的 节点集合，数组
* `parent.childNodes`  当前 parent 节点所有孩子节点集合，数组
  * **换行被当作文本节点**，要特别注意
* `parentNode`  当前节点的父节点
* `firstNode`  当前节点的第一个子节点
* `lastNode`  当前节点的最后一个节点

#### 创建节点

* `creatElementNode()`  创建元素节点
* `creatAttributeNode(String)`  创建属性节点
* `creatTextNode()`  创建文本节点

#### 添加节点

* `parent.appendChild(node)`  将 node 插入到父节点的最后一个子节点位置
* `parent.insertBefore(node, before node) `  将 node 插入到父节点的 before 节点前的位置

#### 删除节点

* `parent.removeChild(node)`  在父节点上删除子节点 node 
* `textNode.deleteData(offset, length)`  删除 文本 节点，从 offset 开始，删除 len 个字符长度
* `elementNode.removeAttribute(String)`  删除元素 属性名为 String 的属性值

#### 修改节点

* `parent.replaceChild(newNode, oldNeode)`   父节点上用 newNode 替换 oldNode
* `textNode.replaceData(offset, length, String)`  文本节点从 offset 开始 length 长度替换为 String
* `elementNode.setAttribute(name value)`  元素节点属性名为 name 的属性值设置为 value

#### 拿到节点的值

* `innerText`
* `innerHtml`
* `nodeValue`  // text attribute 
* **换行被当作文本节点**

#### 复制节点

* `node.cloneNode(deep)`  返回一个新克隆节点，deep : boolean
  * true 深度克隆，当前节点和属性，及其子节点
  * false 浅度克隆，当前节点及其属性

#### Array 对象补充

* 当以字符串作为下标在字符串中存储数据时，存储的数据是作为数组的属性添加的

```javascript
var arr = [];
arr["first"] = [1,2,3,4];
arr["second"] = [5,6,7,8];
// 以字符串作为下标存储数据， arr.length = 0;
```







