#### Spring学习笔记

---

##### Spring概述

###### 简介

- 分层的全栈轻量级开源框架, 以IOC和AOP为主要特性 ,方便整合众多优秀的第三方框架和类库
- 官网:spring.io
- 分层:MVC
- 全栈:web(spring mvc) service(IOC容器) jdbc(jdbcTemplate)
- 轻量级:主要式看对容器的依赖性所决定的,依赖性越小越轻量

###### 优势

- 解耦
- 面向切面编程
- 声明式事务
- 方便测试
- 方便集成各种优秀框架
- 解决企业级应用开发的复杂性
- 源码是经典的Java学习范例

###### 整体架构

![image-20200428134218892](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200428134218892.png)

###### IOC

什么是IOC

​	Inversion of Control控制反转:由IOC容器负责对象的创建和管理

​	描述的事情:对象的创建,管理的问题

​	控制:对象创建的控制权

​	反转:对象的控制权由传统的程序员通过new方式创建变为了由IOC容器负责对象的创建,销毁

IOC解决了什么问题

​	解决了对象之间的耦合

IOC和DI的区别联系

​	Dependence Injection依赖注入

​	IOC和DI描述的是同一件事(对象实例化以及依赖关系维护这件事),只不过角度不同

​	IOC站在对象的角度,对象实例化的控制权交给了IOC容器

​	DI站在容器的角度,容器会把对象依赖的其他对象帮其注入为属性

​	DI注入属性的xml配置(从数据类型划分)

​	基本类型以及String	

```xml
<bean id="" class="xxx">
    <property name="name" value="xcl"></property>
</bean>
```

​	对象类型

```xml
<bean id="userService" class="xxx">
    <property name="userDao" ref="userDao"></property>
</bean>
<bean id="userDao" class="xxx"></bean>
```

​	复杂类型之ist(单值类型)

```xml
<bean id="userService" class="xxx">
    <property name="list">
    	<list>
        	<value></value>
        </list>
    </property>
</bean>

```

​	复杂类型之map(键值对类型)

```xml
<bean id="userService" class="xxx">
    <property name="map">
    	<map>
        	<entry key="" value=""></entry>
            <entry key="" value=""></entry>
        </map>
    </property>
</bean>
```

​	复杂类型之Properties

```xml
<bean id="userService" class="xxx">
    <property name="map">
    	<props>
        	<prop key="">value</prop>
        </props
    </property>
</bean>
```

IOC底层原理(xml方式为例)

- xml配置文件
- dom4j解析xml
- 工厂设计模式
- 反射

###### AOP

什么是AOP

​	面向切面编程

​	常见应用:事务控制,权限校验,日志

​	在多个纵向流程中存在相同子流程代码,我们称之为横切逻辑代码

​	横切逻辑代码存在的问题:

​			代码重复性

​			不可维护性(横切逻辑代码和业务逻辑代码混合在一起)		

AOP解决了什么问题

​	在不改变原有业务逻辑的情况下,增强横切逻辑代码

​	好处:解耦合,避免代码重复

为什么叫面向切面编程

​	切:指的是横切逻辑

​	面:横切逻辑往往作用于多个方法,每一个方法都如同一个点,点构成面

AOP底层原理:动态代理

---

##### 自定义框架(手写IOC和AOP)

###### 问题分析及解决方案

1. new关键字带来的紧耦合,破坏面向接口编程,并非实现的设计原则

   解决方案:反射

   BeanFactory工厂类通过读取并解析xml配置文件,然后通过反射技术实例化对象;实例化对象完成后维护对象的依赖关系(通过setter进行注入);将对象保存在map中,以供后续使用

   ```java
   //实例化
   Class cls = Class.forName("全限定类名");
   Object o = cls.newInstance();
   //维护依赖
   List<Elements> propertyList = rootElement.selectNodes("//property");
   
   ```

   对象的初始化操作在容器启动时就完成(static静态代码块)

   UserDao userDao = new UserDaoImp() 

   ​	-> UserDao userDao = (UserDao)BeanFactory.get("id")

   ​		->UserDao userDao(属性注入)

