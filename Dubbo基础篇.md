### 三、配置

| 标签                   | 用途         | 解释                                                         |
| ---------------------- | ------------ | ------------------------------------------------------------ |
| `<dubbo:service/>`     | 服务配置     | 用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心 |
| `<dubbo:reference/>`   | 引用配置     | 用于创建一个远程服务代理，一个引用可以指向多个注册中心       |
| `<dubbo:protocol/>`    | 协议配置     | 用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受 |
| `<dubbo:application/>` | 应用配置     | 用于配置当前应用信息，不管该应用是提供者还是消费者           |
| `<dubbo:module/>`      | 模块配置     | 用于配置当前模块信息，可选                                   |
| `<dubbo:registry/>`    | 注册中心配置 | 用于配置连接注册中心相关信息                                 |
| `<dubbo:monitor/>`     | 监控中心配置 | 用于配置连接监控中心相关信息，可选                           |
| `<dubbo:provider/>`    | 提供方配置   | 当 ProtocolConfig 和 ServiceConfig 某属性没有配置时，采用此缺省值，可选 |
| `<dubbo:consumer/>`    | 消费方配置   | 当 ReferenceConfig 某属性没有配置时，采用此缺省值，可选      |
| `<dubbo:method/>`      | 方法配置     | 用于 ServiceConfig 和 ReferenceConfig 指定方法级的配置信息   |
| `<dubbo:argument/>`    | 参数配置     | 用于指定方法参数配置                                         |

#### 不同粒度配置的覆盖关系

以 timeout 为例，下图显示了配置的查找顺序，其它 retries, loadbalance, actives 等类似：

- 方法级优先，接口级次之，全局配置再次之。
- 如果级别一样，则消费方优先，提供方次之。

其中，服务提供方配置，通过 URL 经由注册中心传递给消费方。

![dubbo-config-override](F:\GitHub\learn\image\Dubbo基础\dubbo-config-override.jpg)

建议由服务提供方设置超时，因为一个方法需要执行多长时间，服务提供方更清楚，如果一个消费方同时引用多个服务，就不需要关心每个服务的超时设置

理论上 ReferenceConfig 中除了`interface`这一项，其他所有配置项都可以缺省不配置，框架会自动使用ConsumerConfig，ServiceConfig, ProviderConfig等提供的缺省配置。

#### xml配置

##### provider.xml实例

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-provider"/>
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <dubbo:protocol name="dubbo" port="20890"/>
    <bean id="demoService" class="org.apache.dubbo.samples.basic.impl.DemoServiceImpl"/>
    <dubbo:service interface="org.apache.dubbo.samples.basic.api.DemoService" ref="demoService"/>
</beans>
```

##### consumer.xml示例

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-consumer"/>
    <dubbo:registry group="aaa" address="zookeeper://127.0.0.1:2181"/>
    <dubbo:reference id="demoService" check="false" interface="org.apache.dubbo.samples.basic.api.DemoService"/>
</beans>
```

### 属性配置

如果你的应用足够简单，例如，不需要多注册中心或多协议，并且需要在spring容器中共享配置，那么，我们可以直接使用 `dubbo.properties`作为默认配置。

Dubbo可以自动加载classpath根目录下的dubbo.properties，但是你同样可以使用JVM参数来指定路径：`-Ddubbo.properties.file=xxx.properties`。

##### 重写与优先级

![properties-override](F:\GitHub\learn\image\Dubbo基础\dubbo-properties-override.jpg)



### 四、Dubbo协议

+ dubbo原有9大协议，但在2.7.5 版本新增了gRPC协议。总共**10**大协议。

#### 1.dubbo://

+ 缺省（默认）协议，采用单一长连接和NIO异步通讯，适用于小数据量大并发的服务调用。
  + 以及**消费者机器数**远大于**服务提供者机器数量**的情况。
  + 相反，该协议不适合传大数据量的服务，如文件、视频等，除非请求量很低。

​	![dubbo-protocol.jpg](F:\GitHub\learn\image\Dubbo基础\dubbo-protocol.jpg)

