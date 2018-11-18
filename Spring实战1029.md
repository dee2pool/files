## 三种装配Bean的方式

---

- 在xml中显示配置
- 在java中进行显示配置
- 自动化装配

### 1.自动装配bean

spring 从两个角度来实现自动化装配

1.组件扫描:spring会自动发现应用上下文中所创建的bean

2.自动装配:spring自动满足bean之间的依赖

在类上使用@component注解。这个注解表明这个类会作为组件类，并告知spring要为这个类创建bean。

默认情况下组件扫描是不启用的，需要配置一下spring，从而命令它去寻找带有@component注解的类，并为其创建bean

@ComponentScan注解这个注解能够在Spring中启用组件扫描

如果没有配置的话，@ComponentScan默认会扫描与配置类相同的包，如果想要改变扫描的包可以通过@ComponentScan("包名")的方式改变还可以通过@ComponentScan(basePackages={"包1","包2"})的方式设置多个扫描包。还可以通过@ComponentScan(basePackageClasses={CDPlayer.class,DVDPlayer.class})方式将这些类所在的包作为组件扫描的基础包

如果使用xml来启用组件扫描的话，可以使用Spring context命名空间<context:component-scan>元素

例如在xml中加入这样一行:<context:component-scan base-package=""/>

Spring应用上下文中所有的bean都会给定一个id,如果没有显示给定这个id，id默认为类名的第一个字母小写，可以通过@Component("name")的方式设置组件的id

@Autowired注解可以让Spring自动满足bean之间的依赖，这个注解可以用在类的任何方法上，如果有且只有一个bean匹配依赖需求的话，那么这个bean将会被装配进来，如果没有匹配的bean，Spring会抛出异常。为了避免异常，可以将@Autowired的required的属性设置为false。如果有多个bean都能满足依赖关系的话，Spring将会抛出一个异常，表明没有明确指定要选择哪给bean。

> tips : 在测试代码中不要使用print语句，应该使用断言例如assertNotNull()断言对象不为空，
>
> standardoutputStreamLog,源于System Rules库的一个Junit规则，该规则能够基于控制台的输出编写断言

显示装配bean:如果将第三发库中的组件装配到你的应用中，在这种情况下就不能在它的类上添加注解，需要进行显示装配bean

显示装配bean有两种方式:java和xml。

通过javaConfig配置bean:javaConfig是配置代码，应该和业务代码分开，将它放在单独的包中。

通过为类添加@Configuration注解表明这个类是配置类。

在javaConfig中声明bean需要编写一个方法，这个方法会创建所需类型的实例，然后给这个方法添加@Bean注解。

```java
@Bean
Public CompatDisc sgtPeppers(){
    return new SgtPeppers();
}
```

@Bean 注解会告诉Spring这个方法将会返回一个对象，该对象要注册为Spring应用上下文中的bean，默认情况下bean的Id与带有@Bean注解的方法名一样如果向设置成一个不同的名字，可以通过name属性指定一个名字

在javaConfig中依赖注入

```
@Bean
public CDPlayer cdplayer(CompactDisc compactDisc){
    return new CDPlayer(compactDisc);
}
```

这样当Spring调用cdplayer()创建cdplayerbean时，他会自动装配一个CompactDisc到配置方法中。

>tips Spring中的bean都是单例的

通过XML装配bean:不推荐使用xml装配bean

在xml中使用<bean id="" class="">;通过id属性为bean命名

借助构造器注入初始化的bean:有两种方式

1.使用<constructor-arg>元素

2.使用Spring3.0引入的c命名空间

```
#使用<constructor-art>
<bean id='' class=''>
	<constructor-arg ref=""/>
</bean>
```

```
#使用c命名空间
<bean id="" class="" c:cd-ref="">//cd 表示构造器参数名
```

将字面量值注入到构造器中

```xml
<bean id="" class="">
	<constructor-arg value=""/>
    <constructor-arg value=""/>
</bean>
```

```xml
#使用c命名空间
<bean id="" class="" c:_0="" c:_1=""/>
```

将集合装配到构造器中

```xml
<bean id="" class="">
    <consotructor-arg>
        <list>
            <value></value>
        </list>
    </consotructor-arg>
</bean>
<bean id="" class="">
    <consotructor-arg>
        <list>
            <ref bean="">
        </list>
    </consotructor-arg>
</bean>
```

使用set设置属性

```xml
<bean id="" class="">
    <property name="" ref=""/>
</bean>
#使用p命名空间
<bean id="" class="" p:compactDisc-ref="">//compactDisc为属性名
```

将字面量和集合注入属性中

```xml
<bean id="" class="">
    <property name="" value=""></property>
    <property name="">
        <list>
            <value></value>
        </list>
    </property>
</bean>
```

> util命名空间:util命名空间能创建一个list map等类型的bean可以用于p空间引用集合

导入和混合配置

@Import注解可以将其他类导入到当前类中

@Import如果有多个参数，参数之间相互依赖，则可以自动完成依赖关系的配置

@ImportResource可以导入配置文件到当前类中

```
@ImportResource("classpath:xxx.xml")
```

在xml配置中引用JavaConfig

```
<bean class="soundsystem.CDConfig">
```

## 高级装配

---

profile bean

要使用profile，需要将所有不同的bean定义到一个或多个profile中，再将应用部署到每个环境中，并确保对应的profile处于激活状态

使用@profile注解指定某个bean属于哪个profile。

Spring通过设置spring.profiles.active和spring.profiles.default的值确定哪个profile是激活的。

有多种方式来设置这两个属性

- 作为dispatcherServlet的初始化参数
- 作为web应用的上下文参数
- 作为jndi条目
- 作为环境变量
- 作为jvm的系统属性
- 在集成测试类上，使用@ActiveProfiles注解设置

```xml
在DispatcherServlet中设置
//在web.xml中加入
<context-param>
	<param-name>spring.profiles.default</param-name>
    <param-value>dev</param-value>
</context-param>
<servlet>
    <init-param>
        <param-name>spring.profiles.default</param-name>
        <param-value>dev</param-value>
    </init-param>
</servlet>
```

当应用部署到QA、生产或其他环境之中时，负责部署的人根据情况使用系统属性、环境变量或JNDI设置spring.profiles.active即可。spring.profiles.active比spring.profiles.default的优先级高

@ActiveProfiles用在测试类中指定运行测试时要激活哪个profile

@Conditional注解可以用在@Bean注解的方法上。如果给定的条件计算结果为true，就会创建这个bean，否则这个bean会被忽略。

@Conditional参数是一个类，这个类可以是任意实现了Condition接口的类型。