2. service层没有进行事务控制

   ​	数据库事务控制就是Connection的事务

   ​	connection.commit(),connection.rollback(),connection.setAutoCommit(false)

   ​	一个请求中的两个update操作使用了两个connection连接

   ​		*将connection绑定在当前线程ThreadLocal

   ​	事务控制处理目前在dao层,没在service层

   ​		*在service层处理事务:成功手动提交事务,失败回滚事务,关闭连接

   ​	*优化

   ​	封装思想->TransactionManager(开启事务\提交事务\回滚事务)

   ​	AOP:在不改变目标类的情况下,为目标类添加我们的横切逻辑

扩展

- JDK动态代理
- CGLIB动态代理

---

##### Spring IOC深度解读

###### 基础知识

Spring IOC基础

​	BeanFactory与ApplicationContext的区别

​	BeanFactory:spring IOC容器的顶层接口,用于定义一些基础功能,定义一些基础规范

​	ApplicationContext:是BeanFactory的子接口,在具备BeanFactory功能之外,支持国际化以及资源访问等功能

| bean定义模式 | JavaSE                                                     | JavaWeb               |
| ------------ | ---------------------------------------------------------- | --------------------- |
| xml          | new ClassPathXmlApplicationContext("beans.xml")            | ContextLoaderListener |
| xml+注解     | new ClassPathXmlApplicationContext("beans.xml")            | ContextLoaderListener |
| 注解         | new AnnotationConfigApplicationContext(SpringConfig.class) | ContextLoaderListener |

web.xml

```xml
<!--指定IOC容器的配置文件路径-->
<context-param>
	<param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
<!--使用监听器启动spring IOC容器-->
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

###### 实例化Bean的三种方式

无参构造器/使用静态工厂/使用实例工厂

```xml
<!--无参数构造器,如不存在会报错-->
<bean id="util" class="com.xcl.utils.Utilutil"></bean>
<!--使用静态工厂-->
<bean id="util" class="com.xcl.factory.Factory" factory-method="getUtilStatic"></bean>
<!--使用实例工厂-->
<bean id="factory" class="com.xcl.factory.Factory"></bean>
<bean id="util" factory-bean="factory" factory-method="getUtil"></bean>
```

###### Bean属性注入的两种方式

有参构造器/setter()方法

```xml
<!--有参构造器-->
<bean id="util" class="com.xcl.utils.Utilutil">
	<constructor-arg name="name" value="xxx"></constructor-arg>
</bean>
<!--setter-->
<bean id="util" class="com.xcl.utils.Utilutil">
	<property name="name" value="xxx"></property>
</bean>

