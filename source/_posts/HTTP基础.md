---
title: HTTP基础
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2018-04-25 18:18:20
categories: 理解计算机
tags: HTTP
---

## 服务器意图

> * GET：获取资源，用来请求已经被URI识别的资源
> * POST：传输实体主体
> * PUT：传输文件
> * HEAD：获得报文首部
> * DELETE：删除文件
> * OPTIONS：询问支持的方法

## 状态码

|         | 类别   |  原因短语  |
| --------   | -----:  | :----:  |
| 1XX        |   Informational(信息性状态码)      |    接收的请求正在处理    	 |
| 2XX        |   Success(成功状态码)     		  |    请求正常处理完毕          |
| 3XX        |   Redirection (重定向状态码)       |   需要进行附加操作以完成请求 |
| 4XX        |   Client Error(客户端错误状态码)   |   服务器无法处理请求         |
| 5XX        |   Server Error(服务器错误状态码)   |   服务器处理请求出错         |



