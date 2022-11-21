---
title: "IoModel"
tags: [ "IO流",  "客户端与服务端"  ]
categories:
  - "Java基础"
  - "网络通讯"
date: 2022-06-01T15:16:47+08:00
lastmod: 2022-06-01T15:16:47+08:00
draft: false
---
{{< admonition type=abstract  title=" BIO和NIO的不同"  >}}
- BIO以流的方式处理数据，而NIO以块的方式处理数据，后者效率高很多
- BIO是阻塞的，而NIO是非阻塞的
- BIO是基于字节流和字符流进行数据操作，且只能单向操作，即要么读取inputStream要么写出outputStream，而NIO基于Channel(通道)和Buffer(缓冲区)进行数据操作的，Buffer既可以读取get()也可以写入数据put()，只需通过flip进行读写切换即可,Channel也是双向的，可以返回底层OS的情况
- BIO一个线程只能监听一个客户端，而BIO的Selector(选择器)可以用于监听多个通道的事件(比如：连接请求),因此使用单个线程就可以监听多个客户端通道
{{< /admonition >}}

{{< image src="/images/java/2022/io模型.png" caption="IoModel (`nio&bio`)"  >}}

### 演示BIO服务端线程池机制
```java
package com.efl.bio;

import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Author: Azad-eng
 * Date: 2022/2/26
 * Description: 演示线程池机制
 */
public class BioServer {
    public static void main(String[] args) throws IOException {
        /**
         * 思路分析：
         * 1.创建一个线程池
         * 2.如果有客户端连接，就创建一个线程与之通信(单独写一个方法)
         */
        ExecutorService threadPool = Executors.newCachedThreadPool();
        ServerSocket serverSocket = new ServerSocket(6666);
        while (true) {
            //监听，等待客户端连接,得到一个不可更改的Socket
            //如果没有连接会堵塞在这里
            System.out.println("server has started, accepting...");
            Socket socket = serverSocket.accept();
            System.out.println("连接到一个client了！");
            //执行线程池里new出来的线程，重写里面的run方法
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    //这里就可以和客户端通信了，单独写一个通信的方法，把客户端连接上的socket传进去(没有这个socket无法进行通信)
                    //调用该方法handler(socket)进行通信
                    //测试线程信息
                    System.out.println( "启动了线程:id=" + Thread.currentThread().getId() + " name=" +
                            Thread.currentThread().getName() + "\n可以进行通信了！");
                    handler(socket);
                }
            });
        }
    }

    public static void handler(Socket socket) {
        try {
            //建立一个容量为1024的字节数组来接收数据
            byte[] bytes = new byte[1024];
            InputStream inputStream = socket.getInputStream();
            //持续读取client发送的数据(该数据被装在了bytes字节数组中)
            while (true) {
                //read()方法返回的是接收的data的下一个byte(类型为int),如果返回-1代表inputStream到底了
                //与NIO不同的是：如果服务端没有接收到客户端发出的数据线程会堵塞在这里
                System.out.println("reading...");
                int read = inputStream.read(bytes);
                if (read != -1) {
                    //把读取到的数据(字节数组——>字符串)print出来
                    System.out.println(new String(bytes, 0, read));
                } else {
                    //程序执行到这里代表数据读取到底了，那么就退出while循环，接着关闭流，退出通信
                    break;
                }

            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                System.out.println("线程:id=" + Thread.currentThread().getId() + " name=" +
                Thread.currentThread().getName() +"关闭了和server的连接");
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
}
```

### 演示NIO核心组件之一Buffer的使用
```java
package com.efl.nio;

import java.nio.IntBuffer;

/**
 * Author: Azad-eng
 * Date: 2022/2/26
 * Description:演示NIO核心组件之一Buffer的使用
 */
public class BasicBuffer {
    public static void main(String[] args) {
        /**
         * 思路分析：
         * 1.创建一个IntBuffer,容量为5个int
         * 2.向里面存放数据10,11,12,13,14
         * 3.从里面读取数据
         */
        IntBuffer intBuffer = IntBuffer.allocate(5);
        //存放数据,执行该方法一次，position在底层会自动+1
        intBuffer.put(10);
        //循环存放数据(不能超过intBuffer的容量限制),前面已经存放了2个int了，所以后面intBuffer.capacity()-2
        for (int i = 0; i < intBuffer.capacity() - 2; i++) {
            intBuffer.put(i + 12);
        }
        //如何从intBuffer中读取数据？
        //读写切换(这里需要：写——>读)
        intBuffer.flip();
        /**
         * 常用方法如下：
         */
        //将index的位置position设置为数组的第2位，即从第2位开始读，结果输出为11,12,13,14
        intBuffer.position(1);
        //设置能读取到的数组数据的最大限制(不能>=3个数据,即只能输出2个)，结果输出为11,12
        intBuffer.limit(3);

        //循环读取数据:
        while (intBuffer.hasRemaining()){
            //get()里面维护的是一个index，每get一次,索引就往右移动一次
            //一个int=2bytes
            // 10  11  12  13  14
            // ^   ^   ^   ^   ^
            // ————————————————————
            //| 0 | 2 | 4 | 6 | 8 |
            // ————————————————————
            //      ^
            //      |
            System.out.println(intBuffer.get());
        }
        //get(int index) 可以读取指定索引位置处的数据
        System.out.println(intBuffer.get(2));
    }
}
```

