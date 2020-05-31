---
title: Java笔记
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-04-05 12:36:47
categories: Java
tags: 笔记
---
Java笔记
![](http://cdn01.wallconvert.com/_media/wp_200x125/1/5/41791.jpg)

<!--more-->

# 动态数组

```
ArrayList<String> List = new ArrayList<String>();  //定义动态数组
List.add(temp);                                    //添加字符串


List<Integer> ret = new ArrayList<Integer>();
ret.add(i+1);
           
```

# 分割字符串单词

```
String.Trim 方法
·Trim()	 从当前 String 对象移除所有前导空白字符和尾部空白字符。

java.lang.string.split 
split 方法 
将一个字符串分割为子字符串，然后将结果作为字符串数组返回。

String[] res = s.trim().split("\\s+"); 将头尾空格去掉，并且切割成单词

sb.append(res[i]).append(" ");   //每次添加 单词+“ ”   组合

return sb.toString().trim();   //将字符串头尾空格去掉  
```

# 数字转化成字母，存在字符串中

```
arr.insert(0,(char)('A'+n%26));
```

# 进制转换函数

```
十进制转成十六进制： 
Integer.toHexString(int i) 
十进制转成八进制 
Integer.toOctalString(int i) 
十进制转成二进制 
Integer.toBinaryString(int i) 
十六进制转成十进制 
Integer.valueOf("FFFF",16).toString() 
八进制转成十进制 
Integer.valueOf("876",8).toString() 
二进制转十进制 
Integer.valueOf("0101",2).toString() 
```

# 数组

·整数转为二进制 Integer.toBinaryString(i)；

数组排序
·   Arrays.sort(nums);

# Set：

Set<Integer> s=new HashSet<Integer>();

contains()，判断某个元素是否存在于HashSet()中，存在返回true，否则返回false。

add()如果此 set 中尚未包含指定元素，则添加指定元素。如果此Set没有包含满足(e==null ? e2==null : e.equals(e2)) 的e2时，则将e2添加到Set中，否则不添加且返回false。

remove如果指定元素存在于此 set 中，则将其移除。

clear从此 set 中移除所有元素。

# Java.lang.Character.getNumericValue()方法

```
·java.lang.Character.getNumericValue(char ch) 返回指定Unicode字符表示的int值。例如，字符'\ u216C'（罗马数字50）将返回一个int值50。

·字母的大写的AZ（'\ u0041'到'\ u005A'），小写字母（'\ u0061'到'\ u007A'），全宽度变体（'\ uFF21“的通过'\ uFF3A'和'\ uFF41”通过'\ uFF5A'）的形式从10到35的数值。这是独立的Unicode规范，这些字符的值不分配数值。

·如果该字符没有一个数字值，则返回-1。如果字符具有一个数字值，该值不能被表示为一个非负整数（例如，一个分数值），则返回-2。
```

# 基础
```
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class Main{
    public  static void print(int index ,Object obj){
        System.out.println(String.format("{%d},%s",index,obj.toString()));
    }

    public static void main(String[] args){
    //demoString();
        demoList();
    }

    public static void demoList(){
        List<String> strList=new ArrayList<String>();
        for(int i=0;i<4;i++){
            strList.add(String.valueOf(i));
        }
        print(1,strList);

        List<String> strListB=new ArrayList<String>();
        for(int i=0;i<4;i++){
            strListB.add(String.valueOf(i*i));
        }
        strList.addAll(strListB);
        print(2,strList);
        strList.remove(0);
        print(3,strList);
        strList.remove(String.valueOf(1));
        print(4,strList);
        print(5,strList.get(1));

        Collections.sort(strList);
        print(6,strList);
        Collections.sort(strList, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o2.compareTo(o1);
            }
        });
        print(7,strList);
    }
    /*
    {1},[0, 1, 2, 3]
{2},[0, 1, 2, 3, 0, 1, 4, 9]
{3},[1, 2, 3, 0, 1, 4, 9]
{4},[2, 3, 0, 1, 4, 9]
{5},3
{6},[0, 1, 2, 3, 4, 9]
{7},[9, 4, 3, 2, 1, 0]
     */
    
    public static void demoString(){
        String str="Hello nowcoder";
        print(1,"Hello!");
        print(2,str.charAt(6));
        print(3,str.codePointAt(1));
        print(4,str.compareTo("Hello mewcoder"));
        print(5,str.compareTo("Hello pewcoder"));
        print(6,str.contains("Hello"));
        print(7,str.indexOf('e'));
        print(8,str.compareToIgnoreCase("Hello Nowcoder"));
        print(9,str.concat("!!"));
        print(10,str.endsWith("nowcoder"));
        print(11,str.startsWith("Hello"));
        print(12,str.lastIndexOf('o'));
        print(13,str.toUpperCase());
        print(14,str.replace('o','a'));
        print(15,str.replaceAll("o|l","a"));
        print(16,str.replaceAll("Hello","hi"));

        StringBuilder sb =new StringBuilder();
        sb.append(true);
        sb.append(1);
        sb.append(2.2);
        print(17,sb.toString());
        print(18,"a"+"b"+String.valueOf(12));
    }
    /*{1},Hello!
{2},n
{3},101
{4},1
{5},-2
{6},true
{7},1
{8},0
{9},Hello nowcoder!!
{10},true
{11},true
{12},10
{13},HELLO NOWCODER
{14},Hella nawcader
{15},Heaaa nawcader
{16},hi nowcoder
{17},true12.2
{18},ab12
*/
}

```
# 数组随机打乱
```
List<Integer> array = Arrays.asList(new Integer[]{1,2,3,4,5});
Collections.shuffle(array);
```


![](http://cdn01.wallconvert.com/_media/wp_400x250/1/4/38827.jpg)