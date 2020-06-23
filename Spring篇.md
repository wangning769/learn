## Spring篇

### 第一章Spring IOC

---

![](F:\GitHub\learn\image\Spring篇\image-20200505125006614.png)

#### IOC（控制反转）

​	控制：资源的获取方式

+ 主动式：自己创建需要的资源

```java
//主动式
class BookServlet{
	BookServcie service = new BookServcie();
    
}
```

+ 被动式：自己不创建资源，交给一个容器来创建和设置

```java
//被动式
class BookServlet{
	BookServcie service;
    public void test1(){
        service.getBook();//
    }
}
```

容器：管理所有的组件（有功能的类），容器帮我们创建BookService对象，并发BookService对象赋值过去，主动获取变为被动接受。 

#### DI（依赖注入）

容器能知道哪个组件（类）运行的时候，需要另外一个类（组件）；容器通过反射的形式，将容器中准备号的BookService对象注入到BookServlet中；

细节：

+ 容器中的对象在容器创建时被创建

+ 同一个组件，在容器中时单实例的

+ 多个相同对象

+ ```java
  org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'com.atguigu.bean.Person' available: expected single matching bean but found 2: person,person1
  ```

+ 未创建容器

  + ```java
    org.springframework.beans.factory.NoSuchBeanDefinitionException: No bean named 'person3' available
    ```

  

静态工厂：工厂本身不用创建对象，通过静态方法调用，对象 = 工厂类.工厂方法名()

实例工厂：工厂本身需要创建对象

​					工厂类 工厂对象 = new 工厂类();

​					工厂对象.getAirPlane("张三");

---

#### 上下文

多种上下文

+ `ClassPathXmlApplicationContext`表示类路径下的XML配置文件(**重点**)
+ `FileSystemXmlApplicationContext`表示系统路径下的XML配置文件（**重点**）
+ `AnnotationConfigApplicationContext`表示Java配置类中加载Spring应用上下文
+ `AnnotationConfigWebApplicationContext`表示Java配置类中加载Spring Web应用上下文

---

#### Bean的生命周期

+ Spring容器中管理的bean的生命周期要复杂很多

![](F:\GitHub\learn\image\Spring篇\Bean初始化过程.png)

##### Bean实现的接口（如果）

+ 7个如果的接口（3个包装+3个初始化+1个销毁）	

1. **包装**	
   + `BeanNameWare`接口 -> `setBeanName()`
   + `BeanFactoryAware`接口 -> `setBeanFactory()`
   + `ApplicationContextAware`接口->`setApplicationContext()`
2. **初始化**
   + `BeanPostProcessor`接口-> `postProcessBeforeInitialization()`
   + `InitializingBean`接口 -> `afterPropertiesSet()`
   + `BeanPostProcessor`接口 -> `postProcessBeforeInitialization()`
3. **销毁**
   + `DisposableBean`接口 -> `destroy()`

---

#### spring模块

![](F:\GitHub\learn\image\Spring篇\Spring模块.png)

---

#### 常用注解

> `@Component`:告诉Spring会为这个类创建Bean
>
> `@ComponentScan`：扫描包和所有子包，查找带有`@Component`注解的类
>
> `@ContextConfiguration`:加载配置位置，一般为被`@ComponentScan`修饰的`XXXConfig.class`
>
> `@Autowired`:自动装配，当没有匹配到bean时，Spring会抛出异常
>
> > 可以通过`required=false`,不装配，但可能会报**空指针**，所以慎用
>
> `@Configuration`:表示该类是一个配置类，该类应该包含在Spring应用上下文中如何创建Bean的细节。
>
> `@Bean`:用于显示的装配Bean,告诉Spring，将该Bean注册到Spring的上下文中的bean中
>
> `@import`:如果一个Config里的配置Bean过于笨重，则可以对其进行拆分，并通过该注解引入进来
>
> `@ImportResource`:又或者你需要引入的**配置在XML**文件中，因此就需要该标签进行引入
>
> > `@ImportResource("'classpath:cd-config.xml")`
>
> `@Profile`:激活环境
>
> `@Scope`:选择bean的作用域
>
> `@PropertySource`:用于注入外部的值

#### 显示的方式装配Bean

