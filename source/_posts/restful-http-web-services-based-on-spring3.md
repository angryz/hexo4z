title: 使用 Spring 3 构建基于 HTTP 的RESTful Web Services
date: 2013-08-30 15:18:30
tags: [rest, spring]
categories: Develop
---

本文不讲述 RESTful Web Services 的概念和理论，你需要了解什么是 RESTful Web Services。

本文不讲述 Spring Framework 的使用和技巧，你需要熟悉 Spring Framework 和 Spring MVC。

本文内容主要是简单说明使用 Spring 3 来构建一个简易的 RESTful Web Service 相关内容。

Spring 自版本3开始增加了对 RESTful Web Services 开发的支持，且 REST 已被无缝整合到了 Spring 的 MVC 框架中，因此在 Spring 中创建 RESTful Web Services 是十分容易的。这主要体现在:

* 通过 `@RequestMapping`, `@PathVariable`, `@RequestBody`, `@ResponseBody` 等注解(Annotation)可以简单的创建资源标识和URI映射
* 丰富的数据载体支持，比如 XML、JSON
* 完全采用 Spring MVC 模型进行开发，使用 Spring 开发的应用可无缝过渡到 REST
<br>

* * *

下面通过一个例子来说明使用 Spring 3 创建一个 RESTful Web Service 需要的操作。

对于本例的几点说明：

1. 通信协议为 HTTP 协议；
2. 参数类型为 JSON 格式；
3. 例子中的代码均采用省略的写法，只写出需要注意的部分；
<br>

## 代码依赖
* spring-webmvc
* jackson-core-asl
* jackson-mapper-asl


如果使用 Maven 的话可以参考如下配置：
```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>3.2.4.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-core-asl</artifactId>
  <version>1.9.13</version>
</dependency>
<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-mapper-asl</artifactId>
  <version>1.9.13</version>
</dependency>
```
<br>

## Java 实现

本例中我们假设只有一个 Entity 对象 __Customer__，实现如下：

```Java
package demo.rest.model;

public class Customer {
  private long id;
  private String name;
  private String mobile;
  private String address;
  private String email;

  // omit implementation of Getters and Setters below
  // ......
}
```
<br>

同时，针对 __Customer__ 的持久化操作实现 __CustomerRepository__ 大致如下：

```Java
package demo.rest.repo;

public class CustomerRepository {

  public Customer query(long id) {
    // omitted
  }

  public List<Customer> queryAll() {
    // omitted
  }

  public void add(Customer customer) {
    // omitted
  }

  public void update(Customer customer) {
    // omitted
  }

  public void remove(long id) {
    // omitted
  }
}
```
<br>
<!-- more -->

熟悉 Sping MVC 的小伙伴应该知道，我们还需要实现 __Controller__：
```Java
package demo.rest.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class DefaultController {
	
  private CustomerRepository customerRepo;

  @RequestMapping(method=RequestMethod.GET, value="/customer/{id}")
  @ResponseBody
  public Customer getCustomer(@PathVariable String id) {
    return customerRepo.query(Long.parseLong(id));
  }

  @RequestMapping(method=RequestMethod.GET, value="/customers")
  @ResponseBody
  public List<Customer> getAllCustomers() {
    return customerRepo.queryAll();
  }

  @RequestMapping(method=RequestMethod.POST, value="/customer")
  @ResponseBody
  public List<Customer> addCustomer(@RequestBody Customer customer) {
    customerRepo.add(customer);
    return customerRepo.queryAll();
  }

  @RequestMapping(method=RequestMethod.PUT, value="/customer")
  @ResponseBody
  public Customer updateCustomer(@RequestBody Customer customer) {
    customerRepo.update(customer);
    return customerRepo.query(customer.getId());
  }

  @RequestMapping(method=RequestMethod.DELETE, value="/customer/{id}")
  @ResponseBody
  public List<Customer> removeCustomer(@PathVariable String id) {
    customerRepo.remove(id);
    return customerRepo.queryAll();
  }

  public void setCustomerRepo(CustomerRepository customerRepo) {
    this.customerRepo = customerRepo;
  }
}
```

简单解释一下这段代码：

* 通过 `@RequestMapping` 指定 HTTP 方法和资源映射
* 通过 `@ResponseBody` 结合后面要提到的 __MappingJacksonHttpMessageConverter__ 来完成数据对象到 JSON 的序列化转换
* `@PathVariable` 用来获取资源URI中的参数，即 `{}` 中的内容
* `@RequestBody` 用来获取 HTTP 请求体中的数据，同样结合 __MappingJacksonHttpMessageConverter__ 完成了 JSON 到数据对象的反序列化
<br>

## 基本配置

和通常使用 Spring 的时候一样，我们通过配置 _web.xml_ 来激活 __Spring WebApplicationContext__。并添加 Spring MVC 中的 __DispatcherServlet__ 来处理 HTTP 请求。
```xml
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>/WEB-INF/rest-context.xml</param-value>
</context-param>

<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<servlet>
  <servlet-name>rest</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
  <servlet-name>rest</servlet-name>
  <url-pattern>/service/*</url-pattern>
</servlet-mapping>
```
   

在 _rest-context.xml_ 中配置基础设施和系统服务，
```xml
<bean id="customerRepo" class="demo.rest.repo.CustomerRepository" />
```
   

在 _rest-servlet.xml_ 中配置 MVC 和 REST 相关配置，
```xml
<!-- Auto scan controllers -->
<context:component-scan base-package="demo.rest.controller" />

<!-- enable @RequestMapping process on type level and method level -->
<bean class="org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping" />
<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
  <property name="messageConverters">
    <util:list id="beanList">
      <ref bean="mappingJacksonHttpMessageConverter" />
    </util:list>
  </property>
</bean>

<!-- use MappingJacksonHttpMessageConverter to do Serialization/Deserialization of JSON -->
<bean id="mappingJacksonHttpMessageConverter" class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
  <property name="supportedMediaTypes">
    <list>
      <value>application/json;charset=UTF-8</value>
    </list>
  </property>
</bean>

<!-- injection of CustomerRepository -->
<bean id="defaultController" class="demo.rest.controller.DefaultController">
  <property name="customerRepo" ref="customerRepo" />
</bean>
```

简单说明：

* __component-scan__ 会自动扫描添加了 Spring 注解的类，比如本例中添加了 `@Controller` 的类
* __DefaultAnnotationHandlerMapping__ 和 __AnnotationMethodHandlerAdapter__ 会激活类和方法的 `@RequestMapping` 注解，从而将URI映射交给 Spring 来处理
* __MappingJacksonHttpMessageConverter__ 用于处理 HTTP 消息中的 JSON 参数和 Java 对象之间的序列化/反序列化转换


以上就是本文例子的简单实现。
<br>
* * *

### 几点讲解

* 使用 `@ResponseBody` 会跳过视图处理部分，并调用合适的 _HttpMessageConverter_ 将返回值进行处理后直接输出到 HTTP，因此我们在 Controller 中实现的方法不需要返回 _ModelAndView_ 对象。
* 本例中使用的 _MappingJacksonHttpMessageConverter_ 通过 Jackson 的 ObjectMapper 读写 JSON 数据，它可以转换媒体类型为 _application/json_ 的数据。
* 如果要使用 XML 作为数据载体，则本例中的 _DefaultController_ 和 _rest-servlet.xml_ 都需要修改。可以使用 _org.springframework.oxm.jaxb.Jaxb2Marshaller_ 对 XML 进行序列/反序列化，使用 `@XmlRootElement`,`@XmlElement` 注解来定义 Java 对象与 XML 之间的转换关系。