### 演示NIO用FileChannel对本地文件进行IO操作-1
```java
package com.efl.nio;

import java.io.*;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

/**
 * Author: Azad-eng
 * Date: 2022/2/26
 * Description:
 * 演示用FileChannel对本地文件进行IO操作案例1
 * 1.输出.txt到本地文件目录
 * 2.读取.txt文件并打印到控制台屏幕
 */
public class FileChannel01 {
    public static void main(String[] args) throws IOException {
//        new FileChannel01().writeToOut();
        new FileChannel01().readFromIn();

    }

    public void writeToOut() throws IOException {
        /**
         * 思路分析：
         * 1.创建.txt文件的内容和在本地文件目录中的文件路径
         * 2.创建一个文件输出流FileOutputStream
         * 3.通过FileOutputStream得到FileChannel
         * 4.创建缓冲区Buffer并分配容量空间
         * 5.将文件写入put到缓冲区
         * 6.再将缓冲区数据写入write到fileChannel中
         * 7.关闭输出流
         */
        String srcTxt = "hello world~";
        String dstPath = "F:\\linder\\IoTest\\hello01.txt";
        FileOutputStream fileOutputStream = new FileOutputStream(dstPath);
        FileChannel channel = fileOutputStream.getChannel();
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        byteBuffer.put(srcTxt.getBytes());
        byteBuffer.flip();
        //将缓冲区的数据写入到file通道中(byteBuffer write to fileChannel)
        channel.write(byteBuffer);
        fileOutputStream.close();
    }

    public void readFromIn() throws IOException {
        /**
         * 思路分析
         * 1.根据目标文件地址生成文件
         * 2.创建一个文件输入流,将文件数据放进去
         * 3.通过FileInputStream得到FileChannel
         * 4.创建缓冲区Buffer并指定容量空间大小为文件的大小
         * 5.再将缓冲区数据读取read到fileChannel中
         * 6.打印
         * 7.关闭输入流
         */
        String dstPath = "F:\\linder\\IoTest\\hello01.txt";
        File file = new File(dstPath);
        FileInputStream fileInputStream = new FileInputStream(file);
        FileChannel channel = fileInputStream.getChannel();
        ByteBuffer byteBuffer = ByteBuffer.allocate((int) file.length());
        channel.read(byteBuffer);
        byte[] array = byteBuffer.array();
        System.out.println(new String(array));
        fileInputStream.close();
    }
}
```

### 演示NIO用FileChannel对本地文件进行IO操作-2
```java
package com.efl.nio;

import java.io.*;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

/**
 * Author: Azad-eng
 * Date: 2022/2/26
 * Description:通过一个Buffer进行读写操作来完成对指定文件的拷贝
 */
public class FileChannel02 {
    public static void main(String[] args) throws IOException {
        /**
         * 思路分析
         * 1.得到copyFrom文件的文件输入流
         * 2.通过文件输入流得到FileChannel
         * 3.创建缓冲区
         * 4.循环的一边读取copyFrom的channel中的数据一边将数据写入到copyTo的channel中
         * 5.关闭文件输入输出流
         */
        File file = new File("F:\\linder\\IoTest\\hello01.txt");
        FileInputStream fileInputStream = new FileInputStream("F:\\linder\\IoTest\\hello01.txt");
        FileChannel channelIn = fileInputStream.getChannel();
        FileOutputStream fileOutputStream = new FileOutputStream("F:\\linder\\IoTest\\hello02.txt");
        FileChannel channelOut = fileOutputStream.getChannel();
        ByteBuffer byteBuffer = ByteBuffer.allocate((int)file.length());
        while (true){
            /**
             * 注意!!!
             * 这里要将缓冲区重置一下，否则当文件读取到最后时，position=limit=0，然后一直都是read=0，一直循环读取，始终无法读取完数据
             * positon = 0;
             * limit = capacity;
             * mark = 1;
             */
            byteBuffer.clear();
            int read = channelIn.read(byteBuffer);
            System.out.println(read);
            //当read = -1 表示channelIn中的数据已经被读完了，所以要退出读写循环中
            if (read==-1){
                break;
            }
            //翻转byteBuffer的功能，开始将byteBuffer的数据写入到channelOut中
            byteBuffer.flip();
            channelOut.write(byteBuffer);
        }
        fileInputStream.close();
        fileOutputStream.close();
    }
}
```