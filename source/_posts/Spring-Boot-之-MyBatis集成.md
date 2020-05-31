---
title: Spring Boot 之 MyBatis集成
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-07-29 11:40:30
categories: 框架
tags: SpringBoot
---
MyBatis是现在业界互联网流行的数据操作层框架

![](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=1154627446,327324937&fm=26&gp=0.jpg)

<!--more-->
# 流程
1. 用注解的方法
2. 先定义一个Dao，和数据库做交互
3. 注解上写@Mapper： Dao和数据库做一个Mapper
4. 把@Insert通过一个注解的方式写进去
5. 写SQL语句，语法有点不同，导进去
6. 增删改查

 

# Mysql创建数据库
## 创建表user
```
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(64) NOT NULL,
  `password` varchar(128) NOT NULL COMMENT '密码',
  `salt` varchar(32) not null default '',
  `head_url` varchar(256) not null default '',
  PRIMARY KEY (`id`),
  UNIQUE KEY `name_UNIQUE` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 COMMENT='这是用户表'
```
## 创建表question
```
CREATE TABLE `question` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL,
  `content` text,
  `user_id` int(11) NOT NULL,
  `created_date` datetime NOT NULL,
  `comment_count` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `date_index` (`created_date`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```
# 添加pom.xml依赖
```
<dependency>
<groupId>org.mybatis.spring.boot</groupId>
<artifactId>mybatis-spring-boot-starter</artifactId>
<version>1.1.1</version>
</dependency>

<dependency>
<groupId>mysql</groupId>
<artifactId>mysql-connector-java</artifactId>
<scope>runtime</scope>
</dependency>
```

# 添加application.properties配置
```
spring.datasource.url=jdbc:mysql://localhost:3306/wenda?useUnicode=true&characterEncoding=utf8&useSSL=false
spring.datasource.username=root
spring.datasource.password=1621
mybatis.config-location=classpath:mybatis-config.xml
```

# Dao层
先定义一个@Mapper
下面就可以写SQL语句了
未来扩展：独立数据语句

```
@Mapper
public interface UserDao {
    //以后数据库有变更，就只要改这里就可以了
    String TABLE_NAME=" user ";
    String INSERT_FIELDS=" name , password ,salt ,head_url ";
    String SELECT_FIELDS=" id,"+ INSERT_FIELDS ;

    @Insert({"insert into ", TABLE_NAME, "(", INSERT_FIELDS,
            ") values (#{name},#{password},#{salt},#{headUrl})"})
    int addUser(User user);
}
```

# User层

添加与读取
set与get
```
public class User {
	private int id;
	private String name;
	private String password;
	private String salt;
	private String headUrl;

	public User(){

	}
	public User(String name){
	this.name =name;
	this.password="";
	this.salt="";
	this.headUrl="";
	}
	public String getName(){return name;}
	public void setName(String name) {this.name=name;}
	public String getPassword() {return password;}
	public void setPassword(String password) {this.password=password;}
	public String getSalt() {return salt ;}
	public void setSalt(String salt ) {this.salt=salt;}
	public String getHeadUrl() {return headUrl;}
	public void setHeadUrl(String headUrl) {this.headUrl=headUrl;}
	public int  getId() {return id;}
	public void setId(int id) {this.id=id;}
}
```


# 测试
```
@RunWith( SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = WendaApplication.class)
@Sql("/init-schema.sql")
public class InitDatabaseTests {

	@Autowired
	UserDao userDao;

	@Test
	public void contextLoads() {
		Random random=new Random();

		for(int i=0;i<11;++i){
			User user= new User();
			user.setHeadUrl(String.format("http://images.nowcoder.com/head/%dt.png",random.nextInt(1000)));
			user.setName(String.format("USER%d",i));
			user.setPassword("");
			user.setSalt("");
			userDao.addUser(user);

		}
	}

}
```

# 数据库写入结果
```
id   name   password   salt      head_url     
1	USER0						http://images.nowcoder.com/head/165t.png
2	USER1						http://images.nowcoder.com/head/269t.png
3	USER2						http://images.nowcoder.com/head/961t.png
4	USER3						http://images.nowcoder.com/head/395t.png
5	USER4						http://images.nowcoder.com/head/13t.png
6	USER5						http://images.nowcoder.com/head/84t.png
7	USER6						http://images.nowcoder.com/head/179t.png
8	USER7						http://images.nowcoder.com/head/593t.png
9	USER8						http://images.nowcoder.com/head/77t.png
10	USER9						http://images.nowcoder.com/head/600t.png
11	USER10						http://images.nowcoder.com/head/497t.png
```
# 传入多个参数
```
@Mapper
public interface QuestionDao {
    String TABLE_NAME = " question ";
    String INSERT_FIELDS = " title, content, created_date, user_id, comment_count ";
    String SELECT_FIELDS = " id, " + INSERT_FIELDS;

    @Insert({"insert into ", TABLE_NAME, "(", INSERT_FIELDS,
            ") values (#{title},#{content},#{createdDate},#{userId},#{commentCount})"})
    int addQuestion(Question question);

    List<Question> selectLatestQuestions(@Param("userId") int userId, @Param("offset") int offset,
                                         @Param("limit") int limit);
}
```
QuestionDao.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.nowcoder.dao.QuestionDao">
    <sql id="table">question</sql>
    <sql id="selectFields">id, title, content, comment_count,created_date,user_id
    </sql>
    <select id="selectLatestQuestions" resultType="com.nowcoder.model.Question">
        SELECT
        <include refid="selectFields"/>
        FROM
        <include refid="table"/>

        <if test="userId != 0">
            WHERE user_id = #{userId}
        </if>
        ORDER BY id DESC
        LIMIT #{offset},#{limit}
    </select>
</mapper>
```
测试
```
Question question=new Question();
			question.setCommentCount(i);
			Date date=new Date();
			date.setTime(date.getTime()+1000*3600*i);
			question.setCreatedDate(date);
			question.setUserId(i+1);
			question.setTitle(String.format("TITLE{%d}",i));
			question.setContent(String.format("Balalalalalalalala Content %d" , i) );
			questionDao.addQuestion(question);
```
结果
```
1	TITLE{0}	Balalalalalalalala Content 0	1	2017-07-29 20:30:33	0
2	TITLE{1}	Balalalalalalalala Content 1	2	2017-07-29 21:30:33	1
3	TITLE{2}	Balalalalalalalala Content 2	3	2017-07-29 22:30:33	2
4	TITLE{3}	Balalalalalalalala Content 3	4	2017-07-29 23:30:33	3
5	TITLE{4}	Balalalalalalalala Content 4	5	2017-07-30 00:30:33	4
6	TITLE{5}	Balalalalalalalala Content 5	6	2017-07-30 01:30:33	5
7	TITLE{6}	Balalalalalalalala Content 6	7	2017-07-30 02:30:33	6
8	TITLE{7}	Balalalalalalalala Content 7	8	2017-07-30 03:30:33	7
9	TITLE{8}	Balalalalalalalala Content 8	9	2017-07-30 04:30:33	8
10	TITLE{9}	Balalalalalalalala Content 9	10	2017-07-30 05:30:33	9
11	TITLE{10}	Balalalalalalalala Content 10	11	2017-07-30 06:30:33	10
```
![](http://img3.imgtn.bdimg.com/it/u=1530922496,1402123453&fm=11&gp=0.jpg)