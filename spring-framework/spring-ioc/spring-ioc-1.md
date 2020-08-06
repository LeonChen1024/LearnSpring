# 深入 Spring IoC 容器系列 1 



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

# Bean 概览

Spring IoC 容器可以管理多个bean.这些bean是通过你提供的配置元数据生成的(比如,是用XML`<bean/>` 定义).

在容器内部,这些bean定义表示为 `BeanDefinition` 对象, 包含了(除了其他信息外)以下元数据:

- 包限定类名: 通常是定义bean的实际实现类
- bean 行为配置元素, 声明了bean在容器中的行为(作用域,生命周期回调,等等)
- 这个 bean 工作所需的其他 bean 的引用. 这些引用也被称作合作者活着依赖.
- 新创建对象的一些其他配置设置. 比如, 池的大小限制活着连接数量等在一个管理连接池的 bean 中需要使用的参数.

元数据里的配置最后都会转换成 bean 的属性.一些转换的对象可以见下表:

todo

​																	bean 定义转换

| 属性  | 解释 |
| ----- | ---- |
| Class |      |
|       |      |
|       |      |
|       |      |
|       |      |
|       |      |
|       |      |
|       |      |
|       |      |

除了包含如何创建一个指定bean 的bean定义之外, `ApplicationContext` 实现还允许将容器之外创建的对象进行注册.这个操作是通过 `getBeanFactory()` 方法来访问  ApplicationContext 的 BeanFactory 来实现的, 这个操作会返回一个 `DefaultListableBeanFactory` 实现. `DefaultListableBeanFactory` 支持通过 `registerSingleton(..)` 和 `registerBeanDefinition(..)` 来注册. 然而, 通常应用只需要通过常规 bean 元数据 定义就可以了.

> Bean 元数据和手动提供单例实例要尽可能的早,这样可以使得容器在自动装配和其他内部检测步骤中正确的推断出他们.尽管在某种程度上重写已存在的元数据和单例实例是支持的,在运行时注册新的bean(和实时访问工厂同时发生)官方是不支持的,这会导致并发访问异常,Bean容器中的状态不一致,或者两者都有.

## Bean命名

每个 bean 都有一个或多个标识.这些标识在持有他们的容器中必须是唯一的.通常情况下一个 bean 只有一个标识.然而,如果他需要更多标识,其他的标识可以当作是别名.

在基于 XML 的配置元数据中,你可以使用 `id` 属性, `name` 属性,或者都使用来指定bean的标识. `id` 属性允许你指定一个 id. 通常这些名字是字母("myBean",等等),但是他们可以包含特殊字符. 如果你想要给这个bean 使用其他的别名, 你可以使用 `name` 属性来指定,使用 `,` , `;` ,或者空格来隔开.注意 bean `id` 的唯一性由容器来保证,不再由 XML 解析器检测.

