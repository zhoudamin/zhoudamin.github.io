---
title: Linux学习指南
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-08-28 22:22:39
categories: Linux
tags: Linux
---

发现学过的东西很容易忘，所以常常复习、总结还是蛮必要的。

<!--more-->

# Linux基础

* 输出

```shell
echo "Hellbo"
Hellbo
```

* 快捷键

|      快捷键      |           作用           |
| :-----------: | :--------------------: |
|    Ctrl+c     |          终止程序          |
|    Ctrl+d     |      键盘输入结束或退出终端       |
|    Ctrl+s     |  暂停当前程序，暂停后按下任意键恢复运行   |
|    Ctrl+z     | 把当前程序放到后台运行，fg将程序恢复到前台 |
|    Ctrl+a     |       将光标移至输入行头        |
|    Ctrl+e     |       将光标移至输入行尾        |
|    Ctrl+k     |       删除从光标位置到行末       |
| Alt+Backspace |        向前删除一个单词        |
|  Shift+Pgup   |       将终端显示向上滚动        |
|  Shift+PgDn   |       将终端显示向下滚动        |

* 创建文件

```shell
touch adeni.txt  dsaidj.txt   //创建2个文件
touch abc_{1..10}_linux.txt   //创建多个文件
```

* 通配符找文件

```shell
ls *.txt
```

* Shell通配符

| 字符                    | 含义                   |
| --------------------- | -------------------- |
| *                     | 匹配0或多个字符             |
| ？                     | 匹配任意一个字符             |
| [list]                | 匹配list中的任意单一字符       |
| [!list]               | 匹配除list中的任意单一字符以外的字符 |
| [c1-c2]               | 匹配c1-c2终端任意单一字符      |
| {string1,string2,...} | 匹配list中的任意单一字符       |
| {c1..c2}              | 匹配list中的任意单一字符       |

---

# 用户及文件权限管理

* 创建用户

```shell
sudo adduser zdm    //创建用户
ls /home            //默认为新用户创建home目录
su -l zdm           //切换登陆用户
groups zdm          //查看自己属于哪些用户组
cat /etc/group | sort    //查看 /etc/group 文件
// 将其他用户加入sudo用户组
su -l zdm
sudo ls
```

* 变更文件所有者

```shell
touch iphone
cd /home/zdm
ls iphone
sudo chown shiyanlou iphone

echo "echo \"hello shiyanlou\"" > iphone                //加内容
chmod 700 iphone                         //修改权限

cp -r father family    //复制目录

rm test           //删除
```

---

# 环境变量及文件查找

```shell
//创建tmp变量
$ declare tmp

//使用=赋值
$ tmp=zdm

//输出
$ echo $tmp

//创建一个Shell脚本文件
$ gedit hello_shell.sh

--------------------------
#!/bin/bash

for((i=0;i<10;i++));do
echo "hello shell"
done 
 exit 0
 -------------------------
 
//为文件添加可执行权限
$ chmod 755 hello_shell.sh

//执行脚本
$ ./hello_shell.sh

//创建一个C语言程序
$ gedit hello_world.c

-----------------------------
#include <stdio.h>
int main(void){
  printf("hello world!\n");
  return 0;
}
-----------------------------

//使用gcc生成可执行文件
$ gcc -o hello_world hello_world.c

//创建一个mybin目录，将上面2文件放入
$ mkdir mybin
$ mv hello_shell.sh hello_world mybin/

//在mybin中分别运行这两个程序
$ cd mybin
$ ./hello_shell.sh
$ ./hello_world
```

# 磁盘管理

```shell
//查看磁盘容量
df

//以磁盘块大小的方式显示容量
df -h

//du 命令查看目录的容量
du
du -h

//从/dev/zero设备创建一个容量为 256M 的空文件：
dd if=/dev/zero of=virtual.img bs=1M count=256
du -h virtual.img
```



---

# 任务计划crontab

```sh
//添加任务计划
sudo cron -f &
crontab -e

//查看
man crontab

//每分钟创建一个以当前年月日时分秒为名字的空白文件
*/1 * * * * touch /home/shiyanlou/$(date +\%Y\%m\%d\%H\%M\%S)

//查看添加了哪些任务
crontab -l

//cron 是否成功的在后台启动
pgrep cron

//执行任务命令之后在日志中的信息反馈
sudo tail -f /var/log/syslog

//删除任务
crontab -r
```



---

# 命令执行顺序控制与管道

```shell
//sort排序命令

//默认为字典排序
cat /etc/passwd | sort

//反转排序
cat /etc/passwd | sort -r

//按特定字段排序
cat /etc/passwd | sort -t ":" -k 3
```

---

# 简单的文本处理

