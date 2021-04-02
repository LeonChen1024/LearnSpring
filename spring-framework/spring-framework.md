# 深入 Spring Framework 

[TOC]

# 背景

Spring 可以说是在 Java 后台开发中占据了统治地位,而它的核心就是 Spring Framework , Spring Framework 也是一个及其庞大的框架,但是由于他有很多优秀的思想所以值的我们深入学习,出于求知的心理,打算对它进行深入学习.由于它的庞大,所以决定先从核心的部分开始.



# 核心技术



## IoC容器

也就是负责管理控制反转的容器机制.







# 节选知识



## IOC 6.1

你可以配置 Spring 容器在每个 bean 上寻找相同命名的初始化和销毁回调.这意味着你可以指定一个 `init()` 方法作为初始化回调而不需要配置 `init-method="init"` 属性到每个 bean 上.Spring 容器在 bean 创建好的时候调用这个方法.这么做可以保持项目的生命周期回调方法命名风格.使用方式为在xml `beans ` 元素中添加 `default-init-method`

```xml
<beans default-init-method="init">
```

如何 bean 中单独指定了回调方法,以该方法为主.



Spring 容器会保证在 bean 依赖装配完毕后立刻调用配置的初始化回调.因此,初始化回调是在原始 bean 引用上进行的,意味着 AOP 拦截等还没应用到 bean 上.目标 bean 首先会被完全创建然后才应用如 AOP 代理等拦截器.如果 bean 和代理是独立定义的,你的代码甚至可以和原始 bean 交互,而不用经过代理.因此,如果你在 `init` 方法上使用拦截器的话会出现歧义,因为这么做会耦合bean 的生命周期和它的代理或者拦截器,并且在你的代码和原始 bean 交互的时候出现奇怪的现象



## IOC 6.2

注意`org.springframework.context.Lifecycle` 接口只是启动和停止的协议并且没有自动启动的功能.如果想要针对 bean自动启动处理,可以考虑实现`org.springframework.context.SmartLifecycle` .

同样,停止通知并不保证会在销毁前到达.常规的停止时,所有的`Lifecycle` bean 首先会在销毁回调前收到一个停止通知.然而在热更新的情况下或者是终止刷新的时候,只有销毁方法会被调用

启动的时候,拥有更低的 phase 的对象优先启动.当停止的时候,更高的 phase 对象优先关闭.因此,实现了 `SmartLifecycle` 并且`getPhase()` 返回`Integer.MIN_VALUE` 的对象将会最先启动最后关闭.使用`Integer.MAX_VALUE` 则相反.要知道没有实现 `SmartLifecycle` 的 `Lifecycle` 对象的 phase 是 `0` .因此,所有负值的对象会在普通对象之前启动,在他们之后销毁.正值则相反.

















