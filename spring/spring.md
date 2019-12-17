## IOC

### Bean的装配

#### 自动化装配

自动装配就是让Spring自动满足bean依赖的一种方法，在满足依赖的过程中，会在Spring应用上下文中寻找匹配某个bean需求的其他bean。为了声明要进行自动装配，我们可以借助Spring的@Autowired注解。

Spring从两个角度来实现自动化装配：

- 组件扫描（component scanning）：Spring会自动发现应用上下文中所创建的bean。
- 自动装配（autowiring）：Spring自动满足bean之间的依赖。

#### 通过Java代码装配bean

想要将第三方库中的组件装配到你的应用中，在这种情况下，是没有办法在它的类上添加`@Component`和`@Autowired`注解的，因此就不能使用自动化装配的方案了。

通过`@Configuration`和`@Bean`

#### 通过XML装配bean

该选择构造器注入还是属性注入呢？作为一个通用的规则，我倾向于对强依赖使用构造器注入，而对可选性的依赖使用属性注入。

```xml
<bean id="cdPlayer" class="soundSystem.CDPlayer">
    <!-- 1.通过其它bean -->
    <constructor-arg ref="compactDisc">
    <!-- 2.通过参数 -->
    <constructor-arg value="sgt"/>
    <constructor-arg value="beatles"/>
    <!-- 3.通过list/set -->
    <constructor-arg>
        <list>
        	<value> value1</value>
            <value> value2</value>
            <value> value3</value>
            <ref bean="bean1"></ref>
            <ref bean="bean2"></ref>
            <ref bean="bean3"></ref>
        </list>
        <set>
        	<value> value4</value>
            <ref bean="bean4"></ref>
        </set>
    </constructor-arg>
</bean>
```

当Spring遇到这个<bean>元素时，它会创建一个CDPlayer实例。<constructor-arg>元素会告知Spring要将一个ID为compactDisc的bean引用传递到CDPlayer的构造器中。

使用<constructor-arg>元素进行构造器参数的注入。除了使用“ref”属性来引用其他的bean，还可以使用`value`属性，通过该属性表明给定的值要以字面量的形式注入到构造器之中。

<list>元素是<constructor-arg>的子元素，这表明一个包含值的列表将会传递到构造器中。其中，<value>元素用来指定列表中的每个元素。也可以使用<ref>元素替代<value>，实现bean引用列表的装配。

```xml
<bean id="cdPlayer" class="soundSystem.CDPlayer">
    <property name="compactDisc" ref="compactDisc" />
    <property name="title" value="july"/>
    <property name="artist" value="jay"/>
    <property name="tracks">
        <list>
        	<value> value1</value>
            <value> value2</value>
            <value> value3</value>
        </list>
    </property>
</bean>
```

<property>元素为属性的Setter方法所提供的功能与<constructor-arg>元素为构造器所提供的功能是一样的。在本例中，它引用了ID为compactDisc的bean（通过ref属性），并将其注入到compactDisc属性中（通过`setCompactDisc()`方法。

### bean的作用域

在默认情况下，Spring应用上下文中所有bean都是作为以单例（singleton）的形式创建的。也就是说，不管给定的一个bean被注入到其他bean多少次，每次所注入的都是同一个实例。

在大多数情况下，单例bean是很理想的方案。初始化和垃圾回收对象实例所带来的成本只留给一些小规模任务，在这些任务中，让对象保持无状态并且在应用中反复重用这些对象可能并不合理。

有时候，可能会发现，你所使用的类是易变的（mutable），它们会保持一些状态，因此重用是不安全的。在这种情况下，将class声明为单例的bean就不是什么好主意了，因为对象会被污染，稍后重用的时候会出现意想不到的问题。

Spring定义了多种作用域，可以基于这些作用域创建bean，包括：

- 单例（Singleton）：在整个应用中，只创建bean的一个实例。
- 原型（Prototype）：每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例。
- 会话（Session）：在Web应用中，为每个会话创建一个bean实例。
- 请求（Rquest）：在Web应用中，为每个请求创建一个bean实例。

