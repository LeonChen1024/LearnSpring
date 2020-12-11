

# 深入 Spring IoC 系列 - 4.6: 依赖-使用 depends-on,懒初始化,自动装配协同者



[TOC]



## 使用 depends-on

如果一个bean是另一个bean的依赖,通常来说意味着一个bean被设置为另一个bean的属性.通常可以使用 `<ref/>` 元素来配置元数据.然而,有时候bean之间的依赖没那么直接.比如当一个类中的静态初始化器需要被触发的时候,如数据库驱动注册. `depends-on` 属性可以显式的强制一个或者更多的bean在这个元素初始化之前初始化.下面的例子展示了如果使用这个属性初始化单个bean:

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

为了表达一个依赖了多个bean的情况,可以为 `depends-on` 属性赋上一个bean名称列表作为值(可以使用逗号,空格,分号作为分割符):

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

>  `depends-on` 属性可以指定依赖的初始化时间顺序,在单例bean的情况下还能影响依赖的销毁时间顺序.被依赖的bean首先被销毁,然后才是依赖他们的bean.因此, `depends-on` 可以控制关闭的顺序.



## 懒初始化bean

默认情况下, `ApplicationContext` 实现在初始化阶段就会饿汉式的直接创建和配置所有的单例bean.一般来说这个预实例化行为是可取的,因为这样配置和环境的问题会立即被发现,而不是等到几个小时甚至是几天后.如果你不想要这种方式,可以通过在bean定义上标记为懒初始化实现.一个懒初始化的bean告诉IoC容器在它首次被请求的时候初始化它,而不是在一启动的时候就初始化它.

在XML中,这个行为是被 `<bean/>` 元素中的`lazy-init` 属性控制的,如下:

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

当这个配置被 `ApplicationContext` 消费的时候, 声明了懒初始化的bean不会在`ApplicationContext` 启动的时候被预实例化,而其它非懒初始化的bean会被立刻预实例化.

然而,当一个懒初始化的bean是另一个非懒初始化的单例bean的依赖时, `ApplicationContext` 会在启动时就创建这个懒初始化bean,因为它必须满足单例的依赖规则.这个懒初始化bean会被注入到那个单例bean中.

你还可以在容器层面控制懒初始化,使用 `<beans/>` 的 `default-lazy-init` 属性.如下:

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```



## 自动装配协同者

Spring 容器可以自动装配协同bean之间的关联.你可以让Spring 通过在 `ApplicationContext` 中检查内容解决协同者(其它bean).自动装配有以下好处:

- 自动装配可以显著减少指定属性或者构造器参数的需要.(其它机制如本章中介绍的 [bean 模板](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-child-bean-definitions)也是很有用的)
- 自动装配可以随着你的对象演化而更新.比如,如果你需要添加一个依赖到类中,这个依赖可以在你不需要修改配置的情况下自动匹配.因此自动装配在开发的时候是非常有用的,当代码基础变得更加稳定的时候,也不需要切换到显式的装配.

当使用XML配置元数据时(参考[依赖注入](https://github.com/LeonChen1024/LearnSpring/blob/master/spring-framework/spring-ioc/spring-ioc-4-1.md)),你可以通过定义 `<bean/>` 元素的 `autowire` 属性指定自动装配模式,自动装配模式有 4 种方式.你可以对每个bean指定一种.模式内容如下:

| 模式        | 解释                                                         |
| ----------- | ------------------------------------------------------------ |
| no          | (默认选项)不自动装配.Bean引用必须使用 `ref` 元素指定.在大型部署上不建议修改这个默认项,因为显式指定协同者会更加的可控和清晰.在某种程度上,他记录了系统的结构 |
| byName      | 通过属性名自动装配.Spring通过属性名字来寻找自动装配的bean.比如,如果一个bean 包含一个`master` 属性(也就是拥有一个 `setMaster(..)` 方法),Spring 会查找一个bean定义名为 master 的bean并设置到这个属性中. |
| byType      | 通过属性类型来自动装配.如果容器中存在多个这个类型的bean,会抛出一个 致命 异常,会指出你不能使用 `byType` 自动装配.如果没有匹配到bean,一切正常(属性不会被设置) |
| constructor | 和 byType 类似但是是通过构造器参数匹配.如果容器中没有匹配这个构造参数的bean,会抛出一个致命异常 |

使用 `byType` 或者 `constructor` 自动装配模式,你可以装配数组和类型集合.在这种情况下，将提供容器中与预期类型匹配的所有自动装配候选bean.你可以自动装配强类型 `Map` 实例接收类型 `String`. 这个自动装配的 `Map` 实例将会包含所有匹配的bean实例,并且 `Map` 实例的key 包含了对应bean的名字.

### 自动装配的局限和缺点

最好是在一个项目中始终使用自动装配.如果通常没有使用自动装配,那么有一两个地方使用可能会使开发者产生一些疑惑.

思考以下局限和缺点:

- `property` 和 `constructor-arg` 设置的显式依赖总是会覆写自动装配.你不能自动装配简单属性比如 原始属性,`Strings` 和 `Classes` (还有这些属性组成的数组).这个限制是设计的时候产生的
- 自动装配不如显式装配精确.尽管Spring已经小心的避免不明确的使用导致的预期之外的结果.但是Spring管理的对象是没有显式的精确指定.
- 装配信息可能不适用于那些从Spring 容器中生成文档的工具
- 自动装配可能导致容器内的多个指定类型的bean会被匹配到.对于数组集合等实例这不是一个问题.然而对于依赖单个bean的对象来说问题没有得到解决.将会抛出一个异常

对于这些情况,你有以下选择:

- 禁用自动装配,使用精确装配
- 使用下节中介绍的避免对bean定义中设置了 `autowire-candidate` 属性为 `false` 的对象自动装配
- 通过设置 `<bean/>` 元素的 `primary` 属性为 `true` 来指定一个bean为主要的候选人
- 通过基于注释的配置实施更细粒度的控制，如[基于注解的容器配置](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-annotation-config)中描述的那样.

### 从自动装配中排除某个bean

你可以以每个bean为粒度来排除他们的自动装配选项.在XML格式中,设置`<bean/>` 元素的 `autowire-candidate` 属性为`false` 即可.容器会使得指定bean不支持自动装配(即使使用注解如`@Autowired`)

> `autowire-candidate` 属性只会影响基于类型的自动装配.不会影响显式指定名字的方式,也就是说通过名字自动装配只要名字匹配就会装配成功

你还可以通过bean名称的模式匹配来限制自动装配候选.`<beans/>` 元素的 `default-autowire-candidates` 属性接收一个或多个正则.比如,限制自动装配候选为任何名字以`Repository` 结束的bean,可以提供一个值 `*Repository` .如果要定义多个模式,使用`,` 分割他们. bean 定义的 `autowire-candidate` 属性总是有比模式匹配更高的优先级.

这些技术在你想让一些bean永远不要自动装配的时候很有用.这并不意味着这些bean不能自动装配,而是说他们不能成为自动装配其它bean的候选.

