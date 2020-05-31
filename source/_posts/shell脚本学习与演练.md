---
title: shell脚本学习与演练
categories:
  - Linux
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2018-10-14 16:30:10
tags: Linux
---

平时经常接触Linux的操作指导，是不是可以写个脚本一键式解决这些繁琐的操作呢？ok

<!--more-->
## 写一个简单脚本
```shell
vi test1.sh

······························test1.sh
#! /bin/bash
echo "Hello !"
······························

chmod +x ./test1.sh
./test1.sh

```

##  shell 变量
```shell
your_name="zzz"
echo $your_name
your_name="cccc"
echo $your_name
```

## 字符串里写变量
```shell
your_name='aaa'
str="Hello, you are \"$your_name\"!"
echo $str
```

## 数组
```shell
array_name=(value0 value1 value2 value3)

# or
array_name=(
value0
value1
value2
value3
)

# 单个
valuen=${array_name[n]}

# 所有元素
echo ${array_name[@]}
```

## 运算
```shell
#!/bin/bash

# 不是单引号  是斜引号

val=`expr 4 + 2`
echo "and : $val"
```