```

###### Bean作用范围及生命周期

scope作用范围

​	-singleton:单例,默认配置

​	-prototype:原型(多例),每次获取getBean都返回新对象

​	-request:一个请求,存在request域中

​	-session:一次会话,存在session域中

​	-application:上下文

不同作用范围的生命周期

| 作用范围  | 创建                          | 销毁                                | 总结                        |
| --------- | ----------------------------- | ----------------------------------- | --------------------------- |
| singleton | 容器创建时,对象就会被创建     | 销毁容器时                          | 与容器生命周期一致          |
| prototype | 当使用对象时,创建新的对象实例 | 对象长时间不用,被java垃圾回收器回收 | spring只负责创建,不负责销毁 |

###### Bean的标签属性

| 属性           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| id             | 唯一标识一个bean                                             |
| name           | 为bean提供一个或多个名称                                     |
| class          | bean的全限定类名                                             |
| scope          | bean的作用范围                                               |
| factory-bean   | 为当前要创建的bean对象指定工厂(指定改属性后,class失效)       |
| factory-method | 为当前要创建的bean对象指定工厂的方法(如配合class用,则改方法必须静态;如配合factoty-bean用,class失效) |
| init-method    | 指定bean的初始化方法,此方法会在对象装配后使用,必须是哟个无参方法 |
| destory-method | 指定bean的销毁方法,此方法在bean销毁前执行,仅当scope为singleton是起作用 |

###### springBean的生命周期

![image-20200429165545235](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200429165545235.png)

###### 注解与xml的对应关系

|      | 创建对象                                    | 属性注入                             |
| ---- | ------------------------------------------- | ------------------------------------ |
| xml  | <bean></bean>                               | <property></property>                |
| 注解 | @Component/@Service/@Controller/@Repository | @Autowired @Resource(name="userDao") |

*为啥spring要提供@Service/@Controller/@Repository?

@Component/@Service/@Controller/@Repository功能一样,用于创建对象

好处:标记类本身的用途清晰@Service业务层/@Controller控制层/@Repository持久层;便于后续针对性扩展功能

*@Autowied @Resource(name="userDao")的区别

@Autowired根据类型,自动装配;@Resource(name="userDao")根据对象的名称,进行注入

使用@Autowired按照类型进行区分时,如果按照类型不能唯一确定对象,需要结合@Qualifier("name")注解指定具体的id

*xml+注解的使用方式

​	1)开启注解扫描

```xml
<context:component-scan base-package=""></context:component-scan>

```

​	2)使用创建对象的注解

*通常情况下,自定义的类通过注解开发,第三方类通过xml开发

*@Autowired是spring提供的, @Resource是java扩展包中的(如果spring中要用@Resource,需引入spring-annotation的jar包)

*纯注解模式

@Configuration:标明这是一个配置类

@ComponentScan:开启注解扫描

@PropertySource:加载外部资源文件

@Import:导入其他配置

@Value:为变量赋值

@Bean:将方法放回的对象交给IOC容器

@Lazy:延迟加载

###### 高级特性

###### lazy-init延迟加载

​	lazy-init="false",立即加载,在spring启动时就加载

​	lazy-init="true",延迟加载,在getBean()时加载

​	注:只适用于scope="singleton"的bean

###### FactoryBean和BeanFactory

​	BeanFactory:容器的顶层接口,定义一些基础功能

​	FactoryBean:用于配置一些复杂对象的生成,implements FactoryBean

###### 后置处理器

​	spring提供了两种后置处理器:BeanPostProcessor和BeanFactoryPostProcessor

###### 源码刨析

###### 读源码的原则&方法

- 定焦原则:确定主线

- 宏观原则:要有整体思维

- 断点(观察调用栈)

- 反调(Find Usages)

- 经验(学会总结归纳)

- 搭建源码环境

  编译顺序:core-oxm-context-beans-aspects-aop

  ![image-20200429172130525](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200429172130525.png)

###### IOC初始化的主体流程

###### BeanFactory获取子流程

###### BeanDefinition加载注册子流程

###### Bean创建流程

###### 延迟加载原理

###### 循环依赖问题

​	spring中循环依赖场景有:

​		构造器的循环依赖

​		setter方法的循环依赖

​		(单例bean的构造器循环依赖无法解决;prototype原型bean的循环依赖无法解决)

​	单例bean的setter()属性循环依赖的解决原理:

---

##### Spring AOP应用

###### 相关术语

| 术语            | 解释                                                         |
| --------------- | ------------------------------------------------------------ |
| JoinPoint连接点 | 它指的是可以把增强代码加入到业务主线中的点,这些点指的就是业务类中的方法 |
| Pointcut切入点  | 指的是已经把增强代码运用到业务代码中的连接点                 |
| Advice增强/通知 | 提供增强功能的方法(做什么);指定方位点(什么时候做,业务方法执行前/后) |
| Target目标类    | 需要被增强的对象                                             |
| Proxy代理类     | 一个类被织入增强后产生的代理对象                             |
| Weaving织入     | 把增强运用到目标类来创建代理对象的过程(动态代理)             |
| Aspect切面      | 描述增强关注的所有方面,即切入点+通知的集合,表明了"在哪个地方,什么时候执行增强逻辑代码" |

默认情况下,spring会根据目标类是否实现接口来选择JDk动态代理还是CGLIB动态代理.目标类实现了接口,采用JDK,否,采用CGLIB(可通过配置,强制使用CGLIB)

###### spring 中AOP的使用

​	xml

​		引入坐标	

```xml
<dependency>    																			<groupId>org.springframework</groupId>    			
    <artifactId>spring-aop</artifactId>    				
    <version>5.1.12.RELEASE</version>
