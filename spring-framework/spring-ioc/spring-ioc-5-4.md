

# 深入 Spring IoC 系列 - 5.4:  自定义作用域



[TOC]

## 自定义作用域

bean 作用域机制是可继承的.你可以定义自己的作用域,甚至重新定义现有作用域,后者被认为是不好的做法，并且你不能重写内置的 `singleton` 和 `prototype` 作用域

### 创建一个自定义作用域

为了集成你的自定义作用域到 Spring 容器中,你需要实现 `org.springframework.beans.factory.config.Scope` 接口,本章会介绍.要了解更详细的实现自定义作用域的方法可以参考Spring 中 `Scope` 的实现还有 [Scope](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/Scope.html) 的 javadoc.

`Scope` 接口有四个方法来从作用域中获取对象,移除对象,以及 销毁对象.

比如 session 作用域实现,会返回一个 session 作用域内的 bean(如果不存在,会返回一个 bean 的新实例,并会将这个 bean 绑定到这个 session 以供之后使用).下面的方法返回作用域下的对象

```java
Object get(String name, ObjectFactory<?> objectFactory)
```

比如 session 作用域实现,移除 session 作用域下所有session 作用域的 bean.会返回这个 bean,如果找不到指定名称的对象会返回 null.下面的方法从当前 scope 中移除对象

```java
Object remove(String name)
```

下面的方法注册了当作用域销毁或者指定作用域对象销毁的时候要执行的回调

```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```

想要了解更多销毁回调的内容详见[javadoc](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/factory/config/Scope.html#registerDestructionCallback)

下面的方法获取当前作用域下的会话标识

```java
String getConversationId()
```

每个作用域的标识都是不同的.对于一个 session 作用域实现,这个标识可以是 session 标识



### 使用自定义作用域

在你编写和测试了自定义`Scope` 实现之后,你要让 Spring 容器知道你的心作用域.下面的方法是 Spring 容器中注册新的 `Scope` 的中心方法

```java
void registerScope(String scopeName, Scope scope);
```

这个方法是在 `ConfigurableBeanFactory` 中声明的,可以通过 Spring 提供的大多数 `ApplicationContext` 实现的`BeanFactory` 来获取.

这个方法的第一个参数是这个 Scope 的独有的名字.比如 Spring 容器中的 `singleton` 和 `prototype` .第二个参数是这个自定义 `Scope` 的实现实例

假设你编写自定义 `Scope` 实现,并且像下面的例子那样注册它

> 下面的例子使用了 `SimpleThreadScope` ,它是 Spring 里已有的但是默认没有注册.和你注册自定义 `Scope` 实现是一样的过程

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

然后你可以使用你的自定义`Scope` 来编写 bean 定义,如下:

```xml
<bean id="..." class="..." scope="thread">
```

你不仅可以使用编程式的注册.还可以使用声明式的注册,使用 `CustomScopeConfigurer` 类,如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>

</beans>
```

> 当你在一个`FactoryBean` 实现中使用 `<aop:scoped-proxy/>` ,它限定的作用于是这个工厂 bean 自身,而不是 `getObject()` 返回的对象















































