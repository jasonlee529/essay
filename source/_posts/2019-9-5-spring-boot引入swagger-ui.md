---
title: spring-boot引入swagger-ui
date: 2019-09-05 17:01:54
categories:
- Java
- swagger
tags:
- Java
- SpringBoot
- swagger
---
# Swagger-ui Spring-boot集成方法

> 现在做的Java项目基本上都是前后端分离的,后端只提供api接口即可,和前端交互的时候都是用json,程序员都讨厌写文档,但是不写接口文档肯定要被前端开发喷死.

### Swagger-ui介绍
Swagger是个前后端协作的利器，解析代码里的注解生成JSON文件，通过Swagger UI生成网页版的接口文档，可以在上面做简单的接口调试 也可以自动生成文档给前端人员调试

### 页面：
{% asset_img swagger-ui-web.png swagger-ui %}
### 使用方式
#### 引入依赖：
``` pom.xml
<springfox.swagger2.version>2.8.0</pringfox.swagger2.version>


<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>${springfox.swagger2.version}</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>${springfox.swagger.ui.version}</version>
</dependency>
```
#### 配置
``` Java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                //包路径
                .apis(RequestHandlerSelectors.basePackage("cn.infisa.swagger")) 
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("测试系统") 
                .description("测试系统")
                .version("1.0")
                .build();
    }
}
```
#### 标记RestController
主要用到两个注解 __@API__ 和 __@ApiOperation__
@API 
用于注解类
@ApiOperation
用于注解方法

将上述注解加到需要生成接口文档的RestController和RequestMapping上，之后启动服务即可生成接口文档。
输入：http://localhost:8080/swagger-ui.html即可访问API列表。

###  使用
1. get接口
{% asset_img get.png swagger-ui %}

2. shiro get接口
{% asset_img shiro-get.png swagger-ui %}


### Q&A
#### 1. shiro的集成
1. 在shiro中放行swagger-ui的页面 

``` java
//开放swagger
chains.put("/swagger-resources/**", "anon");
chains.put("/webjars/**", "anon");
chains.put("/v2/**", "anon");
chains.put("/swagger-ui.html/**", "anon");
```

2. 请求中加入ctoken参数，方法上增加注解

``` Java
@ApiImplicitParams({
        @ApiImplicitParam(name = "ctoken", required = true, value = "a", paramType = "header"),
})
```
