[toc]

## 4. The IoC container

### 4.1 Spring IoC容器和beans

Spring IoC容器的基础是`org.springframework.beans`和`org.springframework.context`。

### 4.2 容器概述

`org.springframework.context.ApplicationContext`表示Spring IoC容器，负责管理beans。

Spring自带的`ApplicationContext`实现：In standalone applications it is common to create an instance of `ClassPathXmlApplicationContext` or `FileSystemXmlApplicationContext`.

#### 4.2.1 配置元数据

配置元数据传统上是XML格式。还支持基于注解的配置元数据，以及基于Java的配置。

The following example shows the basic structure of XML-based configuration metadata:

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://www.springframework.org/schema/beans
	        http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	    <bean id="..." class="...">
	        <!-- collaborators and configuration for this bean go here -->
	    </bean>
	
	    <bean id="..." class="...">
	        <!-- collaborators and configuration for this bean go here -->
	    </bean>
	
	    <!-- more bean definitions go here -->
	
	</beans>
	
#### 4.2.2 实例化容器

	ApplicationContext context =
	    new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});

#### 组合XML配置元数据

It can be useful to have bean definitions span multiple XML files. Often each individual XML configuration file represents a logical layer or module in your architecture.

You can use the application context constructor to load bean definitions from all these XML fragments. This constructor takes multiple Resource locations, as was shown in the previous section. Alternatively, use one or more occurrences of the `<import/>` element to load bean definitions from another file or files. For example:

	<beans>
	    <import resource="services.xml"/>
	    <import resource="resources/messageSource.xml"/>
	    <import resource="/resources/themeSource.xml"/>
	
	    <bean id="bean1" class="..."/>
	    <bean id="bean2" class="..."/>
	</beans>

> You can always use fully qualified resource locations instead of relative paths: for example, "file:C:/config/services.xml" or "classpath:/config/services.xml". However, be aware that you are coupling your application’s configuration to specific absolute locations. It is generally preferable to keep an indirection for such absolute locations, for example, through "${…}" placeholders that are resolved against JVM system properties at runtime.

#### 4.2.3 使用容器

	// create and configure beans
	ApplicationContext context =
	    new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});
	
	// retrieve configured instance
	PetStoreService service = context.getBean("petStore", PetStoreService.class);
	
	// use configured instance
	List<String> userList = service.getUsernameList();

### （未）4.3 Bean概述

### （未）4.4 依赖

### （未）4.5. Bean作用域

### （未）4.6. Customizing the nature of a bean ###

### （未）4.7. Bean definition inheritance ###

### （未）4.8. Container Extension Points ###

### （未）4.9 基于注解的容器配置

### （未）4.10. Classpath scanning and managed components ###

### （未）4.11. Using JSR 330 Standard Annotations ###

### 4.12 基于Java的容器配置

#### 4.12.1 基本概念：@Bean 和 @Configuration

关键点是`@Configuration`注解的类和`@Bean`注解的方法。

`@Bean`扮演`<bean/>`的角色。You can use @Bean annotated methods with any Spring @Component, however, they are most often used with @Configuration beans.

`@Configuration`注解类表示这个类的主要目的是定义beans。Furthermore, @Configuration classes allow inter-bean dependencies to be defined by simply calling other @Bean methods in the same class. The simplest possible @Configuration class would read as follows:

	@Configuration
	public class AppConfig {
	    @Bean
	    public MyService myService() {
	        return new MyServiceImpl();
	    }
	}

等价的XML：

	<beans>
	    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
	</beans>

#### 4.12.2 使用AnnotationConfigApplicationContext实例化Spring容器

`AnnotationConfigApplicationContext`不仅能接受`@Configuration`类，也能接受普通`@Component`类和注解 JSR-330 metadata 的类。

当传入一个`@Configuration`类时，`@Configuration`类自身会被注册为一个bean定义，其中所有`@Bean`方法也会被注册成bean定义。

若提供`@Component`或 JSR-330 类，他们自身被注册为bean定义。And it is assumed that DI metadata such as @Autowired or @Inject are used within those classes where necessary.

**Full @Configuration vs lite @Beans mode?**

When @Bean methods are declared within classes that are not annotated with @Configuration they are referred to as being processed in a lite mode. For example, bean methods declared in a @Component or even in a plain old class will be considered lite.

