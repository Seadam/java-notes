# Spring AOP

#### 内容摘要

AOP 简介，原理，术语，实战，Proxy，CGLib，Spring AOP，AspectJ，AspectJ 注解

---

#### AOP 简介

***Aspect Oriented Programming*** 面向切面编程，代码重用 —— 切面

通过预编译方式和运行期动态代理实现程序功能，**增强**功能

特点

* AOP 采取横向抽取机制，取代了传统纵向继承重复性代码
* Spring AOP 使用纯 Java 实现，在运行期间通过动态代理方式**织入增强**代码

经典应用

* 事务管理、性能监视、安全检查、缓存、日志等

#### AOP 原理

**JDK 动态代理 Proxy**

* 接口 + 实现类
* 缺点：如果目标类没有实现接口无法使用

**cglib 字节码增强**

* 实现类，无需接口也能使用
* 基于继承
* 缺点：不能被继承的类无法使用，**final class**

#### AOP 术语

**Target 目标类**：需要被增强（代理）类

**JoinPoint 连接点**：**可能会**被增强的点（方法）

* **坑坑坑**
* `org.aopalliance.intercept.Joinpoint`  小写的 **p** 要小心
* `org.aspectj.lang.JoinPoint`  使用此包！！！ **JoinPoint**  

**Pointcut 切入点**：已经被增强的连接点（方法）

**Advice 通知（增强）**：执行到 **JoinPoint** 连接点 所做的事情，增强的代码

**Weaver 织入**：把 **Advice** 增强的代码应用到目标对象创建新的代理对象

**Proxy 代理类**：增强（代理）的类

**Aspect 切面**：**PointCut + Advice** （增强后的方法）

#### AOP 实战

手动方式

* **Proxy**
* **CGLib**

半自动方式

* **Spring AOP**

自动方式

* **AspectJ**

#### Proxy

JDK 实现 Proxy 动态代理，见 Day 68 动态代理模式

手动 **PostBeanProcessor** 后处理增强 Bean

```java
public class ProxyProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if ("service".equals(beanName)) {
            return Proxy.newProxyInstance(
                bean.getClass().getClassLoader(),
                bean.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) {
                        boolean flag = false;
                        try {
                            TransactionUtil.beginTransaction();
                            method.invoke(bean, args);
                            TransactionUtil.commitTransaction();
                            flag = true;
                        } catch (Exception e) {
                            System.out.println("代理捕获到异常：" + e.getCause());
                            TransactionUtil.rollbackTransaction();
                        }
                        return flag;
                    }
                }
            );
        } else {
            return bean;
        }
    }
}
```

#### CGLib

`org.springframework.cglib.proxy.Enhancer`

* `Object create(Class superClass, Callback invocationHandler)`
* **superClass** 被代理类
* **invocationHandler** 代理动作执行

使用继承来生成代理类，**final Class** 无法使用

手动 **PostBeanProcessor** 后处理增强 Bean

```java
public class CglibProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
        throws BeansException {
        if ("service".equals(beanName)) {
            return Enhancer.create(AccountServiceImpl.class,
                    new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) 
                    throws Throwable {
                    boolean flag = false;
                    try {
                        TransactionUtil.beginTransaction();
                        method.invoke(bean, args);
                        TransactionUtil.commitTransaction();
                        flag = true;
                    } catch (Exception e) {
                        System.out.println("代理捕获到异常：" + e.getCause());
                        TransactionUtil.rollbackTransaction();
                    }
                    return flag;
                }
            });
        } else {
            return bean;
        }
    }
}
```

#### Spring AOP

导包（4 + 1 + **2**）

* core 、 expression 、beans 、context 、logging


* `spring-aop-5.0.4.RELEASE.jar`
* `com.springsource.org.aopalliance-1.0.0.jar`

半自动，不用写 **PostBeanProcessor**

* 使用静态工厂生成代理 `org.springframework.aop.framework.ProxyFactoryBean`


