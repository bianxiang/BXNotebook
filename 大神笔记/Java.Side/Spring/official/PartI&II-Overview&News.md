
## Part I. 框架概览

## 1. 入门

## 2. 框架介绍

### 2.3 使用场景

#### 2.3.2 日志

Spring必需的日志依赖是Jakarta Commons Logging API (JCL)。We compile against JCL and we also make JCL Log objects visible for classes that extend the Spring Framework. It’s important to users that all versions of Spring use the same logging library: migration is easy because backwards compatibility is preserved even with applications that extend Spring. The way we do this is to make one of the modules in Spring depend explicitly on **commons-logging** (the canonical implementation of JCL), and then make all the other modules depend on that at compile time. If you are using Maven for example, and wondering where you picked up the dependency on commons-logging, then it is from Spring and specifically from the central module called **spring-core**.

The nice thing about commons-logging is that you don’t need anything else to make your application work. It has a runtime discovery algorithm that looks for **other logging frameworks** in well known places on the classpath and uses one that it thinks is appropriate (or you can tell it which one if you need to). If nothing else is available you get pretty nice looking logs just from the JDK (**java.util.logging** or JUL for short). 

**不使用Commons Logging**

不幸的是commons-logging的运行时发现机制有问题。若你创建新的Spring工程，建议使用别的日志依赖。第一选择是SLF4J。

弃用commons-logging有两种方式：

- Exclude the dependency from the **spring-core** module (as it is the only module that explicitly depends on commons-logging)
- Depend on a special commons-logging dependency that replaces the library with an empty jar (more details can be found in the SLF4J FAQ) 

To exclude commons-logging, add the following to your `dependencyManagement` section:

	<dependencies>
	    <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-core</artifactId>
	        <version>4.0.6.RELEASE</version>
	        <exclusions>
	            <exclusion>
	                <groupId>commons-logging</groupId>
	                <artifactId>commons-logging</artifactId>
	            </exclusion>
	        </exclusions>
	    </dependency>
	</dependencies>

Now this application is probably broken because there is no implementation of the JCL API on the classpath, so to fix it a new one has to be provided. In the next section we show you how to provide an alternative implementation of JCL using **SLF4J** as an example.

**使用 SLF4J**

SLF4J is a cleaner dependency and more efficient at runtime than commons-logging because it uses **compile-time bindings** instead of runtime discovery of the other logging frameworks it integrates. This also means that you have to be more explicit about what you want to happen at runtime, and declare it or configure it accordingly. SLF4J provides bindings to many common logging frameworks, so you can usually choose one that you already use, and bind to that for configuration and management.

SLF4J provides bindings to many common logging frameworks, including JCL, and it also does the reverse: bridges between other logging frameworks and itself. So to use SLF4J with Spring you need to replace the commons-logging dependency with the **SLF4J-JCL** bridge. Once you have done that then logging calls from within Spring will be translated into logging calls to the SLF4J API, so if other libraries in your application use that API, then you have a single place to configure and manage logging.

A common choice might be to bridge Spring to SLF4J, and then provide explicit binding from SLF4J to Log4J. You need to supply 4 dependencies (and exclude the existing commons-logging): the bridge, the SLF4J API, the binding to Log4J, and the Log4J implementation itself. In Maven you would do that like this

	<dependencies>
	    <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-core</artifactId>
	        <version>4.0.6.RELEASE</version>
	        <exclusions>
	            <exclusion>
	                <groupId>commons-logging</groupId>
	                <artifactId>commons-logging</artifactId>
	            </exclusion>
	        </exclusions>
	    </dependency>
	    <dependency>
	        <groupId>org.slf4j</groupId>
	        <artifactId>jcl-over-slf4j</artifactId>
	        <version>1.5.8</version>
	    </dependency>
	    <dependency>
	        <groupId>org.slf4j</groupId>
	        <artifactId>slf4j-api</artifactId>
	        <version>1.5.8</version>
	    </dependency>
	    <dependency>
	        <groupId>org.slf4j</groupId>
	        <artifactId>slf4j-log4j12</artifactId>
	        <version>1.5.8</version>
	    </dependency>
	    <dependency>
	        <groupId>log4j</groupId>
	        <artifactId>log4j</artifactId>
	        <version>1.2.14</version>
	    </dependency>
	</dependencies>

