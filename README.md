# 后端学习路线

## 阶段一：算法部分

这一部分主要是大一同学还没分路线的时候进行学习的部分。

比较重要，无论是打比赛、读研还是找工作等等都是很重要的。

这里我们团队的路线一直都是 `acwing` 的yxc的课。

看到 Acwing 的算法基础课就可以了：

> 算法基础课 https://www.acwing.com/activity/content/11/

大一同学结束`蓝桥`和`ACM`新生赛就可以开始阶段二了。

具体填充内容交给 算法 精通的选手填充吧。

## 阶段二： 基础知识与工具

**好记性不如烂笔头。做笔记没有规范，让自己能快速看懂就好，又不是写教程。**

### 1. Markdown

Markdown 主要就是用来记笔记， 写法简单有很好的可读性。（甚至写的时候 txt 都可以写）

#### 2. 知识管理工具

- Typora ： 简约
- Obsidian : 个人比较推荐， 比Typora好看，功能更强大。
- 同步工具: Github Gitee

#### Typora

**记笔记神器**，语法比较简单，界面展示也比较清晰，大力推荐

因为语法简单而且只需要掌握比较常使用的语法，所以上手很快，建议快速过一遍语法就直接投入使用

```
基本语法：
	标题： # 一级标题
		  ## 二级标题
		  ### 三级标题
		  ...
	加粗：**要加粗的字**
	插入代码块：```
	列表：- （-后面加一个空格）
