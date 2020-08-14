# 深入 Spring IoC - 1.2 Bean 概览



[TOC]



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
>    如果你使用的是 Java 配置,  `@Bean` 注解也可以使用别名, 见  [Using the `@Bean` Annotation](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-bean-annotation)  获得细节



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









