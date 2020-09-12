---
title: SpringBoot使用Swagger生成API文档
date: 2020-04-12 20:22:32
tags:
- springboot
category:
- 后端
---
## 简介
Swagger是一种REST APIs的简单但强大的表示方式，标准的，语言无关，这种表示方式不但人可读，而且机器可读。可以作为REST APIs的交互式文档，也可以作为REST APIs的形式化的接口描述，生成客户端和服务端的代码。
## 使用
### 引入依赖
```xml
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.2.2</version>
</dependency>
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.2.2</version>
</dependency>
```
### 创建配置类
```Java
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import springfox.documentation.builders.ApiInfoBuilder;
	import springfox.documentation.builders.PathSelectors;
	import springfox.documentation.builders.RequestHandlerSelectors;
	import springfox.documentation.service.ApiInfo;
	import springfox.documentation.spi.DocumentationType;
	import springfox.documentation.spring.web.plugins.Docket;
	import springfox.documentation.swagger2.annotations.EnableSwagger2;
	
	@Configuration
	@EnableSwagger2
	public class Swagger2Config {
	    @Bean
	    public Docket createRestApi(){
	        return new Docket(DocumentationType.SWAGGER_2)
	                .apiInfo(apiInfo())
	                .select()
	                //为当前包下controller生成API文档
	                .apis(RequestHandlerSelectors.basePackage("com.macro.mall.tiny.controller"))
	                .paths(PathSelectors.any())
	                .build();
	    }
	
	    private ApiInfo apiInfo() {
	        return new ApiInfoBuilder()
	                .title("SwaggerUI演示")
	                .description("myblog")
	                .contact("ezio")
	                .version("1.0")
	                .build();
	    }
	}
```

### 在Controller层中添加相应注解
- @Api 修饰Controller,在文档中显示相应tags
- @ApiImplicitParam 修饰请求参数，添加参数描述信息
- @ApiOperation 修饰请求的方法，生成方法对应的文档
```java
@Api(tags = "PostBlogController")
@RestController
@RequestMapping("api/blog/")
public class PostBlogController {
	@Autowired
	private PostBlogService postBlogService;	
	@ApiOperation("新增博客")
	@ApiImplicitParam(name = "postVO", value = "新增博客VO", required = true, dataType = "PostVO")
	@PostMapping("add")
	public ServerResponse blogAdd(@Valid @RequestBody PostVO postVO, BindingResult errors) {
		if (errors.hasErrors()) {
			//异常处理
			List<String> errorList = new ArrayList<>();
			errors.getAllErrors().stream().forEach(error -> errorList.add(error.getDefaultMessage()));
			return ServerResponse.createByErrorMessage(errorList.get(0));
		}
		return postBlogService.addPost(postVO);
	}
}	
```
	
		
	
运行项目，访问[http://localhost:8080/swagger-ui.html](http://localhost:8866/swagger-ui.html)即可