That might seem like a lot of dependencies just to get some logging. Well it is, but it is optional, and it should behave better than the vanilla commons-logging with respect to classloader issues, notably if you are in a strict container like an OSGi platform. Allegedly there is also a performance benefit because the bindings are at compile-time not runtime.

A more common choice amongst SLF4J users, which uses fewer steps and generates fewer dependencies, is to bind directly to **Logback**. This removes the extra binding step because Logback implements SLF4J directly, so you only need to depend on two libraries not four ( jcl-over-slf4j and logback). If you do that you might also need to exclude the slf4j-api dependency from other external dependencies (not Spring), because you only want one version of that API on the classpath.

**使用 Log4J**

Many people use Log4j as a logging framework for configuration and management purposes. It’s efficient and well-established, and in fact it’s what we use at runtime when we build and test Spring. Spring also provides some utilities for configuring and initializing Log4j, so it has an optional compile-time dependency on Log4j in some modules.

To make Log4j work with the default JCL dependency ( commons-logging) all you need to do is put Log4j on the classpath, and provide it with a configuration file ( log4j.properties or log4j.xml in the root of the classpath). So for Maven users this is your dependency declaration:

	<dependencies>
	    <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-core</artifactId>
	        <version>4.0.6.RELEASE</version>
	    </dependency>
	    <dependency>
	        <groupId>log4j</groupId>
	        <artifactId>log4j</artifactId>
	        <version>1.2.14</version>
	    </dependency>
	</dependencies>

And here’s a sample log4j.properties for logging to the console:

	log4j.rootCategory=INFO, stdout
	
	log4j.appender.stdout=org.apache.log4j.ConsoleAppender
	log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
	log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %t %c{2}:%L - %m%n
	
	log4j.category.org.springframework.beans.factory=DEBUG

**Runtime Containers with Native JCL**

Many people run their Spring applications in a container that itself provides an implementation of JCL. IBM Websphere Application Server (WAS) is the archetype. This often causes problems, and unfortunately there is no silver bullet solution; simply excluding commons-logging from your application is not enough in most situations.

To be clear about this: the problems reported are usually not with JCL per se, or even with commons-logging: rather they are to do with binding commons-logging to another framework (often Log4J). This can fail because commons-logging changed the way they do the runtime discovery in between the older versions (1.0) found in some containers and the modern versions that most people use now (1.1). Spring does not use any unusual parts of the JCL API, so nothing breaks there, but as soon as Spring or your application tries to do any logging you can find that the bindings to Log4J are not working.

In such cases with WAS the easiest thing to do is to invert the class loader hierarchy (IBM calls it "parent last") so that the application controls the JCL dependency, not the container. That option isn’t always open, but there are plenty of other suggestions in the public domain for alternative approaches, and your mileage may vary depending on the exact version and feature set of the container.

## Part II. Spring Framework 4.x的新特性 ##

## 3. 新特性与增强

全面支持Java 8特性。