* 如果有实现接口，内部使用 **JDK Proxy** 动态代理
  * `AccountServiceImpl$$FastClassBySpringCGLIB$$fcaf2c2a`
* 如果没有实现接口，内部使用 **CGLib** 动态代理 
  * `com.sum.proxy.$Proxy15`

不足之处

* 需要增强的方法需要写代码进行判断
* 增强的代码有要求，实现 `MethodInterceptor` 接口

配置 xml

```xml
<bean id="service" class="demo.service.impl.ServiceImpl">
    <property name="dao" ref="dao"/>
</bean>

<bean id="advise" class="demo.Advise"/>

<bean id="serviceProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="service"/>
    <!-- Stirng[] interceptorNames 字符串数组用 value -->
    <property name="interceptorNames" value="advise"/>
</bean>            
```

**Advise** 实现 `org.aopalliance.intercept.MethodInterceptor` 接口

* `Object invoke(MethodInvocation methodInvocation)`
  * `methodInvocation.proceed()` 可以不关心参数，只需要调用即可

```java
public class Advise implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation methodInvocation) {
        boolean flag = false;
        try {
            TransactionUtil.beginTransaction();
            methodInvocation.proceed();
            TransactionUtil.commitTransaction();
            flag = true;
        } catch (Throwable e) {
            System.out.println("代理捕获到异常：" + e.getMessage());
            TransactionUtil.rollbackTransaction();
        }
        return flag;
    }
}
```

#### AspectJ

**AspectJ** 语法，特定的编译器生成 **Java** 字节码文件

* 切入点表达式
* 切面可以是实现 `MethodInterceptor` 接口的类，也可以是 **AspectJ** 推荐的 **Advise** 类


* 不改动现有代码的前提下，**织入代码**
* 允许在 **POJO** 类定义切面
  * **POJO**  *plain old java object* 普通 **Java** 对象

导包（4 + 1 + 2 + **2**）

* core 、 expression 、beans 、context 、logging 、aop 、aopalliance


* `spring-aspects-5.0.4.RELEASE.jar`
* `com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar`
  * *Could not initialize class org.aspectj.util.Lang Util*
  * 换个高版本的 jar 包 `aspectjweaver-1.8.13.jar`

引入约束

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">
    <context:component-scan base-package="day70.spring.demo.transaction"/>
</beans>
```

配置 xml 

```xml
<bean id="advise" class="day70.spring.demo.aop.Advise"/>
<aop:config>
    <aop:pointcut id="myPointcut" 
 expression="execution(public boolean day70.spring.demo.transaction.
             service.impl.AccountServiceImpl.transfer(..) 
             throws java.sql.SQLException)"/>
    <aop:advisor advice-ref="advise" pointcut-ref="myPointcut"/>
</aop:config>
```

**expression **切入点表达式

* **execution** （修饰符 返回值类型 包名.类名.方法名（方法参数）throws 异常列表）
* 修饰符
  * 一般省略
* 返回值类型
  * `*` 通配
* 包名
  * 可以省略 `.` 占位
  * `*` 通配，一个星表示一层包 `com.sample.www` => `*.*.*`
* 类名
  * 可以省略 `.` 占位
  * `*` 通配
* 方法名
  * **不能省略**
  * `*` 通配
* 方法参数
  * `()`
  * `(String, int)` 只需要写出方法参数类型
  * **`(..)` 可变参数**
* throws 异常类型
  * 可以省略，一般不写
  * 如果不省略，要写全类名 `java.sql.SQLException`

**AspectJ Advise**  类型

* **Before** 前置通知，切入点之前执行，参数类型校验
* **AfterReturning** 后置通知，切入点之后执行，日志
* **Around** 环绕通知，在切入点前后都会执行，事务
* **AfterThrowing** 切入点抛出异常后执行，可以拿到异常参数
* **After** 最终通知，等同 **finally**，无论发生异常都会执行
* 可选方法参数
  * `(ProceedingJoinPoint joinPoint)` around 
  * `(JoinPoint joinPoint)` before, afterReturning, after
  * `(JoinPoint joinPoint, Throwable e)` afterThrowing
    * **Throwable e**  需要在配置文件声明，参数名要对应
    *  `<aop:after-throwing .. throwing="e"/>`

```java
try {
    before();
    around() {
    	method.proceed();
    }
    atferReturning();
} catch (Exception e) {
    atferThrowing();
} finally {
    after();
}

