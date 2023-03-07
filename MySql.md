# MySql数据库

- 数据库学习最重要的两点就是**数据库表的设计**和**SQL语句的编写**，比如要你设计一个学生管理系统，可以想想自己要设计哪些表，每个表有哪些字段，字段类型是什么，需要大家多想多练！

## 知识点

- 基本概念
- 数据库创建
- SQL语句编写
- 约束
- 索引
- 事务
- 锁机制
- 数据库设计
- 性能优化

## SQL语言基础

- 在mysql学习中sql语言的编写是最重要的一环，这个阶段一定要好好学，多自己写。

### DDL

#### 对数据库的操作

- show databases    查看所有数据库
- create database    创建数据库
- use db1（数据库名）   切换数据库
- drop database   删除数据库

#### 对数据表的操作

- 建表：

  create table [if not exists] 表名（

  ​	字段名1 类型[(宽度)] [约束条件] [comment '字段说明']，

  ​	字段名2 类型[(宽度)] [约束条件] [comment '字段说明']，

  ​	字段名3 类型[(宽度)] [约束条件] [comment '字段说明']

  ）;

创建表就是构建一张空表，指定这个表的名字，有几列，每一列叫什么名字，以及每一列储存的类型

```sql
create DATABASE if not EXISTS mydb1;
	use mydb1;
	create table if not EXISTS srudent(
	name VARCHAR(20),
	age int,
	score double #这里别打,
);
```

- 数据类型

  - ```sql
    int	整数型
    
    bigint	长整型
    
    float	浮点型
    
    char	定长字符串
    
    varchar	可变长字符串
    
    date	日期类型
    
    BLOB	二进制大对象（储存图片、视频等流媒体信息）Binary Large OBject
    
    CLOB	字符大对象（存放大文本）Character Large OBject
    
    ......
    ```

- 其他操作

  ```
  -- 查看数据库所有表
  show tables;
  -- 查看指定表的创建语句
  show create table 表名;
  -- 查看表结构
  desc 表名;
  -- 删除表
  drop table 表名;
  ```

- 修改表结构

  - 添加列

    alter table 表名 add 列名 类型（长度） [约束];

  - 修改列名和类型

    alter table 表名 change 旧列名 新列名 类型（长度）[约束]；

  - 删除列

    alter table 表名 drop 列名;

  - 修改表名

    rename table 表名 to 新表名；

```sql
rename TABLE student to stu; student -> stu
alter table student add score double; 对student表新增一个score列
alter table student CHANGE score new_name double; 把score列的名字改成new_name
alter table student drop score; 删除student的score列
```

### DQL 重点！！！

数据库查询语言

- **简单查询：**

```sql
select 字段名1,字段名2... from 表名;
select 字段名 as 新字段名 from 表名;
```

 select (字段名1),(字段名2)... from (table_name) ; ps: 字段可以参与数学运算

 在字段号后加上 as (rename) 可以给查询结果的列重命名。命名可以为中文。

 标准sql语句要求字符串用单引号括起来，尽管mysql支持双引号。

 将字段名写作 "*" 代表查询该表所有数据。效率较低。

- **条件查询：**

```sql
select
    字段,字段...
from
    表名
where
    条件 ;
```

 条件语句包括：

 =, ＜, >, <=, >=, != ,<> (不等于),

 between ... and ... (左闭右开区间),

 is (not) NULL (NULL不等于0.0),

 or(或), and(并),

 (not) in(或): in (条件一,条件二),

 like (模糊查询) ：

 %表示任意多个字符，_表示任意一个字符

 like ‘%字符%’ ;

 like '_字符%' 表示查找第二位是字符的数据；

 要查询带有_或%的字符，写成`\_`或`\%`

- **排序：**

```sql
select
    字段,字段...
from
    表名
order by
    条件
;
```

 asc升序，desc降序，默认升序

 用逗号隔开排序条件，表示在前面字段无法排序，才启动后面的字段排序

 可以用数字代替字段，表示指定列

 条件和排序可以组合，先执行条件，再执行排序，语句顺序也同上

