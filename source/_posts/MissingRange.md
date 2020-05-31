---
title: MissingRange
categories:
  - 算法与数学
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2019-09-08 17:39:20
tags: 算法
---

  Given a sorted integer array where the range of elements are [0, 99] inclusive, return its
  missing ranges.    
  For example, given [0, 1, 3, 50, 75]   
  return [“2”, “4->49”, “51->74”, “76->99”]
<!--more-->

```java
package com.zdm.demo;

import java.util.ArrayList;
import java.util.List;

/**
 * Given a sorted integer array where the range of elements are [0, 99] inclusive, return its
 * missing ranges.
 * For example, given [0, 1, 3, 50, 75], return [“2”, “4->49”, “51->74”, “76->99”]
 */
public class MissingRange {
    public static void main(String[] args) {
        int[] arr = new int[]{0, 1, 3, 50, 75};
        int low = 0;
        int up = 99;
        List<String> rst = findMissingRange(arr, low, up);
        System.out.print(rst.toString());
    }

    private static List findMissingRange(int[] num, int low, int up) {
        // 定义返回值
        List rst = new ArrayList<>();

        //处理异常
        if (num == null || num.length == 0) {
            rst.add(low + "->" + up);
            return rst;
        }
        
        //do business
        //处理首
        doAdd(rst, low, num[0]);

        //处理中间
        int i = 1;
        int pre = num[0];

        while (i < num.length) {
            int cur = num[i];
            if (cur != pre + 1) {
                doAdd(rst, pre, cur);
            }
            i++;
            pre = cur;
        }

        //处理尾巴
        doAdd(rst, num[num.length - 1], up + 1);
        return rst;
    }

    private static List doAdd(List rst, int low, int up) {
        //拼接
        if (low < up && (low + 1) < up) {
            if ((up - low) == 2) {
                rst.add((low + 1));
            } else {
                rst.add((low + 1) + "->" + (up - 1));
            }
        }
        return rst;
    }
}
```

时间：2019年9月8日   
地点：百草园   
天气：雷雨   
心情：平静...   
