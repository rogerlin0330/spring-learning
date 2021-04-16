# <img src="images/icons/spring-framework.png" width="80" height="80"> spring-learning

[![GitHub license](https://img.shields.io/github/license/Naereen/StrapDown.js.svg)](https://github.com/Naereen/StrapDown.js/blob/master/LICENSE) [![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://GitHub.com/Naereen/StrapDown.js/graphs/commit-activity)

_This repo contains learning notes about the analysis of the source code of the **Spring Framework (v5.2.x)** and **Spring Boot (v2.4.x)**. I am trying to make the notes as easy to be understood as possible as I can, Therefore, I mainly organize the notes into diagrams combining with just some tiny code snippets in case for a clearer explanation._

_<p align="right">Roger Lin, 04/15/2021, Los Angeles</p>_

## Spring Framwork

### Spring IoC Container ([click for reading](./notes/Spring-IoC-Container/Spring-IoC-Container.md))

In this section, we will discuss:

- What happens behind when you exectue the following code?

  ```java
  public class SpringDemo {
      public static void main(String[] args) {
          ApplicationContext applicationContext = new XmlWebApplicationContext(
            new ClasspathResource("applicationContext.xml")
          )
      }
  }

  ```

  or

  ```java
  @Configuration
  public class SpringDemo {
      public static void main(String[] args) {
          ApplicationContext applicationContext =
            new AnnotationConfigApplicationContext(SpringDemo.class)
      }
  }
  ```

- What's the relationship between `ApplicationContext` and `BeanFactory`?

- How bean definitions in XML-based configuration file are being loaded?

- How bean definitions in JavaConfig classes are being loaded?

- The extension points of the Spring IoC Container

  - BeanFactoryPostProcessor

  - BeanPostProcessor

- etc.

### Spring AOP

_TODO: waiting for new content_

## Spring Boot

### Spring Boot AutoConfiguration ([click for reading](./notes/Spring-Boot-AutoConfigureation/Spring-Boot-AutoConfiguration.md))

In this section, we will discuss:

- How does Spring Boot implement its auto-configuration feature?

- How to encapsulate a Spring Boot starter for custom needs?

<br>

---

_This repo is frequently and continuously updating. If you are interested in it, I appreciate your click of the like and the watch button. Thanks._

_The diagrams in this repo are created by [OmniGraffle](https://www.omnigroup.com/omnigraffle/)._
