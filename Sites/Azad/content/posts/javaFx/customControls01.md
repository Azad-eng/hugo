---
title: "JavaFx自定义控件（一）- 使用CSS重新设置现有控件的样式"
tags: [ "controls", "JavaFx", "自定义控件"  ]
categories:
  - "JavaFx"
date: 2022-06-06
lastmod: 2022-06-13
draft: false
---

### 前言
{{< admonition type=question title="为什么需要自定义控件？"  >}}
* JavaFx中很多内置控件的代码都是私有的或最终的，因此无法访问或覆盖它们的内部结构，当涉及到个性化的需求时，就会束手无策
* 当需要定制一个全新的符合项目需求的组件，而JavaFX中内置的组件无法满足需求
  {{< /admonition >}}
  
### 导航
**创建自定义控件的不同方法**
* [使用CSS重新设置现有控件的样式](http://localhost:1313/customcontrols01/)
* [组合现有控件来创建自定义控件](http://localhost:1313/customcontrols02/)
* [扩展现有控件](http://localhost:1313/customcontrols03/)
* [使用Control + Skin类](http://localhost:1313/customcontrols04/)
* [使用Region类](http://localhost:1313/customcontrols05/)
* [使用Canvas类](http://localhost:1313/customcontrols06/)

### 具体实现
**步骤1：在 modena.css 文件中获取指定控件的css样式**

[modena.css](https://raw.githubusercontent.com/openjdk/jfx/master/modules/javafx.controls/src/main/resources/com/sun/javafx/scene/control/skin/modena/modena.css) 是包含JavaFx内置的每个控件的所有css样式的文件，其前身是 caspian.css，但进行了许多优化。

【:> [JFoenix-css](https://github.com/sshahine/JFoenix/tree/master/jfoenix/src/main/resources/com/jfoenix/assets/css)是包含JFoenix包里每个控件的所有css样式的文件，是对JavaFx控件的扩展。】

**步骤2：复制指定控件样式至新的css文件中**

**步骤3：修改JavaFX CSS**

如果不熟悉 JavaFX 中使用的 CSS 变体，可以说看看[JavaFX CSS 参考指南](https://openjfx.io/javadoc/15/javafx.graphics/javafx/scene/doc-files/cssref.html#popupwindow) 。
它是一个非常强大的工具，可以设置/重新设置控件的样式。原则上，它与 web CSS 非常相似，除了它基于 CSS 2.1，所有属性都以“-fx-”为前缀，并且它有一些特殊的东西，比如对变量的内置支持等。

**步骤4：覆盖项目中的控件样式**

`方法A: 样式表覆盖`

样式表从 Scene 对象的 getStylesheets 属性中指定的 URL 加载，比如 

```java
scene.getStylesheets().add(ClassNameHere.class.getResource("/styles/restyled.css").toExternalForm());

```

`方法B：内联样式覆盖`

内联样式通过在fxml文件首行添加 Node setStyle API 指定，比如
```java
stylesheets="@../../styles/restyled.css"
```
{{< admonition >}}
**样式选择器的优先顺序：**
内联样式>场景样式表>用户代理样式表(默认)

`样式选择器的优先顺序可以在样式声明中使用“！important”进行修改`
{{</ admonition >}}

### 实例
{{< image src="/images/java/2022/restyled.png" caption="成果"  >}}
```css
 * Here are the things we need to do...
 *
 * 1.No gradients which leads to a more flat ui
 * 2.Different kind of checkmark
 * 3.Background filled when selected
 * 4.Different focus indicator

/** 在组件CheckBox的css默认类check-box上做改动 **/
.check-box {
    -material-design-color               : #3f51b5;
    -material-design-color-transparent-12: #3f51b51f;
    -material-design-color-transparent-24: #3f51b53e;
    -material-design-color-transparent-40: #3f51b566;
    -fx-font-family  : "Arial"; /* Roboto Regular */
    -fx-font-size    : 13px;
    -fx-label-padding: 0em 0em 0em 1.1em;
    -fx-text-fill    : -fx-text-background-color;
}
.check-box > .box {
    -fx-background-color  : transparent;
    -fx-background-insets : 0;
    -fx-border-color      : #0000008a;
    -fx-border-width      : 2px;
    -fx-border-radius     : 2px;
    -fx-padding           : 0.083333em; /* 1px */
    -fx-text-fill         : -fx-text-base-color;
    -fx-alignment         : CENTER;
    -fx-content-display   : LEFT;
}
.check-box:hover > .box {
    -fx-background-color  : #61616110, transparent;
    -fx-background-insets : -14, 0;
    -fx-background-radius : 1024;
    -fx-cursor            : hand;
}
.check-box:focused > .box {
   /* -fx-background-color  : #6161613e, transparent;*/
    -fx-background-insets : -14, 0;
    -fx-background-radius : 1024;
}
.check-box:pressed > .box {
    -fx-background-color  : -material-design-color-transparent-12, transparent;
    -fx-background-insets : -14, 0;
    -fx-background-radius : 1024;
}
.check-box:selected > .box {
    -fx-background-color  : -material-design-color;
    -fx-background-radius : 2px;
    -fx-background-insets : 0;
    -fx-border-color      : transparent;
}
.check-box:selected:hover > .box {
    -fx-background-color  : -material-design-color-transparent-12, -material-design-color;
    -fx-background-insets : -14, 0;
    -fx-background-radius : 1024, 2px;
    -fx-border-color      : transparent;
    -fx-cursor            : hand;
}
.check-box:selected:focused > .box {
    -fx-background-color  : -material-design-color-transparent-24, -material-design-color;
    -fx-background-insets : -14, 0;
    -fx-background-radius : 1024, 2px;
    -fx-border-color      : transparent;
}
.check-box:disabled {
    -fx-opacity: 0.46;
}
.check-box > .box > .mark {
    -fx-background-color: null;
    -fx-padding         : 0.45em;
    -fx-scale-x         : 1.1;
    -fx-scale-y         : 0.8;
    -fx-shape           : "M-0.25,6.083c0.843-0.758,4.583,4.833,5.75,4.833S14.5-1.5,15.917-0.917c1.292,0.532-8.75,17.083-10.5,17.083C3,16.167-1.083,6.833-0.25,6.083z";
}
.check-box:indeterminate:hover > .box {
    cursor:hand;
}
.check-box:indeterminate > .box {
    -fx-background-color  : -material-design-color-transparent-40;
    -fx-background-radius : 2px;
    -fx-background-insets : 0;
    -fx-border-color      : transparent;
}
.check-box:indeterminate  > .box > .mark {
    -fx-background-color: rgba(255, 255, 255, 0.87);
    -fx-shape           : "M0,0H10V2H0Z";
    -fx-scale-shape: false;
    -fx-padding    : 0.666667em;
}
.check-box:selected > .box > .mark {
    -fx-background-color : rgba(255, 255, 255, 0.87);
    -fx-background-insets: 0;
}

/** 新建一个css类switch覆盖组件CheckBox的css默认类check-box **/
.switch {
    -material-design-color               : #3f51b5;
    -material-design-color-transparent-12: #3f51b51f;
    -material-design-color-transparent-24: #3f51b53e;
    -material-design-color-transparent-40: #3f51b566;

    -fx-font-family  : "Arial";
    -fx-font-size    : 13.0px;
    -fx-label-padding: 0em 0em 0em 1.1em;
    -fx-text-fill    : -fx-text-background-color;
}
.switch > *.box {
    -fx-background-color : #00000066;
    -fx-pref-height      : 20;
    -fx-pref-width       : 40;
    -fx-background-radius: 1024px;
    -fx-background-insets: 2.5;
    -fx-padding          : 0;
}
.switch:selected > *.box {
    -fx-background-color: -material-design-color-transparent-40;
}
.switch:disabled > *.box {
    -fx-background-color: #0000001f;
}
.switch > *.box > *.mark {
    -fx-background-color : fafafa;
    -fx-padding          : 0;
    -fx-background-insets: 0 10 0 10;
    -fx-background-radius: 1024px;
    -fx-translate-x      : -8px;
    -fx-effect           : dropshadow(gaussian, rgba(0, 0, 0, 0.3), 4.0, 0.5, 0.0, 1);
}
.switch:hover > *.box > *.mark {
    -fx-background-color : #61616110, white;
    -fx-background-insets: -14 -4 -14 -4, 0 10 0 10;
    -fx-background-radius: 1024px, 1024px;
    -fx-effect           : dropshadow(gaussian, rgba(0, 0, 0, 0.3), 4.0, 0.2, 0.0, 1);
}
.switch:selected:hover > *.box > *.mark {
    -fx-background-color : -material-design-color-transparent-12, -material-design-color;
    -fx-background-insets: -14 -4 -14 -4, 0 10 0 10;
    -fx-background-radius: 1024px, 1024px;
    -fx-effect           : dropshadow(gaussian, rgba(0, 0, 0, 0.3), 4.0, 0.2, 0.0, 1);
}
.switch:selected > *.box > *.mark {
    -fx-background-color : -material-design-color;
    -fx-background-insets: 0 10 0 10;
    -fx-background-radius: 1024px;
    -fx-translate-x      : 8px;
}
.switch:focused > *.box > *.mark {
    -fx-background-color : #6161613e, white;
    -fx-background-insets: -14 -4 -14 -4, 0 10 0 10;
    -fx-background-radius: 1024px, 1024px;
    -fx-effect           : dropshadow(gaussian, rgba(0, 0, 0, 0.3), 4.0, 0.2, 0.0, 1);
}
.switch:selected:focused > *.box > *.mark {
    -fx-background-color : -material-design-color-transparent-24, -material-design-color;
    -fx-background-insets: -14 -4 -14 -4, 0 10 0 10;
    -fx-background-radius: 1024px, 1024px;
    -fx-effect           : dropshadow(gaussian, rgba(0, 0, 0, 0.3), 4.0, 0.2, 0.0, 1);
}
.switch:indeterminate > *.text {
    -fx-fill: -required-text-color;
}
.switch:indeterminate > *.box > *.mark {
    -fx-background-color : transparent;
    -fx-border-color     : -material-design-color;
    -fx-border-radius    : 1024px;
    -fx-border-insets    : 0 10 0 10;
    -fx-border-width     : 3;
    -fx-translate-x      : 0;
}
.switch:disabled > *.box > *.mark {
    -fx-background-color: bdbdbd;
}
.switch:disabled {
    -fx-opacity: 1;
}
```
[——>完整代码参考<——](https://github.com/Azad-eng/custom/blob/master/src/main/java/com/ryl/custom/restyleCss)

### 扩展 ###
{{<admonition type=question title="如何将自定义控件导入场景生成器？" >}}
`要将自定义控件导入 SceneBuilder，需要从控件创建一个 jar 并将此 jar 添加到 Library 文件夹。在 SceneBuilder中有一个可以单击的小齿轮图标。它的弹出菜单有一个名为“自定义库文件夹”的条目，其中有一个条目可以让您在资源管理器或查找器（取决于您的操作系统）中打开库文件夹。如果将创建的 jar 复制到此文件夹中，组件应出现在 Scene Builder 的“自定义”选项卡中。`
  {{< /admonition >}}

### 参考链接
* [Custom Controls in JavaFX (Part I)](https://foojay.io/today/custom-controls-in-javafx-part-i/)
* [HanSolo/JavaFXCustomControls(github)](https://github.com/HanSolo/JavaFXCustomControls)
* [JavaFX CSS 参考指南(API)](https://openjfx.io/javadoc/15/javafx.graphics/javafx/scene/doc-files/cssref.html)