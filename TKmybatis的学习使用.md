# SpringBoot集成TkMybatis插件

基于SpringBoot项目,正常集成Mybatis后,为了简化sql语句的编写,甚至达到无mapper.xml文件。

 tkmybatis是在mybatis框架的基础上提供了很多工具，让开发更加高效 。

[TOC]

## 1.实现步骤：

```
1.引入TMMybatis的Maven依赖
2.实例类的相关配置@Id，@Table
3.Mapper继承TkMybatis的Mapper接口
4.启动类Application或自定义Mybatis配置类上使用@MapperScan注解扫描Mapper接口。
5.在application.properties配置文件中，配置mapper.xml文件指定的位置【可选】
6.使用TkMybatis提供的SQL执行方法。
ps:
   1.TkMybatis默认使用继承Mapper接口中传入的实体类对象去数据库寻找对应的表，因此如果表名和实体类名不满足对应规则时，会报错，这时使用@Table为实体指定表。（这种对应规则为驼峰命名规则）
   2.使用TkMybatis可以无xml文件实现数据库操作，只需要继承TkMybatis的Mapper接口即可。
   3.如有需要，实现mapper.xml自定义的SQL语句。
```

## 2.实现细节：

### 2.1 引入TkMybatis的maven依赖

```
<!-- https://mvnrepository.com/artifact/tk.mybatis/mapper -->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper</artifactId>
            <version>4.0.3</version>
        </dependency>
<!-- https://mvnrepository.com/artifact/tk.mybatis/mapper-spring-boot-starter -->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
            <version>2.0.3</version>
        </dependency>

// 注意：在公司的pom文件中好像只引入了springboot对mapper接口的支持。
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>1.1.2</version>
</dependency>
```

### 2.2实体类的配置

```
package com.asiainfo.lxc.production.score_query.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

/**
 * @author lixiaochi
 * @date 2019/11/18-10:18
 */
// @Table指定该实体类对应的表名，如表名为user，类名为User（驼峰命名法）可以不需要此注解。
@Data
@Table(name="user")
@NoArgsConstructor
@AllArgsConstructor
public class User {
    // @Id表示字段对应数据库表的主键id
    // @GeneratedValue中strategy表示使用数据库自带的主键生成策略。
    // @GeneratedValue中generator配置为"JDBC"，在数据插入完毕后，会自动将主键id填充到实体类中，类似于mapper.xml中的selectKey。
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY,generator = "JDBC")
    @Column(name="userid")
    private Long userId;
    @Column(name="username")
    private String userName;
    @Column(name="password")
    private String password;
    @Column(name="phone")
    private String phone;
    @Column(name="address")
    private String address;
}
// 其中@Table即数据表表名，@Column即列名，@Id作为主键，需要注意，@Id注解不可有多个，@Transient即冗余字段，不与数据库任何字段对应。
@Column描述了数据库表中该字段的详细定义,这对于根据JPA注解生成数据库表结构的工具非常 有作用.
name:表示数据库表中该字段的名称,默认情形属性名称一致
nullable:表示该字段是否允许为null,默认为true
unique:表示该字段是否是唯一标识,默认为false
length:表示该字段的大小,仅对String类型的字段有效
@Basic注解表示是否懒加载。
```

### 2.3Mapper继承TkMybatis的mapper接口

```
package com.asiainfo.lxc.production.score_query.mapper;

import com.asiainfo.lxc.production.score_query.pojo.User;
import tk.mybatis.mapper.common.Mapper;

/**
 * @author lixiaochi
 * @date 2019/11/18-10:33
 */
public interface UserMapper extends Mapper<User> {
	// 这里需要注意的是，实现了UserMapper的实现类因为继承了TkMybatis插件的
	// Mapper接口，所以提供了一些单表查询的方法。不需要我们重新在写了
	// 提高我们的开发速度。
}

```

### 2.4启动Application或自定义Mybatis配置类上使用@MapperScan注解扫描Mapper接口。

```
package com.asiainfo.lxc.production.score_query;

import lombok.extern.slf4j.Slf4j;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("com.asiainfo.lxc.production.score_query.mapper")
@Slf4j
public class ScoreQueryApplication {

    public static void main(String[] args) {

        log.info("程序启动......");
        SpringApplication.run(ScoreQueryApplication.class, args);
        log.info("程序启动成功......");
    }

}
```

### 2.5application.properties配置文件mapper.xml配置文件的扫描路径。

