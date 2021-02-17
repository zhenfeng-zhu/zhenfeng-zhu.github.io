---
date: "2018-03-24 17:57:15"
title: Guice快速入门
---
# Guice快速入门

接手的新项目主要是使用kotlin+vert.x来写的，使用gradle构建，依赖注入框架使用了guice。这段时间都是在熟悉代码的过程，恶补一些知识。

guice是谷歌推出的一个轻量级的依赖注入框架，当然spring也可以实现依赖注入，只是spring太庞大了。

## 1 基本使用

### 引入依赖

使用gradle或者maven，引入guice。

maven:

```
<dependency>
    <groupId>com.google.inject</groupId>
    <artifactId>guice</artifactId>
    <version>4.1.0</version>
</dependency>
```

Gradle:

```
compile "com.google.inject:guice:4.1.0"
```

### 项目骨架

首先需要一个业务接口，包含一个方法来执行业务逻辑，它的实现非常简单：

```
package com.learning.guice;
public interface UserService {
    void process();
}


package com.learning.guice;
public class UserServiceImpl implements UserService {
    @Override
    public void process() {
        System.out.println("我需要做一些业务逻辑");
    }
}
```

然后写一个日志的接口：

```
package com.learning.guice;
public interface LogService {
    void log(String msg);
}

package com.learning.guice;
public class LogServiceImpl implements LogService {
    @Override
    public void log(String msg) {
        System.out.println("------LOG: " + msg);
    }
}
```

最后是一个系统接口和相应的实现，在实现中使用了业务接口和日志接口处理业务逻辑和打印日志信息：

```
package com.learning.guice;
public interface Application {
    void work();
}


package com.learning.guice;
import com.google.inject.Inject;
public class MyApp implements Application {
    private UserService userService;
    private LogService logService;

    @Inject
    public MyApp(UserService userService, LogService logService) {
        this.userService = userService;
        this.logService = logService;
    }

    @Override
    public void work() {
        userService.process();
        logService.log("程序正常运行");
    }
}
```

### 配置依赖注入

guice是使用java代码来配置依赖。继承AbstractModule类，并重写其中的config方法。在config方法中，调用AbstractModule类中提供的方法来配置依赖关系。最常用的是bind(接口).to(实现类)。

```
package com.learning.guice;

import com.google.inject.AbstractModule;

public class MyAppModule extends AbstractModule {

    @Override
    protected void configure() {
        bind(LogService.class).to(LogServiceImpl.class);
        bind(UserService.class).to(UserServiceImpl.class);
        bind(Application.class).to(MyApp.class);
    }
}
```

### 单元测试

guice配置完之后，我们需要调用Guice.createInjector方法传入配置类来创建一个注入器，然后使用注入器中的getInstance方法获取目标类。

```
package com.learning.guice;

import com.google.inject.Guice;
import com.google.inject.Injector;
import org.junit.BeforeClass;
import org.junit.Test;

public class MyAppTest {
    private static Injector injector;

    @BeforeClass
    public static void init(){
        injector= Guice.createInjector(new MyAppModule());
    }

    @Test
    public void testMyApp(){
        Application application=injector.getInstance(Application.class);
        application.work();
    }
}
```

程序执行结果是：

```
/Library/Java/JavaVirtualMachines/jdk1.8.0_152.jdk/Contents/Home/bin/java -ea -...
我需要做一些业务逻辑
------LOG: 程序正常运行

Process finished with exit code 0
```

## 2 基本概念

### 2.1 Bingdings 绑定

- 链式绑定

  在绑定依赖的时候不仅可以将父类和子类绑定，还可以将子类和子类的子类进行绑定。

  ```
  public class BillingModule extends AbstractModule {
    @Override 
    protected void configure() {
      bind(TransactionLog.class).to(DatabaseTransactionLog.class);
      bind(DatabaseTransactionLog.class).to(MySqlDatabaseTransactionLog.class);
    }
  }
  ```

  在这种情况下，injector 会把所有 TransactionLog 替换为 MySqlDatabaseTransactionLog。

- 注解绑定

  当我们需要将多个同一类型的对象注入不同对象的时候，就需要使用注解区分这些依赖了。最简单的办法就是使用@Named注解进行区分。

  首先需要在要注入的地方添加@Named注解。

  ```
  public class RealBillingService implements BillingService {

    @Inject
    public RealBillingService(@Named("Checkout") CreditCardProcessor processor,
        TransactionLog transactionLog) {
      ...
    }
  ```

  然后在绑定中添加annotatedWith方法指定@Named中指定的名称。由于编译器无法检查字符串，所以Guice官方建议我们保守地使用这种方式。

  ```
  bind(CreditCardProcessor.class)
          .annotatedWith(Names.named("Checkout"))
          .to(CheckoutCreditCardProcessor.class);
  ```

- 实例绑定

  有时候需要直接注入一个对象的实例，而不是从依赖关系中解析。如果我们要注入基本类型的话只能这么做。

  ```
  bind(String.class)
          .annotatedWith(Names.named("JDBC URL"))
          .toInstance("jdbc:mysql://localhost/pizza");
  bind(Integer.class)
          .annotatedWith(Names.named("login timeout seconds"))
          .toInstance(10);
  ```

