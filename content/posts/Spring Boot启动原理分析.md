---
date: "2018-03-24 19:03:22"
title: Spring Boot启动原理分析
---
# Spring Boot启动原理分析

我们在开发spring boot应用的时候，一般会遇到如下的启动类：

```
@SpringBootApplication
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```
从这段代码可以看出，注解@SpringBootApplication和SpringApplication.run()是比较重要的两个东西。
## 1 @SpringApplication注解

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
...
}
```
在这段代码里，比较重要的只有三个注解：

- @Configuration（@SpringBootConfiguration点开查看发现里面还是应用了@Configuration）
- @EnableAutoConfiguration
- @ComponentScan

其实，我们使用这三个注解来修饰springboot的启动类也可以正常运行,如下所示：
```
@ComponentScan
@EnableAutoConfiguration
@Configuration
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```
每次写这三个注解的话，比较繁琐，所以就spring团队就封装了一个@SpringBootApplication。

## 1.1 @Configuration
@Configuration就是JavaConfig形式的Spring Ioc容器的配置类使用的那个@Configuration，SpringBoot社区推荐使用基于JavaConfig的配置形式，所以，这里的启动类标注了@Configuration之后，本身其实也是一个IoC容器的配置类。

XML跟config配置方式的区别可以从如下几个方面来说：
- 表达形式层面
  基于xml的配置方式是这样的：
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
       default-lazy-init="true">
    <!--bean定义-->
  </beans>
  ```


  基于java config配置方式是这样的：

  ```
  @Configuration
  public class MockConfiguration{
      //bean定义
  }
  ```

- 注册bean定义层面
  基于XML的配置形式是这样：	

  ```xml
  <bean id="mockService" class="..MockServiceImpl">
      ...
  </bean>
  ```

  而基于Java config的配置形式是这样的：

  ```
  @Configuration
  public class MockConfiguration{
  	@Bean
  	public MockService mockService(){
      	return new MockServiceImpl();
  	}
  }
  ```
  任何一个标注了@Bean的方法，其返回值将作为一个bean定义注册到Spring的IoC容器，方法名将默认成该bean定义的id。

- 表达依赖注入关系层面
  为了表达bean与bean之间的依赖关系，在XML形式中一般是这样：

  ```xml
  <bean id="mockService" class="..MockServiceImpl">
      <propery name ="dependencyService" ref="dependencyService" />
  </bean>

  <bean id="dependencyService" class="DependencyServiceImpl"></bean>
  ```

  而基于Java config的配置形式是这样的：

  ```
  @Configuration
  public class MockConfiguration{
      @Bean
      public MockService mockService(){
          return new MockServiceImpl(dependencyService());
      }
      
      @Bean
      public DependencyService dependencyService(){
          return new DependencyServiceImpl();
      }
  }
  ```

  如果一个bean的定义依赖其他bean,则直接调用对应的JavaConfig类中依赖bean的创建方法就可以了。

### 1.2 @ComponentScan

@ComponentScan的功能其实就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。

我们可以通过basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。

所以SpringBoot的启动类最好是放在root package下，因为默认不指定basePackages。

### 1.3 @EnableAutoConfiguration

Spring框架提供了各种名字为@Enable开头的Annotation定义，比如@EnableScheduling、@EnableCaching、@EnableMBeanExport等。@EnableAutoConfiguration的理念和做事方式其实一脉相承，简单概括一下就是，借助@Import的支持，收集和注册特定场景相关的bean定义。

@EnableAutoConfiguration也是借助@Import的帮助，将所有符合自动配置条件的bean定义加载到IoC容器。

```
@SuppressWarnings("deprecation")
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
...
}
```

@EnableAutoConfiguration作为一个复合Annotation，

其中，最关键的要属@Import(EnableAutoConfigurationImportSelector.class)，借助EnableAutoConfigurationImportSelector，@EnableAutoConfiguration借助于SpringFactoriesLoader的支持可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器。SpringFactoriesLoader的支持。

### 1.4 SpringFactoriesLoader

SpringFactoriesLoader属于Spring框架私有的一种扩展方案，其主要功能就是从指定的配置文件`META-INF/spring.factories`加载配置。

