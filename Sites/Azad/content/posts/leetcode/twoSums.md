---
title: "TwoSums"
description: 返回和为目标值的两数的下标数组
tags: [ "哈希查找", " LeetCode-Easy" , "数组", "int[]"]
categories:
  - "LeetCode"
date: 2022-06-01T03:27:55+08:00
draft: false
---

<!--more--> 
```java
package efl.ryl.easy;

import java.util.*;

/**
 * Author: Azad-eng
 * Date: 2022/3/5
 * Description: 返回和为目标值的两数的下标数组
 */
public class TwoSums {
    public int[] method1(int[] nums, int target) {
        /**
         * 暴力解法——线性查找(完整的遍历一次数组的时间复杂度为——O[n])
         * 当i=0时，j={1,2...nums.length-1}即需要将i与j比较n-1次
         * 当i=1时，j={2,3...nums.length-1}即需要将i与j比较n-2次
         * ......
         * 当i=n-1时，j={nums.length-1}即需要将i与j比较1次
         * 一共比较了(n-1)+(n-2)+(n-3)+...1次
         * 等差序列求和：n*(n-1)/2 = 1/2 n^2 - 1/2n
         * 因此时间复杂度=O(n^2)
         */
        int[] ints = new int[2];
        for (int i = 0; i < nums.length; i++) {
            for (int j = 0; j < nums.length; j++) {
                if (i != j && (nums[j] == target - nums[i])) {
                    ints[0] = j;
                    ints[1] = i;
                    break;
                }
            }
        }
        return ints;
    }

    public int[] method2(int[] nums, int target) {
        /**
         * 优化——哈希查找
         * 时间复杂度=O(n)
         * [优势只有在数据量大时才能凸显]
         */
        if (nums == null || nums.length == 0) {
            return new int[0];
        }
        HashMap<Integer, Integer> map = new HashMap<>();
        Set<Integer> keySet = map.keySet();
        for (int i = 0; i < nums.length; i++) {
            map.put(i, nums[i]);
            if (map.containsValue(target - nums[i])) {
                for (Integer key : keySet) {
                    if (map.get(key).equals(target - nums[i]) && key != i) {
                        return new int[]{key, i};
                    }
                }
            }
        }
        return new int[0];
    }

    public static void main(String[] args) {
        System.out.println(Arrays.toString(new TwoSums().method2(new int[]{3, 2, 4}, 6)));
    }
}
```