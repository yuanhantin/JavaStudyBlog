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

![image-20230405132628430](https://s2.loli.net/2023/06/02/8LzQsGXtVUy5hBg.png)

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

