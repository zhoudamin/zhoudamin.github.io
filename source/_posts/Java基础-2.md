---
title: Java基础-2
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-08-11 14:38:55
categories: Java
tags: 基础
---

实干兴邦
<!--more-->

 J2SE基础

# 九种基本数据类型的大小，以及他们的封装类。

|  基本类型   | 大小(位) | 字节   |    最小值    |      最大值       |    封装类    |
| :-----: | :---: | ---- | :-------: | :------------: | :-------: |
| boolean |   -   | 1    |     -     |       -        |  Boolean  |
|  byte   |   8   | 1    |   -128    |      127       |   Byte    |
|  char   |  16   | 2    | Unicode 0 | Unicode 2^16-1 | Character |
|  short  |  16   | 2    |   -2^15   |     2^15-1     |   Short   |
|   int   |  32   | 4    |   -2^31   |     2^31-1     |  Integer  |
|  float  |  32   | 4    |           |                |   Float   |
| double  |  64   | 8    |           |                |  Double   |
|  long   |  64   | 8    |   -2^63   |     2^63-1     |   Long    |
|  void   |   -   |      |     -     |       -        |   Void    |

---

# Switch能否用string做参数？

JDK1.7之前是只支持int 或char

JDK1.7开始支持String

JDK1.5 开始支持 Enum 类

***

# equals与==的区别

“==” 用于基本数据类型的比较，判断引用是否指向堆内存的同一快地址。

equals 用于判断两个变量是否是对同一个对象的引用，即堆中的内容是否相同，返回值为布尔类型。



可以用equals方法检测两个字符串是否相等。

一定不能使用 == 运算符检测两个字符串是否相等！== 用来确定两个字符串是否放置在同一个位置上。

如果虚拟机始终将相同的字符串共享，就可以使用 == 运算符检测是否相等。但是实际上只有字符串常量是共享的，而+或者substring等操作产生的结果并不是共享的。

---

# Object有哪些公用方法？

1. equals方法

Object类中的equals方法用于检测一个对象是否等于另外一个对象。

2. hashCode方法

该方法用于哈希查找，可以减少在查找中使用equals的次数，重写了equals方法一般都要重写hashCode方法。这个方法在一些具有哈希功能的Collection中用到。

3. toString方法

用于返回表示对象值的字符串。

---

# Java的四种引用，强弱软虚，用到的场景

1. 强引用

强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收期绝不会回收它。

2. 软引用

如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些 对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可以用来实现内存敏感的高速缓存。

3. 弱引用

弱引用与软引用的区别在于：只具有弱引用的对象具有更短暂的生命周期。

在垃圾回收器线程扫描它所管辖的内存区域过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。

不过，由于垃圾回收器是一个优先级很低的线程，因此不一样很快发现那些只有弱引用的对象。

如果你想引用一个对象，但这个对象具有自己的生命周期，你不想介入这个对象的生命周期，这时候你就是用弱引用。

4. 虚引用

虚引用不会决定对象的生命周期，如果一个对象仅持有虚引用，那么他就和没有任何引用一样，在任何适合都可能被垃圾回收器回收。

总结：

Java 4种引用的级别由高到低依次为：

强》软》弱》虚

---

# Hashcode的作用

hashCode的存在主要是用于查找的快捷性，如Hashtable，HashMap等，hashCode是用来在散列存储结构中确定对象的存储地址的

---

# ArrayList、LinkedList、Vector的区别

ArrayList本质上是一个数组，当更多的元素添加到ArrayList中时，其大小会动态的增长，内部元素通过get和set方式进行访问。不是线程安全的。

LinkedList是一个双链表，因此在删除和添加元素的时候优于数组形式的ArrayList，但是在get和set方面弱于ArrayList；

Vector几乎和ArrayList一样，但是Vector是线程安全的，在更多元素进来时，Vector每次请求双倍的空间，而ArrayList每次对size增长50%。

---

# String、StringBuffer与StringBuilder的区别

String是不可变类

StringBuffer是可变类

StringBuilder不是线程安全的

在执行效率方面，StringBuilder最高，StringBuffer次之，String最低

如果要操作的数据量比较小，应优先用String类

如果是在单线程下操作大量数据，应优先用StringBuilder类

如果在多线程下操作大量数据，应优先考虑StringBuffer类。

---

# Map、Set、List、Queue、Stack的特点与用法

* Map

键映射到值的对象。  一个映射不能包含重复的键，每个键最多只能映射到一个值。

