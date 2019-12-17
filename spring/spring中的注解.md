## spring

###### @Component("ID")

等同于java注解`@Named`，用于类的注解，表明该类会作为组件类，并告知Spring要为这个类创建bean。Spring会根据类名为其指定一个ID。具体来讲，这个bean所给定的ID为将类名的第一个字母变为小写。获取通过参数来设置类的名字(ID)。

###### @ComponentScan(basePackages={"system","video"})

和@configuration一起使用，这个注解能够在Spring中启用组件扫描。，@ComponentScan默认会扫描与配置类相同的包。Spring将会扫描这个包以及这个包下的所有子包，查找带有@Component注解的类。并且会在Spring中自动为其创建一个bean。也可以通过指定特定的类。

```java
@ComponentScan(basePackageClasses={Player.class,Video.class})
```

###### @Autowired(requierd=false)

`@Autowired`按照byType自动注入。等同于java注解`@Inject`，`@Autowired`注解不仅能够用在构造器上，还能用在属性的Setter方法和成员变量上。将required属性设置为false时，Spring会尝试执行自动装配，但是如果没有匹配的bean的话，Spring将会让这个bean处于未装配的状态。但是，把required属性设置为false时，你需要谨慎对待。如果在你的代码中没有进行null检查的话，这个处于未装配状态的属性有可能会出现NullPointerException。

###### @Resource

@Resource默认按照ByName自动注入，由J2EE提供。`@Resource`有两个重要的属性：name和type，而Spring将`@Resource`注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。

###### @Configuration

表明这个类是一个配置类，该类应该包含在Spring应用上下文中如何创建bean的细节。

###### @Bean

```java
@Configuration
public class CdPlayerConfig{
    
    @Bean(name="sgt")
	public	CompactDisc sgtPeppers(){
		return new SgtPeppers();
	}
}

```

@Bean注解会告诉Spring这个方法将会返回一个对象，该对象要注册为Spring应用上下文中的bean。方法体中包含了最终产生bean实例的逻辑。默认情况下，bean的ID与带有`@Bean`注解的方法名是一样的.

###### @Profile

`@Profile`注解应用在了类级别上。它会告诉Spring这个配置类中的bean只有在`dev profile`激活时才会创建。如果`dev profile`没有激活的话，那么带有`@Bean`注解的方法都会被忽略掉。

###### @Primary

当spring装配匹配到多个类时，可以通过@Primary来表达最喜欢的方案。@Primary能够与@Component组合用在组件扫描的bean上，也可以与@Bean组合用在Java配置的bean声明中。

###### @Qualifier("beanID")

`@Qualifier`注解是使用限定符的主要方式。它可以与`@Autowired`和`@Inject`协同使用，在注入的时候指定想要注入进去的是哪个bean。

###### @Scope

设置bean的作用域。

###### <u>*@DeclareParents*</u>

利用被称为引入的AOP概念，切面可以为Spring bean添加新方法。

```java
@Aspect
@Component
public class AspectConfig {
    //"+"表示person的所有子类；defaultImpl 表示默认需要添加的新的类
    @DeclareParents(value = "com.annotation.Person+", defaultImpl = FemaleAnimal.class)
    public Animal animal;
}
```

`@DeclareParents`注解由三部分组成：

- value属性指定了哪种类型的bean要引入该接口。在本例中，也就是所有实现Person的类型。（标记符后面的加号表示是Person的所有子类型，而不是Person本身。）
- defaultImpl属性指定了为引入功能提供实现的类。在这里，我们指定的是FemaleAnimal提供实现。
- `@DeclareParents`注解所标注的静态属性指明了要引入了接口。在这里，我们所引入的是animal接口。

## SpringMVC

###### @Controller

通常是被使用服务于web 页面的。默认，你的controller方法返回的是一个string 串，是表示要展示哪个模板页面或者是要跳转到哪里去。它基于`@Component`注解。

###### **@RestController**

是`@ResponseBody`和`@Controller`的组合注解。返回的应该是一个对象。

###### @RequestMapping

RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

RequestMapping注解有六个属性:

- value：指定请求的实际地址，指定的地址可以是URI Template 模式
- method：指定请求的method类型， GET、POST、PUT、DELETE等；
- consumes：指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
- produces：指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；
- params：指定request中必须包含某些参数值是，才让该方法处理。
- headers：指定request中必须包含某些指定的header值，才能让该方法处理请求。

###### @requestParam

主要用于在SpringMVC后台控制层获取参数，类似`name=request.getParameter("name")`。

有三个常用参数：

- defaultValue：表示默认值
- required：设置boolean值表示是否为必须
- value：用来将前端jsp页面中的传入参数与SpringMVC控制层接收参数名做映射。如jsp中<input name="id">，接收参数名为bookId，那么对于这样的参数即可使用`@requestParam(value=“id”)`bookId进行映射既可成功接收。

###### **@PathVariable**

用于将请求URL中的模板变量映射到功能处理方法的参数上，即取出uri模板中的变量作为参数。

###### *@ModelAttribute:

用于数据回显，springMvc默认回显pojo数据，以pojo的类名首字母小写作为key，存入request