

# Mysql数据库

## 一、数据库

### 使用命令行窗口连接MYSQL数据库

```sql
mysql服务启动,在cmd输入net start mysql

1.mysql -h主机名-Р端口-u用户名-p密码

2.登录前，保证服务启动net start mysql

3.mysql -h 127.0.0.1 -P 3306 -u root -p20030515
```

```sql
#创建数据库 
CREATE DATABASE hsp_db01; 
#创建一个使用 utf8 字符集的 hsp_db02 数据库 
CREATE DATABASE hsp_db02 CHARACTER SET utf8 
#创建一个使用 utf8 字符集，并带校对规则的 hsp_db03 数据库 
CREATE DATABASE hsp_db03 CHARACTER SET utf8 COLLATE utf8_bin 
#校对规则 utf8_bin 区分大小 默认 utf8_general_ci 不区分大小写

#查看当前数据库服务器中的所有数据库
SHOW DATABASES
#查看创建的 hsp_db01 数据库的定义信息
SHOW CREATE DATABASE `hsp_db01` 	#为了规避关键字，可以使用反引号

#删除前面创建的 hsp_db01 数据库
DROP DATABASE hsp_db01
```

### 数据库的备份、恢复（导入）（必须在dos下运行）

#### 备份恢复数据库

==mysqldump== -u ==用户名== -p ==密码== -B ==数据库名== > ==路径==（例：d:\\\database.sql）

#### 备份恢复数据库的表

备份数据库的多个表不能带-B，否则会被认为是多个数据库

 mysqldump -u 用户名 -p 密码  数据库名 表1 表2 > 路径（例：d:\\\database.sql）

使用mysqldump命令来恢复一个叫data01数据库的一个叫table01的表(已有备份文件)。具体步骤如下：

- 打开cmd窗口，输入mysql -u 用户名 -p，然后输入mysql密码，进入mysql数据库。
- 输入use data01，选择您要恢复的数据库。
- 输入source d:/table01.sql，执行您的备份文件，恢复您的表。

#### 恢复数据库（必须在mysql命令行下运行）

先进入mysql命令行：mysql -u 用户名 -p 密码

恢复：source 文件路径.sql（例：d:\\\database.sql）