```
public abstract class SpringFactoriesLoader {
    public static <T> List<T> loadFactories(Class<T> factoryClass, ClassLoader classLoader) {
        ...
    }

    public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader){
        ....
    }
}

```

配合@EnableAutoConfiguration使用的话，它更多是提供一种配置查找的功能支持，即根据@EnableAutoConfiguration的完整类名org.springframework.boot.autoconfigure.EnableAutoConfiguration作为查找的Key,获取对应的一组@Configuration类。

![](http://7xqch5.com1.z0.glb.clouddn.com/springboot3-2.jpg)

@EnableAutoConfiguration自动配置流程就是：

- 从classpath中搜寻所有的META-INF/spring.factories配置文件；
- 并将其中org.springframework.boot.autoconfigure.EnableutoConfiguration对应的配置项通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类；
- 然后汇总为一个并加载到IoC容器。

## 2 SpringApplication

SpringApplication的run该方法的主要流程大体可以归纳如下：

**1）** 如果我们使用的是SpringApplication的静态run方法，那么，这个方法里面首先要创建一个SpringApplication对象实例，然后调用这个创建好的SpringApplication的实例方法。在SpringApplication实例初始化的时候，它会提前做几件事情：

- 根据classpath里面是否存在某个特征类（org.springframework.web.context.ConfigurableWebApplicationContext）来决定是否应该创建一个为Web应用使用的ApplicationContext类型。
- 使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationContextInitializer。
- 使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationListener。
- 推断并设置main方法的定义类。

**2）** SpringApplication实例初始化完成并且完成设置后，就开始执行run方法的逻辑了，方法执行伊始，首先遍历执行所有通过SpringFactoriesLoader可以查找到并加载的SpringApplicationRunListener。调用它们的started()方法，告诉这些SpringApplicationRunListener，“嘿，SpringBoot应用要开始执行咯！”。

**3）** 创建并配置当前Spring Boot应用将要使用的Environment（包括配置要使用的PropertySource以及Profile）。

**4）** 遍历调用所有SpringApplicationRunListener的environmentPrepared()的方法，告诉他们：“当前SpringBoot应用使用的Environment准备好了咯！”。

**5）** 如果SpringApplication的showBanner属性被设置为true，则打印banner。

**6）** 根据用户是否明确设置了applicationContextClass类型以及初始化阶段的推断结果，决定该为当前SpringBoot应用创建什么类型的ApplicationContext并创建完成，然后根据条件决定是否添加ShutdownHook，决定是否使用自定义的BeanNameGenerator，决定是否使用自定义的ResourceLoader，当然，最重要的，将之前准备好的Environment设置给创建好的ApplicationContext使用。

**7）** ApplicationContext创建好之后，SpringApplication会再次借助Spring-FactoriesLoader，查找并加载classpath中所有可用的ApplicationContext-Initializer，然后遍历调用这些ApplicationContextInitializer的initialize（applicationContext）方法来对已经创建好的ApplicationContext进行进一步的处理。

**8）** 遍历调用所有SpringApplicationRunListener的contextPrepared()方法。

**9）** 最核心的一步，将之前通过@EnableAutoConfiguration获取的所有配置以及其他形式的IoC容器配置加载到已经准备完毕的ApplicationContext。

**10）** 遍历调用所有SpringApplicationRunListener的contextLoaded()方法。

**11）** 调用ApplicationContext的refresh()方法，完成IoC容器可用的最后一道工序。

**12）** 查找当前ApplicationContext中是否注册有CommandLineRunner，如果有，则遍历执行它们。

**13）** 正常情况下，遍历执行SpringApplicationRunListener的finished()方法、（如果整个过程出现异常，则依然调用所有SpringApplicationRunListener的finished()方法，只不过这种情况下会将异常信息一并传入处理）

去除事件通知点后，整个流程如下图所示：

![](http://7xqch5.com1.z0.glb.clouddn.com/springboot3-3.jpg)

## 3 参考资料
[Spring Boot干货系列：（三）启动原理解析](http://tengj.top/2017/04/24/springboot0/)
[SpringBoot揭秘快速构建为服务体系](http://product.dangdang.com/23964779.html)