+ 当一个类没有被`@Compontent`修饰，就没有办法用`@Autowired`进行注入，那么我们就需要**显示的装配Bean**

+ 当一个方法被`@Bean`修饰时，Spring会拦截所有调用方法，实际只返回Spring所创建的bean，也就是Spring本身调用`sgtPeppers()`时所创建的bean

+ 具体方式

+ ```java
  @Bean
  public CompactDisc sgtPeppers(){
      return new SgtPeppers();
  }
  //调用sgtPeppers()方法被拦截
  @Bean
  public CDPlayer cdPlayer(){
      return new CDPlayer(sgtPeppers());
  }
  ```

#### 通过XML装配Bean

  ```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="person"  class="testDemo.Person" />
    
    as<bean id="cdPlayer"  class="testDemo.CDPlayer">
		<constructor-arg ref="person" />
	</bean>
</beans>
  ```

#### 高级装配

##### Spring Profile

##### 条件化的bean

+ 当需要一个或多个bean只有在应用的类路径下包含特定的库是才创建，或者希望某个bean只有当另外某个特定的bean也声明了之后才创建
+ 需要实现`Condition`接口并实现`matches()`方法

> `@Conditional`：用于条件化初始化bean
>
> `@ConditionalOnBean`：配置了某个特定Bean
>
> `@ConditionalOnClass`：

---

##### bean的作用域

+ 默认情况下，Spring应用上下文中所有bean都作为单例的形式创建的。
+ Spring定义了多种作用域，可以基于这些作用域创建bean
  + 单例（Singleton）：整个应用中只创建一个
  + 原型（Prototype）：Spring上下文获取，每次都会创建一个新的bean实例
  + 会话（Session）：Web应用中，为每个会话创建一个bean实例
  + 请求（request）：Web应用中，妹妹一个请求创建一个bean实例
+ **代理声明**：`proxyMode`,如果bean类型是具体的类的话，就必须设置为`ScopedProxyMode.TARGET_CLASS`以此**表明要以生成目标类拓展的方式创建代理**
+ `@Scope`声明

 ```java
/*
 * 使用@Scope("prototype") 和 @Scope(scopeName = ConfigurableBeanFactory.SCOPE_PROTOTYPE)是一样的，
 * 但后者是常量更加安全不易出错
 * 	@Scope(scopeName = WebApplicationContext.SCOPE_REQUEST,proxyMode = ScopedProxyMode.INTERFACES)
 */
@Scope("prototype") // 单例
public Person1 createBean() {
    Person1 p = new Person1("admin");
    return p;
}
 ```

> ***另外注意：当bean的作用于方法参数时，Spring并不会实际注入bean,而是注入bean的代理；在真正调用方法时，代理会进行懒解析并将调用委托给会话作用域内的真正bean。***

+ XML方式声明代理作用域

```xml
<bean id="person" class=" com.test.Person">
    <!-- 生成基于接口的代理 -->
    <aop:scoped-proxy proxy-target-class="false" />
</bean>
```

> **aop命名空间暂不考虑**

---

##### 注入外部值

+ 需要用到`@PropertySource`注解，引入配置文件
+ 属性文件会自动加载到`Environment`中，可以通过这里检索属性

```java
@Configuration
@PropertySource("classpath:app.properties")
public class Player{
    @Autowired
	Environment evn;
    @Bean
    public Person createBean() {
        System.out.println("--------------"+evn);
        String name = evn.getProperty("dist.name");
        Person p = new Person(name);
        return p;
    }
}
	
```

>***值得说明的是Environment提供了4个getProperty()重载方法，和2个获取必须有值，否则抛出异常的方法***

```java
getProperty(String)
getProperty(String, String)
getProperty(String, Class<T>)
getProperty(String, Class<T>, T)
    
getRequiredProperty(String)
getRequiredProperty(String, Class<T>)
```

+ **也可以使用XML配置文件**

```xml
<context:property-placeholder />
```

##### 总结

> 本章重点是Spring装配的一些高级方法，以及Spring对bean创建方式，**单例、多实例、会话、请求**

---

### 第二章Spring AOP

#### 概念

+ 查电表的例子
+ 事实上，服务模块更简介，以至于普通bean甚至不知道切面的存在。

![image-20200615121428986](C:\Users\wangning\Desktop\面试题\二、github面试题\image\Spring篇\Spring切面.png)asd