![image-20230318155923277](https://s2.loli.net/2023/03/18/UvgPuoAWM24qNLK.png)

左上角写了mysql -u 用户名 -p 的界面才叫mysql命令行

#### 备份，恢复数据库实践

先用==mysqldump== -u ==用户名== -p ==密码== -B ==数据库名== > ==路径==（例：d:\\\database.sql）

从数据库里备份一个到E盘里，然后在数据库中用drop指令将其删除

然后通过mysql -u 用户名 -p 密码进入mysql命令行，输入source d:\\\database.sql将其备份

## 二、Mysql基本数据类型

![image-20230318162214492](https://s2.loli.net/2023/03/19/qRKzSFTvNybED8k.png)

### 基本数据类型的使用细节

#### 数值型：

decimal（M,D)中的M默认为10（最大65），D默认为0（最大30），M是位数，D是小数点后有几位

如果没有指定 unsinged , 就是有符号

适合当做**价格**的数据类型

```sql
#创建一个无符号的tinyint类型数据，0~255
CREATE TABLE t4 (id TINYINT UNSIGNED)
```

int:

```sql
 CREATE TABLE `furn`(
 `id` INT(11)UNSIGNED
 )
 -- 技术细节
 -- 有时会看到 id int(11) ... 11 也表示显示宽度,存放的数据范围和int 配合zerofill
 -- 不够的前面会补0     67890 => int(11) 00000067890
 -- 够了的就不补        67890 => int(2)  67890
 
 -- 注意有无unsigned的区别,如果没有unsigned,就是没有正负符号也就是最多10位,此时一般设计为int(10)
 -- 有unsigned,这时候多一个符号位所以一般设计为int(11)
```



#### 字符串型：

![image-20230318214705705](https://s2.loli.net/2023/03/19/iLheOmG5paPxVuB.png)

VARCHAR虽然可以有65535个字节，但是首先去掉3个字节用于记录字段大小，所以最大为65532个字节，

如果用的编码是UTF-8的话，一个字符消耗3个字节，最多存放65532 / 3 = 21844个字符，

GBK是2个字节，最多存放65532 / 2 = 32766个字符。

```sql
#我们来创建一个数据类型为varchar，编码为GBK的表
CREATE TABLE t10 (`name` VARCHAR(32766)) CHARSET gbk;
```

![image-20230318215554723](https://s2.loli.net/2023/03/19/STPB8H7yDQFrLEM.png)

![image-20230318215740389](https://s2.loli.net/2023/03/19/QUdNnSFVrqcZgfo.png)

![image-20230318215813468](https://s2.loli.net/2023/03/19/Hgiob42I9c7qMRs.png)

![image-20230318220415081](https://s2.loli.net/2023/03/19/9DWTrHeGvAIh4UE.png)

#### 日期类型：

DATE / DATETIME / TIMESTAMP

```sql
CREATE TABLE t14 ( 
		birthday DATE ,		 -- 年月日
		job_time DATETIME, 	  -- 年月日 时分秒 
		login_time TIMESTAMP   -- 如果希望 login_time 列自动更新, 需要配置这两句
		NOT NULL DEFAULT CURRENT_TIMESTAMP 		-- 不允许为空，默认为当前时间
		ON UPDATE CURRENT_TIMESTAMP				-- 更新当前时间戳	
);
```

## 三、表

### 创建表

```sql
CREATE TABLE `emp` (
id INT, 
`name` VARCHAR(32), 
sex CHAR(1), 
brithday DATE,
entry_date DATETIME,
job VARCHAR(32), 
salary DOUBLE, 
`resume` TEXT
) 
CHARSET utf8 COLLATE utf8_bin ENGINE INNODB
#charset是character set的简写，即字符集
#字符集，校验规则，存储引擎不写的话就是默认的（默认的话跟随当前数据库）
```

要用cmd在一个叫database的数据库下新建一个表，你需要先用mysql命令登录到数据库，然后用use命令选择database数据库，再用create table命令创建表。具体的步骤如下：

- 打开cmd，输入mysql -u root -p，回车后输入密码，登录到数据库。
- 输入use database;，选择database数据库。
- 输入如上代码完成创建。

### 删除表

```sql
DROP TABLE `emp`
```

### 修改表

举例演示:

```sql
ALTER TABLE emp
-- 员工表 emp 的上增加一个 image 列，varchar 类型(要求在 birthday 后面)。
ADD image VARCHAR(32) NOT NULL DEFAULT '' AFTER birthday ;
-- DEFAULT ''默认为''这个字符

-- 修改 job 列，使其长度为 60。
ALTER TABLE emp
MODIFY job VARCHAR(60) NOT NULL DEFAULT '' ;

-- 删除 sex 列。
ALTER TABLE emp
DROP sex;

-- 表名改为 employee。
RENAME TABLE emp TO employee;

-- 修改表的字符集为 utf8
ALTER TABLE emp CHARACTER SET utf8;

-- 列名 name 修改为 user_name
ALTER TABLE emp;

-- 显示表结构，可以查看表的所有列
DESC emp ;
```

## 四、CRUD增删改查

### 1. Insert

```sql
用法:
INSERT INTO 表名 (列名1,列名2...)
values (对应列名的值);
```

```sql
INSERT INTO `emp` (id, birthday)
VALUES(1, '2003-05-15');	-- 注意date等日期类型一定要按照规则写否则报错或者错位写入
```

#### insert使用细节

- 插入的数据应与字段的数据类型相同

- 数据的长度应在列的规定范围内

- 在 values 中列出的数据位置必须与被加入的列的排列位置相对应

- 字符和日期型数据应包含在单引号中

- 列可以插入空值[前提是该字段允许为空]

- 可以用 values ( ),( ) ,( ) ;  的形式添加多条记录 

  ```sql
  INSERT INTO `goods` (id, goods_name, price) 
  VALUES(50, '三星手机', 2300),(60, '海尔手机', 1800);
  ```

- 如果是给表中的所有字段添加数据，可以不写前面的字段名称

- 默认值的使用，当不给某个字段值时，如果有默认值就会添加默认值，否则报错

   -- 如果某个列 没有指定 not null ,那么当添加数据时，没有给定值，则会默认给 null 

  -- 如果我们希望指定某个列的默认值，可以在创建表时通过 not null default xxx 指定

```sql
#表里有3个数据,id,goodname,price有默认值10
INSERT INTO `goods` (id, goods_name)
VALUES(80, '格力手机');
#最后会插进去一个(80,格力手机,10)进去
```



### 2. Update

```sql
用法:
UPDATE 表名 SET 列名 = 值 WHERE 布尔表达式，筛选满足条件的行
```

```sql
#将所有员工薪水修改为 5000 元。如果不带 where 条件，会修改该列所有的记录，因此要小心
UPDATE employee SET salary = 5000;

#将姓名为 小妖怪 的员工薪水修改为 3000 元
UPDATE employee
SET salary = 3000
WHERE user_name = '小妖怪';  


#也可以修改多个列的值
UPDATE employee
SET salary = salary + 1000 , job = '出主意的' 
WHERE user_name;
```



### 3. Delete

```sql
用法:
DELETE FROM 表名
where 布尔等值式;   -- 如果不带 where 条件，会修改所有记录,小心删库跑路
```

```sql
#删除表中名称为’老妖怪’的记录。
DELETE FROM employee
WHERE user_name = '老妖怪';
```

#### delete使用细节

- delete不能删除某一列的值，只能使用update将列设为空

- **如果不带 where 条件，会修改所有记录,小心删库跑路！！！**

  

### 4. 单表Select

#### 用法1：基础查询

```SQL
SELECT [DISTINCT] * 或 指定列 FROM 表名;   
-- distinct功能是去重(写在from前的每一个列的值都相同才去)
-- *代表所有列,也可以写多个列(用逗号隔开)

比如:
-- 查询表中所有学生的信息
SELECT * FROM student; 

-- 查询表中所有学生的姓名和对应的英语成绩
SELECT `name`,english FROM student;

-- 过滤表中重复数据
SELECT DISTINCT `name`,english FROM student;
```

#### 用法2：列名使用表达式进行运算

```sql
比如:
-- 统计每个学生的总分
SELECT `name`, (chinese + english + math ) FROM student;   
#但是使用这种方法系统会自动命名第二列为chinese + english + math,所以我们可以采用下面的方式美化一下

-- 使用AS给列命名
SELECT `name` AS '名字', (chinese + english + math ) AS total
FROM student;
```

#### 常用运算符表

![image-20230319151215414](https://s2.loli.net/2023/03/19/wjtvcdIpK85PVlS.png)

#### 用法3:  使用where子句运算符,进行过滤查询

```sql
SELECT [DISTINCT] * 或 指定列 FROM 表名 WHERE 布尔表达式;	-- select定最后显示多少列,from定从哪个表查询,where定最后显示满足条件的行

比如:
-- 查询姓名为赵云的学生成绩
SELECT * FROM student
WHERE `name` = '赵云';

-- 查询 math 大于 60 并且(and) id 大于 4 的学生成绩
SELECT * FROM student
WHERE math >60 AND id > 4;

-- 查询英语成绩大于语文成绩的同学
SELECT * FROM student
WHERE english > chinese;

-- 查询总分大于 200 分 并且 数学成绩小于语文成绩 并且 姓赵的学生, 赵% 表示名字以赵开头
SELECT * FROM student
WHERE (chinese + english + math) > 200 AND
math < chinese AND `name` LIKE '赵%';

-- 查询英语分数在 80－90 之间的同学。
SELECT * FROM student
WHERE english >= 80 AND english <= 90;
SELECT * FROM student
WHERE english BETWEEN 80 AND 90; -- between .. and .. 是 闭区间

-- 查询数学分数为 89,90,91 的同学。
SELECT * FROM student
WHERE math = 89 OR math = 90 OR math = 91;
SELECT * FROM student
WHERE math IN (89,90,91);
```

#### 用法4: 使用 order by子句排序查询结果

```sql
SELECT * FROM 表名 ORDER BY 列名(排序的依据) [DESC];		-- 默认为升序,DESC改为降序
比如:
-- 对数学成绩排序后输出【升序】。
SELECT * 
FROM student
ORDER BY math; 

-- 对总分按从高到低的顺序输出 [降序] -- 使用别名排序
SELECT `name` , (chinese + english + math) AS '总分' 
FROM student
ORDER BY total_score DESC; 

-- 对姓韩的学生成绩[总分]排序输出(升序) where + order by
SELECT `name`, (chinese + english + math) AS '总分' FROM student
WHERE `name` LIKE '韩%' 
ORDER BY total_score;
```

#### 用法5：使用like模糊查询

```sql
比如：
-- %  表示 0 到多个的任意字符
-- _  表示单个任意字符  其中GBK中需要两个_ _来表达一个字符

-- 显示首字符为 S 的员工姓名和工资
SELECT name, salary FROM emp
WHERE name LIKE 'S%' 
-- 显示第三个字符为大写 O 的所有员工的姓名和工资
SELECT name, salary FROM emp
WHERE name LIKE '__O%'
```

#### 用法6:使用DESC查询表结构

```sql
DESC 表名
```

#### 用法7:使用LIMIT分页查询

```sql
select * from table limit 参数1,参数2;  	-- 参数1 : 代表偏移量offset（从第offset + 1数据开始查），注意是从0开始，所以应该+1代表第一个数据
										-- 参数2 : 取出的数据条数rows
/* 查询第1-10条数据 */
SELECT * FROM Student LIMIT 0,10;
/* 查询第11-20条数据 */
SELECT * FROM Student LIMIT 10 , 10;        -- 讲到这我们会想知道如何快速算出 参数1，接下来我们看一下计算公式
```

**公式**

- offset：(要查询的页数 - 1) * 每页显示的数据条数
- rows：显示的数据条数

在进行分页之前，要注意一下是否超出了最大页数，所以我们需要先根据数据总量来得出总页数，这需要用到统计函数**COUNT**和向上取整函数**CEIL**，SQL操作如下：

```sql
#获得数据总条数
SELECT COUNT(*) FROM Student;
#假设每页显示10条，则直接进行除法运算，然后向上取整，这样我们就获得了总页数，在实际操作中，要注意一下是否超出了最大页数
SELECT CEIL(COUNT(*) / 10) AS '总页数' FROM Student;
```

### 5. 多表查询Select

**多表查询的条件不能少于 要查询的表个数 -  1, 否则会出现笛卡尔集**

#### 自连接

```sql
自连接的特点 
1. 把同一张表当做两张表使用
2. 需要给表取别名 如果from同一张表两次，会报错，所以需要对表重命名，无需使用 AS，直接表加别名就好
3. 列名有时不明确，可以通过 AS 指定列的别名 

举例：
#显示公司员工名字和他上级的名字，员工名字和上级的名字都在emp表
#其中员工和上级是通过 emp 表的 manager列 关联
SELECT worker.name AS '职员名' , 
boss.name AS '上级名' 
FROM emp worker, emp boss
WHERE worker.manager = boss.empno
```

#### 子查询（嵌套查询）

- 单行子查询：只返回一行数据
- 多行子查询：返回多行数据  ，因为子查询的结果是不确定的，所以使用比较运算符 IN （用于确定IN前面的值是否与IN的括号中的值或子查询中的任何值匹配）

```sql
#多行子查询中使用in运算符
#查询和10号部门的工作相同的雇员的 名字、岗位、工资、部门号, 但是不含10号部门的员工
select ename, job, sal, deptno
from emp
where job in (
SELECT DISTINCT job
FROM emp
WHERE deptno = 10
) and deptno != 10

#多行子查询中使用 all 操作符
#查询工资比部门 30 的所有员工的工资高的员工的姓名、工资和部门号
SELECT ename, sal, deptno
	FROM emp
	WHERE sal > ALL(
		SELECT sal
			FROM emp
    		WHERE deptno = 30
)
#多行子查询中使用 any 操作符
#查询工资比部门 30 的任意/其中一个员工的工资高的员工的姓名、工资和部门号
SELECT ename, sal, deptno
	FROM emp
	WHERE sal > any(
		SELECT sal
			FROM emp
			WHERE deptno = 30
)
```

- 把子查询当做临时表使用 : 避免mysql重复计算表，提高查询性能，并且可以解决很多复杂的查询

  ```sql
  #查找每个部门工资 高于 他们自己部门平均工资的人的资料
  -- 1. 先得到每个部门的 部门号和 对应的平均工资
  SELECT deptno, AVG(sal) AS avg_sal
  	FROM emp 
  	GROUP BY deptno  -- 当聚合列和非聚合列出现在一起时必须使用group by语句
  	
  -- 2. 把上面的结果当做子查询, 和 emp 进行多表查询
  SELECT ename, sal, temp.avg_sal, emp.deptno
  	FROM emp,（
  	SELECT deptno, AVG(sal) AS avg_sal
  		FROM emp
  		GROUP BY deptno
      ) temp	-- 把子查询当作一个临时表使用，重命名为temp
  WHERE emp.deptno = temp.deptno AND emp.sal > temp.avg_sal
  ```

#### 合并查询

- union all	单纯取并集不去重

- union   去重

  ```sql
  举例：
  -- union all 就是将两个查询结果合并，不会去重
  SELECT ename,sal,job FROM emp WHERE sal>2500 -- 5条
  UNION ALL
  SELECT ename,sal,job FROM emp WHERE job='MANAGER' -- 3条
  -- 最后有8条
  ```

#### 6.连接

##### 自然连接

**通常用于将两个关系表中具有相同属性的行组合在一起，以便我们可以从两个表中同时获取信息。**

**自然连接是一种特殊的等值连接**（区别在于等值连接需要指定值），他要求两个表中进行连接的必须是相同的属性列（名字相同），无须添加连接条件，**并且在结果中消除重复的属性列。**

natural表1：

![image-20230322113310173](https://s2.loli.net/2023/03/22/s3VGMznDryISKAl.png)

natural表2：

![image-20230322113315186](https://s2.loli.net/2023/03/22/FmCS5JYf1ua7DeO.png)

```sql
select 列名 from 左表 natural join 右表
-- 将2个表自然连接一下
SELECT * FROM `natural` NATURAL JOIN `natural2` 
```

说人话，直接先把两张表的列名**去重**后列出来

![image-20230322114724079](https://s2.loli.net/2023/03/22/p9mGnq78CPirHTJ.png)

然后开始对比他们两相同的列（id）的值，我们就找到id = 2时相同，直接把左表和右表连接起来，结果：

![image-20230322113511257](https://s2.loli.net/2023/03/22/rQV2DmRowinXqAB.png)

##### 内连接

内连接基本与自然连接相同，不同之处在于自然连接的是同名属性列的连接，而内连接则不要求两属性列同名，可以用**using或on**来指定某两列字段相同的连接条件。

```sql
select 列名 from 左表 inner join 右表 on 条件  -- inner可以省略
```

##### 外连接

**通常用于查询在一张表中存在但在另一张表中不存在的记录**

- 左外：只保留左表的所有元素，右表没有则置null

- 右外：同理

- 全外连接 ：mysql不支持

  ```sql
  用法：
  select 列名 from 左表 left join 右表 on 条件
  select 列名 from 左表 right join 右表 on 条件
  
  #下面是一个简单的例子，演示如何使用外连接查询两张表中的数据。
  假设我们有两张表：员工表和部门表。员工表包含员工的姓名和所属部门，部门表包含部门名称和部门编号。
  
  #员工表：
  姓名	  部门编号
  张三		1
  李四		2
  王五		3
  赵六		NULL
  
  #部门表：
  部门编号	部门名称
  1			销售部
  2			人事部
  如果我们想要查询所有员工及其所属部门，可以使用以下SQL语句：
  
  SELECT 员工姓名, 部门名称 FROM 员工表 LEFT JOIN 部门表 ON 员工的部门编号 = 部门的编号;
  
  执行上述语句后，将得到以下结果：
  
  姓名	部门名称
  张三	销售部
  李四	人事部
  王五	NULL
  赵六	NULL
  可以看到，上述查询结果中包含了员工表中所有记录，并且显示了每个员工所属的部门。对于那些在部门表中没有匹配记录的员工（即王五和赵六），其所属部门显示为NULL。
  ```

  

## 五、函数

### 1.聚簇函数（合计、统计）

#### count：

```sql
SELECT COUNT(*) 或 (列)FROM 表名 WHERE 表达式;
-- count(*) 和 count(列) 的区别
-- count(*) 返回满足条件的行数
-- count(列): 统计满足条件的某列个数，会排除为 null 的
```

#### sum:

```sql
SELECT SUM(列) FROM 表名 WHERE 表达式;
比如:
-- 统计一个班级语文、英语、数学各科的总成绩并使用别名
SELECT SUM(math) AS math_total_score,
SUM(english) AS english_total_score,
SUM(chinese) AS chinese_total_score
FROM student;

-- 统计一个班级语文成绩平均分
SELECT SUM(chinese)/ COUNT(*) FROM student;

-- 注意有些数据类型不能统计比如字符串之类
SELECT SUM(`name`) FROM student;  #不报错但无意义
```

#### avg:

```sql
SELECT AVG(列) FROM 表名 WHERE 表达式;
比如:
-- 求一个班级数学平均分？
SELECT AVG(math) FROM student; 

-- 求一个班级总分平均分
SELECT AVG(math + english + chinese) FROM student;

-- 求班级最高分和最低分
SELECT MAX(math + english + chinese), MIN(math + english + chinese)
FROM student; 

-- 求出班级数学最高分和最低分并使用别名
SELECT MAX(math) AS math_high_socre,
MIN(math) AS math_low_socre
FROM student;
```

#### 使用group by 对列进行分组  ：

**==注意==：当聚合列和非聚合列出现在一起时必须使用group by语句**

```sql
SELECT 列名 FROM 表名 group by 列名(从select后面的几个列名之中挑);
比如:
-- 显示每个部门的每种岗位的平均工资和最低工资
SELECT AVG(salary), MIN(salary) ,  department, job
FROM emp GROUP BY department, job;

-- GROUP BY也可以对查询结果进行去重
```

#### 使用 having 对group by 分组后的结果进行过滤

```sql
SELECT 列名 FROM 表名 group by 列名 having 布尔表达式;
比如:
-- 显示平均工资低于 2000 的部门编号和它的平均工资 并使用别名
SELECT AVG(salary) AS '平均工资', 
deptno AS '部门编号'
FROM emp 
GROUP BY deptno
HAVING AVG(salary) < 2000;	# 注意HAVING 子句中不能使用别名，只能使用原始的表达式
```

#### 2.字符串函数

#### 3.数学函数

#### 4.日期函数

#### 5.加密函数

MySQL有一些加密函数，可以用来对字符串进行加密和解密。常用的有以下几种：

```sql
AES_ENCRYPT(str,key)：返回用密钥key对字符串str利用高级加密标准算法加密后的结果，调用AES_ENCRYPT的结果是一个二进制字符串，以BLOB类型存储。

AES_DECRYPT(crypt_str,key)：返回用密钥key对字符串crypt_str利用高级加密标准算法解密后的结果1。

DECODE(str,key)：使用key作为密钥解密加密字符串str2。

ENCODE(str,key)：使用key作为密钥加密字符串str2。

MD5(str)：返回str的MD5校验和，是一个32位十六进制数2。

#演示MD5使用
-- 插入
INSERT INTO users (username, password) VALUES ('admin', MD5('123456'));
-- 这样就可以把用户名和密码加密后存储在users表中。你也可以使用MD5函数来验证用户输入的密码是否正确，例如：
SELECT * FROM users WHERE username = 'admin' AND password = MD5('123456');

这样就可以根据用户名和密码查询users表中是否有匹配的记录。你明白了吗？
```



### 六、约束

MySQL约束语句是用来限制表中数据的条件。MySQL支持以下几种约束：

- 主键（PRIMARY KEY）：唯一标识表中的每一行。
- 外键（FOREIGN KEY）：引用另一个表中的主键或唯一键。
- 唯一（UNIQUE）：保证列中的值不重复。
- 非空（NOT NULL）：保证列中的值不为空。
- 检查（CHECK）：保证列中的值满足指定的条件。
- 默认（DEFAULT）：为列提供默认值。

可以在创建表或修改表时添加或删除约束。

我来介绍一下这6种约束的用法。

#### 主键（PRIMARY KEY）

**不能重复而且不能为 null**

在创建表时，可以使用PRIMARY KEY关键字来指定一个或多个列作为主键，例如：

```sql
CREATE TABLE student (
  id INT PRIMARY KEY,
  name VARCHAR(20),
  age INT
);
#复合主键
CREATE TABLE t(
    id INT , 
    `name` VARCHAR(32),
	PRIMARY KEY (id, `name`) 	-- 这里就是复合主键
);
```

也可以在创建表后，使用ALTER TABLE语句来添加或删除主键，例如：

```sql
ALTER TABLE student ADD PRIMARY KEY (id);
ALTER TABLE student DROP PRIMARY KEY;
```

#### 外键（FOREIGN KEY）

外键是用于建立和加强两个表数据之间的链接的列。 外键的作用是保持数据的一致性、完整性，主要体现在下面两个方面

- 约束作用：外键可以防止在**外键所在表**中插入无效的数据，或者删除**外键指向表**中引用的数据。
- 连接作用：外键可以通过连接操作实现两个表之间的联合查询。

在创建表时，可以使用FOREIGN KEY关键字来指定一个或多个列作为外键，并引用主表中的主键(primary key)或唯一键(unique)

```sql
用法:FOREIGN KEY (外键所在表的外键列列名) REFERENCES 主表名(主表中的主键或唯一键)

比如:
-- 创建 主表 my_class  (外键指向表)
CREATE TABLE my_class (
id INT PRIMARY KEY , 	-- 班级编号
`name` VARCHAR(32) NOT NULL DEFAULT ''
); 

-- 创建 从表 my_stu	  (外键所在表)   
CREATE TABLE my_stu (
id INT PRIMARY KEY , -- 学生编号
`name` VARCHAR(32) NOT NULL DEFAULT '', 
class_id INT , -- 学生所在班级的编号
FOREIGN KEY (class_id) REFERENCES my_class(id)	-- 外键
);
```

![image-20230320230325752](https://s2.loli.net/2023/03/20/xqePbNnwFfjmBd9.png)

#### 唯一（UNIQUE）

- 在创建表时，可以使用UNIQUE关键字来指定一个或多个列的值不重复，例如：

```sql
CREATE TABLE user (
  id INT PRIMARY KEY,
  username VARCHAR(20) UNIQUE NOT NULL,		-- unique + not null = primary key
  password VARCHAR(20) NOT NULL
);
```

#### 非空（NOT NULL）

在创建表时，可以使用NOT NULL关键字来指定一个列的值不为空，例如：

```sql
CREATE TABLE book (
  id INT PRIMARY KEY,
  title VARCHAR(50) NOT NULL,
);
```

#### 检查（CHECK）

- 在创建表时，可以使用CHECK关键字来强制指定一个列的值满足指定的条件，（注意MySQL目前不支持检查约束）

```sql
CREATE TABLE employee (
  id INT PRIMARY KEY,
  name VARCHAR(20) NOT NULL,
  salary DECIMAL(10,2),
  CHECK (salary >0)
);
```

#### 默认（DEFAULT）

- 在创建表时，可以使用DEFAULT关键字来为列提供默认值，在插入数据时如果没有指定该列的值，则会自动填充默认值。默认值必须是常量。如果要设置当前日期时间作为默认值，则需要用CURRENT_TIMESTAMP函数。例如：

```sql
CREATE TABLE post (
  id INT PRIMARY KEY,
  title VARCHAR(50) NOT NULL,
  content TEXT,
  create_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 七、自增长

目的：当我们希望一列每插一个数据进去会自动增长时可以使用自增长,这样可以避免手动输入主键值，也可以保证主键值的唯一性

要实现MySQL自增长，需要满足以下条件：

- **只能有一个字段**使用AUTO_INCREMENT约束
- 该字段必须**有唯一索引**（主键或主键的一部分）
- 该字段必须**具备NOT NULL**属性（要么直接primary key 要么就加 not null ）
- 该字段只能是**整型**（TINYINT、SMALLINT、INT、BIGINT)

```sql
用法：
字段名 整型 primary key auto_increment

-- 创建一个名为student的表，包含id、name和age三个字段
CREATE TABLE student (
  id INT NOT NULL AUTO_INCREMENT, -- id字段为自增长字段，必须为整数类型且非空
  name VARCHAR(20) NOT NULL, -- name字段为字符串类型，非空
  age INT NOT NULL, -- age字段为整数类型，非空
  PRIMARY KEY (id) -- id字段为主键，保证唯一性
);

-- 插入两条记录，不需要指定id值，数据库会自动赋值
INSERT INTO student (name, age) VALUES ('Alice', 18);
INSERT INTO student (name, age) VALUES ('Bob', 19);

-- 查询表中的数据，可以看到id值自动递增了
SELECT * FROM student;

输出结果如下：
id	name	age
1	Alice	18
2	Bob		19

-- 如果插入的时候插入一个不是自增长的值时，再插入数据则从上一个开始自增
INSERT INTO student (id ,name, age) VALUES (666,'Tom', 18);
INSERT INTO student (name, age) VALUES ('John', 18);
-- 查询
SELECT * FROM student;

输出结果如下：
id	name	age
1	Alice	18
2	Bob		19
666  TOM     18
667  John    18
```

### 八、索引

MySQL索引是一种帮助MySQL高效获取数据的数据结构。它可以根据一列或多列的值进行排序，从而加快数据库的查询速度

MySQL中常用的索引结构有B-TREE，B+TREE，HASH等。不同的索引结构适用于不同的查询场景，例如：

- B-TREE索引适用于全键值、键值范围和前缀查找
- B+TREE索引是B-TREE索引的改进版，具有更高的查询效率和空间利用率
- HASH索引适用于等值查找，但不支持范围查找和排序

```sql
用法：
CREATE INDEX 索引名 ON 表名 (列名);
ALTER TABLE 表名 ADD INDEX 索引名 （列名）;
在创建表时，可以使用 PRIMARY KEY，UNIQUE或INDEX关键字来定义主键、唯一或普通索引12。例如：
-- 创建一个名为student的表，包含id、name和age三个字段

CREATE TABLE student (
  id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,	 -- id字段为主键索引
  name VARCHAR(20) NOT NULL UNIQUE,			    -- name字段为唯一索引
  age INT NOT NULL INDEX					   -- age字段为普通索引
);

在已有的表上，可以使用 ALTER TABLE 或 CREATE INDEX语句来创建索引
-- 在student表上创建一个名为idx_name_age的复合索引，包含name和age两个字段
ALTER TABLE student ADD INDEX idx_name_age (name, age);
-- 或者
CREATE INDEX idx_name_age ON student (name, age);

单独创建主键索引，例如：
ALTER TABLE student ADD PRIMARY KEY (id);
删除主键索引，例如：
ALTER TABLE student DROP PRIMARY KEY;

在已有的表上，可以使用 DROP INDEX 或 ALTER TABLE语句来删除索引。
-- 删除student表上的idx_name_age索引
DROP INDEX idx_name_age ON student;
-- 或者
ALTER TABLE student DROP INDEX idx_name_age;
```

#### 数据库索引的优点与缺点

- 优点：
  - 索引可以提高数据检索的效率，降低数据库的I/O成本
  - 索引可以通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗
  - 索引可以加快表与表之间的连接，提高关联查询的性能
  - 索引可以保证行或列的唯一性，生成唯一的row
- 缺点：
  - 索引需要占用额外的存储空间，增加数据库的负担
  - 索引会降低数据更新（插入、删除、修改）的速度，因为每次更新都需要维护索引结构
  - 不恰当地使用索引可能导致查询性能下降或者无法使用索引

#### 创建索引的条件

![image-20230321195334139](https://s2.loli.net/2023/03/21/vp7WIa8VbuFEHXh.png)

### 九、事务

事务是指一组操作量大，复杂度高的数据操作，它们要么都执行成功，要么都执行失败，不会出现中间状态。

- 原子性（Atomicity）：事务中的所有操作要么全部完成，要么全部不完成，不会停滞在中间某个环节。
- 一致性（Consistency）：事务在完成时，必须使所有数据恢复到一致性状态。在一致性状态下，所有相关的数据规则都得到满足。
- 隔离性（Isolation）：事务的隔离性是指多个用户并发访问数据库时，一个用户的事务不能被其他用户的事务所干扰，多个并发事务之间要相互隔离。
- 持久性（Durability）：持久性是指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的。

MySQL默认每条DML（增删改）操作都是一个事务。可以通过 SHOW VARIABLES LIKE 'autocommit’  查看自动提交模式是否开启。如果关闭自动提交模式，则需要手动使用COMMIT或ROLLBACK来提交或回滚事务

MySQL只有使用了InnoDB数据库引擎的数据库或表才支持事务。

```sql
START TRANSACTION; / SET autocommit = off;    -- 开启一个新的事务
SAVEPOINT 保存点名;	    -- 设置保存点
ROLLBACK TO 保存点名;   -- 回滚到某事务,
					  -- (注意)假如有按顺序的3个保存点A,B,C  如果回滚到A时,B和C保存点就被删除
ROLLBACK;			  -- 回滚到最开始的状态
COMMIT; 		       -- 提交当前事务,一旦执行,就不能再回滚了,保存点也被删除,也只有提交事务之后,其他客户端才(可能)看见数据库的更新,为什么是可能,引用到后面的隔离级别知识点
```

### 十、事务隔离级别

MySQL事务的隔离级别。事务的隔离级别是指多个并发事务之间如何相互影响的问题，不同的隔离级别会导致不同的并发问题。由于篇幅问题这里不能细讲，只是大概概括一下，0基础的同学自行看《MySQL是怎样运行的》。

#### 四个事务隔离级别

- 读未提交（read uncommitted）：最低的隔离级别，一个事务可以读取另一个未提交事务的修改数据，可能会出现脏读，不可重复读，幻读
- 读已提交（read committed）：一个事务只能读取另一个已提交事务的修改数据，可以避免脏读，但可能会出现不可重复读、幻读
- 可重复读（repeatable read）：MySQL**默认隔离级别**，一个事务在执行期间看到的数据始终和这个事务在启动时看到的数据是一致的，可以避免脏读和不可重复读，但可能会出现幻读，不过InnoDB实现的RR通过mvcc机制避免了这种幻读现象。[对于MVCC以及如何在RR上解决幻读的理解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/180350695#:~:text=一、InnoDB如何解决幻读 1 幻读 ：在InnoDB的可重复度隔离级别下，使用当前读，一个事务前后两次查询同一个范围，后一次查询会看到期间新插入的行； 2 幻读的影响 ：会导致一个事务中先产生的锁，无法锁住后加入的行，会产生数据一致性问题； 3,5 同时添加间隙锁与行锁称为 Next-key lock ，注意间隙锁只有在InnoDB的可重复度隔离级别下生效； 6 MVCC只实现读取已提交和可重复读，InnoDB在可重复度的隔离级别下，使用MVCC%2BNext-key lock解决幻读；)==面试重点之一==，核心就是采用了一个间隙锁（**Next-key lock**），行锁只能锁一行，所以对于幻读这种范围查询的数据不能避免数据的插入，
- 可串行化（serializable）：最高的隔离级别，所有事务都必须按照顺序执行，可以避免所有并发问题，但性能最差。

**那么脏读、不可重复读、幻读都是什么意思呢?**

- 脏读（dirty read）：一个事务读取了另一个还未提交事务的修改数据，如果后者回滚了操作，则前者所读取的数据就是不正确的

- 不可重复读（non-repeatable read）：一个事务在**执行期间**多次读取**同一份数据**(比如某行某列的一个值)，但由于另一个事务对该数据进行了==update==并提交，导致前者两次读取的结果不一致

- 幻读（phantom read）：一个事务在**执行期间**多次查询**某段**数据(比如一张表)，但由于另一个事务对该段数据进行了==delete，insert==并提交，导致前者查询的结果不一致

- **不可重复读的重点是修改:**

  同样的条件, 你读取过的数据, 再次读取出来发现值不一样了

  **幻读的重点在于新增或者删除**

  同样的条件, 第1次和第2次读出来的记录数不一样

  当然, 从总的结果来看, 似乎两者都表现为两次读取的结果不一致.

  但如果你从**控制的角度**来看, 两者的区别就比较大

  - 对于前者, 只需要锁住满足条件的记录
  - 对于后者, 要锁住满足条件及其相近的记录

![image-20230321203008519](https://s2.loli.net/2023/03/21/wo6tHbmK3Sv5V7d.png)

#### 设置事务隔离级别

```sql
在MySQL中，可以使用以下命令设置事务的隔离级别：
set session transaction isolation level [隔离级别];  -- read committed /read uncommitted / repeatable read / serializable
例如，设置隔离级别为读取未提交：
set session transaction isolation level read uncommitted;
设置隔离级别为读取已提交：
set session transaction isolation level read committed;

你可以使用以下语句查看当前数据库的隔离级别：
SELECT @@global.tx_isolation;  -- 查看全局事务隔离级别
SELECT @@tx_isolation;	   	   -- 会话事务当前级别
```

### 十一、锁理论

上一章唠叨了事务并发执行时可能带来的各种问题，并发事务访问相同记录的情况大致可以划分为3种：

- 读-读 情况：即并发事务相继读取相同的记录。 读取操作本身不会对记录有一毛钱影响，并不会引起什么问题，所以允许这种情况的发生。 
- 写-写 情况：即并发事务相继对相同的记录做出改动。可能发生脏写，不过MySQL任何一种隔离级别都解决了这个事，所以不再赘述。
- 读-写 或 写-读 情况：也就是一个事务进行读取操作，另一个进行改动操作。可能发生 脏读 、 不可重复读 、 幻读 的问题。

怎么解决 脏读 、 不可重复读 、 幻读 这些问题呢？其实有两种可选的解决方案：

方案一：读操作利用多版本并发控制（ MVCC ），写操作进行加锁。具体看书或者上面的链接

方案二：读、写操作都采用 加锁 的方式。

- 如果我们的一些业务场景不允许读取记录的旧版本，而是每次都必须去读取记录的最新版本，比方在银行存款的事务中，你需要先把账户的余额读出来，然后将其加上本次存款的数额，最后再写到数据库中。在将账户余额读取出来后，就不想让别的事务再访问该余额，直到本次存款事务执行完成，其他事务才可以访问账户的余额。这样在读取记录的时候也就需要对其进行 加锁 操作，这样也就意味着 **读** 操作和 **写** 操作也像 **写-写** 操作那样排队执行。



接下来我们来正式讲讲锁吧。

#### 锁定读

- 共享锁和独占锁（读锁和写锁）（S锁和X锁）

   S锁和S锁是兼容的， S锁和X锁是不兼容的， X锁和X锁也是不兼容的

  ```sql
  对读取的记录加 S锁 ： 
  SELECT ... LOCK IN SHARE MODE;
  
  对读取的记录加 X锁 ：
   SELECT ... FOR UPDATE;
  ```

####  写操作中锁定读的应用

DELETE 、 UPDATE 、 INSERT 这三种：

- DELETE ： 对一条记录做 DELETE 操作的过程其实是先在 B+ 树中定位到这条记录的位置，然后获取一下这条记录的 X 锁 ，然后再执行 delete mark 操作。我们也可以把这个定位待删除记录在 B+ 树中位置的过程看成是一个获 取 X锁 的 锁定读 。 
- UPDATE ： 在对一条记录做 UPDATE 操作时分为三种情况：
  -  如果**未修改该记录的键值**并且被更新的列占用的**存储空间在修改前后未发生变化**，则先在 B+ 树中定位 到这条记录的位置，然后再获取一下记录的 X锁 ，最后在原记录的位置进行修改操作。其实我们也可以 把定位到待修改记录在 B+ 树中位置的过程看成是一个获取 X锁 的 锁定读 。 
  - 如果**未修改该记录的键值**并且至少有一个被更新的列占用的**存储空间在修改前后发生变化**，则先在 B+ 树中定位到这条记录的位置，然后获取一下记录的 X锁 ，将该记录彻底删除掉（就是把记录彻底移入垃圾链表），最后再插入一条新记录。这个定位待修改记录在 B+ 树中位置的过程看成是一个获取 X 锁 的 锁定读 ，新插入的记录由 INSERT 操作提供的 隐式锁 进行保护。 
  - 如果**修改了该记录的键值**，则相当于在原记录上做 DELETE 操作之后再来一次 INSERT 操作，加锁操作就 需要按照 DELETE 和 INSERT 的规则进行了。
-  INSERT ： 一般情况下，新插入一条记录的操作并不加锁，设计 InnoDB 的大叔通过一种称之为 隐式锁 的东东来保护这 条新插入的记录在本事务提交前不被别的事务访问，更多细节我们后边看哈～

#### 多粒度锁

- 行锁：我们上面将的都是行锁，都是对应一条记录的。

- 表锁：与上面基本相同，就是把S和X锁放到表上

  我们以大学教学楼中的教室为例来分析一下加锁的情况： 

  教室一般都是公用的，我们可以随便选教室进去上自习。当然，教室不是自家的，一间教室可以容纳很多同 学同时上自习，每当一个人进去上自习，就相当于在教室门口挂了一把 S锁 ，如果很多同学都进去上自习， 相当于教室门口挂了很多把 S锁 （类似行级别的 S锁 ）。 有的时候教室会进行检修，比方说换地板，换天花板，换灯管啥的，这些维修项目并不能同时开展。如果教 室针对某个项目进行检修，就不允许别的同学来上自习，也不允许其他维修项目进行，此时相当于教室门口 会挂一把 X锁 （类似行级别的 X锁 ）。

  上边提到的这两种锁都是针对 **教室/一行记录** 而言的，不过有时候我们会有一些特殊的需求：

  - 有领导要来参观教学楼的环境
    - 确保教学楼中的没有正在维修的教室，如果有正在维修的教室， 需要等到维修结束才可以对教学楼整体上 S锁 
  - 学校要占用教学楼进行考试
    - 确保教学楼中的没有上自习的教室以及正在维修的教室，如果有上自习的教室或者正在维修的教室，需要等到全部上自习的同学都上完自习离开，以及维修工维修完教室离开后才可以对教学楼整体上X锁

  

  我们在对教学楼整体上锁（ 表锁 ）时，怎么知道教学楼中有没有教室已经被上锁（ 行锁 ）了呢？依次检查每间教室门口有没有上锁？那这效率也太慢了吧！遍历是不可能遍历的，这辈子也不可能遍历的，于是乎设计 InnoDB 的大叔们提出了一种称之为 **意向锁**

  - 意向共享锁（意向读锁）（IS锁）
  - 意向独占锁（意向写锁）（IX锁）
    - 如果有学生到教室中上自习，那么他先在整栋教学楼门口放一把 IS锁 （表级锁），然后再到教室门口放一 把 S锁 （行锁）
    - 如果有维修工到教室中维修，那么它先在整栋教学楼门口放一把 IX锁 （表级锁），然后再到教室门口放一把X锁 （行锁）

  它们的提出仅仅为了在之后加表级别的S锁和X锁时可以快速判断表中的记录是否被上锁，也就是说其实IS锁和IX锁是兼容的，IX锁和IX锁也是兼容的。

###  MySQL中的行锁和表锁

不同存储引擎对锁的支持也是不一样的，我们重点还是讨论 InnoDB 存储引擎中的锁，对于其他的存储引擎来说，它们只支持表级锁，而且这些引擎并不支持事务。

#### InnoDB中的行级锁

设计 InnoDB 的大叔很有才，一个 行锁 玩出了各种花样，也就是把 行锁 分成了各种类型。换句话说即使对同一条记录加 行锁 ，如果类型不同，起到的功效也是不同的。

- 正常的记录锁 Record Locks ：与锁定读一模一样

- 间隙锁 Gap Locks ：

  ![image-20230619123812004](C:\Users\windows\AppData\Roaming\Typora\typora-user-images\image-20230619123812004.png)

  如图中为 number 值为 8 的记录加了 gap锁 ，意味着不允许别的事务在 number 值为 8 的记录前边的 间隙 插入新记录，其实就是 number 列的值 (3, 8) 这个区间的新记录是不允许立即插入的。比方说有另外一个事 务再想插入一条 number 值为 4 的新记录，它定位到该条新记录的下一条记录的 number 值为8，而这条记录上又有一个 gap锁 ，所以就会阻塞插入操作，直到拥有这个 gap锁 的事务提交了之后， number 列的值在区 间 (3, 8) 中的新记录才可以被插入。

   gap锁 的提出**仅仅是为了防止插入幻影记录**而提出的，虽然有 共享gap锁 和 独占gap锁 这样的说法， 但是它们起到的作用都是相同的。而且如果你对一条记录加了 gap锁 （不论是 共享gap锁 还是 独占gap 锁 ），并不会限制其他事务对这条记录加 正经记录锁 或者继续加 gap锁。

  你可能会说，hero 表中 number 值为 20 的记录之后的间隙该咋办呢，在 数据页 中有两条伪记录（页面中最小和最大的记录），那么直接给20的所在页面的 最大 记录加上一个 gap锁就好了。

  

- Next-Key Locks 正常记录锁 和 gap锁的合体：

- 插入意向锁 Insert Intention Locks ：事实上插入意向锁并不会阻止别的事务继续获取该记录上任何类型的锁（ 就是这么鸡肋）。

- 隐式锁：

### 十二、视图

```sql
用法： CREATE VIEW 视图名 AS 
		SELECT 列名... FROM 表名(基表);
-- 创建一个视图 emp_view01，只能查询 emp 表的(empno、ename, job 和 deptno ) 信息
CREATE VIEW emp_view01 AS
	SELECT empno, ename, job, deptno FROM emp;
-- 1. 创建视图后，到数据库去看，对应视图只有一个视图结构文件(形式: 视图名.frm)
-- 2. 视图的数据变化会影响到基表，基表的数据变化也会影响到视图[insert update delete]
```

![image-20230321225739470](https://s2.loli.net/2023/03/21/oxRb7kHKgyni4aZ.png)

### 十三、用户管理

```sql
#创建用户
CREATE USER '用户名' @ '允许登录的IP地址' IDENTIFIED BY '密码'
-- 创建用户 shunping 密码 123 , 从本地登录
CREATE USER 'shunping'@ 'localhost' IDENTIFIED BY '123'

#删除用户
DROP USER '用户名'@ '登录地址'
DROP USER 'shunping'@ 'localhost'

#修改密码
-- 改自己的
set password = password('密码');
-- 改别人的
set password for '用户名'@ '登录地址' =  password('密码');

-- '用户名' @ '登录地址'也叫用户账户

#给用户授权
GRANT 权限列表..   		-- 比如增删改查等权限
	ON 库.对象名..   	-- 比如赋予对象访问student数据库的(表,视图等对象)的权限
	TO 用户账户 [WITH GRANT OPTION];  -- 加上最后的[WITH.. ]代表允许该用户将权限授予给其他用户
-- 给 shunping 分配查看 (SELECT) test库下的 news表 的权限 和 添加 (INSERT) 的权限
GRANT SELECT , INSERT
ON test.news
TO 'shunping'@'localhost';

#回收用户权限
REVOKE 权限列表 
	ON 库.对象
	FROM 用户账户
-- 回收 shunping 用户在 testdb.news 表的增改查权限
REVOKE SELECT , UPDATE, INSERT 
	ON testdb.news 
	FROM 'shunping'@'localhost' 
-- 回收 shunping 用户在 testdb.news 表的所有权限
REVOKE ALL 
	ON testdb.news 
	FROM 'shunping'@'localhost
```

#### 细节说明

![image-20230321234817384](https://s2.loli.net/2023/03/21/EAUTL6fGjBnFRKz.png)

![image-20230321234836847](https://s2.loli.net/2023/03/21/Uw4yb5zqaNe7DmK.png)

![image-20230321234904064](https://s2.loli.net/2023/03/21/tIjh5boMpXVzykx.png)

```sql
比如:
-- xxx 用户可以在任何ip登录 mysql
create user 'xxx';
-- xxx 用户在 192.168.1.*的 ip 可以登录 mysql
create user 'xxx'@ '192.168.1.%' ;


DROP USER jack 		-- 不填host默认就是 DROP USER 'jack'@ '%' 
```