> ```
> 注意在这里可以使用两种方法，定义自定义的SQL查询语句。
> // 一种xml配置文件方式：可以在Mapper.xml配置文件中，自定义SQL语句的编写。
> mybatis.mapperLocations=classpath*:com.asiainfo.lxc.production.score_query.mapper/*.xml
> 
> // 一种注解配置方法：直接在UserMapper接口中使用mybatis中CRUD注解。
> package com.asiainfo.lxc.production.score_query.mapper;
> 
> import com.asiainfo.lxc.production.score_query.pojo.User;
> import org.apache.ibatis.annotations.Select;
> import org.springframework.stereotype.Repository;
> import tk.mybatis.mapper.common.Mapper;
> 
> import java.util.List;
> 
> /**
>  * @author lixiaochi
>  * @date 2019/11/18-10:33
> */
> @Repository
> public interface UserMapper extends Mapper<User> {
>  @Select("select * from user")
>  public List<User> mySelectAll();
> }
> ```

### 2.6使用TkMybatis提供的SQL执行方法

```
package com.asiainfo.lxc.production.score_query.service.Impl;

import com.asiainfo.lxc.production.score_query.mapper.UserMapper;
import com.asiainfo.lxc.production.score_query.pojo.User;
import com.asiainfo.lxc.production.score_query.service.UserService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

/**
 * @author lixiaochi
 * @date 2019/11/18-10:43
 */
@Service
@Transactional
@Slf4j
public class UserServiceImpl implements UserService {
    @Autowired
    private UserMapper userMapper;

    /*
    使用Mapper接口提供的方法实现用户查找
     */
    @Override
    public List<User> selectAllUser() {
        log.info("使用接口中定义的方法查询，当前用户列表是：");
        List<User> users = userMapper.selectAll();
        System.out.println(users);
        return users;
    }
    
    @Override
    public List<User> mySelectAllUser() {
        log.info("使用用户自定义的方法查询，当前用户列表是：");
        return userMapper.mySelectAll();
    }
}
```

### 2.7TkMybatis提供的常用方法和注意事项

实体类按照如下规则和数据库表进行转换，注解全部是JPA中的注解

```
定义实体类：
  1.表名默认使用类名，驼峰转下划线（只对大写字母进行处理），如UserInfo默认对一个的表名为user_info。
  2.表名可以使用@Table(name="tableName")进行指定，对不符合第一条默认规则的可以通过这种方式指定表名。
  3.字段默认和@Column一样，都会作为表字段，表字段默认为Java对象的Field名字驼峰转下划线形式。
  4.可以使用@Column(name="fieldName")指定不符合第三条规则的字段名。
  5.使用@Transient注解可以忽略字段，添加该注解的字段不会作为表字段使用。
  6.建议一定是有一个@Id注解作为主键的字段，可以有多个@Id注解的字段作为联合主键。
```

TkMybatis提供的查询方法：

```
// userMapper继承Mapper接口。
UserMapper extends Mapper<User>
// Mapper接口继承自其它几个接口
Mapper<T> extends BaseMapper<T>, ExampleMapper<T>, RowBoundsMapper<T>, Marker
// BaseMapper接口实现了几个重要的接口
BaseMapper<T> extends BaseSelectMapper<T>, BaseInsertMapper<T>, BaseUpdateMapper<T>, BaseDeleteMapper<T>
```

**所有的Mapper继承此类将具有以下通用方法**  

比如UserMapper extends Mapper<User>就包含了这个借口提供的一些基础方法。

也可以使用其他接口   UserMapper extends SelectByIdsMapper<User>

1.查询方法

BaseSelectMapper下的通用方法

| 方法名称                                 | 作用                       |
| ---------------------------------------- | -------------------------- |
| List<tableName/实体类名>  selectAll      | 查询指定表的所有数据       |
| T selectByPrimaryKey(Object key)         | 通过主键查询               |
| T  selectOne(T record)                   | 通过实体查询单个数据       |
| List<T> select(T record)                 | 通过实体查询多个数据       |
| int selectCount(T record)                | 通过实体查询实体数量       |
| boolean existsWithPrimaryKey(Object key) | 通过主键查询此主键是否存在 |

具体代码：

