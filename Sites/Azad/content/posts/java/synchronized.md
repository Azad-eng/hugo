---
title: "关键字Synchronized的运用"
tags: [ "Synchronized关键字", "Java同步代码" ]
categories:
  - "Java多线程与并发"
date: 2022-06-02T13:26:32+08:00
lastmod: 2022-06-02T13:26:32+08:00
draft: false
---
{{< admonition type=tip title="技巧：带着问题去学习与理解Synchronized" open=false >}}
* Synchronized可以作用在哪里?分别通过对象锁和类锁进行举例。 
* Synchronized本质上是通过什么保证线程安全的? 分三个方面回答：加锁和释放锁的原理，可重入原理，保证可见性原理。
* Synchronized由什么样的缺陷?  Java Lock是怎么弥补这些缺陷的。 
* Synchronized和Lock的对比，和选择? 
* Synchronized在使用时有何注意事项? 
* Synchronized修饰的方法在抛出异常时,会释放锁吗? 
* 多个线程等待同一个snchronized锁的时候，JVM如何选择下一个获取锁的线程? 
* Synchronized使得同时只有一个线程可以执行，性能比较差，有什么提升的方法? 
* 我想更加灵活的控制锁的释放和获取(现在释放锁和获取锁的时机都被规定死了)，怎么办? 
* 什么是锁的升级和降级? 什么是JVM里的偏斜锁、轻量级锁、重量级锁? 
* 不同的JDK中对Synchronized有何优化?
  {{< /admonition >}}

### Synchronized的使用场景
**synchronized最常用于多线程并发编程时线程的同步**。
synchronized可以修饰普通方法，静态方法和代码块。当synchronized修饰一个方法或者一个代码块的时候，它能够保证在同一时刻最多只有一个线程执行该段代码。
* 对于普通同步方法，锁是当前实例对象（不同实例对象之间的锁互不影响）。
* 对于静态同步方法，锁是当前类的Class对象。
* 对于同步代码块，锁是Synchonized括号里配置的对象。
当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。
#### 对象锁
**应用于普通方法和代码块的同步**
#####   # 同步普通方法
```java
public class UseSynchronized implements Runnable {

    static UseSynchronized instance = new UseSynchronized();

    @Override
    public void run() {
        invokeMe();
    }

    synchronized public void invokeMe(){
        System.out.println("线程" + Thread.currentThread().getName() + "开始");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("线程" +Thread.currentThread().getName() + "结束");
    }

    public static void main(String[] args) {
        //启动的线程数
        int threadNumbers = 4;
        for (int i = 0; i < threadNumbers; i++) {
            Thread thread = new Thread(instance);
            thread.start();
        }
    }
}
```
{{< admonition type=info title="控制台输出：" open=false >}}
{{< image src="/images/java/2022/output02.png" caption="output" >}}
{{< /admonition >}}

{{< admonition >}}
synchronized修饰普通方法时，默认的锁就是this对象锁，即当前实例对象锁
{{< /admonition >}}
#####  # 同步代码块—this
`即默认锁对象为当前实例对象`
```java
public class UseSynchronized implements Runnable {

    static UseSynchronized instance = new UseSynchronized();
    
    @Override
    public void run() {
        //多个线程使用的锁是一样的,线程必须要等上一个线程释放锁后才能执行
        synchronized (this) {
            System.out.println("线程" + Thread.currentThread().getName() + "开始");
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程" +Thread.currentThread().getName() + "结束");
        }
    }

    public static void main(String[] args) {
        //启动的线程数
        int threadNumbers = 4;
        for (int i = 0; i < threadNumbers; i++) {
            Thread thread = new Thread(instance);
            thread.start();
        }
    }
}
```
{{< admonition type=info title="控制台输出：" open=false >}}
{{< image src="/images/java/2022/output02.png" caption="output" >}}
{{< /admonition >}}

{{< admonition >}}
this指当前实例对象，所以synchronized (this)只能在多个线程作用于同一个实例对象时才能发挥作用。
{{< /admonition >}}
#####  # 同步代码块—自定义锁 
`即自己手动创建锁：Object lock = new Object()` 
```java
/**
 * @author Azad-eng
 * Description:因为第二段代码块的锁与第一段代码块不一样，所以不会等到所有线程全部执行完第一段代码块后才执行第二段代码块
 */
public class UseSynchronized implements Runnable {

    static UseSynchronized instance = new UseSynchronized();

    //手动创建两把锁
    Object lock1 = new Object();
    Object lock2 = new Object();

    @Override
    public void run() {
        synchronized (lock1) {
            System.out.println("线程" + Thread.currentThread().getName() + "拿到lock1锁后开始");
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程" +Thread.currentThread().getName() + "释放lock1锁后结束");
        }
        synchronized (lock2) {
            System.out.println("线程" + Thread.currentThread().getName() + "拿到lock2锁后开始");
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程" +Thread.currentThread().getName() + "释放lock2锁后结束");
        }
    }

    public static void main(String[] args) {
        //启动的线程数
        int threadNumbers = 4;
        for (int i = 0; i < threadNumbers; i++) {
            Thread thread = new Thread(instance);
            thread.start();
        }
    }
}
```
{{< admonition type=info title="控制台输出：" open=false >}}
{{< image src="/images/java/2022/output01.png" caption="output" >}}
{{< /admonition >}}
#### 类锁
**应用于静态方法的同步或指定锁对象为class对象**
#####  # 同步静态方法
```java
public class UseSynchronized implements Runnable {

    @Override
    public void run() {
        invokeMe();
    }

    static synchronized public void invokeMe(){
        System.out.println("线程" + Thread.currentThread().getName() + "开始");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("线程" +Thread.currentThread().getName() + "结束");
    }

    public static void main(String[] args) {
        //启动的线程数
        int threadNumbers = 4;
        for (int i = 0; i < threadNumbers; i++) {
            Thread thread = new Thread(new UseSynchronized());
            thread.start();
        }
    }
}
```
{{< admonition type=info title="控制台输出：" open=false >}}
{{< image src="/images/java/2022/output02.png" caption="output" >}}
{{< /admonition >}}
{{< admonition >}}
不同于普通方法只能作用于同一实例对象，修饰静态方法的synchronized可以保证不同实例对象的线程也能在执行方法时得到同步
{{< /admonition >}}

#####  # 指定对象锁为class对象
```java
public class UseSynchronized implements Runnable {

    @Override
    public void run() {
        synchronized (UseSynchronized.class) {
            System.out.println("线程" + Thread.currentThread().getName() + "开始");
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程" + Thread.currentThread().getName() + "结束");
        }
    }

    public static void main(String[] args) {
        //启动的线程数
        int threadNumbers = 4;
        for (int i = 0; i < threadNumbers; i++) {
            Thread thread = new Thread(new UseSynchronized());
            thread.start();
        }
    }
}
```
{{< admonition type=info title="控制台输出：" open=false >}}
{{< image src="/images/java/2022/output02.png" caption="output" >}}
{{< /admonition >}}

### 参考链接：
* [Java 全栈知识体系之关键字: synchronized详解](https://pdai.tech/md/java/thread/java-thread-x-key-synchronized.html)
* [程序员自由之路之synchronized 的使用场景和原理简介 ](https://www.cnblogs.com/54chensongxia/p/11899031.html)