切面概念之前：切面是什么？何时切？切何处？

+ Aspect(切面)：一个关注点的模块化，就比较笼统的一个概念，关注点可能横切多个对象。
+ advice(通知)：表示在一个连接点执行的具体的动作，比如after、before表明通知的具体动作，（**何时？是什么？**）其中分为5种通知类型
  + 前置通知：在目标方法被调用前调用通知功能；
  + 后置通知：在目标方法被调用后调用通知功能，**此时并不关心方法的输出是什么**；
  + 返回通知：在目标方法被调用执行成功之后调用通知功能；
  + 异常通知：在目标方法抛出异常后调用通知；
  + 环绕通知：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。
+ Join Point(连接点)：在程序执行过程中某个特定的点，一个连接点总是代表一个方法的执行。
+ Pointcut(切点)：通过一个表达式去表明我所定义的通知在那个地点具体执行。
+ 引入(IntroductionIntroduction)：允许我们向现有的类添加新方法属性。
+ 织入(Weaving)：把切面应用到目标对象来创建新的代理对象的过程。有三个地方可以进行织入
  + 编译器：切面在目标类编译时被织入。需要特殊编译器，如AspectJ的植入编译器就是以改方式织入切面。
  + 类加载器：切面在目标类加载到JVM时被织入。该方式需要特殊的类加载器（ClassLoader）,它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5的加载时织入就支持该方式织入切面。
  + 运行期：切面在运行的狗哥时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态的创建一个代理对象。SpringAOP就是以该方式织入切面。

---

#### Spring对AOP的支持

+ Spring提供了4种类型的AOP支持
  + 基于代理的经典Spring AOP；
  + 纯POJO切面；
  + @AspectJ注解驱动的切面；
  + 注入式AspectJ切面（适用于Spring各版本）

前三种都是Spring AOP实体的变体，Spring AOP构建在动态代理基础之上，因此，Spring对AOP的支持局限于方法拦截。

+ `经典`的AOP编程模型并不是很好用；现代Spring提供了更简介和干净的面向切面编程方式。引入简单的**声明式AOP**和**基于注解的AOP**之后，Spring经典的AOP就明显看起来非常笨重和过于复杂。直接使用**ProxyFactory Bean**会让人厌烦。

##### **1.编写切点**

```java
package concert;
public interface Performance {
	public void perform();   
}
```

+ Spring借助AspectJ提供了多种指示器

```xml
<!--方式1 当perform()方法执行时触发通知的调用 -->
execution(* concert.Performance.perform(..))

<!--方式2 当perform()方法执行时触发通知的调用，并且在`concert`包下-->
execution(* concert.Performance.perform(..)) && within(concert.*)

<!--方式2 当perform()方法执行时触发通知的调用，并且在bean名称为'woodstock'中 -->
execution(* concert.Performance.perform(..)) and bean('woodstock')

<!--方式2 当perform()方法执行时触发通知的调用，并且在名称为 非 'woodstock' 的所有bean中 -->
execution(* concert.Performance.perform(..)) and !bean('woodstock')
```

##### 2.定义切面

```java
package concert;

import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class Aduience {
	
	@Pointcut("execution( ** concert.Performance.perform(..))")
	public void performance() {}
    
	@Before("performance()")
	public void silenceCellPhones() {	// 表演之前
		System.out.println("1.手机静音===========");
	}
	
	@Before("performance()")
	public void takeSeats() {	// 表演之前
		System.out.println("2.观众坐下===========");
	}
	
	@AfterReturning("execution( ** concert.Performance.perform(..))")
	public void applause() {	// 表演之前
		System.out.println("3.观众退场===========");
	}
	
	@AfterThrowing("execution( ** concert.Performance.perform(..))")
	public void demandRefund() {	// 表演之前
		System.out.println("表演失败============");
	}
}
```

+ 目前切面还不会生效，注解还不会被解析，也不会创建将其转换为切面的代理。（**注解方式启动**），使用`@EnableAspectJAutoProxy`注解

```java
@Configuration
@ComponentScan
@EnableAspectJAutoProxy // 启动自动代理
public class ConcertConfig {
	@Bean
	public Aduience aduience() {
		return new Aduience();
	}
	
}
```

