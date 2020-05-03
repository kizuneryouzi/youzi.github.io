# MyBatis

###  传统的JDBC写法

##### 回顾默写

1.class.forname-2.drivemanager.getconnection

3.定义sql语句4.preparestatement-ps .queryList/one

4.遍历resultset 5关闭连接

````java
//1.注册驱动，驱动是能够让java连接数据库也就是实现jdbc接口规范
Class.forName("ocm.mysql.jdbc.Driver");
//2.通过驱动管理器获取数据库连接DriveManager是驱动实现类的管理者
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306~","","");//硬编码
//3.获得sql语句执行者preparedStatement预处理（常用）sql语句可以不是完整的用？代替，然后在预编译后传入未知参数--定义sql语句？表示占位符
String sql = "select * from user where id= ?";
PrepareStatement ps = conn.prepareStatement(sql);
ps.setInt(1,1);
//4.获取sql语句执行结果resuleset ，executeQuery（）查询操作，executeUpdate（）增删改操作
ResultSet rs =ps.executeQuery();
//5.处理结果 用while(rs.next()){}
while(rs.next()){
    System.out.println(rs.getInt(1));//Index表示第几列 从1开始
}
//6.关闭连接
rs.close();
ps.close();
conn.close();
````

#### 学习思路

mybatis入门--全局配置文件和映射文件--高级映射1对多多对多--延迟加载机制

一级缓存二级缓存--spring整合--逆向工程

#### 什么是MyBatis

MyBatis是一款优秀的**持久层框架**

MyBatis几乎避免了所有的JDBC代码和手动设置参数以及获取结果集的过程

MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 实体类 【Plain Old Java Objects,普通的 Java对象】映射成数据库中的记录。

##### 持久化

持久化是将程序数据在持久状态和瞬时状态间转换的机制

将鲜肉冷藏，吃的时候再解冻的方法也是。将水果做成罐头的方法也是

为什么需要持久化服务：内存断电后数据会丢失 内存过于昂贵

##### 持久层

完成持久化工作的代码块 --dao层 data access Object 数据访问对象

大多数情况下特别是企业级应用，数据持久化往往也就意味着将内存中的数据保存到磁盘上加以固化，而持久化的实现过程 大多是通过关系数据库来完成

用来操作数据库而存在的

为什么需要MyBatis？

MyBatis就是帮助程序员将数据存入数据库中 和从数据库中取出数据 

MyBatis 是一个半自动化的**ORM框架 (Object Relationship Mapping) -->对象关系映射**

### MyBatis的第一个程序

**思路流程：搭建环境-导入MyBatis-编写代码-测试**

1.搭建数据库

2.导入jar包 

3.编写MyBatis的全局配置文件 sqlMapConfig.xml

 sqlMapConfig.xml内容

1. 配置jdbc事务控制
2. 配置数据源 采用jdbc连接池
3. 加载mapper映射文件 在mappers标签的mapper标签中配置

````xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
       PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
   <environments default="development">
       <environment id="development">
           <transactionManager type="JDBC"/>
           <dataSource type="POOLED">
               <property name="driver" value="com.mysql.jdbc.Driver"/>
               <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
               <property name="username" value="root"/>
               <property name="password" value="123456"/>
           </dataSource>
       </environment>
   </environments>
   <mappers>
       <mapper resource="com/kuang/dao/userMapper.xml"/>
   </mappers>
</configuration>
````

4.编写MyBatis工具类

````java
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import java.io.IOException;
import java.io.InputStream;

public class MybatisUtils {

   private static SqlSessionFactory sqlSessionFactory;

   static {
       try {
           String resource = "mybatis-config.xml";
           InputStream inputStream = Resources.getResourceAsStream(resource);
           sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
      } catch (IOException e) {
           e.printStackTrace();
      }
  }

   //获取SqlSession连接
   public static SqlSession getSession(){
       return sqlSessionFactory.openSession();
  }

}
````

5.创建实体类

````java
public class User {
   
   private int id;  //id
   private String name;   //姓名
   private String pwd;   //密码
   
   //构造,有参,无参
   //set/get
   //toString()
   
}
````

6.编写Mapper接口

````java
import com.kuang.pojo.User;
import java.util.List;