- **分组函数：**

 count ()计数

 sum ()求和

 max ()最大值

 min ()最小值

 avg ()平均数

 对某一组数据处理，又称多行处理函数：多行处理，一行输出。

 **分组函数自动忽略NULL**

 单行处理函数：ifnull(字段，替换值)如果数据为null则替换

 只要有NULL参与的计算，结果都为NULL

 分组函数不能直接用于where子句中

 count(*)表示统计表中的所有不为NULL的数据总数量

- **分组查询：**

 group by 和 having

 group by：按照某个字段或者某些字段进行分组。

 having：是对分组后的数据进行再次过滤。

```sql
select
    字段，字段...
    分组函数（）
from
    表
group by
    字段，字段;
```

 分组函数一般和group by组合使用，

 并且任何一个分组函数都在group by执行后才会执行。

 group by在where之后执行

 如果没有group by整张表自成一组。 == select from group by * ;

 没有被group by分组的字段，查询出的结果没有意义。

 因此，select后面只能够被分组的字段和分组函数。

```sql
select
    字段，字段...
from
    表
group by
    字段，字段...
having
    条件 ;
```

 where搞不定的情况再用having。

 因为先过滤再分组效率较高。

 where 可以和 having 一起用。

- **查询去重**

```sql
select
distinct
    字段，字段...
from
    表
```

 distinct 只能出现在所有字段的最前面。

 distinct 会对所有字段的重复去重

- **连接查询（跨表查询）**

1. 内连接：

   表之间是平等的，只有都能匹配上的数据才会查询出来

   等值连接（条件是等值关系）

   非等值连接（条件是非等值关系）

   自连接（在同一张表）

   ```sql
   SQL92
   select
       字段，字段...
   from
       表，表...
   where
       条件
   
   SQL99
   select
       字段，字段...
   from
       表
   (inner) join
       表
   on
       条件
   ```

2. 外连接：

   分为主表和副表，主要查询主表，顺带查询副表，主表有的数据，附表没有，则会用null代替(主表数据无条件查出来)

   左（外）连接：左边是主表

   右（外）连接：右边是主表

   ```sql
   左连接
   select
       字段，字段...
   from
       表(主表)
   left (outer) join
       表
   on
       条件 ;
   右连接
   select
       字段，字段...
   from
       表
   right (outer) join
       表(主表)
   on
       条件 ;
   ```

3. 全连接：

4. 注意：

   笛卡尔乘积现象：当两张表连接查询时，没有任何条件限制下，查询结果条数是两张表的数据条数的乘积

   避免了笛卡尔现象也不会减少查询次数，只会影响查询结果的显示

   在from子句中，在表名后输入一个字符串表示用该字符串代表 表，俗称取别名。

   ```sql
   ... from emp e ...
   ```

   给表取别名是个好习惯，通过表名（表的别名）和字段用' . '连接，来指定那张表中的字段，避免重名造成的困扰。

   查询多个表时，就多写几个join on

- **嵌套查询（子查询）**

  select语句中可嵌套的位置在：

  ```sql
  select
      (select)
  from
      (select)
  where
      (select)
  ```

- **union（可以将查询结果集相加）**

  ```sql
  select ...
  union
  select ...
  ```

  合并后的列名为第一句的字段名

  第一句的查询列数和第二句的必须一样

- **limit**

  limit是mysql独有的

  limit取结果集中的部分数据

  ```
  select
      字段，字段...
  from
      表名
  limit 
      startIndex
      length
  ```

  startIndex表示起始位置，从0开始，0表示第一条数据。

  length表示取几个

  startIndex省略时，默认从0开始往后取

  查询分页：

  ```java
  int pageNo = n;
  int pageSize = 10;
  limit (pageNo - 1) * pageSize , pageSize ;
  ```

- **执行顺序**

```sql
select		5

from		1

where		2

group by	3

having		4

order by	6

limit		7
```

### DML

对表数据的增删改操作

- **insert插入数据**

```sql
insert into 表名(字段1,字段2,字段3,...) values(值1,值2,值3,...) ;
insert into 表名 select语句 ;//将查询结果插入到表中
```

- **delete删除数据**

  ```sql
  delete from 表 where 条件 ;
  ```

  不加where条件表示删除所有。

- **update修改数据**

```sql
update 表名 set 字段1=值1 , 字段2=值2 , ... where 条件 ;
```

 **不加where条件表示修改所有。**