- Transporter: mina, netty, grizzy
- Serialization: dubbo, hessian2, java, json
- Dispatcher: all, direct, message, execution, connection
- ThreadPool: fixed, cached

##### 特性

+ 连接数量：单连接
+ 连接方式：长连接
+ 传输协议：TCP
+ 传输方式：NIO异步传输
+ 序列化：Hessian二进制序列化
+ 适用范围：
  + 传输参数数据包较小（100k）
  + 消费者比提供者多，单一消费者无法压满提供者
  + 尽量不要用dubbo协议传输大文件或超大字符串
+ 适用场景：常规远程服务调用

##### 约束

+ 参数及返回值需要实现Serializable接口
+ 参数及返回值不能自定义实现 `List`, `Map`, `Number`, `Date`, `Calendar` 等接口，只能用 JDK 自带的实现，因为 hessian 会做特殊处理，自定义实现类中的属性值都会丢失。
+ Hessian 序列化，只传成员属性值和值的类型，不传方法或静态变量，兼容情况 。

+ 接口增加方法，对客户端无影响，如果该方法不是客户端需要的，客户端不需要重新部署。输入参数和结果集中增加属性，对客户端无影响，如果客户端并不需要新属性，不用重新部署。

+ 输入参数和结果集属性名变化，对客户端序列化无影响，但是如果客户端不重新部署，不管输入还是输出，属性名变化的属性值是获取不到的。
+ **总之**：服务器端和客户端对领域对象并不需要完全一致，而是按照最大匹配原则。

##### 配置

配置协议：

```xml
<dubbo:protocol name="dubbo" port="20880" />
```

设置默认协议：

```xml
<dubbo:provider protocol="dubbo" />
```

设置服务协议：

```xml
<dubbo:service protocol="dubbo" />
```

多端口：

```xml
<dubbo:protocol id="dubbo1" name="dubbo" port="20880" />
<dubbo:protocol id="dubbo2" name="dubbo" port="20881" />
```

配置协议选项：

```xml
<dubbo:protocol name="dubbo" port="9090" server="netty" client="netty" 
                codec="dubbo" serialization="hessian2" charset="UTF-8" threadpool="fixed" 
                threads="100" queues="0" iothreads="9" buffer="8192" accepts="1000" payload="8388608" />
```

多连接配置：

Dubbo 协议缺省每服务每提供者每消费者使用单一长连接，如果数据量较大，可以使用多个连接。

```xml
<dubbo:service connections="1"/>
<dubbo:reference connections="1"/>
```

- `<dubbo:service connections="0">` 或 `<dubbo:reference connections="0">` 表示该服务使用 JVM 共享长连接。**缺省**
- `<dubbo:service connections="1">` 或 `<dubbo:reference connections="1">` 表示该服务使用独立长连接。
- `<dubbo:service connections="2">` 或`<dubbo:reference connections="2">` 表示该服务使用独立两条长连接。

为防止被大量连接撑挂，可在服务提供放限制最大接收连接数，以实现服务提供方的自我保护。

```xml
<dubbo:protocol name="dubbo" accepts="1000" />
```

`dubbo.properties` 配置：

```properties
dubbo.service.protocol=dubbo
```

##### 问题

###### 1.为什么要消费者比提供者个数多?

+ 因 dubbo 协议采用单一长连接，根据测试经验数据每条连接最多只能压满 7MByte(不同的环境可能不一样，供参考)，理论上 1 个服务提供者需要 20 个服务消费者才能压满网卡。

###### 2.为什么不能传大包?

+ 因 dubbo 协议采用单一长连接，如果每次请求的数据包大小为 500KByte，因 dubbo 协议采用单一长连接，如果每次请求的数据包大小为 500KByte，7MByte / 500KByte = 14。如果能接受，可以考虑使用，否则网络将成为瓶颈。

###### 3.为什么采用异步单一长连接?

+ 因为服务的现状大都是服务提供者少，通常只有几台机器，而服务的消费者多，可能整个网站都在访问该服务，比如 Morgan 的提供者只有 6 台提供者，却有上百台消费者，每天有 1.5 亿次调用，如果采用常规的 hessian 服务，服务提供者很容易就被压跨，通过单一连接，保证单一消费者不会压死提供者，长连接，减少连接握手验证等，并使用异步 IO，复用线程池，防止 C10K 问题。

