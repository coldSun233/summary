Spring 是⼀种轻量级的控制反转和面向切面编程的开发框架，旨在提⾼开发⼈员的开发效率以及系统的可维护性  

### IOC创建对象的方式

**ICO**

IoC（Inverse of Control:控制反转）是⼀种设计思想，就是 将原本在程序中⼿动创建对象的控制权，交由Spring框架来管理。  

1. 默认使用无参构造器

2. 使用有参构造器创建对象

    ```java
    package examples;
    
    public class ExampleBean {
    
        // Number of years to calculate the Ultimate Answer
        private int years;
    
        // The Answer to Life, the Universe, and Everything
        private String ultimateAnswer;
    
        public ExampleBean(int years, String ultimateAnswer) {
            this.years = years;
            this.ultimateAnswer = ultimateAnswer;
        }
    }
    ```

    - 下标赋值

        ```xml
        <bean id="exampleBean" class="examples.ExampleBean">
            <constructor-arg index="0" value="7500000"/>
            <constructor-arg index="1" value="42"/>
        </bean>
        ```

    - 类型(不建议使用)

        ```xml
        <bean id="exampleBean" class="examples.ExampleBean">
            <constructor-arg type="int" value="7500000"/>
            <constructor-arg type="java.lang.String" value="42"/>
        </bean>
        ```

    - 参数名

        ```xml
        <bean id="exampleBean" class="examples.ExampleBean">
            <constructor-arg name="years" value="7500000"/>
            <constructor-arg name="ultimateAnswer" value="42"/>
        </bean>
        ```

在配置文件加载的时候，容器管理的对象就已经初始化了。

### 别名

```java
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
<alias name="exampleBean" alias="eBean"/>
```

### Bean的配置

```xml
<!-- 
id:bean的唯一标识
class:类的全限定名，包名+类型
name：别名，可以取多个，通过逗号或空格分开
-->
<bean id="exampleBean" class="examples.ExampleBean" name="eBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
<alias name="exampleBean" alias="eBean"/>
```

#### import

将多个配置文件导入合并为一个

```xml
<import source="beans1.xml"/>
<import source="beans2.xml"/>
```

### 依赖注入

1. 构造器注入

2. Set方式注入

    bean对象的创建依赖于容器，bean对象中的所有属性由容器注入

    ```xml
    <bean id="student" class="com.leng.pojo.Student">
            <!-- 普通注入 value -->
            <property name="name" value="spring"/>
            <!-- ref -->
            <property name="address" ref="address"/>
            <!-- 数组 -->
            <property name="books">
                <array>
                    <value>SQL：从删库到跑路</value>
                    <value>C++：从入门到入土</value>
                    <value>摸鱼学导论</value>
                </array>
            </property>
            <!-- List -->
            <property name="hobbies">
                <list>
                    <value>抽烟</value>
                    <value>喝酒</value>
                    <value>烫头</value>
                </list>
            </property>
            <!-- Map -->
            <property name="cards">
                <map>
                    <entry key="身份证" value="1234"/>
                    <entry key="护照" value="357349"/>
                </map>
            </property>
            <!-- Set -->
            <property name="games">
                <set>
                    <value>oll</value>
                    <value>lol</value>
                    <value>llo</value>
                </set>
            </property>
            <!-- properties-->
            <property name="info">
                <props>
                    <prop key="age">12</prop>
                    <prop key="gender">男</prop>
                </props>
            </property>
            <!-- null -->
            <property name="wife">
                <null/>
            </property>
        </bean>
    
        <bean id="address" class="com.leng.pojo.Address">
            <property name="address" value="hust"/>
        </bean>
    </beans>
    ```

3. 扩展方式注入

    使用p-namespace和c-namespace

    ```xml
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:p="http://www.springframework.org/schema/p"
        xmlns:c="http://www.springframework.org/schema/c"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd">
    
        <bean name="classic" class="com.example.ExampleBean">
            <property name="email" value="someone@somewhere.com"/>
        </bean>
    
        <bean name="p-namespace" class="com.example.ExampleBean"
            p:email="someone@somewhere.com"/>
        
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

### Bean的作用域scope

1. singleton 单例模式，默认机制
2. prototype 原型模式，每次get都会产生一个新的对象

### Bean的自动装配

1. `<bean id="..." class="..." autowire="byName">`，byName会自动在容器上下文中进行查找和自己对象set方法后面的值对应的`bean`
2. `<bean id="..." class="..." autowire="byType">`，byType会自动在容器上下文中进行查找和自己对象类型相同的bean`，**但这个类型的bean必须全局唯一**

#### 注解实现自动装配

首先要导入约束，配置注解支持

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

##### @Autowire

`@Autowire`默认按类型，找不到就报错，找到多个则按名字，若还是找不到，则报错，这时可以通过**`@Qualifier(value="xx")`指定id来确定具体是哪一个**。

将@Autowire注解加在属性上，也可以加在set方法上，建议加在set上，在属性上加@Autowire可以不需要set方法（此时，注解直接反射给属性复制）。XML和注解都是反射！！！ **注解是反射直接给属性赋值，xml的是反射获得get、set方法赋值**

**@Autowire与@Resource**

@Autowired注解是按类型装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它required属性为false。

**@Resource**注解和@Autowired一样，也可以标注在字段或属性的setter方法上，但它**默认按名称装配，找不到再按类型装配，按类型找到多个则报错。名称可以通过@Resource的name属性指定**，如果没有指定name属性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。 

@Resources按名字，是JDK的，@Autowired按类型，是Spring的。

**@Component("id")**

放在类上，说明这个类被Spring管理，等同于直接在xml文件中进行bean的注册，如果不设置id，默认为首字母小写的类名

web开发，提供3个@Component注解衍生注解（功能一样）取代
@Repository(“名称”)：dao层
@Service(“名称”)：service层
@Controller(“名称”)：controller层

**一般来说，XML负责管理bean，注解只负责完成属性的注入**

### 代理模式

#### 静态代理

代理对象和真实对象都要实现同一个接口，代理对象好代理真实对象。代理是对方法的增强，装饰器是对对象的增强

好处：代理对象可以做很多真实对象做不了的事情，真实对象只专注于自己的事，