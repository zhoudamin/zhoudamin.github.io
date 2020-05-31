---
title: Java之广度优先搜索算法
categories:
  - 算法与数学
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2018-08-12 22:35:39
tags: 算法与数学
---

BFS，全称是Breadth First Search。 简单说就是图搜索算法。
<!--more-->
# Word Ladder
Given two words (beginWord and endWord), and a dictionary’s word list, find the length of shortest transformation sequence frombeginWord to endWord, such that:

Only one letter can be changed at a time 
Each intermediate word must exist in the word list 
For example,

Given: 
beginWord = “hit” 
endWord = “cog” 
wordList = [“hot”,”dot”,”dog”,”lot”,”log”]

As one shortest transformation is “hit” -> “hot” -> “dot” -> “dog” -> “cog”, 
return its length 5.

Note:

Return 0 if there is no such transformation sequence. 
All words have the same length. 
All words contain only lowercase alphabetic characters. 


简单来说就是从一个单词出发，每次只改变一个字母，直到变到最后一个单词，求最短路径。

再简单点思考就是 hit--->hot--->lot/dot--->log/dog--->cog

思路就是用两个set,一个存老的单词，如果添加进路径，就删除；一个存路径。

循环终止条件是路径匹配到尾巴的目标单词；

关键代码
```java
public int ladderLen(String beginWord ,String endWord ,List<String> wordList){
    Set<String> wordSet = new HashSet<>(wordList);
    Set<String> visited = new HashSet<>;
    visited.add(beginWord);
    int len=1;

    //开始计算路径与清楚单词集合里的单词
    //算法终止条件
    while(!visited.contains(endWord)){
        Set<String> temp = new HashSet<>();
        for(String word:visited){
            for(int i=0;i<word.length;i++){
                char[] chars=word.toCharArray();
                for(int j=(int)'a';j<(int)'z'+1;j++){
                    chars[i]=(char)j;
                    String newWord=new String(chars);
                    if(wordSet.contains(newWord)){
                        temp.add(newWord);
                        wordSet.remove(newWord);
                    }
                }
            }
        }
        len += 1;
        if(temp.size()==0){
            return 0;
        }
        vistied=temp;
    }
    return len;
}
```


