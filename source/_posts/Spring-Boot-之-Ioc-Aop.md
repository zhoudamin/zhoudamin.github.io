---
title: Spring Boot 之 Ioc-Aop
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-07-29 00:36:35
categories: 框架
tags: SpringBoot
---

非常重要的特性！

![](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=3659295201,1870844406&fm=11&gp=0.jpg)
<!--more-->

# Ioc

依赖注入：
无需关注这些变量的初始化，只需要通过注解表示
## Service层服务层

```
@Service
public class WendaService {
    public String getMessage(int userId){
        return "Hello Message: " +String.valueOf(userId);
    }
}
```
## Controller层
原本是：
```
WendaService wendaService=new WendaService();
```
在用的地方依赖注入
```
@Autowired
WendaService wendaService;
```
完整code:
```
@Controller
public class IndexController {
    @Autowired
    WendaService wendaService;

    @RequestMapping(path = {"/setting" },method = {RequestMethod.GET})
    @ResponseBody
    public String setting(HttpSession httpSession){
        return "Setting OK. " + wendaService.getMessage(1);
    }
}
```
`http://127.0.0.1:8080/setting/`
```
Setting OK. Hello Message: 1
```


# Aop/Log

面向切面，所有业务都要处理的业务
好处：关注系统性能时，可以查看每个方法调用多少次，每个方法调用时间，额外注释，数据记录统计等都非常方便！

## 添加依赖
```
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
## aspect
```
@Component
@Aspect
public class LogAspect {
    private static final Logger logger= LoggerFactory.getLogger(LogAspect.class);
    @Before("execution(* com.nowcoder.controller.IndexController.*(..))")
    public void beforeMethod(){
        logger.info("before method");

    }

    @After("execution(* com.nowcoder.controller.IndexController.*(..))")
    public void afterMethod(){
        logger.info("after method");
    }
}
```
调用IndexController.class前后会执行切面代码
```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.3.6.RELEASE)

2017-07-29 09:00:17.625  INFO 5652 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring FrameworkServlet 'dispatcherServlet'
2017-07-29 09:00:17.625  INFO 5652 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization started
2017-07-29 09:00:17.635  INFO 5652 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization completed in 10 ms
2017-07-29 09:00:17.655  INFO 5652 --- [nio-8080-exec-1] com.nowcoder.aspect.LogAspect            : before method
2017-07-29 09:00:17.657  INFO 5652 --- [nio-8080-exec-1] com.nowcoder.aspect.LogAspect            : after method
```


![](https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=728688327,4195684297&fm=26&gp=0.jpg)