某些映射实现可明确保证其顺序，如TreeMap类；另一些映射实现不保证顺序，如HashMap类。

Map中元素，可以将key序列，value序列单独抽取出来。

使用keySet()抽取key序列，将map中所有keys生成一个Set。

使用 values() 抽取value序列，将map中的所有values生成一个Collection。

为什么一个是Set，一个是Collection。因为key是独一无二的，value允许重复。

* Set

一个不包含重复元素的Collection。

不可随机访问包含的元素

只能用lterator实现单向遍历

Set没有同步方法

* List

可随机访问包含的元素

元素是有序的

可在任意位置增删元素

不管访问多少次，元素位置不变

允许重复元素

用Iterator实现单向遍历，也可用ListIterator实现双向遍历

* Queue

先进先出

用offer()来加入元素

用poll()来获取并移出元素

peek()方法查看或使用前端元素

Queue实现通常不允许插入null元素

* Stack

后进先出

Stack继承自Vector，是同步的

提供了push、pop、peek、empty方法

* 用法

如果涉及到堆栈，队列等操作，应该考虑用List

对于需要快速插入，删除元素，应该用LinkedList

如果需要快速随机访问元素，应该用ArrayList

如果单线程环境，考虑非同步的类，效率较高。

---

# HashMap和HashTable的区别

主要区别：线程安全，同步，速度

1. HashMap几乎等价于HashTable，除了HashMap是非同步的，并可以接受null
# HashMap和ConcurrentHashMap的区别，HashMap的底层源码

HashMap的本质是数组加链表。根据key取得hash值，然后计算出数组下标，如果多个key对应到同一个下标，就用链表串起来，新插入的在前面。

ConcurrentHashMap在HashMap的基础上，将数据分为多个segment，默认16个，然后每次操作对一个segment加锁，避免多线程锁的几率，提高并发效率。

HashMap就是一个Entry数组，Entry数组中包含了键和值，其中next也是一个Entry对象，当hash冲突时，形成一个链表。

```java
static class Entry<K,V> implements Map.Entry<K,V> {
         final K key;
         V value;
         Entry<K,V> next;
         final int hash;
  
         /**
           * Creates new entry.
           */
         Entry(int h, K k, V v, Entry<K,V> n) {
             value = v;
             next = n; //hash值冲突后存放在链表的下一个
             key = k;
             hash = h;
         } 
         .........
     }
```

HashMap存储数据的put方法

```java
 public V put(K key, V value) {
          if (key == null) //如果键为null的话，调用putForNullKey(value)
              return putForNullKey(value);
          int hash = hash(key.hashCode());//根据键的hashCode计算hash码
          int i = indexFor(hash, table.length);
          for (Entry<K,V> e = table[i]; e != null; e = e.next) { 
            //处理冲突的，如果hash值相同，则在该位置用链表存储
              Object k;
              if (e.hash == hash && ((k = e.key) == key || key.equals(k))) { 
                //如果key相同则覆盖并返回旧值
                  V oldValue = e.value;
                 e.value = value;
                 e.recordAccess(this);
                 return oldValue;
             }
         }
 
         modCount++;
         addEntry(hash, key, value, i);
         return null;
     }
```

当我们往HashMap中put元素时，先根据key的hash值得到这个元素在数组中的位置（即下标），然后就可以把这个元素放到对应的位置中了，如果这个元素的位置上已经存在其他元素了，那么在同一个位子上的元素将会以链表的形式存放，新加入的放链表头，之前加入的放后面。

从HashMap中get元素时，首先计算key的hash值，找到数组中对应位置的某一元素，然后通过key的equals方法在对应位置的链表中找到需要的元素。

如果没有与key相同的键，则调用addEntry方法创建一个Entry对象：

```java
 void addEntry(int hash, K key, V value, int bucketIndex) {
         Entry<K,V> e = table[bucketIndex]; 
   //如果要加入的位置有值，将该位置原先的值设置为新entry的next,也就是新entry链表的下一个节点
         table[bucketIndex] = new Entry<>(hash, key, value, e);
         if (size++ >= threshold) //如果大于临界值就扩容
             resize(2 * table.length); //以2的倍数扩容
     }
```

扩容数据