你不一定要提供 `name` 或者 `id` 给bean. 如果你不显式的提供 `id` 和 `name` , 容器会生成一个唯一的名字给它. 然而,如果你想要通过名字来获取bean , 通过使用 `ref` 元素或者是一个[服务定位器风格的查找]([https://github.com/LeonChen1024/Excellent-Javaer/blob/master/principle/ioc.md#%E6%9C%8D%E5%8A%A1%E5%AE%9A%E4%BD%8D%E5%99%A8%E6%A8%A1%E5%BC%8Fservice-locator-pattern](https://github.com/LeonChen1024/Excellent-Javaer/blob/master/principle/ioc.md#服务定位器模式service-locator-pattern)) , 你必须要提供一个名字. 一般情况下不提供一个关联的名字的原因是使用了 [inner beans](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-inner-beans) and [autowiring collaborators](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-autowire).

> Bean 命名风格
>
> 通常在命名 bean 的时候是使用标准Java 规范中给实例字段命名的风格.也就是 bean 命名以小写字母开头并使用驼峰风格.比如  `accountManager`, `accountService`, `userDao` 等.
>
> 保持 bean 命名的一致性使得你的配置更容易阅读和理解.同样,如果你使用了 Spring AOP, 这在要对 bean 使用 advice 的时候是很有用的.



> 在类路径中使用组件扫描的时候,Spring 会为未命名组件生成bean 名字,遵循前面描述的规则: 使用简单类名并将它的首字母改为小写. 然而,在特殊情况下可能头两个字母都是大写,这个时候就保留原来的大写. 这和 `java.beans.Introspector.decapitalize` 的规则一致. 原因推测是由于如果开头多个连续大写一般是这个词语是特殊名词,所以保留风格.

### 在 bean 定义外定义别名

在 bean 定义本身中, 你可以给 bean 多个名字, 通过在 `id` 属性中指定一个名字和 `name` 属性中指定任意个数的名字. 这些名字对于这个 bean 来说都是等价的别名并且在某些情况下很有用, 比如为每个组件都引入一个公共依赖但是这个依赖名字对于每个组件而言又是特定的. 也就是提高了非侵入性,使得组件复用度提高,而不需要让应用的容器中 bean 定义被绑死. 

然而,只在bean 定义中指定别名不一定能应付所有场景.有时候我们想要在其它地方定义别名. 这在配置被分散到各个子系统的大型系统中是很常见的,每个子系统都有他们自己的对象定义. 在基于 XML 的配置元数据中,你可以使用  ` <alias/>` 元素来做到. 如下:

```xml
<alias name="fromName" alias="toName"/>
```

在这种情况下, 一个叫做   `fromName`  的bean (在同一个容器下) 在使用了这个别名定义之后,也可以使用 `toName` 引用.

比如,子系统 A 的配置元数据可能使用  `subsystemA-dataSource` 来引用一个数据源. 子系统 B 中的配置元数据可能使用名字  `subsystemB-dataSource` 来引用一个数据源. 当将这两个子系统组合成一个应用的时候,主应用使用名字 `myApp-dataSource` 来引用数据源. 为了让这三个名字引用到相同的对象,你可以添加以下别名定义到配置元数据:

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

现在每个和主应用可以通过一个唯一的名字来引用到数据源,并且保证不会和其他定义冲突,而且引用的还是相同的bean.

>    Java 配置
>
> 如果你使用的是 Java 配置,  `@Bean` 注解也可以使用别名, 见  [Using the `@Bean` Annotation](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-bean-annotation)  获得细节



## Bean 实例化

一个bean 定义本质上是创建对象的方法.容器根据bean 名字查找对应的方法并使用bean定义中封装的配置元数据来创建(或者 获取)一个实际的对象.

如果你使用的是基于 XML 的配置元数据,你通常必须在 `<bean/>` 元素中的 `class` 属性中指定对象的类型(或者class).这个 `class` 属性(实际上是 `BeanDefinition` 实例的 `Class` 属性 ) .(特殊情况可以看  [Instantiation by Using an Instance Factory Method](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-class-instance-factory-method) and [Bean Definition Inheritance](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-child-bean-definitions).) 你可以通过两种方式使用 `Class` 属性:

- 通常情况下,指定bean 要构造的类是为了在容器通过反射调用构造器的时候使用,等同于 Java 代码中的 `new` 操作符
- 用来指定静态工厂方法创建对象的实际类,偶尔容器调用静态工厂方法来创建一个bean.这个静态工厂方法返回的类型可能是相同的类或者是另外一个类

> 内部类命名
>
> 如果你想要配置一个静态内部类的bean 定义,你必须使用内部类的 binary 名字.
>
> 比如, 你有一个类叫做 `SomeThing` 在 `com.example` 包下,并且这个类拥有一个内部类叫做 `OtherThing` ,这个类的bean 定义 `class` 属性值应该是 `com.example.SomeThing$OtherThing`.
>
> 注意要使用 `$` 连接内部类和外部类

### 使用构造函数实例化

当你使用构造函数的方式来创建bean,所有正常的类都可以被Spring使用和兼容.也就是说,开发的类不需要实现任何制定的接口或者是使用特别的方式来编码.简单的指定bean就可以了.然而,根据你使用什么类型的IoC ,你需要一个默认(空)构造器.

Spring IoC 容器可以管理你想要被管理的所有类.不仅仅是管理真正的JavaBean.大多数用户喜欢只有一个默认(无参)构造器并且适量的getter和setter的 JavaBean.你也可以在容器里放入一些不太符合bean风格的类.比如,你需要使用一个常规的连接池它并不符合一个JavaBean的规范,Spring也可以管理它.

使用XML 配置元数据如下:

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

给构造器添加参数的机制以及在对象构造后设置实例对象的属性的内容可以看 [Injecting Dependencies](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-collaborators).

### 通过静态工厂方法实例化

当使用静态工厂方法定义一个bean的时候,使用 `class` 属性来指定包含 `static` 工厂方法的类并且使用`factory-method` 来指定工厂方法的名字.这个方法应该可以直接调用(可以带有一些参数,后面会介绍)并且返回一个可用对象,后面就可以将这个对象当作通过构造器生成的一样使用.这个bean定义也叫做静态工厂方法.

下面的bean定义指定了一个bean使用工厂方法生成.这个定义并没有指定返回对象的类型(类),只有包含了工厂方法的类.在这个例子中  `createInstance()` 必须是一个静态方法.

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

下面是一个符合这个定义的类

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

```kotlin
class ClientService private constructor() {
    companion object {
        private val clientService = ClientService()
        fun createInstance() = clientService
    }
}
```

对于提供可选的参数给工厂方法并且在工厂返回对象后设置实例属性的详细内容见  [Dependencies and Configuration in Detail](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-properties-detailed).

### 通过实例工厂方法来实例化

和通过静态工厂方法实例化类似,通过实例工厂方法实例化是通过调用容器中一个已存在bean的非静态方法来生成一个新的bean.为了使用这个机制,将 `class` 属性放空,并且在  `factory-bean`  属性中指定当前容器(或者父容器或祖先容器)包含了要调用的生成对象的实例工厂方法的bean名称.在 `factory-method` 它的工厂方法的名称.下面展示了如何配置这样的bean:

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

对应的类如下:

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```



```kotlin
class DefaultServiceLocator {
    companion object {
        private val clientService = ClientServiceImpl()
    }
    fun createClientServiceInstance(): ClientService {
        return clientService
    }
}
```

一个工厂类可以同时持有多个工厂方法,直接复制使用一个的方式即可

这个方式展示了工厂bean的配置和管理可以使用 依赖注入.见 [Dependencies and Configuration in Detail](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-properties-detailed).

> 在Spring 中, "工厂bean" 指的是一个bean配置在Spring容器中并且通过实例或者静态工厂方法创建了对象. `FactoryBean` 指的是Spring中的一个类.

### 判断一个bean 的运行时类型

判断一个指定bean的运行时类型并不是一件简单的事.一个bean定义中的指定类只是一个初始类引用,潜在的和一个工厂方法或者 `FactoryBean` 组合可能会使得这个类的运行时类型是不同的.或者是在定义中完全没有指定类的初始引用比如使用实例工厂方法,它只指定了  `factory-bean`  的名字.此外,AOP代理可能会将bean实例包裹在一个基于接口的代理这会限制了目标bean暴露它实际的类型.

推荐对指定bean名称使用 `BeanFactory.getType` 来获取实际运行时类型.这个方式考虑到了上面的所有情况并且返回对象的类型,而  `BeanFactory.getBean` 调用返回了相同的bean名字.









