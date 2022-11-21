---
title: "Maven-Pom"
description: "通过配置maven的pom文件来自定义打包jar"
tags: [ "pom配置", "maven" ]
categories:
  - "工作总结"
date: 2022-06-01T03:34:35+08:00
lastmod: 2022-07-01
draft: false
---
<!--more-->
### 方法一：使用 maven-jar-plugin 和 maven-dependency-plugin
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.***</groupId>
    <artifactId>***</artifactId>
    <version>1.2.n</version>
    <description>*** Printer</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.0.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <java.version>8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
         <!-- 其他的一些dependency... -->
    </dependencies>

    <build>
        <plugins>
         <!--该插件的作用是配置mainClass，classpath-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.1.2</version>
                <configuration>
                    <archive>
                        <manifest>
                        <!--是否在manifest文件中添加classpath。默认为false。如果为true，则会在manifest文件中添加classpath-->
                            <addClasspath>true</addClasspath>
                            <!--classpath的前缀，比如libs/xx.jar，所有外部依赖jar包所在的文件夹名libs+jar包共同组成了类路径Class-Path-->
                            <classpathPrefix>libs/</classpathPrefix>
                            <!--这里改成你主类的全限定名称,mainClass = 启动时的Main Class-->
                            <mainClass>
                                com.efl.javafx.desktop.ClientApplication
                            </mainClass>
                        </manifest>
                    </archive>
                    <classifier>bak</classifier>
                </configuration>
                <!--在执行package动作的时候，自动打包-->
                <executions>
                    <execution>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <!-- 该插件的作用是把所依赖的jar包copy到指定目录outputDirectory -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>3.3.0</version>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                          <!--outputDirectory中的类路径要与maven-jar-plugin中的classpath一致-->
                            <outputDirectory>
                                E:/packet/***/1-2-8/libs
                            </outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <!-- 其他的一些plugin... -->
        </plugins>
    </build>
</project>
```
{{< admonition type=info title="打包结果" open=false >}}
#### ├── PotatoOSp-Client-1.2.n-bak.jar
#### ├── libs
		├── spring-boot-starter.jar
		├── 其它jar
{{< /admonition >}}

{{< admonition type=success title="优点"  >}}
有诸多配置项，很自由，每个步骤都可控，然后生成的jar包是轻量级的，大小就是我们自己程序的大小
{{< /admonition >}}

{{< admonition type=bug title="缺点"  >}}
最终jar包中没有所依赖的jar包。依赖跟自己的代码不在一个jar包中。部署或者移动的时候，要考虑到多个文件，比较麻烦
{{< /admonition >}}

### 方法二：使用spring-boot-maven-plugin

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <!-- 工程主入口-->
        <classifier>spring-boot</classifier>
        <mainClass>
            com.***.javafx.desktop.ClientApplication
        </mainClass>
        <includeSystemScope>true</includeSystemScope>
        <addResources>false</addResources>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
{{< admonition type=success title="优点"  >}}
能同时打可执行jar包和war包，所有依赖都在可执行jar包中，可以方便的在任何位置都能直接运行
{{< /admonition >}}

{{< admonition type=bug title="缺点"  >}}
添加了一些不必要的Spring和Spring Boot依赖
{{< /admonition >}}


### 参考链接：
* [Maven - 打包可执行jar包](https://www.jianshu.com/p/0d85d0539b1a#%E6%96%B9%E6%B3%95%E4%B8%80%E4%BD%BF%E7%94%A8maven-jar-plugin%E5%92%8Cmaven-dependency-plugin)