---

#### 2.rmi://

+ RMI（Remote Method Invoke 远程方法调用）
  + RMI 协议采用 JDK 标准的 `java.rmi.*` 实现，采用阻塞式短连接和 JDK 标准序列化方式。

##### 特性

- 连接个数：多连接
- 连接方式：短连接
- 传输协议：TCP
- 传输方式：同步传输
- 序列化：Java 标准二进制序列化
- 适用范围：传入传出参数数据包大小混合，**消费者与提供者个数差不多**，可传文件。
- 适用场景：常规远程服务方法调用，与原生RMI服务互操作

##### 约束

- 参数及返回值需实现 `Serializable` 接口
- dubbo 配置中的超时时间对 RMI 无效，需使用 java 启动参数设置：`-Dsun.rmi.transport.tcp.responseTimeout=3000`，参见下面的 RMI 配置

**dubbo.properties 配置**

```properties
dubbo.service.protocol=rmi
```

**RMI配置**

```
java -Dsun.rmi.transport.tcp.responseTimeout=3000
```

##### 接口

如果服务接口继承了 `java.rmi.Remote` 接口，可以和原生 RMI 互操作，即：

- **提供者用 Dubbo 的 RMI 协议暴露服务，消费者直接用标准 RMI 接口调用**
- **或者提供方用标准 RMI 暴露服务，消费方用 Dubbo 的 RMI 协议调用。**

如果服务接口没有继承 `java.rmi.Remote` 接口：

- 缺省 Dubbo 将自动生成一个 `com.xxx.XxxService$Remote` 的接口，并继承 `java.rmi.Remote` 接口，并以此接口暴露服务，
- 但如果设置了 `<dubbo:protocol name="rmi" codec="spring" />`，将不生成 `$Remote` 接口，而使用 Spring 的 `RmiInvocationHandler` 接口暴露服务，和 Spring 兼容。

##### 配置

定义 RMI 协议：

```xml
<dubbo:protocol name="rmi" port="1099" />
```

设置默认协议：

```xml
<dubbo:provider protocol="rmi" />
```

设置服务协议：

```xml
<dubbo:service protocol="rmi" />
```

多端口：

```xml
<dubbo:protocol id="rmi1" name="rmi" port="1099" />
<dubbo:protocol id="rmi2" name="rmi" port="2099" />
 
<dubbo:service protocol="rmi1" />
```

Spring 兼容性：

```xml
<dubbo:protocol name="rmi" codec="spring" />
```

---

#### 3.hessian://

Hessian 协议用于集成 Hessian 的服务，Hessian 底层采用 Http 通讯，采用 Servlet 暴露服务，Dubbo 缺省内嵌 Jetty 作为服务器实现。

Dubbo 的 Hessian 协议可以和原生 Hessian 服务互操作，即：

- 提供者用 Dubbo 的 Hessian 协议暴露服务，消费者直接用标准 Hessian 接口调用
- 或者提供方用标准 Hessian 暴露服务，消费方用 Dubbo 的 Hessian 协议调用。

##### 特性

- 连接个数：多连接
- 连接方式：短连接
- 传输协议：HTTP
- 传输方式：同步传输
- 序列化：Hessian二进制序列化
- 适用范围：传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件。
- 适用场景：页面传输，文件传输，或与原生hessian服务互操作

##### 依赖

```xml
<dependency>
    <groupId>com.caucho</groupId>
    <artifactId>hessian</artifactId>
    <version>4.0.7</version>
</dependency>
```

##### 约束

- 参数及返回值需实现 `Serializable` 接口
- 参数及返回值不能自定义实现 `List`, `Map`, `Number`, `Date`, `Calendar` 等接口，只能用 JDK 自带的实现，因为 hessian 会做特殊处理，自定义实现类中的属性值都会丢失。

##### 配置

定义 hessian 协议：

```xml
<dubbo:protocol name="hessian" port="8080" server="jetty" />
```

设置默认协议：

```xml
<dubbo:provider protocol="hessian" />
```