```java
 void resize(int newCapacity) {
         Entry[] oldTable = table;
         int oldCapacity = oldTable.length;
         if (oldCapacity == MAXIMUM_CAPACITY) {
             threshold = Integer.MAX_VALUE;
             return;
         }
 
         Entry[] newTable = new Entry[newCapacity];
         transfer(newTable);//用来将原先table的元素全部移到newTable里面
         table = newTable;  //再将newTable赋值给table
         threshold = (int)(newCapacity * loadFactor);//重新计算临界值
     }
```

---

# TreeMap、HashMap、LindedHashMap的区别。

# Collection包结构，与Collections的区别。

# try catch finally，try里有return，finally还执行么？

1. try中没有异常且try中有return

try---finally---return

2. try中有异常，try中有return

try---catch---finally---return

总之finally永远执行

---




1. Excption与Error包结构。OOM你遇到过哪些情况，SOF你遇到过哪些情况。
2. Java面向对象的三个特征与含义。



# Override和Overload的含义与区别

* 重写（Override）

重写是子类对父类的允许访问的方法的实现过程进行重新编写。

返回值和形参都不能改变。即外壳不变，重写内在实现。

重写的好处在于子类可以根据需要，定义特定于自己的行为。

```java
public class LiftOff {
       public static void main(String args []){
           Animal a= new Animal();
           Animal b = new pig();
           a.eat();
           b.eat();
       }
    }


    class Animal{
        public void eat(){
            System.out.println("Animal eat xx ");
        }
    }

    class pig extends Animal{
        public void eat(){
            System.out.println("pig eat siliao");
        }
    }
```

```java
Animal eat xx 
pig eat siliao
```

重写规则：

参数列表必须完全与被重写方法相同（也就是说重写，不能添加写方法，不能添加参数。）

返回类型必须完全与被重写方法的返回类型相同

父类的成员方法只能被他的子类重写

声明为final的方法不能被重写

声明为static的方法不能被重写，但是能再次声明。

构造方法不能被重写

* 重载（Overload）

重载是在同一个类里，方法名字相同，参数不同，返回类型可相同也可不同的多个方法。

每个重载的方法都必须有一个独一无二的参数类型列表。

被重载的方法必须改变参数列表，被重载的方法可以改变返回类型，被重载的方法可以改变访问修饰符，方法能在同一个类中或在一个子类中被重载。

```java
  public class LiftOff {
        public int test(){
            System.out.println("Overload1");
            return 1;
        }

    public void test(int a){
        System.out.println("Overload1"+ a);
    }

    public String test(String a1 , String a2){
        System.out.println("Overload1"+ a1+a2);
        return a1;
    }

       public static void main(String args []){
         LiftOff t=new LiftOff();
         System.out.println(t.test());
         t.test(2);
         System.out.println(t.test("asd","erf"));
       }
    }
```

```java
Overload1
1
Overload12
Overload1asderf
asd
```

重载和重写区别：

|       | 重载方法（Overload） | 重写方法（Override） |
| :---: | :------------: | :------------: |
| 参数列表  |      必须修改      |      不能改       |
| 返回类型  |      可以修改      |      不能改       |
|  异常   |      可以修改      | 可以减少或删除，不能抛出新的 |
| 访问修饰符 |      可以修改      | 只能降低限制，不能变高限制  |

限制又低到高： public、protected、private

---

# Interface与abstract类的区别

如果一个类中包含抽象方法，那么这个类就是抽象类。类或方法声明为abstract

接口就是指一个方法的集合，接口中所有方法都没有方法体，接口通过关键字interface实现。

接口和抽象类的相同点：

1. 都不能被实例化
2. 接口的实现类或抽象类的子类都只有实现了接口或抽象类中的方法后才才能被实例化。

不同点：

1. 接口只有定义，其方法不能在接口中实现，只有实现接口的类才能实现接口中定义的方法；而抽象类可以有定义和实现，即其方法可以在抽象类中被实现。
2. 接口需要实现（用implement），但抽象类只能被继承（用extends），一个类可以实现多个接口，但是一个类只可以继承一个抽象类，因此使用接口可以间接达到多重继承的目的。
3. 接口强调特定功能的实现，"has - a" ；抽象类强调所属关系，"is - a "
4. 接口中定义的成员变量默认为 public static final ， 只能够有静态的不能被修改的数据成员，而且，必须给其赋值，其所以的成员方法都是public，abstract的，而且只能被这两个关键字修饰。而抽象类可以有自己的数据成员变量，也可以有非抽象的成员方法，而且，抽象类中的成员变量默认为default，当然也可以定义为private，protected，public。所以，当功能需要累积时，用抽象类，不需要累积时，用接口。
5. 接口被运用于实现比较常用的功能，便于以后的维护；抽象类更倾向于充当公共类的角色，不适用于日后重新修改内部代码。

