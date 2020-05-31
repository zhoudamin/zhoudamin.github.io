---
title: ValidParentheses
categories:
  - 算法与数学
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2019-09-22 16:25:18
tags: 算法
---
给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。  
左括号必须以正确的顺序闭合。   
注意空字符串可被认为是有效字符串。   

<!--more-->
解题思路：   
1.用堆栈的数据结构处理括号的入栈与对比   
2.用map存储键值对，来进行入栈与对比判断  

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Stack;

/**
 * 给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。
 *
 * 有效字符串需满足：
 *
 * 左括号必须用相同类型的右括号闭合。
 * 左括号必须以正确的顺序闭合。
 * 注意空字符串可被认为是有效字符串。
 */
public class ValidParentheses {
    public static void main (String [] args){
        String str = "()[]{}";
        System.out.println(checkValid(str));
    }

    public static boolean checkValid (String str){
        Stack<Character> stk = new Stack<>();
        Map<Character,Character> map = new HashMap() ;
        map.put(')','(');
        map.put(']','[');
        map.put('}','{');

        for(int i= 0;i<str.length();i++){
            if(map.containsValue(str.charAt(i))){
                stk.push(str.charAt(i));
            }else {
                if(stk.peek().equals(map.get(str.charAt(i)))){
                    stk.pop();
                }else {
                    return false;
                }
            }
        }
        return stk.size()==0;
    }
}

```
2019.9.22    
晴天   
百草园  

