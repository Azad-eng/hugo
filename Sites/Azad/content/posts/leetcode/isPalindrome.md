---
title: "IsPalindrome"
description: 判断字符串是否为回文串(正读反读一样的字符串，比如level,noon)
tags: [ "String", " LeetCode-Easy" ]
categories:
  - "LeetCode"
date: 2022-06-01T03:27:55+08:00
draft: false
---

<!--more--> 
```java
package efl.ryl.easy;
/**
 * Author: Azad-eng
 * Date: 2022/3/5
 * Description:
 * 判断字符串是否是回文串(正读反读一样的字符串，比如level,noon)
 */
public class IsPalindrome {
    public boolean method1(String srcString) {
        /**
         * 思路分析：
         * 方法1——筛选+判断
         * 1)遍历字符串srcString进行筛选(Character.isLetterOrDigit(srcString.charAt()))，只保留字母字符和数字字符
         * 2)将筛选后的字符串的每一个字符都转换成小写(Character.toLowerCase())并放进另一个新的字符串中desString(new->append),
         * 3)判断desString是否为回文串：
         * -使用翻转字符串的方法reverse得到把desString倒序读的字符串revString(注意这里要生成一个新的倒置的字符串，不能直接改变desString)
         * -将revString与desString进行相等判断,如果相等返回true，表示是回文串，反之false
         */
        StringBuffer desString = new StringBuffer();
        for (int i = 0; i < srcString.length(); i++) {
            if (Character.isLetterOrDigit(srcString.charAt(i))) {
                desString.append(Character.toLowerCase(srcString.charAt(i)));
            }
        }
//        StringBuffer reverse = desString.reverse(); x
//        desString是StringBuffer类的,保存的是字符串变量,不同于String,里面的值是可以更改的。为了避免这种更改,所以要新建一个StringBuffer
        StringBuffer revString = new StringBuffer(desString).reverse();
//        return revString.equals(desString); x
//        这里的revString和desString是两个object，这里equals判断的是两个对象是否是同一个对象，而不是判断它们的值是否相等
        return (new String(revString)).equals(new String(desString));
    }

    public boolean method2(String srcString) {
        /**
         * 思路分析
         * 方法2——双指针
         * 1)引入两个index变量int left=0 , int right = srcString.length()-1
         * 2)+————————————————————+
         *   |  |  |  |  |  |  |  |
         *   +————————————————————+
         *     ^                 ^
         *     |                 |
         *   left             right
         * 3)让left向右移动，right向左移动，一直循环到left>right,如果期间遇到非数字或字母的字符就让left++,或right--，即跳过它们
         * 4)对全部转成成小写的字符做出相等判断，一旦有一组字符不等，就返回false，如果循环结束依然没有返回false，代表该字符串是回文串，那么返回true
         */
        int left = 0;
        int right = srcString.length() - 1;
        while (left <= right){
            if(Character.toLowerCase(srcString.charAt(left))!=Character.toLowerCase(srcString.charAt(right))){
                return false;
            }
            left++;
            right--;
            if (!Character.isLetterOrDigit(srcString.charAt(left))){
                left++;
            }
            if (!Character.isLetterOrDigit(srcString.charAt(right))){
                right--;
            }
        }
        return true;
    }

    public static void main(String[] args) {
        System.out.println(new IsPalindrome().method2("l,ev e l"));
    }
}
```