</dependency> 
<dependency>    
    <groupId>org.aspectj</groupId>           												<artifactId>aspectjweaver</artifactId>    			
    <version>1.9.4</version>
</dependency>


```

​		配置	

```xml
<!--约束--> xmlns:aop="http://www.springframework.org/schema/aop"      http://www.springframework.org/schema/aop          https://www.springframework.org/schema/aop/spring-aop.xsd  

<bean id="logUtil" class="com.lagou.utils.LogUtil"></bean>

<aop:config>
	<!--配置切面-->
    <aop:aspect id="log" ref="logUtil">
    <!--前置通知:可通过JoinPoint.getArgs()获取参数-->
        <aop:before method+"beforeMethod" pointcut-ref="pointcut"></aop:before>
    <!--后置通知-->
        <!--结束通知:无论是否异常都执行-->
        <aop:after></aop:after>
        <!--正常结束:可获取返回值-->
        <aop:after-returing returing="retValue"></aop:after-returing>
        <!--异常通知-->
        <aop:after-throwing></aop:after-throwing>
    <!--环绕通知:ProceedingJoinPoint.proceed()-->
        <aop:around></aop:around>
    <!--切入点-->
        <!--execution(访问修饰符 方法返回值 全限定类名.方法(参数))-->
        <!--访问修饰符可省略,返回值可使用*表任意-->
        <!--方法(参数) 参数*表明有参数但不知道具体类型 ..也可能有也可能没有参数-->
    <aop:pointcut id="pointcut" expression="execution()"/>
    </aop:aspect>
</aop:config>


 


```

​	xml+注解

```xml
<!--开启aop注解-->
<!--强制使用CGLIB动态代理:proxy-target-class="true"-->
<aop:aspectj-autoproxy proxy-target-class="true"/>

```

​	注解

```java
//开启aop注解
@EnableAspectAutoProxy

```

###### AOP代理对象创建流程





---

##### 事务

###### 概念

​	事务指一组操作要么全都执行,要么全都不执行.保证数据的一致性和完整性

###### 四大特性ACID

​	原子性:一组操作要么都执行,要么都不执行

​	一致性:事务必须使数据库数据从一个一致性状态变换为另一个一致性状态

​	隔离性:每个事务都不能被其他事务所干扰

​	持久性:一个事务一旦提交,它对数据库数据的改变就是永久性的,即使数据库产生异常也不该对其有任何影响

###### 隔离级别	

​	解决事务并发中的问题

| 隔离级别                   | 可解决的问题                |
| -------------------------- | --------------------------- |
| Read uncommitted(读未提交) | 最低级别,所有问题都不能解决 |
| Read committed(读已提交)   | 可避免脏读                  |
| Repeatable read(可重复读)  | 可避免脏读\不可重复读       |
| Serializable(串行化)       | 可避免脏读\不可重复读\幻读  |

脏读:一个事务读到了另一个事务未提交的数据

不可重复读):一个事务读到了另一个事务已提交的update数据

幻读:一个事务读到另一个事务已经提交的insert或delete数据(前后数据不一样)

MySQL默认隔离级别:Repeatable read

查询当前使用的隔离级别:select @@tx_isolation

设置MySQL事务的隔离级别(会话级别):set session transaction isolation level xxx

###### 传播行为	

| 传播行为                   | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| required(必须存在一个事务) | 如果当前存在事务则加入,否则新建一个事务                      |
| supports(佛性)             | 如果当前存在事务则加入,否则已非事务方式运行                  |
| manadatory(强制要求)       | 如果当前存在事务则加入,否则抛出异常                          |
| required_new(新建事务)     | 新建事务,如果当前存在事务将其挂起                            |
| not_supported(不支持事务)  | 已非事务方式运行,如果当前存在事务将其挂起                    |
| never(不支持事务)          | 已非事务方式运行,如果当前存在事务抛出异常                    |
| nested(嵌套事务)           | 如果当前存在事务,则在嵌套事务内运行.如果当前没有事务新建事务 |

###### 事务API

​	PlatformTransactionManager事务管理器接口

​	TransactionDefinition事务定义信息(隔离级别/传播行为/超时/是否只读)

​	TransactionStatus事务运行状态

###### 声明式事务

​	通过xml或注解的方式完成事务的控制(AOP实现)

​		xml模式

```xml
<!--数据源-->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
	<property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql:///custom_mybatis"></property>
    <property name="username" value="root"></property>
    <property name="password" value="root"></property>
