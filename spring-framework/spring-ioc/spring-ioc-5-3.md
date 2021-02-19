

# 深入 Spring IoC 系列 - 5.3:  限定作用域



[TOC]



### 限定作用域的 bean 作为依赖

Spring IOC 容器管理的不仅仅是对象的实例化,同时还有装载协作者(或者依赖)的作用.如果你想要注入一个 HTTP 请求作用域的 bean 到另一个生命周期更长 bean 中,你可以选择注入一个 AOP 代理这个 bean.也就是说,你需要注入一个代理对象暴露出和短生命周期作用域 bean相同的public 接口但是它同时还能接收一个相关作用域的真实目标对象并且代理调用方法到真实对象.

> 你还可以在作用域是`singleton` 的 bean 使用 `<aop:scoped-proxy/>` ,带有这个引用将会使用中间代理来序列化而且可以在反序列化的时候重新获取.
>
> 当定义一个`<aop:scoped-proxy/>` 给一个作用域是 `prototype` 的 bean,每次调用代理的方法将会导致创建一个新的实例来调用这个方法.
>
> 同时,作用域代理不是唯一的方法可以以生命周期安全的方式来访问较短生命周期的 bean.你还可以定义你的注入点`ObjectFactory<MyTargetBean>` (也就是使用构造器或者是 setter 注入或者是字段注入),它有一个 `getObject()` 调用来在每次需要的时候获取当前实例 -- 而不用单独持有或者保存这个实例.
>
> 作为一个扩展辩题,你可以定义`ObjectProvider<MyTargetBean>` 使用几种额外的访问变体,包括 `getIfAvailable` 还有`getIfUnique` 
>
> JSR-330 变体是叫做 `Provider` 可以通过 `Provider<MyTargetBean>` 定义来使用它,这会在每次访问的时候调用对应的 `get()` 方法.详见[这里](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-standard-annotations)

下面的配置示例只有一行,但是了解"为什么"和"怎么做"也是同样重要的:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> <!-- 这一行定义了代理 -->
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

为了创建一个代理,你需要插入一个 `<aop:scoped-proxy/>` 到 bean 定义中(详见[选择创建的代理类型](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection-proxies) 和 [基于 xml 配置](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#xsd-schemas)). 为什么作用域是request`, `session 和自定义作用域的 bean 需要使用 `<aop:scoped-proxy/>` 元素呢? 思考下面的单例 bean 和你对作用域的需求(注意下面的`userPreferences` 定义是不完整的)

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

如上,单例 bean `userManager` 注入了一个 HTTP `Session` 作用域的 bean (`userPreferences`) .重要的是单例 `userManager` 在每个容器中只会实例化一次,而他的依赖(`userPreferences`)也只会注入一次.这意味着`userManager` bean 操作的一直是同一个 `userPreferences` 对象(最早注入的那个对象).

这不是你想要的行为,通常我们注入一个短生命 bean 到一个长生命 bean 的时候,我们是想要一个 `userManager` 对象,在每个 `Session` 作用域内使用这个 `Session`指定的 `userPreferences` . 所以我们需要的是容器创建一个对象暴露出和 `UserPreferences` 一样的公开接口(理想情况下这个对象是一个 `UserPreferences` 实例),它可以从作用域中获取到真实的`UserPreferences` 对象.在这个例子中,当`UserManager` 实例调用一个依赖注入的 `UserPreferences` 对象的方法时,它实际会调用代理的方法.代理会从 HTTP `Seesion ` 作用域中获取真实的 `UserPreferences` 对象.

因此,你应该使用如下的注入配置 

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

**选择创建的代理类型**

默认情况下 Spring 容器队标记了 `<aop:scoped-proxy/>` 的元素创建一个bean 代理的时候,会创建一个基于CGLIB类的代理.

> CGLIB代理只会拦截 public 调用,而不会代理非 public 方法.

或者,你可以配置使得 Spring 容器生成一个标准的基于 JDK接口 的代理 bean,只需要指定 `<aop:scoped-proxy/>` 元素的 `proxy-target-class` 属性的值为 `false` 即可.这意味着你不需要额外的库来提供代理.然而,这同样意味着 bean 必须实现至少一个接口,并且所有的合作bean 必须通过他的接口来引用他.如下:

```xml
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

更多内容见[代理机制](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#aop-proxying)













































