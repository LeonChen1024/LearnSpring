

# 深入 Spring IoC 系列 - 4.5: 依赖-依赖及配置详情3



[TOC]



### p-命名空间的XML快捷方式

p命名控件使你可以使用 `bean` 元素的属性(而不是嵌套 `<property/>` 元素)来描述你的属性值和协同bean.

Spring 支持带有[命名空间](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#xsd-schemas)的可扩展配置格式,同样是基于 XML 模式定义. 本章中关于 `beans` 配置格式的讨论都是基于 XML 模式定义文档的.然而,p命名空间未在XSD文件中定义,仅存在于Spring的核心中.

下面的示例展示了两个XML代码段(第一个使用标准XML格式,第二个使用p命名空间),它们可以解析为相同的结果:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

这个例子展示了一个p命名控件叫 `email` .并且Spring 会包含这个属性的定义.如前面所说,p命名空间没有模式定义,所以你可以将参数名设置为映射的属性名称

下面的例子展示了将bean定义指向其它的bean的情况:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

这个示例展示了定义属性引用的情况. 标准用法使用  `<property name="spouse" ref="jane"/>`  来创建一个引用使得 `john` 引用 `jane` , 而第二个bean定义使用  `p:spouse-ref="jane"`  作为一个参数来做相同的事情. 在这里 `spouse` 代表属性名,而 `-ref` 表明了这不是一个直接值而是一个对其它bean的引用.

> p命名空间的灵活性是不如标准xml格式的.比如,引用的情况下会和以 `Ref` 结尾的属性冲突,而标准格式不会.应该要根据自身的情况和团队沟通避免使用过多的方式来做相同的事.



### c-命名空间的XML快捷方式

和 [p-命名空间的XML快捷方式](#p-命名空间的XML快捷方式) 类似,c-命名空间,从Spring3.1开始引入,允许使用行内属性配置构造器参数而不用使用嵌套 `constructor-arg` 元素.

如下:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

`c:` 命名空间使用了和 `p:` 相同的风格(在尾部加上 `-ref` 来表明bean引用)来通过他们的名字设置构造器参数.同样,即使未在XSD模式中定义它(也存在于Spring内核中),也需要在XML文件中声明它.

对于极少数情况下无法使用构造函数自变量名称的情况(通常,如果字节码是在没有调试信息的情况下编译的),则可以使用参数索引,如下所示:

```xml
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

> 由于XML语法的原因,索引标记要求使用前导`_`,如同XML属性名称不能以数字开头（即使某些IDE允许）.相应的索引标记也可用于`<constructor-arg>`元素,但并不常用,因为声明使用普通顺序通常就足够了.

实际上,构造函数解析机制在匹配参数方面非常有效,因此,除非你确实需要,否则我们建议在整个配置过程中使用名称表示法.

### 复合属性名称

设置bean属性时,可以使用复合属性名称或嵌套属性名称,只要路径中除最终属性名称以外的所有组件都不为空即可.思考如下bean 定义:

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

`something` bean具有 `fred` 属性,该属性具有 `bob` 属性,该属性具有 `sammy` 属性,并且最终的`sammy` 属性被设置为`123`.为了使它生效,在构造bean之后,`something`的`fred`属性和`fred`的`bob`属性一定不能为`null`.否则,将导致`NullPointerException`.