---

# Static class 与non static class的区别

# java多态的实现原理

# 实现多线程的两种方法：Thread与Runable



# 线程同步的方法：sychronized、lock、reentrantLock等



---

# 锁的等级：方法锁、对象锁、类锁

* `Java中锁的机制`：

synchronized  在修饰代码块的时候需要一个reference对象作为锁的对象。

在修饰方法的时候默认是当前对象作为锁的对象。

在修饰类的时候默认是当前的Class对象作为锁的对象。

线程同步的方法： sychronized、lock、reentrantLock分析

* 方法锁（synchronized修饰方法时）

通过在方法声明中加入synchronized关键字来声明synchronized方法。

* 对象锁（synchronized修饰方法或代码块）

当一个对象中有synchronized method 或 synchronized block 的时候调用此对象的同步方法或进入其同步区域时，就必须先获得对象锁，如何此对象的对象锁已被其他调用者占用，则需要等待此锁被释放。（方法锁也是对象锁）

---





# 写出生产者消费者模式。

# ThreadLocal的设计理念与作用。

# ThreadPool用法与优势。

# Concurrent包里的其他东西：ArrayBlockingQueue、CountDownLatch等等。

# wait()和sleep()的区别

他们都是一种使线程暂停执行的方法

区别：

1. 原理不同。sleep()是Thread类的静态方法；wait()方法是Object类的方法。
2. 对锁的处理机制不同。sleep不放，wait释放。
3. 使用区域不同
4. sleep()必须捕获异常，wait()不需要

---



# foreach与正常for循环效率对比



# Java IO与NIO。









# 反射的作用与原理

反射机制能够实现在运行时对类进行装在，因此能增加程序的灵活性。

功能：

1. 得到一个对象所属的类；
2. 获取一个类所有的成员变量和方法；
3. 在运行时创建对象
4. 在运行时调用对象的方法


---

# Java创建对象的方式有几种

1. 通过new实例化一个对象
2. 通过反射机制创建对象
3. clone()
4. 反序列化的方式

---

# 泛型常用特点，List<String>能否转为List<Object>。

# 解析XML的几种方式的原理与特点：DOM、SAX、PULL。

# Java与C++对比。

# Java8新特性

- **Lambda 表达式** − Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中。
- **方法引用** − 方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。
- **默认方法** − 默认方法就是一个在接口里面有了一个实现的方法。
- **新工具** − 新的编译工具，如：Nashorn引擎 jjs、 类依赖分析器jdeps。
- **Stream API** −新添加的Stream API（java.util.stream） 把真正的函数式编程风格引入到Java中。
- **Date Time API** − 加强对日期与时间的处理。
- **Optional 类** − Optional 类已经成为 Java 8 类库的一部分，用来解决空指针异常。
- **Nashorn, JavaScript 引擎** − Java 8提供了一个新的Nashorn javascript引擎，它允许我们在JVM上运行特定的javascript应用。



---

# 设计模式：单例、工厂、适配器、责任链、观察者等等

# JNI的使用



Java里有很多很杂的东西，有时候需要你阅读源码，大多数可能书里面讲的不是太清楚，需要你在网上寻找答案。

推荐书籍：《java核心技术卷I》《Thinking in java》《java并发编程》《effictive java》《大话设计模式》

# JVM

1. 内存模型以及分区，需要详细到每个区放什么。

2. 堆里面的分区：Eden，survival from to，老年代，各自的特点。

3. 对象创建方法，对象的内存分配，对象的访问定位。

4. GC的两种判定方法：引用计数与引用链。

5. GC的三种收集方法：标记清除、标记整理、复制算法的原理与特点，分别用在什么地方，如果让你优化收集方法，有什么思路？

6. GC收集器有哪些？CMS收集器与G1收集器的特点。

7. Minor GC与Full GC分别在什么时候发生？

8. 几种常用的内存调试工具：jmap、jstack、jconsole。

9. 类加载的五个过程：加载、验证、准备、解析、初始化。

10. 双亲委派模型：Bootstrap ClassLoader、Extension ClassLoader、ApplicationClassLoader。

11. 分派：静态分派与动态分派。

