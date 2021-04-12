# Spring IoC Container

This section explains how Spring initializes the IoC container based on the following setting in web.xml when Tomcat is on.

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<web-app version="3.0" xmlns="http://java.sun.com/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">
    <!-- Reference: https://my.oschina.net/u/4381731/blog/3673717 -->

    <!-- Other configurations -->

    <listener>  
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>  
    </listener>

    <context-param>  
        <param-name>contextConfigLocation</param-name>  
        <param-value>classpath:spring/applicationContext.xml</param-value>  
    </context-param>

    <!-- Other configurations -->
</web-app>
```

The following sequence diagram shows the business logic of how Spring initializes an xml based ApplicationContext by letting Tomcat invoke the `initWebApplicationContext(servletContext:javax.servlet.ServletContext)` method contracted in the `javax.servlet.ServletContextListener` interface, and implemented in `org.springframework.web.context.ContextLoaderListener`.

![ContextLoaderListner_to_BeanFactory](../../images/Spring-IoC-Container/ContextLoaderListner->XmlWebApplictionContext->BeanFactory.png)

As one can see, the core logic of obtaining the bean factory with the specified bean definitions is implemtned in the `AbstractApplicationContext#refresh` method. The following sequence diagram shows the most important logics of this method.

![AbstractApplicationContext_refresh](../../images/Spring-IoC-Container/AbstractApplicationContext_refresh.png)

Notice that XML-based application context classes have a different entrance of loading bean definitions from configuration files than the annotation-based application context classes. The former use the `loadBeanDefinitions(beanFactory:DefaultListableBeanFactory)` defined in `AbstractRefreshableApplicationContext` wheras the latter one use the `register` and `scan` methods in `AnnotationConfigApplicationContext`. For more details, please refer to the diagram shown above.