在Web应用中，如果能够实例化在会话和请求范围内共享的bean，那将是非常有价值的事情。例如，在典型的电子商务应用中，可能会有一个bean代表用户的购物车。如果购物车是单例的话，那么将会导致所有的用户都会向同一个购物车中添加商品。另一方面，如果购物车是原型作用域的，那么在应用中某一个地方往购物车中添加商品，在应用的另外一个地方可能就不可用了，因为在这里注入的是另外一个原型作用域的购物车。就购物车bean来说，会话作用域是最为合适的，因为它与给定的用户关联性最大。

### 运行时值注入

将一个值注入到bean的属性或者构造器参数中。

在Spring中，处理外部值的最简单方式就是声明属性源并通过Spring的Environment来检索属性。

Spring一直支持将属性定义到外部的属性的文件中，并使用占位符值将其插入到Spring bean中。在Spring装配中，占位符的形式为使用“`${... }`”包装的属性名称。

## AOP

Spring AOP构建在动态代理基础之上，因此，Spring对AOP的支持局限于**方法拦截**。如果你的AOP需求超过了简单的方法调用（如构造器或属性拦截），那么你需要考虑使用AspectJ来实现切面。

### AOP术语

#### 通知（Advice）

切面的工作被称为通知,通知定义了切面是什么以及何时使用。

Spring切面可以应用5种类型的通知：

- 前置通知（Before）：在目标方法被调用之前调用通知功能；
- 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
- 返回通知（After-returning）：在目标方法成功执行之后调用通知；
- 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
- 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

#### 连接点（Join point）

连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

#### 切点（Poincut）

如果说通知定义了切面的“什么”和“何时”的话，那么切点就定义了“何处”。切点的定义会匹配通知所要织入的一个或多个连接点。一个切面并不需要通知应用的所有连接点。切点有助于缩小切面所通知的连接点的范围。

我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。

#### 切面（Aspect）

切面是通知和切点的结合。通知和切点共同定义了切面的全部内容——它是什么，在何时和何处完成其功能。

#### 引入（Introduction）

引入允许我们向现有的类添加新方法或属性。例如，我们可以创建一个Auditable通知类，该类记录了对象最后一次修改时的状态。这很简单，只需一个方法，setLastModified(Date)，和一个实例变量来保存这个状态。然后，这个新方法和实例变量就可以被引入到现有的类中，从而可以在无需修改这些现有的类的情况下，让它们具有新的行为和状态。

#### 织入（Weaving）

织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。

在目标对象的生命周期里有多个点可以进行织入：

- 编译期：切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ的织入编译器就是以这种方式织入切面的。
- 类加载期：切面在目标类加载到JVM时被织入。这种方式需要特殊的类加载器（ClassLoader），它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5的加载时织入（load-timeweaving，LTW）就支持以这种方式织入切面。
- 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。Spring AOP就是以这种方式织入切面的。

### Spring AOP

代理类封装了目标类，并拦截被通知方法的调用，再把调用转发给真正的目标bean。当代理拦截到方法调用时，在调用目标bean方法之前，会执行切面逻辑。

因为Spring基于动态代理，所以Spring只支持方法连接点。这与一些其他的AOP框架是不同的，例如AspectJ和JBoss，除了方法切点，它们还提供了字段和构造器接入点。Spring缺少对字段连接点的支持，无法让我们创建细粒度的通知，例如拦截对象字段的修改。而且它不支持构造器连接点，我们就无法在bean创建时应用通知。

#### 关于环绕通知

环绕通知是最为强大的通知类型。它能够让你所编写的逻辑将被通知的目标方法完全包装起来。实际上就像在一个通知方法中同时编写前置通知和后置通知。

关于这个新的通知方法，它接受`ProceedingJoinPoint`作为参数。这个对象是必须要有的，因为你要在通知中**通过它来调用被通知的方法**。通知方法中可以做任何的事情，当要将控制权交给被通知的方法时，它需要调用`ProceedingJoinPoint`的`proceed()`方法。需要注意的是，别忘记调用`proceed()`方法。如果不调这个方法的话，那么你的通知实际上会阻塞对被通知方法的调用。有可能这就是你想要的效果，但更多的情况是你希望在某个点上执行被通知的方法。

