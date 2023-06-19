# Servlet

### 1.什么是Servlet

Servlet(java 服务器小程序)

1. 他是由**服务器端**调用和执行的(一句话：是Tomcat解析和执行) 
2. 他是用java语言编写的, 本质就是Java类 
3.  他是按照Servlet规范开发的(除了tomcat->Servlet weblogic->Servlet)
4.  功能强大，可以完成几乎所有的网站功能

### 2.开发方式

1. 编写一个类去实现 Servlet 接口 / 编写一个类去继承 HttpServlet 类 (在实际项目中，继承 HttpServlet 类开发 Servlet 程序，更加方便)
2. 根据业务需要重写 doGet 或 doPost 方法
3. 在 web.xml/注解 中去配置 servlet 程序的访问地址
4. 当然IDEA也可以直接右键create new servlet自动完成以上过程

### 3.浏览器调用 Servlet 流程分析

![image-20230403165424099](https://s2.loli.net/2023/04/03/sdpMR6UjlJxrD48.png)

![image-20230403165951952](https://s2.loli.net/2023/04/03/DoLMpITjwcnbOuh.png)



### 4.在web.xml定义一个servlet

```xml
<servlet>  
<servlet-name>t1</servlet-name>  
<servlet-class>类</servlet-class>  
<!-- 如果需要自动加载，加下面一句 -->  
<load-on-startup>1</load-on-startup>  
</servlet>  
```

<load-on-startup>标记web容器是否在启动的时候就加载这个servlet 

当值为0或者大于0时，表示web容器在应用启动时就加载这个servlet； 

当是一个负数时或者没有指定时，则指示容器在该servlet被选择时才加载； 

正数的值越小，启动该servlet的优先级越高。 

### 5.Servlet 注意事项和细节 

1. **Servlet 是一个**供其他 Java 程序（Servlet 引擎）调用的 **Java 类**，不能独立运行 
2.  针对浏览器的多次 Servlet 请求，通常情况下，服务器只会创建一个 Servlet 实例对象， 也就是说 Servlet 实例对象一旦创建，它就会驻留在内存中，为后续的其它请求服务，直至 web 容器退出/或者 redeploy 该 web 应用，servlet 实例对象才会销毁 
3. 在 Servlet 的整个生命周期内，**init 方法只被调用一次**。而对**每次请求**都导致 Servlet 引 擎**调用一次 servlet 的 service 方法**。
4. 对于**每次**访问请求，Servlet 引擎**都会创建一个新的** HttpServletRequest **请求对象**和一个 新的 HttpServletResponse **响应对象**，然后**将这两个对象作为参数传递给它调用的 Servlet 的 service()方法**，service 方法再**根据请求方式分别调用 doXXX 方法**  
5. 如果在元素中配置了一个<load-on-startup>元素，那么 WEB 应用程序在启动时， 就会装载并创建 Servlet 的实例对象、以及调用 Servlet 实例对象的 init()方法

### 6.ServletConfig

#### 基本介绍:

1. ServletConfig 类是为 Servlet 程序的配置信息的类 
2. Servlet 程序和 ServletConfig 对象都是由 Tomcat 负责创建 
3.  Servlet 程序默认是第 1 次访问的时候创建，ServletConfig 在 Servlet 程序创建时，就创建一个对应的 ServletConfig 对象

#### ServletConfig 类能干什么:

1. 获取 Servlet 程序的 servlet-name (下文中的DBServlet)

2. 获取xml中配置好的初始化参数 init-param中的param-name和value

3. 获取 ServletContext 对象

   **一言以蔽之:获取在web.xml中配置好的信息和ServletContext **

   ```xml
    <servlet>
           <servlet-name>DBServlet</servlet-name>
           <servlet-class>DBServlet</servlet-class>
           <init-param>
               <param-name>username</param-name>
               <param-value>root</param-value>
           </init-param>
           <init-param>
               <param-name>pwd</param-name>
               <param-value>20030515</param-value>
           </init-param>
    </servlet>
       <servlet-mapping>
           <servlet-name>DBServlet</servlet-name>
           <url-pattern>/dbservlet</url-pattern>
       </servlet-mapping>
   ```

### 7.ServletContext

#### 基本介绍:

1. ServletContext 是一个接口，它表示 Servlet 上下文对象
2.  一个 web 工程，只有一个 ServletContext 对象实例 
3. ServletContext 对象 是在 web 工程启动的时候创建，在 web 工程停止的时销毁
4. ServletContext 对象可以通过 ServletConfig.getServletContext 方法获得对 ServletContext 对象的引用，也可以通过 this.getServletContext()来获得其对象的引用。 
5. 由于一个 WEB 应用中的所有 Servlet 共享同一个 ServletContext 对象，因此 Servlet 对象 之间可以通过 ServletContext 对象来实现多个 Servlet 间通讯。ServletContext 对象通常也被 称之为域对象。

#### ServletContext 能干什么 :

1. 获取 web.xml 中配置的上下文参数 **context-param** （信息和整个 web 应用相关，而不是属于某个Servlet）

2. 获取当前的**工程路径**，( 比如 /servlet )

3. 获取工程部署后在服务器硬盘上的**绝对路径 ** **注意是out目录下的**，因为访问资源都是访问编译后的源文件

   ( 比 如 : D:\hspedu_javaweb\servlet\out\artifacts\servlet_war_exploded)

4. 像 Map 一样存取数据, 多个 Servlet**共享数据**

![image-20230405132628430](https://s2.loli.net/2023/04/05/3ct8sjBxyDTaFnW.png)

### 8.HttpServletRequest 

#### 基本介绍: 

1. HttpServletRequest 对象代表客户端的请求 
2.  当客户端/浏览器通过 HTTP 协议访问服务器时，HTTP 请求头中的所有信息都封装在这个对象中 
3.  通过这个对象的方法，可以获得客户端这些信息。

#### HttpServletRequest 能干什么:

获取封装好的 HTTP 请求消息中的各类元素(请求头、附带的数据等）

#### 请求转发

请求转发指一个 web 资源收到客户端请求后，通知服务器去调用另外 一个 web 资源进行处理

请求转发注意事项和细节 

1. 浏览器地址不会变化(地址会保留在第 1 个 servlet 的 url) 
2. 在同一次 HTTP 请求中，进行多次转发，仍然是一次 HTTP 请求
3.  在同一次 HTTP 请求中，进行多次转发，多个 Servlet 可以共享 request 域/对象的数据(因 为始终是同一个 request 对象) 
4. 可以转发到 WEB-INF 目录下(后面做项目使用)
5. 不能访问当前 WEB 工程外的资源



# Servlet三大作用域：Request、Session、Application

## Request作用域

当请求从一个action转发到一个jsp的时候，如果jsp中要使用action类中的变量，那么我们需要将action中的变量放入到request作用域中传给jsp。那么jsp中就可以通过request作用域获取到该变量。

例如：登陆成功后需要在成功页面显示人员信息。

Request对象类似于一个map集合。放数据的时候，放入键值对，取数据时通过键取值。

```java
request.setAttribute(key,value);		//往request作用域里面放入数据。

request.getAttribute(key);				//从requtest作用域里面获取数据。
```

其他作用域类似，所有的作用域都有上面的两个方法用来放和取。

### Request作用域 注意点：

**放数据的servlet和取数据的servlet必须保证使用的是同一个request对象。一旦request对象改变了，那么，request作用域里面的值就会失效。**

## session作用域

### session 底层实现机制

![image-20230407191553851](https://s2.loli.net/2023/06/02/rpgX7E5fLUsdz94.png)

```java
session.setAttribute(key,value);	//到session作用域放数据

session.getAttribute(key);			//从session作用域获取数据

session.removeAttibute(key);		//从session作用域里面移除数据。
```

## Application作用域：ServletContext

#### 基本介绍:

1. ServletContext 是一个接口，它表示 Servlet 上下文对象
2. 一个 web 工程，只有一个 ServletContext 对象实例 
3. ServletContext 对象 是在 web 工程启动的时候创建，在 web 工程停止的时销毁
4. ServletContext 对象可以通过 ServletConfig.getServletContext 方法获得对 ServletContext 对象的引用，也可以通过 this.getServletContext()来获得其对象的引用。 
5. 由于一个 WEB 应用中的所有 Servlet 共享同一个 ServletContext 对象，因此 Servlet 对象 之间可以通过 ServletContext 对象来实现多个 Servlet 间通讯。ServletContext 对象通常也被 称之为域对象。

#### ServletContext 能干什么 :

1. 获取 web.xml 中配置的上下文参数 **context-param** （信息和整个 web 应用相关，而不是属于某个Servlet）

2. 获取当前的**工程路径**，( 比如 /servlet )

3. 获取工程部署后在服务器硬盘上的**绝对路径 ** **注意是out目录下的**，因为访问资源都是访问编译后的源文件

   ( 比 如 : D:\hspedu_javaweb\servlet\out\artifacts\servlet_war_exploded)

4. 像 Map 一样存取数据, 多个 Servlet**共享数据**

![image-20230405132628430](https://s2.loli.net/2023/06/02/EnODxHughNyf1lS.png)

### 

```java
sc.setAttribute(key,value);		//把变量放入作用域

sc.getAttribute(key);			//从作用域里面获取变量

sc.removeAttribute(key);		//从作用域里面移除对象。
```



## 三大作用域的比较

request，session，application（ServletContext）

Request作用域的范围**最小**，对应一次请求。

Session作用域的范围要大一些。对应**一次会话**，只要浏览器不关就是一个session。

Application作用域的范围**最大**。对应一个应用程序。不管有多少个浏览器，只要访问的是同一个项目，那么他们对应的就是一个application对象。

Application对象跟一个项目相关，他会在tomcat启动的时候被创建，tomcat关闭的时候被销毁。

