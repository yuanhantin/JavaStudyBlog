# Spring

**Spring 核心学习内容 IOC、AOP、 JdbcTemplate、声明式事务**

![image-20230327175118115](https://s2.loli.net/2023/04/20/cij1g2PSZn4leQ8.png)

## 1.Spring 几个重要概念 

1. Spring 可以整合其他的框架(Spring 是管理框架的框架) 
2. Spring 有两个核心的概念: IOC 和 AOP
3. IOC Inversion Of Control 控制反转
4. 动态代理（学好了才能学好AOP）
5. AOP Aspect-oriented programming 面向切面编程

那么接下来我们就开始逐个击破他们吧

## 2.IOC    / DI（ 注入依赖）

我们传统的开发模式是：编写程序 ==> 在程序中读取配置文件信息，通过new或反射创建对象，用对象完成任务

IOC开发模式：在配置文件中配置好对象的属性和依赖 ==> 在程序中利用IOC直接根据配置文件，创建对象， 并放入到容器(ConcurrentHashMap)中, 并可以完成对象之间的依赖 ，当需要使用某个对象实例的时候, 就直接从容器中获取即可（那么容器到底是什么呢，我们待会再讲）

总结：由传统的new、反射创建对象  ==> IOC通过注解或配置直接创建对象

### 有什么好处呢？

- 程序员可以更加关注如何使用对象完成相应的业务
- IOC 也体现了 Spring 最大的价值，通过配置，给程序提供需要使用的web层对象（Servlet、Service、Dao、JavaBean、Entity），实现解耦。

## 3.Spring容器结构与机制

我们先快速入门一下使用IOC基本的代码

如上文步骤所说，先配置一个xml文件，当然配之前我们要先创建一个类，这样才能在配置xml的class中输入类的全路径

所以我们先简单的创建一个Monster类

```java
package com.spring.bean;

public class Monster {
    private Integer id;
    private String name;
    private String skill;

    @Override
    public String toString() {
        return "Monster{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", skill='" + skill + '\'' +
                '}';
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSkill() {
        return skill;
    }

    public void setSkill(String skill) {
        this.skill = skill;
    }

    public Monster(Integer id, String name, String skill) {
        this.id = id;
        this.name = name;
        this.skill = skill;
    }

    public Monster() {
    }
}

```

然后配置我们的xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--
        1.配置Monster对象/javabean
        2.在beans中可以配置多个bean
        3.bean就是一个java对象,只是符合几个条件而已
        4.class属性是类的全路径,记得类要有无参构造器,就可以让spring底层使用反射
        5.id 表示 该类在spring容器中的id,将来可通过id获取该对象（id一定要不同，后面会解释）
        6.property是用于给该对象属性赋值
    -->
    <bean class="com.spring.bean.Monster"  id="monster01">
        <property name="id" value="1"/>
        <property name="name" value="牛魔王"/>
        <property name="skill" value="芭蕉扇"/>
    </bean>
</beans>
```

然后我们再简单的写一个测试类展示一下基本代码

```java
public class springBeanstest {
    @Test
    public void getMonster(){
        //1.创建容器 --> 和容器配置文件关联（debug处）
        ApplicationContext ioc = new ClassPathXmlApplicationContext("beans.xml");
        //2.通过getBean获取对象
        //Object monster01 = ioc.getBean("monster01");  //默认返回obj
        //也可以使用另一个getBean方法直接获取他的运行类型的对象,这样就无需强转获取monster的方法
        Monster monster01 = ioc.getBean("monster01", Monster.class);
        System.out.println(monster01.getSkill());
    }
}
```

那么简单的介绍了一下代码之后，我们正式开始分析ioc容器的底层机制，怎么办呢，毫无疑问的方法 ==> debug

断点打在第一行的创建容器上，让我们开始进入ioc的内部世界。

![image-20230420150328709](https://s2.loli.net/2023/04/20/8voqBbnCtuy64Ye.png)

点开我们的beanFactory，找到我们重点关注第3个：beanDefinitionMap，我们可以看到它的右边灰色括号内（就是它的类型）是一个ConcurrentHashMap，打开它我们看到有一个table，table的类型是ConcurrentHashMap的一个内部类Node，而且还是一个数组，每一个数组都存放着配置文件中的不同bean对象，它的初始化大小是512个，超过才会自动扩容

![image-20230420150522823](https://s2.loli.net/2023/04/20/DrLyvXzOExhnFZA.png)

我们往下翻，终于找到第217个数组存放着我们的Monster01对象

![image-20230420151357665](https://s2.loli.net/2023/04/20/y9EAjhZceFTiC46.png)

我们继续往下翻，找到一个propertyValues

![image-20230420151442784](https://s2.loli.net/2023/04/20/91Cmv4riXABnJfU.png)

其实也就是指xml里的property

```xml
<bean class="com.spring.bean.Monster"  id="monster01">
        <property name="id" value="1"/>
        <property name="name" value="牛魔王"/>
        <property name="skill" value="芭蕉扇"/>
    </bean>
```

好，hold on，hold on，hold on，所以说我现在考你一个问题，真真正正创建出来的monster01到底放在哪？你可能会说在beanDefinitionMap的table中，错，其实beanDefinitionMap的table存的只是beans.xml配置文件的对象，最终创建出来的monster01放在接下来的singletonObjects中

![image-20230420152229708](https://s2.loli.net/2023/04/20/ysBbqmrJV8AInZF.png)

![image-20230420152317844](https://s2.loli.net/2023/04/20/bEsPpMFYNGHdKXw.png)

到此，我们终于找到了创建出来的monster01最终存放的地方，在beanFactory的singletonObjects的table里。我们可以看到它的类型雀雀实实是Monster了

那么我们来解释一下这行代码，getBean是怎么查到monster01的

```java
Monster monster01 = ioc.getBean("monster01", Monster.class);
```

### getBean是怎么查到monster01的

- 首先，他会在beanDefinitionMap的table里找是否有monster01
- 如果发现有一个单例的对象，就又用monster01这个id去singletonObjects的table里获取真正的monster01。
- 如果发现不是单例的对象，他就通过反射机制动态创建一个单例的对象返回给ioc.getBean

说了这么多，肯定有小伙伴不知道什么叫单例，具体可以去网上了解，这里简要介绍一下，默认情况下，一个Spring容器的每一个bean对象都只能有一个实例对象，创建的再多也只有一个相同的对象，这就叫单例，这同时也是一个设计模式叫单例模式。

当然你要是写两遍这个代码，monster01 就不等于 monster02了，因为是两个spring容器

```java
ApplicationContext ioc = new ClassPathXmlApplicationContext("beans.xml");//创建容器
Monster monster01 = ioc.getBean("monster01", Monster.class);			//获取对象

ApplicationContext ioc = new ClassPathXmlApplicationContext("beans.xml");//创建容器
Monster monster02 = ioc.getBean("monster01", Monster.class);			//获取对象
```

到这里，我们也验证了上面配置xml文件时为什么说每个bean的id要不同的原因了。（如果相同创建的一直是同一个对象）

最后再额外补充一个

![image-20230420160541218](https://s2.loli.net/2023/04/20/P2oWq6r3RwvOiGh.png)

### Spring容器结构总结：

![image-20230420212327288](https://s2.loli.net/2023/04/20/BqMWNRjhIFuTbi1.png)

### Bean的生命周期：

说明: bean 对象创建是由 JVM 完成的，然后执行如下方法

1. 执行构造器 
2. 执行 set 相关方法
3.  调用 bean 的初始化的方法（需要配置）
4. 使用 bean 
5. 当容器关闭时候，调用 bean 的销毁方法（需要配置）

```java
public class House {
	private String name;
	public House() {
		System.out.println("House() 构造器");
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		System.out.println("House setName()...");
		this.name = name;
	}
	public void init() {	//不一定要叫init，只要在xml里配置init-method这里的方法名就好
		System.out.println("House init()..");
	}
	public void destory() {	 //同理，不一定要叫destory
		System.out.println("House destory()..");
	}
}

//输出顺序：
    House() 构造器  
    House setName()... 
    House init()..  
    业务操作  
    House destory()..
```

```xml
<!-- 配置 bean 的初始化方法和销毁方法 -->
<bean id="house" class="com.spring.beans.House"
init-method="init" destroy-method="destory">
<property name="name" value="北京豪宅"/>
</bean>
```

### 后置处理器：

后置处理器会在 bean **初始化init方法调用前**和**初始化init方法调用后**被调用，对所有的类都生效，这也是切面编程的核心之一(在不影响源代码情况下切进来,对多个对象进行操作)，后面我们会细讲Spring是如何实现他的。



## 4.实现sping的底层的注解方式注入bean（重点！！！）

1. 写一个简单的 Spring 容器 , 通过读取类的注解 (@Component @Controller @Service @Reponsitory)，将对象注入到 IOC 容器
2. 也就是说，不使用 Spring 原生框架，我们自己使用 IO+Annotaion+反射+集合 技术实现, 打通 Spring 注解方式开发的技术痛点

### 思路分析图：

那么开发一个程序首先做的就是画图分析思路啦

- 我们用虚线分隔开官方实现和我们自己实现的两个部分，上面的是官方的实现，下面是我们自己的。
- 我们将beans.xml分成两部分从而实现替代，一个是自定义注解（ComponentScan），一个是类似beans.xml的配置类（HspSpringConfig），配置类会写上我们的自定义注解，注解里的value就相当于xml文件中的base-package的作用，由此我们的配置类（HspSpringConfig）就达到了替代beans.xml的作用
- 那么我们再来实现自己的容器类（HspSpringApplication）作为官方的ClassPathXmlApplicationContext的替代品，思路很简单，不过就是拿到到配置类的.class文件，然后获取自定义注解下的value值（也就是要扫描的包的全路径），通过反射包的全路径获取包下的各种class文件（编译后的java类），再进行一系列操作实现getBean的平替方法

![image-20230423203909952](https://s2.loli.net/2023/04/23/1Q8kKtr7UNVSJwz.png)

![image-20230423205911453](https://s2.loli.net/2023/04/23/xzMwkSBlhVRQCXr.png)

### 实现：

思路分析完了，可能现在小伙伴就开始无从下手了，那么我们首先开始照着图片搭建一个框架，也就是在IDEA中创建好包和类先。其中Component就是写了一些自定义注解的类（也就是要扫描的包），test是测试用的

![image-20230423232008008](https://s2.loli.net/2023/04/23/r14oUnTxYHhptSf.png)

于是乎，我们就可以开始辛苦的敲代码了

先从beans.xml的平替类下手，自定义注解和配置类

```java
package com.spring.annotation;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * @Author: Yjc
 * @Description: 自定义注解
 * @DateTime: 2023/4/23 20:23
 **/
@Target(ElementType.TYPE)          // @Target 注解用于定义注解的使用范围，即被描述的注解可以用在什							        么地方。ElementType.TYPE 表示该注解可以用于类、接口、枚举和注解类型
@Retention(RetentionPolicy.RUNTIME)         //作用范围是运行时
public @interface ComponentScan {
    String value() default "";              //可以给一个叫value的字符串 , 也就是可以@ComponentScan(value = "xxx")这样使用
}
```

```java
package com.spring.annotation;

/**
 * @Author: Yjc
 * @Description: 配置类
 * @DateTime: 2023/4/23 20:29
 **/
@ComponentScan(value = "com.spring.Component")
public class yuanSpringConfig {
}
```

再写几个简单的组件类用来测试，为了简洁我这就放一起了，实际上拆开放在各自的类里

```java
@ComponentScan
public class MyComponent {
}

public class NoComponent {		//没有注解的类，后面要写逻辑来排除
}

@Repository
public class UserDao {
}

@Service
public class UserService {
}

@Controller
@Component(value = "id1")		//相当于给UserServlet命名为id1，最后测试getBean是否能够获取它
public class UserServlet {
}
```

然后就是我们最重要的容器类了，但是我们这里先实现一半，防止朋友们过于困惑，我们分段实现

```java
package com.spring.annotation;

import java.io.File;
import java.lang.annotation.Annotation;
import java.net.URL;
import java.util.Objects;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @Author: Yjc
 * @Description: 容器类
 * @DateTime: 2023/4/23 21:18
 **/
public class yuanSpringApplicationContext {
    private Class configClass;
    private final ConcurrentHashMap<String, Objects> ioc = new ConcurrentHashMap<>();      //我们之前分析过了，其实ApplicationContext底层就是一个ConcurrentHashMap

    public yuanSpringApplicationContext(Class configClass) {
        //获取传入的配置类yuanSpringConfig.class
        this.configClass = configClass;
        //获取传入的配置类的注解
        ComponentScan ComponentScan = (ComponentScan) this.configClass.getDeclaredAnnotation(ComponentScan.class);
        String path = ComponentScan.value();

        //扫描获取的value（包）下的所有文件
        //1.获取类加载器
        ClassLoader classLoader = yuanSpringApplicationContext.class.getClassLoader();
        //2.通过类加载器获取包下文件的url
        path = path.replace(".","/");       //将.全部改成 /  这样才是路径
        URL resource = classLoader.getResource(path);     //注意:通过阅读源码，这个空里的路径应该是以/为分割，所以有了上一句话

        File file = new File(resource.getFile());         //IO中目录也可以当成一个文件
        if (file.isDirectory()){
            File[] files = file.listFiles();              //获取包下的所有文件
            for (File f : files) {
                System.out.println("===========");       //检查一下有没有拿到
                File absoluteFilePath = f.getAbsoluteFile();
                System.out.println(absoluteFilePath);
            }
        }
    }
}

```

写完这些，我们写个测试用例试一下，是否拿到了要扫描的包下的资源

```java
package com.spring.test;
import com.spring.annotation.yuanSpringApplicationContext;
import com.spring.annotation.yuanSpringConfig;
import org.junit.Test;

/**
 * @Author: Yjc
 * @Description: 测试用例
 * @DateTime: 2023/4/23 21:26
 **/
public class SpringTest {
    @Test
    public void testspring(){
        //调用容器类的构造器，放入配置类的class类。
        yuanSpringApplicationContext ioc = new yuanSpringApplicationContext(yuanSpringConfig.class);
    }
}
```

很显然，我们成功了

![image-20230423215246778](https://s2.loli.net/2023/04/23/3sUrkRDSC2laNQj.png)

那么我们接下来继续实现，想办法拿到类的全路径从而通过反射去除没有web组件注解的类

```java
package com.spring.annotation;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;
import java.io.File;
import java.lang.annotation.Annotation;
import java.net.URL;
import java.util.Objects;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @Author: Yjc
 * @Description: 容器类
 * @DateTime: 2023/4/23 21:18
 **/
public class yuanSpringApplicationContext {
    private Class configClass;
    private final ConcurrentHashMap<String, Object> ioc = new ConcurrentHashMap<>();      //我们之前分析过了，其实ApplicationContext底层就是一个ConcurrentHashMap

    public yuanSpringApplicationContext(Class configClass) {
        //获取传入的配置类yuanSpringConfig.class
        this.configClass = configClass;
        //获取传入的配置类的注解
        ComponentScan ComponentScan = (ComponentScan) this.configClass.getDeclaredAnnotation(ComponentScan.class);
        String path = ComponentScan.value();

        //扫描获取的value（包）下的所有文件
        //1.获取类加载器
        ClassLoader classLoader = yuanSpringApplicationContext.class.getClassLoader();
        //2.通过类加载器获取包下文件的url
        path = path.replace(".", "/");       //将.全部改成 / ，这样才是路径
        URL resource = classLoader.getResource(path);     //注意:通过阅读源码，这个空里的路径应该是以/为分割

        File file = new File(resource.getFile());         //IO中目录也可以当成一个文件
        if (file.isDirectory()) {
            File[] files = file.listFiles();              //获取包下的所有文件
            for (File f : files) {
                System.out.println("===========");       //检查一下有没有拿到
                String absoluteFilePath = String.valueOf(f.getAbsoluteFile());
                System.out.println(absoluteFilePath);

                //E:\JavaProject\Spring\out\production\Spring\com\spring\Component\MyComponent.class
                //获取com.spring.Component.MyComponent

                //1.获取类名MyComponent，也就是想办法去掉.class和前面的
                String className = absoluteFilePath.substring(absoluteFilePath.lastIndexOf("\\") + 1,
                        absoluteFilePath.lastIndexOf(".class"));
                //2.想办法拿到com.spring.Component.   其实只要把获取到的注解拿过来就好了（也就是path）
                String classFullName = path.replace("/", ".") + "." + className;
                System.out.println(classFullName);
                //3.把没有加自定义注解Component和相关的web组件注解的类去掉
                try {
                    //通过类加载器反射获取该类的class对象
                    //那么有人可能会问,为什么不用forName方法反射呢,因为forName还会调用static静态方法
                    //而classLoader只是获取class对象的信息，相当于一个轻量级的反射
                    //Class<?> classForName = Class.forName(classFullName);

                    Class<?> aClass = classLoader.loadClass(classFullName);
                    if (aClass.isAnnotationPresent(ComponentScan.class)
                            || aClass.isAnnotationPresent(Controller.class)
                            || aClass.isAnnotationPresent(Service.class)
                            || aClass.isAnnotationPresent(Repository.class)) {
                        //这个时候我们就需要完整的反射,如果该类有静态方法就可以执行了
                        Class<?> classForName = Class.forName(classFullName);
                        Object newInstance = classForName.newInstance();
                        //将其放入容器中
                        ioc.put(className, newInstance);
                    }else {
                        System.out.println("有一个不是组件的类");
                    }
                } catch (Exception e) {
                    System.out.println("出错了");
                }
            }
        }
    }
}
```

经过测试，冇问题。

![image-20230423232558375](https://s2.loli.net/2023/04/23/Jh1qI4DgkKZXLnb.png)

至此，我们已经可以将对象放入ioc容器中了，所以我们再写个简单的getBean方法作为获取实例对象的getter吧

```java
public Object getBean(String name){
        return ioc.get(name);
    }
```

可以再写个简单的测试类试用一下

```java
package com.spring.test;
import com.spring.annotation.yuanSpringApplicationContext;
import com.spring.annotation.yuanSpringConfig;
import org.junit.Test;

/**
 * @Author: Yjc
 * @Description: 测试用例
 * @DateTime: 2023/4/23 21:26
 **/
public class SpringTest {
    @Test
    public void testspring()  {
        yuanSpringApplicationContext ioc = new yuanSpringApplicationContext(yuanSpringConfig.class);
        System.out.println(ioc.getBean("id1"));
    }
}

```

![image-20230424132004495](https://s2.loli.net/2023/04/24/HbFsadR3qYScZz4.png)

那么到此为止，一个简单的Spring注解的过程就实现完毕了。以后见到注解就知道它是怎么来的，有脚踏实地的感觉了😏以后还会手写Spring或者其他框架（比如Tomcat、Dubbo）之类的一些底层机制，把基础打牢才是最重要的。

## 5.自动装配

### @Autowired规则说明 

1) 在 IOC 容器中查找待装配的组件的类型，如果有唯一的 bean 匹配，则使用该 bean 装配 
2) 如待装配的类型对应的 bean 在 IOC 容器中有多个，则使用待装配的属性的属性名作为 id 值再进行查找, 找到就装配，找不到就抛异常

### @Resource 规则说明 

- @Resource 有两个属性是比较重要的，分别是 name 和 type
  1) Spring 将 name 属性解析为 bean 的名字，而 type 属性解析为 bean 的类型。
  2) 如果使用 name 属性，则使用 byName 的自动注入策略，而使用 type 属性时则使用 byType 自动注入策略。
- 如果@Resource 没有指定 name 和 type 
  1) 则先使用byName注入策略
  2) 如果匹配不上， 再使用 byType 策略
  3) 如果都不成功，就会报错 
- 不管是@Autowired 还是 @Resource 都保证属性名是规范的写法就可以注入。不过一般使用@Resource没有指定 name 和 type 的形式即可

到了这里肯定有小伙伴听不懂，那么我们来举个例子

先配置好xml信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
   
    <context:component-scan base-package="com.spring.Component"/>	 //自动装配com.spring.Component下的所有符合条件的类

</beans>
```

配个最底层的Dao层，然后在Service层自动装配Dao层，并调用Dao的方法，如果调用成功则说明自动装配OK

```java
package com.spring.Component;
import com.spring.annotation.ComponentScan;
import org.springframework.stereotype.Repository;

/**
 * @Author: Yjc
 * @Description: Dao层
 * @DateTime: 2023/4/23 20:33
 **/
@Repository
public class UserDao {
    public void testUserDao(){
        System.out.println("UserDao Ok");	//如果输出了就说明自动装配OK
    }
}
```

```java
package com.spring.Component;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;

/**
 * @Author: Yjc
 * @Description: Service层
 * @DateTime: 2023/4/23 20:34
 **/
@Service
public class UserService {
    @Resource
    private UserDao userDao;	//这里就是所谓的自动装配了，省略了new的动作，因而我们可以不这里浪费时间想怎么创建对象了
    public void printuserDao(){
        userDao.testUserDao();
    }
}
```

写个简单的测试类

```java
package com.spring.test;

import com.spring.Component.UserService;
import com.spring.annotation.yuanSpringApplicationContext;
import com.spring.annotation.yuanSpringConfig;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @Author: Yjc
 * @Description: TODO
 * @DateTime: 2023/4/23 21:26
 **/
public class SpringTest {
    @Test
    public void testAutowired()  {
        ApplicationContext ioc = new ClassPathXmlApplicationContext("beans3.xml");
        UserService userService = (UserService) ioc.getBean("userService");
        System.out.println(userService);
        userService.printuserDao();
    }
}
//输出结果当然是UserDao Ok，说明自动装配成功
```

## 6.泛型依赖注入

举一个例子:如何将PhoneDao 装配到 PhoneSerive 中?

传统方法是将 PhoneDao /BookDao 自动装配到 BookService/PhoneSerive 中，如下图：

![image-20230426132529400](C:\Users\windows\AppData\Roaming\Typora\typora-user-images\image-20230426132529400.png)

就像这样，虽然没有任何的问题，但是东西一旦多了起来，就开始麻烦起来，此时可以使用 spring 提供的泛型依赖注入，如下图：

![image-20230426141904040](https://s2.loli.net/2023/04/26/jblowKkpNSaA3Cx.png)

如何实现呢？

- 让BaseDao和BaseService成为一个泛型类，让Dao和Service继承Base时传入相应的类
- 将装配的过程向上提一个档次，将在每个Service装配Dao提到BaseService装配BaseDao
- 此时Spring底层会自动通过类似上图的方式自动将PhoneDao装配至PhoneService

我们来代码实现一下，同样，先建好总框架，我们这里只演示Phone的

![image-20230426142902987](https://s2.loli.net/2023/04/26/1l5BoIswtHnDMWA.png)

```java
public abstract class basicDao <T> {
    public abstract void call();
}

public class basicService <T>{
    @Autowired
    private basicDao<T> basicDao;

    public void Service(){
        basicDao.call();
    }
}
```

```java
@Repository
public class PhoneDao extends basicDao<Phone>{
    @Override
    public void call() {
        System.out.println("phoneDao");
    }
}

@Service
public class Phoneservice extends basicService<Phone>{
    @Override
    public void Service() {			//甚至这段都可以不写，最后也可以调用
        super.Service();
    }
}

```

我们再来写个测试类看看

```java
public class SpringTest {
    @Test
    public void testGeneric()  {
        ApplicationContext ioc = new ClassPathXmlApplicationContext("Generic.xml");
        Phoneservice phoneservice = ioc.getBean("phoneservice", Phoneservice.class);//前面讲过未装配id时，默认为首字母小写的类名
        phoneservice.Service();
    }
}

//最后输出结果为phoneDao，可以看出我们实现了泛型装配,使Service调用到了Dao类的call方法
```

## 7.动态代理(手写简易AOP底层机制)（重点！！！）

问题出现：如何解决，在某一个方法调用前和调用后都执行一些业务逻辑 ==>动态代理机制

```
我们来解决一个实际问题吧：
写2个方法getSum()、getSub()可以完成简单的加减法, 要求在执行 getSum()和 getSub()时，
输出：执行前，执行后，执行完毕的日志，格式如下。

执行前-传入的方法参数  [2, 1]
执行后-结果  1
方法执行完毕

注意：同时要可以catch
```

还是老方法，先写框架，再添细节，我们动态代理的核心三件套，接口，要被代理的类，代理提供类

![image-20230427113729931](https://s2.loli.net/2023/04/27/PUhZoByA61siCLW.png)

我们动态代理必须使用接口，因为一个类不能同时继承多个类但是可以继承多个接口，如果不使用接口的话我们动态代理就变成只能代理一个了，那还要动态代理做什么呢。

先写个接口和实现类

```java
public interface get {
    public int getsub(int a, int b);
    public int getsum(int a, int b);
}
```

```java
public class log implements get{
    @Override
    public int getsub(int a, int b) {
        return a-b;
    }

    @Override
    public int getsum(int a, int b) {
        return a+b;
    }
}
```

好，接下来的代理提供类就是我们的重头戏了，3个需要实现的重点

- 一个代理对象属性（代理的类的接口）

- 构造器

- 一个getProxy方法用来获取动态代理对象，里面又有4个重要步骤

  1. 通过 Proxy.newProxyInstance（） 实例化一个代理对象 ，里面有三个参数，按照下面的3个步骤获取
  2. 获取代理对象的 classLoader  ——类加载器，用于反射
  3. 获取代理对象的 interfaces      ——代理对象接口（也就是最开始写的那个接口啦）
  4. 一个匿名内部类InvocationHandler  —— InvocationHandler  其实是一个接口它里面只有一个方法叫invoke用于动态代理，但是我们知道接口正常是不能变成实例化成一个对象的，所以只能通过匿名内部类变成一个对象，从而作为一个参数传入上面的 Proxy.newProxyInstance（） 

  

  说了怎么多，我们来具体实现一下上面的3大重点和4个步骤吧

```java
/**
 * @Author: Yjc
 * @Description: 代理提供类
 * @DateTime: 2023/4/27 10:48
 **/
public class logProxyProvider  {
    private get Get;            //代理对象  坑！！！ 这里应当是要代理的类的接口

    public logProxyProvider(log Get) {
        this.Get = Get;
    }

    public get getProxy(){
        ClassLoader classLoader = Get.getClass().getClassLoader();
        Class<?>[] interfaces = Get.getClass().getInterfaces();
        
        InvocationHandler invocationHandler = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                Object result = null;
                try {
                    //横切关注点-前置
                    System.out.println("日志-执行前-传入的方法参数 " + Arrays.asList(args));
                    result = method.invoke(Get, args);
                    //横切关注点-后置
                    System.out.println("日志-执行后-结果 " + result);
                    return result;
                } catch (Exception e) {
                    System.out.println(method.getName() + "方法执行异常 异常类型为 " + e.getClass().getName());
                } finally {
                    System.out.println("方法执行完毕");
                }
                return result;
            }
        };
        get proxyInstance = (get)Proxy.newProxyInstance(classLoader,interfaces,invocationHandler);	
        //注意这里要强制转换成代理对象，否则return的proxyInstance使用不了被代理类的方法，因为默认编译类型是object
        return proxyInstance;
    }
}
```

ok，这就是一个完整的动态代理了，写个测试类试试吧

```java
/**
 * @Author: Yjc
 * @Description: TODO
 * @DateTime: 2023/4/26 22:47
 **/
public class proxyTest {
    @Test
    public void logProxyTest(){
        log log = new log();
        logProxyProvider logProxy= new logProxyProvider(log);
        get proxy = logProxy.getProxy();
        proxy.getsub(2,1);
    }
}
/**		
最后输出：
日志-执行前-传入的方法参数 [2, 1]
日志-执行后-结果 1
方法执行完毕
*/
```

## 8.AOP

什么是 AOP ?  AOP 的全称(aspect oriented programming) ，面向切面编程

其实刚刚我们在说动态代理时已经讲过了为什么需要aop，也就是想要在某一个方法的调用前和调用后时执行一些业务逻辑，比如说在用户登录前验证密码，在用户登录成功后干嘛干嘛等等。。。

**底层思想是代理思想，它可以通过不修改源程序实现业务功能的拓展**

总的来说，有4个可以切入的点

- 调用前
- 调用后
- catch到异常后
- finally最后位置

那么相对应的AOP框架是如何实现上面四个东西的呢，第一是注解，第二是xml配置文件

### 注解：

以下四个就是4个切入点对应的在切面类中声明的通知方法

- 前置通知：@Before 
- 返回通知：@AfterReturning
- 异常通知：@AfterThrowing 
- 后置通知：@After 
- 环绕通知：@Around

那么多的这个环绕通知其实就是一次性实现上述的4个切入功能的注解，在上述4个都需要实现时可以使用，从而减少代码量

那么我们来具体实现一下案例吧

首先新建你的xml文件，加入下面两句话，否则无法启用aop框架

```xml
<context:component-scan base-package="com.spring.Aop"/>			//第一步扫描你要的包
<aop:aspectj-autoproxy/>									//启用aop注解类框架
```

包里面的类和接口

![image-20230505154953992](https://s2.loli.net/2023/05/05/IfPMw6vlmskGNc3.png)

第二步，像之前一样，给除了TEST类以外的类 都配上@Component，这样我们才能将类注入spring容器（前面已经讲过了），并实现usbInterface接口

```java
public abstract interface UsbInterface {
    void work();
}
```

```java
@Component
public class Camera implements  UsbInterface{
    @Override
    public void work() {
        System.out.println("Camera......");
    }
}
```

```java
@Component
public class Phone implements UsbInterface {
    @Override
    public void work() {
        System.out.println("Phone......");
    }
}
```

第三步，也是最重要的一步来了，编写我们的  **切入类** 

```java
@Order(value = 1)		//下面会讲
@Component
@Aspect		//必须有这个注解才能被框架解析为  切入类
public class UsbAspect {
    @Before(value = "execution(public void com.spring.Aop.UsbInterface.work())")
    public void Before(JoinPoint joinPoint){
        Signature signature = joinPoint.getSignature();		
        System.out.println("前置通知——目标方法为= "+signature.getName());
    }
    @AfterReturning(value = "execution(public void com.spring.Aop.UsbInterface.work())")
    public void AfterReturning(JoinPoint joinPoint){
        Signature signature = joinPoint.getSignature();
        System.out.println("返回通知——目标方法为= "+signature.getName());
    }

    @AfterThrowing(value = "execution(public void com.spring.Aop.UsbInterface.work())")
    public void AfterThrowing(JoinPoint joinPoint){
        Signature signature = joinPoint.getSignature();
        System.out.println("异常通知——目标方法为= "+signature.getName());
    }
    @After(value = "execution(public void com.spring.Aop.UsbInterface.work())")
    public void after(JoinPoint joinPoint){
        Signature signature = joinPoint.getSignature();
        System.out.println("后置通知——目标方法为= "+signature.getName());
    }
}
```

我们的切入类中又有几个重要的知识点：

#### 切入表达式：

value里要填入我们的切入表达式：

execution( [权限修饰符] 返回值类型 包名.类名.方法名(参数列表))

![image-20230505161119445](https://s2.loli.net/2023/05/05/MnxkWmUcpqbOePI.png)

#### JoinPoint

通过 JoinPoint 可以获取到调用方法的签名
**常用api:**

| 方法名                    | 功能                                                         |
| :------------------------ | :----------------------------------------------------------- |
| Signature getSignature(); | ***\*获取封装了署名信息的对象,在该对象中可以获取到目标方法名,所属类的Class等信息\**** |
| Object[] getArgs();       | **获取传入目标方法的参数对象**                               |
| Object getTarget();       | **获取被代理的对象**                                         |
| Object getThis();         | **获取代理对象**                                             |

getSignature()之后又可以获取很多有用的信息，比如：

```java
	joinPoint.getSignature().getName(); 							// 获取目标方法名
	joinPoint.getSignature().getDeclaringType().getSimpleName(); // 获取目标方法所属类的简单类名
	joinPoint.getSignature().getDeclaringTypeName(); // 获取目标方法所属类的类名
	joinPoint.getSignature().getModifiers(); // 获取目标方法声明类型(public、private、protected)
	Object[] args = joinPoint.getArgs(); // 获取传入目标方法的参数，返回一个数组
	joinPoint.getTarget(); // 获取被代理的对象
	joinPoint.getThis(); // 获取代理对象自己
```



写到这里我们可能会切入表达式怎么要写这么多次，能不能重复使用他呢？哈哈当然可以

#### 切入点表达式重用技术

@Pointcut注解

```java
@Pointcut(value = "execution(public void com.spring.Aop.UsbInterface.work())")
public void Pointcut(){
}

@Before(value = "Pointcut()")
省略。。。
```

先用一个Pointcut注解固定好一个类，然后下面的直接在value中写入定义好的类即可。

写到这里可能有小伙伴会说，为什么切入类只实现了4个通知方式，还有一个环绕通知没写呢，因为环绕通知不能和上面4个通知一起使用，我们来实现一下吧。

### 环绕通知

```java
@Around(value = "execution(public void com.spring.Aop.UsbInterface.work())")
public Object Around(ProceedingJoinPoint joinPoint){
    Object result = null;
    String methodName = joinPoint.getSignature().getName();
    try {
        //1.相当于前置通知完成的事情
        Object[] args = joinPoint.getArgs();
        List<Object> argList = Arrays.asList(args);
        System.out.println("AOP环绕通知[前置通知]" + methodName + "方法开始了--参数有：" + argList);
        //在环绕通知中一定要调用joinPoint.proceed()来执行目标方法
        result = joinPoint.proceed();
        //2.相当于返回通知完成的事情
        System.out.println("AOP环绕通知[返回通知]" + methodName + "方法结束了--结果是：" + result);
    } catch (Throwable throwable) {
        //3.相当于异常通知完成的事情
        System.out.println("AOP环绕通知[异常通知]" + methodName + "方法抛异常了--异常对象：" + throwable);
    } finally {
        //4.相当于最终通知完成的事情
        System.out.println("AOP环绕通知[后置通知]" + methodName + "方法最终结束了...");
    }
    return result;
}
```

这里有2个重点

1. 类的参数中必须使用ProceedingJoinPoint类，而不能使用JoinPoint类了

   

2. 我们知道环绕通知代表了4个通知，但这里只有三个try-catch-finally，只有三个要这么实现4个通知呢，答案是在try代码块中必须有一句一定要调用  **joinPoint.proceed()**  来执行目标方法作为分割，这句话**上面的就是前置通知**，**下面的就是返回通知**。

### 切面优先级问题

如果同一个方法，有多个切面类在同一个切入点切入，那么执行的优先级如何控制呢，可以采用@order注解来控制 

@order(value=n)  n 值越小，优先级越高。只要加在切面类外面就好了，上面代码已经有演示过，这里不是重点。

#### 优先级执行顺序

重点在于这个地方，即我们不能把上面的优先级理解成：优先级高的每个通知类型都先执行，而是类似 Filter 的过滤链式调用机制

我们来画个图理解一下

![image-20230505184012825](https://s2.loli.net/2023/05/05/AGIPFLbMvkzphJy.png)

也就是说，除了前置通知确实是按照优先级顺序输出的，其他的通知其实是倒序输出的。（也就是优先级越高的返回、异常、最终通知都是越靠后输出来的）

### XML配置：

这里主要展示几个关键步骤，因为用的比较少就不细讲了。

```xml
<!--注册Bean-->
<bean id="student" class="com.org.entity.Student"/>

<!--配置aop-->
<aop:config>
    <!--配置切面-->
    <aop:aspect ref="qieMian">
        <!--切入点 excution(修饰词 返回值 类名 方法名 参数) ，excution代表要执行的位置-->
        <aop:pointcut id="point" expression="execution(* com.org.serviceImpl.StudentServiceImpl.*(..))"/>
        <!--通知-->
        <aop:before method="before" pointcut-ref="point"/>
        <aop:after method="after" pointcut-ref="point"/>
    </aop:aspect>
</aop:config>
```
到此，我们的AOP基本就学完了。

我们之前已经实现了sping的底层的注解方式注入bean和AOP动态代理的底层机制，但是还是有很多细节没有考虑到，只是一个过于简单的机制，为了更加了解spring底层，我们来继续相对完整的实现一个spring吧。

我们之前实现的spring还少了很多东西,比如说主要有以下几点

- ### Spring如何实现依赖注入及实现单例、多例不同的getBean机制的

- ### 原生 Spring 如何实现 BeanPostProcessor 的

- ### 原生 Spring 是如何实现 AOP的

## 9.继续实现spring底层机制

### Spring如何实现依赖注入及实现单例、多例不同的getBean机制的

把问题拆分一下，先来讲如何实现依赖注入，先看表象，我们所谓的依赖注入就是在代码中就是在注入的地方加一个自动装配的注解比如说@Autowired，那么我们分析一下如何将带有@Autowired的字段注入。首先是不是要先判断是否有@Autowired这个注解，如果有的话，我们就判断一下这个字段叫什么，然后将其实例化变成一个对象。实例化完过后呢要怎么放进去呢，毫无疑问是Java基础的反射机制了，通过反射的set方法将其注入即可。

好，我们再来看看如何实现单例和多例不同的getBean机制，我们想要将一个东西区分开来是不是需要保存他的信息，然后在需要的时候通过获取他的信息来判断不同的使用方式。所以我们需要一个类用来保存对象的信息，我们这里就叫这个类BeanDefinition好了，里面需要两个属性，一个用来存储单例还是多例，一个用来存储他的Class类型到时候用于反射。那么我们解决了存储问题，但是这只是一个，我们要如何存储多个组件的各自的BeanDefinition呢，毫无疑问使用数据结构，比如将其放入一个Map中就解决啦。

那么我们知道，单例多例的机制是不同的，单例应该在初始化时就创建成一个实例并保存好，多例则是在使用时才动态创建。所以我们要有一个用于存放单例的单例池，毫无疑问也是使用一个Map就可以了。为了防止多线程等问题我们就是用ConcurrentHashMap好了。

综上所述并画成一个图便于理解。

### Spring 整体架构分析图

![image-20230510161036875](https://s2.loli.net/2023/05/10/wXzPONyv41to3Qf.png)

由于代码过长，这里只分析下思路。需要代码的可以查看置顶随笔找我。

### 

### 原生 Spring 如何实现 BeanPostProcessor 的

先说为什么要实现后置处理器，因为Spring的AOP机制是由后置处理器+（判断是否执行）动态代理从而实现切面编程的。

我们先来简答介绍一个经常使用的编程思想：面向接口编程。也就是说我们有时候要判断一个机制是否实现时可以通过判断它是否实现了某个接口，如果实现了那个接口我们就实现刚刚判断的机制。有时候甚至这个接口里面没有任何方法，只是单纯的用于判断，这种接口又叫标记接口。

那么在这里我们就使用到了这个思想，先模仿Spring写一个接口InterBeanPostProcessor

```java
package com.spring.processer;

/**
 * @Author: Yjc
 * @Description: 用于判断的接口
 * @DateTime: 2023/5/9 18:16
 **/
public interface InterBeanPostProcessor {
    /**
     * 在Bean的初始化方法前调用
     */
    default Object BeforeInitialization(Object bean,String beanName){
        return bean;
    }

    /**
     * postProcessAfterInitialization在Bean的初始化方法后调用
     */
    default Object AfterInitialization(Object bean, String beanName) {
        return bean;
    }
}
```

然后我们来写一个类实现该接口，就叫他后置处理器类吧

```java
package com.spring.Component;

import com.spring.annotation.SelfComponent;
import com.spring.processer.InterBeanPostProcessor;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
* @Author: Yjc
* @Description: 后置处理器,对所有的类都生效,这里就是一个切面编程的核心(在不影响源代码情况下切进来,对多个对象进行操作)
* @DateTime: 2023/5/9 18:22
**/
@SelfComponent
public class BeanPostProcessor implements InterBeanPostProcessor {
    @Override
    public Object BeforeInitialization(Object bean, String beanName) {
        System.out.println("前置处理..");
        return InterBeanPostProcessor.super.BeforeInitialization(bean, beanName);
    }

    @Override
    public Object AfterInitialization(Object bean, String beanName) {    //后置处理+判断是否执行动态代理(Proxy) = AOP
        System.out.println("后置处理..");
        return InterBeanPostProcessor.super.AfterInitialization(bean, beanName);
    }
}
```

这个类其实也是一个组件，我们要在初始化的时候就将其实例化，同时为了考虑到可能会有多个后置处理类，所以也是用一个数据结构将其存储，我们就用个List作为一个后置处理类池需要时全部取出即可（为什么全部，因为多个后置处理器应当全部执行）。这里代码就省略了，要的私我即可。这里大致了解个思路就好。

然后就是在初始化前和初始化后执行所有的后置处理器，只需在初始化代码前后都遍历一次List后置处理类池并调用方法即可。不同的是初始化前调用的BeforeInitialization，初始化后调用AfterInitialization

### 原生 Spring 是如何实现 AOP的

其实AOP就是在后置处理器的机制上做些修改即可，核心就是因为每个组件类都会执行后置处理器，我们只需在后置通知的地方判断该类是否为我们需要切入的类，如果是，就通过动态代理机制执行该类的目标方法，如果不是就返回正常的对象即可。

```java
package com.spring.Component;

public interface SmartAnimalable {
    float getSum(float i, float j);

    float getSub(float i, float j);
}
```

```java
package com.spring.Component;

import com.spring.annotation.SelfComponent;

@SelfComponent(value = "smartDog")
public class SmartDog implements SmartAnimalable {
    public float getSum(float i, float j) {
        float res = i + j;
        System.out.println("SmartDog-getSum-res=" + res);
        return res;
    }

    public float getSub(float i, float j) {
        float res = i - j;
        System.out.println("SmartDog-getSub-res=" + res);
        return res;
    }
}
```

```java
package com.spring.Component;

import com.spring.annotation.SelfComponent;
import com.spring.processer.InterBeanPostProcessor;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
* @Author: Yjc
* @Description: 后置处理器,对所有的类都生效,这里就是一个切面编程的核心(在不影响源代码情况下切进来,对多个对象进行操作)
* @DateTime: 2023/5/9 18:22
**/
@SelfComponent
public class BeanPostProcessor implements InterBeanPostProcessor {
    @Override
    public Object BeforeInitialization(Object bean, String beanName) {
        System.out.println("前置处理..");
        return InterBeanPostProcessor.super.BeforeInitialization(bean, beanName);
    }

    @Override
    public Object AfterInitialization(Object bean, String beanName) {       //后置处理+判断是否执行动态代理(Proxy) = AOP
        System.out.println("后置处理..");
        //实现AOP,返回代理对象(即对bean进行包装)

        if("smartDog".equals(beanName)){
            Object proxyInstance = Proxy.newProxyInstance(BeanPostProcessor.class.getClassLoader(), bean.getClass().getInterfaces(), new InvocationHandler() {
                Object res = null;
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    System.out.println("method = " +method.getName());
                    if("getSum".equals(method.getName())){      //假如发现我们的方法是getSum,就进行前置通知处理
                        SmartAnimalAspect.showBeginLog();       //进行前置通知处理
                        res = method.invoke(bean,args);         //然后调用目标方法,也就是getSum
                        SmartAnimalAspect.showSuccessLog();     //后置通知处理
                    }else {                                     //如果不是getSum目标方法
                        res = method.invoke(bean,args);         //直接调用目标方法,也就是getSum
                    }
                    return res;
                }
            });
            return proxyInstance;       //返回代理对象
        }
        //如果不需要代理,则返回原生对象
        return InterBeanPostProcessor.super.AfterInitialization(bean, beanName);
    }
}
```



## 10.JdbcTemplate

如果程序员就希望使用 spring 框架来做项目，spring 框架如何处理对数据库的操作呢?

方案 1. 使用之前做项目开发的 JdbcUtils 类

方案 2. 其实 spring 提供了一个操作数据库(表)功能强大的类 JdbcTemplate 。我们可以同 ioc 容器来配置一个 jdbcTemplate 对象，使用它来完成对数据库表的各种操作

基本介绍

1. 通过 Spring 可以配置数据源(可以简单理解为就是配置JDBC中的c3p0或德鲁伊)，从而完成对数据表的操作

2. JdbcTemplate 是 Spring 提供的访问数据库的技术。可以将 JDBC 的常用操作封装为模板方法。

展示一下api，可以看出有很多功能非常强大的操作，比如说设置查询超时时间等等

![image-20230512111440143](https://s2.loli.net/2023/05/12/J2Q8I7AbRsehlrp.png)

导入相应的jar包之后我们来实操一下吧，不是很难这里就不写的太详细了。

创建配置文件 jdbc.properties 	（放在src下）

```properties
jdbc.user=root
jdbc.pwd=你的数据库密码
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:你的数据库端口/数据库名字

例如我的
jdbc.user=root
jdbc.pwd=hanlinyuan
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring
```

配置文件 JdbcTemplate_ioc.xml（放在src下）

这里配置两个东西一个是数据源（c3p0 or 德鲁伊等等），一个是JdbcTemplate，然后将c3p0数据源分配给JdbcTemplate（可以使JdbcTemplate使用c3p0连接池管理数据库连接，提高应用程序的性能和扩展性)

```xml
    <!--引入上面配置的jdbc.properties文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <!--配置数据源对象-DataSoruce，这里使用c3p0-->
    <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
        <!--给数据源对象配置属性值-->
        <property name="user" value="${jdbc.user}"/>	--${jdbc.user}会自动找到properties中的信息
        <property name="password" value="${jdbc.pwd}"/>
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
    </bean>
    
    <!--配置JdbcTemplate对象，将c3p0 数据源分配给 JdbcTemplate bean-->
<bean class="org.springframework.jdbc.core.JdbcTemplate" id="jdbcTemplate">
    <!--给JdbcTemplate对象配置dataSource-->
    <property name="dataSource" ref="dataSource"/>
</bean>
```

整个表出来用于教学  创建数据库 spring 和表 monster   

```sql
-- 创建数据库
CREATE DATABASE spring
USE spring
-- 创建表 monster
CREATE TABLE monster(
id INT PRIMARY KEY, 
`name` VARCHAR(64) NOT NULL DEFAULT '',
skill VARCHAR(64) NOT NULL DEFAULT '' 
)CHARSET=utf8

INSERT INTO monster VALUES(100, '青牛怪', '吐火');
INSERT INTO monster VALUES(200, '黄袍怪', '吐烟');
INSERT INTO monster VALUES(300, '蜘蛛怪', '吐丝');
```

创建和表对应的字段类

```java
public class Monster {
    private Integer monsterId;
    private String name;
    private String skill;
//全参构造器
public Monster(Integer monsterId, String name, String skill) {
    this.monsterId = monsterId;
    this.name = name;
    this.skill = skill;
}
//无参构造器一定要写，Spring反射创建对象时，需要使用
public Monster() {
}
 
public Integer getMonsterId() {
    return monsterId;
}
 
public void setMonsterId(Integer monsterId) {
    this.monsterId = monsterId;
}
 
public String getName() {
    return name;
}
 
public void setName(String name) {
    this.name = name;
}
 
public String getSkill() {
    return skill;
}
 
public void setSkill(String skill) {
    this.skill = skill;
}
 
@Override
public String toString() {
    return "Monster{" +
            "monsterId=" + monsterId +
            ", name='" + name + '\'' +
            ", skill='" + skill + '\'' +
            '}';
	}
}	
```
好，可以开始介绍使用方式了

### 添加

```java
    @Test
public void addDataByJdbcTemplate() {
    //获取到容器
    ApplicationContext ioc = new ClassPathXmlApplicationContext("JdbcTemplate_ioc.xml");
    //获取JdbcTemplate对象
    JdbcTemplate jdbcTemplate = ioc.getBean(JdbcTemplate.class);
    
    //1. 添加方式1
    String sql = "INSERT INTO monster VALUES(400, '红孩儿', '枪法')";
    jdbcTemplate.execute(sql);
    
    //2. 添加方式2
    String sql = "INSERT INTO monster VALUES(?, ?, ?)";
    //affected表示 执行后表受影响的记录数
    int affected = jdbcTemplate.update(sql, 500, "红孩儿2", "枪法2");
    System.out.println("add ok affected=" + affected);
  }
```

### 批量添加

```java
//批量添加2个monster白蛇精和青蛇精
    @Test
    public void addBatchDataByJdbcTemplate() {
        ApplicationContext ioc = new ClassPathXmlApplicationContext("JdbcTemplate_ioc.xml");
        //得到JdbcTemplate bean
        JdbcTemplate jdbcTemplate = ioc.getBean(JdbcTemplate.class);
 
        //1. 如何使用API？先猜测API名称，一看有一个叫批量更新的 batchUpdate就先试试看 [如果出现问题，重新猜或搜索引擎]
        //对照着给出的提示准备参数 public int[] batchUpdate(String sql, List<Object[]> batchArgs){}
        //2. 准备参数
        String sql = "INSERT INTO monster VALUES(?, ?, ?)";
        List<Object[]> batchArgs = new ArrayList<>();
        batchArgs.add(new Object[]{600, "老鼠精", "偷吃粮食"});
        batchArgs.add(new Object[]{700, "老猫精", "抓老鼠"});
        //3. 调用
        //说明：返回结果是一个数组，每个元素对应上面的sql语句对表的影响记录数
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        //输出试试看是否正确
        for (int anInt : ints) {
            System.out.println("anInt=" + anInt);
        }
    }
```

### 查询并封装到实体对象

```java
//查询 id=100 的 monster
public void selectDataByJdbcTemplate() {
        ApplicationContext ioc = new ClassPathXmlApplicationContext("JdbcTemplate_ioc.xml");
        //得到JdbcTemplate bean
        JdbcTemplate jdbcTemplate = ioc.getBean(JdbcTemplate.class);
        //1. 猜测 API ：queryForObject()
        //public <T> T queryForObject(String sql, RowMapper<T> rowMapper, @Nullable Object... args)
        //2.准备参数
        //组织SQL
        String sql = "SELECT id AS monsterId, NAME, skill FROM monster WHERE id = 100";
        //通过BeanPropertyRowMapper获取rowmapper 是一个接口，可以将查询的结果，封装到你指定的对象中
        RowMapper<Monster> rowMapper = new BeanPropertyRowMapper<>(Monster.class);
        //RowMapper 接口是对返回的数据，进行一个封装，底层就是使用的反射调用setter方法
      
     
        //使用jdbcTemplate调用API  返回值就是rowMapper定的类型 这里就是Monster
        Monster monster = jdbcTemplate.queryForObject(sql, rowMapper);
        System.out.println("monster= " + monster);
    }

//查询id>=200的monster并封装到Monster实体对象
@Test
    public void selectMulDataByJdbcTemplate() {
        ApplicationContext ioc =
                new ClassPathXmlApplicationContext("JdbcTemplate_ioc.xml");
        //得到JdbcTemplate bean
        JdbcTemplate jdbcTemplate = ioc.getBean(JdbcTemplate.class);
        //1.    确定API
        //public <T> T query(String sql, RowMapper<T> rowMapper, Object... args){}
        //2. 组织参数
        //SQL
        String sql = "SELECT id AS monsterId, NAME, skill FROM monster WHERE id >= ?";
        //rowmapper
  	    RowMapper<Monster> rowMapper = new BeanPropertyRowMapper<>(Monster.class);
        //args 就是100
        //3. 调用
        List<Monster> monsterList = jdbcTemplate.query(sql, rowMapper, 100);
        for (Monster monster : monsterList) {
            System.out.println("monster= " + monster);
        }
    }
```

**这里有一个细节: 你查询的记录的表的字段需要和要封装到的实体对象的字段名保持一致**

说人话，就是select  xxx  的xxx要和实体对象的字段相同  ，就是下面三个

```java
    private Integer monsterId;
    private String name;		//无需区分大小写
    private String skill;
```

但是xxx又必须与数据库里的字段相同，我们发现数据库里是  id ， name ， skill ，所以id 和 monsterId 不同

```sql
CREATE TABLE monster(
id INT PRIMARY KEY, 						 -- id
`name` VARCHAR(64) NOT NULL DEFAULT '',	       -- name
skill VARCHAR(64) NOT NULL DEFAULT '' 		   -- skill
)CHARSET=utf8
```

  这个时候我们就要通过别名来解决问题，就是使用 AS语句 改成 SELECT id AS monsterId, NAME, skill 即可 ，这里我们之前数据库里其实学习过，忘记的可以去看看我的mysql笔记。

### 具名参数

作用：可以一眼看出参数是什么，而不是各种问号？ 便于协同开发

但是需要使用NamedParameterJdbcTemplate 类，所以先来增加一些配置

**增加xml配置**

```xml
 <!--引入外部的jdbc.properties文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <!--配置数据源对象-DataSoruce-->
    <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
        <!--给数据源对象配置属性值-->
        <property name="user" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.pwd}"/>
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
    </bean>
 
    <!--配置JdbcTemplate对象-->
    <bean class="org.springframework.jdbc.core.JdbcTemplate" id="jdbcTemplate">
        <!--给JdbcTemplate对象配置dataSource-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
 
    <!--配置NamedParameterJdbcTemplate对象-->
    <bean class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate"
          id="namedParameterJdbcTemplate">
        <!--通过构造器，设置数据源-->
        <constructor-arg name="dataSource" ref="dataSource"/>
    </bean>
```

```java
 //使用Map传入具名参数完成操作
@Test
    public void testDataByNamedParameterJdbcTemplate() {
        ApplicationContext ioc = new ClassPathXmlApplicationContext("JdbcTemplate_ioc.xml");
        //得到NamedParameterJdbcTemplate bean
        NamedParameterJdbcTemplate namedParameterJdbcTemplate = ioc.getBean(NamedParameterJdbcTemplate.class);
 
        //1. 确定使用API
        //public int update(String sql, Map<String, ?> paramMap)
        //2. 准备参数 [:my_id, :name, :skill] 要求按照规定的名字来设置参数
        String sql = "INSERT INTO monster VALUES(:id, :name, :skill)";
        Map<String, Object> paramMap = new HashMap<>();
        //给paramMap填写数据
        paramMap.put("id", 800);
        paramMap.put("name", "蚂蚁精");
        paramMap.put("skill", "喜欢打洞");
        //3.通过namedParameterJdbcTemplate调用API
        int affected = namedParameterJdbcTemplate.update(sql, paramMap);
        System.out.println("add ok affected=" + affected);
    }
```

```java
//使用sqlparametersoruce可以将一个实体对象封装成具名参数
 @Test
    public void operDataBySqlparametersoruce() {
        ApplicationContext ioc = new ClassPathXmlApplicationContext("JdbcTemplate_ioc.xml");
        //得到NamedParameterJdbcTemplate bean
        NamedParameterJdbcTemplate namedParameterJdbcTemplate = ioc.getBean(NamedParameterJdbcTemplate.class);
        //确定API
        //public int update(String sql, SqlParameterSource paramSource)
        //public BeanPropertySqlParameterSource(Object object)
        //准备参数
        String sql = "INSERT INTO monster VALUES(:monsterId, :name, :skill)";
        Monster monster = new Monster(900, "大象精", "搬运木头");
        SqlParameterSource sqlParameterSource = new BeanPropertySqlParameterSource(monster);
        //3.通过namedParameterJdbcTemplate调用API
        int affected = namedParameterJdbcTemplate.update(sql, sqlParameterSource);
        System.out.println("add ok affected= " + affected);
    }
```

### Dao 对象中使用 JdbcTemplate 完成对数据操作   

 创建MonsterDao类

```java
@Repository //将MonsterDao 注入到spring容器
public class MonsterDao {
 
    //注入一个属性
    @Resource
    private JdbcTemplate jdbcTemplate;
 
    //写一个方法用于插入
    public void save(Monster monster) {
        //组织sql
        String sql = "INSERT INTO monster VALUES(?,?,?)";
        int affected = jdbcTemplate.update(sql, monster.getMonsterId(), monster.getName(), monster.getSkill());
        System.out.println("affected= " + affected);
    }
}
```

配置也要相应的改变，因为使用了注解方式，所以要扫描一下包，在xml里添加下面这句话

```xml
 <!--配置要扫描包-->
    <context:component-scan base-package="com.spring.jdbctemplate.dao"/>
```

```java
 //测试MonsterDAO
    @Test
    public void monsterDaoSave() {
        ApplicationContext ioc = new ClassPathXmlApplicationContext("JdbcTemplate_ioc.xml");
        MonsterDao monsterDao = ioc.getBean(MonsterDao.class);
        Monster monster = new Monster(1000, "小鸭精", "吃鱼");
        monsterDao.save(monster);
        System.out.println("MonsterDAO保存 ok ..");
    }
```

欧克，Spring讲笔记到此为止。