有意思的是，你可以不调用`proceed()`方法，从而阻塞对被通知方法的访问，与之类似，你也可以在通知中对它进行**多次**调用。要这样做的一个场景就是实现重试逻辑，也就是在被通知方法失败后，进行重复尝试。

## SpringMVC

![](D:\books\Import\java_base\assets\spring\mvc\mvc.png)

[^]: springMVC调用流程

1. DispatcherServlet:`Spring MVC所有的请求都会通过一个前端控制器（front controller）Servlet。前端控制器是常用的Web应用程序模式，在这里一个单实例的Servlet将请求委托给应用程序的其他组件来执行实际的处理。任务是将请求发送给Spring MVC控制器（controller）。
2. `controller`控制器是一个用于处理请求的Spring组件。在典型的应用程序中可能会有多个控制器，DispatcherServlet需要知道应该将请求发送给哪个控制器。所以DispatcherServlet以会查询一个或多个处理器映射（handler mapping） 来确定请求的下一站在哪里。处理器映射会根据请求所携带的URL信息来进行决策。
3. 一旦选择了合适的控制器，DispatcherServlet会将请求发送给选中的控制器。控制器将业务逻辑委托给一个或多个服务对象进行处理。
4. 在完成逻辑处理后，通常会产生一些信息，这些信息需要返回给用户并在浏览器上显示。这些信息被称为**模型**（model）。不过仅仅给用户返回原始的信息是不够的——这些信息需要以用户友好的方式进行格式化，一般会是HTML。所以，信息需要发送给一个**视图**（view），通常会是JSP。控制器所做的最后一件事就是将模型数据打包，并且标示出用于渲染输出的视图名。它接下来会将请求连同模型和视图名发送回DispatcherServlet
5. DispatcherServlet将会使用**视图解析器**（view resolver）来将逻辑视图名匹配为一个特定的视图实现。
6. 既然DispatcherServlet已经知道由哪个视图渲染结果，那请求的任务基本上也就完成了。它的最后一站是视图的实现（可能是JSP） ，在这里它交付模型数据。请求的任务就完成了。视图将使用模型数据渲染输出，这个输出会通过响应对象传递给客户端。

## Spring Security

Spring Security是为基于Spring的应用程序提供声明式安全保护的安全性框架。Spring Security提供了完整的安全性解决方案，它能够在Web请求级别和方法调用级别处理身份认证和授权。因为基于Spring框架，所以Spring Security充分利用了依赖注入（dependency injection，DI）和面向切面的技术。

## Spring事务

所谓的事物管理指的是**“按照指定事物规则执行提交或者回滚操作”**，而**spring的事物管理，指的是spring对事物管理规则的抽象**。具体来说就是spring不直接管理事物，而是为了不同的平台(如jdbc hibernate jpa 等)提供了不同的事物管理器 ，将具体的事物管理委托给相应的平台实现，spring并不关心具体事物管理的实现，从而为事物管理提供了通用编程模型。

### spring事务管理器

#### jdbc事物管理器

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
 </bean>
```

#### hibernate事物管理器

```xml
<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
```

#### 其它事务管理器

`org.springframework.orm.jpa`，`org.springframework.transaction.jta.JtaTransactionManager`，`org.springframework.jms.connection.JmsTransactionManager`

### spring 事物的基本行为

事物的相关属性就定义在java.sql.Connection.TransactionDefinition类中

```java
 	/**
     * 获取传播行为
     */
    int getPropagationBehavior();

    /**
     * 获取隔离级别
     */
    int getIsolationLevel();

    /**
     * 获取超时时间
     */
    int getTimeout();

    /**
     * 获取只读属性
     */
    boolean isReadOnly();

    /**
     * 获取事物的名称 默认为classname+"."+methodName
     */
    String getName();
```

#### 事物的传播行为

所谓事物的传播行为，指的是被事物标记的方法（注解或者xml声明），之间进行嵌套调用时事物的传播规则。拿jdbc事物管理器来说，就是共用同一个jdbc connection，还是新建connection，还是抛异常。这个规则就叫做事物的传播行为。