Unlike full @Configuration, lite @Bean methods cannot easily declare inter-bean dependencies. Usually one @Bean method should not invoke another @Bean method when operating in lite mode.

Only using @Bean methods within @Configuration classes is a recommended approach of ensuring that full mode is always used. This will prevent the same @Bean method from accidentally being invoked multiple times and helps to reduce subtle bugs that can be hard to track down when operating in lite mode.


**Simple construction**

传入`@Configuration`类

	public static void main(String[] args) {
	    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
	    MyService myService = ctx.getBean(MyService.class);
	    myService.doStuff();
	}

传入`@Component`或 JSR-330 注解的类：

	public static void main(String[] args) {
	    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
	    MyService myService = ctx.getBean(MyService.class);
	    myService.doStuff();
	}

**使用register(Class<?>…)构造容器**

可以先用无参构造器实例化`AnnotationConfigApplicationContext`，然后使用`register()`方法配置。

	public static void main(String[] args) {
	    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
	    ctx.register(AppConfig.class, OtherConfig.class);
	    ctx.register(AdditionalConfig.class);
	    ctx.refresh();
	    MyService myService = ctx.getBean(MyService.class);
	    myService.doStuff();
	}

**启用组件扫描：scan(String…)**

扫描`com.acme`包下注解`@Component`的类，将它们注册为beans：

	public static void main(String[] args) {
	    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
	    ctx.scan("com.acme");
	    ctx.refresh();
	    MyService myService = ctx.getBean(MyService.class);
	}

等价的XML：

	<beans>
	    <context:component-scan base-package="com.acme"/>
	</beans>

> Remember that @Configuration classes are meta-annotated with @Component, so they are candidates for component-scanning! In the example above, assuming that AppConfig is declared within the com.acme package (or any package underneath), it will be picked up during the call to scan(), and upon refresh() all its @Bean methods will be processed and registered as bean definitions within the container.

**Web应用使用AnnotationConfigWebApplicationContext**

