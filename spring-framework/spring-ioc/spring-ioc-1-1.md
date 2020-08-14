# 深入 Spring IoC - 1.1 总概览及容器概览



[TOC]



# 概述

Inversion of Control (IoC,控制反转) [IoC 更多内容](https://github.com/LeonChen1024/Excellent-Javaer/blob/master/principle/ioc.md),这是一个设计原则,可以减少程序的耦合度.而IoC通常是和 dependency injection (DI,依赖注入) 一起出现的,可以说DI是实现IoC原则的一种主要方式.通过从外部注入依赖到使用者,使得控制创建依赖的职责反转到了外部,使得代码的耦合度减少,灵活度和扩展能力得到提高. 

Spring中提供了这种模式的实现,通过把类交给IoC容器,并告诉它你是什么,那么当有类需要用到你的时候Spring 就可以从容器中获取并注入给调用者.



## IoC 容器

在 `spring-framework` 项目中, `spring-beans` 模块下的 `org.springframework.beans` 包和 `spring-context` 模块下的 `org.springframework.context` 包是 Spring 的IoC的基础所在. 所以如果要了解Spring 中的IoC容器的话,这两个包是值的阅读的.

`BeanFactory` 接口提供了一种高级配置机制,能够管理任何类型的对象.Spring中使用原型设计模式或者单例设计模式的变种(在工厂作用域中是单例的)来管理bean实例. `ApplicationContext` 是 `BeanFactory` 的一个子接口.

如图所示

![](res/ApplicationContextDiag.png)



 `ApplicationContext` 在 `BeanFactory` 的基础上添加了一些额外内容,比如:

- 更方便和 Spring 的AOP 集成
- 用于国际化的消息资源处理
- 事件发布
- 提供了应用层特定的上下文,比如给web应用使用的 `WebApplicationContext`

也就是说 `ApplicationContext` 在 `BeanFactory` 的基础上添加了很多面向企业级开发的功能.它是包含了 `BeanFactory` IoC 容器的功能.我们一般使用它就可以了.

## Beans

在Spring 中,构成应用的骨架并且被 Spring IoC 容器管理的对象叫做 bean. bean对象会被IoC容器实例化,组装和管理.除此之外,bean也是你应用中的简单对象.bean 和它依赖都是通过容器中的配置元数据反射而来的.

# 容器概览

`ApplicationContext` 接口就可以代表 Spring IoC 容器. 容器通过配置元数据来获取应该管理那些对象.我们可以使用 XML,Java 注解或者Java代码来表示配置元数据.通过这些方式我们可以表示出应用中的组成对象和他们之间的依赖关系.

Spring 提供了一些 `ApplicationContext` 的实现,比如  `ClassPathXmlApplicationContext` 和 `FileSystemXmlApplicationContext` 以及 `AnnotationConfigApplicationContext` .在最开始的时候大部分的配置元数据都是使用xml进行配置的.所以用到前两个的频率比较高,但是到了后面xml的数量越来越多,开始变得难以维护,就开始去xml化了,大量使用 Java 代码和注解的方式减少xml的配置数量.当然,即使你使用了前两个Context 也可以通过在xml中配置一些属性开启额外的元数据格式.

Spring 容器的工作模式如spring 官方图示:

![container magic](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/images/container-magic.png)

spring 容器读取你的 pojo 并根据配置元数据进行操作后,在你的应用中就可以直接使用了.

Spring通过这种方式实现了一个应用组件的中心注册表,并且集中的配置了应用组件(使得单个对象不再需要读取配置文件).这种方式的优点是通过使用一个应用上下文或者注册表来避免单元素的激增,可以促进设计的灵活性.使我们能够把"单元素集"实现为普通Java组件;它们将通过它们的Bean属性被配置.配置管理代码将由应用上下文--一个通用框架对象来处理而不是个别应用对象来处理.应用开发人员将永不需要编写代码来读取属性文件.最大限度减少对具体API(比如专有API)的依赖性.通过将应用组件变成JavaBean 使他们可以被通用的应用框架来配置,从而消除了让应用代码来实现配置管理的需要.并且和基于接口设计结合起来,使用JavaBean就成为了一种无需修改Java 代码就可以装配和参数化应用的强有力手段.配置管理集中化使得无需修改应用代码来读取配置成为可能.通过配置JavaBean属性保证了整个应用内及体系结构层间的一致性.更多的讨论可以看 Spring 之父 Rod Johnson 大神写的 "Expert One-on-One J2EE Design and Development" 中的第4章和第11章的讨论.



## 配置元数据

配置元数据的作用是告诉 Spring 容器如何去实例化,配置,以及装配对象.传统方式是使用XML格式来表示.还可以使用注解,和Java代码表示.

通常情况下,你可以在容器中定义服务层对象,数据访问层对象(DAO),描述对象(比如 Structs 的`Action` 对象),基础对象(比如 Hibernate 的 `SessionFactories`)等等.但是一些细颗粒的领域对象就不会放到容器中管理了,一半是交给 DAO 层和业务逻辑层去创建和处理.容器中最好只放入骨架性的对象,再由这些骨架再去管理和丰富细节对象.这样的软件才符合分层的概念,而不是将所有层级的对象统统混杂在一起.当然,如果在这些层级中你也想使用依赖注入的方式实现IoC也是可以的,Spring 可以集成 AspectJ 来配置管理对象.[详情见](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#aop-atconfigurable)



### XML 表示

通常情况下容器管理了多个bean.在XML中统一放到一个顶级元素 `<beans>` 下使用 `<bean>` 元素表示.

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

` <bean id="..." class="...">`  中的

-  `id` 是一个字符串代表bean的唯一标识符
- `class` 代表的是bean的类型,使用全限定类名表示



## 实例化容器

通过给 `ApplicationContext` 提供一个字符串资源路径使得容器可以在这些外部资源中加载配置元数据,比如本地文件系统,Java `CLASSPATH` 等等.

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

比如有这么两个xml.

services.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/> 
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

`<property name="accountDao" ref="accountDao"/> ` 其中的 `name` 指的是这个参数名的 JavaBean , `ref` 指的是引用这个名字的 bean 定义. 也就是引用到下一个XML 中对应的 `id` 属性.

daos.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```



### 组合基于XML 的配置元数据

通过使用多个 XML 文件来声明 bean 定义是非常有用的.通常,每个单独的XML配置文件代表一个逻辑层或者你架构中的模块.

你可以直接往应用上下文构造器中传递所有的 XML 配置.这个构造器接收多个 `Resource` 定位,也就是如同上一部分的那样.我们还可以在XML中使用 `<import/>` 元素来加载其他的bean定义文件.如下所示:

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

这种方式导入的路径都是与该文件的相对路径. 也就是说 `services.xml` 必须和该文件在同一个文件夹层级.并且开头的"/"符号也会被忽略.

> 虽然可以使用"../" 来引用到上一个层级的文件,但是不推荐这么使用.这样做可能会让我们依赖一个不属于这个应用的文件.会导致额外的维护成本. **特别要注意**不推荐对`classpath:` URL 使用(比如,`classpath:../services.xml` ),在运行时会选择"最近"的classpath 根目录然后查找它的父文件夹.Classpath 配置改变可能会导致这个选择变得不同.提高我们的维护成本.
>
> 你也可以使用全限定名资源路径而不是相对路径:比如,`file:C:/config/services.xml`或者`classpath:/config/services.xml`.然而,要注意你这是将你的应用配置和指定的绝对路径耦合起来.通常要为这种绝对定位保留一个间接地址,比如通过运行时针对JVM解析 "${…}" 占位符.

除了普通的配置功能,Spring 提供的一系列XML命名空间还提供了超出普通bean定义的配置功能(比如  `context` 和 `util` 命名空间 ).



### Groovy Bean 定义 DSL

Spring 同时支持使用 Spring’s Groovy Bean Definition DSL来进行bean 定义,和 Grails 框架类似.通常是在一个  ".groovy"  文件中,结构如下:

```groovy
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

它和 XML bean定义基本是相同的,也支持Spring 的XML配置命名空间.还可以通过 `importBeans` 导入XML bean定义.



## 使用容器

`ApplicationContext` 是给高级工厂使用的接口,它维护了一个注册表管理不同的bean和它们的依赖.通过使用方法 `T getBean(String name, Class<T> requiredType)` 你可以获得一个你需要的bean实例.

`ApplicationContext` 使得你可以读取bean定义并且访问它们,如下所示:

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

`ClassPathXmlApplicationContext` 已经绑定了XML bean 读取器,所以如果你是使用其他bean定义的话要使用别的上下文,比如 `GenericGroovyApplicationContext` 可以读取 `.groovy` 配置,或者是更灵活的做法,使用 `GenericApplicationContext` 上下文,搭配 `XmlBeanDefinitionReader` 或者 `GroovyBeanDefinitionReader` .

虽然我们可以使用 `getBean` 来获取bean实例,或者是`ApplicationContext` 提供的其他的获取方法,但是我们不应该使用它们.这样可以避免依赖Spring提供的API.Spring提供了元数据的方式声明依赖(比如使用自动装配注解).

