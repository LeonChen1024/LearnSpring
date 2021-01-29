

# 深入 Spring IoC 系列 - 5.1: bean 作用域 单例和原型



[TOC]

当你创建一个bean定义的时候,你也就创建了生成这个类定义实例的步骤清单.bean定义是一个步骤清单的概念是狠重要的,因为它意味着对于一个类,你可以通过这个清单创建出很多个实例

在bean定义中你不仅可以控制创建对象时注入的依赖和配置值还可以控制生成对象的作用域.这种方式是强大而自由的,因为你可以选择在配置中定义作用域而不是在 Java 类级别来定义.Spring提供了6个作用域范围,其中四个仅在使用 web 的 `ApplicationContext` 的时候可以用.你还可以创建一个 [自定义作用域](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-custom)

支持的作用域如下:

| Scope                                                        | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [singleton](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-singleton) | (默认) 一个bean定义在每个Spring IoC 容器中都只对应了一个对象实例 |
| [prototype](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-prototype) | 将单个bean定义的作用域限定为任意数量的对象实例               |
| [request](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-request) | 将bean定义的作用域限定为单次 HTTP 请求.只在 Spring 的`ApplicationContext` 上下文中生效 |
| [session](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-session) | 将bean定义的作用域限定为单个 HTTP  `Session` 的生命周期. 只在 Spring 的`ApplicationContext` 上下文中生效 |
| [application](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-application) | 将bean定义的作用域限定为单个 `ServletContext` 的生命周期. 只在 Spring 的`ApplicationContext` 上下文中生效 |
| [websocket](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/web.html#websocket-stomp-websocket-scope) | 将bean定义的作用域限定为单个 `WebSocket` 的生命周期. 只在 Spring 的`ApplicationContext` 上下文中生效 |



## Singleton 作用域

容器中只管理了一个这个bean的共享实例,所有请求该bean的时候容器都会返回同一个指定bean实例.

换一句话说,当你定义了一个作用域为单例的bean,Spring IoC容器只会创建一个该bean定义的实例.这个单一实例存储在这类bean的缓存中,所有后续的请求和引用都会返回这个缓存对象.如下:

![singleton](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/images/singleton.png)

Spring 中的 单例不同于 GoF 书中的单例模式. GoF 的单例对一个对象的单例进行了硬编码使得每个类加载器中有且只有生成一个指定类的实例. Spring 的单例作用域最好是描述为每个容器1个bean.这意味着,如果你在一个容器中对指定类定义了一个bean,Spring容器只会创建一个该bean定义的实例.单例作用域是Spring的默认项.在XML 中定义bean如下:

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```



## Prototype 作用域

使用 原型作用域会导致每次请求该指定bean的时候都会生成一个新的bean实例. 通常你应该对所有有状态bean使用原型作用域而对无状态bean 使用 单例作用域.

下图解释了Spring 原型作用域:



![prototype](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/images/prototype.png)

(和图示不同的是通常情况下一个数据访问对象(DAO)是不会配置为原型的,因为DAO通常不会持有状态.应该使用单例更加合适)

如下是XML中的配置:

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

与其它作用域不通的是,Spring 没有管理原型bean完整的生命流程.容器 实例化,配置,组装一个原型对象并且将它交给客户端,之后就没有再跟踪这个实例了.因此,尽管生命周期初始化调用方法会被所有作用域的对象调用,但是原型实例的销毁回调不会被调用. 客户端代码必须自己清理原型对象和它所持有的资源.想要让Spring容器来释放被原型bean持有的资源,可以使用自定义  [bean post-processor ](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-extension-bpp) ,它会持有需要被清理的bean的引用.

在某些方面,Spring 容器中的 原型bean 可以看做是 Java `new` 操作的一个替代方案. 所有在这个阶段之后的生命周期管理都必须由客户端自己处理.(更多关于 bean 的生命周期可以见 [Lifecycle Callbacks](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle).)



## Singleton Bean 带有 Prototype-bean 依赖的情况

当你使用单例作用域的bean 依赖了 原型bean ,要记得依赖在初始化的时候解析.因此,如果你依赖注入了一个原型bean到一个单例bean中,一个新的原型bean被实例然后注入到单例bean中.原型实例是唯一一个提供给单例bean的实例.

然而,如果你想要单例bean在运行时获得新的原型bean实例.你不能依赖注入一个原型bean到单例bean中,因为这个注入只有一次,发生在Spring 容器实例化单例bean并且解析注入依赖的时候.如果你需要在运行时不仅一次获取新的原型bean实例,详见[方法注入](https://github.com/LeonChen1024/LearnSpring/blob/master/spring-framework/spring-ioc/spring-ioc-4-7.md#%E6%B7%B1%E5%85%A5-spring-ioc-%E7%B3%BB%E5%88%97---47-%E4%BE%9D%E8%B5%96-%E6%96%B9%E6%B3%95%E6%B3%A8%E5%85%A5) 







