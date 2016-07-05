[toc]

### 16.16 配置pring MVC

本节介绍 MVC Java config 和 MVC XML namespace 两种配置方式。

The MVC Java config and the MVC namespace provide similar default configuration that overrides the DispatcherServlet defaults.

With the MVC Java config it is easier to see the underlying configuration as well as to make fine-grained customizations directly to the created Spring MVC beans.

#### 16.16.1 启用 MVC Java Config 或 the MVC XML Namespace

要启用 MVC Java config，向`@Configuration`类加注`@EnableWebMvc`：

    @Configuration
    @EnableWebMvc
    public class WebConfig {
    }

To achieve the same in XML use the `mvc:annotation-driven` element in your DispatcherServlet context (or in your root context if you have no DispatcherServlet context defined):

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc.xsd">

        <mvc:annotation-driven />

    </beans>

The above registers a `RequestMappingHandlerMapping`, a `RequestMappingHandlerAdapter`, and an `ExceptionHandlerExceptionResolver` (among others) in support of processing requests with annotated controller methods using annotations such as `@RequestMapping`, `@ExceptionHandler`, and others.

同时启用了：

1. Spring 3 style type conversion through a `ConversionService` instance in addition to the JavaBeans PropertyEditors used for Data Binding.
1. Support for formatting Number fields using the `@NumberFormat` annotation through the ConversionService.
1. Support for formatting Date, Calendar, Long, and Joda Time fields using the `@DateTimeFormat` annotation.
1. Support for validating `@Controller` inputs with `@Valid`, if a JSR-303 Provider is present on the classpath.
1. `HttpMessageConverter` support for `@RequestBody` method parameters and `@ResponseBody` method return values from `@RequestMapping` or `@ExceptionHandler` methods.

This is the complete list of `HttpMessageConverter`s set up by `mvc:annotation-driven`:

1. ByteArrayHttpMessageConverter converts byte arrays.
2. StringHttpMessageConverter converts strings.
3. ResourceHttpMessageConverter converts to/from org.springframework.core.io.Resource for all media types.
4. SourceHttpMessageConverter converts to/from a javax.xml.transform.Source.
5. FormHttpMessageConverter converts form data to/from a `MultiValueMap<String, String>`.
6. Jaxb2RootElementHttpMessageConverter converts Java objects to/from XML — added if JAXB2 is present on the classpath.
7. `MappingJackson2HttpMessageConverter` (or MappingJacksonHttpMessageConverter) converts to/from JSON — added if Jackson 2 (or Jackson) is present on the classpath.
8. `AtomFeedHttpMessageConverter` converts Atom feeds — added if Rome is present on the classpath.
9. RssChannelHttpMessageConverter converts RSS feeds — added if Rome is present on the classpath.

#### 16.16.2 定制提供的配置

要在Java中定制默认的配置，只需要实现`WebMvcConfigurer`接口，或扩展`WebMvcConfigurerAdapter`类。

    @Configuration
    @EnableWebMvc
    public class WebConfig extends WebMvcConfigurerAdapter {

        @Override
        protected void addFormatters(FormatterRegistry registry) {
            // Add formatters and/or converters
        }

        @Override
        public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
            // Configure the list of HttpMessageConverters to use
        }

    }

To customize the default configuration of `<mvc:annotation-driven />` check what attributes and sub-elements it supports. You can view the Spring MVC XML schema or use the code completion feature of your IDE to discover what attributes and sub-elements are available. The sample below shows a subset of what is available:

    <mvc:annotation-driven conversion-service="conversionService">
        <mvc:message-converters>
            <bean class="org.example.MyHttpMessageConverter"/>
            <bean class="org.example.MyOtherHttpMessageConverter"/>
        </mvc:message-converters>
    </mvc:annotation-driven>

    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="formatters">
            <list>
                <bean class="org.example.MyFormatter"/>
                <bean class="org.example.MyOtherFormatter"/>
            </list>
        </property>
    </bean>

#### 16.16.3 配置拦截器

可以配置`HandlerInterceptor`或`WebRequestInterceptor`。

    @Configuration
    @EnableWebMvc
    public class WebConfig extends WebMvcConfigurerAdapter {

        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new LocaleInterceptor());
            registry.addInterceptor(new ThemeInterceptor()).addPathPatterns("/").excludePathPatterns("/admin/");
            registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
        }

    }

