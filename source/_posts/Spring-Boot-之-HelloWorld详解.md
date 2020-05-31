---
title: Spring Boot 之 HelloWorld详解
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-07-28 23:49:43
categories: 框架
tags: SpringBoot
---
SpringBoot介绍~<暂时假装有>

![](https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=2911227163,188786552&fm=26&gp=0.jpg)
<!--more-->
# 配置
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.nowcoder</groupId>
	<artifactId>wenda</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>wenda</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.3.6.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
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
# Controller层
## 简单Hello输出
```
@Controller
public class IndexController {
@RequestMapping(path = {"/","/index"})
@ResponseBody
    public String index(){
        return "Hello";
    }
}
```
访问：
```
http://127.0.0.1:8080/index
http://127.0.0.1:8080/
```
输出：
```
Hello
```

## 参数解析输出
路径传递
```
@Controller
public class IndexController {
@RequestMapping(path = {"/","/index"})
@ResponseBody
    public String index(){
        return "Hello";
    }

    @RequestMapping(path = {"/profile/{userId}"})
    @ResponseBody
    public String profile(@PathVariable("userId")  int  userId ){
        return String.format("Profile Page of %d",userId);
    }
}
```
访问`http://127.0.0.1:8080/profile/2` ：
```
Profile Page of 2
```
## 参数解析输出例2
```
@RequestMapping(path = {"/profile/{groupId}/{userId}"})
@ResponseBody
public String profile(@PathVariable("userId")  int  userId ,
                      @PathVariable("groupId")  String  groupId ){
    return String.format("Profile Page of %s / %d",groupId,userId);
    }
```
访问`http://127.0.0.1:8080/profile/admin/10086` 
```
Profile Page of admin / 10086
```
## eg3请求参数传递
```
@RequestMapping(path = {"/profile/{groupId}/{userId}"})
@ResponseBody
public String profile(@PathVariable("userId")  int  userId ,
                      @PathVariable("groupId")  String  groupId ,
                      @RequestParam("type")  int  type ,
                      @RequestParam("key")  String  key ){
    return String.format("Profile Page of %s / %d , t:%d k:%s",groupId,userId,type,key);
}
```
访问`http://127.0.0.1:8080/profile/admin/10086?type=2&key=w`
```
Profile Page of admin / 10086 , t:2 k:w
```
# 模板Velocity使用
controller 
```
@RequestMapping(path = {"/vm"},method = {RequestMethod.GET})
public String template(Model model){
    model.addAttribute("value1","vvvv1");
    return "home";
}
```
指向home.html 渲染
```
<html>
<body>
<pre>
    #*
    你看不到我~~~~
    *#
    $!{value1}
    $!{value2} ## 如果不存在，强制为空
    ${value3}
</pre>
</body>
</html>
```
依赖
```
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-velocity</artifactId>
</dependency>
```
application.properties 配置
```
spring.velocity.suffix=.html
```
访问`http://127.0.0.1:8080/vm`
```
    vvvv1
         ${value3}
```

# 总结
1. @Controller 注释来定义说这是一个网页入口
2. @RequestMapping 指定访问地址or传入参数
3. 用@PathVariable和@RequestParam对传入参数做解析
4. 用@ResponseBody or 模板文件 返回结果

![](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=2796235416,419635037&fm=26&gp=0.jpg)