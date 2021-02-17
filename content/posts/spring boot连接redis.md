---
date: "2018-03-24 19:00:48"
title: spring boot连接redis
---
**Spring-data-redis**为spring-data模块中对redis的支持部分，简称为“SDR”，提供了基于jedis客户端API的高度封装以及与spring容器的整合，

jedis客户端在编程实施方面存在如下不足：

- connection管理缺乏自动化，connection-pool的设计缺少必要的容器支持。
- 数据操作需要关注“序列化”/“反序列化”，因为jedis的客户端API接受的数据类型为string和byte，对结构化数据(json,xml,pojo等)操作需要额外的支持。
- 事务操作纯粹为硬编码
- pub/sub功能，缺乏必要的设计模式支持，对于开发者而言需要关注的太多。

## 1 spring-data-redis特性

1. 连接池自动管理，提供了一个高度封装的“RedisTemplate”类
2. 针对jedis客户端中大量api进行了归类封装,将同一类型操作封装为operation接口
    - `ValueOperations`：简单K-V操作
    - `SetOperations`：set类型数据操作
    - `ZSetOperations`：zset类型数据操作
    - `HashOperations`：针对map类型的数据操作
    - `ListOperations`：针对list类型的数据操作
3. 提供了对key的“bound”(绑定)便捷化操作API，可以通过bound封装指定的key，然后进行一系列的操作而无须“显式”的再次指定Key，即BoundKeyOperations：
    - `BoundValueOperations`
    - `BoundSetOperations`
    - `BoundListOperations`
    - `BoundSetOperations`
    - `BoundHashOperations`
4. 将事务操作封装，有容器控制。
5. 针对数据的“序列化/反序列化”，提供了多种可选择策略(RedisSerializer)
    - `JdkSerializationRedisSerializer`：POJO对象的存取场景，使用JDK本身序列化机制，将pojo类通过ObjectInputStream/ObjectOutputStream进行序列化操作，最终redis-server中将存储字节序列。是目前最常用的序列化策略。
    - `StringRedisSerializer`：Key或者value为字符串的场景，根据指定的charset对数据的字节序列编码成string，是“new String(bytes, charset)”和“string.getBytes(charset)”的直接封装。是最轻量级和高效的策略。
    - `JacksonJsonRedisSerializer`：jackson-json工具提供了javabean与json之间的转换能力，可以将pojo实例序列化成json格式存储在redis中，也可以将json格式的数据转换成pojo实例。因为jackson工具在序列化和反序列化时，需要明确指定Class类型，因此此策略封装起来稍微复杂。
    - `OxmSerializer`：提供了将javabean与xml之间的转换能力，目前可用的三方支持包括jaxb，apache-xmlbeans；redis存储的数据将是xml工具。不过使用此策略，编程将会有些难度，而且效率最低；不建议使用。
6. 基于设计模式，和JMS开发思路，将pub/sub的API设计进行了封装，使开发更加便捷。
7. spring-data-redis中，并没有对sharding提供良好的封装，如果你的架构是基于sharding，那么你需要自己去实现，这也是sdr和jedis相比，唯一缺少的特性。


## 2 引入依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## 3 配置

```properties
# REDIS (RedisProperties)
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=localhost
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=root
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=0
```

其中spring.redis.database的配置通常使用0即可，Redis在配置的时候可以设置数据库数量，默认为16，可以理解为数据库的schema

### 3.1 `StringRedisTemplate`

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoApplicationTests {

	@Autowired
	private StringRedisTemplate stringRedisTemplate;

	@Test
	public void testRedis(){
		stringRedisTemplate.opsForValue().set("myKey", "hello redis");
		Assert.assertEquals("hello redis", stringRedisTemplate.opsForValue().get("myKey"));
	}

}
```

通过上面这段极为简单的测试案例演示了如何通过自动配置的StringRedisTemplate对象进行Redis的读写操作，该对象从命名中就可注意到支持的是String类型。如果有使用过spring-data-redis的开发者一定熟悉RedisTemplate<K, V>接口，StringRedisTemplate就相当于RedisTemplate<String, String>的实现。

除了String类型，我们还经常会在Redis中存储对象。

### 3.2 `RedisTemplate<Object, Object>`

#### 3.2.1 新建User类

```
@Data
@AllArgsConstructor
public class User implements Serializable{
    private static final long serialVersionUID = 1L;
    private Integer id;
    private String username;
    private Integer age;
}
```

#### 3.2.2 创建UserRepository

```
@Repository
public class UserRepository {
    @Autowired
    private RedisTemplate<Object, Object> redisTemplate;

    //
    @Resource(name = "redisTemplate")
    ValueOperations<Object, Object> valOps;

    /**
     * 保存
     * @param user
     */
    public void save(User user) {
        int id = user.getId();
        valOps.set(id, user);
    }

    /**
     * 获取
     * @param id
     * @return
     */
    public User getUserById(int id) {
        return (User) valOps.get(id);
    }

}
```

@Resource注解和@Autowired一样，也可以标注在字段或属性的setter方法上，但它默认按名称装配。名称可以通过@Resource的name属性指定，如果没有指定name属性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。

#### 3.2.3 单元测试

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoApplicationTests {
	@Autowired
	private UserRepository userRepository;

	@Test
	public void testRedis(){
		User user=new User(1, "hello", 12);
		userRepository.save(user);
		Assert.assertEquals("hello", userRepository.getUserById(1).getUsername());
	}
}
```

## 4 参考资料

[SpringBoot之Redis的支持](http://blog.csdn.net/smartdt/article/details/78894013)

[Spring-data-redis特性与实例](http://shift-alt-ctrl.iteye.com/blog/1886831)