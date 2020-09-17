

# 深入 Spring IoC 系列 - 4.4: 依赖-依赖及配置详情2



[TOC]



### 集合

 `<list/>`, `<set/>`, `<map/>`, 和 `<props/>`  元素代表了 Java `Collection` 类型 `List`, `Set`, `Map`, and `Properties` 如下展示如何使用他们:

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

map 键或值,或者是 set值也可以是如下元素:

```xml
bean | ref | idref | list | set | map | props | value | null
```



#### 集合合并

Spring 容器同样支持合并集合. 你可以定义一个父  `<list/>`, `<map/>`, `<set/>` 或者 `<props/>`  元素并且拥有子  `<list/>`, `<map/>`, `<set/>` 或者 `<props/>` 元素继承并重写父集合的值.也就是说子集合的值是父元素和子元素的合并,并且子集合的元素值会覆盖父集合的值.

这里介绍了父子bean合并机制.如果对父子bean定义不熟悉的可以先阅读[这里](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-child-bean-definitions).

下面的例子展示了集合合并:

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

注意 `child` bean定义中  `adminEmails` 属性中 `<props/>` 元素上的  `merge=true`  属性. 当 `child` bean 被解析并且被容器实例化,最后的实例会拥有一个 `adminEmails` `Properties` 集合,它包含了子bean的 `adminEmails` 集合和 父bean `adminEmails` 集合的合并.结果如下:

```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

子 `Properties` 集合的值继承了父类的 `<props/>`,并且子bean 的 `support` 值覆盖了父bean的该值.

这个合并行为和  `<list/>`, `<map/>`, 以及 `<set/>` 集合类型的合并相似.在`<list/>` 元素中, `List` 集合类型的语义被保存下来(也就是,这个集合的值是 `ordered` 有序的).父bean的值是优先于子列表的值的.当使用  `Map`, `Set`, 和 `Properties` 时,不存在顺序.因此,在容器内部使用这些类型是没有排序效果的语义的.

#### 集合合并的限制

你不可以合并不同的集合类型(比如 `Map` 和 `List` ).如果你这么做的话会抛出一个 `Exception` . `merge` 属性必须指定在一个更低层级的,继承的,子定义中.在一个父集合定义中指定 `merge` 属性是多余的并且不会产生期望的合并效果.



#### 强类型集合

随着 Java 5 引入了泛型,你可以使用强类型的集合. 也就是说,可以声明一个 `Collection` 类型只能包含(比如) `String` 元素. 如果你使用 Spring 来依赖注入一个强类型的 `Collection` 到bean中,你可以得到Spring的类型转换支持,这使得你的强类型 `Collection` 实例会在添加到 `Collection` 之前转换成合适的类型.如下所示:

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

 当 `something` bean 的 `accounts` 属性准备好注入的时候, 可以通过反射获得有关强类型Map <String，Float>的元素类型的泛型信息.因此,因此，Spring的类型转换机制将各种值元素识别为 `Float` 类型,并将字符串值（`9.99`、`2.75`和`3.99`）转换为实际的Float类型.



### null 和 空白的 字符串值

Spring中将空白的参数值视为空白的 `String` .下面的基于XML的配置元数据设置  `email` 属性为空白 `String` 值("")

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

前面的例子等同于 

```java
exampleBean.setEmail("");
```

`<null/>` 元素处理 `null` 的值.如下:

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

前面的例子等同于 

```java
exampleBean.setEmail(null);
```



