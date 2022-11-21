---
title: "ControlFlow"
description: 流程控制图｛顺序｝｛分支｝｛循环｝｛break｝{continue}
tags: [ "java基础", "程序流程图" ]
categories:
  - "Java基础"
  - "知识汇总"
date: 2022-05-31T23:23:18+08:00
draft: false
---

<!--more-->
### 顺序控制
- **程序从上到下逐行执行，中间没有任何判断和跳转**

### 分支控制

- **让程序有选择的执行**
{{< image src="/images/java/2022/分支控制-if.png" caption="分支控制 (`if`)"  >}}
{{< image src="/images/java/2022/分支-switch.png" caption="分支控制 (`switch`)"  >}}

### 循环控制

- **满足条件则程序循环执行，通过变量迭代，等到条件无法满足时就退出循环。因此引入合适的迭代的变量可以有效的规避死循环**
{{< image src="/images/java/2022/循环控制.png" caption="循环控制"  >}}

### 循环跳转控制

- **break[用于终止某个语句块的执行，一般用于switch分支控制或者循环控制程序中]**
{{< image src="/images/java/2022/break.png" caption="循跳转环控制 (`break`)"  >}}

- **continue[用于结束本次循环，继续执行下一次循环]**
{{< admonition>}}
1. 注意不同于break，**continue**语句执行前一定要有**迭代变量操作**，否则就会陷入死循环
2. for循环比较特殊，虽然**for循环内的迭代变量的执行顺序的确在循环体操作之后**，但是它不在循环体内，所以continue没有跳过它，执行完continue语句后，会接着执行迭代变量操作，再执行条件判断
{{< /admonition >}}

{{< image src="/images/java/2022/continue.png" caption="循跳转环控制 (`continue`)"  >}}

- **return[表示让程序跳出当前所在的方法，然后继续执行]**