```shell
//tr:删除一段文本中的某些文字

tr [option]...SET1 [SET2]

//col: 将Tab换成等数量的空格键

col [option]
```

---

# 常用命令

1. 查看目录与文件：ls
2. 切换目录：cd
3. 显示当前目录：pwd
4. 创建空文件：touch
5. 创建目录：mkdir
6. 查看文件内容：cat
7. 分页查看文件内容：more
8. 查看文件尾内容：tail
9. 拷贝：cp
10. 剪切或改名：mv
11. 删除：rm
12. 搜索文件：find
13. 创建链接文件：In
14. 显示或配置网络设备：ifconfig
15. 显示网络相关信息：netstat
16. 显示进程状态：ps
17. 查看目录使用情况：du
18. 查看磁盘空间使用情况：df
19. 显示系统当前进程信息：top
20. 杀死进程：kill
21. 压缩和解压：tar
22. 改变文件或目录的拥有者和组：chown
23. 改变文件或目录的访问权限：chmod
24. 文本编辑：vim
25. 关机或重启：shutdown
26. 帮助命令：man

---

# 20个常用问题

[20条Linux命令面试问答](https://linux.cn/article-4790-1.html)



**问:1 如何查看当前的Linux服务器的运行级别？**
答: ‘who -r’ 和 ‘runlevel’ 命令可以用来查看当前的Linux服务器的运行级别。

**问:2 如何查看Linux的默认网关？**
答: 用 “route -n” 和 “netstat -nr” 命令，我们可以查看默认网关。除了默认的网关信息，这两个命令还可以显示当前的路由表。

**问:3 如何在Linux上重建初始化内存盘镜像文件？**
答: 在CentOS 5.X / RHEL 5.X中，可以用mkinitrd命令来创建初始化内存盘文件，举例如下：

> \# mkinitrd -f -v /boot/initrd-$(uname -r).img $(uname -r)

如果你想要给特定的内核版本创建初始化内存盘，你就用所需的内核名替换掉 ‘uname -r’ 。
在CentOS 6.X / RHEL 6.X中，则用dracut命令来创建初始化内存盘文件，举例如下：

> \# dracut -f

以上命令能给当前的系统版本创建初始化内存盘，给特定的内核版本重建初始化内存盘文件则使用以下命令：

> \# dracut -f initramfs-2.x.xx-xx.el6.x86_64.img 2.x.xx-xx.el6.x86_64



**问:4 cpio命令是什么？**
答: cpio就是复制入和复制出的意思。cpio可以向一个归档文件（或单个文件）复制文件、列表，还可以从中提取文件。

**问:5 patch命令是什么？如何使用？**
答: 顾名思义，patch命令就是用来将修改（或补丁）写进文本文件里。patch命令通常是接收diff的输出并把文件的旧版本转换为新版本。举个例子，Linux内核源代码由百万行代码文件构成，所以无论何时，任何代码贡献者贡献出代码，只需发送改动的部分而不是整个源代码，然后接收者用patch命令将改动写进原始的源代码里。
创建一个diff文件给patch使用，

> \# diff -Naur old_file new_file > diff_file

旧文件和新文件要么都是单个的文件要么都是包含文件的目录，-r参数支持目录树递归。
一旦diff文件创建好，我们就能在旧的文件上打上补丁，把它变成新文件：

> \# patch < diff_file



**问:6 aspell有什么用 ?**
答: 顾名思义，aspell就是Linux操作系统上的一款交互式拼写检查器。aspell命令继任了更早的一个名为ispell的程序，并且作为一款免费替代品 ，最重要的是它非常好用。当aspell程序主要被其它一些需要拼写检查能力的程序所使用的时候，在命令行中作为一个独立运行的工具的它也能十分有效。

**问:7 如何从命令行查看域SPF记录？**
答: 我们可以用dig命令来查看域SPF记录。举例如下：

> linuxtechi@localhost:~$ dig -t TXT [http://google.com](http://link.zhihu.com/?target=http%3A//google.com)



**问:8 如何识别Linux系统中指定文件(/etc/fstab)的关联包？**
答:

> \# rpm -qf /etc/fstab

以上命令能列出提供“/etc/fstab”这个文件的包。

**问:9 哪条命令用来查看bond0的状态？**
答:

> cat /proc/net/bonding/bond0



**问:10 Linux系统中的/proc文件系统有什么用？**
答: /proc文件系统是一个基于内存的文件系统，其维护着关于当前正在运行的内核状态信息，其中包括CPU、内存、分区划分、I/O地址、直接内存访问通道和正在运行的进程。这个文件系统所代表的并不是各种实际存储信息的文件，它们指向的是内存里的信息。/proc文件系统是由系统自动维护的。

**问:11 如何在/usr目录下找出大小超过10MB的文件？**
答:

> \# find /usr -size +10M



**问:12 如何在/home目录下找出120天之前被修改过的文件？**
答:

> \# find /home -mtime +120



**问:13 如何在/var目录下找出90天之内未被访问过的文件？**
答:

> \# find /var \! -atime -90



**问:14 在整个目录树下查找文件“core”，如发现则无需提示直接删除它们。**
答:

> \# find / -name core -exec rm {} \;



**问:15 strings命令有什么作用？**
答: strings命令用来提取和显示非文本文件中的文本字符串。（LCTT 译注：当用来分析你系统上莫名其妙出现的二进制程序时，可以从中找到可疑的文件访问，对于追查入侵有用处）

**问:16 tee 过滤器有什么作用 ?**
答: tee 过滤器用来向多个目标发送输出内容。如果用于管道的话，它可以将输出复制一份到一个文件，并复制另外一份到屏幕上（或一些其它程序）。

> linuxtechi@localhost:~$ ll /etc | nl | tee /tmp/ll.out

在以上例子中，从ll输出可以捕获到 /tmp/ll.out 文件中，并且同样在屏幕上显示了出来。

**问:17 export PS1 = ”$LOGNAME@hostname:\$PWD: 这条命令是在做什么？**
答: 这条export命令会更改登录提示符来显示用户名、本机名和当前工作目录。

**问:18 ll | awk ‘{print $3,”owns”,$9}’ 这条命令是在做什么？**
答: 这条ll命令会显示这些文件的文件名和它们的拥有者。

**问:19 :Linux中的at命令有什么用？**
答: at命令用来安排一个程序在未来的做一次一次性执行。所有提交的任务都被放在 /var/spool/at 目录下并且到了执行时间的时候通过atd守护进程来执行。

**问:20 linux中lspci命令的作用是什么？**
答: lspci命令用来显示你的系统上PCI总线和附加设备的信息。指定-v，-vv或-vvv来获取越来越详细的输出，加上-r参数的话，命令的输出则会更具有易读性。

---

# 360Linux运维面试题

[奇虎360Linux运维工程师面试真题](https://zhuanlan.zhihu.com/p/29693614)

**1、写一个脚本查找最后创建时间是3天前，后缀是\*.log的文件并删除。**

 find / -name “*.log” -ctime +3 -exec rm -f {} \;

**2、写一个脚本将某目录下大于100k的文件移动至/tmp下。**

 for i in `find /test -type f -size +100k`;do cd /test && mv $i /tmp;done

**3、写一个脚本将数据库备份并打包至远程服务器192.168.1.1 /backup目录下。**

 mount 192.168.1.1:/backup /mnt cd /mnt /usr/local/mysql/bin/mysqldump -hlocalhost -uroot test >test.sql tar czf test.sql.tar.gz test.sql rm -f test.sql

**4、写一个防火墙配置脚本，只允许远程主机访问本机的80端口。**

 iptables -P INPUT ACCEPT iptables -P OUTPUT ACCEPT iptables -P FORWARD ACCEPT iptables -F iptables -X iptables -A INPUT -i eth0 -p tcp –dport 80 -j ACCEPT iptables -P INPUT DROP

**5、写一个脚本进行nginx日志统计，得到访问ip最多的前10个(nginx日志路径：/home/logs/nginx/default/access.log** awk ‘{a[$1]++}END{for (j in a) print a[j],j}’ /home/logs/nginx/default/access.log|sort -nr|head -10

**6、写出下列配置的含义** 

（1）MaxKeepAliveRequests 100 

（2）Options FollowSymLinks Order Deny Allow Deny from all Allow from 192.168.1.1

（1）MaxKeepAliveRequests — 100 连接的最大请求数 

（2）Options FollowSymLinks — 允许192.168.1.1可以列目录 Order Deny Allow Deny from all Allow from 192.168.1.1

**7、写一个脚本把指定文件里的/usr/local替换为别的目录。** 

sed ‘s:/user/local:/tmp:g’ filename

**8、简要描述Linux的启动过程？**

 BIOS启动引导(从mbr中装载启动管理器grub)—-GRUB启动引导(装载kernel和initrd到内存)—–内核启动参数-sys init初始化..

**9、简要叙述下列端口所运行的服务**

 21、22、23、25、110、143、873、3306 对应的服务是 ftp ssh telnet snmp pop3 IMAP rsync

**10、TCP断头最小长度是多少字节？**

 64字节

**11、让某普通用户能进行cp /dir1/file1 /dir2的命令时，请说明dir1 file1最小具有什么权限？**

读取和执行权限rx

---

















