And in XML use the `<mvc:interceptors>` element:

    <mvc:interceptors>
        <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor" />
        <mvc:interceptor>
            <mvc:mapping path="/"/>
            <mvc:exclude-mapping path="/admin/"/>
            <bean class="org.springframework.web.servlet.theme.ThemeChangeInterceptor" />
        </mvc:interceptor>
        <mvc:interceptor>
            <mvc:mapping path="/secure/*"/>
            <bean class="org.example.SecurityInterceptor" />
        </mvc:interceptor>
    </mvc:interceptors>

####（未）16.16.4 Configuring Content Negotiation

####（未）16.16.5 Configuring View Controllers

#### 16.16.6 配置伺服资源

复合特性URL的静态资源请求交由`ResourceHttpRequestHandler`处理。The `cache-period` property may be used to set far future expiration headers (1 year is the recommendation of optimization tools such as Page Speed and YSlow) so that they will be more efficiently utilized by the client. The handler also properly evaluates the `Last-Modified` header (if present) so that a 304 status code will be returned as appropriate, avoiding unnecessary overhead for resources that are already cached by the client. For example, to serve resource requests with a URL pattern of `/resources/**` from a public-resources directory within the web application root you would use:

    @Configuration
    @EnableWebMvc
    public class WebConfig extends WebMvcConfigurerAdapter {

        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler("/resources/**").addResourceLocations("/public-resources/");
        }

    }

And the same in XML:

	<mvc:resources mapping="/resources/**" location="/public-resources/"/>

To serve these resources with a 1-year future expiration to ensure maximum use of the browser cache and a reduction in HTTP requests made by the browser:

    @Configuration
    @EnableWebMvc
    public class WebConfig extends WebMvcConfigurerAdapter {

        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler("/resources/**").addResourceLocations("/public-resources/").setCachePeriod(31556926);
        }

    }

And in XML:

	<mvc:resources mapping="/resources/**" location="/public-resources/" cache-period="31556926"/>

`mapping`特性必须是一个Ant模式（用于`SimpleUrlHandlerMapping`的）。`location`特性必须是一个或多个（逗号分隔）有效的目录。The locations specified will be checked in the specified order for the presence of the resource for any given request. For example, to enable the serving of resources from both the web application root and from a known path of /META-INF/public-web-resources/ in any jar on the classpath use:

    @EnableWebMvc
    @Configuration
    public class WebConfig extends WebMvcConfigurerAdapter {

        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler("/resources/**")
                    .addResourceLocations("/", "classpath:/META-INF/public-web-resources/");
        }

    }

And in XML:

	<mvc:resources mapping="/resources/**" location="/, classpath:/META-INF/public-web-resources/"/>

When serving resources that may change when a new version of the application is deployed, it is recommended that you incorporate a version string into the mapping pattern used to request the resources, so that you may force clients to request the newly deployed version of your application’s resources. Such a version string can be parameterized and accessed using SpEL so that it may be easily managed in a single place when deploying new versions.

As an example, let’s consider an application that uses a performance-optimized custom build (as recommended) of the Dojo JavaScript library in production, and that the build is generally deployed within the web application at a path of **/public-resources/dojo/dojo.js**. Since different parts of Dojo may be incorporated into the custom build for each new version of the application, the client web browsers need to be forced to re-download that custom-built dojo.js resource any time a new version of the application is deployed. A simple way to achieve this would be to manage the version of the application in a properties file, such as:

	application.version=1.0.0

and then to make the properties file’s values accessible to SpEL as a bean using the `util:properties` tag:

	<util:properties id="applicationProps" location="/WEB-INF/spring/application.properties"/>

With the application version now accessible via SpEL, we can incorporate this into the use of the resources tag:

	<mvc:resources mapping="/resources-#{applicationProps[application.version]}/**" location="/public-resources/"/>

In Java, you can use the `@PropertySource` annotation and then inject the Environment abstraction for access to all defined properties:

    @Configuration
    @EnableWebMvc
    @PropertySource("/WEB-INF/spring/application.properties")
    public class WebConfig extends WebMvcConfigurerAdapter {

        @Inject Environment env;

        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler(
                    "/resources-" + env.getProperty("application.version") + "/**")
                    .addResourceLocations("/public-resources/");
        }

    }

and finally, to request the resource with the proper URL, we can take advantage of the Spring JSP tags:

    <spring:eval expression="@applicationProps[application.version]" var="applicationVersion"/>

    <spring:url value="/resources-{applicationVersion}" var="resourceUrl">
        <spring:param name="applicationVersion" value="${applicationVersion}"/>
    </spring:url>

    <script src="${resourceUrl}/dojo/dojo.js" type="text/javascript"> </script>

#### （未）16.16.7 Configuring Path Matching

#### （未）16.16.8 mvc:default-servlet-handler

#### （未）16.16.10 Advanced Customizations with MVC Java Config













