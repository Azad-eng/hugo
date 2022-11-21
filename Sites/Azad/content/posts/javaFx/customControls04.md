---
title: "JavaFx自定义控件（四）- 使用Control plus Skin的实现类来自定义全新的组件"
tags: [ "controls", "JavaFx", "自定义控件"  ]
categories:
  - "JavaFx"
date: 2022-06-08T11:16:38+08:00
lastmod: 2022-06-13T19:16:38+08:00
draft: false
---

### 导航
**创建自定义控件的不同方法**
* [使用CSS重新设置现有控件的样式](http://localhost:1313/customcontrols01/)
* [组合现有控件来创建自定义控件](http://localhost:1313/customcontrols02/)
* [扩展现有控件](http://localhost:1313/customcontrols03/)
* [使用Control plus Skin类](http://localhost:1313/customcontrols04/)
* [使用Region类](http://localhost:1313/customcontrols05/)
* [使用Canvas类](http://localhost:1313/customcontrols06/)

### 使用Control plus Skin的实现类来自定义全新的组件
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

```java
package controlskin;
import javafx.application.Application;
import javafx.geometry.Insets;
import javafx.scene.Scene;
import javafx.scene.layout.VBox;
import javafx.scene.paint.Color;
import javafx.stage.Stage;
import controlskin.CustomControl.SkinType;

/**
 * @author Gerrit Grunwald
 * Description: 全新自定义组件程序demo
 */
public class DemoControlSkinBased extends Application {
    private CustomControl ledControl;
    private CustomControl switchControl;

    @Override
    public void init() throws Exception {
        ledControl = new CustomControl();
        ledControl.setState(true);
        ledControl.setPrefSize(100, 100);
        ledControl.setColor(Color.LIME);

        switchControl = new CustomControl(SkinType.SWITCH);
        switchControl.setState(true);
        switchControl.setColor(Color.web("#4bd865"));
        switchControl.stateProperty().addListener((o, ov, nv) -> ledControl.setState(nv));
    }

    @Override
    public void start(Stage stage) throws Exception {
        VBox pane = new VBox(20, ledControl, switchControl);
        pane.setPadding(new Insets(20));
        Scene scene = new Scene(pane, 200, 200);
        scene.getStylesheets().add(DemoControlSkinBased.class.getResource("/styles.css").toExternalForm());
        stage.setTitle("Control-Skin based Control");
        stage.setScene(scene);
        stage.show();
    }

    @Override
    public void stop() throws Exception {
        super.stop();
    }
}
```
### Control
{{<admonition type="info" title="前言">}}
1. 控件需要两个属性，第一个应该定义 LED 的状态，第二个应该定义 LED 的颜色;

2. 众所周知，我们有能力在 JavaFX 中使用 CSS 来设置控件样式，问题是我们如何将 CSS 属性链接到我们在代码中定义的属性？
答案是使用所谓的 StyleableProperties。这个属性有一个指向 CSS 属性的链接，这意味着如果我们加载一个覆盖例如 -color 属性的 CSS 文件，它将触发我们在代码中定义的属性。这很棒，因为我们可以通过在代码中调用 setColor() 方法或加载覆盖 -color 属性的 CSS 文件来更改属性；

3. 最后，我们需要一个 BooleanProperty 来表示控件的状态。为此，我们还可以利用 JavaFX 中的 CSS 特性，即 CSS PseudoClass。这可以看作是一个布尔开关，如果在 CSS 中触发，则可用于为真/假状态定义单独的样式。
{{</admonition>}}
```java
package controlskin;
import javafx.beans.property.BooleanProperty;
import javafx.beans.property.BooleanPropertyBase;
import javafx.beans.property.ObjectProperty;
import javafx.css.*;
import javafx.scene.control.Control;
import javafx.scene.control.Skin;
import javafx.scene.paint.Color;
import java.util.List;

/**
 * @author Gerrit Grunwald
 * description：全新自定义的控件——实现功能：开关切换
 */
public class CustomControl extends Control {
    public enum SkinType{ LED, SWITCH }
    
    private static final StyleablePropertyFactory<CustomControl> FACTORY
            = new StyleablePropertyFactory<>(Control.getClassCssMetaData());
    /** css样式属性 **/
    private static final CssMetaData<CustomControl, Color> COLOR
            = FACTORY.createColorCssMetaData("-color", s -> s.color, Color.RED, false);
            
    /** css伪类 **/
    private static final PseudoClass ON_PSEUDO_CLASS = PseudoClass.getPseudoClass("on");       

    /** Properties **/
    private SkinType skinType;
    private BooleanProperty state;
    private final StyleableProperty<Color> color;

    /** 用户代理样式表 **/
    private static String defaultUserAgentStyleSheet;
    private static String switchUserAgentStyleSheet;

    /**
     * 构造器重载
     */
    public CustomControl() {
        this(SkinType.LED);
    }

    public CustomControl(final SkinType skinType){
        getStyleClass().add("custom-control");
        this.skinType = skinType;
        this.state = new BooleanPropertyBase(false){
            @Override
            protected void invalidated(){
                pseudoClassStateChanged(ON_PSEUDO_CLASS,get());
            }
            @Override
            public Object getBean() {
                return this;
            }
            @Override
            public String getName() {
                return "state";
            }
        };
        this.color = new SimpleStyleableObjectProperty<>(COLOR,this,"color");
    }

    /**
     * state
     */
    public boolean getState(){
        return state.get();
    }
    public void setState(final boolean state) {
        this.state.set(state);
    }
    public BooleanProperty stateProperty() {
        return state;
    }

    /**
     * color
     */
    public Color getColor() {
        return color.getValue();
    }
    public void setColor(final Color color){
        this.color.setValue(color);
    }
    public ObjectProperty<Color> colorProperty(){
        return (ObjectProperty<Color>) color;
    }

    /**
     * style related
     */
    @Override
    protected Skin<?> createDefaultSkin() {
        switch (skinType){
            case SWITCH: return new SwitchSkin(CustomControl.this);
            case LED:
            default: return new LedSkin(CustomControl.this);
        }
    }

    @Override
    public String getUserAgentStylesheet() {
        switch(skinType) {
            case SWITCH:
                if (null == switchUserAgentStyleSheet) {
                    switchUserAgentStyleSheet = CustomControl.class.getResource("/switch.css").toExternalForm();
                }
                return switchUserAgentStyleSheet;
            case LED   :
            default    :
                if (null == defaultUserAgentStyleSheet) {
                    defaultUserAgentStyleSheet = CustomControl.class.getResource("/custom-control.css").toExternalForm();
                }
                return defaultUserAgentStyleSheet;
        }
    }

    @Override
    public List<CssMetaData<? extends Styleable, ?>> getControlCssMetaData() {
        return FACTORY.getCssMetaData();
    }
}
```
{{<admonition type="bug" title="总结">}}
1. 通过css样式属性定义了一个名为COLOR的 CssMetaData 对象，而这个对象又定义了将在 CSS 中使用的属性-color；

2. 再去css文件中定义这个属性，比如紫色部分：
```CSS
.custom-control {
    -color: red;
}
```
3. PseudoClass ON_PSEUDO_CLASS 定义了到 CSS 伪类“on”的链接，为了使用它，我们通过调用 pseudoClassStateChanged(ON_PSEUDO_CLASS.get()) 在 state 属性的 invalidated() 方法中触发它;

4. 为了完成这项工作，我们还需要 CSS 文件中的 on 伪类。请记住，主要 LED 部分（绿色部分）是在 LED 亮起时应从深绿色渐变变为浅绿色渐变的部分。下面是实现此效果所需的 CSS 代码：

```CSS
.custom-control .main {
    -fx-background-color : linear-gradient(from 15% 15% to 83% 83%,
                                      derive(-color, -80%) 0%,
                                      derive(-color, -87%) 49%,
                                      derive(-color, -80%) 100%);
    -fx-background-radius: 1024px;
}
.custom-control:on .main {
    -fx-background-color: linear-gradient(from 15% 15% to 83% 83%,
                                     derive(-color, -23%) 0%,
                                     derive(-color, -50%) 49%,
                                    -color 100%);
}
```
5. 我们定义了一个具有 LED 和 SWITCH 的枚举 SkinType，它们将在 getUserAgentStyleSheet() 方法中使用。根据 skinType 变量加载不同的样式表;

6. 默认在 custom-control.css 中定义 LED样式，在 switch.css 中定义开关样式。
{{</admonition>}}

### Skin
#### LED
```java
package controlskin;
import javafx.beans.InvalidationListener;
import javafx.scene.control.SkinBase;
import javafx.scene.effect.BlurType;
import javafx.scene.effect.DropShadow;
import javafx.scene.effect.InnerShadow;
import javafx.scene.layout.Region;
import javafx.scene.paint.Color;

/**
 * @author Gerrit Grunwald
 * description：led皮肤
 */
public class LedSkin extends SkinBase<CustomControl> {
    private static final double PREFERRED_WIDTH = 16;
    private static final double PREFERRED_HEIGHT = 16;
    private static final double MINIMUM_WIDTH = 8;
    private static final double MINIMUM_HEIGHT = 8;
    private static final double MAXIMUM_WIDTH = 1024;
    private static final double MAXIMUM_HEIGHT = 1024;
    private double size;
    /** 组件 **/
    private Region frame;
    private Region main;
    private Region highlight;
    private InnerShadow innerShadow;
    private DropShadow glow;
    /** 自定义组件 **/
    private CustomControl control;
    /** 监听器 **/
    private final InvalidationListener sizeListener;
    private final InvalidationListener colorListener;
    private final InvalidationListener stateListener;

    /**
     * 构造器
     * @param control 自定义组件
     */
    public LedSkin(final CustomControl control) {
        super(control);
        this.control = control;
        sizeListener = o -> handleControlPropertyChanged("RESIZE");
        colorListener = o -> handleControlPropertyChanged("COLOR");
        stateListener = o -> handleControlPropertyChanged("STATE");
        initGraphics();
        registerListeners();
    }

    /**
     * 初始化
     */
    private void initGraphics() {
        if (Double.compare(control.getPrefWidth(), 0.0) <= 0 || Double.compare(control.getPrefHeight(), 0.0) <= 0 ||
                Double.compare(control.getWidth(), 0.0) <= 0 || Double.compare(control.getHeight(), 0.0) <= 0) {
            if (control.getPrefWidth() > 0 && control.getPrefHeight() > 0) {
                control.setPrefSize(control.getPrefWidth(), control.getPrefHeight());
            } else {
                control.setPrefSize(PREFERRED_WIDTH, PREFERRED_HEIGHT);
            }
        }
        frame = new Region();
        frame.getStyleClass().setAll("frame");
        main = new Region();
        main.getStyleClass().setAll("main");
        main.setStyle(String.join("", "-color: ",
                control.getColor().toString().replace("0x", "#"), ";"));
        innerShadow = new InnerShadow(BlurType.TWO_PASS_BOX,
                Color.rgb(0, 0, 0, 0.65), 8, 0, 0, 0);
        glow = new DropShadow(BlurType.TWO_PASS_BOX, control.getColor(), 20, 0, 0, 0);
        glow.setInput(innerShadow);
        highlight = new Region();
        highlight.getStyleClass().setAll("highlight");
        getChildren().addAll(frame, main, highlight);
    }

    /**
     * 添加监听器
     */
    private void registerListeners() {
        control.widthProperty().addListener(sizeListener);
        control.heightProperty().addListener(sizeListener);
        control.colorProperty().addListener(colorListener);
        control.stateProperty().addListener(stateListener);
    }

    @Override
    protected double computeMinWidth(final double height, final double top, final double right, final double bottom, final double left) {
        return MINIMUM_WIDTH;
    }

    @Override
    protected double computeMinHeight(final double width, final double top, final double right, final double bottom, final double left) {
        return MINIMUM_HEIGHT;
    }

    @Override
    protected double computePrefWidth(final double height, final double top, final double right, final double bottom, final double left) {
        return super.computePrefWidth(height, top, right, bottom, left);
    }

    @Override
    protected double computePrefHeight(final double width, final double top, final double right, final double bottom, final double left) {
        return super.computePrefHeight(width, top, right, bottom, left);
    }

    @Override
    protected double computeMaxWidth(final double width, final double top, final double right, final double bottom, final double left) {
        return MAXIMUM_WIDTH;
    }

    @Override
    protected double computeMaxHeight(final double width, final double top, final double right, final double bottom, final double left) {
        return MAXIMUM_HEIGHT;
    }

    /**
     * 处理控件属性变化
     */
    protected void handleControlPropertyChanged(final String property) {
        if ("RESIZE".equals(property)) {
            resize();
        } else if ("COLOR".equals(property)) {
            main.setStyle(String.join("", "-color: ", (control.getColor()).toString().replace("0x", "#"), ";"));
            resize();
        } else if ("STATE".equals(property)) {
            main.setEffect(control.getState() ? glow : innerShadow);
        }
    }

    /**
     * 注销监听器
     */
    @Override
    public void dispose() {
        control.widthProperty().removeListener(sizeListener);
        control.heightProperty().removeListener(sizeListener);
        control.colorProperty().removeListener(colorListener);
        control.stateProperty().removeListener(stateListener);
        control = null;
    }

    /**
     * 布局
     */
    @Override
    public void layoutChildren(final double x, final double y, final double width, final double height) {
        super.layoutChildren(x, y, width, height);
    }

    private void resize() {
        double width = control.getWidth() - control.getInsets().getLeft() - control.getInsets().getRight();
        double height = control.getHeight() - control.getInsets().getTop() - control.getInsets().getBottom();
        size = width < height ? width : height;
        if (size > 0) {
            innerShadow.setRadius(0.07 * size);
            glow.setRadius(0.36 * size);
            glow.setColor(control.getColor());
            frame.setMaxSize(size, size);
            main.setMaxSize(0.72 * size, 0.72 * size);
            main.relocate(0.14 * size, 0.14 * size);
            main.setEffect(control.getState() ? glow : innerShadow);
            highlight.setMaxSize(0.58 * size, 0.58 * size);
            highlight.relocate(0.21 * size, 0.21 * size);
        }
    }
}
```
{{<admonition  type="bug"  title="总结">}}

1. 主要思想是使用三个 Region 对象（frame-main-highlight，对于 LED 的每一层一个 Region）并使用 CSS 设置它们的样式。使用这种方法，我们只需要注意区域的大小和定位。其余的将在 CSS 中完成；

2.  在 LedSkin 中，为我们的属性定义了侦听器，以便我们可以对状态和颜色的变化做出反应；

3. Skin 类都有一个 dispose 方法，您应该使用它来取消注册侦听器并进行清理，以避免在更改控件的 Skin 时发生内存泄漏。

{{</admonition>}}

#### Switch
```java
package controlskin;
import javafx.animation.TranslateTransition;
import javafx.beans.InvalidationListener;
import javafx.event.EventHandler;
import javafx.scene.control.SkinBase;
import javafx.scene.input.MouseEvent;
import javafx.scene.layout.Pane;
import javafx.scene.layout.Region;
import javafx.util.Duration;

/**
 * @author Gerrit Grunwald
 * description：切换开关皮肤
 */
public class SwitchSkin extends SkinBase<CustomControl>{
    private static final double PREFERRED_WIDTH = 76;
    private static final double PREFERRED_HEIGHT = 46;
    private Region switchBackground;
    private Region thumb;
    private Pane pane;
    private TranslateTransition translate;
    private CustomControl control;
    private InvalidationListener colorListener;
    private InvalidationListener state;
    private EventHandler<MouseEvent> mouseEventHandler;

    /**
     * 构造器
     * @param control 自定义组件
     */
    public SwitchSkin(final CustomControl control) {
        super(control);
        this.control = control;
        colorListener = o -> handleControlPropertyChanged("COLOR");
        state = o -> handleControlPropertyChanged("STATE");
        mouseEventHandler = e -> this.control.setState(!this.control.getState());
        initGraphics();
        registerListeners();
    }

    /**
     * 初始化
     */
    private void initGraphics() {
        switchBackground = new Region();
        switchBackground.getStyleClass().add("switch-background");
        switchBackground.setStyle(String.join("", "-color: ", control.getColor().toString().replace("0x", "#"), ";"));
        thumb = new Region();
        thumb.getStyleClass().add("thumb");
        thumb.setMouseTransparent(true);
        if (control.getState()) {
            thumb.setTranslateX(32);
        }
        translate = new TranslateTransition(Duration.millis(70), thumb);
        pane = new Pane(switchBackground, thumb);
        getChildren().add(pane);
    }

    /**
     * 添加监听器
     */
    private void registerListeners() {
        control.colorProperty().addListener(colorListener);
        control.stateProperty().addListener(state);
        switchBackground.addEventHandler(MouseEvent.MOUSE_PRESSED, mouseEventHandler);
    }

    /**
     * 处理控件属性变化
     */
    @Override
    public void layoutChildren(final double x, final double y, final double width, final double height) {
        super.layoutChildren(x, y, width, height);
        thumb.relocate((width - PREFERRED_WIDTH) * 0.5, (height - PREFERRED_HEIGHT) * 0.5);
        switchBackground.relocate((width - PREFERRED_WIDTH) * 0.5, (height - PREFERRED_HEIGHT) * 0.5);
    }

    protected void handleControlPropertyChanged(final String property) {
        if ("COLOR".equals(property)) {
            switchBackground.setStyle(String.join("", "-color: ", control.getColor().toString().replace("0x", "#"), ";"));
        } else if ("STATE".equals(property)) {
            if (control.getState()) {
                // move thumb to the right
                translate.setFromX(2);
                translate.setToX(32);
            } else {
                // move thumb to the left
                translate.setFromX(32);
                translate.setToX(2);
            }
            translate.play();
        }
    }

    /**
     * 注销监听器
     */
    @Override
    public void dispose() {
        control.colorProperty().removeListener(colorListener);
        control.stateProperty().removeListener(state);
        switchBackground.removeEventHandler(MouseEvent.MOUSE_PRESSED, mouseEventHandler);
    }
}
```
{{<admonition type="bug" title="总结">}}
1. 这个 Control 甚至只需要两个 Region。一个用于背景(switchBackground)，一个用于拇指(thumb);

2. 除了视觉设计之外，LED 和开关之间的最大区别在于您可以通过点击开关与它进行交互。意思是说我们需要在背景区域中添加一个 MouseEvent.MOUSE_PRESSED 监听器，并使拇指区域鼠标透明。当鼠标事件监听器被触发时，我们需要切换状态并将拇指动画到另一边。

{{</admonition>}}

### CSS
#### LED

```CSS
.custom-control .frame {
    -fx-background-color : linear-gradient(from 14% 14% to 84% 84%,
                           rgba(20, 20, 20, 0.64706) 0%,
                           rgba(20, 20, 20, 0.64706) 15%,
                           rgba(41, 41, 41, 0.64706) 26%,
                           rgba(200, 200, 200, 0.40631) 85%,
                           rgba(200, 200, 200, 0.3451) 100%);
    -fx-background-radius: 1024px;
}
.custom-control .main {
    -fx-background-color : linear-gradient(from 15% 15% to 83% 83%,
                           derive(-color, -80%) 0%,
                           derive(-color, -87%) 49%,
                           derive(-color, -80%) 100%);
    -fx-background-radius: 1024px;
}
.custom-control:on .main {
    -fx-background-color: linear-gradient(from 15% 15% to 83% 83%,
                          derive(-color, -23%) 0%,
                          derive(-color, -50%) 49%,
                          -color 100%);
}
.custom-control .highlight {
    -fx-background-color : radial-gradient(center 15% 15%, radius 50%, white 0%, transparent 100%);
    -fx-background-radius: 1024;
}
```
{{<admonition  type="bug" title="总结">}}
1. 在该CSS 文件中，实际上只是定义了每个区域的背景半径和始终为渐变的绘制；

2. 通过触发 :on 伪类，我们只会将渐变从较暗的版本更改为较亮的版本，仅此而已。
{{</admonition>}}

#### Switch
```CSS
.custom-control .switch-background {
    -fx-pref-width       : 76;
    -fx-pref-height      : 46;
    -fx-min-width        : 76;
    -fx-min-height       : 46;
    -fx-max-width        : 76;
    -fx-max-height       : 46;
    -fx-background-radius: 1024;
    -fx-background-color : #a3a4a6;
}
.custom-control:on .switch-background {
    -fx-background-radius: 1024;
    -fx-background-color : -color;
}
.custom-control .thumb {
    -fx-translate-x      : 2;
    -fx-translate-y      : 2;
    -fx-pref-width       : 42;
    -fx-pref-height      : 42;
    -fx-min-width        : 42;
    -fx-min-height       : 42;
    -fx-max-width        : 42;
    -fx-max-height       : 42;
    -fx-background-radius: 1024;
    -fx-background-color : white;
    -fx-effect           : dropshadow(two-pass-box, rgba(0, 0, 0, 0.3), 1, 0.0, 0, 1);
}
```
#### Styles
{{< image src="/images/java/2022/led-change.png" caption="演示">}}
```CSS
.custom-control {
    -color: magenta;
}
```
{{<admonition tip>}}
通过添加场景样式表覆盖 CSS 文件中的一个属性-color，控件属性就会被覆盖，而无需更改代码，这显示了 CSS 的强大功能：
```java
scene.getStylesheets().add(ClassNameHere.class.getResource("/styles.css").toExternalForm());
```
```CSS
.custom-control {
    -color: magenta;
}
```

{{</admonition>}}
### 参考链接
* [Custom Controls in JavaFX (Part 4)](https://foojay.io/today/custom-controls-in-javafx-part-iv/)
* [HanSolo/JavaFXCustomControls(github)](https://github.com/HanSolo/JavaFXCustomControls)
* [adobe illustrator to CSS——矢量绘图程序：视觉编码之操作文档](https://helpx.adobe.com/uk/illustrator/using/css-extraction.html)