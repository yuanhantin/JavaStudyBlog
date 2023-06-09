# MyBatis

**什么是MyBatis**

- MyBatis是优秀的持久层框架 
- MyBatis使用XML将SQL与程序解耦，便于维护 
-  MyBatis学习简单，执行高效，是JDBC的延伸

### 1.MyBatis开发流程 

- 引入MyBatis依赖 
- 创建核心配置文件 
- 创建实体(Entity)类 
- 创建Mapper映射文件 
- 初始化SessionFactory
- 利用SqlSession对象操作数据

#### 1.1引入MyBatis依赖 

利用maven直接从仓库导入即可，有的小伙伴肯定不知道怎么去官网找，先来教一下好了。

[官网链接：Maven Repository: maven (mvnrepository.com)](https://mvnrepository.com/tags/maven)

直接搜索mybatis然后点击第一个就好了

![image-20230521112100242](https://s2.loli.net/2023/05/21/EBYgrhFVqx7K23R.png)

这里我们选择3.5.1版本

![image-20230521132339229](https://s2.loli.net/2023/05/21/IyPnHolpzRerYuB.png)

往下滑我们可以看到他的配置信息,复制到pom.xml即可

![image-20230521132504045](https://s2.loli.net/2023/05/21/7Ncfgk3Vw8jDKRI.png)

##### 关于配置pom.xml时IDEA报错解决方案

有时候我们的idea会抽风，复制过来之后会报红或者显示无法在中央仓库中找到依赖，一般剪切掉它再贴一遍就好了，实在不行重启一下idea

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>Mybatis</artifactId>
  <version>3.5.1</version>
</dependency>

//mysql包顺便也放这里了,如果使用mysql的同学记得加上
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.47</version>
</dependency>
```

#### 1.2创建核心配置文件 

差不多和Spring的配置文件差不多格式

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--核心配置文件-->
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/ajaxdb?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai"/>
                <property name="username" value="root"/>
                <property name="password" value="20030515"/>
            </dataSource>
        </environment>
    <!--    在这里可以配置多个环境,比如生产环境,只需要切换第8行的default为每个环境的id即可-->
        <environment id="produce">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/ajaxdb?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai"/>
                <property name="username" value="root"/>
                <property name="password" value="20030515"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <!--用来配置mapper的xml配置文件，相当于告诉mybatis我的用于sql语句的配置文件放哪里-->
        <mapper resource="mapper/ajaxdb.xml"/>
    </mappers>
</configuration>

```

#### 1.3创建实体(Entity)类 

这里不再赘述，跟之前的JDBC连接数据库时要创建出一个跟表中字段相同的类是一样的。[韩顺平Spring体系化笔记(内含ioc,aop,动态代理等底层原理) - 翰林猿 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hanlinyuan/p/17394096.html#批量添加)

具体可以查看本文中第10小节，不过既然都学到了mybatis想必对此熟悉的不能再熟悉了。

#### 1.4创建Mapper映射文件 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace相当于一个包名，这个包下放了很多种sql语句，每种又有自己的id，
比如说到时候要使用select * from user时就直接填入user.selectAll即可-->
<mapper namespace="user">
    <!--id 相当于给这段用于select的sql语句取一个别名，到时候直接用selectAll代替 select * from user 这句话-->
    <!--resultType中填写entity类的全路径-->
    <select id="selectAll" resultType="entity.ajaxdbEntity">
        select * from user;
    </select>
</mapper>
```

#### 1.5初始化SessionFactory

在这里我们顺便编写一个工具类

```java
package utils;
import entity.ajaxdbEntity;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.Reader;
import java.util.List;

/**
 * @Author: 翰林猿
 * @Description: TODO
 **/
public class MyBatisUtils {
    /**
     * MyBatisUtils工具类,创建全局唯一的SqlSessionFactory对象
     */

    //利用static(静态)属于类不属于对象,且全局唯一
    private static SqlSessionFactory sqlSessionFactory = null;

    //利用静态块在初始化类时实例化sqlSessionFactory
    static {
        Reader reader = null;
        try {
            这里的Resources
            reader = Resources.getResourceAsReader("mybatisConfig.xml");
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
        } catch (IOException e) {
            e.printStackTrace();
            //初始化错误时,通过抛出异常ExceptionInInitializerError通知调用者
            throw new ExceptionInInitializerError(e);
        }
    }

    /**
     * openSession 创建一个新的SqlSession对象
     * SqlSession对象类似于JDBC中的Connection
     * @return SqlSession对象
     */
    public static SqlSession openSession() {
        return sqlSessionFactory.openSession();
    }

    /**
     * 释放一个有效的SqlSession对象
     *类似于JDBC中释放获取到的Connection
     * @param session 准备释放SqlSession对象
     */
    public static void closeSession(SqlSession session) {
        if (session != null) {
            session.close();
        }
    }
    
    //简单测试一下
	@Test
    public void test(){ 
        //类似于获取Connection
        SqlSession sqlSession = MyBatisUtils.openSession();
        List<ajaxdbEntity> ajaxdbDaoList = sqlSession.selectList("user.selectAll");
        for (ajaxdbEntity dao : ajaxdbDaoList) {
            System.out.println(dao.getId());
            System.out.println(dao.getUsername());
        }
    }
}

```

### mybatis底层实现（造轮子）

![image-20230522202933837](https://s2.loli.net/2023/05/22/u32hXcDGNAr6m8d.png)





### 2.SQL

#### SQL传参及查询

回忆之前的JDBC中在组织sql语句时 使用问号动态传入参数 进行查询的操作，同样我们mybatis也有。

```xml
<!--parameterType也就是传入的参数的类型是什么-->
<select id="selectList" resultType="entity.ajaxdbEntity" parameterType="Map">
        select * from user 
        where id between #{min} and #{max}
        order by id;
    </select>
```

这里我们使用map的形式作为参数传递，那么怎么使用呢，修改一下我们的 Test 函数作为示例。

```java
@Test
    public void test(){
        //类似于获取Connection
        SqlSession sqlSession = MyBatisUtils.openSession();
        HashMap map = new HashMap();
        map.put("min",1);
        map.put("max",3);
        //mybatis底层会将传入的map解析，找到#{key}对应的value填入sql语句中
        List<ajaxdbEntity> ajaxdbDaoList = sqlSession.selectList("user.selectList",map);
        for (ajaxdbEntity map : ajaxdbDaoList) {
            System.out.println(map);
        }
    }
```

#### 多表查询

多表查询时我们不再需要填入parameterType，而且resultType使用Map或者LinkHashMap

使用Map的话，查询出来的字段顺序是混乱的，具体看Map的底层原理，而LinkHashMap是按顺序的。

但是这种查询方式有一个很大的缺点，你应该也已经发现了，为什么我们的resultType不再是实体Entity类了？

这种查询方式不需要经过验证，他什么东西都可以直接查询出来。所以在企业中开发大型项目还是一般不使用这种方式。

```xml
    <select id="selectTwice"  resultType="Map">
        select * from user,user2;
    </select>
```

所以我们使用ResultMap结果映射，来解决一下这个问题。

#### ResultMap结果映射查询

作用：首先是完成数据库字段与实体类字段的映射（类似之前为了解决数据库字段和实体类字段不相同采用AS语句完成映射），其次就是解决上面所提到的问题。

说白了，还是Entity那套，再创建一个类，作为多表查询的实体类，只不过这个类里有多个entity的字段罢了。那么我们来写一下这个字段吧。

```java
package entity;

/**
 * @Author: 翰林猿
 * @Description: 多表查询实体类
 **/
public class UserDTO {
    private user user = new user();		//user表
    private String cn;				   //user2表中的cn字段（ChineseName的缩写）

    @Override
    public String toString() {
        return "UserDTO{" +
                "user=" + user +
                ", cn='" + cn + '\'' +
                '}';
    }

    public user getUser() {
        return user;
    }

    public void setUser(user user) {
        this.user = user;
    }

    public String getCn() {
        return cn;
    }

    public void setCn(String cn) {
        this.cn = cn;
    }

    public UserDTO() {
    }

    public UserDTO(user ajaxdbEntity, String cn) {
        user = ajaxdbEntity;
        this.cn = cn;
    }
}

```

好，我们再来看看基本使用方法，我们要定义一个resultMap标签，id值就是作为这段resultMap的别名，type是最后查询返回的类的全路径，然而里面又有很多种标签，这里我们就介绍2个，其他的请大家自己前往mybatis官网查看教程。

我们的id和result标签中又包括两个参数，property以及column

- property ：填写的必须与上面的type中的类的字段完全相同，（也就是UserDTO中的字段），其中有一个字段是user类的，那么为了拿到user类中的字段直接使用  .  运算符即可。
- column  ：数据库中的列名，或者是列的别名。

```xml
    <resultMap id="selectDTO" type="entity.UserDTO">
        <id property="user.id" column="id"></id>
        <result property="user.username" column="username"/>
        <result property="user.pwd" column="pwd"/>
        <result property="user.email" column="email"/>
        <result property="cn" column="cn"/>
         property对应实体类的属性 ，colum对应数据库的字段
    </resultMap>

    <select id="selectTwice"  resultMap="selectDTO">
        select user.* , cn from user,user2;
    </select>
```

```java
/**
 * @Author: 翰林猿
 * @Description: 多表查询实体类，测试
 **/
@Test
    public void test(){
        //类似于获取Connection
        SqlSession sqlSession = MyBatisUtils.openSession();
        List<UserDTO> ajaxdbDaoList = sqlSession.selectList("User.selectTwice");
        for (UserDTO map : ajaxdbDaoList) {
            System.out.println(map);
        }
    }
```





#### SQL插入

我们在写sql插入语句的时候有时候要写很多列名，真的是非常令人烦恼啊，很显然，mybais开发人员也想到了这点，用foreach标签节约开发时间。

foreach元素的属性主要有 item，index，collection，open，separator，close。

-   item表示集合中每一个元素进行迭代时的别名，
-   index指定一个名字，用于表示在迭代过程中，每次迭代到的位置
-   collection 填写数据源的，如果是List就填list，是Array就填array，是Map就填mykey
-   open表示该语句以什么开始，
-   separator表示在每次进行迭代之间以什么符号作为分隔符，
-   close表示以什么结束。

```xml
<!--INSERT INTO table-->
<!--Mysql中插入多条数据是用逗号分开的形式-->
<!--VALUES ("a" , "a1" , "a2"),("b" , "b1" , "b2"),(....)-->
    <insert id="batchInsert" parameterType="java.util.List">
        INSERT INTO t_goods(title, sub_title, original_cost, current_price, discount, is_free_delivery, category_id)
        VALUES
        <!--collection指数据源是哪来的，这里是list就是从上面那个parameterType的List来的-->
        <foreach collection="list" item="item" index="index" separator=",">
            (#{item.title},#{item.subTitle}, #{item.originalCost}, #{item.currentPrice}, #{item.discount}, #{item.isFreeDelivery}, #{item.categoryId})
        </foreach>
    </insert>
```

注意新增数据之后，要记得提交事务

```java
    /**
     * 新增数据，示范用例
     */
    @Test
    public void testInsert() throws Exception {
        SqlSession session = null;
        try{
            session = MyBatisUtils.openSession();
            Goods goods = new Goods();
            goods.setTitle("测试商品");
            goods.setSubTitle("测试子标题");
            goods.setOriginalCost(200f);
            goods.setCurrentPrice(100f);
            goods.setDiscount(0.5f);
            goods.setIsFreeDelivery(1);
            goods.setCategoryId(43);
            //insert()方法返回值代表本次成功插入的记录总数
            int num = session.insert("goods.insert", goods);
            session.commit();//提交事务数据
            System.out.println(goods.getGoodsId());
        }catch (Exception e){
            if(session != null){
                session.rollback();//回滚事务
            }
            throw e;
        }finally {
            MyBatisUtils.closeSession(session);
        }
    }
```

#### SQL删除

删除同理

```xml
<delete id="batchDelete" parameterType="java.util.List">
    DELETE FROM t_goods WHERE goods_id in
    <foreach collection="list" item="item" index="index" open="(" close=")" separator=",">
        #{item}
    </foreach>
</delete>
```

```java
/**
 * 删除数据
 */
@Test
public void testDelete() throws Exception {
    SqlSession session = null;
    try{
        session = MyBatisUtils.openSession();
        int num = session.delete("goods.delete" , 739);
        session.commit();//提交事务数据
    }catch (Exception e){
        if(session != null){
            session.rollback();//回滚事务
        }
        throw e;
    }finally {
        MyBatisUtils.closeSession(session);
    }
}
```

#### 动态SQL

在我们逛淘宝的时候，是否有发现在搜索商品时有很多标签可以选择，比如说指定某个品牌，某种类型等等。其实从开发者的角度上来看，是不是还是一个SQL语句罢了，只不过多加了几个参数，但是问题就在于，这些参数如何动态的加载进去，在想要的时候才用他呢？

这里引出mybatis的动态SQL语句技术，不过是加个标签罢了。底层依旧是之前那套，dom4j+动态代理+反射完成的。

```xml
<select id="dynamicSQL" parameterType="java.util.Map" resultType="com.imooc.mybatis.entity.Goods">
    select * from t_goods
    <where>									//SQL种的where改成使用where标签
      <if test="categoryId != null">		  //如果map里有这个参数，就加上下面这句
          and category_id = #{categoryId}
      </if>
      <if test="currentPrice != null">
          and current_price &lt; #{currentPrice}
      </if>
    </where>
</select>
```

```java
/**
 * 动态SQL语句
 */
@Test
public void testDynamicSQL() throws Exception {
    SqlSession session = null;
    try{
        session = MyBatisUtils.openSession();
        Map param = new HashMap();
        param.put("categoryId", 44);
        param.put("currentPrice", 500);
        //查询条件
        List<Goods> list = session.selectList("goods.dynamicSQL", param);
        for(Goods g:list){
            System.out.println(g.getTitle() + ":" +
                    g.getCategoryId()  + ":" + g.getCurrentPrice());

        }
    }catch (Exception e){
        throw e;
    }finally {
        MyBatisUtils.closeSession(session);
    }
}
```

### 3.Mybatis二级缓存机制

为了提高查询效率，减少数据库的访问次数，Mybatis采用了两层缓存机制，并分为一级缓存和二级缓存

因为一级缓存的命中率可能较低，所以还有一层二级缓存

- 一级缓存**默认开启**,缓存范围SqlSession

  - （换句话说就是，只在SqlSession session = MyBatisUtils.openSession()的这个session里有效，当我们写了两遍这个代码，就是两个不同的SqlSession对象）
  - 拿上面的代码举例，就是你使用这个session不管查询多少次相同的语句，都是从缓存里拿出来的一个结果

  ```java
  session.selectList("goods.dynamicSQL", param);
  session.selectList("goods.dynamicSQL", param);
  session.selectList("goods.dynamicSQL", param);
  //最后都是相同的结果，内存地址都是一样的
  ```

- 二级缓存**手动开启**,属于范围Mapper Namespace

  - （换句话说，就是只要是使用了之前我们定义mapper的namespace中的SQL语句里都有效）

  ```xml
  <mapper namespace="goods">
  ....各种SQL....
  </mapper>
  ```

![image-20230524152746106](https://s2.loli.net/2023/05/24/czl71bXdv8msPUN.png)

#### 注意：

当调用SqlSession的修改、添加、删除、commit()、close()等方法时，为了保证数据的一致性，mybatis会强制清空一级缓存。

如何理解这句话？用代码举例一下

```java
session = MyBatisUtils.openSession();
Goods goods = session.selectOne("goods.selectById" , 1603);
session.commit();		//commit提交时对该namespace缓存强制清空
Goods goods2 = session.selectOne("goods.selectById" , 1603);
```

这个时候，因为提交过一次事务，所以第二次的查询goods2时的hashcode与第一次的goods其实是不一样的，本质上是2次查询。而不是直接拿出goods已经查询出来的内容赋给goods2

#### 开启二级缓存

在xml中的mapper配置一句话即可

```xml
<mapper namespace="goods">
    <!--
        eviction是缓存的清除策略,当缓存对象数量达到上限后,自动触发对应算法对缓存对象清除
            1.LRU – 最近最久未使用:移除最长时间不被使用的对象。
            O1 O2 O3 O4 .. O512     最多只有512个对象
            14 99 83 1     893（秒） 记录对象上一次被访问的时间，如果512个满了，就会先把时间最长的893秒的那个对象移除
            2.FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
            3.SOFT – 软引用:移除基于垃圾收集器状态和软引用规则的对象。
            4.WEAK – 弱引用:更积极的移除基于垃圾收集器状态和弱引用规则的对象。

    	flushInterval属性表示刷新缓存的时间间隔ms
    	size属性表示缓存的大小
    	readOnly属性表示缓存是否只读，可选值为true或false，true是返回对象本身（效率高），false则是返回对象的副本（安全好）
  -->
    <cache eviction="LRU" flushInterval="600000" size="512" readOnly="true"/>
    
    ....各种SQL....
    
    </mapper>
```

#### 缓存大量数据性能问题

既然我们开启了缓存，那么就要考虑一下如果缓存了大量数据影响性能怎么办，其实对于查询大量数据的语句可以不使用缓存，而且在实际过程中，这种大量的查询也不会经常复用。

```xml
<!-- useCache="false"代表该语句不保存至缓存 --> 
<select id="selectAll" resultType="com.imooc.mybatis.entity.Goods" useCache="false">
    select * from t_goods order by goods_id desc limit 10
</select>
```

```xml
<!-- flushCache="true"执行完这句话马上刷新缓存 = 一个commit操作 --> 
<!-- 并且这句话查询出来的数据本身也不会放入缓存 --> 
<select id="selectGoodsMap" resultType="java.util.LinkedHashMap" flushCache="true">
    select g.* , c.category_name,'1' as test from t_goods g , t_category c
    where g.category_id = c.category_id
</select>
```

### 4.对象关联查询（一对多、多对一）

我们知道，一个表内有很多个实体，可能有一个实体对应了很多个对象，有一些则是一对一，多对一，多对多等等。

![image-20230524174033263](https://s2.loli.net/2023/05/24/fRcgPIN6lnpaC2e.png)

所以引出我们的关联查询

#### 一对多查询：

先来举个例子：比如说我们要查询一个商品，但是一个商品Goods又对应了很多个商品细节GoodsDetail（一对多）

所以我们的商品细节表GoodsDetail要持有我们的商品表Goods的主键 goodsId，那么除掉建表的过程，先写一下对应的实体类吧

```java
public class Goods {
    private Integer goodsId;//商品编号
    private String title;//标题
    private String subTitle;//子标题
    private Float originalCost;//原始价格
    private Float currentPrice;//当前价格
    private Float discount;//折扣率
    private Integer isFreeDelivery;//是否包邮 ,1-包邮 0-不包邮
    private Integer categoryId;//分类编号
    private List<GoodsDetail> goodsDetails;
}
//省略getter，setter，构造器等等
```

```java
public class GoodsDetail {
    private Integer gdId;
    private Integer goodsId;
    private String gdPicUrl;
    private Integer gdOrder;
    private Goods goods;
}
//省略getter，setter，构造器等等
```

那么我们要如何实现一对多的查询呢，我们来看看本质是什么，不过就是去Goods里面找GoodsDetail，再去GoodsDetail里面找具体的属性罢了。

所以，还是使用我们的resultMap进行映射就好了，只要将goodsDetails的属性映射到Goods里面，不就相当于Goods拥有了goodsDetails的属性了嘛，可以理解为类似下面这个类。

```java
public class GoodsAfterMapping {
    private Integer goodsId;//商品编号
    private String title;//标题
    private String subTitle;//子标题
    private Float originalCost;//原始价格
    private Float currentPrice;//当前价格
    private Float discount;//折扣率
    private Integer isFreeDelivery;//是否包邮 ,1-包邮 0-不包邮
    private Integer categoryId;//分类编号
        private Integer gdId;
    	private Integer goodsId;
    	private String gdPicUrl;
    	private Integer gdOrder;
    	private Goods goods;
}
//省略getter，setter，构造器等等
```

ok，有了思路，来具体实现一下吧，写好查询语句

```xml
<mapper namespace="goodsDetail">
<select id="selectOneToMany" resultMap="RmGoods1">
    select * from t_goods limit 0,10
</select>
</mapper>
```

想一想我们的思路，是不是要利用这句select语句查询出来的ID，去我们的我们商品细节表GoodsDetail里把其他属性也查出来，所以我们再写一个select语句用于查询商品细节表GoodsDetail

```xml
<mapper namespace="goodsDetail">
<select id="selectByGoodsId" parameterType="Integer"
        resultType="com.imooc.mybatis.entity.GoodsDetail">
    select * from t_goods_detail where goods_id = #{value}
</select>
</mapper>
```

然后开始配置我们的 resultMap ，因为要持有一个对象集合，所以我们使用collection标签返回一个集合

```xml
<mapper namespace="goodsDetail">
<!--
    resultMap可用于说明一对多或者多对一的映射逻辑
    id 是resultMap属性引用的标志
    type 指向我们所谓的一对多的一的实体(也就是Goods)
-->
<resultMap id="RmGoods1" type="com.imooc.mybatis.entity.Goods">
    <!-- 还是先确定主键id，把数据库字段和实体类同步一下 -->
    <id column="goods_id" property="goodsId"></id>
     <!--这里不需要像多表查询那里大量配置result，因为数据库字段和POJO字段符合驼峰命名规则，框架会自动转换-->
    
    <!--collection的含义是,在select * from t_goods limit 0,1 得到结果后,
        将得到的goods_id值,
        代入到goodsDetail命名空间的selectByGoodsId的SQL中执行查询,
        最后框架会自动将查询得到的"商品细节"的结果（一个集合）
	    赋值给 private List<GoodsDetail> goodsDetails 这个对象  -->
    <collection property="goodsDetails" select="goodsDetail.selectByGoodsId" column="goods_id"/>
</resultMap>
</mapper>
```

为了好看放一下全部配置吧

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="goodsDetail">
     查询Goods表
    <select id="selectOneToMany" resultMap="rmGoods1">
        select * from t_goods limit 0,10
    </select>
    利用上句查询出来的ID去GoodsDetail查询
    <select id="selectByGoodsId" parameterType="Integer"
            resultType="com.imooc.mybatis.entity.GoodsDetail">
        select * from t_goods_detail where goods_id = #{value}
    </select>
    配置resultMap
  <resultMap id="RmGoods1" type="com.imooc.mybatis.entity.Goods">
    <id column="goods_id" property="goodsId"></id>
    <collection property="goodsDetails" select="goodsDetail.selectByGoodsId" column="goods_id"/>
  </resultMap>
</mapper>
```

#### 多对一查询：

已经知道了一对多，其实多对一就非常的简单了，一对多是在“一”对象里加一个“多”对象的集合，多对一只要在“多”对象中加一个对象即可。由于在这里我们只需要持有一个对象，所以我们使用association标签来在一个对象里关联另一个对象。

还是类似一对多的思路，先把“多”查询出来，然后再利用“多”的ID去“一”中查询即可。

查询“多”：

```xml
<select id="selectManyToOne" resultMap="rmGoodsDetail">
    select * from t_goods_detail limit 0,20
</select>
```

利用“多”的ID去“一”中查询：

```xml
<select id="selectById" parameterType="Integer" resultType="com.imooc.mybatis.entity.Goods">
    select * from t_goods where  goods_id = #{value}
</select>
```

编写映射resultMap

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="goodsDetail">
    <select id="selectByGoodsId" parameterType="Integer"
            resultType="com.imooc.mybatis.entity.GoodsDetail">
        select * from t_goods_detail where goods_id = #{value}
    </select>

    <!--为什么这里不需要大量配置result呢，因为数据库字段和POJO字段符合驼峰命名规则，mybatis会自动转换-->
    <resultMap id="RmGoodsDetail" type="com.imooc.mybatis.entity.GoodsDetail">
        <id column="gd_id" property="gdId"/>
        <!-- 为什么这里要用result映射“一”表的id，讲道理不应该直接由mybatis自动配置嘛？
		    因为下一句的association中，已经利用过一次“一”表的goods_id去selectById并将结果赋给goods对象了，
 		    已经使用过一次，所以这里mybatis底层就会抛弃掉goods_id了，不能再使用，所以要重新配置一次  -->
        <result column="goods_id" property="goodsId"/>
        <association property="goods" select="goods.selectById" column="goods_id"></association>
    </resultMap>

    <select id="selectManyToOne" resultMap="RmGoodsDetail">
        select * from t_goods_detail limit 0,20
    </select>
</mapper>
```

### 5.Mybatis-PageHelper

学过数据库的小伙伴都知道分页查询真的是非常令人头疼啊，需要你自己去计算偏移量什么的难受死，这里我们就可以使用一个国人开发的插件PageHelper来帮助我们提高效率。

#### PageHelper使用流程 

- maven引入PageHelper与jsqlparser 

  ```xml
  <dependency>
      <groupId>com.github.pagehelper</groupId>
      <artifactId>pagehelper</artifactId>
      <version>5.1.10</version>
  </dependency>
  <dependency>
      <groupId>com.github.jsqlparser</groupId>
      <artifactId>jsqlparser</artifactId>
      <version>2.0</version>
  </dependency>
  ```

- mybatis-config.xml增加Plugin配置 

  ```xml
  <plugins>
      <plugin interceptor="com.github.pagehelper.PageInterceptor">
          <!--设置数据库类型-->
          <property name="helperDialect" value="mysql"/>
          <!--分页合理化-->
          <property name="reasonable" value="true"/>
      </plugin>
  </plugins>
  ```

- 使用非常方便，只需要在代码中使用PageHelper.startPage()自动分页即可，其他的正常编写

  ```xml
  <!--先来写个SQL语句，注意xml中＜号要用lt;代替，具体去百度搜索一下-->
  <select id="selectPage" resultType="com.imooc.mybatis.entity.Goods">
      select * from t_goods where current_price &lt; 1000
  </select>
  ```

  OK来写个测试类玩玩

  ```java
  @Test
  /**
   * PageHelper分页查询
   */
  public void testSelectPage() throws Exception {
      SqlSession session = null;
      try {
          session = MyBatisUtils.openSession();
          /*startPage方法会自动将下一次查询进行分页*/
          PageHelper.startPage(2,10);//相当于查询第二页的数据，每页数据有10条，这里查出来的也就是第10-20条
          Page<Goods> page = (Page) session.selectList("goods.selectPage");
          System.out.println("总页数:" + page.getPages());
          System.out.println("总记录数:" + page.getTotal());
          System.out.println("开始行号:" + page.getStartRow());
          System.out.println("结束行号:" + page.getEndRow());
          System.out.println("当前页码:" + page.getPageNum());
          List<Goods> data = page.getResult();//当前页数据
          for (Goods g : data) {
              System.out.println(g.getTitle());
          }
          System.out.println("");
      } catch (Exception e) {
          throw e;
      } finally {
          MyBatisUtils.closeSession(session);
      }
  }
  ```

​	

### 6.使用C3P0连接池

首先写一个类，继承UnpooledDataSourceFactory，并写一个构造器

```java
public class C3P0DataSourceFactory extends UnpooledDataSourceFactory {
    public C3P0DataSourceFactory(){
        this.dataSource = new ComboPooledDataSource();
    }
}
```

然后只需要在之前的config.xml中修改一下配置即可

```xml
type中填写上面那个类的全限定名
<dataSource type="com.imooc.mybatis.datasource.C3P0DataSourceFactory">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/xxxx?useUnicode=true&amp;characterEncoding=UTF-8"/>
    <property name="user" value="root"/>
    <property name="password" value="root"/>
    <property name="initialPoolSize" value="5"/>
    <property name="maxPoolSize" value="20"/>
    <property name="minPoolSize" value="5"/>
    <!--...-->
</dataSource>
```

### 7.Mybatis批处理

这里我们就讲讲怎么批量插入吧，批量删除请各位自行百度学习一下即可。

首先来编写一下SQL语句

```xml
<!--INSERT INTO table-->
<!--Mysql中插入多条数据是用逗号分开的形式-->
<!--VALUES ("a" , "a1" , "a2"),("b" , "b1" , "b2"),(....)-->
<insert id="batchInsert" parameterType="java.util.List">
    INSERT INTO t_goods(title, sub_title, original_cost, current_price, discount, is_free_delivery, category_id)
    VALUES
    <!--collection指数据源是哪来的，这里是list就是从上面那个parameterType的List来的-->
    <!--如果这里没看懂，待会看代码-->
    <foreach collection="list" item="item" index="index" separator=",">
        (#{item.title},#{item.subTitle}, #{item.originalCost}, #{item.currentPrice}, #{item.discount}, #{item.isFreeDelivery}, #{item.categoryId})
    </foreach>
</insert>
```

```java
    /**
     * 批量插入测试
     */
    @Test
    public void testBatchInsert() throws Exception {
        SqlSession session = null;
        try {
            long st = new Date().getTime();
            session = MyBatisUtils.openSession();
            List list = new ArrayList();
            for (int i = 0; i < 10000; i++) {
                Goods goods = new Goods();
                goods.setTitle("测试商品");
                goods.setSubTitle("测试子标题");
                goods.setOriginalCost(200f);
                goods.setCurrentPrice(100f);
                goods.setDiscount(0.5f);
                goods.setIsFreeDelivery(1);
                goods.setCategoryId(43);
                //insert()方法返回值代表本次成功插入的记录总数
                list.add(goods);
            }
            //这里的参数就是list，这也是刚刚collection为什么写"list"的原因
            session.insert("goods.batchInsert", list);
            session.commit();//提交事务数据
        } catch (Exception e) {
            if (session != null) {
                session.rollback();//回滚事务
            }
            throw e;
        } finally {
            MyBatisUtils.closeSession(session);
        }
    }
```

#### 批处理弊端：

需要注意的是，我们通过批处理插入的数据我们无法获取他的ID值，其次是大量的插入数据有可能被服务区拒绝，而且我们的网络通讯是有数据量限制的，一次能插入的数据不能超过多少mb等等。所以在插入之前我们需要进行压力测试，**少量多次分批进行插入即可**，当然也有更好的解决方法，请各位自行百度吧。

### 8.注解开发

写了这么多XML文件，肯定你也感觉到很繁琐了，那么可以使用注解开发模式，但是不幸的是，Java 注解的表达能力和灵活性十分有限。尽管我们花了很多时间在调查、设计和试验上，但最强大的 MyBatis 映射并不能用注解来构建——我们真没开玩笑。而 C# 属性就没有这些限制，因此 MyBatis.NET 的配置会比 XML 有更大的选择余地。

那么我们来具体操作一下吧

首先建个包，包名就叫他dao了，然后写一个接口，我们在接口里替代之前的xml来进行开发

具体使用的各个参数可以去官网查看，学的比在这里讲解快

```java
package com.imooc.mybatis.dao;

import com.imooc.mybatis.dto.GoodsDTO;
import com.imooc.mybatis.entity.Goods;
import org.apache.ibatis.annotations.*;

import java.util.List;

public interface GoodsDAO {
    
    @Select("select * from t_goods where current_price between  #{min} and #{max} order by current_price limit 0,#{limt}")
    //参数中使用@Param指定 #{xxx}中的xxx，然后加上xxx的包装类
    public List<Goods> selectByPriceRange(@Param("min") Float min ,@Param("max") Float max ,@Param("limt") Integer limt);

    @Insert("INSERT INTO t_goods(title, sub_title) VALUES (#{title} , #{subTitle})")
    //SelectKey相当于<selectKey>
    @SelectKey(statement = "select last_insert_id()" , before = false , keyProperty = "goodsId" , resultType = Integer.class)
    public int insert(Goods goods);

    @Select("select * from t_goods")
    //Results相当于<resultMap>
    @Results({
            //Result相当于<id>
          @Result(column = "goods_id" ,property = "goodsId" , id = true) ,
            //相当于<result>
            @Result(column = "title" ,property = "title"),
            @Result(column = "current_price" ,property = "currentPrice")
    })
    public List<GoodsDTO> selectAll();
}
```

在这里我们主要讲解一个令人费解的 @SelectKey 注解，里面主要有4个参数

- statement是要运行的SQL语句，它的返回值通过resultType来指定（里面填什么待会讲）
- keyProperty表示查询结果赋值给代码中的哪个对象，keyColumn表示将查询结果赋值给数据库表中哪一列（这两个不是必须的）
- before表示查询语句statement运行的时机
- before=true，插入之前进行查询，可以将查询结果赋给keyProperty和keyColumn，赋给keyColumn相当于更改数据库
- befaore=false，先插入，再查询，这时只能将结果赋给keyProperty

大部分时候使用的是MySQL的数据源，一般就固定使用select last_insert_id()函数来获取自增后的值，这个语句会返回最后插入行的ID，然后将其设置到keyProperty指定的属性中。


最后，除了这些以外，我们还要在config.xml中配置一下，让mybatis知道我们包在哪。

```xml
<mappers>
    <package name="com.imooc.mybatis.dao"/>
</mappers>
```

那么使用注解的话我们代码也要换一个使用方式了，要用getMapper（接口.class）获取到一个接口的实现类，然后再用该类调用接口里面的方法完成增删改查的操作。这里的底层实现和我们在Spring那里造过的轮子差不多，也就是用个匿名内部类将接口实现了，再利用各种反射和动态代理就实现了我们的注解开发。具体可以看这：[韩顺平Spring体系化笔记(内含ioc,aop,动态代理等底层原理) - 翰林猿 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hanlinyuan/p/17394096.html)

```java
@Test
public void testSelectByPriceRange() throws Exception {
    SqlSession session = null;
    try{
        session = MyBatisUtils.openSession();
        GoodsDAO goodsDAO = session.getMapper(GoodsDAO.class);
        List<Goods> list = goodsDAO.selectByPriceRange(100f, 500f, 20);
        System.out.println(list.size());
    }catch (Exception e){
        throw e;
    } finally {
        MyBatisUtils.closeSession(session);
    }
}
```

### 9.Mybatis-Generator

OK讲了这么多，其实我们有没有发现，Mapper接口，Mapper.xml的编辑和创建数据库对应的实体类还是很麻烦的事，有没有一个东西能帮我们简化？当然有。就是我们的Mybatis-Generator插件，也称为Mybatis三剑客之一。

#### 使用方式：

首先要在resource文件夹下新建一个**必须**叫generatorConfig.xml的文件，[Mybatis Generator 配置详解 - 翰林猿 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hanlinyuan/p/17440169.html) 具体怎么配置也可以看这个链接，里面超级具体。当然也可以看下面贴的傻瓜版，不过是以一个项目作为讲解。

![image-20230529132151001](https://s2.loli.net/2023/05/29/EcDzA3ldLYMPjOW.png)

按顺序看注释就好了

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- 这里需要自行去找你的mysql-connector-java-5.1.47.jar下载到哪了，一般在maven仓库里的mysql里有，将其路径复制即可
		（windows：右键jar选择属性，安全，里面有一个对象名称，复制过来即可）
	-->
    <classPathEntry location="C:\Users\windows\.m2\repository\mysql\mysql-connector-java\5.1.47\mysql-connector-java-5.1.47.jar" />

    <context id="DB2Tables" targetRuntime="MyBatis3">
        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />

        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
		 <!--告诉Mybatis-Generator你的数据库驱动-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/Pay?characterEncoding=utf-8"
                        userId="root"
                        password="20030515">
        </jdbcConnection>

        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
        <!--告诉Mybatis-Generator你的实体类要放到哪-->
        <javaModelGenerator targetPackage="com.hanlinyuan.pojo" targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaModelGenerator>
        <!--mapper.xml文件存放位置-->
        <sqlMapGenerator targetPackage="mappers"  targetProject="src/main/resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!--告诉Mybatis-Generator你的Mapper接口要怎么生成，有3种方式，我们这里选择完全依赖xml的形式，也可以选择带注解的
            type：选择怎么生成mapper接口（在MyBatis3/MyBatis3Simple下）：
                1，ANNOTATEDMAPPER：会生成使用Mapper接口+注解的方式创建（SQL在annotation中），不会生成对应的XML；
                2，MIXEDMAPPER：使用混合配置，会生成Mapper接口，并适当添加合适的Annotation，但是XML会生成在XML中；
                3，XMLMAPPER：会生成Mapper接口，接口完全依赖XML；
        -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.hanlinyuan.dao"  targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        
		<!--告诉Mybatis-Generator你的数据库表名（table），以及生成之后的实体类名（domainObjectName）
			剩下的填false就好，具体含义自行百度搜索，就是将一些新手少用的SQL语句就不生成了，看起来舒服一些
		-->
                <table tableName="mall_order" domainObjectName="Order" enableCountByExample="false" enableDeleteByExample="false" enableSelectByExample="false" enableUpdateByExample="false"/>
                <table tableName="mall_order_item" domainObjectName="OrderItem" enableCountByExample="false" enableDeleteByExample="false" enableSelectByExample="false" enableUpdateByExample="false"/>
                <table tableName="mall_user" domainObjectName="User" enableCountByExample="false" enableDeleteByExample="false" enableSelectByExample="false" enableUpdateByExample="false"/>
                <table tableName="mall_category" domainObjectName="Category" enableCountByExample="false" enableDeleteByExample="false" enableSelectByExample="false" enableUpdateByExample="false"/>
                <table tableName="mall_product" domainObjectName="Product" enableCountByExample="false" enableDeleteByExample="false" enableSelectByExample="false" enableUpdateByExample="false">
                    <columnOverride column="detail" jdbcType="VARCHAR" />
                    <columnOverride column="sub_images" jdbcType="VARCHAR" />
                </table>
        <table tableName="mall_shipping" domainObjectName="Shipping" enableCountByExample="false" enableDeleteByExample="false" enableSelectByExample="false" enableUpdateByExample="false"/>

    </context>
</generatorConfiguration>
```

最后一步，在IDEA自带的控制台上输入这句话。最后显示BUILD SUCCESS即可

```
mvn mybatis-generator:generate
```

生成之后一下子就完成了这么多的配置，是不是非常方便哈哈哈。

![image-20230529133745375](https://s2.loli.net/2023/05/29/dZG1fP7TAX5xaez.png)

### SpringBoot整合Mybatis配置问题

 在配置文件application.yml或者properties中主要加入这两句话或者下面那句，注意配置名和配置项中间要有一个空格(就是冒号和classpath中间)

```yml
mybatis:
  # 指定全局配置文件位置
  config-location: classpath:xxx/xxx.xml
  
  # 指定sql映射文件位置
  mapper-locations: classpath:xxx/*.xml
```

#### mapper-locations

以下转载自[mybatis mapper-locations作用_米兰的小铁匠z的博客-CSDN博客](https://blog.csdn.net/jayu_37/article/details/106549901)

顾名思义是一个定义mapper位置的属性
在yml或[properties](https://so.csdn.net/so/search?q=properties&spm=1001.2101.3001.7020)下配置，作用是实现mapper接口配置见mapper和接口的绑定。

当mapper接口和mapper接口对应的配置文件在

- 命名上相同

- 所在的路径相同

  ![在这里插入图片描述](https://s2.loli.net/2023/05/29/YUAcs9tiFqSNRV5.png)

则mapper-locations可以不用配置，配置也不会生效。

**但是**，如果当mapper接口和mapper接口对应的配置文件在

- 命名上不同
- 所在的路径不同

则需要配置mapper-locations才能实现接口的绑定

![在这里插入图片描述](https://s2.loli.net/2023/05/29/6prdtV2EqxgUFQ4.png)

至于如何填写，就是填写相对于application.yml的路径，如果的用来存放SQL的xml放在mappers里，那么同时利用通配符  *  匹配多个文件 ，在classpath:那里填写mappers/*.xml就好

![image-20230529165329583](https://s2.loli.net/2023/05/29/fKdtT4hvMgrxJ1Y.png)



**未完待续...**
