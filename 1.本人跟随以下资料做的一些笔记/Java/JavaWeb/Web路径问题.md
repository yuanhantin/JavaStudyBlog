# JavaWeb中Servlet、web应用和web站点的路径细节（"/"究竟代表着什么）



### 1 开门见山

新建一个tomcat web项目，配置tomcat的虚拟目录，取默认值(/项目名_war_exploded)

![在这里插入图片描述](https://s2.loli.net/2023/04/09/WBrvjs1wMJGlgnD.png)

那么如果你的tomcat的默认站点（即http://localhost:8080）没有更改的话，这个项目的两个重要的根目录就出来了

web**站点**根目录为：http://localhost:8080
web**应用**的根目录为：http://localhost:8080/WebPathDemo_war_exploded

注意:只有web应用才能访问应用里的资源

### 2 查看web应用的根目录的方法

第一种方式查看：
转到tomcat的Server，我们的URL就相当于web应用的根目录(去掉“/”就算当前目录)

![在这里插入图片描述](https://s2.loli.net/2023/04/09/XTmY8eVlRc6EHWt.png)

第二种方式查看：
在sevlet中使用request.getContextPath();查看，查看的是web应用的相对目录

```java
@WebServlet("/WebPath")
public class WebPath extends HttpServlet {
    
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws
     ServletException, IOException {
        //得到web应用的相对地址
        String contextPath = request.getContextPath();// 得到“/WebPathDemo_war_exploded”
        System.out.println("得到web应用的相对地址为：" + contextPath);
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws
     ServletException, IOException {
        doPost(request, response);
    }
}

```

浏览器访问http://localhost:8080/WebPathDemo_war_exploded/WebPath
控制台输出结果为得到web应用的相对地址为：/WebPathDemo_war_exploded

request.getContextPath();告诉了我们web应用的相对目录，相对于谁呢？相对于web站点根目录：http://localhost:8080

那么一切似乎清晰了，"web应用的根目录"就是在"web站点的根目录"后加入tomcat配置的虚拟目录 (第一张图里设置的东西)；

### 3 搞清楚什么时候是根据web站点根目录作为相对目录，什么时候是根据web应用根目录作为相对目录

#### 3.1 如何访问资源

访问一个资源，一定是相对于web应用根目录的,比如在我这个项目中，一定是
http://localhost:8080/WebPathDemo_war_exploded/  + 资源路径(servlet或者html、jsp)，才能访问资源。

![在这里插入图片描述](https://s2.loli.net/2023/04/09/eXsECNJnypkLgR8.png)

比如我访问demo1下的index1.jsp,是必须要带上web应用根目录在前方的

####  3.2判断“/”是相对于web站点还是web应用根目录进行访问

a标签内的相对于web站点根目录还是web应用根目录的判断条件是，最前方有没有“/”

- 无“/”，代表是相对于web应用根目录,即http://localhost:8080/ Application context/
- 有“/”，代表是相对于web站点根目录,即http://localhost:8080/

jsp的include、form表单等等也是如此的判断条件。

比如说: 下面的"  /   "被浏览器解析成http:// ip:端口/   所以我们这样写会被浏览器解析成http://localhost:8080/index.jsp很显然,少了web应用的根目录,我们无法访问

```jsp
<a href="/index1.jsp">index1.jsp</a>
```

我们可以采用另外的方式解决此问题:

- 死板方式，直接去掉  " / "	浏览器解析下面的代码为http://localhost:8080/WebPathDemo_war_exploded/index1.jsp

  ```jsp
  <a href="index1.jsp">index1.jsp</a>
  ```

- 灵活方式，采用EL表达式（这里不再进行介绍）其中<%=request.getContextPath()%>会被翻译为上文所提过的web应用的相对目录  /WebPathDemo_war_exploded，此时浏览器也就访问的也是http://localhost:8080/WebPathDemo_war_exploded/index1.jsp

  ```jsp
  <a href="<%=request.getContextPath()%>/index1.jsp">index1.jsp</a>
  ```

### 4.Servlet配置url路径问题

不管是注解还是xml的配置都是一样的这里我们用注解进行讲解

```java
@WebServlet(urlPatterns ="/WebPath")
```

其中的第一个" / " ，最终会被浏览器解析为web**应用**的根目录路径http://localhost:8080/WebPathDemo_war_exploded/，不仅是注解，还有请求转发也是这样解析为web应用根目录路径。

注意：java中，将" / " 解析成web应用根目录又与前文的jsp和html中的a标签、form表单等等恰恰相反，个人认为可以这么理解

- 将" / " 解析成web**应用**根目录是java和xml的特点
- 将" / " 解析成web**站点**根目录是JSP和HTML的特点

```java
 RequestDispatcher requestDispatcher = request.getRequestDispatcher("/html/a.html");
```

### 遇到无法访问的路径怎么办

- 先检查代码的web路径问题
- 如果没有问题可能是浏览器缓存的问题，在开发者模式中勾选禁用缓存并检查网页源代码是否与你的代码相同

### 虚拟目录

tomcat的配置中Deploment的Application context是配置当前项目的虚拟目录，它是实际物理路径的映射。我们可以通过http://localhost:8080/WebPathDemo_war_exploded/demo1/index1.jsp访问一个jsp，那么这个jsp必须在我的本地物理路径上存在，才能访问成功，那么tomcat究竟去哪里寻找这个jsp呢？

答案就在编译后产生的artifacts中，tomcat将实际物理路径映射成了简单的虚拟目录，可以更方便的访问物理路径

![在这里插入图片描述](https://s2.loli.net/2023/04/09/Fal5fWBY4kpMw1K.png)

### Javaweb中如何读入配置文件问题

在此我们采用JDBC的druid工具类进行举例

```java
public class JDBCUtilsByDruid {

    private static DataSource dataSource;

    public static void main(String[] args) throws SQLException {
        Connection connection = JDBCUtilsByDruid.getConnection();
        System.out.println(connection);
    }
    //在静态代码块完成 dataSource初始化,注意此处为了好看一点已省略try-catch
    static {
        	Properties properties = new Properties();
        //JavaSe的正常情况直接读入
          properties.load(new FileInputStream("src\\druid.properties"));
        //如果是JavaWeb方式访问的话不能使用FileInputStream,要先获取当前类的加载器的路径再获取配置资源
            properties.load(JDBCUtilsByDruid.class.getClassLoader().getResourceAsStream("druid.properties"));
            dataSource = DruidDataSourceFactory.createDataSource(properties);
        
    }
```

这是一张真正的web路径所在的out文件夹下的结构

![image-20230411115856025](https://s2.loli.net/2023/04/11/y59jYDgNfc1oQVp.png)

1. 就是JDBCUtilsByDruid.class.getClassLoader()所获取的classes
2. 那么刚刚获取的classes.getResourceAsStream("druid.properties")才是真正的配置文件

### 琐碎知识点

- java中通过调用api等获取到的路径path是最后带 / 的，它代表是一个路径而不是资源

- 在路径问题中，/ 和 \ 都可以用作路径分隔符，用于分隔路径中的各个部分。但是，它们在不同的操作系统中的使用方式不同。

  在类 Unix 系统（如 Linux 和 macOS）中，/ 是标准的路径分隔符。例如，一个绝对路径可能是 /home/user/documents/file.txt。

  而在 Windows 系统中，\ 是标准的路径分隔符。例如，一个绝对路径可能是 C:\Users\User\Documents\file.txt。

  但是！！！在 Java 程序中，\ 是一个转义字符，为了避免出现转义问题。所以可以在代码中使用 / 作为路径分隔符。Java 会自动将它转换为当前操作系统的标准路径分隔符。


本文参考的原文链接：https://blog.csdn.net/qq_42495847/article/details/105706900