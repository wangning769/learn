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