# 深入 Spring IoC 系列 - 6.3:   Aware 相关回调



[TOC]



## `ApplicationContextAware` and `BeanNameAware`



当一个`ApplicationContext` 创建一个实现了 `org.springframework.context.ApplicationContextAware` 接口的 bean,这个实例就提供了一个`ApplicationContext` 的引用.`ApplicationContextAware` 如下

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

 因此,bean 可以编码操作创建了他们的`ApplicationContext` ,通过 `ApplicationContext` 接口或者是将引用转型成一个这个接口的子类(比如 `ConfigurableApplicationContext` ,它提供了额外的函数).一种用途是编程式的索引其他的 bean.有时候这个功能很有用.但是通常情况下你应该避免这么使用它,因为他会将你的代码和 Spring 耦合并且没有遵循依赖倒置,并没有将协作者当做属性提供.还有 `ApplicationContext` 访问文件资源,发布应用事件,访问`MessageSource` 等详见 [`ApplicationContext` 的额外功能](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#context-introduction)

自动装配是获取 `ApplicationContext` 的另一种方式.传统的`constructor` 和`byType` 模式 (详见[自动装配协作](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-autowire)) 可以提供一个`ApplicationContext` 的依赖.为了更加的灵活,推荐使用注解`@Autowired` 来实现,详见[使用`@Autowired`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation)

当一个`ApplicationContext` 创建了一个类实现了`org.springframework.beans.factory.BeanNameAware` 接口,这个类就提供了一个关联到对象定义名字的回调.

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

这回调会在填充了 bean 属性之后但是在如`InitializingBean` ,`afterPropertiesSet` 或者自定义初始方法之前.



## 其他`Aware` 接口

除了上面说的那些,Spring 还提供了很多 `Aware` 回调接口来让 bean 表明他们需要获取一些基础依赖.通常情况下,名字表明了这个依赖类型.如下总结了一些重要`Aware` 接口:

| Name                             | Injected Dependency                                          | Explained in…                                                |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `ApplicationContextAware`        | 声明 `ApplicationContext`.                                   | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware) |
| `ApplicationEventPublisherAware` | `ApplicationContext` 内的事件发布.                           | [Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#context-introduction) |
| `BeanClassLoaderAware`           | 加载 bean 类的类加载器                                       | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-class) |
| `BeanFactoryAware`               | 声明 `BeanFactory`.                                          | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware) |
| `BeanNameAware`                  | 声明 bean的名字.                                             | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware) |
| `BootstrapContextAware`          | `BootstrapContext` 的资源适配器. 通常只在 JCA-aware `ApplicationContext` 实例有效. | [JCA CCI](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/integration.html#cci) |
| `LoadTimeWeaverAware`            | 定义在加载时处理类定义的 weaver .                            | [Load-time Weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#aop-aj-ltw) |
| `MessageSourceAware`             | 配置处理消息的策略 (支持参数化和国际化).                     | [Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#context-introduction) |
| `NotificationPublisherAware`     | Spring JMX 通知发布者.                                       | [Notifications](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/integration.html#jmx-notifications) |
| `ResourceLoaderAware`            | 配置底层资源的加载器.                                        | [Resources](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#resources) |
| `ServletConfigAware`             | 当前容器运行的 `ServletConfig` . 只在 web 的 Spring `ApplicationContext` 有效. | [Spring MVC](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/web.html#mvc) |
| `ServletContextAware`            | 当前容器运行的 `ServletContext` . 只在 web 的 Spring `ApplicationContext` 有效. | [Spring MVC](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/web.html#mvc) |

再次提醒使用这些接口会将代码和 Spring API 耦合并且没有遵循控制反转.所以,我们推荐在需要编程式访问容器的基础设施 bean 中使用.



































