- @Privides方法

  当一个对象很复杂，无法使用简单的构造器来生成的时候，我们可以使用@Provides方法，也就是在配置类中生成一个注解了@Provides的方法。在该方法中我们可以编写任意代码来构造对象。

  @Provides方法也可以应用@Named和自定义注解，还可以注入其他依赖，Guice会在调用方法之前注入需要的对象。

  ```
  public class BillingModule extends AbstractModule {
    @Override
    protected void configure() {
      ...
    }

    @Provides
    TransactionLog provideTransactionLog() {
      DatabaseTransactionLog transactionLog = new DatabaseTransactionLog();
      transactionLog.setJdbcUrl("jdbc:mysql://localhost/pizza");
      transactionLog.setThreadPoolSize(30);
      return transactionLog;
    }
  }
  ```

- Provider绑定

  如果项目中存在多个比较复杂的对象需要构建，使用@Provide方法会让配置类变得比较乱。我们可以使用Guice提供的Provider接口将复杂的代码放到单独的类中。办法很简单，实现Provider<T>接口的get方法即可。在Provider类中，我们可以使用@Inject任意注入对象。

  ```
  public class DatabaseTransactionLogProvider implements Provider<TransactionLog> {
    private final Connection connection;

    @Inject
    public DatabaseTransactionLogProvider(Connection connection) {
      this.connection = connection;
    }

    public TransactionLog get() {
      DatabaseTransactionLog transactionLog = new DatabaseTransactionLog();
      transactionLog.setConnection(connection);
      return transactionLog;
    }
  }
  ```

  然后在config方法中，调用.toProvider方法：

  ```
  public class BillingModule extends AbstractModule {
    @Override
    protected void configure() {
      bind(TransactionLog.class)
          .toProvider(DatabaseTransactionLogProvider.class);
    }
  }
  ```

- 无目标绑定

  无目标绑定没有to子句

- 构造器绑定

  某些场景下，你能需要把某个类型绑定到任意一个构造函数上。以下情况会有这种需求：1、 @Inject 注解无法被应用到目标构造函数；2、目标类是一个第三方类；3、目标类有多个构造函数参与DI。

  为了解决这个问题，guice 提供了 toConstructor()绑定 ，它需要你指定要使用的确切的某个目标构造函数，并处理 "constructor annot be found" 异常：

  ```
  public class BillingModule extends AbstractModule {
    @Override 
    protected void configure() {
      try {
        bind(TransactionLog.class).toConstructor(
            DatabaseTransactionLog.class.getConstructor(DatabaseConnection.class));
      } catch (NoSuchMethodException e) {
        addError(e);
      }
    }
  }
  ```

- 内置绑定

  除了显示绑定和即时绑定 just-in-time bindings，剩下的绑定都属于injector的内置绑定。这些绑定只能由injector自己创建，不允许外部调用。

- 即时绑定

  当 injector 需要某一个类型的实例的时候，它需要获取一个绑定。在Module类中的绑定叫做显式绑定，只要他们可用，injector 就会在任何时候使用它们。如果需要某一类型的实例，但是又没有显式绑定，那么injector将会试图创建一个即时绑定（Just-in-time Bindings），也被称为JIT绑定 或 隐式绑定。

### 2.2 作用域

默认情况下Guice会在每次注入的时候创建一个新对象。如果希望创建一个单例依赖的话，可以在实现类上应用@Singleton注解。

```
@Singleton
public class InMemoryTransactionLog implements TransactionLog {
  /* everything here should be threadsafe! */
}
```

或者也可以在配置类中指定。

```
bind(TransactionLog.class).to(InMemoryTransactionLog.class).in(Singleton.class);
```

在`@Provides`方法中也可以指定单例。

```
@Provides @Singleton
  TransactionLog provideTransactionLog() {
    ...
  }
```

如果一个类型上存在多个冲突的作用域，Guice会使用bind()方法中指定的作用域。如果不想使用注解的作用域，可以在bind()方法中将对象绑定为Scopes.NO_SCOPE。

Guice和它的扩展提供了很多作用域，和spring一样，有单例Singleton，Session作用域SessionScoped，Request请求作用域RequestScoped等等。我们可以根据需要选择合适的作用域。

### 2.3 注入

guice的注入和spring类似，而且还做了一些扩展。

- 构造器注入

  使用 @Inject 注解标记类的构造方法，这个构造方法需要接受类依赖作为参数。大多数构造子将会把接收到的参数分派给内部成员变量。

- 方法注入

  Guice 可以向标注了 @Inject 的方法中注入依赖。依赖项以参数的形式传给方法，Guice 会在调用注入方法前完成依赖项的构建。注入方法可以有任意数量的参数，并且方法名对注入操作不会有任何影响。

- 字段注入

  使用 @Inject 注解标记字段。这是最简洁的注入方式。

  注意：不能给final字段加@Inject注解。

- 可选注入

  有的时候，可能需要一个依赖项存在则进行注入，不存在则不注入。此时可以使用方法注入或字段注入来做这件事，当依赖项不可用的时候Guice 就会忽略这些注入。如果你需要配置可选注入的话，使用 @Inject(optional = true) 注解就可以了。

- 按需注入

  方法注入和字段注入可以可以用来初始化现有实例，你可以使用 Injector.injectMembers。

  这个不常用。

- 静态注入

  不建议使用静态注入。

- 自动注入

  Guice 会对以下情形做自动注入：

  - 在绑定语句里，通过 toInstance() 注入实例。
  - 在绑定语句里，通过 toProvider() 注入 Provider 实例。这些对象会在注入器创建的时候被创建并注入容器。如果它们需要满足其他启动注入，Guice 会在它们被使用前将他们注入进去。

### 2.4 AOP

guice的aop功能较弱，时间原因还没研究透，后续继续写。