| 传播行为                  | 说明                                                         |                                                              |
| :------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 如果存在事务，则加入当前事务。如果不存在事务，则新建。       | 默认传播行为                                                 |
| PROPAGATION_SUPPORTS      | 支持事务，如果不存在事务则以非事务的状态运行。               |                                                              |
| PROPAGATION_MANDATORY     | 必须存在事务，不存在抛异常。                                 |                                                              |
| PROPAGATION_NEVER         | 不支持事务，如当前存在事务则抛异常。                         |                                                              |
| PROPAGATION_REQUIRES_NEW  | 无论当前是否存在事务，都新建事务。                           | 仅支持JtaTransactionManager作为事务管理器。                  |
| PROPAGATION_NOT_SUPPORTED | 不支持事务，如当前存在事务则将当前事物挂起，以非事务的方式运行 | 仅支持JtaTransactionManager作为事务管理器。                  |
| PROPAGATION_NESTED        | 嵌套事务。如果当前不存在事务以PROPAGATION_REQUIRED的方式运行。如果存在事务则以嵌套事务的方式运行。 | 仅支持DataSourceTransactionManager作为事务管理器和部分JTA事务管理器 |

`关于PROPAGATION_NESTED`

```java
@Transactional(propagation = Propagation.REQUIRED)
    void methodA(){
        //doSomethingA()
        ((ServiceName)AopContext.currentProxy()).methodB();
    }

    @Transactional(propagation = Propagation.NESTED)
    void methodB(){
        //doSomethingB()
        //successOrNot
    }
```

单独调用methodB 时，相当于 PROPAGATION_REQUIRED。当调用methodA时，则以嵌套事物的方式运行，methodB作为methodA的子事物，提交和回滚都会受methodA的事物的影响。

一般我们用PROPAGATION_NESTED来执行分支逻辑。当methodB执行失败的时候，则methodA 回滚到之前的保存点，然后执行catch块中的其它业务逻辑，就像methodB从未执行过一样。这也是PROPAGATION_NESTED最常用的用法。

#### 事物的隔离级别

事物的隔离级别，指的是并发情况下，事物之间的隔离度。不同的隔离级别，可以解决不同的隔离问题，也对应着不同的并发度，使用的时候，要结合实际情况，按需选择。

| 问题       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| 脏读       | 事物A读到了事物B未提交的数据，如果事物B的操作被回滚了，那么事物A读到的就是无效数据。 |
| 不可重复读 | 事物A执行select 查询到结果集R1，此时事物B执行`update` 并提交，事物A再执行同样的select 查询到结果集R2。两次查询到的结果集不一致。 |
| 幻读       | 事物A执行select 查询到结果集R1，此事事物B执行了 `insert 或者 delete` 操作 并提交，事物A再执行同样的select 查询到结果集R2。此时发现R2相比R1多了或者少了一些记录 |

spring事务隔离级别



| 级别                       | 描述                                                         | 脏读 | 不可重复读 | 幻读 |
| -------------------------- | ------------------------------------------------------------ | :--: | :--------: | :--: |
| ISOLATION_DEFAULT          | 使用数据库隔离级别。                                         |      |            |      |
| ISOLATION_READ_UNCOMMITTED | 读未提交                                                     |  X   |     X      |  X   |
| ISOLATION_READ_COMMITTED   | 读已提交`oracle 默认隔离级别，利用快照读解决脏读`            |  Y   |     X      |  X   |
| ISOLATION_REPEATABLE_READ  | 可重复读`mysql 默认隔离级别，除了读快照之外，事物启动后不允许执行update操作` |  Y   |     Y      |  X   |
| ISOLATION_SERIALIZABLE     | 串行化                                                       |  Y   |     Y      |  Y   |

#### 事物的只读属性-readOnly

spring 事物的只读属性是通过设置，java.sql.Connection 的 readOnly 属性来实现的。当设置了只读属性后，数据库会对只读事物进行一些优化，如不启用回滚段，不启用回滚日志等。

#### 事物的超时时间-timeout

事物的超时指的是设置一个时间，当执行时间超过这个时间后，抛异常并回滚。是通过设置一个截至时间来实现的，值得注意的是，`这个超时时间只有特定情况下才会生效，如dao层使用jdbcTemplete 执行sql`

#### 回滚规则-rollbackFor

回滚规则只得是spring 事物遇到哪种异常会回滚。默认只回滚RuntimeException和Error