JVM过去过来就问了这么些问题，没怎么变，内存模型和GC算法这块问得比较多，可以在网上多找几篇博客来看看。

推荐书籍：《深入理解java虚拟机》



# TCP/IP

1. OSI与TCP/IP各层的结构与功能，都有哪些协议。

2. TCP与UDP的区别。

3. TCP报文结构。

4. TCP的三次握手与四次挥手过程，各个状态名称与含义，TIMEWAIT的作用。

5. TCP拥塞控制。

6. TCP滑动窗口与回退N针协议。

7. Http的报文结构。

8. Http的状态码含义。

9. Http request的几种类型。

10. Http1.1和Http1.0的区别

11. Http怎么处理长连接。

12. Cookie与Session的作用于原理。

13. 电脑上访问一个网页，整个过程是怎么样的：DNS、HTTP、TCP、OSPF、IP、ARP。

14. Ping的整个过程。ICMP报文是什么。

15. C/S模式下使用socket通信，几个关键函数。

16. IP地址分类。

17. 路由器与交换机区别。

网络其实大体分为两块，一个TCP协议，一个HTTP协议，只要把这两块以及相关协议搞清楚，一般问题不大。

推荐书籍：《[TCP/IP协议族](http://www.amazon.cn/gp/product/B004JZY9BI/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&tag=importnew-23&linkCode=as2&camp=536&creative=3200&creativeASIN=B004JZY9BI)》





# 项目

关于项目，这部分每个人的所做的项目不同，所以不能具体的讲。项目不再与好与不好，在于你会不会包装，有时候一个很low的项目也能包装成比较高大上的项目，多用一些专业名词，突出关键字，能使面试官能比较容易抓住重点。在聊项目的过程中，其实你的整个介绍应该是有一个大体的逻辑，这个时候是在考验你的表达与叙述能力，所以好好准备很重要。

面试官喜欢问的问题无非就几个点：

1. XXX（某个比较重要的点）是怎么实现的？

2. 你在项目中遇到的最大的困难是什么，怎么解决的？

3. 项目某个部分考虑的不够全面，如果XXXX，你怎么优化？

4. XXX（一个新功能）需要实现，你有什么思路？

其实你应该能够预料到面试官要问的地方，请提前准备好，如果被问到没有准备到的地方，也不要紧张，一定要说出自己的想法，对不对都不是关键，主要是有自己的想法，另外，你应该对你的项目整体框架和你做的部分足够熟悉。

# SpringBoot的主要特性





#  其他

你应该问的问题

面试里，最后面完之后一般面试官都会问你，你有没有什么要问他的。其实这个问题是有考究的，问好了其实是有加分的，一般不要问薪资，主要应该是：关于公司的、技术和自身成长的。

以下是我常问的几个问题，如果需要可以参考：

1. 贵公司一向以XXX著称，能不能说明一下公司这方面的特点？

2. 贵公司XXX业务发展很好，这是公司发展的重点么？

3. 对技术和业务怎么看？

4. 贵公司一般的团队是多大，几个人负责一个产品或者业务？

5. 贵公司的开发中是否会使用到一些最新技术？

6. 对新人有没有什么培训，会不会安排导师？

7. 对Full Stack怎么看？

8. 你觉得我有哪些需要提高的地方？

# 知识面

除了基础外，你还应该对其他领域的知识有多少有所涉猎。对于你所熟悉的领域，你需要多了解一点新技术与科技前沿，你才能和面试官谈笑风生。

# 软实力

什么是软实力，就是你的人际交往、灵活应变能力，在面试过程中，良好的礼节、流畅的表达、积极的交流其实都是非常重要的。很多公司可能不光看你的技术水平怎么样，而更看重的是你这个人怎么样的。所以在面试过程中，请保持诚信、积极、乐观、幽默，这样更容易得到公司青睐。

很多时候我们都会遇到一个情况，就是面试官的问题我不会，这时候大多数情况下不要马上说我不会，要懂得牵引，例如面试官问我C++的多态原理，我不懂，但我知道java的，哪我可以向面试官解释说我知道java的，类似的这种可以往相关的地方迁移（但是需要注意的是一定不要不懂装懂，被拆穿了是很尴尬的），意思就是你要尽可能的展示自己，表现出你的主动性，向面试官推销自己。

还有就是遇到智力题的时候，不要什么都不说，面试官其实不是在看你的答案，而是在看你的逻辑思维，你只要说出你自己的见解，有一定的思考过程就行。



![]()