public interface UserMapper {
   List<User> selectUser();
}
````

7、**编写Mapper.xml配置文件**

- namespace 十分重要，不能写错！

````xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
       PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kuang.dao.UserMapper">
 <select id="selectUser" resultType="com.kuang.pojo.User">
  select * from user
 </select>
</mapper>
````

外层是mapper标签 有命名空间namespace 

8.编写测试类

Junit 包测试

````java
public class MyTest {
   @Test
   public void selectUser() {
       SqlSession session = MybatisUtils.getSession();
       //方法一:
       //List<User> users = session.selectList("com.kuang.mapper.UserMapper.selectUser");
       //方法二:
       UserMapper mapper = session.getMapper(UserMapper.class);
       List<User> users = mapper.selectUser();

       for (User user: users){
           System.out.println(user);
      }
       session.close();
  }
}
````

9、运行测试，成功的查询出来的我们的数据，ok！

**可能出现问题说明：Maven静态资源过滤问题**

````xml
<resources>
   <resource>
       <directory>src/main/java</directory>
       <includes>
           <include>**/*.properties</include>
           <include>**/*.xml</include>
       </includes>
       <filtering>false</filtering>
   </resource>
   <resource>
       <directory>src/main/resources</directory>
       <includes>
           <include>**/*.properties</include>
           <include>**/*.xml</include>
       </includes>
       <filtering>false</filtering>
   </resource>
</resources>
````

配置文件中namespace中的名称为对应Mapper接口或者Dao接口的完整包名,必须一致！

#### Select标签

- SQL语句返回值类型。【完整的类名或者别名】
- 传入SQL语句的参数类型 。【万能的Map，可以多尝试使用】
- 命名空间中唯一的标识符
- 接口中的方法名与映射文件中的SQL语句ID 一一对应
- id
- parameterType
- resultType

#### 需求：根据id查询用户

1.UserMapper中添加对应方法

````java
public interface UserMapper {
   //查询全部用户
   List<User> selectUser();
   //根据id查询用户
   User selectUserById(int id);
}
````

2.在UserMapper.xml中添加Select语句

````xml
<select id="selectUserById" resultType="com.kuang.pojo.User">
select * from user where id = #{id}
</select>
````



3.测试类中测试

````java
@Test
public void tsetSelectUserById() {
   SqlSession session = MybatisUtils.getSession();  //获取SqlSession连接
   UserMapper mapper = session.getMapper(UserMapper.class);
   User user = mapper.selectUserById(1);
   System.out.println(user);
   session.close();
}
````

测试：

1.通过工具类获取session

2.通过session获取mapper（传入mapper类）

3.mapper对象执行查询mapper.selectUserById(1);

4.关闭session-session.close();

#### 需求2：根据密码和名字查询用户

##### 思路1：直接在方法中传递参数

1.在接口方法的参数前面加@Param属性

2.sql语句编写的时候，直接取@param中设置的值即可，不需要单独设置参数类型

````java
//通过密码和名字查询用户
User selectUserByNP(@Param("username") String username,@Param("pwd") String pwd);

/*
   <select id="selectUserByNP" resultType="com.kuang.pojo.User">
     select * from user where name = #{username} and pwd = #{pwd}
   </select>
*/
````

##### 思路2 ：使用万能的Map

1.接口方法中，参数直接传递Map；

````java
User selectUserByNP2(Map<String,Object> map);
````



2.编写sql语句的时候，需要传递参数类型，参数类型为map

````xml
<select id="selectUserByNP2" parameterType="map" resultType="com.kuang.pojo.User">
select * from user where name = #{username} and pwd = #{pwd}
</select>
````



3.在使用方法的时候，Map的Key为sql中取得值

````java
Map<String, Object> map = new HashMap<String, Object>();
map.put("username","小明");
map.put("pwd","123456");
User user = mapper.selectUserByNP2(map);
````

### Insert标签

#### 需求：给数据库增加一个用户

1.在UserMaspper接口总添加对应的方法

````java
//添加一个用户，返回int
int addUser(User user);
````



2.在UserMapper.xml 中添加insert语句

