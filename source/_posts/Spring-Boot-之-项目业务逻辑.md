---
title: Spring Boot 之 项目业务逻辑
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-08-01 10:54:17
categories: 编程语言
tags: SpringBoot
---
业务逻辑
![](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=3655339541,1765017154&fm=26&gp=0.jpg)
<!--more-->
# 开发流程
1. Database Column
2. Model：模型定义，和数据库相匹配
3. DAO：数据读取
4. Service：服务包装
5. Controller：业务入口
6. Test

***

# 评论Comment
## DAO
1. 添加评论 addComment
2. 根据一个实体选出全部评论 selectCommentByEntity
3. 选一个实体下有多少评论

## Service
1. 添加评论
2. 得到评论数
3. 删除评论-controller调用Service，Service更新状态就可以了
 
## Controller
1. 增加评论 POST
questionId 是多少
content 是多少
评论内容也需要过滤一次( Html & 自己写的算法)
hostHolder！=null   说明你是登陆的
没登陆？：要不登陆、要不匿名





![](https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=2189742753,2078713309&fm=26&gp=0.jpg)