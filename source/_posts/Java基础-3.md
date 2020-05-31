---
title: Java基础-3
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-09-11 10:46:51
categories: Java
tags: 基础
---

一些Java基本功的记录。

Java基础系列：

[Java基础-1](http://zhoudm.com/2017/08/07/Java%E5%9F%BA%E7%A1%80-1/)

[Java基础-2](http://zhoudm.com/2017/08/11/Java%E5%9F%BA%E7%A1%80-2/)

<!--more-->

# ArrayList与LinkedList的底层实现

ArrayList是List接口的一个实现类，特点是查询效率高，增删效率低，线程不安全

原因是ArrayList底层封装了一个数组，他是用数组实现的。

|      地址      |                | 数组空间 |
| :----------: | :------------: | :--: |
|     2000     | -------------> | a[0] |
|     2004     | -------------> | a[1] |
|     2008     | -------------> | a[2] |
|     ---      | -------------> | ...  |
| 2000+（n-1）*4 | -------------> | a[n] |

定义一个int[]数组，首地址是2000，int类型占4个字节，所以a[0]的首地址2000，a[1]就是2004

每次查询只要一个偏移量就可以了，所以查询效率高

增删效率低的原因：



|      地址      |                |  数组空间  |
| :----------: | :------------: | :----: |
|     2000     | -------------> |  a[0]  |
|     2004     | -------------> | (新增元素) |
|     2008     | -------------> |  a[1]  |
|     ---      | -------------> |  ...   |
| 2000+（n-1）*4 | -------------> |  a[n]  |

新增元素会引起后面的元素的移动，所以增删效率低。

* LinkedList

增删效率高，查询效率低

LinkedList底层采用双向循环链表实现的List，链表的存储特点是不挨着，它存储每个元素分为三段：上一项的地址，下一项的地址，元素的内容。

每个元素在内存中的排列像是随机的，得根据地址来找元素，所以很慢

增删很快是因为，删除一个元素，前后元素会自动连上，而且删除一个元素只影响前后元素，所以增删效率高。

---

# TCP三次握手，以及为什么不是两次或四次

* 三次握手过程：

第一次握手：客户端发送TCP包，置SYN标志位为1，将初始序号X，保存在包头的序列号（Seq）里。

第二次握手：服务端回应确认包，置SYN标志位为1，置ACK为X+1，将初始序列号Y，保存在包头的序列号里。

第三次握手：客户端对服务端的确认包进行确认，置SYN标志位为0，置ACK为Y+1，置序列号为Z。

* 如果只有2次

第二次握手后，服务端发送请求给客户端，服务端以为连接成功了，但是如果实际上客户端没收到的话，客户端会认为连接没有建立，服务端会对已建立的连接保存必要的资源，如果出现大量这种情况，服务端会崩溃。

* 如果是4次

无谓的第四次



---

# 二叉树深度、结点

​二叉树的第 i 层至多有 2^(i-1) 个结点；

深度为 k 的二叉树至多有 2^k - 1 ​个结点；

对任何一棵二叉树 T，如果其终端结点数为 n0，度为 2 的结点数为 n2，则n0 = n2 + 1。

---

# 排序算法的稳定性是指

* 经过排序之后,能使值相同的数据保持原顺序中的相对位置不变

---

# HTTP方法

* GET

获取接口信息

* PUT

支持幂等性的POST

* HEAD

紧急查看接口的HTTP的头

* POST

提交数据到服务器

* DELETE

删除服务器上的资源

* OPTIONS

查看支持的方法

---

# **常见的HTTP相应状态码**

```tex
1xx：指示信息--表示请求已接收，继续处理

2xx：成功--表示请求已被成功接收、理解、接受

3xx：重定向--要完成请求必须进行更进一步的操作

4xx：客户端错误--请求有语法错误或请求无法实现

5xx：服务器端错误--服务器未能实现合法的请求

```

```tex
101 – 切换协议

201 – 已创建。
202 – 已接受。
203 – 非权威性信息。
204 – 无内容。
205 – 重置内容。
206 – 部分内容。

302 – 对象已移动。
304 – 未修改。
307 – 临时重定向。

401 – 访问被拒绝
403 – 禁止访问
404 – 未找到
405 – 用来访问本页面的 HTTP 谓词不被允许（方法不被允许）
406 – 客户端浏览器不接受所请求页面的 MIME 类型。
407 – 要求进行代理身份验证。
412 – 前提条件失败。
413 – 请求实体太大。
414 – 请求 URI 太长。
415 – 不支持的媒体类型。
416 – 所请求的范围无法满足。
417 – 执行失败。
423 – 锁定的错误。

500 – 内部服务器错误。

200：请求被正常处理
204：请求被受理但没有资源可以返回
206：客户端只是请求资源的一部分，服务器只对请求的部分资源执行GET方法，相应报文中通过Content-Range指定范围 的资源。
301：永久性重定向
302：临时重定向
303：与302状态码有相似功能，只是它希望客户端在请求一个URI的时候，能通过GET方法重定向到另一个URI上
304：发送附带条件的请求时，条件不满足时返回，与重定向无关
307：临时重定向，与302类似，只是强制要求使用POST方法
400：请求报文语法有误，服务器无法识别
401：请求需要认证
403：请求的对应资源禁止被访问
404：服务器无法找到对应资源
500：服务器内部错误
503：服务器正忙
```

---

# 反射机制功能

* 获得一个对象所属的类
* 获得一个类所有的成员变量和方法
* 运行时创建对象
* 运行时调用对象的方法

---

# Java创建对象的方式

* new一个
* 反射机制
* clone()方法
* 反序列化的方式

---

# Java程序初始化顺序

* 父类静态变量
* 父类静态代码块


* 子类静态变量
* 子类静态代码块


* 父类非静态变量
* 父类非静态代码块
* 父类构造函数


* 子类非静态变量
* 子类非静态代码块
* 子类构造函数

---

#  volatile

用来修饰被不同线程访问和修改的变量

被volatile类型定义的变量，系统每次用它都是直接从内存中取，而不会利用缓存。

---

# Java堆溢出

```java
public class HeapOOM1 {
    static class OOMObject{}
    public static void main(String [] args){
        List<OOMObject> list = new ArrayList<OOMObject>();
        while (true){
            list.add(new OOMObject());
        }
    }
}
```

---

# 单例模式

单例模式就是在应用程序中只创建一个该类的对象。又分为饿汉模式和懒汉模式。**\*实现套路也就是只提供私有构造函数，然后提供公有的 getInstance 方法。***

- 饿汉模式：也就是一开始就创建该对象
- 懒汉模式：等到需要用到的时候才创建该对象

## 饿汉模式

```java
public class Singleton {  
    private static Singleton singleton = new Singleton();  
    private Singleton(){};  
    public static Singleton getInstance(){  
        return singleton;  
    }  
}  
```



## 懒汉模式

```java
public class Singleton {  
    private static Singleton singleton;      
    private Singleton(){};  
    public static Singleton getInstance(){  
        if(singleton == null)  
            singleton = new Singleton();  
        return singleton;  
    }  
}  
```



## 多线程 synchronized

```java
public static synchronized Singleton getInstance(){  
    if(singleton == null)  
        singleton = new Singleton();   
    return singleton;  
}  
```



## 双重检验锁

```java
public class Singleton {

    private static volatile Singleton singleton;

    public static Singleton getInstance() {
        if (singleton == null) {                           //Single Checked
            synchronized (Singleton.class) {
                if (singleton == null) {                   //Double Checked
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

---