## 约束

在创建表时可以对表的字段添加相应的约束，添加约束的目的在于保证表中数据的合法性、有效性、完整性。

### 常见约束

- 非空约束（not null）：约束字段不能为NULL 唯一约束（unique）：约束字段不能重复，但可以为NULL

  - 方式一：字段名 数据类型 not null；

  ```sql
  create table if not exists emp3(
   id int not null,
   salary int not null
  );
  ```

  - 方式二：alter table 表名 modify 字段 类型 not null

  ```sql
  create table if not exists emp3(
   id int,
   salary int
  );
  alter table emp3 modify id int not null;
  alter table emp3 modify salary int not null;
  ```

- 主键约束（primary key）：约束字段既不能为NULL也不能重复（简称PK）

  - 作用：主键值是这行记录在这张表中的唯一标识

  -  分类：单一主键 和 复合主键（多个字段联合起来添加一个主键）

    - 单一主键

      法一：直接在建表的时候写上

      ```sql
      create table if not exists test(
       name varchar(20) primary key,
       gender varchar(2),
       salary int
       );
      ```

      法二：定义完后再指定主键

      ```sql
      create table if not exists test(
       name varchar(20),
       gender varchar(2),
       salary int,
       constraint pk primary key (name) #consteaint pk 可以省略
       );
      ```

    - 复合主键

      只能在定义后再指定主键

      复合主键的每一列都不能为空

      ```sql
      create table if not exists emp1(
       id varchar(20),
       name varchar(20),
       salary int,
       dopid int,
       primary key (id,dopid)
      );
      ```

  - 一张表只能有一个主键

- 自增长约束

  - 用来实现主键的自增长，实现主键的自增长

  ```sql
  create table if not exists emp2(
   id int primary key auto_increment,#默认初始值是1，最多到类型上限
   name varchar(20),
   salary int,
   dopid int
  );
  ```

  - 创建表的时候指定初始值

    ```sql
    create table if not exists emp2(
     id int primary key auto_increment,#默认初始值是1，最多到类型上限
     name varchar(20),
     salary int,
     dopid int
    )auto_increment = 100;
    ```

  - 创建表之后指定

    alter table 名 auto_increment = 100;

- 唯一性约束

  ```sql
  create table if not exists emp3(
   id int unique,
   salary int
  );
  create table if not exists emp3(
   id int,
   salary int
  );
  alter table emp3 add constraint unique_id unique(id);
  ```

- 外键约束（foreign key）：添加外键约束的值必须来自于某个字段（简称FK）

  ```sql
  foreign key （字段）references 表（字段）;
  ```

## 视图

- 概念：一张虚拟表，数据在原表

- 作用：简化代码，数据安全

- 特点：

  - 本身不存数据

  - 表中数据改变，视图中也改变

- 创建/删除视图

```sql
create view name as select语句;
drop view name;
```

- 对视图进行增删改查，会影响到原表数据。（通过视图影响原表数据，不是直接操作的原表）
- 可以对视图进行CRUD操作
- 面向视图操作

```sql
select 字段名 from name;
update name set 字段1=值1 , 字段2=值2 , ... where 条件;
delete from name where 条件;
```

## 索引

- 最主要的目的就是快速查找！！

- 建立索引可以缩小扫描范围，从而实现高效查询。

- 索引不被推荐在需要频繁修改的数据表中，一旦数据改动，索引就需要重新排序、进行维护。

- 添加索引是给某个字段或者某些字段添加

- 什么时候考虑给字段添加索引？

  - 数据量庞大（根据需求和环境）
  - 该字段很少的DML操作
  - 该字段经常出现在where子句中

- 主键和具有unique约束的字段会自动添加索引

   根据主键查询效率高。

- 添加/删除索引：

```sql
create index name on 表名(字段名);
drop index name;
```

- 底层索引

  - 索引底层使用数据结构：B + Tree
  - 生成的索引存在硬盘文件或内存中（根据储存引擎），并且会携带每个数据的物理地址
  - 索引会自动排序，之后对数据（物理地址）分区，存进B+Tree中
  - 对有索引的数据列查询，等于直接查询物理地址，通过地址定位表中数据。

