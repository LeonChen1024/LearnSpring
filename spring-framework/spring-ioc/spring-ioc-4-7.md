

# 深入 Spring IoC 系列 - 4.7: 依赖-方法注入



[TOC]

## 方法注入

在大多数应用场景,大多数容器中的bean时单例的.当一个单例bean需要和另一个单例bean合作或者一个非单例bean需要和另一个非单例bean合作,通常你需要通过将一个bean定义为另一个bean 的属性.当bean的声明周期不一样的时候会出现问题.假设一个单例bean A需要使用非单例 bean B,可能在 A的每个方法调用上.可是容器只会创建A一次,因此只有一次机会来设置属性. 容器无法在每次需要的时候为bean A 提供一个新的B实例.

一个方案时放弃一些控制反转.你可以通过实现 `ApplicationContextAware` 接口[让 bean A 感知到容器](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-aware),并且通过[一个 `getBean("B")` 方法在每次需要的时候或取](https://github.com/LeonChen1024/LearnSpring/blob/master/spring-framework/spring-ioc/spring-ioc-2.md#%E4%BD%BF%E7%94%A8%E5%AE%B9%E5%99%A8)一个全新的 bean B.如下:

```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

这种方式不太可取,这样使得业务代码感知并耦合了 Spring 框架.方法注入,一个Spring IoC 容器的高级功能可以帮助你干净的解决这个问题.

> 你可以在[这个博客](https://spring.io/blog/2004/08/06/method-injection/) 了解到方法注入的更多动机



### 查找方法注入

查找方法注入是容器的一个能力,它可以重写容器管理的bean中的方法并且返回查找结果给容器中另一个命名bean. 查找通常涉及到一个原型bean,如上一节描述的那样. Spring 框架通过使用CGLIB来动态生成子类字节码并重写这个方法.

> - 为了让这个动态子类生效,Spring bean 容器要继承的类不能是 `final` 并且要重写的方法也不能是 `final`
> - 对包含 `abstract` 方法的类进行单元测试要求你自己子类化这个类并且提供 `abstract` 方法的实现
> - 组件扫描的时候同样需要具体方法,需要具体的类
> - 一个进一步的限制是差找方法不能在工厂方法的情况下生效,特别是带有 `@Bean` 方法的配置类,因此,在这种情况下,容器不负责创建实例因此无法运行时生成子类

对于前面的 `CommandManager` 类代码中,Spring 容器动态重写了 `createCommand()` 方法的实现. `CommandManager` 类没有任何的 Spring 依赖,如下:

```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

在包含要注入方法的客户端类（在本例中为CommandManager）中, 被注入的方法需要以下格式的签名:

```xml
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法是 `abstract` 的,动态生成的子类实现这个方法.否则,动态生成的子类会重写原始类中定义的具体方法.思考如下代码:

```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

bean  `commandManager` 在每次需要一个新的 `myCommand` bean 实例的时候都会调用它的 `createCommand()` 方法. 要注意 如果需要的话要将 `myCommand` bean 定义为 prototype. 如果它是 singleton,那么 `myCommand` 每次都会返回相同的实例.

或者,在基于注解的组件模型中, 你可以通过 `@Lookup` 定义一个查找方法,如下:

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

或者,更符合惯例的,你可以通过返回类型来声明查找方法:

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```

注意通常你应该在具体实现类的方法上使用 lookup 注解,使得他们和 Spring 的组件扫描规则匹配,抽象类默认会被忽略.这个限制不会作用在显式注册和导入的bean类中.

> 另一个获取不同作用域的bean的方式是 `ObjectFactory` / `Provider` 注入.参考 [通过依赖设置bean的作用域](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection) .
>
> 你可能也会发现 `ServiceLocatorFactoryBean` (在 `org.springframework.beans.factory.config`  )是很有用的



### 任意方法替换

与查找方法注入相比,方法注入的一种不太有用的形式是能够用另一种方法替换bean中的任意方法.你可以跳过这个章节直到你确实需要这个功能的时候.

这部分略.

