A migration guide for upgrading to Spring 4.0 is available on the [Spring Framework GitHub Wiki](https://github.com/spring-projects/spring-framework/wiki).

### 3.3 Java 8 (as well as 6 and 7)

You can also use Java 8’s parameter name discovery (based on the -parameters compiler flag) as an alternative to compiling your code with debug information enabled.

### 3.5 Groovy Bean Definition DSL

With Spring Framework 4.0 it is now possible to define external bean configuration using a Groovy DSL. Using Groovy also allows you to easily embed bean definitions directly in your bootstrap code. 例如：

	def reader = new GroovyBeanDefinitionReader(myApplicationContext)
	reader.beans {
	    dataSource(BasicDataSource) {
	        driverClassName = "org.hsqldb.jdbcDriver"
	        url = "jdbc:hsqldb:mem:grailsDB"
	        username = "sa"
	        password = ""
	        settings = [mynew:"setting"]
	    }
	    sessionFactory(SessionFactory) {
	        dataSource = dataSource
	    }
	    myService(MyService) {
	        nestedBean = { AnotherBean bean ->
	            dataSource = dataSource
	        }
	    }
	}

### 3.6 核心容器增强

- Spring now treats generic types as a form of qualifier when injecting Beans. For example, if you are using a Spring Data Repository you can now easily inject a specific implementation: `@Autowired Repository<Customer> customerRepository`.
- If you use Spring’s meta-annotation support, you can now develop custom annotations that expose specific attributes from the source annotation.
- Beans can now be ordered when they are autowired into lists and arrays. Both the `@Order` annotation and `Ordered` interface are supported.
- The `@Lazy` annotation can now be used on injection points, as well as on `@Bean` definitions.
- The `@Description` annotation has been introduced for developers using Java-based configuration.
- A generalized model for conditionally filtering beans has been added via the `@Conditional` annotation. This is similar to `@Profile` support but allows for user-defined strategies to be developed programmatically.
- CGLIB-based proxy classes no longer require a default constructor. Support is provided via the **objenesis** library which is repackaged inline and distributed as part of the Spring Framework. With this strategy, no constructor at all is being invoked for proxy instances anymore.
- There is managed time zone support across the framework now, e.g. on `LocaleContext`.

### 3.7 Web增强

In addition to the **WebSocket** support mentioned later, the following general improvements have been made to Spring’s Web modules:

- You can use the new `@RestController` annotation with Spring MVC applications, removing the need to add `@ResponseBody` to each of your `@RequestMapping` methods.
- The `AsyncRestTemplate` class has been added, allowing non-blocking asynchronous support when developing REST clients.
- Spring now offers comprehensive timezone support when developing Spring MVC applications.

### 3.8 WebSocket, SockJS, and STOMP Messaging

A new **spring-websocket** module provides comprehensive support for WebSocket-based, two-way communication between client and server in web applications. It is compatible with JSR-356, the **Java WebSocket API**, and in addition provides SockJS-based fallback options (i.e. WebSocket emulation) for use in browsers that don’t yet support the WebSocket protocol (e.g. Internet Explorer < 10).

A new **spring-messaging** module adds support for STOMP as the WebSocket sub-protocol to use in applications along with an annotation programming model for routing and processing STOMP messages from WebSocket clients. As a result an `@Controller` can now contain both `@RequestMapping` and `@MessageMapping `methods for handling HTTP requests and messages from WebSocket-connected clients. The new spring-messaging module also contains key abstractions formerly from the Spring Integration project such as Message, MessageChannel, MessageHandler, and others to serve as a foundation for messaging-based applications.

### 3.9 测试增强

In addition to pruning of deprecated code within the spring-test module, Spring Framework 4.0 introduces several new features for use in unit and integration testing.

- Almost all annotations in the spring-test module (e.g., @ContextConfiguration, @WebAppConfiguration, @ContextHierarchy, @ActiveProfiles, etc.) can now be used as meta-annotations to create custom composed annotations and reduce configuration duplication across a test suite.
- Active bean definition profiles can now be resolved programmatically, simply by implementing a custom `ActiveProfilesResolver` and registering it via the resolver attribute of `@ActiveProfiles`.
- A new SocketUtils class has been introduced in the spring-core module which enables you to scan for free TCP and UDP server ports on localhost. This functionality is not specific to testing but can prove very useful when writing integration tests that require the use of sockets, for example tests that start an in-memory SMTP server, FTP server, Servlet container, etc.
- As of Spring 4.0, the set of mocks in the `org.springframework.mock.web` package is now based on the Servlet 3.0 API. Furthermore, several of the Servlet API mocks (e.g., MockHttpServletRequest, MockServletContext, etc.) have been updated with minor enhancements and improved configurability.
