

# 深入 Spring IoC 系列 - 2.2: 依赖注入细节



[TOC]



## 依赖和配置细节

如前面[文章](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-collaborators)所说,你可以定义属性和构造器参数为其它被管理的bean引用或者使用一个基础值. Spring中基于XML的配置元数据支持使用 `<property/>` 和 `<constructor-arg/>` 子元素来支持这种需求.



### 直接值(基础类型,子符串,等) 

`<property/>` 中的 `value` 属性使用一个人类可读的字符串表达式指定了一个属性或者是构造器参数. Spring 的[转换服务](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#core-convert-ConversionService-API)用来将这些值从一个 `String` 转换为一个属性或者参数的实际类型.下面的例子展示了设置多种值的情况:

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

下面的例子使用了[p命名空间](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-p-namespace) 得到一个更加简洁的 XML 配置:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>

</beans>
```

你还可以配置一个 `java.util.Properties` 实例,如下:

```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

Spring 容器将会将 `<value/>` 元素里的内容通过JavaBean `PropertyEditor` 机制转换为 `java.util.Properties` 实例. 这是一个好用的捷径,并且是 Spring 团队更加喜欢使用的方式,而不是使用 `value` 属性风格



**`idref` 元素**

`idref` 元素是一种在容器中的其它 bean 元素中传递 `id` (一个字符串值-不是一个引用) 的防错方式.下面的例子展示如何使用它:

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

上面的bean定义和下面的片段的作用(在运行时)是一样的

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

我们更推荐使用第一种方式,因为使用 `idref` 标签使得容器在部署的时候可以检验被引用的bean确实存在.而第二种变体,当值被传到 `client` bean 的 `targetName` 属性中时不会进行校验.当对 `client` 进行实例化的时候只有拼写异常会被发现.如果 `client` bean 是一个 [原型](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes) bean ,这个错别字和导致的异常可能只有在容器部署之后的很长时间才会被发现.

> 一个常见的用途(至少在Spring 2.0之前)是在`ProxyFactoryBean` 定义中配置 [AOP 拦截器](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#aop-pfb-1) 的时候使用 `<idref/>` . 可以让你在指定拦截器名字的时候防止你拼错一个拦截器ID.



### 引用其它的Bean (协作者)

`ref` 元素是 `<constructor-arg/>` 或者 `<property/>` 中最后的元素. 在这里,你设置一个bean的指定的属性为容器管理的另一个bean(协作者).被引用的bean是设置属性的bean的依赖,并且是在属性被设置之前按需加载.(如果这个协作者是一个单例bean,它可能已经被容器初始化了) 所有的引用最终都是指向另一个对象的引用.作用域和校验取决于你通过 `bean` 还是 `parent` 属性指定了其它对象的ID或者名字.



























