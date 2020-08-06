

# 深入 Spring IoC 容器系列 2



[TOC]

 # 依赖

一个常规的企业应用不会只由一个对象(或者是Spring 中的bean)组成.甚至是最简单的应用也会有一些对象共同作用来代表用户最终看到的应用.接下来这章解释你要如何定义一些独立的bean到使用对象合作来达成一个目标.



## 依赖注入

依赖注入(DI,更多关于依赖注入[点此](https://leonchen1024.com/2020/06/03/dependency-injection/))是一个定义他们的依赖(也就是运行时需要的其他对象)的方式,依赖注入只通过构造函数参数,工厂方法参数,或者是通过实例设置的属性或者是工厂方法的返回.然后容器会在创建这个bean的时候将这些依赖注入到bean中.这个方式从根本上来说是通过类的构造函数或者是服务定位模式(更多信息[点此跳转文章中服务定位器模式章节](https://leonchen1024.com/2020/06/03/ioc/))对bean自参控制实例化或者定位自身依赖的反转(控制反转,更多信息[点此](https://leonchen1024.com/2020/06/03/ioc/))

使用 DI 使得代码更加的简洁,并且可以更加高效的解耦.对象不需要查找他们的依赖并且不需要知道依赖类的位置.因此,你的类更加容易测试,尤其是当依赖是在接口或者抽象基类的时候,这使得在单元测试中可以使用stub和mock实现.

DI 有两种主要的类型: [基于构造器的依赖注入](#基于构造器的依赖注入) 和 [基于setter的依赖注入](#基于setter的依赖注入)

### 基于构造器的依赖注入

基于构造器的DI是通过容器调用一个带有一定数量的参数的构造器实现的,每个参数代表了一个依赖.与调用一个 `static` 工厂方法并带有指定的参数来构建一个bean基本上是等同的,并且对于构造器的参数和 `static` 工厂方法基本也是相似的.下面的例子展示了一个类只能通过构造器注入的依赖注入:

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

注意这个类没有什么特殊的地方.它是一个不依赖于指定接口,基类或者是注解的 POJO 

#### 构造器参数解析

构造器参数解析是使用参数类型实现的.如果在bean定义中没有潜在的歧义出现的话,在bean实例化的时候bean定义中的构造器参数的顺序就是提供给合适的构造器的参数顺序.思考如下类:

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

假设 `ThingTwo` 和 `ThingThree` 从继承的角度来说是不相关的,不存在潜在的歧义.因此,下面的配置可以正常运行,你也不需要在 `<constructor-arg/>` 中显式指定构造器参数索引或者是类型.

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当引用了另外一个bean的时候,这个类型是已知的,并且可以发生匹配(和前面的例子相同).当使用了一个简单的类型时,比如 `<value>true</value>` ,Spring 无法判断这个值的类型,因此无法在没有其他帮助的情况下通过类型匹配.参考如下类:

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```



**构造器参数类型匹配**

在前面的场景中,如果你通过使用 `type` 属性显式指定了构造器参数的类型,容器就可以对简单类型使用类型匹配.如下所示:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```



**构造器参数索引**

你可以使用 `index` 属性来显式指定它在构造器参数中的索引,如下所示:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

除了可以消除多个简单值的类型歧义,指定索引值还可以解决构造器中用有两个参数是相同类型的歧义.



**构造器参数名**

你还可以使用构造器参数名来消除值歧义,如下:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

要记得,要让这个功能开箱即用的话,你的代码编译的时候必须启用 debug 标识,这样Spring 可以从构造器中查找参数名.如果你无法或者是不想使用 debug标识来编译你的代码,你可以使用 [@ConstructorProperties](https://download.oracle.com/javase/8/docs/api/java/beans/ConstructorProperties.html) JDK 注解来显式命名你的构造器参数.如下:

```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```



### 基于setter的依赖注入

基于setter 的DI是通过容器在调用一个无参构造器或者一个无参 `static` 工厂方法来实例化bean 之后调用bean 的setter方法实现的.

下面的例子展示了一个类只能通过 setter 的方式依赖注入.这个类是一个常规的Java.是一个不依赖于容器指定接口,基类,或者注解的POJO.

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

`ApplicationContext` 管理的bean 支持基于构造器和setter的DI.它同样支持已经使用了构造器注入的方式的bean再使用setter注入的DI. 你可以使用 `BeanDefinition` 的形式配置依赖,同时使用 `PropertyEditor` 来将属性从一个格式转为另一个.然而,大部分Spring用户不会直接使用这些类(这是符合编程风格的)而是使用 XML `bean` 定义,注解组件(也就是注解了 `@Component` 之类的类),或者是java 中 `@Configuration` 类中的 `@Bean` 方法. 这些源然后会被内部转化为 `BeanDefinition` 实例并且用来加载一个完整的 Spring IoC 容器实例.

> 基于构造器还是要基于setter?
>
> 因为你可以混合使用基于构造器和基于setter的DI,一个好的方式是对于必须依赖项使用构造器方式而可选依赖则使用setter方式.注意可以使用  [@Required](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-required-annotation) 注解到一个setter方法上使得这个依赖变成必须的;尽管如此,推荐还是使用带有程序校验参数的构造器注入.
>
> Spring团队通常提倡注入构造函数,因为它使您可以将应用程序组件实现为不可变的对象,并确保所需的依赖项不为 `null` .此外,注入构造函数的组件始终以完全初始化的状态返回给客户端(调用)代码.附带说明一下,大量的构造函数自变量是一种不好的代码风味,这表明该类可能承担了太多的职责,应进行重构以更好地解决关注点分离问题.
>
> Setter注入主要应该只用于可以在类中分配合理的默认值的可选依赖项.否则,必须在代码使用依赖项的任何地方执行非空检查.setter注入的一个好处是,setter方法可使该类的对象在以后合适的时侯重新配置或重新注入.因此,通过 [JMX MBeans](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/integration.html#jmx) 进行管理是强制使用setter注入的用例.
>
> 使用最适合指定类的DI风格.有时,在处理您没有源代码的第三方类时,你可能没有选择的权力.例如,如果第三方类未公开任何setter方法,则构造函数注入可能是DI的唯一可用形式



### 依赖解析过程

容器按照如下过程执行 bean 依赖关系解析:

- `ApplicationContext` 的创建和初始化需要描述了所有bean的配置元数据.配置元数据可以通过 XML , Java 代码 ,或者是注解进行指定.
- 对于每个bean,它的依赖会在 属性,构造器参数,或者是静态工厂方法参数(如果你使用它来代替常用的构造器) 中表示. 当bean 被创建的时候这些依赖会被提供给 bean
- 每个属性或者是构造器参数都是一个实际值的定义或者是容器中另一个bean的引用.
- 每个属性或者构造器参数都会从它的格式转换为该属性或构造器参数的实际类型.默认情况下,Spring 可以将一个string格式的值转换为内置类型,比如 `int` , `long` ,  `String` , `boolean` 等等.

Spring 在创建容器的时候会校验每个bean的配置.然而,bean自身的属性直到bean真正被创建的时候才会被创建. 当容器创建的时候被设置为 单例 范围和预加载(默认)的 bean会被创建.  范围定义为可以参考   [Bean Scopes](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes). 除此之外,bean只会在被请求的时候创建.因为创建bean的时候bean的依赖以及bean依赖的依赖都会被创建和指向,所以创建bean的时候会隐式的生成一个bean图.注意这些依赖解析不匹配的情况可能会在后面发生-也就是第一次创建这个受影响的bean的时候.

> ​                                            循环依赖
>
> 如果你主要使用的是构造器依赖,有可能会碰到无法解决的循环依赖的场景.
>
> 比如:类A构造器注入需要一个类B实例,而类B构造器注入需要一个类A实例.如果你将类A和B注入到彼此中,Spring IoC 容器在运行时会检测到这个循环引用,并且抛出一个 `BeanCurrentlyInCreationException`.
>
> 一个可能的解决方案是修改源码将一些使用构造器注入的方式修改为使用 setter 注入.或者,避免使用构造器注入只使用setter注入.换句话说,虽然我们不推荐,但是使用setter注入确实可以避免循环依赖.
>
> 与通常情况不同(没有循环依赖),bean A与bean B发生了循环依赖迫使其中一个bean要在完全初始化之前就注入到另一个bean中(一个经典的鸡和蛋的问题)

通常你可以相信Spring会做出正确的事情.它会在容器加载的时候检测配置的问题,比如引用了不存在的bean或者是循环依赖.当bean被创建的时候,Spring 会尽可能晚的设置属性和解析依赖.这意味着Spring容器加载的时候可能是正确的,但是当你请求一个对象,而这个对象或它的依赖在创建的时候可能会出现异常--比如,bean 在丢失或者出现无效属性的时候会抛出一个异常作为结果. 这可能会导致一些配置异常会被延迟发现,这就是为什么 `ApplicationContext` 实现的默认情况下 bean 是预先实例化的单例模式. 这让你在实际需要使用这些bean之前要花一些前期时间和内存,但是你会在创建 `ApplicationContext` 时发现配置问题,而不是稍后.您仍然可以覆盖此默认行为,以便单例bean延迟初始化,而不是预先实例化.

如果不存在循环依赖关系,则当将一个或多个协同的bean注入到依赖bean中时,每个协同bean在注入到依赖bean中之前都已完全配置好了.这意味着,如果bean A依赖于bean B,则Spring IoC容器会在调用bean A的setter方法之前完全配置好 bean B.换句话说,一个bean被实例化(如果它不是预先实例化的单例),则它的依赖会被准备好,并调用相关的生命周期方法(例如[配置的初始方法](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)或[InitializingBean回调方法](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)).



### 依赖注入示例



