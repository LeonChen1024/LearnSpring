# 深入 Spring IoC 系列 - 7.1:  bean 定义继承



[TOC]

一个 bean 定义会包含很多配置信息,包括构造器参数,属性值,还有容器指定的信息,比如初始化方法,静态工厂名等.子 bean 定义继承了父 bean 定义的配置.子定义可以重写或者添加属性值.使用父子 bean 关系可以节省大量代码.也是模板化的一种模式.

如果你编程式的使用 `ApplicationContext` 接口,子定义使用 `ChildBeanDefinition` 来表示.大部分用户不用在这个级别上使用他们,二市在 `ClassPathXmlApplicationContext` 这样的类中声明 bean 定义.当你使用 XML 配置元数据时,你可以使用 `parent` 属性来指定他的父bean.如下:

```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  <!-- 这里的 parent 指定了父 bean -->
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

一个子 bean 定义使用了父类bean 的属性但是也可以重写这些属性.没有重写的时候子类必须接受父类的属性值.

子bean 定义会继承父bean 的作用域,构造器参数值,属性值,方法等,并且可以添加新的值.你指定的任何作用域,初始化方法,销毁方法,或者`static` 工厂方法都会覆盖父bean 的设置.

其余的设置都会从子定义中获取:依赖,装配模式,依赖检查,单例,和懒初始化.

上面的例子通过`abstract` 属性明确指出这个父定义是个抽象的.如果一个父定义没有指定一个类,必须声明为 `abstract` ,如下:

```xml
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```

这个父bean 不能够实例化因为它是不完整的,并且它还被显式声明为`abstract` .当一个定义是 `abstract` ,它只能作为一个单纯的模板 bean 定义提供给子 bean 使用.如果尝试使用这个 `abstract` 父bean 本身,比如通过引用它作为一个属性值或者通过 `getBean()` 调用都会返回一个异常.类似的,容器内部的 `preInstantiateSingletons()` 方法也会忽略定义为 `abstract` 的 bean.

> `ApplicationContext` 默认会预先实例化所有的单例.因此,如果你又一个(父)bean 定义你只想把它当做一个模板,并且这个定义指定了一个类,你要确保设置了 abstract 为 true,否则应用上下文会预先实例化这个 `abstract` bean









































