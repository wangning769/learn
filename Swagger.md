# Swagger

## swagger简介

+ Vue  + SpringBoot
  + 后端时代：前端只用管理静态页面；html=>后端。模板引擎 JSP => 后端主力
+ 前后端分离式时代
  + 后端：控制层，服务层、数据访问层
  + 前端：前端控制层、视图层

---

## SpringBoot集成Swagger

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.7.0</version>
</dependency>

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.7.0</version>
</dependency>
```

```java
@Configuration
@EnableSwagger2
public class Swagger2Config {

	// 配置 Swagger的Docket的 Bean实例
	@Bean
	public Docket api() {
		return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo());
				/**
				 * RequestHandlerSelectors.basePackage(basePackage)
				 * 基于某个包去扫描
				 */
	}

	private ApiInfo apiInfo() {
		return new ApiInfoBuilder().title("swagger-goods-api文档").description("swagger接入教程").build();
	}

}

```

