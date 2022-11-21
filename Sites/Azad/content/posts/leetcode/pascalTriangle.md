---
title: "PascalTriangle"
description: 二维列表的运用——杨辉三角
tags: [ "二维列表", " LeetCode-Easy" , "ArrayList"]
categories:
  - "LeetCode"
date: 2022-06-01T03:27:55+08:00
draft: false
---
<!--more--> 
```java
package efl.ryl.easy;

import java.util.ArrayList;
import java.util.List;

/**
 * Author: Azad-eng
 * Date: 2022/3/3
 * Description: 杨辉三角
 */
public class Pascal_Triangle {
//              [1]
//             [1,1]
//            [1,2,1]
//           [1,3,3,1]
//          [1,4,6,4,1]
//         [............]
    
    public List<List<Integer>> generate(int numRows) {
        //二维列表的运用——杨辉三角
        /**
         * 思路分析：
         * 1.主集合List<List<Integer>> ret有且仅有一个，但子集合List<Integer> row 有numRows个(numRows=ret.size()),因此要new创建ret.size()个row
         * -所以 for (int i = 0; i < numRows; ++i) {}
         * 2.row集合中元素的add规律：第1行，add 1次，第2行，add 2次...第numRows行，add numRows次
         * -所以 for (int j = 0; j <= i; ++j){}
         * 3.add的值也有规律：
         * -3.1 每一行头尾(第一个行头尾重合)的值都是1：row.get(0)=1，row.get(i)=1
         * -所以if (j == 0 || j == i) {row.add(1);}
         * -3.2 非头尾的值 =上一行的相同位置的前一个元素的值 +上一行的相同位置的元素的值
         * -所以 row.add(ret.get(i-1).get(j-1) + ret.get(i-1).get(j))
         */
        //List是接口，不能实例化,因此List<List<Integer>> list = new List<>(); x
        List<List<Integer>> ret = new ArrayList<>();
        for (int i = 0; i <= numRows - 1; i++) {
            List<Integer> row = new ArrayList<>();
            for (int j = 0; j <= i; j++) {
                if (j == 0 || j == i) {
                    row.add(1);
                } else {
                    row.add(ret.get(i - 1).get(j - 1) + ret.get(i - 1).get(j));
                }
            }
            ret.add(row);
        }
        return ret;
    }

    public static void main(String[] args) {
        System.out.println(new Pascal_Triangle().generate(5));
    }
}

```
