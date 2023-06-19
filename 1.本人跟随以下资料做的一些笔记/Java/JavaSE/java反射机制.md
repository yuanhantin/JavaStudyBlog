## java反射机制

### 反射机制原理示意图

![java反射机制示意图](https://s2.loli.net/2023/03/19/bOirj7BkNEHYwWX.png)

​		**Class.forName(字节码文件)**								              		**类.class**																		  **对象.getClass()**

## 用法:

根据配置的properties文件(不仅是properties)从而无需修改源代码的情况下调用和修改类的东西。

一.  初始化properties

```java
Properties properties = new Properties();
        FileInputStream fileInputStream = new FileInputStream("out/production/reflect.properties");
        properties.load(fileInputStream);                                  //把文件加载到properties对象
        String classpath = properties.get("classpath").toString();         //获取key的value,然后用toString转换成string值留给Class.forName使用
        String methodName = properties.get("method").toString();
```

二.  

```java
	    //1.加载类,返回Class类型的对象cls				
        Class cls = Class.forName(classpath);

        //2.通过cls获取你加载的类(Hsp.Reflect)的  对象实例obj     		注意要有public的无参构造方法!!!
	    //因为在newInstance时底层会调用cls的无参构造器获得一个新的实例对象
        Object obj = cls.newInstance();

        //3.通过cls获取加载的类的    方法对象
        Method method = cls.getMethod(methodName);

        //4.通过method方法对象,调用方法
        method.invoke(obj);     //传统方法:对象.方法        反射机制:方法对象.invoke(对象)
       
        //5.如何获取类的成员变量对象  (没有用反爆破前不可以是private)         getField();
        Field fieldName = cls.getField("name".toString());
        System.out.println(fieldName.get(obj));

        //6.获取类的构造器
	    //没填就是无参构造器
        Constructor constructor1 = cls.getConstructor();     
 		//通过有参构造器的  参数类型.class    获取有参构造器
        Constructor constructor2 = cls.getConstructor(String.class);       
```



## 通过反射获取类的结构信息

![image-20230318121437998](https://s2.loli.net/2023/03/19/nkejtME8QdLIalq.png)

![image-20230318121441399](https://s2.loli.net/2023/03/19/ZPxBoiJdn4UFEbm.png)

![image-20230318121443653](https://s2.loli.net/2023/03/19/FzVnwUkuY1eq5WX.png)

![image-20230318121445555](https://s2.loli.net/2023/03/19/BDp51AQbxSmZOjY.png)

### 反射爆破

#### 取消访问调用检查

method、constructor、Field都继承了Accessiable，通过method / constructor / Field.setAccessible(true)忽略访问控制检查，从而可以**操作私有的成员**或**优化性能**加快速度。
