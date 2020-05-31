---
title: Java并发编程实战
categories:
  - 开发者手册
  - 编程语言
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2018-06-16 10:54:24
tags: 并发
---
基于《Java并发编程实战》，作者Brain等。


<!--more-->
# 任务执行
## 顺序执行
SingleThreadWebServer顺序处理他的任务：接收到达80端口的HTTP请求
```Java
class SingleThreadWebServer{
	public static void main(String [] args)throws IOException{
    ServerSocket socket = new ServerSocket(80);
    while(true){
    Socket connection = socket.accept();
    handleRequest(connection);
    }
    }
}
```

### 无限创建线程的缺点
1. 线程生命周期的开销
2. 资源消耗量、尤其是内存
3. 稳定性差


