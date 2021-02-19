

# 深入 Spring IoC 系列 - 5.2: Request, Session, Application, 和 WebSocket 作用域



[TOC]

## Request, Session, Application, 和 WebSocket 作用域

`request` , `session` , `application` 还有 `websocket` 作用域只在你使用了 web 的Spring `ApplicationContext` 实现(比如 `XmlWebApplicationContext` )的时候可用.如果你在正常的Spring IoC 容器中使用了这些作用域,比如 `ClassPathXmlApplicationContext` ,那么会抛出一个未知bean异常 `IllegalStateException`.

### 初始化web配置

为了支持`request` , `session` , `application` 还有 `websocket` 这些等级(web 作用域)的bean作用域,在定义你的bean之前需要一些最小初始化配置.(这些初始化设置在使用标准作用域:`singleton` 和  `prototype` 的时候是非必需的)

如何初始化这个设置取决于你的 Servlet 环境.

如果你在 Spring Web MVC 中访问bean 作用域,实际上,请求被 Spring 的 `DispatcherServlet` 处理,这时候不需要特殊的设置. `DispatcherServlet` 已经暴露了所有相关的状态.

如果你使用的是 Servlet 2.5 web 容器, 请求是在 Spring 的 `DispatcherServlet` 外处理的(比如,当使用 JSF 或者 Struts ),你需要注册 `org.springframework.web.context.request.RequestContextListener`  `ServletRequestListener` .对于 Servlet 3.0+, 这个可以通过使用 `WebApplicationInitializer` 编程处理. 或者,如果使用更老的容器,可以添加一下代码到 web 应用的 `web.xml` 文件:

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

或者,如果你设置监听器出现了问题,可以考虑使用 Spring 的 `RequestContextFilter` .这个过滤器映射取决于所处的web应用配置,所以你必须将其设置正确.下面列出了web应用 过滤器 部分的内容:

```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

`DispatcherServlet` , `RequestContextListener` , 还有 `RequestContextFilter` 所做的事情都是一样的, 即将HTTP请求对象绑定到为该请求提供服务的线程. 这使得对应 请求 和 session 作用域的bean在调用链中可以使用

### 请求作用域

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

Spring 容器在每个HTTP 请求的时候会使用 `loginAction` bean定义生成一个新的 `LoginAction` 实例. 也就是说这个bean的作用域是在请求级别上的.你可以随意改变这个实例的状态,并不会影响到其它的请求级别生成的实例.每个请求的bean实例都是独立的.当请求执行结束的时候,这个bean就被丢弃了.

使用注释驱动的组件或Java配置时，可以使用 `@RequestScope` 注解在一个组件上来指定这个 `request` 作用域.如下:

```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```



### Session 作用域

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

```

Spring 容器在每个 HTTP `Session` 生命中使用 `UserPreferences` bean 定义来生成一个 `UserPreferences` bean 实例. 也就是说这个`userPreferences` bean 作用域是 `Session` 级别的.和请求作用域的bean类似,你也可以随意更改bean状态,注意这个修改的可见范围是同一个Session的情况下,其它的 Session 中生成其它实例对这些修改是不可见的,因为他们在每个 HTTP `Session` 中都是独立的.当HTTP `Session` 被丢弃的时候,这个bean实例也同样被丢弃了

使用注释驱动的组件或Java配置时，可以使用 `@SessionScope`  注解在一个组件上来指定这个 `session` 作用域,如下:

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

### 应用作用域

思考如下 xml bean 定义

```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

Spring 容器为整个 web 应用通过该定义创建一个 `AppPreferences` 的实例.也就是说这个`appPreferences` bean 的作用域是整个 `ServletContext` 级别的并且作为一个 `ServletContext` 属性保存.这和单例 bean 类似,但是有两个重要的不同点:它是每个 `ServletContext` 的单例,而不是每个 Spring '应用 Context'(因此在 web 应用中可能会有多个),它实际上是公开的就像 `ServletContext` 的属性一样.

当使用注解驱动的组件或者 Java 配置时,你可以使用 `@ApplicationScope` 注解来将一个组件设置为 `application` 作用域.如下:

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```



### 限定作用域的 bean 作为依赖

Spring IOC 容器管理的不仅仅是对象的实例化,同时还有装载协作者(或者依赖)的作用.如果你想要注入一个 HTTP 请求作用域的 bean 到另一个生命周期更长 bean 中,你可以选择注入一个 AOP 代理这个作用域 bean.也就是说,你需要注入一个代理对象暴露出和作用域 bean相同的public 接口但是它同时还能接收一个相关作用域的真实目标对象并且代理调用方法到真实对象.



















