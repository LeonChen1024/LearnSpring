

# 深入 Spring IoC 系列 - 4.2: 依赖 - 依赖注入解析及示例



[TOC]



## 依赖注入

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

下面的例子使用了基于 XML 配置元数据来实现基于 setter 的DI, 一个Spring XML 配置文件中的一部分如下:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

下面的例子展示了相关联的 `ExampleBean` 类:

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的例子中,setter 会和 XML 中指定的属性进行匹配.

下面的例子使用基于构造器的DI:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

下面展示了对应的 `ExampleBean` 类:

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

bean 定义中指定的 `constructor-arg` 在 `ExampleBean` 中作为构造器参数使用.

现在思考一下这个例子的变形,告诉Spring 使用 `static` 工厂方法来返回一个对象的实例:

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

下面展示的是关联的 `ExampleBean` 类:

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

`static` 工厂方法的参数使用的是 `<constructor-arg/>` 元素,和使用构造器的情况是一样的.工厂方法返回的类的类型不一定要是包含这个 `static` 工厂方法的类型(尽管这个例子是这样的).实例(非静态) 工厂方法使用的方式基本是一致的(除了使用 `factory-bean` 属性替代 `class` 属性),所以我们在这里不讨论这些细节.

































































