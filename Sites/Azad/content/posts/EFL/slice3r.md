---
title: "Slice3r参数设置——打印设置参数"
tags: [ "3d打印", "Slice3r", "参数定义"  ]
categories:
  - "工作总结"
date: 2022-08-24T11:16:38+08:00
lastmod: 2022-08-30T18:16:38+08:00
draft: true
---

### 导航
**创建自定义控件的不同方法**
* [Slice3r参数设置——打印设置参数](http://localhost:1313/slice3r01/)

### 使用Control + Skin的实现类来自定义全新的组件
`Control 将包含所有逻辑，而 Skin 将包含所有 UI 相关代码`

**步骤1：通过[矢量绘图程序](https://helpx.adobe.com/uk/illustrator/using/css-extraction.html)绘制好组件**
{{< image src="/images/java/2022/ledSvg.png" caption="演示">}}

**步骤2：将矢量图形转换成css代码**

**步骤3：创建程序，引用css**

### 示例demo
{{< admonition example >}}
 **自定义全新组件示例** ：展示如何使用 Control 和 Skin 类在 JavaFX 中创建自定义控件—LED开关

注意：
`由 Control 和 Skin 类创建的自定义控件只有在您为控件提供多个皮肤或者您希望让人们能够为您的控件创建自己的皮肤时才有意义。在任何其他情况下，您应该选择另一种方法（例如，使用基于区域或画布的控件）。因此，通常在 UI 库中使用 Control 和 Skin 方法，其中您有一个 Control 和多个 Skin。`
 {{< /admonition >}}

{{< image src="/images/java/2022/led-green.png" caption="LED"  >}}

### Control
{{<admonition type="info" title="前言">}}
1. 控件需要两个属性，第一个应该定义 LED 的状态，第二个应该定义 LED 的颜色;

2. 众所周知，我们有能力在 JavaFX 中使用 CSS 来设置控件样式，问题是我们如何将 CSS 属性链接到我们在代码中定义的属性？
答案是使用所谓的 StyleableProperties。这个属性有一个指向 CSS 属性的链接，这意味着如果我们加载一个覆盖例如 -color 属性的 CSS 文件，它将触发我们在代码中定义的属性。这很棒，因为我们可以通过在代码中调用 setColor() 方法或加载覆盖 -color 属性的 CSS 文件来更改属性；

3. 最后，我们需要一个 BooleanProperty 来表示控件的状态。为此，我们还可以利用 JavaFX 中的 CSS 特性，即 CSS PseudoClass。这可以看作是一个布尔开关，如果在 CSS 中触发，则可用于为真/假状态定义单独的样式。
{{</admonition>}}

{{<admonition type="bug" title="总结">}}
1. 通过css样式属性定义了一个名为COLOR的 CssMetaData 对象，而这个对象又定义了将在 CSS 中使用的属性-color；

2. 再去css文件中定义这个属性，比如紫色部分：

3. PseudoClass ON_PSEUDO_CLASS 定义了到 CSS 伪类“on”的链接，为了使用它，我们通过调用 pseudoClassStateChanged(ON_PSEUDO_CLASS.get()) 在 state 属性的 invalidated() 方法中触发它;

4. 为了完成这项工作，我们还需要 CSS 文件中的 on 伪类。请记住，主要 LED 部分（绿色部分）是在 LED 亮起时应从深绿色渐变变为浅绿色渐变的部分。下面是实现此效果所需的 CSS 代码：

5. 我们定义了一个具有 LED 和 SWITCH 的枚举 SkinType，它们将在 getUserAgentStyleSheet() 方法中使用。根据 skinType 变量加载不同的样式表;

6. 默认在 custom-control.css 中定义 LED样式，在 switch.css 中定义开关样式。
{{</admonition>}}

### 填充
#### 填充密度
#### 填充图案
Slic3r 提供了几种填充模式、四种常规和三种特殊样式。每个图下方括号中给出的数字是对简单 20 毫米立方体模型所用材料和所用时间的粗略估计。请注意，这只是指示性的，因为模型复杂性和其他因素会影响时间和材料：
{{< image src="/images/EFL/2022/infill.png" caption=""  >}}
某些模型类型更适合特定模式，例如有机类型与机械类型。该图显示了蜂窝状填充物如何更好地适合该机械部件，因为每个六边形与每一层的相同底层图案结合，形成一个强大的垂直结构：
{{< image src="/images/EFL/2022/complex.png" caption=""  >}}
不同密度的填充图案。从左到右：20%、40%、60%、80%
{{< image src="/images/EFL/2022/infills.png" caption=""  >}}
{{<admonition type="tip" title="填充总结">}}

1. 选择填充图案时有几个考虑因素：物体强度、时间和材料、个人喜好。可以推断，更复杂的模式将需要更多的动作，因此需要更多的时间和材料;

2. 大多数模型只需要低密度填充物，因为提供超过 50% 的填充物将产生非常紧密的模型。出于达到节省材料和打印时间的目的，常见的图案范围在 `10% ~ 30%` 之间，但是模型的要求将决定哪种密度最好。

{{</admonition>}}

### 参考链接
* [Slice3r Manual - Print Settings](https://manual.slic3r.org/expert-mode/print-settings)