+ xml方式启动

```xml
<!-- 气动阀AspectJ自动代理 -->
<aop:aspectj-autoproxy/>

<bean class="com.test.aop/Aduience" /> <!-- bean声明 -->
```

| 注解            | 通知                                     |
| --------------- | ---------------------------------------- |
| @After          | 通知方法会在目标方法返回或抛出异常后调用 |
| @AfterReturning | 通知方法会在目标方法返回后调用           |
| @AfterThrowing  | 通知方法会在目标方法抛出异常后调用       |
| @Around         | 通知方法会将目标方法封装起来             |
| @Before         | 通知方法会在目标方法执行前被调用         |
| @Pointcut       | 重新定义切点                             |

##### 3.环绕通知

+ 环绕通知比较强大，可以将你所编写的逻辑，将被通知的目标方法**包装起来**，相当于在一个通知方法中**同时编写前置通知和后置通知**。 使用`@Around`注解
+ 环绕通知接收的是**ProceedingJoinPoint**作为参数，并调用`proceed()`方法，其中`proceed()`方法是可以多次调用的，可用于失败重试

```java
@Around("performance()")
public	void watchPerformance(ProceedingJoinPoint jp) {

    try {
		System.out.println("1.手机静音===========");
		System.out.println("2.观众坐下===========");
        jp.proceed();
		System.out.println("3.观众退场===========");
    } catch (Throwable e) {
        System.out.println("表演失败");
    }
}
```

+ **带参数通知**

```java
@Pointcut("execution( ** com.test.aop.Performance.performTrack(int)) && args(trackCount)")
public void performTrack(int trackCount) {}

@Before("performTrack(trackCount)")
public	void watchPerformanceTrack(int trackCount) {
    try {
        System.out.println("环绕通知1xxxxxxxxxxxxx");
        System.out.println(trackCount);
        System.out.println("环绕后置通知3xxxxxxxxxxxxxxxx");
    } catch (Throwable e) {
        System.out.println("异常环绕通知");
    }
}

```

##### 4.通过注解引入新功能

+ 我们并没有为对象增加心的方法，但是已经为对象用有的方法添加了新功能。

```java
package com.test.aop;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.DeclareParents;

@Aspect
public class EncoreableIntroducer {

	@DeclareParents( value = "com.test.aop.Performance+",defaultImpl = DefalutEncoreable.class)
	public static Encoreable encoreable;
}
```

**`@DeclareParents`注解由三部分组成：**

+ value：哪种类型的bean要引入该接口，本例中是`Performance`的类型。后面的 **`+` **表示`Performace`类的子类，而不是类本身
+ defaultImpl：属性指定了为引入功能提供实现的类。这里用`DefalutEncoreable`实现类
+ @DeclareParents：注解所标注的静态属性指明了要引入了接口。我们所引入的是Encoreable接口

##### 5.在XML中声明切面

```xml
<bean id="aduience2" class="com.test.aop.Aduience2" />
<aop:config>
	<!-- 声明切点 -->
    <aop:pointcut expression="execution(** com.test.aop.Performance2.perform(..))" id="perform"/>
    <!-- 切面 -->
    <aop:aspect ref="aduience2">
        <!-- 前置通知 -->
        <aop:before method="silenceCellPhones"
                    pointcut="execution(** com.test.aop.Performance2.perform(..))" />

        <aop:before method="takeSeats"
                    pointcut="execution(** com.test.aop.Performance2.perform(..))" />

        <aop:after method="applauseActuor"
                   pointcut-ref="perform" />
        <aop:after method="applause"
                   pointcut-ref="perform" />
		<!-- 环绕通知 -->
        <aop:around method="watchPerformance" pointcut-ref="perform" />
    </aop:aspect>
</aop:config>
```

```java
package com.test.aop;
// 普通的javaBean,被声明为切面
public class Aduience2 {

	public void performance() {}
	
	public void performTrack(int trackCount) {}
	
	public void silenceCellPhones() {	// 表演之前
		System.out.println("1.手机静音===========2");
	}
	
	public void takeSeats() {	// 表演之前
		System.out.println("2.观众坐下===========2");
	}
	
	public void applauseActuor() {	// 表演之前
		System.out.println("3.演员退场===========2");
	}
	
	public void applause() {	// 表演之前
		System.out.println("4.观众退场===========2");
	}
	
	public void demandRefund() {	// 表演之前
		System.out.println("表演失败============2");
	}
    
	//环绕通知
	public	void watchPerformance(ProceedingJoinPoint jp) {
		
		try {
			System.out.println("环绕通知1");
			System.out.println("环绕通知2");
			jp.proceed();
			System.out.println("环绕后置通知3");
		} catch (Throwable e) {
			System.out.println("异常环绕通知");
		}
	}
	
}
```