````xml
<insert id="addUser" parameterType="com.kuang.pojo.User">
    insert into user (id,name,pwd) values (#{id},#{name},#{pwd})
</insert>
````



3、测试--   session.commit(); //提交事务,重点!不写的话不会提交到数据库

**注意点：增、删、改操作需要提交事务！**

1.通过工具类获取session

2.通过session获取mapper 传入mapper类

3.通过mapper执行insert方法

4.提交事务 session.commit(); session.close();

````java
@Test
public void testAddUser() {
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   User user = new User(5,"王五","zxcvbn");
   int i = mapper.addUser(user);
   System.out.println(i);
   session.commit(); //提交事务,重点!不写的话不会提交到数据库
   session.close();
}
````

### update

````java
//修改一个用户
int updateUser(User user);
````

### delete

````java
//根据id删除用户
int deleteUser(int id);
````

**小结：**

- **所有的增删改操作都需要提交事务！**
- 接口所有的普通参数，尽量都写上@Param参数，尤其是多个参数时，必须写上！
- 有时候根据业务的需求，可以考虑使用map传递参数！
- 为了规范操作，在SQL的配置文件中，我们尽量将Parameter参数和resultType都写上！

### **模糊查询like语句该怎么写?**

1.在java代码中添加sql通配符

2.在sql语句中拼接通配符 ，会引起sql注入

## MyBatis配置解析

### 核心配置文件

- mybatis-config.xml 系统核心配置文件
- MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。
- 能配置的内容如下：

````jsp
configuration（配置）
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）--浦路跟
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）-船在可孙 
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）--麻婆
<!-- 注意元素节点的顺序！顺序不对会报错 -->
````

#### mappers元素

- 映射器 : 定义映射SQL语句文件
- 既然 MyBatis 的行为其他元素已经配置完了，我们现在就要定义 SQL 映射语句了。但是首先我们需要告诉 MyBatis 到哪里去找到这些语句。

````xml
<!--
将包内的映射器接口实现全部注册为映射器
但是需要配置文件名称和接口名称一致，并且位于同一目录下
-->
<mappers>
 <package name="org.mybatis.builder"/>
</mappers>

<!--
使用映射器接口实现类的完全限定类名
需要配置文件名称和接口名称一致，并且位于同一目录下
-->
<mappers>
 <mapper class="org.mybatis.builder.AuthorMapper"/>
</mappers>
````

#### **Mapper文件**

````xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
       PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kuang.mapper.UserMapper">
   
</mapper>
````

- namespace中文意思：命名空间，作用如下：

- - namespace的命名必须跟某个接口同名
  - 接口中的方法与映射文件中sql语句id应该一一对应

- 1. namespace和子元素的id联合保证唯一  , 区别不同的mapper
  2. 绑定DAO接口
  3. namespace命名规则 : 包名+类名

### Properties优化 -噗绕噗踢死

第一步 ; 在资源目录下新建一个db.properties

````properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=utf8
username=root
password=123456
````

character 凯瑞科特 

Encoding 音口定

第二步 : 将文件导入properties 配置文件

````xml
<configuration>
   <!--导入properties文件-->
   <properties resource="db.properties"/>

   <environments default="development">
       <environment id="development">
           <transactionManager type="JDBC"/>
           <dataSource type="POOLED">
               <property name="driver" value="${driver}"/>
               <property name="url" value="${url}"/>
               <property name="username" value="${username}"/>
               <property name="password" value="${password}"/>
           </dataSource>
       </environment>
   </environments>
   <mappers>
       <mapper resource="mapper/UserMapper.xml"/>
   </mappers>
</configuration>
````

```xml
<!--导入properties文件-->
   <properties resource="db.properties"/>
```

### 生命周期和作用域（Scope）-是狗破

#### Mybatis的执行过程

![img](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JdnS939HH5TayIhQo5s0aJbReBExSQO1U23XeLAXlhTWUeL87mJZL0lDzPstpY3CSIwvW0dN9ccA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

SqlSessionFactoryBuilder 的作用在于创建 SqlSessionFactory，创建成功后，**SqlSessionFactoryBuilder 就失去了作用**

**SqlSessionFactory 可以被认为是一个数据库连接池，**它的作用是创建 SqlSession 接口对象。因为 MyBatis 的本质就是 Java 对数据库的操作，所以 SqlSessionFactory 的生命周期存在于整个 MyBatis 的应用之中，所以一旦创建了 SqlSessionFactory，就要长期保存它，直至不再使用 MyBatis 应用

我们往往**希望 SqlSessionFactory 作为一个单例**，让它在应用中被共享。所以说 **SqlSessionFactory 的最佳作用域是应用作用域。**

如果说 SqlSessionFactory 相当于数据库连接池，那么 **SqlSession 就相当于一个数据库连接（Connection 对象）**，你可以在一个事务里面执行多条 SQL，然后通过它的 commit、rollback 等方法，提交或者回滚事务。所以它应该存活在一个业务请求中，处理完整个请求后，应该关闭这条连接，让它归还给 SqlSessionFactory，

**所以 SqlSession 的最佳的作用域是请求或方法作用域。**



常识：Constructor 看死cua科特-构造器

Repository -锐跑着崔

Pagination page内损

Interceptor 因特赛破特  - 阻止的人

Generator -捐呢瑞特 -生产者 发生器

Entity 安特提 -实体

daemon 地门 -守护进程

Binary -白呢瑞 -二进制

Device -地外事 -设备

Enumeration  一扭末锐孙 --枚举

collection 克莱克顺 --收藏物 集合

解决属性名和数据库字段名不一致的问题--使用结果集映射 **ResultMap**

![image-20200502190743184](image-20200502190743184.png)

### ResultMap

##### 自动映射

- `resultMap` 元素是 MyBatis 中最重要最强大的元素。它可以让你从 90% 的 JDBC `ResultSets` 数据提取代码中解放出来。
- 实际上，在为一些比如连接的复杂语句编写映射代码的时候，一份 `resultMap` 能够代替实现同等功能的长达数千行的代码。
- ResultMap 的设计思想是，对于简单的语句根本不需要配置显式的结果映射，而对于复杂一点的语句只需要描述它们的关系就行了。

你已经见过简单映射语句的示例了，但并没有显式指定 `resultMap`。比如：

````xml
<select id="selectUserById" resultType="map">
select id , name , pwd
  from user
  where id = #{id}
</select>
````

##### 手动映射

1、返回值类型为resultMap

````xml
<select id="selectUserById" resultMap="UserMap">
  select id , name , pwd from user where id = #{id}
</select>
````



2、编写resultMap，实现手动映射！

````xml
<resultMap id="UserMap" type="User">
   <!-- id为主键 -->
   <id column="id" property="id"/>
   <!-- column是数据库表的列名 , property是对应实体类的属性名 -->
   <result column="name" property="name"/>
   <result column="pwd" property="password"/>
</resultMap>
````

column 考了木

property 蒲饶破题

association 额手洗诶孙

### Log4j


Log4j是Apache的一个开源项目

通过使用Log4j，我们可以控制日志信息输送的目的地：控制台，文本，GUI组件....

我们也可以控制每一条日志的输出格式；

通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。最令人感兴趣的就是，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。

**使用步骤：**

1、导入log4j的包

2、配置文件编写

3、setting设置日志实现

````xml
<settings>
   <setting name="logImpl" value="LOG4J"/>
</settings>
````

4、在程序中使用Log4j进行输出！

关键语句：Logger logger = Logger.getLogger(MyTest.class);

````java
//注意导包：org.apache.log4j.Logger
static Logger logger = Logger.getLogger(MyTest.class);

@Test
public void selectUser() {
   logger.info("info：进入selectUser方法");
   logger.debug("debug：进入selectUser方法");
   logger.error("error: 进入selectUser方法");
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   List<User> users = mapper.selectUser();
   for (User user: users){
       System.out.println(user);
  }
   session.close();
}
````

测试步骤：

1.通过mybatisutil获取session

2.通过session获取mapper

3.通过mapper操作增删改查方法

4.遍历结果 ，关闭连接

### limit实现分页

为什么需要分页？

在学习mybatis等持久层框架的时候，会经常对数据进行增删改查操作，使用最多的是对数据库进行查询操作，如果查询大量数据的时候，我们往往使用分页进行查询，也就是每次处理小部分数据，这样对数据库压力就在可控范围内。

**使用Limit实现分页**

语法： SELECT * FROM  TABLE LIMIT startIndex,pageSize;

````sql
#语法
SELECT * FROM table LIMIT stratIndex，pageSize

SELECT * FROM table LIMIT 5,10; // 检索记录行 6-15  

#为了检索从某一个偏移量到记录集的结束所有的记录行，可以指定第二个参数为 -1：   
SELECT * FROM table LIMIT 95,-1; // 检索记录行 96-last.  

#如果只给定一个参数，它表示返回最大的记录行数目：   
SELECT * FROM table LIMIT 5; //检索前 5 个记录行  

#换句话说，LIMIT n 等价于 LIMIT 0,n。 
````

**步骤：**

1、修改Mapper文件

````xml
<select id="selectUser" parameterType="map" resultType="user">
  select * from user limit #{startIndex},#{pageSize}
</select>
````



2、Mapper接口，参数为map

````java
//选择全部用户实现分页
List<User> selectUser(Map<String,Integer> map);
````



3、在测试类中传入参数测试

- **推断：起始位置 =  （当前页面 - 1 ） * 页面大小**

```java
//分页查询 , 两个参数startIndex , pageSize
@Test
public void testSelectUser() {
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);

   int currentPage = 1;  //第几页
   int pageSize = 2;  //每页显示几个
   Map<String,Integer> map = new HashMap<String,Integer>();
   map.put("startIndex",(currentPage-1)*pageSize);
   map.put("pageSize",pageSize);

   List<User> users = mapper.selectUser(map);

   for (User user: users){
       System.out.println(user);
  }

   session.close();
}
```

#### RowBounds分页

我们除了使用Limit在SQL层面实现分页，也可以使用**RowBounds在Java代码层面实现分页**，当然此种方式作为了解即可。我们来看下如何实现的！

**步骤：**

1、mapper接口

2、mapper文件

3、测试类

在这里，我们需要使用RowBounds类

````java
@Test
public void testUserByRowBounds() {
   SqlSession session = MybatisUtils.getSession();

   int currentPage = 2;  //第几页
   int pageSize = 2;  //每页显示几个
   RowBounds rowBounds = new RowBounds((currentPage-1)*pageSize,pageSize);

   //通过session.**方法进行传递rowBounds，[此种方式现在已经不推荐使用了]
   List<User> users = session.selectList("com.kuang.mapper.UserMapper.getUserByRowBounds", null, rowBounds);

   for (User user: users){
       System.out.println(user);
  }
   session.close();
}
````

### PageHelper

略

### 面向接口编程

真正的开发中，很多时候我们会选择面向接口编程

**根本原因 :  解耦 , 可拓展 , 提高复用 , 分层开发中 , 上层不用管具体的实现 , 大家都遵守共同的标准 , 使得开发变得容易 , 规范性更好**

**关于接口的理解**

接口从更深层次的理解，应是定义 规范 约束 与实现的分离

接口本身反映了系统设计人员对系统的抽象理解

接口应有两类

第一类是对一个个体的抽象 可以理解为抽象体abstract class

第二类是对个体某一方面的抽象，即形成一个抽象面interface

个体可能有多个抽象面 抽象体和抽象面是有区别的

### 利用注解开发

**mybatis最初配置信息是基于 XML ,映射语句(SQL)也是定义在 XML 中的。而到MyBatis 3提供了新的基于注解的配置。不幸的是，Java 注解的的表达力和灵活性十分有限。最强大的 MyBatis 映射并不能用注解来构建**

- sql 类型主要分成 :

- - @select ()
  - @update ()
  - @Insert ()
  - @delete ()

**注意：**利用注解开发就不需要mapper.xml映射文件了 .

1、我们在我们的接口中添加注解

````java
//查询全部用户
@Select("select id,name,pwd password from user")
public List<User> getAllUser();
````



2、在mybatis的核心配置文件中注入

````xml
<!--使用class绑定接口-->
<mappers>
   <mapper class="com.kuang.mapper.UserMapper"/>
</mappers>
````



3、我们去进行测试

````java
@Test
public void testGetAllUser() {
   SqlSession session = MybatisUtils.getSession();
   //本质上利用了jvm的动态代理机制
   UserMapper mapper = session.getMapper(UserMapper.class);

   List<User> users = mapper.getAllUser();
   for (User user : users){
       System.out.println(user);
  }

   session.close();
}
````



### Mybatis详细的执行流程

![img](640.png)

### 注解增删改

【注意点：增删改一定记得对事务的处理】

改造MybatisUtils工具类的getSession( ) 方法，重载实现。

````java
  //获取SqlSession连接
  public static SqlSession getSession(){
      return getSession(true); //事务自动提交
  }
 
  public static SqlSession getSession(boolean flag){
      return sqlSessionFactory.openSession(flag);
  }
````

查询

![image-20200502201047115](image-20200502201047115.png)

新增

![image-20200502201214570](image-20200502201214570.png)

修改

![image-20200502201329939](image-20200502201329939.png)

删除

![image-20200502201422036](image-20200502201422036.png)

### 关于@Param

@Param注解用于给方法参数起一个名字。以下是总结的使用原则：

- 在方法只接受一个参数的情况下，可以不使用@Param。
- 在方法接受多个参数的情况下，建议一定要使用@Param注解给参数命名。
- 如果参数是 JavaBean ， 则不能使用@Param。
- 不使用@Param注解时，参数只能有一个，并且是Javabean。

### \#与$的区别

![image-20200502201610355](image-20200502201610355.png)

#占位符

$字符串替换

### 多对一（重点）

### 一对多（重点）

### 动态sql（重点）

##### 什么是动态sql？

动态sql指的是根据不同的查询条件，生成不同的sql语句。

```
动态 SQL 元素和 JSTL 或基于类似 XML 的文本处理器相似。在 MyBatis 之前的版本中，有很多元素需要花时间了解。MyBatis 3 大大精简了元素种类，现在只需学习原来一半的元素便可。MyBatis 采用功能强大的基于 OGNL 的表达式来淘汰其它大部分元素。
 -------------------------------
  - if
  - choose (when, otherwise)
  - trim (where, set)
  - foreach
  -------------------------------
```

#### 搭建环境

**新建一个数据库表：blog**

字段：id，title，author，create_time，views

1、创建Mybatis基础工程

![img](640-1588423870349.png)

2、IDutil工具类

````java
public class IDUtil {

   public static String genId(){
       return UUID.randomUUID().toString().replaceAll("-","");
  }

}
````

3、实体类编写  【注意set方法作用】

````java
import java.util.Date;

public class Blog {

   private String id;
   private String title;
   private String author;
   private Date createTime;
   private int views;
   //set，get....
}
````

4、编写Mapper接口及xml文件

![image-20200502205303028](image-20200502205303028.png)

5、mybatis核心配置文件，下划线驼峰自动转换

````xml
<settings>
   <setting name="mapUnderscoreToCamelCase" value="true"/>
   <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
<!--注册Mapper.xml-->
<mappers>
 <mapper resource="mapper/BlogMapper.xml"/>
</mappers>
````

6、插入初始数据

![image-20200502205547173](image-20200502205547173.png)

### if语句

需求：根据作者名字和博客名字查询博客，如果作者名字为空，那么直根据博客名字查询，反之，则根据作者名字查询

````xml
<!--需求1：
根据作者名字和博客名字来查询博客！
如果作者名字为空，那么只根据博客名字查询，反之，则根据作者名来查询
select * from blog where title = #{title} and author = #{author}
-->
<select id="queryBlogIf" parameterType="map" resultType="blog">
  select * from blog where
   <if test="title != null">
      title = #{title}
   </if>
   <if test="author != null">
      and author = #{author}
   </if>
</select>
````



### Where标签

where标签会知道如果它包含的标签中有返回值的话，它就插入一个where 此外如果标签返回的内容是以and 或 or开头的 则它会剔除掉

**where标签套在了if标签外面**

````xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
  select * from blog
   <where>
       <if test="title != null">
          title = #{title}
       </if>
       <if test="author != null">
          and author = #{author}
       </if>
   </where>
</select>
````

### Set标签

上面的对于sql语句包含where关键字，如果在进行更新操作的时候时候含有set关键字 我们用set标签进行处理

set标签套在了if标签外面

````xml
<!--注意set是用的逗号隔开-->
<update id="updateBlog" parameterType="map">
  update blog
     <set>
         <if test="title != null">
            title = #{title},
         </if>
         <if test="author != null">
            author = #{author}
         </if>
     </set>
  where id = #{id};
</update>
````

### choose语句

有时候我们不想用到所有的查询条件 只想选择其中的一个 查询条件有一个满足即可，使用choose标签可以解决此类问题 类似于java中的switch语句

choose标签中包含 when when otherwise标签

````xml
<select id="queryBlogChoose" parameterType="map" resultType="blog">
  select * from blog
   <where>
       <choose>
           <when test="title != null">
                title = #{title}
           </when>
           <when test="author != null">
              and author = #{author}
           </when>
           <otherwise>
              and views = #{views}
           </otherwise>
       </choose>
   </where>
</select>
````

### Foreach标签

将数据库中前三个数据的id修改为1,2,3；

需求：**我们需要查询 blog 表中 id 分别为1,2,3的博客信息**

separator -赛破瑞特 --分隔符

````xml
<select id="queryBlogForeach" parameterType="map" resultType="blog">
  select * from blog
   <where>
       <!--
       collection:指定输入对象中的集合属性
       item:每次遍历生成的对象
       open:开始遍历时的拼接字符串
       close:结束时拼接的字符串
       separator:遍历对象之间需要拼接的字符串
       select * from blog where 1=1 and (id=1 or id=2 or id=3)
     -->
       <foreach collection="ids"  item="id" open="and (" close=")" separator="or">
          id=#{id}
       </foreach>
   </where>
</select>
````

collection 指定输入对象中的集合属性 

item 每次遍历生成的对象

open 开始遍历时候的凭借字符串

close 结束时候拼接的字符串

separator 遍历对象之间需要拼接的字符串

select * from blog where 1=1 and （id =1 or id=2 or id=3）

小结：其实动态 sql 语句的编写往往就是一个拼接的问题，为了保证拼接准确，我们最好首先要写原生的 sql 语句出来，然后在通过 mybatis 动态sql 对照着改，防止出错。多在实践中使用才是熟练掌握它的技巧。

## 缓存 Cache -卡死

### 缓存原理图

![img](640.webp)

什么 是缓存？

- 存在内存中的临时数据
- 将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘（关系型数据库）查询，从缓存查询从而提高查询效率，解决了高并发系统的性能问题。

为什么使用缓存？

- 减少和数据库的交互次数，减少系统开销，提高系统效率。

什么样的数据能使用缓存？

- 经常查询并且不经常修改的数据。

### MyBatis缓存

MyBatis中系统默认定义了两级缓存

- 默认情况下，只有一级缓存开启（sqlsession级别的缓存，也曾为本地缓存）
- 二级缓存需要手动开启和配置 他是基于命名空间级别的缓存。

#### 一级缓存

一级缓存就是一个map是sqlsession级别的。

- 一级缓存也叫本地缓寸
- 与数据库同一次会话期间查询到的数据会放在本地缓存中 sqlsession
- 以后如果需要获取同样的数据，直接充缓存中拿，没必要再去查询数据库。

#### 一级缓存失效的四种情况

一级缓存是sqlsession级别的缓存，是一直开启的，我们关闭不了它。

一级缓存失效的情况就是好还需要再向数据库中发起一次查询请求。

1.sqlsession不同

2.sqlsession相同，查询条件不同

3.sqlsession相同，两次查询之间执行了增删改操作。

4.sqlsession相同，手动清除了一级缓存。 session.clearCache();//手动清除缓存

### 二级缓存

- 二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存。
- 基于namespace级别的缓存，一个名称空间，对应一个二级缓存。
- 工作机制
  - 一个会话查询一条数据，这个数据就会被放到当前会话的一级缓存中；
  - 如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据被保存到二级缓存中；
  - 新的会话查询信息，就可以从二级缓存中获取内容。
  - 不同的mapper查出的数据会放在自己对应的缓存中。

总结：

- 只要开启了二级缓存，我们在同一个Mapper中的查询，可以在二级缓存中拿到数据
- 查出的数据都会被默认先放在一级缓存中
- 只有会话提交或者关闭以后，一级缓存中的数据才会转到二级缓存中

