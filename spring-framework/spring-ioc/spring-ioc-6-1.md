# 深入 Spring IoC 系列 - 6.1:  自定义bean: 生命周期



[TOC]



# 自定义bean

Spring 提供了一系列的接口让你可以通过他们自定义 bean



## 生命周期回调

为了和容器管理 bean 的生命周期做交互,你可以实现 Spring 的 `InitializingBean` 和 `DisposableBean` 接口.容器会在初始化完成后调用 `afterPropertiesSet()` 以及销毁前调用 `destroy()` 来让bean 执行某些操作.

> JSR-250 `@PostConstruct` 和 `@PreDestroy` 注解通常被认为是Spring 应用中接受生命回调的最佳实现.使用这些注解意味着你的 bean 没有和 Spring 特定接口绑定.详情见 [此](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations). 
>
> 如果你不想使用 JSR-250 注解但是你仍然想要解耦,考虑使用`init-method` 和`destroy-method` bean 定义

在内部,Spring 使用`BeanPostProcessor` 实现来处理它能找到的回调接口并且调用合适的方法.如果你需要自定义功能或者其他 Spring 默认没有提供的生命周期行为,你可以自己实现一个 `BeanPostProcessor` .详见[容器扩展点](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-extension).

除了初始化和销毁的回调,Spring 管理的对象还实现了`Lifecycle` 接口,所以这些对象可以参加到受容器自身生命周期驱动的启动和关闭流程



### 初始化回调

`org.springframework.beans.factory.InitializingBean` 接口使得 bean 在容器设置了所有bean需要的属性后执行初始化工作.`InitializingBean` 声明了唯一方法:

```java
void afterPropertiesSet() throws Exception;
```

我们推荐你不要使用 `InitializingBean` 接口,因为没有必要将代码和 Spring 耦合.换句话说,我们推荐使用 [`@PostConstruct`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations) 注解或者指定一个 POJO 初始化方法.如果使用的是基于 xml 配置元数据,你可以使用`init-method`属性来指定一个 void 无参方法.如果使用 Java 配置,你可以使用 `@Bean` 的 `initMethod` 属性.见[接受生命周期回调](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-lifecycle-callbacks).观察如下例子:

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

上面的例子的作用和下面例子基本相同

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

然而第一个写法的代码没有和 Spring 耦合

### 销毁回调

实现 `org.springframework.beans.factory.DisposableBean` 接口使得一个bean 在它的容器销毁时获得一个回调.`DisposableBean` 接口声明了单一方法:

```java
void destroy() throws Exception;
```

我们推荐你不要使用 `DisposableBean` 回调接口,因为没有必要把代码和 Spring 耦合.相反,我们推荐你使用 [`@PreDestroy`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations) 注解或者通过 bean 定义来制定一个方法.使用xml 来定义配置元数据时,你可以使用`<bean/>` 的 `destroy-method` 属性.如果使用 Java 配置,你可以使用`@Bean` 的`destroyMethod`属性.见[接受生命周期回调](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-java-lifecycle-callbacks).观察如下例子:

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

上面的例子的作用和下面例子基本相同

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

然而第一个写法的代码没有和 Spring 耦合

> 你可以指定 `<bean>` 元素的`destroy-method` 属性为一个特殊的 `(inferred)` 值,这会命令 Spring 自动检测指定 bean 类的公开的 `close` 或者`shutdown` 方法.(任何实现了`java.lang.AutoCloseable` 或者`java.io.Closeable` 的类都会匹配到)你还可以设置这个特殊的`(inferred)` 值到`default-destroy-method`属性上使得所有 bean 都会应用这种行为(见[默认初始化和销毁方法](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-default-init-destroy-methods)).记得这事 Java 配置的默认行为



### 默认初始化和销毁方法

当你没有使用 Spring 指定的`InitializingBean` 和 `DisposableBean` 编写初始化和销毁方法回调,你通常会使用名称如  `init()`, `initialize()`, `dispose()`等.理想情况下,这些生命周期回调方法的命名应该是标准化的,这样所有的开发者使用相同的方法名并保持一致性.

**你可以配置 Spring 容器在每个 bean 上寻找相同命名的初始化和销毁回调.这意味着你可以指定一个 `init()` 方法作为初始化回调而不需要配置 `init-method="init"` 属性到每个 bean 上.Spring 容器在 bean 创建好的时候调用这个方法.这么做可以保持项目的生命周期回调方法命名风格**

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

你可以使用如下 bean 定义

```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

`<beans/>` 中的 `default-init-method` 属性使得 Spring IoC 容器使用方法 `init` 作为初始化回调.当一个 bean 被创建和装配的时候,如果 bean 类有这样的一个方法,它会在合适的时候被调用.

你可以用类似的方法使用属性`default-destroy-method` 配置销毁方法.

当已有的 bean 类已经拥有了自己的回调方法名的话,你可以重写它自身的 `init-method` 和 `destroy-method` 来覆盖默认声明.

**Spring 容器会保证在 bean 依赖装配完毕后立刻调用配置的初始化回调.因此,初始化回调是在原始 bean 引用上进行的,意味着 AOP 拦截等还没应用到 bean 上.目标 bean 首先会被完全创建然后才应用如 AOP 代理等拦截器.如果 bean 和代理是独立定义的,你的代码甚至可以和原始 bean 交互,而不用经过代理.因此,如果你在 `init` 方法上使用拦截器的话会出现歧义,因为这么做会耦合bean 的生命周期和它的代理或者拦截器,并且在你的代码和原始 bean 交互的时候出现奇怪的现象**













