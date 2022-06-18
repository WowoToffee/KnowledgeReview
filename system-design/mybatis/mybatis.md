# mybatis

## jdbc 问题总结：

原始jdbc开发存在的问题如下：

1.  数据库连接创建、释放频繁造成系统资源浪费，从⽽影响系统性能。

2.  Sql语句在代码中硬编码，造成代码不易维护，实际应⽤中sql变化的可能较⼤，sql变动需要改变java代码。

3.  使⽤preparedStatement向占有位符号传参数存在硬编码，因为sql语句的where条件不⼀定，可能多也可能少，修改sql还要修改代码，系统不易维护。

4.  对结果集解析存在硬编码(查询列名)，sql变化导致解析代码变化，系统不易维护，如果能将数据库记录封装成pojo对象解析⽐较⽅便

解决思路：

1.  使用数据库连接池初始化连接资源

2.  将SQL语句抽取到XML配置文件中

3.  使用反射，内省等底层技术，⾃动将实体与表进⾏属性与字段的⾃动映射

## 自定义框架设计

### 	使用端：

​	提供核心配置文件：

​	sqlMapConfig.xml 存放数据源信息，引用mapper.xml。Mapping.xml ：SQL语句配置信息

	### 框架端：

1. 读取配置文件

   直接读取`sqlMapConfig.xm`文件，将其信息转换为流，然后将相关信息用JavaBean的方式存放在内存中。

   1. Configuration:存放数据库基本信息，Map <唯一标识，Mapping>, 唯一标识：namespace+'.'+id
   2. MapperdStatement: SQL语句，statement类型，输入类型java参数，输出类型java参数

2. 解析配置文件

   创建SQLSessionFactoryBuild类型，方法：sqlSessionFactory build();

   1. 使用 dom4j 解析设置文件，将解析出来的内容放入Configuration 和 MappedStatement中
   2. 创建SqlSessionFactory的实现类 DefaultSqlSessiong

3. 创建SqlSessionFactory:

   方法 openSession（）获取sqlSession接口的实现方法

4. 创建SQLSession接口以及实现类，主要封装curl方法

   ⽅法：selectList(String statementId,Object param)：查询所有 

   selectOne(String statementId,Object param)：查询单个 

   具体实现：封装JDBC完成对数据库表的查询操作

涉及设计模式：

Builder 构建者设计模式，工厂模式，代理模式

## 多对多查询

```xml
</resultMap> 
<result column="id" property="id"></result>
 <result column="username" property="username"></result>
 <result column="password" property="password"></result>
 <result column="birthday" property="birthday"></result>
 <collection property="roleList" ofType="com.lagou.domain.Role">
     <result column="rid" property="id"></result>
     <result column="rolename" property="rolename"></result>
 </collection>
</resultMap> 
<select id="findAllUserAndRole" resultMap="userRoleMap">
 select u.*,r.*,r.id rid from user u left join user_role ur on
u.id=ur.user_id
 inner join role r on ur.role_id=r.id
</select>

```

## 注解

```
@Insert：实现新增
@Update：实现更新
@Delete：实现删除
@Select：实现查询
@Result：实现结果集封装
@Results：可以与@Result ⼀起使⽤，封装多个结果集
@One：实现⼀对⼀结果集封装
@Many：实现⼀对多结果集封装
```

### 一对一

```
public interface OrderMapper {
 @Select("select * from orders")
 @Results({
 @Result(id=true,property = "id",column = "id"),
 @Result(property = "ordertime",column = "ordertime"),
 @Result(property = "total",column = "total"),
 @Result(property = "user",column = "uid",
 javaType = User.class,
 one = @One(select =
"com.lagou.mapper.UserMapper.findById"))
 })
 List<Order> findAll();
}
```

### 一对多

```
public interface UserMapper {
 @Select("select * from user")
 @Results({
 @Result(id = true,property = "id",column = "id"),
 @Result(property = "username",column = "username"),
 @Result(property = "password",column = "password"),
 @Result(property = "birthday",column = "birthday"),
 @Result(property = "orderList",column = "id",
 javaType = List.class,
 many = @Many(select =
"com.lagou.mapper.OrderMapper.findByUid"))
 })
 List<User> findAllUserAndOrder();
}

public interface OrderMapper {
 @Select("select * from orders where uid=#{uid}")
 List<Order> findByUid(int uid);
}
```

### 多对多

```java
public interface UserMapper {
 @Select("select * from user")
 @Results({
 @Result(id = true,property = "id",column = "id"),
 @Result(property = "username",column = "username"),
 @Result(property = "password",column = "password"),
 @Result(property = "birthday",column = "birthday"),
 @Result(property = "roleList",column = "id",
 javaType = List.class,
 many = @Many(select =
"com.lagou.mapper.RoleMapper.findByUid"))
})
List<User> findAllUserAndRole();}
public interface RoleMapper {
 @Select("select * from role r,user_role ur where r.id=ur.role_id and
ur.user_id=#{uid}")
 List<Role> findByUid(int uid);
}
```



