## 浏览器访问 Web 服务过程详解



### web服务是什么

Web Service是一个 **平台 独立的，低耦合的，自包含的、基于可 编程 的web的应用程序**，可使用开放的 XML （ 标准通用标记语言 下的一个子集） 标准 来 描述 、发布、发现、协调和配置这些应用程序，用于开发分布式的交互操作的 应用程序 。 Web Service技术， 能使得运行在不同机器上的不同应用无须借助附加的、专门的第三方软件或硬件， 就可相互交换数据或集成。

**一言以蔽之：WebService是一种跨编程语言和跨操作系统平台的远程调用技术。**

### web服务部署在哪？

放在Tomcat下的webapps文件夹中，如果直接访问http：//ip:端口  默认访问的是其中的ROOT文件

![image-20230405124310193](https://s2.loli.net/2023/04/05/4Yla29q3xRVPKQ7.png)



![image-20230403223928624](https://s2.loli.net/2023/04/03/LwDG4ndySPImUpv.png)