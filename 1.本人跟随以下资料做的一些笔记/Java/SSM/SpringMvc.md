# SpringMvc

### 1.SpringMVC-基本介绍

1. SpringMVC 从易用性，效率上 比曾经流行的 Struts2 更好

2. SpringMVC 是 WEB 层框架【SpringMVC 接管了 Web 层组件, 比如控制器, 视图, 视图解析, 返回给用户的数据格式, 同时支持 MVC 的开发模式/开发架构】 
3. SpringMVC 通过注解，让 POJO 成为控制器，不需要继承类或者实现接口 
4. SpringMVC 采用低耦合的组件设计方式，具有更好扩展和灵活性. 
5. 支持 REST 格式的 URL 请求
6. SpringMVC 是基于 Spring 的, 也就是 SpringMVC 是在 Spring 基础上的。SpringMVC 的核 心包 spring-webmvc-xx.jar 和 spring-web-xx.jar

####  Spring SpringMVC 与 SpringBoot 的关系 

1. Spring MVC 只是 Spring 处理 WEB 层请求的一个模块/组件, Spring MVC 的基石是 Servlet[Java WEB] 
2. Spring Boot 是为了简化开发者的使用, 推出的封神框架(约定优于配置，简化了 Spring 的配置流程), SpringBoot 包含很多组件/框架，Spring就是最核心的内容之一，也包含 Spring MVC 
3. 他们的关系大概是: Spring Boot > Spring > Spring MVC





### 2.配置方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!-- 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理 -->
    <servlet>
        <servlet-name>springMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 通过初始化参数指定SpringMVC配置文件的位置和名称 -->
        <init-param>
            <!-- contextConfigLocation为固定值 -->
            <param-name>contextConfigLocation</param-name>
            <!-- 使用classpath:表示从类路径查找配置文件，例如maven工程中的src/main/resources -->
            <param-value>classpath:springMVC.xml</param-value>
        </init-param>
        <!--
             作为框架的核心组件，在启动过程中有大量的初始化操作要做
            而这些操作放在第一次请求时才执行会严重影响访问速度
            因此需要通过此标签将启动控制DispatcherServlet的初始化时间提前到服务器启动时
        -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springMVC</servlet-name>
        <!--
            设置springMVC的核心控制器所能处理的请求的请求路径
            /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
            但是/不能匹配.jsp请求路径的请求
        -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```





### 3.@RequestMapping注解

**作用**：顾名思义，@RequestMapping注解的作用就是将请求和处理请求的控制器方法关联起来，建立映射关系。SpringMVC 接收到指定的请求，就会来找到在映射关系中对应的控制器方法来处理这个请求。

**位置**

- @RequestMapping标识一个类：设置请求路径的初始信息，作为类里所有控制器方法映射路径的的公共前缀部分
- @RequestMapping标识一个方法：设置映射请求请求路径的具体信息