</bean>
<!--事务管理器-->
<bean id="transactionManage" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <constructor-arg name="dataSource" ref="dataSource"></constructor-arg>
</bean>
<!--配置增强-->
<tx:advice id="txAdvice" transaction-manager="transactionManage">
    <!--定制事务细节:隔离级别/传播行为/超时/是否只读/回滚-->
    <tx:attributes>
    	<tx:method name="method" read-only="true" propagation="REQUIRED" isolation="DEFAULT"></tx:method>
        <tx:method name="*" read-only="false" propagation="REQUIRED"></tx:method>
    </tx:attributes>
</tx:advice>
<!--aop配置-->
<aop:config>
	<!--advice-ref指向增强:横切逻辑+方位点-->
    <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.xcl.service.UserImp.*(..))"/>
</aop:config>

```

​		xml+注解模式

```xml
<!--开启注解支持--> 
<tx:annotation-driven transaction-manager="transactionManager"/>
//使用
@Transactional(readOnly = true,propagation = Propagation.SUPPORTS

```

​		注解模式

```java
//开启注解支持
@EnableTransactionManagement 

```

###### 编程式事务

​	在业务代码中添加事务控制代码

```java
//数据库的事务本质上就是jdbc的connection事务
//开启事务
connection.setAutoCommit(false);
//事务提交
connection.commit();
//事务回滚
connection.rollback();
//事务关闭
connection.close();

```

​	

---

##### 设计模式

###### 工厂模式:生产实例对象

​	*简单工厂

​	*工厂方法

###### 单例模式:确保一个类只有一个实例

​	*懒汉式	

​	*饿汉式

​	*双重检测式

​	*静态内部类

​	*枚举式

###### 代理模式:为目标类增强功能

​	*静态代理

​		目标类与代理类需要实现同一个接口(ps:接口定义了两者的行为)

​	*JDK动态代理:给定目标类,根据目标类生成代理类(要求:目标类需要实现接口)

```java
Object proxy = Proxy.newProxyInstance(ClassLoader classLoader,Class<?>[] interfaces,myHandle);

class MyHandle implements Invocation{
    @Override
    //代理对象,需要执行的方法,方法参数 
    public Object invoke(Object proxy, Method method,Object[] args){
        System.out.printlh("to do something");
         Object result = method.invoke(目标对象,参数);
        System.out.printlh("to do something");     
    }
}

```

​	*CGLIB动态代理:给定目标类,根据目标类生成代理类	

```java
Enhance.create(Class<?> cls,new MethodInterceptor(){
	@Override
	public Object intercept(Object o, Method method,Object[] args, MethodProxy methodProxy){
	    System.out.println("前置增强");
         Object result = method.invoke(目标对象,参数);
        System.out.println("后置增强");     
	}
});

```