```java
@Override
    public List<User> selectAllUser() {
        List<User> users = userMapper.selectAll();
        log.info("使用接口中定义的方法查询，当前所有用户的列表：");
        System.out.println(users);
        return users;
    }

    @Override
    public User selectPrimaryKey() {
        log.info("使用mapper接口提供的selectByPrimaryKey查询数据");
        User user = userMapper.selectByPrimaryKey(2L);
        return user;
    }

    @Override
    public User selectOne() {
        User userdemo = new User();
        userdemo.setUserId(1L);
        userdemo.setUserName("李小池");
        log.info("使用mapper接口提供的selectOne查询单个数据:用户Id{},用户名name{}",userdemo.getUserId(),userdemo.getUserName());
        User user = userMapper.selectOne(userdemo);
        return user;
    }

    @Override
    public List<User> select() {
        User userdemo = new User();
        userdemo.setPassword("123456");
        log.info("使用mapper接口提供的select查询多个相同用户密码的数据:用户密码{}",userdemo.getPassword());
        List<User> select = userMapper.select(userdemo);
        return select;
    }
```

SelectByIdsMapper下的通用方法

| 方法名称                          | 作用                 |
| :-------------------------------- | :------------------- |
| List<T> selectByIds(String var1); | 通过多个主键查询数据 |

添加方法：

BaseInsertMapper的通用方法

**注意：**如果想使用数据库自带的主键自增方法可以在实体类中添加注解。插入成功1，否则报错。

// @Id表示字段对应数据库表的主键id
// @GeneratedValue中strategy表示使用数据库自带的主键生成策略。

// @GeneratedValue中generator配置为"JDBC"，在数据插入完毕后，会自动将主键id填充到实体类中，类似于mapper.xml中的selectKey。
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY,generator = "JDBC")
@Column(name="userid")
private Long userId;

| 方法名称                      | 作用                     |
| ----------------------------- | ------------------------ |
| int insert(T record)          | 全部添加                 |
| int insertSelective(T record) | 选择性（不为null）的添加 |

```
    @Override
    public int insert() {
        User userdemo = new User();
        userdemo.setUserName("为null都插入");
        log.info("正在插入数据");
        int status = userMapper.insert(userdemo);
        log.info("插入数据成功");
        return status;
    }

    @Override
    public int insertSelective() {
        User userdemo = new User();
        userdemo.setUserName("选择(不为null)插入");
        log.info("正在插入数据");
        int status = userMapper.insert(userdemo);
        log.info("插入数据成功并且用户id{}",userdemo.getUserId());
        return status;
    }
```

MySqlMapper下的通用方法

| 方法名称                              | 作用                                         |
| :------------------------------------ | :------------------------------------------- |
| int insertList(List<T> list);         | 批量插入                                     |
| int insertUseGeneratedKeys(T record); | 如果主键为自增可使用此方法获取添加成功的主键 |

OracleMapper下的通用方法

| 方法名称                      | 作用     |
| :---------------------------- | :------- |
| int insertList(List<T> list); | 批量插入 |

 修改方法 

 BaseUpdateMapper下的通用方法 

| 方法名称                                   | 作用                     |
| :----------------------------------------- | :----------------------- |
| int updateByPrimaryKey(T record);          | 按照实体进行修改         |
| int updateByPrimaryKeySelective(T record); | 按照实体进行有选择的修改 |

```
// 使用BaseUpdateMapper中要注意这两种方法的使用区别。
// 如果使用第一个可能会把已有数据进行类修改。
比如使用第一种方法，
数据库中的user 4, '更新测试开始:updateByPrimaryKeySelective', '123456', NULL, NULL
user为   4,'123',null等，就会把已有数据’123456‘替换成null。
而第二种方法不会。
    @Override
    public int updateByPrimaryKey() {
        log.info("更新测试开始:updateByPrimaryKey");
        User userdemo = new User();
        userdemo.setUserId(3L);
        userdemo.setUserName("更新测试开始:updateByPrimaryKey");
        int status = userMapper.updateByPrimaryKey(userdemo);
        return status;
    }

    @Override
    public int updateByPrimaryKeySelective() {
        log.info("更新测试开始:updateByPrimaryKeySelective");
        User userdemo = new User();
        userdemo.setUserId(4L);
        userdemo.setUserName("更新测试开始:updateByPrimaryKeySelective");
        int status = userMapper.updateByPrimaryKeySelective(userdemo);
        return status;
    }
```

删除方法

BaseUpdateMapper下的通用方法 返回的是删除的个数。

用户属性为null时，即都匹配。否则为要和指定的值匹配。且需要所有的条件都匹配。

| 方法名称                          | 作用             |
| :-------------------------------- | :--------------- |
| int delete(T record);             | 按照实体进行删除 |
| int deleteByPrimaryKey(Object o); | 按照主键进行删除 |

IdsMapper下的通用方法

| 方法名称                      | 作用             |
| :---------------------------- | :--------------- |
| int deleteByIds(String var1); | 按照主键批量删除 |