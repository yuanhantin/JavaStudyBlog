# JavaWeb技术栈图（web服务器+web容器是何物）

![image-20230406221105785](https://s2.loli.net/2023/04/06/rCj9BUXuWVb1h6D.png)

**两个重要概念web服务器+web容器**

### 什么是Web服务器？

Tomcat 服务器就是一个免费的开放源代码的 Web 应用服务器

web服务实际上就是解析了客户端/浏览器发来的http请求，并将其做出一定的处理。比如说将请求头和请求体中的各个元素拆开打包成一个对象，从而供后端（容器）调用API获取。

### 什么是web 容器？

WEB 容器给处于其中的应用程序组件（jsp，servlet）提供一个环境，使 jsp，servlet 直接跟容器中的环境变量交互，不必关注其它系统问题（从这个角度来说，web 容器应该属于架构上的概念）。web **容器**主要**由** WEB **服务器**来实现。例如：Tomcat，weblogic

WEB 容器更多的是跟基于 HTTP 的请求打交道，比如说tomcat里的容器就利用了dom4j读取web.xml的各个元素，并放入一个大的类似HashMap里，从而可以通过HTTP的URI获取到相应的servlet的全类名，从而通过反射机制获取URI所对应的servlet对象，实现servlet其中的业务逻辑。

### 应用程序服务器（The Application Server）

  根据定义，作为应用程序服务器，要求可以通过各种协议（比如HTTP 协议）把商业逻辑暴露给客户端应用程序。应用程序使用此商业逻辑就像你调用对象的一个方法一样。

**【serverlet】**

​      Servlet（Server Applet），全称 Java Servlet，未有中文译文。是用 Java 编写的服务器端程序。其主要功能在于交互式地浏览和修改数据，生成动态 Web 内容。狭义的 Servlet 是指 Java 语言实现的一个接口，广义的 Servlet 是指任何实现了这个 Servlet 接口的类，一般情况下，人们将 Servlet 理解为后者。

​      Servlet 运行于支持 Java 的应用服务器中。从实现上讲，Servlet 可以响应任何类型的请求，但绝大多数情况下 Servlet 只用来扩展基于 HTTP 协议的 Web 服务器。

**【Tomcat】**

​      Tomcat 服务器是一个免费的开放源代码的 Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试 JSP 程序的首选。对于一个初学者来说，可以这样认为，当在一台机器上配置好 Apache 服务器，可利用它响应对 HTML 页面的访问请求。实际上 Tomcat 部分是Apache 服务器的扩展，但它是独立运行的，所以当你运行 tomcat 时，它实际上作为一个与 Apache 独立的进程单独运行的。