设置 service 协议：

```xml
<dubbo:service protocol="hessian" />
```

多端口：

```xml
<dubbo:protocol id="hessian1" name="hessian" port="8080" />
<dubbo:protocol id="hessian2" name="hessian" port="8081" />
```

直连：

```xml
<dubbo:reference id="helloService" interface="HelloWorld" url="hessian://10.20.153.10:8080/helloWorld
```

---

#### 4.http://

+ 基于 HTTP 表单的远程调用协议，采用 Spring 的 HttpInvoker 实现。

##### 特性

- 连接个数：多连接
- 连接方式：短连接
- 传输协议：HTTP
- 传输方式：同步传输
- 序列化：表单序列化
- 适用范围：
  - 传入传出参数数据包大小混合，**提供者比消费者个数多**
  - 可用浏览器查看，可用表单或URL传入参数
  - 暂不支持传文件。
- 适用场景：需同时给应用程序和浏览器 JS 使用的服务。

##### 约束

- 参数及返回值需符合 Bean 规范

##### 配置

配置协议：

```xml
<dubbo:protocol name="http" port="8080" />
```

配置 Jetty Server (默认)：

```xml
<dubbo:protocol ... server="jetty" />
```

配置 Servlet Bridge Server (推荐使用)：

```xml
<dubbo:protocol ... server="servlet" />
```

配置 DispatcherServlet：

```xml
<servlet>
         <servlet-name>dubbo</servlet-name>
         <servlet-class>org.apache.dubbo.remoting.http.servlet.DispatcherServlet</servlet-class>
         <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
         <servlet-name>dubbo</servlet-name>
         <url-pattern>/*</url-pattern>
</servlet-mapping>
```

注意，如果使用 servlet 派发请求：

- 协议的端口 `<dubbo:protocol port="8080" />` 必须与 servlet 容器的端口相同，
- 协议的上下文路径 `<dubbo:protocol contextpath="foo" />` 必须与 servlet 应用的上下文路径相同。

---

#### 5.webservice://

