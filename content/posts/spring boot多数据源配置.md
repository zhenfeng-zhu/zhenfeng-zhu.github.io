---
date: "2018-03-24 19:01:37"
title: spring boot多数据源配置
---
# spring boot多数据源配置

在单数据源的情况下，Spring Boot的配置非常简单，只需要在application.properties文件中配置连接参数即可。但是往往随着业务量发展，我们通常会进行数据库拆分或是引入其他数据库，从而我们需要配置多个数据源。

## 1 准备

### 1.1 禁止DataSourceAutoConfiguration

首先要将spring boot自带的`DataSourceAutoConfiguration`禁掉，因为它会读取`application.properties`文件的`spring.datasource.*`属性并自动配置单数据源。在`@SpringBootApplication`注解中添加`exclude`属性即可：

```
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(JpaDemoApplication.class, args);
	}
}
```

### 1.2 配置数据库连接

然后在`application.properties`中配置多数据源连接信息：

```properties
spring.datasource.primary.url=jdbc:mysql://localhost:3306/test
spring.datasource.primary.username=root
spring.datasource.primary.password=root
spring.datasource.primary.driver-class-name=com.mysql.jdbc.Driver

spring.datasource.secondary.url=jdbc:mysql://localhost:3306/test1
spring.datasource.secondary.username=root
spring.datasource.secondary.password=root
spring.datasource.secondary.driver-class-name=com.mysql.jdbc.Driver
```

### 1.3 手段创建数据源

由于我们禁掉了自动数据源配置，因些下一步就需要手动将这些数据源创建出来：

```
@Configuration
public class DataSourceConfig {

    @Bean(name = "primaryDataSource")
//    @Qualifier(value = "primaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.primary")
    public DataSource primaryDataSource(){
        return DataSourceBuilder.create().build();
    }


    @Bean(name = "secondaryDataSource")
//    @Qualifier(value = "secondaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

## 2 jdbcTemplate多数据源

### 2.1 jdbcTemplate的数据源配置

新建jdbcTemplate的数据源配置：

```
@Configuration
public class JdbcTemplateConfig {
    @Bean(name = "primaryJdbcTemplate")
    public JdbcTemplate primaryJdbcTemplate(
            @Qualifier("primaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean(name = "secondaryJdbcTemplate")
    public JdbcTemplate secondaryJdbcTemplate(
            @Qualifier("secondaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

### 2.2 单元测试

然后编写单元测试用例：

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class JpaDemoApplicationTests {
    @Autowired
    @Qualifier("primaryJdbcTemplate")
    protected JdbcTemplate jdbcTemplate1;

    @Autowired
    @Qualifier("secondaryJdbcTemplate")
    protected JdbcTemplate jdbcTemplate2;

    @Test
    public void testJdbc() {
        // 往第一个数据源中插入两条数据
        jdbcTemplate1.update("insert into users(id,name,age) values(?, ?, ?)", 1, "aaa", 20);
        jdbcTemplate1.update("insert into users(id,name,age) values(?, ?, ?)", 2, "bbb", 30);

        // 往第二个数据源中插入一条数据，若插入的是第一个数据源，则会主键冲突报错
        jdbcTemplate2.update("insert into users(id,name,age) values(?, ?, ?)", 1, "aaa", 20);

        // 查一下第一个数据源中是否有两条数据，验证插入是否成功
        Assert.assertEquals("2", jdbcTemplate1.queryForObject("select count(1) from users", String.class));

        // 查一下第一个数据源中是否有两条数据，验证插入是否成功
        Assert.assertEquals("1", jdbcTemplate2.queryForObject("select count(1) from users", String.class));

    }


    @Test
    public void contextLoads() {
    }
}
```

## 3 mybatis多数据源配置

### 3.1 自定义SqlSessionFactory

新建两个mybatis的SqlSessionFactory配置：

```
@Configuration
@MapperScan(basePackages = {"com.example.jpademo.primary.mapper"}, sqlSessionFactoryRef = "sqlSessionFactory1")
public class MybatisPrimaryConfig {
    @Autowired
    @Qualifier("primaryDataSource")
    private DataSource primaryDataSource;

    @Bean
    public SqlSessionFactory sqlSessionFactory1() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        // 使用primaryDataSource数据源
        factoryBean.setDataSource(primaryDataSource);
        return factoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplate1() throws Exception {
        // 使用上面配置的Factory
        SqlSessionTemplate template = new SqlSessionTemplate(sqlSessionFactory1());
        return template;
    }
}
```

这样，`com.example.jpademo.primary.mapper`包下的所有mapper就会用`sqlSessionFactory1`。同理可以创建

`sqlSessionFactory2`：

```
@Configuration
@MapperScan(basePackages = {"com.example.jpademo.secondary.mapper"}, sqlSessionFactoryRef = "sqlSessionFactory2")
public class MybatisSecondaryConfig {
    @Autowired
    @Qualifier("secondaryDataSource")
    private DataSource secondaryDataSource;

    @Bean
    public SqlSessionFactory sqlSessionFactory2() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        // 使用primaryDataSource数据源
        factoryBean.setDataSource(secondaryDataSource);
        return factoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplate1() throws Exception {
        // 使用上面配置的Factory
        SqlSessionTemplate template = new SqlSessionTemplate(sqlSessionFactory2());
        return template;
    }
}
```

### 3.2 mapper和实体类

然后编写mapper和实体类：

```
@Data
public class User {
    private Integer id;
    private String name;
    private Integer age;
}
```

```
package com.example.jpademo.primary.mapper;

import com.example.jpademo.domain.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import org.springframework.beans.factory.annotation.Qualifier;

@Mapper
@Qualifier("userMapper1")
public interface UserMapper1 {
    @Select("select * from users where id=#{id}")
    User findById(@Param("id") Integer id);
}
```

```
package com.example.jpademo.secondary.mapper;

import com.example.jpademo.domain.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import org.springframework.beans.factory.annotation.Qualifier;

@Mapper
@Qualifier("userMapper2")
public interface UserMapper2 {
    @Select("select * from users where id=#{id}")
    User findById(Integer id);
}
```

### 3.3 单元测试

编写单元测试用例：

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class JpaDemoApplicationTests {

    @Autowired
    private UserMapper1 userMapper1;

    @Autowired
    private UserMapper2 userMapper2;

    @Test
    public void testMybatis() {
        User user1 = userMapper1.findById(1);
        User user2 = userMapper2.findById(1);

        Assert.assertEquals("aaa", user1.getName());
        Assert.assertEquals("ccc", user2.getName());
    }
}
```

## 4 参考资料

[Spring Boot + Mybatis多数据源和动态数据源配置](http://blog.csdn.net/neosmith/article/details/61202084)

[Spring Boot 两种多数据源配置：JdbcTemplate、Spring-data-jpa](http://www.spring4all.com/article/253)

