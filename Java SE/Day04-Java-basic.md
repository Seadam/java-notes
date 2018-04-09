## Java 基础03

#### 内容摘要

运算符优先级、键盘录入数据、流程控制、顺序结构、选择结构、循环结构

---

#### 优先级

* ` +` `-` ：优先级高于单目运算符


* 其他更详细的优先级不需要记忆，可以通过加括号来改变优先级

#### 键盘录入数据：

* Scanner

```Java
// 引入包（放到 class 定义上面）
import java.util.Scanner;

// 创建对象
Scanner scanner = new Scanner(System.in);

// 接收数据
int x = scanner.nextInt();
```

#### 流程控制

* 在一个程序执行过程中，程序的流程对运行结果有直接的影响
* 分类
  * 顺序结构
  * 选择结构
  * 循环结构

#### 顺序结构

* 按照代码先后顺序，依次执行，写在前的先执行，写在后的后执行

#### 选择结构

* 也叫分支结构，`if` `switch`

* 逻辑运算结果有两个，产生选择，按照不同的选择执行不同的代码

* `if`

  * 关系表达式结果必须是 `boolean` 类型
  * 如果执行体里面只有一条语句，大括号可以省略
  * 有左大括号就没有分号，有分号就没有左大括号
  * 三目运算符可以转化为 `if` 语句，反之不一定，如：输出语句时，三目运算符不行

  ```Java
  /*
  	if(关系表达式) {
  		语句体
  	}
  */

  if(...) {
    // if true then to do
  }

  if(...) {
    // true then to do
  } else {
    // false then to do 
  }

  if(...) {
    // true then to do 
  } else if(...) {
    // true then to do 
  } else {
    // false then to do
  }
  ```

* `switch`

  * 表达式的取值可以是 `byte` `short` `int` `char` `枚举` `String`

  * `case` 中的值是要和表达式相比的值，只能是常量，不能是变量，不能重复

  * 执行过程：计算出表达式的值，与每一个case的值进行匹配( `equalTo` )，匹配成功则执行该 case 语句

  * `break` 表示中断，可以结束 switch 语句，**如果没有 break 就会发生 case 穿越现象**

    * **case 穿越**：从匹配到的 case 开始，依次执行以下所有分支，直到匹配完毕或遇到下一个 break

    * 可以利用 case 穿越实现需求

      ```Java
      switch(month) {
          case 1:
      	case 2:
      	case 3:
      		System.out.println("Spring");
      	case 4:
      	case 5:
      	case 6:
      		System.out.println('Summer');
      }
      // 也可以将12个月映射到4个case x = (x - 1) / 3
      switch( (month - 1) / 3) {
        case 0:
          	System.out.println("Spring");
        case 1:
          	System.out.println("Summer");
      }
      ```

  * `default` 所有情况不匹配时执行该语句，类似于 `else` ，位置可以出现在 switch 语句中任意位置

  * break, default 可以省略，但不建议

  ```Java
  /*
      switch(表达式)	{
          case 值1:
              语句1;
              break;
          case 值2:
              语句2;
              break;
          ...
          default:
              break;
      }
  */
  switch(...) {
    case 1:
      // to do
      break;
    case 2:
      // to do
      break;
    default:
      //false then to do
      break;
  }
  ```


#### 循环结构

* `while` `do….while` `for`

* 在满足循环条件的情况下，反复执行某一段代码

* `for` 

  * 适合针对一个范围判断进行操作

  ```Java
  /*
      for(初始化语句；判断条件语句；控制条件语句) {
          循环体语句；
      }
  */
  for(int 1 = 1; i <= 10; i ++) {
    System.out.println(i);
  }
  ```

* `while`

  * 适合判断次数不明确的操作

  ```Java
  /*
      while(判断条件语句) {
          循环体语句；
          控制条件语句；
      }
  */
  int i = 1;
  while(i <= 10) {
    System.out.println(i);
    i++;
  }
  ```

* `do…while`

  * 循环体至少执行1次，再判断条件语句

  ```Java
  /*
      do {
      	循环体语句；
      	控制条件语句
      } while(判断条件语句);
  */
  ```

* 死循环 `while(true) {}` `for(;;) {} `

* 循环控制

  * `break`
  * `continue`
  * `return`