基于 WebService 的远程调用协议，基于 Apache CXF 的 `frontend-simple` 和 `transports-http` 实现 [[2\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/webservice.html#fn2)。

可以和原生 WebService 服务互操作，即：

- **提供者用 Dubbo 的 WebService 协议暴露服务，消费者直接用标准 WebService 接口调用**，
- 或者提供方用标准 WebService 暴露服务，消费方用 Dubbo 的 WebService 协议调用。

##### 依赖

```xml
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-frontend-simple</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-transports-http</artifactId>
    <version>2.6.1</version>
</dependency>
```

##### 特性

- 连接个数：多连接
- 连接方式：短连接
- 传输协议：HTTP
- 传输方式：同步传输
- 序列化：SOAP 文本序列化
- 适用场景：系统集成，跨语言调用

##### 约束

- 参数及返回值需实现 `Serializable` 接口
- 参数尽量使用基本类型和 POJO

##### 配置

配置协议：

```xml
<dubbo:protocol name="webservice" port="8080" server="jetty" />
```

配置默认协议：

```xml
<dubbo:provider protocol="webservice" />
```

配置服务协议：

```xml
<dubbo:service protocol="webservice" />
```

多端口：

```xml
<dubbo:protocol id="webservice1" name="webservice" port="8080" />
<dubbo:protocol id="webservice2" name="webservice" port="8081" />
```

直连：

```xml
<dubbo:reference id="helloService" interface="HelloWorld" url="webservice://10.20.153.10:8080/com.foo.HelloWorld" />
```

WSDL：

```
http://10.20.153.10:8080/com.foo.HelloWorld?wsdl
```

Jetty Server (默认)：

```xml
<dubbo:protocol ... server="jetty" />
```

Servlet Bridge Server (推荐)：

```xml
<dubbo:protocol ... server="servlet" />
```

配置 DispatcherServlet：

```xml
<servlet>
         <servlet-name>dubbo</servlet-name>
         <servlet-class>org.apache.dubbo.remoting.http.servlet.DispatcherServlet</servlet-class>
         <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
         <servlet-name>dubbo</servlet-name>
         <url-pattern>/*</url-pattern>
</servlet-mapping>
```

注意，如果使用 servlet 派发请求：

- 协议的端口 `<dubbo:protocol port="8080" />` 必须与 servlet 容器的端口相同，
- 协议的上下文路径 `<dubbo:protocol contextpath="foo" />` 必须与 servlet 应用的上下文路径相同。

---

#### 6.thrift://

当前 dubbo 支持 [[1\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/thrift.html#fn1)的 thrift 协议是对 thrift 原生协议 [[2\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/thrift.html#fn2) 的扩展，在原生协议的基础上添加了一些额外的头信息，比如 service name，magic number 等。

使用 dubbo thrift 协议同样需要使用 thrift 的 idl compiler 编译生成相应的 java 代码，后续版本中会在这方面做一些增强。

##### 依赖

```xml
<dependency>
    <groupId>org.apache.thrift</groupId>
    <artifactId>libthrift</artifactId>
    <version>0.8.0</version>
</dependency>
```

##### 配置

所有服务共用一个端口 [[3\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/thrift.html#fn3)：

```xml
<dubbo:protocol name="thrift" port="3030" />
```

##### 使用

可以参考 [dubbo 项目中的示例代码](https://github.com/apache/dubbo/tree/master/dubbo-rpc/dubbo-rpc-thrift/src/test/java/org/apache/dubbo/rpc/protocol/thrift)

##### 常见问题

- Thrift 不支持 null 值，即：不能在协议中传递 null 值

---

#### 7.memcached://

+ 基于 memcached [[1\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/memcached.html#fn1) 实现的 RPC 协议

##### 注册 memcached 服务的地址

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("memcached://10.20.153.11/com.foo.BarService?category=providers&dynamic=false&application=foo&group=member&loadbalance=consistenthash"));
```

##### 在客户端引用

在客户端使用 [[3\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/memcached.html#fn3)：

```xml
<dubbo:reference id="cache" interface="java.util.Map" group="member" />
```

或者，点对点直连：

```xml
<dubbo:reference id="cache" interface="java.util.Map" url="memcached://10.20.153.10:11211" />
```

也可以使用自定义接口：

```xml
<dubbo:reference id="cache" interface="com.foo.CacheService" url="memcached://10.20.153.10:11211" />
```

方法名建议和 memcached 的标准方法名相同，即：get(key), set(key, value), delete(key)。

如果方法名和 memcached 的标准方法名不相同，则需要配置映射关系 [[4\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/memcached.html#fn4)：

```xml
<dubbo:reference id="cache" interface="com.foo.CacheService" url="memcached://10.20.153.10:11211" p:set="putFoo" p:get="getFoo" p:delete="removeFoo" />
```

------

1. [Memcached](http://memcached.org/) 是一个高效的 KV 缓存服务器 [↩︎](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/memcached.html#fnref1)

---

#### 8.redis://

基于 Redis [[1\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/redis.html#fn1) 实现的 RPC 协议 [[2\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/redis.html#fn2)。

##### 注册 redis 服务的地址

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("redis://10.20.153.11/com.foo.BarService?category=providers&dynamic=false&application=foo&group=member&loadbalance=consistenthash"));
```

##### 在客户端引用

在客户端使用 ：

```xml
<dubbo:reference id="store" interface="java.util.Map" group="member" />
```

或者，点对点直连：

```xml
<dubbo:reference id="store" interface="java.util.Map" url="redis://10.20.153.10:6379" />
```

也可以使用自定义接口：

```xml
<dubbo:reference id="store" interface="com.foo.StoreService" url="redis://10.20.153.10:6379" />
```

方法名建议和 redis 的标准方法名相同，即：get(key), set(key, value), delete(key)。

如果方法名和 redis 的标准方法名不相同，则需要配置映射关系 [[4\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/redis.html#fn4)：

```xml
<dubbo:reference id="cache" interface="com.foo.CacheService" url="redis://10.20.153.10:6379" p:set="putFoo" p:get="getFoo" p:delete="removeFoo" />
```

---

#### 9.rest://



---

#### 10.grpc://

Dubbo 自 2.7.5 版本开始支持 gRPC 协议，对于计划使用 HTTP/2 通信，或者想利用 gRPC 带来的 Stream、反压、Reactive 编程等能力的开发者来说， 都可以考虑启用 gRPC 协议。

##### 支持 gRPC 的好处

- 为期望使用 gRPC 协议的用户带来服务治理能力，方便接入 Dubbo 体系
- 用户可以使用 Dubbo 风格的，基于接口的编程风格来定义和使用远程服务

##### 如何在 Dubbo 中使用 gRPC

大概需要以下步骤：

1. 使用 IDL 定义服务
2. 配置 compiler 插件，本地预编译
3. 配置暴露/引用 Dubbo 服务

---

### 五、注册中心

#### 1.zookeeper注册中心

![/user-guide/images/zookeeper.jpg](F:\GitHub\learn\image\Dubbo基础\zookeeper.jpg)

流程说明：

- 服务提供者启动时: 向 `/dubbo/com.foo.BarService/providers` 目录下写入自己的 URL 地址
- 服务消费者启动时: 订阅 `/dubbo/com.foo.BarService/providers` 目录下的提供者 URL 地址。并向 `/dubbo/com.foo.BarService/consumers` 目录下写入自己的 URL 地址
- 监控中心启动时: 订阅 `/dubbo/com.foo.BarService` 目录下的所有提供者和消费者 URL 地址。

支持以下功能：

- 当提供者出现断电等异常停机时，注册中心能自动删除提供者信息
- 当注册中心重启时，能自动恢复注册数据，以及订阅请求
- 当会话过期时，能自动恢复注册数据，以及订阅请求
- 当设置 `<dubbo:registry check="false" />` 时，记录失败注册和订阅请求，后台定时重试
- 可通过 `<dubbo:registry username="admin" password="1234" />` 设置 zookeeper 登录信息
- 可通过 `<dubbo:registry group="dubbo" />` 设置 zookeeper 的根节点，不配置将使用默认的根节点。
- 支持 `*` 号通配符 `<dubbo:reference group="*" version="*" />`，可订阅服务的所有分组和所有版本的提供者

#### 依赖

```xml
<dependency>
    <groupId>com.github.sgroschupf</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.1</version>
</dependency>
```

Zookeeper 单机配置:

```xml
<dubbo:registry address="zookeeper://10.20.153.10:2181" />
```

或：

```xml
<dubbo:registry protocol="zookeeper" address="10.20.153.10:2181" />
```

Zookeeper 集群配置：

```xml
<dubbo:registry address="zookeeper://10.20.153.10:2181?backup=10.20.153.11:2181,10.20.153.12:2181" />
```

或：

```xml
<dubbo:registry protocol="zookeeper" address="10.20.153.10:2181,10.20.153.11:2181,10.20.153.12:2181" />
```

同一 Zookeeper，分成多组注册中心:

```xml
<dubbo:registry id="chinaRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="china" />
<dubbo:registry id="intlRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="intl" />
```

---

### 六、负载均衡

+ LoadBalance 中文意思为负载均衡，他的职责是将网络请求或者其他形式的负载“均摊”到不同的机器上，避免其群众部分服务器压力过大，而另一些服务器比较空闲的情况。通过负载均衡，可以让每台服务器获取到适合自己处理能力的负载。在为高负载分流的同时，还可以避免资源浪费。一举两得。
+ dubbo提供了4种负载均衡实现，其做法都是继承自**AbstractLoadBalance**，该类实现了**LoadBalance**接口
  + 基于权重随机算法的 ：**RandomLoadBalance**
  + 基于最少活跃调用数算法的：**LeastActiveLoadBalance**
  + 基于Hash一致性的：**ConsistentHashLoadBalance**
  + 基于加权轮询算法的 ：**RoundRobinLoadBalance**

#### RandomLoadBalance

+ RandomLoadBalance 是加权随机算法的具体实现，它的算法思想很简单。假设我们有一组服务器 servers = [A, B, C]，他们对应的权重为 weights = [5, 3, 2]，权重总和为10。现在把这些权重值平铺在一维坐标值上，[0, 5) 区间属于服务器 A，[5, 8) 区间属于服务器 B，[8, 10) 区间属于服务器 C。比如，经过一万次选择后，服务器 A 被选中的次数大约为5000次，服务器 B 被选中的次数约为3000次，服务器 C 被选中的次数约为2000次。

####  LeastActiveLoadBalance

+   LeastActiveLoadBalance 翻译过来是最小活跃数负载均衡。活跃调用数越小，表明该服务提供者效率越高，单位时间内可处理更多的请求。此时应优先将请求分配给该服务提供者。在具体实现中，每个服务提供者对应一个活跃数 active。初始情况下，所有服务提供者活跃数均为0。每收到一个请求，活跃数加1，完成请求后则将活跃数减1。在服务运行一段时间后，性能好的服务提供者处理请求的速度更快，因此活跃数下降的也越快，此时这样的服务提供者能够优先获取到新的服务请求、这就是最小活跃数负载均衡算法的基本思想。

#### ConsistentHashLoadBalance

一致性 hash 算法由麻省理工学院的 Karger 及其合作者于1997年提出的，算法提出之初是用于大规模缓存系统的负载均衡。它的工作过程是这样的，首先根据 ip 或者其他的信息为缓存节点生成一个 hash，并将这个 hash 投射到 [0, 232 - 1] 的圆环上。当有查询或写入请求时，则为缓存项的 key 生成一个 hash 值。然后查找第一个大于或等于该 hash 值的缓存节点，并到这个节点中查询或写入缓存项。如果当前节点挂了，则在下一次查询或写入缓存时，为缓存项查找另一个大于其 hash 值的缓存节点即可。大致效果如下图所示，每个缓存节点在圆环上占据一个位置。如果缓存项的 key 的 hash 值小于缓存节点 hash 值，则到该缓存节点中存储或读取缓存项。比如下面绿色点对应的缓存项将会被存储到 cache-2 节点中。由于 cache-3 挂了，原本应该存到该节点中的缓存项最终会存储到 cache-4 节点中。

![img](F:\GitHub\learn\image\Dubbo基础\consistent-hash.jpg)

下面来看看一致性 hash 在 Dubbo 中的应用。我们把上图的缓存节点替换成 Dubbo 的服务提供者，于是得到了下图：

![img](F:\GitHub\learn\image\Dubbo基础\consistent-hash-invoker.jpg)

这里相同颜色的节点均属于同一个服务提供者，比如 Invoker1-1，Invoker1-2，……, Invoker1-160。这样做的目的是通过引入虚拟节点，让 Invoker 在圆环上分散开来，避免数据倾斜问题。所谓数据倾斜是指，由于节点不够分散，导致大量请求落到了同一个节点上，而其他节点只会接收到了少量请求的情况。比如：

![img](F:\GitHub\learn\image\Dubbo基础\consistent-hash-data-incline.jpg)

如上，由于 Invoker-1 和 Invoker-2 在圆环上分布不均，导致系统中75%的请求都会落到 Invoker-1 上，只有 25% 的请求会落到 Invoker-2 上。解决这个问题办法是引入虚拟节点，通过虚拟节点均衡各个节点的请求量。

---

#### RoundRobinLoadBalance

+ 所谓轮询是指将请求轮流分配给每台服务器。举个例子，我们有三台服务器 A、B、C。我们将第一个请求分配给服务器 A，第二个请求分配给服务器 B，第三个请求分配给服务器 C，第四个请求再次分配给服务器 A。这个过程就叫做轮询。轮询是一种无状态负载均衡算法，实现简单，适用于每台服务器性能相近的场景下。但现实情况下，我们并不能保证每台服务器性能均相近。如果我们将等量的请求分配给性能较差的服务器，这显然是不合理的。因此，这个时候我们需要对轮询过程进行加权，以调控每台服务器的负载。经过加权后，每台服务器能够得到的请求数比例，接近或等于他们的权重比。比如服务器 A、B、C 权重比为 5:2:1。那么在8次请求中，服务器 A 将收到其中的5次请求，服务器 B 会收到其中的2次请求，服务器 C 则收到其中的1次请求。