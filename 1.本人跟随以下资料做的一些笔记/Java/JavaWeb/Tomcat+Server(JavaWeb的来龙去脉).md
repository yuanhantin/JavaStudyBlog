# Tomcat+Server(服务器+web服务)

![image-20230330151825586](https://s2.loli.net/2023/03/30/eyA49S3krCZTqv8.png)

**Tomcat其实本质就是一个java程序，而且他就是java写的，只不过他实现了接收浏览器的HTTP请求，并在后端操作过后响应浏览器而已。**



### 如何理解容器 ?  容器并不完全是tomcat

我们的钱（Servlet）要放在钱包(Wrapper)里，钱包要放在书包(Context)里，书包要放在行李箱(Host)里，行李箱要放在飞机(Engine)上。

所以，如果你问我“Engine放哪？”就相当于问我“飞机放哪？”

答案是不再需要更高层次的容器了，因为没有必要了。

**总结**
在Tomcat中，容器分为：

1. Wrapper
2. Context
3. Host
4. Engine