- 索引分类：

  - 单一索引：给单一字段添加索引
  - 复合索引：给多个字段联合起来添加一个索引
  - 主键索引：主键上自动添加索引
  - 唯一索引：有unique约束的字段会自动添加索引
  - ...

- 索引什么时候失效？

  模糊查询时，第一个通配符使用的是%，这时候索引失效。

  ...

## 学习资源推荐

- 视频资源
  -  老杜 - mysql入门基础 + 数据库实战：https://www.bilibili.com/video/BV1Vy4y1z7EX 
  - 尚硅谷 - MySQL基础教程：https://www.bilibili.com/video/BV1xW411u7ax 

- 文档资源
  - SQL - 菜鸟教程：https://www.runoob.com/sql/sql-tutorial.html
  - MySQL - 菜鸟教程：https://www.runoob.com/mysql/mysql-tutorial.html

# JDBC（Java Database Connectivity）

- 学习JDBC最主要的目的是为了能使用java语言来操作数据库
- 我们主要学习的是用JDBC来操作MySql数据库，对于其他数据库的操作有兴趣大家可以自行探索

## JDBC六步走

- 注册驱动（告诉Java程序，要连接那个品牌的数据库）

  ```java
  //注册驱动
  DriverManager.registerDriver(new com.mysql.cj.jdbc.Driver());
  ```

- 获取连接（表示jvm的进程和数据库进程之间的通道打开了，这属于进程间的通信，重量级的，使用完一定要关闭）

  ```java
  //获取连接
  String url = "jdbc:mysql://127.0.0.1:3306/数据库名;
  String user = "username";
  String password = "password";
  conn = DriverManager.getConnection(url, user, password);
  ```

- 获取数据库操作对象（专门执行sql语句的对象）

  ```java
  //获取数据库操作对象 Statement
  stat = conn.createStatement();
  ```

- 执行sql指令（DQL，DML…）

  ```java
  //执行sql
  String sql = "insert into stu values('xx',xx,xx)";
  int cnt = stat.executeUpdate(sql);//专门执行DML中的（insert，update，delete），返回影响记录的数量
  
  //执行
  String sql = "select * from t_user where username = '"+username+"' && password = '"+password+"'";
  ```

- 处理服务器返回的结果(当第四步是select语句时，才有第五步)

  ```java
  //处理，使用ResultSet类型来接受sql语句执行的返回类型
  //并对其进行操作
  rs = stmt.executeQuery(sql);
  ```

- 释放资源（使用完资源之后一定要关闭资源，java和数据库属于进程间的通信，开启之后一定要关闭）

  ```java
  //释放资源
  //为了保证资源一定能够释放，在finally语句中执行关闭
  //并且要遵循从小到大依次关闭
  //分别对其try catch
  try{
      if(statement != null){
          statement.close();
      }
  }catch (SQLException e) {
      e.printStackTrace();
  }
  try{
      if(connection != null){
          connection.close();
      }
  }catch (SQLException e) {
      e.printStackTrace();
  }
  ```

## Sql注入

- 即输入: fdsa fdsa' or '1' = '1' 这种明显错误的密码也会登录成功 原因在于用户输入的信息也参与了sql语句的编译

  - 例如：

    使用

    ```sql
    String sql = "select * from t_user where username = '"+username+"' && password = '"+password+"'";
    ```

    语句查询数据库中是否存在这样一条目录，可以在后面加上一段 ‘or 1=1’

    最终拼接成的sql语句就是

    ```
    select * from t_user where username = username && password = password or 1 = 1 
    ```

    将永远有数据，就会存在安全隐患

### 解决Sql注入问题

- 只要用户提供的信息中不参与sql语句的编译

- 要达到以上目的，必须使用java.sql.PreparedStatement

- PreparedStatement接口继承了Statement

- PreparedStatement属于预编译的数据库操作对象

- PreparedStatement原理是预先对sql语句的框架编译，在填入用户信息

### Statement与PreparedStatement的比较

- Statement存在sql注入问题，PreparedStatement解决了sql注入问题
- 前者时编译一次执行一次，后者是编译一次执行n次
- 后者会在编译阶段做类型的安全检查

综上所述，后者使用更多，前者使用少

只有在业务要求支持SQL语句注入或者SQL语句拼接时，才会使用前者

# Spring

