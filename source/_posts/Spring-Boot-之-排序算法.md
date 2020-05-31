---
title: Spring-Boot-之-排序算法
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-08-06 09:49:25
categories: 编程语言
tags: SpringBoot
---

一些网站内容排序算法

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1501996227800&di=9b1181fa8d889a6da506cf1273ad6efd&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Flib%2FuploadImg%2F20140708%2F20140708090444_305.gif)
<!--more-->

# 前言-网页排序核心思想

1. 交互越好，排名越好
2. 时间越新，排名越好

***

# Hacker News

https://news.ycombinator.com/


$$
Score=(P-1)/(T+2)^G
$$
P：投票数，-1是把自己投的过滤掉
T：发布到现在的时间间隔，单位小时，+2防止除数太小
G：重力加速度，分值根据时间降低速率

![微信图片_20170806095556](E:\Program Files\DarminBlog\public\Picture\微信图片_20170806095556.png)

***

# Reddit

https://www.reddit.com/

t = (time of entry post) - (Dec 8, 2005)
x = upvotes - downvotes

y = {1 if x > 0,

​        0 if x = 0, 

​       -1 if x < 0)
z = {1 if x < 0, otherwise x}


$$
f(t,y,z)=log(z) + (y * t)/45000
$$
时间是最重要的权重，由于流量比较大，所以对于高赞文章有所优势，适合新闻类排序

***

# StackOverflow

http://stackoverflow.com

```java
(log(Qviews)*4) + ((Qanswers * Qscore)/5) + sum(Ascores)
--------------------------------------------------------
((QageInHours+1) - ((QageInHours - Qupdated)/2)) ^ 1.5
```

Qviews：问题浏览数，通过log来平滑
Qanswer：问题回答数，有回答的题目才是好问题
Qscore：问题赞踩差，赞的越多，问题越好
sum（Ascores）：回答赞踩差，回答的越多问题越好
QageInHours：题目发布时间差，时间越久排名越后
Qupdated：最新的回答时间，越新关注度越高

***

# IMDB

http://www.imdb.com/chart/top

加权排名(WR) = (v ÷(v+m)) ×R + (m ÷(v+m)) ×C

R = 某电影投票平均分

v = 有效投票人数

m = 最低投票人数，1250

C = 所有电影平均值

投票人数越多，越偏向于用户打分值，防止冷门电影小数人高分导致的高分

http://www.imdb.com/help/show_leaf?votestopfaq

***

# Marathon Match Rating System 

适合用于比赛中，互相排名。



https://community.topcoder.com/longcontest/?module=Static&d1=support&d2=ratings



![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1501996276177&di=05d24cbed6e6d7d46bfe8e80b84f2283&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Flib%2FuploadImg%2F20140708%2F20140708090444_59.gif)