Annotation.around.before
Annotation.before
AnnotationOrder.show
Annotation.around.after
Annotation.after
Annotation.afterReturning
--------------------------
Xml.before
Xml.around.before
XmlOrder.show
Xml.after
Xml.around.after
Xml.afterReturning
```

```java
// around
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    System.out.println("before");
    Obejct proceed = joinPoint.proceed();
    System.out.println("after");
    return proceed;
}
```

基于 xml 实现

```xml
<aop:config>
    <aop:aspect ref="advise">
        <aop:pointcut id="pointcut" expression="execution(* transfer(..))"/>
        <aop:before method="before" pointcut-ref="pointcut"/>
        <aop:after-returning method="afterReturning" pointcut-ref="pointcut"/>
        <aop:around method="around" pointcut-ref="pointcut"/>
        <aop:after-throwing method="afterThrowing" pointcut-ref="pointcut" 
                            throwing="e"/>
        <aop:after method="after" pointcut-ref="pointcut"/>
    </aop:aspect>
</aop:config>
```

#### 基于注解的 AspectJ

配置 xml

```xml
<context:component-scan base-package="day70.spring"/>
<aop:aspectj-autoproxy/>
```

**CLASS** `@Aspect`  ==>  `<aop:aspect ref="advise"/>`

**METHOD** `@Pointcut(expression)` 

* `public void pointcut()`  ==> `<aop:pointcut id="pointcut"/>`
* **expression** ==>  `<aop:pointcut expression=""/>`

**METHOD** `@Before(pointcut())`  ==> `<aop:before pointcut-ref="pointcut"/>`

**METHOD** `@AfterReturning("pointcut()")`  ==>  `<aop:after-returning pointcut-ref="pointcut"/>`

**METHOD** `@Around("pointcut()")`  ==>  `<aop:around pointcut-ref="pointcut"/>`

**METHOD** `@After("pointcut()")`  ==>   `<aop:after pointcut-ref="pointcut"/>`

**METHOD** `@AfterThrowing(pointcut = "pointcut()", throwing = "e")`

* ` <aop:after-throwing pointcut-ref="pointcut" throwing="e"/>`

```java
@Aspect
@Component
public class Advise {

    @Pointcut("execution(* transfer(..))")
    private void pointcut(){} // 返回值必须是 void

    @Before("pointcut()")
    public void before() {
        System.out.println("Advise.before");
    }

    @AfterReturning("pointcut()")
    public void afterReturning() {
        System.out.println("Advise.afterReturning");
    }

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("Advise.around.before");
        Object proceed = joinPoint.proceed();
        System.out.println("Advise.around.after");
        return proceed;
    }

    @AfterThrowing(pointcut = "pointcut()", throwing = "e")
    public void afterThrowing(Throwable e) {
        System.out.println("Advise.afterThrowing: " + e.getMessage());
    }

    @After("pointcut()")
    public void after() {
        System.out.println("Advise.after");
    }
}
```

#### 切点表达式 expression

```java
execution(modifiers-pattern?
          ret-type-pattern
          declaring-type-pattern?
          name-pattern(param-pattern)
          throws-pattern?)
```

`execution` 匹配方法调用

```java
// 匹配方法名为 transfer 的所有方法
@Pointcut("execution(* transfer(..))")
```

`within` 限制匹配确定类型

`this` 限制切面 Aspect 实例引用

`target` 限制匹配到 Bean 实例引用

`args` 限制匹配到方法所传递到切点方法的参数

`@target`

`@args`

`@within`