```

其他的大家自己上网站摸索，多用是关键

网站： [Markdown 语法速查表 | Markdown 官方教程](https://markdown.com.cn/cheat-sheet.html#总览)

### IDEA

IDEA是java开发利器，可以让我们代码敲得更加快速，更加方便。分为免费的社区版和付费的专业版，选择哪一种就看大家喜欢了，专业版功能要多些，需要的可以上网找找教程，这种都迭代的比较快，但是愿意找总能找到

需要熟悉的基本操作有，新建项目、运行项目、打断点调试、配置插件等等，其他的操作都会在后续的学习中用到

常用快捷键：[IntelliJ Idea 常用快捷键列表 - 简书 (jianshu.com)](https://www.jianshu.com/p/5de7cca0fefc)







### 阶段三： 小探后端

### 1. 代码规范

### 2. Java 语言

- 推荐课程：

1. 青空 的 Java SE: [青空JavaSE] 
2. 尚硅谷的Java SE: [尚硅谷JavaSE] （圣剑学长推荐）

[青空JavaSE]:https://www.bilibili.com/video/BV1YP4y1o75f/?spm_id_from=333.999.0.0&vd_source=ccfdcae1ff9c9416058528dd2026d0ed
[尚硅谷JavaSE]:https://www.bilibili.com/video/BV1PY411e7J6/?spm_id_from=333.337.search-card.all.click&vd_source=ccfdcae1ff9c9416058528dd2026d0ed


- 目标：
	- 熟练掌握 Java 常见集合
	- 掌握 Java 各种特性，如String为什么只能用`.equals()` 方法判断
	- 了解反射机制、注解

- 重点
	 - Java 基础语法
	 - 面向对象的思想
	 - Java 类的基本结构
	 - Java 泛型 (**Important**)
	 - Java 数据结构实现（**Very Important**）
	 - Java Collection类 **(Very Important**, 青空会带着看源码, 好好看看)
	 - Java Stream **(Important)**  , 能减少代码量提高可读性。
	 - 反射和注解  **(Important)**
	 - IO操作

- 作业
用 Java 做一些 Leetcode  简单题练练熟练度


### 3. MySQL 基本使用

**Tips:** 
- 目前处于小探后端的路径，所以基本原理对于你们来说还太早了，应该做了项目考虑怎么优化的时候去看看怎么做。
- 因此现在的主要任务是会用


*  目标：
	* 熟练掌握数据库的 DDL、DCL、DML、DQL
		* 要做到给条件能手写出相应的SQL语句
	* 了解 MySQL 的数据类型，以及对应的Java类型
	* 掌握数据库的视图、触发器以及**事务**

* 课程推荐：
1. 青空的JavaWeb课程: [JavaWeb青空]
2. 待补充

[JavaWeb青空]:bilibili.com/video/BV1CL4y1i7qR/?spm_id_from=333.999.0.0&vd_source=ccfdcae1ff9c9416058528dd2026d0ed








### 4. JDBC



### java Web基础



## 阶段四：工具篇

### Maven



### git



### linux



### 消息队列



### Docker



### Redis

Redis是一种键值型的NoSql数据库，而NoSql则是相对于传统关系型数据库而言，有很大差异的一种数据库。

对于存储的数据，没有类似Mysql那么严格的约束，比如唯一性，是否可以为null等等，所以我们把这种松散结构的数据库，称之为NoSQL数据库。

**学习目标**

+ 学会使用redis常见的指令
+ 了解使用java操作redis(Jedis，其实不怎么直接使用)
+ 熟悉spring封装的工具类RedisTemplate，以后都用这个操作redis
+ 熟悉redis的key数据结构(这个不急，可以放到后面再学)
+ 了解一下redis集群和分布式管理

redis其实不像mysql那样储存大量的持久化数据，redis更多的应用还是用来作缓存作用。

这里强烈推荐[黑马的redis](https://www.bilibili.com/video/BV1cr4y1671t)一套通关



### nginx



## 阶段五：框架篇



### Spring



### SpringMVC



### MyBatis/MyBatis-Plus

​	mybatis是一种持久层框架，用于简化JDBC的开发，其实底层还是JDBC，只不过对JDBC进行了封装，使用者查询数据库操作更为简单，不用手动管理数据库连接对象等琐碎的准备工作。

​	**学习目标：**

+ 理解并学会使用Mybatis配置文件，了解配置文件中各个标签及其属性的含义，能灵活修改配置文件
+ 学会编写mybatis的sql映射文件，统一管理sql语句
+ 在Java代码中使用SqlSession来调用映射文件中的sql语句来得到数据库里的数据
+ 学会编写动态SQL
+ 学会使用注解编写sql语句



由于mybatis的配置文件和映射文件编写起来相对复杂，mybatisplus为我们封装了很多sql语句，不需要我们写相关的配置文件和sql，直接调用接口方法就能直接查询数据库。其功能与mybatis一样，但使用起来非常简便优雅

**学习目标：**

对于mybatisplus的学习目标就是要熟练运用相关注解和方法

+ mybatisplus为我们提供最主要的两个类就是BaseMapper和Iservice
+ BaseMapper是属于dao层的一个接口，里面封装了很多查询数据库的方法。
+ Iservice是属于Service层的一个接口，里面封装了很多BaseMapper的方法，j基本BaseMapper能实现的Iservice都能实现，所以一定要多去学习和运用IService中的方法(太多了，用到的时候现查都行，用多了就熟练了)。
+ 另外在使用mybatisplus进行数据查询的时候，多半会带有一些条件的查询，Wrapper类就是就是用来设置条件的，里面也封装了很多条件查询相关的方法，也要学一下
+ 实体类的有关注解



学完以上两个框架，以后在项目中就不要使用JDBC了，直接用框架会使coding更简单。但是底层的JDBC也需要理解。看个人喜好选择使用mybatisplus还是mybatis，个人一直在用mybatisplus，用过都说爽，不用写乱七八糟的配置文件，拿着API直接用，sql也基本不怎么写，要写的时候也是直接作为参数传进方法里，十分的简单优雅。



资料：[mybatis官方文档](https://mybatis.org/mybatis-3/zh/index.html)

[MyBatis-Plus 官方文档](https://baomidou.com/pages/24112f/)

[尚硅谷mybatis](https://www.bilibili.com/video/BV1VP4y1c7j7)

[三更草堂mybatisplus](https://www.bilibili.com/video/BV1Bq4y1f7YD)，mybatis主要还是多看文档学会API

### SpringBoot

Spring Boot是Spring 家族中的一个全新的框架，它用来简化Spring 应用程序的创建和开发过程，也可以说Spring Boot 能简化我们之前采用SpringMVC + Spring + MyBatis 框架进行开发的过程。

采用Spring Boot可以非常容易和快速地创建基于Spring 框架的应用程序，它让编码变简单了，配置变简单了，部署变简单了，监控变简单了。正因为Spring Boot 它化繁为简，让开发变得极其简单和快速，所以在业界备受关注。

### **Spring Boot 的特性**

1）能够直接使用java main方法启动内嵌的 Tomcat 服务器运行 Spring Boot 程序，不需部署war包文件

2）提供约定的 starter POM 来简化 Maven 配置，让 Maven 的配置变得简单

3）自动化配置，根据项目的 Maven 依赖配置，Spring boot 自动配置 Spring、Spring mvc 等

4）提供了程序的健康检查等功能

5）基本可以完全不使用 XML 配置文件，采用注解配置

**学习目标**

+ 理解起步依赖的概念以及结构
+ spring boot的配置
+ 学会各种技术框架与springboot的整合，例如日志技术，缓存控制等等



资料

[springboot官方文档]([Spring | Quickstart](https://spring.io/quickstart)) (还是看视频好点)

[黑马springboot](https://www.bilibili.com/video/BV1Lq4y1J77x)





### Spring Security



### Springcloud



## 项目实战