---

#### AspectJ切面

+ 虽然Spring AOP能满足许多应用的切面需求，但相比AspectJ，还是比较弱的方案。AspectJ提供了Spring AOP所不能支持的许多类型的切点
+ 例如：对象在创建时，构造器切点就非常方便，然而Spring基于代理的AOP无法把通知应用于对象的创建过程
+ AspectJ和Spring是相互独立的，但我们可以借助Spring的依赖注入把bean装配进AspectJ切面中

---

### 第三章Spring Web

![](F:\GitHub\learn\image\Spring篇\SpringMVC.png)

#### DispatcherServlet

+ `DispatcherServlet`和`ContextLoaderListener`(Servlet监听器)的关系

##### 两个应用的上下文直接的故事

+ 当DispatcherServlet启动的时候，会创建Spring应用上下文，并加载配置文件或配置类中所声明的bean。但在SPring Web应用中，通常还会有另外一个应用上下文。
+ 另外的一个应用上下文是由`ContextLoaderListener`创建的
+ **我们希望`DispatcherServlet`加载包涵Web组件的bean，如控制器、视图解析器以及处理器映射，而ContextLoaderListener要加载应用中的其他bean，这些bean通常是驱动应用后端的中间层和数据层组件**

+ `AbstractAnnotationConfigDispatcherServletInitializer`是对传统`web.xml`的替代方案，只支持Servlet3.0以后的规范。

使用注解

+ `@EnableWebMvc：启动Spring MVC





### 第四章Spring JDBC

+ 





### 第十章SpringBoot

#### 1.helloworld

##### 1.父项目

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>springboot</artifactId>
    <version>1.0-SNAPSHOT</version>


    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

##### 2.启动器

```java
@SpringBootApplication
public class HelloWorld {

    public static void main(String[] args) {
        SpringApplication.run(HelloWorld.class, args);
    }
}
```

##### 3.Controller

```java
@Controller
public class HelloController {

    @ResponseBody
    @GetMapping("/hello")
    public String hello(){
        return "hello world!";
    }
}

```

#### 2.主启动类、程序入口

> `@SpringBootApplication`：在配置类下`@SpringBootConfiguration`标识出注解

>`@EnableAutoConfiguration`：开启自动配置功能，自动配置才生效
>
>> ```java
>> @AutoConfigurationPackage // 自动配置包
>> @Import(AutoConfigurationImportSelector.class)  // AutoConfigurationImportSelector组件选择器
>> public @interface EnableAutoConfiguration {
>> ```
>
>`@ConfigurationProperties`：加载配置文件
>
>`@Value`：从配置文件里注入值
>
>`@ConditionalOnMissingBean`：容器中必须没有该组件
>
>

```yaml
#开启SpringBoot的debug模式
debug: true 
```



#### 自动配置

> org\springframework\boot\spring-boot-autoconfigure\2.2.6.RELEASE\spring-boot-autoconfigure-2.2.6.RELEASE.jar!\org\springframework\boot\autoconfigure

![](F:\GitHub\learn\image\Spring篇\springBoot.png)

自动加到容器中

+ 每一个类自动配置类，自动配置到容器中

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.ElasticsearchRestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
org.springframework.boot.autoconfigure.r2dbc.R2dbcAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketRequesterAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketServerAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketStrategiesAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.rsocket.RSocketSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.saml2.Saml2RelyingPartyAutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
```

+ 以**HttpEncodingAutoConfiguration**为例解释：

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(ServerProperties.class)
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnClass(CharacterEncodingFilter.class)
@ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)
public class HttpEncodingAutoConfiguration {
```

> 由上面所有`@Conditional`的配置类判断**`HttpEncodingAutoConfiguration`**类是否生效，条件判断不通过，则忽略该配置类