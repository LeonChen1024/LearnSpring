# 深入 Spring IoC 系列 - 6.2:  自定义bean: 生命周期



[TOC]



### 组合生命周期机制

在 Spring2.5 中,你有 3 种控制 bean 生命周期行为的方式:

-  [`InitializingBean`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) 和  [`DisposableBean`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean)  回调接口
- 自定义 `init()` 和 `destroy()` 方法
-  [`@PostConstruct` 和 `@PreDestroy` 注解](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations).

你可以组合这些机制来控制指定 bean.

> 如果一个bean 配置了多个生命周期函数并且都拥有不同的名字,那么他们将按下面的顺序执行.然而,如果配置了相同的方法名,那么这个方法只会执行一次

生命周期配置执行顺序如下:

1. 使用了 `@PostConstruct` 的方法
2. `InitializingBean` 回调接口定义的`afterPropertiesSet()` 方法
3. 自定义的 `init()` 方法

销毁方法调用顺序:

1. 使用了 `@PreDestroy` 注解的方法
2. `DisposableBean` 回调定义的`destroy()` 方法
3. 自定义的 `destroy()` 方法



### 启动和关闭回调

`Lifecycle` 接口定义了一些基础方法用于在自身生命周期做一些处理(比如启动或者停止一些后台进程)

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何 Spring 管理的对象都可以实现这个接口.然后,当 `ApplicationContext` 接收到启动和停止信号(比如运行时的关闭/重启),它会调用所有这个上下文中 `Lifecycle` 的实现.他通过代理如下 `LifecycleProcessor` 来实现:

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

注意 `LifecycleProcessor` 是 `Lifecycle`接口的继承.它还添加了两个方法来响应上下文的刷新和关闭

> 注意`org.springframework.context.Lifecycle` 接口只是启动和停止的协议并且没有自动启动的功能.如果想要针对 bean自动启动处理,可以考虑实现`org.springframework.context.SmartLifecycle` .
>
> 同样,停止通知并不保证会在销毁前到达.常规的停止时,所有的`Lifecycle` bean 首先会在销毁回调前收到一个停止通知.然而在热更新的情况下或者是终止刷新的时候,只有销毁方法会被调用

启动和关闭调用的顺序是非常重要的.如果两个对象之间存在依赖关系,被依赖项会先启动,但是会晚于依赖方销毁.然而,有时候并没有直接的依赖关系.你只是知道一个类型的对象应该早于另一个类型的对象.这个时候可以考虑`SmartLifecycle` 接口定义的另一个可选方式,就是他的父接口 `Phase` 定义的 `getPhase()` 方法.如下:

```java
public interface Phased {

    int getPhase();
}
```



```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

**启动的时候,拥有更低的 phase 的对象优先启动.当停止的时候,更高的 phase 对象优先关闭.因此,实现了 `SmartLifecycle` 并且`getPhase()` 返回`Integer.MIN_VALUE` 的对象将会最先启动最后关闭.使用`Integer.MAX_VALUE` 则相反.要知道没有实现 `SmartLifecycle` 的 `Lifecycle` 对象的 phase 是 `0` .因此,所有负值的对象会在普通对象之前启动,在他们之后销毁.正值则相反.**

`SmartLifecycle` 定义的停止方法接收一个回调.所有的实现都会在关闭处理完毕的时候调用回调的 `run()`方法.这就可以用来处理异步关闭, 由于默认的 `LifecycleProcessor` 实现 `DefaultLifecycleProcessor` ,它的每个phase回调都有一个超时设置,每个 phase 的超时时间是 30s.你可以通过定义一个 bean 的名字为 `lifecycleProcessor` 来重写设置.如果你只想修改超时时间,下面的设置就足够了:

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

如上所说,`LifecycleProcessor` 接口定义了 context 刷新和关闭的回调.在 context 关闭的时候如同显式调用 `stop()` 方法一样.另一方面,'refresh'回调允许`SmartLifecycle` 的另一个特性.当 context 是刷新的时候(所有对象被实例化并初始化),会调用这个回调. **这个时候默认的生命流程处理器会校验每个 `SmartLifecycle` 对象的 `isAutoStartup()` 方法的Boolean 返回值.如果是 `true` ,对象会在这时启动而不是等待 context 显示调用它自身的 `start()` 方法(标准的 context 实现不会自动调用 start). `phase` 的值和依赖关系决定了启动的顺序.** 



### 在非 web 应用中优雅的关闭 Spring IoC 容器

> 只针对非 web 应用,`ApplicationContext` 实现已经可以在关联 web 应用关闭的时候关闭 IoC 容器

在非 web 应用(如桌面环境)注册一个JVM 的关闭钩子.要确保优雅的关闭并且调用单例 bean 中销毁方法以保证所有的资源被释放.你必须正确的配置和实现这些销毁回调

为了注册一个关闭钩子,调用`ConfigurableApplicationContext` 接口的 `registerShutdownHook()` 方法,如下:

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```