This implementation may be used when configuring the Spring `ContextLoaderListener` servlet listener, Spring MVC `DispatcherServlet`, etc. What follows is a web.xml snippet that configures a typical Spring MVC web application. Note the use of the contextClass context-param and init-param:

	<web-app>
	    <!-- 配置ContextLoaderListener使用AnnotationConfigWebApplicationContext
	        。默认是XmlWebApplicationContext -->
	    <context-param>
	        <param-name>contextClass</param-name>
	        <param-value>
	            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
	        </param-value>
	    </context-param>

	    <!-- 逗号或空格分隔的@Configuration类的全限名。或者全限的包名用于组件扫描 -->
	    <context-param>
	        <param-name>contextConfigLocation</param-name>
	        <param-value>com.acme.AppConfig</param-value>
	    </context-param>

	    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
	    <listener>
	        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	    </listener>

	    <!-- Declare a Spring MVC DispatcherServlet as usual -->
	    <servlet>
	        <servlet-name>dispatcher</servlet-name>
	        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	        <!-- 配置DispatcherServlet使用AnnotationConfigWebApplicationContext，而不是默认的XmlWebApplicationContext -->
	        <init-param>
	            <param-name>contextClass</param-name>
	            <param-value>
	                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
	            </param-value>
	        </init-param>
	        <!-- 逗号风格或空格分隔的@Configuration类 -->
	        <init-param>
	            <param-name>contextConfigLocation</param-name>
	            <param-value>com.acme.web.MvcConfig</param-value>
	        </init-param>
	    </servlet>
	
	    <!-- map all requests for /app/* to the dispatcher servlet -->
	    <servlet-mapping>
	        <servlet-name>dispatcher</servlet-name>
	        <url-pattern>/app/*</url-pattern>
	    </servlet-mapping>
	</web-app>

#### 4.12.3 使用@Bean注解

@Bean是方法级别的注解，与`<bean/>`等价。

You can use the @Bean annotation in a `@Configuration`-annotated or in a `@Component`-annotated class.

##### 声明一个bean

要声明一个bean，对方法注解@Bean。Bean类型是方法的返回类型。Bean名默认是方法名。

	@Configuration
	public class AppConfig {
	
	    @Bean
	    public TransferService transferService() {
	        return new TransferServiceImpl();
	    }
	
	}
	

##### 接收生命周期回调事件

Any classes defined with the @Bean annotation support the regular lifecycle callbacks and can use the @PostConstruct and @PreDestroy annotations from JSR-250, see JSR-250 annotations for further details.

The regular Spring lifecycle callbacks are fully supported as well. If a bean implements `InitializingBean`, `DisposableBean`, or `Lifecycle`, their respective methods are called by the container.

The standard set of `*Aware` interfaces such as `BeanFactoryAware`, `BeanNameAware`, `MessageSourceAware`, `ApplicationContextAware`, and so on are also fully supported.

The `@Bean` annotation supports specifying arbitrary initialization and destruction callback methods, much like Spring XML’s init-method and destroy-method attributes on the bean element:

	public class Foo {
	    public void init() {
	        // initialization logic
	    }
	}
	
	public class Bar {
	    public void cleanup() {
	        // destruction logic
	    }
	}
	
	@Configuration
	public class AppConfig {
	
	    @Bean(initMethod = "init")
	    public Foo foo() {
	        return new Foo();
	    }
	
	    @Bean(destroyMethod = "cleanup")
	    public Bar bar() {
	        return new Bar();
	    }
	
	}

但为何不在构建时直接调用初始化方法：

	@Configuration
	public class AppConfig {
	    @Bean
	    public Foo foo() {
	        Foo foo = new Foo();
	        foo.init();
	    	return foo;
	    }
	
	    // ...
	
	}

##### 指定Bean的作用域

**利用@Scope注解**

默认作用域是singleton，可以通过`@Scope`注解覆盖：

	@Configuration
	public class MyConfiguration {
	
	    @Bean
	    @Scope("prototype")
	    public Encryptor encryptor() {
	        // ...
	    }
	
	}

**@Scope 与 scoped-proxy**

Spring offers a convenient way of working with scoped dependencies through scoped proxies. The easiest way to create such a proxy when using the XML configuration is the `<aop:scoped-proxy/>` element. Configuring your beans in Java with a `@Scope` annotation offers equivalent support with the `proxyMode` attribute. The default is no proxy ( `ScopedProxyMode.NO`), but you can specify `ScopedProxyMode.TARGET_CLASS` or `ScopedProxyMode.INTERFACES`.

If you port the scoped proxy example from the XML reference documentation (see preceding link) to our @Bean using Java, it would look like the following:

	// an HTTP Session-scoped bean exposed as a proxy
	@Bean
	@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
	public UserPreferences userPreferences() {
	    return new UserPreferences();
	}
	
	@Bean
	public Service userService() {
	    UserService service = new SimpleUserService();
	    // a reference to the proxied userPreferences bean
	    service.setUserPreferences(userPreferences());
	    return service;
	}

##### Bean的命名

默认Bean名是方法名。可以通过`name`特性覆盖：

	@Configuration
	public class AppConfig {
	
	    @Bean(name = "myFoo")
	    public Foo foo() {
	        return new Foo();
	    }
	
	}

**Bean的别名**

Bean可以有多个名字，称为别名。The `name` attribute of the `@Bean` annotation accepts a String array for this purpose.

	@Configuration
	public class AppConfig {
	
	    @Bean(name = { "dataSource", "subsystemA-dataSource", "subsystemB-dataSource" })
	    public DataSource dataSource() {
	        // instantiate, configure and return DataSource bean...
	    }
	
	}

**Bean description**

This can be particularly useful when beans are exposed (perhaps via JMX) for monitoring purposes.

To add a description to a @Bean the @Description annotation can be used:
	
	@Configuration
	public class AppConfig {
	
	    @Bean
	    @Desciption("Provides a basic example of a bean")
	    public Foo foo() {
	        return new Foo();
	    }
	
	}

#### 4.12.4 使用@Configuration注解

Calls to @Bean methods on @Configuration classes can also be used to define inter-bean dependencies.

##### 注解跨Bean的依赖

When @Beans have dependencies on one another, expressing that dependency is as simple as having one bean method call another:

	@Configuration
	public class AppConfig {
	
	    @Bean
	    public Foo foo() {
	        return new Foo(bar());
	    }
	
	    @Bean
	    public Bar bar() {
	        return new Bar();
	    }
	
	}

In the example above, the foo bean receives a reference to bar via constructor injection.

> This method of declaring inter-bean dependencies only works when the @Bean method is declared within a `@Configuration` class. You cannot declare inter-bean dependencies using plain `@Component` classes.

##### Lookup method injection

As noted earlier, lookup method injection is an advanced feature that you should use rarely. It is useful in cases where a singleton-scoped bean has a dependency on a prototype-scoped bean. Using Java for this type of configuration provides a natural means for implementing this pattern.

	public abstract class CommandManager {
	    public Object process(Object commandState) {
	        // grab a new instance of the appropriate Command interface
	        Command command = createCommand();
	
	        // set the state on the (hopefully brand new) Command instance
	        command.setState(commandState);
	    return command.execute();
	    }
	
	    // okay... but where is the implementation of this method?
	    protected abstract Command createCommand();
	}

Using Java-configuration support , you can create a subclass of `CommandManager` where the abstract `createCommand()` method is overridden in such a way that it looks up a new (prototype) command object:

	@Bean
	@Scope("prototype")
	public AsyncCommand asyncCommand() {
	    AsyncCommand command = new AsyncCommand();
	    // inject dependencies here as required
	    return command;
	}
	
	@Bean
	public CommandManager commandManager() {
	    // return new anonymous implementation of CommandManager with command() overridden
	    // to return a new prototype Command object
	    return new CommandManager() {
	        protected Command createCommand() {
	            return asyncCommand();
	        }
	    }
	}

##### Further information about how Java-based configuration works internally

The following example shows a @Bean annotated method being called twice:

	@Configuration
	public class AppConfig {
	
	    @Bean
	    public ClientService clientService1() {
	        ClientServiceImpl clientService = new ClientServiceImpl();
	        clientService.setClientDao(clientDao());
	        return clientService;
	    }
	
	    @Bean
	    public ClientService clientService2() {
	        ClientServiceImpl clientService = new ClientServiceImpl();
	        clientService.setClientDao(clientDao());
	        return clientService;
	    }
	
	    @Bean
	    public ClientDao clientDao() {
	        return new ClientDaoImpl();
	    }
	
	}

clientDao() has been called once in clientService1() and once in clientService2(). Since this method creates a new instance of ClientDaoImpl and returns it, you would normally expect having 2 instances (one for each service). That definitely would be problematic: in Spring, instantiated beans have a **singleton** scope by default. This is where the magic comes in: All @Configuration classes are subclassed at startup-time with **CGLIB**. In the subclass, the child method checks the container first for any cached (scoped) beans before it calls the parent method and creates a new instance. 注意到从Spring 3.2开始，不再需要添加CGLIB到类路径。因为CGLIB类已被重新打包到org.springframework包下。

There are a few restrictions due to the fact that CGLIB dynamically adds features at startup-time:

- Configuration classes should not be `final`
- They should have a constructor with no arguments 

#### 4.12.5 组合基于Java的配置

##### 使用@Import注解

Much as the `<import/>` element is used within Spring XML files to aid in modularizing configurations, the @Import annotation allows for loading @Bean definitions from another configuration class:
	
	@Configuration
	public class ConfigA {
	
	    @Bean
	    public A a() {
	        return new A();
	    }
	
	}
	
	@Configuration
	@Import(ConfigA.class)
	public class ConfigB {
	
	    @Bean
	    public B b() {
	        return new B();
	    }
	
	}

Now, rather than needing to specify both ConfigA.class and ConfigB.class when instantiating the context, only ConfigB needs to be supplied explicitly:

	public static void main(String[] args) {
	    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);
	
	    // now both beans A and B will be available...
	    A a = ctx.getBean(A.class);
	    B b = ctx.getBean(B.class);
	}

This approach simplifies container instantiation, as only one class needs to be dealt with, rather than requiring the developer to remember a potentially large number of @Configuration classes during construction.

##### Injecting dependencies on imported @Bean definitions

The example above works, but is simplistic. In most practical scenarios, beans will have dependencies on one another across configuration classes. When using XML, this is not an issue, per se, because there is no compiler involved, and one can simply declare ref="someBean" and trust that Spring will work it out during container initialization. Of course, when using @Configuration classes, the Java compiler places constraints on the configuration model, in that references to other beans must be valid Java syntax.

Fortunately, solving this problem is simple. Remember that @Configuration classes are ultimately just another bean in the container - this means that they can take advantage of @Autowired injection metadata just like any other bean!

Let’s consider a more real-world scenario with several @Configuration classes, each depending on beans declared in the others:

	@Configuration
	public class ServiceConfig {
	
	    @Autowired
	    private AccountRepository accountRepository;
	
	    @Bean
	    public TransferService transferService() {
	        return new TransferServiceImpl(accountRepository);
	    }
	
	}
	
	@Configuration
	public class RepositoryConfig {
	
	    @Autowired
	    private DataSource dataSource;
	
	    @Bean
	    public AccountRepository accountRepository() {
	        return new JdbcAccountRepository(dataSource);
	    }
	
	}
	
	@Configuration
	@Import({ServiceConfig.class, RepositoryConfig.class})
	public class SystemTestConfig {
	
	    @Bean
	    public DataSource dataSource() {
	        // return new DataSource
	    }
	
	}
	
	public static void main(String[] args) {
	    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
	    // everything wires up across configuration classes...
	    TransferService transferService = ctx.getBean(TransferService.class);
	    transferService.transfer(100.00, "A123", "C456");
	}

In the scenario above, using @Autowired works well and provides the desired modularity, but determining exactly where the autowired bean definitions are declared is still somewhat ambiguous. For example, as a developer looking at ServiceConfig, how do you know exactly where the @Autowired AccountRepository bean is declared? It’s not explicit in the code, and this may be just fine. Remember that the SpringSource Tool Suite provides tooling that can render graphs showing how everything is wired up - that may be all you need. Also, your Java IDE can easily find all declarations and uses of the AccountRepository type, and will quickly show you the location of @Bean methods that return that type.

In cases where this ambiguity is not acceptable and you wish to have direct navigation from within your IDE from one @Configuration class to another, consider autowiring the configuration classes themselves:

	@Configuration
	public class ServiceConfig {
	
	    @Autowired
	    private RepositoryConfig repositoryConfig;
	
	    @Bean
	    public TransferService transferService() {
	        // navigate through the config class to the @Bean method!
	        return new TransferServiceImpl(repositoryConfig.accountRepository());
	    }
	
	}

In the situation above, it is completely explicit where AccountRepository is defined. However, ServiceConfig is now tightly coupled to RepositoryConfig; that’s the tradeoff. This tight coupling can be somewhat mitigated by using interface-based or abstract class-based @Configuration classes. Consider the following:

	@Configuration
	public class ServiceConfig {
	
	    @Autowired
	    private RepositoryConfig repositoryConfig;
	
	    @Bean
	    public TransferService transferService() {
	        return new TransferServiceImpl(repositoryConfig.accountRepository());
	    }
	}
	
	@Configuration
	public interface RepositoryConfig {
	
	    @Bean
	    AccountRepository accountRepository();
	
	}
	
	@Configuration
	public class DefaultRepositoryConfig implements RepositoryConfig {
	
	    @Bean
	    public AccountRepository accountRepository() {
	        return new JdbcAccountRepository(...);
	    }
	
	}
	
	@Configuration
	@Import({ServiceConfig.class, DefaultRepositoryConfig.class}) // import the concrete config!
	public class SystemTestConfig {
	
	    @Bean
	    public DataSource dataSource() {
	        // return DataSource
	    }
	
	}
	
	public static void main(String[] args) {
	    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
	    TransferService transferService = ctx.getBean(TransferService.class);
	    transferService.transfer(100.00, "A123", "C456");
	}

Now ServiceConfig is loosely coupled with respect to the concrete DefaultRepositoryConfig, and built-in IDE tooling is still useful: it will be easy for the developer to get a type hierarchy of RepositoryConfig implementations. In this way, navigating @Configuration classes and their dependencies becomes no different than the usual process of navigating interface-based code.

##### Conditionally including @Configuration classes or @Beans

It is often useful to conditionally enable to disable a complete @Configuration class, or even individual @Bean methods, based on some arbitrary system state. One common example of this it to use the @Profile annotation to active beans only when a specific profile has been enabled in the Spring Environment (see Section 4.13, “Bean definition profiles and environment abstraction” for details).

The @Profile annotation is actually implemented using a much more flexible annotation called @Conditional. The @Conditional annotation indicates specific org.springframework.context.annotation.Condition implementations that should be consulted before a @Bean is registered.

Implementations of the Condition interface simply provide a matches(...) method that returns true or false. For example, here is the actual Condition implementation used for @Profile:

	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
	    if (context.getEnvironment() != null) {
	        // Read the @Profile annotation attributes
	        MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
	        if (attrs != null) {
	            for (Object value : attrs.get("value")) {
	                if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
	                    return true;
	                }
	            }
	            return false;
	        }
	    }
	    return true;
	}

See the @Conditional javadocs for more detail.

##### 组合Java和XML配置

Some facilities such as Spring XML namespaces remain an ideal way to configure the container. In cases where XML is convenient or necessary, you have a choice: either instantiate the container in an "XML-centric" way using, for example, `ClassPathXmlApplicationContext`, or in a "Java-centric" fashion using `AnnotationConfigApplicationContext` and the `@ImportResource `annotation to import XML as needed.

**XML为中心时使用@Configuration类**

It may be preferable to bootstrap the Spring container from XML and include @Configuration classes in an ad-hoc fashion.

Remember that @Configuration classes are ultimately just bean definitions in the container. In this example, we create a @Configuration class named `AppConfig` and include it within **system-test-config.xml** as a `<bean/>` definition. Because `<context:annotation-config/>` is switched on, the container will recognize the `@Configuration` annotation, and process the `@Bean` methods declared in `AppConfig` properly.
	
	@Configuration
	public class AppConfig {
	
	    @Autowired
	    private DataSource dataSource;
	
	    @Bean
	    public AccountRepository accountRepository() {
	        return new JdbcAccountRepository(dataSource);
	    }
	
	    @Bean
	    public TransferService transferService() {
	        return new TransferService(accountRepository());
	    }
	
	}

system-test-config.xml

	<beans>
	    <!-- enable processing of annotations such as @Autowired and @Configuration -->
	    <context:annotation-config/>
	    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
	
	    <bean class="com.acme.AppConfig"/>
	
	    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	        <property name="url" value="${jdbc.url}"/>
	        <property name="username" value="${jdbc.username}"/>
	        <property name="password" value="${jdbc.password}"/>
	    </bean>
	</beans>

	jdbc.properties
	jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
	jdbc.username=sa
	jdbc.password=

	public static void main(String[] args) {
	    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
	    TransferService transferService = ctx.getBean(TransferService.class);
	    // ...
	}

Because @Configuration is meta-annotated with @Component, @Configuration-annotated classes are automatically candidates for component scanning. Using the same scenario as above, we can redefine system-test-config.xml to take advantage of component-scanning. Note that in this case, we don’t need to explicitly declare `<context:annotation-config/>`, because `<context:component-scan/>` enables all the same functionality.

system-test-config.xml
	
	<beans>
	    <!-- picks up and registers AppConfig as a bean definition -->
	    <context:component-scan base-package="com.acme"/>
	    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
	
	    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	        <property name="url" value="${jdbc.url}"/>
	        <property name="username" value="${jdbc.username}"/>
	        <property name="password" value="${jdbc.password}"/>
	    </bean>
	</beans>

**@Configuration类为中心，利用@ImportResource导入XML**

	@Configuration
	@ImportResource("classpath:/com/acme/properties-config.xml")
	public class AppConfig {
	
	    @Value("${jdbc.url}")
	    private String url;
	
	    @Value("${jdbc.username}")
	    private String username;
	
	    @Value("${jdbc.password}")
	    private String password;
	
	    @Bean
	    public DataSource dataSource() {
	        return new DriverManagerDataSource(url, username, password);
	    }
	
	}

properties-config.xml

	<beans>
	    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
	</beans>
	
	jdbc.properties
	jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
	jdbc.username=sa
	jdbc.password=
	
	public static void main(String[] args) {
	    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
	    TransferService transferService = ctx.getBean(TransferService.class);
	    // ...
	}

#### 4.13 Bean definition profiles and environment abstraction

Bean definition profiles is a mechanism in the core container that allows for registration of different beans in different environments. This feature can help with many use cases, including:

- working against an in-memory datasource in development vs looking up that same datasource from JNDI when in QA or production
- registering monitoring infrastructure only when deploying an application into a performance environment
- registering customized implementations of beans for customer A vs. customer B deployments 

Find out more about Environment, XML Profiles and the @Profile annotation.

#### （未）4.14 PropertySource Abstraction

#### （未）4.15 Registering a LoadTimeWeaver

#### （未）4.16 Additional Capabilities of the ApplicationContext

#### （未）4.17 The BeanFactory ####