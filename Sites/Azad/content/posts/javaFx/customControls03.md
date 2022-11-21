---
title: "JavaFx自定义控件（三）- 扩展现有控件来创建自定义控件"
tags: [ "controls", "JavaFx", "自定义控件"  ]
categories:
  - "JavaFx"
date: 2022-06-06T11:15:38+08:00
lastmod: 2022-06-13T12:16:38+08:00
draft: false
---

### 导航
**创建自定义控件的不同方法**
* [使用CSS重新设置现有控件的样式](http://localhost:1313/customcontrols01/)
* [组合现有控件来创建自定义控件](http://localhost:1313/customcontrols02/)
* [扩展现有控件](http://localhost:1313/customcontrols03/)
* [使用Control + Skin类](http://localhost:1313/customcontrols04/)
* [使用Region类](http://localhost:1313/customcontrols05/)
* [使用Canvas类](http://localhost:1313/customcontrols06/)

### 扩展现有控件来生成自定义控件
{{< admonition example >}}
 **自定义扩展组件示例** ：根据焦点状态添加文本的动画组件
 {{< /admonition >}}
{{< image src="/images/java/2022/extended.png" caption="成果"  >}}

```java
package extended;
import javafx.application.Application;
import javafx.application.Platform;
import javafx.geometry.Insets;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.layout.VBox;
import javafx.stage.Stage;

/**
 * @author Gerrit Grunwald
 */
public class DemoExtended extends Application {
    private ExtendedControl control;
    private Button button;

    @Override
    public void init() {
        control = new ExtendedControl();
        control.setPromptText("Name");
        button = new Button("Focus");
    }

    @Override
    public void start(final Stage stage) {
        VBox pane = new VBox(24, control, button);
        pane.setPadding(new Insets(20));
        Scene scene = new Scene(pane);
        stage.setTitle("Extended");
        stage.setScene(scene);
        stage.show();
        button.requestFocus();
    }

    @Override
    public void stop() {
        Platform.exit();
        System.exit(0);
    }

    public static void main(String[] args) {
        launch(args);
    }
}
```
```java
package extended;
import javafx.animation.KeyFrame;
import javafx.animation.KeyValue;
import javafx.animation.Timeline;
import javafx.beans.property.DoubleProperty;
import javafx.beans.property.ObjectProperty;
import javafx.beans.property.SimpleDoubleProperty;
import javafx.css.*;
import javafx.scene.control.TextField;
import javafx.scene.layout.HBox;
import javafx.scene.paint.Color;
import javafx.scene.text.Font;
import javafx.scene.text.Text;
import javafx.util.Duration;

import java.util.List;

/**
 * @author Gerrit Grunwald
 * Here are the things we need to do...
 *
 * 1.Extend the JavaFX TextField
 * 2.Add a Text to it that can be set using the promptTextProperty
 * 3.Add animation of the Text dependent on the focus state
 * 4.Add some CSS styling
 */
public class ExtendedControl extends TextField {
    private static final StyleablePropertyFactory<ExtendedControl> FACTORY
            = new StyleablePropertyFactory<>(TextField.getClassCssMetaData());
    private static final double STD_FONT_SIZE = 13;
    private static final double SMALL_FONT_SIZE = 10;
    private static final double TOP_OFFSET_Y = 4;
    private static final int ANIMATION_DURATION = 60;

    /** 组件 **/
    private Text promptText;
    private HBox promptTextBox;

    /** paint **/
    private static final Color DEFAULT_MATERIAL_DESIGN_COLOR = Color.web("#009688");
    private static final Color DEFAULT_PROMPT_TEXT_COLOR = Color.web("#757575");

    /** 动画时间线 **/
    private Timeline timeline;

    /** Properties **/
    private final StyleableProperty<Color> materialDesignColor;
    private final StyleableProperty<Color> promptTextColor;
    private DoubleProperty fontSize;

    /** 用户代理样式表 **/
    private static String userAgentStyleSheet;

    /** css样式属性 **/
    private static final CssMetaData<ExtendedControl, Color> MATERIAL_DESIGN_COLOR
            = FACTORY.createColorCssMetaData
            ("-material-design-color", s -> s.materialDesignColor, DEFAULT_MATERIAL_DESIGN_COLOR, false);
    private static final CssMetaData<ExtendedControl, Color> PROMPT_TEXT_COLOR
            = FACTORY.createColorCssMetaData
            ("-prompt-text-color", s -> s.promptTextColor, DEFAULT_PROMPT_TEXT_COLOR, false);

    public ExtendedControl() {
        this("");
    }

    public ExtendedControl(final String promptTextBox) {
        super(promptTextBox);
        materialDesignColor = new SimpleStyleableObjectProperty<>(MATERIAL_DESIGN_COLOR, this, "materialDesignColor");
        promptTextColor = new SimpleStyleableObjectProperty<>(PROMPT_TEXT_COLOR, this, "promptTextColor");
        fontSize = new SimpleDoubleProperty(ExtendedControl.this, "fontSize", getFont().getSize());
        timeline = new Timeline();
        initGraphics();
        registerListeners();
    }

    /**
     * 初始化
     */
    private void initGraphics() {
        getStyleClass().addAll("material-field");
        final String fontFamily = getFont().getFamily();
        final int length = getText().length();
        promptText = new Text(getPromptText());
        promptText.getStyleClass().add("prompt-text");
        promptTextBox = new HBox(promptText);
        promptTextBox.getStyleClass().add("material-field");
        if (!isEditable() || isDisabled() || length > 0) {
            promptText.setFont(Font.font(fontFamily, SMALL_FONT_SIZE));
            promptTextBox.setTranslateY(-STD_FONT_SIZE - TOP_OFFSET_Y);
        } else {
            promptText.setFont(Font.font(fontFamily, STD_FONT_SIZE));
        }
        getChildren().addAll(promptTextBox);
    }

    /**
     * 注册监听器
     */
    private void registerListeners() {
        textProperty().addListener(o -> handleTextAndFocus(isFocused()));
        promptTextProperty().addListener(o -> promptText.setText(getPromptText()));
        focusedProperty().addListener(o -> handleTextAndFocus(isFocused()));
        promptTextColorProperty().addListener(o -> promptText.setFill(getPromptTextColor()));
        fontSize.addListener(o -> promptText.setFont(Font.font(fontSize.get())));
        timeline.setOnFinished(evt -> {
            final int length = null == getText() ? 0 : getText().length();
            if (length > 0 && promptTextBox.getTranslateY() >= 0) {
                promptTextBox.setTranslateY(-STD_FONT_SIZE - TOP_OFFSET_Y);
                fontSize.set(SMALL_FONT_SIZE);
            }
        });
    }

    /**
     * css样式属性
     */
    public Color getMaterialDesignColor() {
        return materialDesignColor.getValue();
    }
    public Color getPromptTextColor() {
        return promptTextColor.getValue();
    }
    public ObjectProperty<Color> promptTextColorProperty() {
        return (ObjectProperty<Color>) promptTextColor;
    }

    /**
     * style related
     */
    @Override
    public String getUserAgentStylesheet() {
        if (null == userAgentStyleSheet) {
            userAgentStyleSheet = getClass().getResource("/extended.css").toExternalForm();
        }
        return userAgentStyleSheet;
    }

    @Override
    public List<CssMetaData<? extends Styleable, ?>> getControlCssMetaData() {
        return FACTORY.getCssMetaData();
    }

    /**
     * function method
     */
    private void handleTextAndFocus(final boolean isFocused) {
        final int length = null == getText() ? 0 : getText().length();
        KeyFrame kf0;
        KeyFrame kf1;
        KeyValue kvTextY0;
        KeyValue kvTextY1;
        KeyValue kvTextFontSize0;
        KeyValue kvTextFontSize1;
        KeyValue kvPromptTextFill0;
        KeyValue kvPromptTextFill1;
        if (isFocused | length > 0 || isDisabled() || !isEditable()) {
            if (Double.compare(promptTextBox.getTranslateY(), -STD_FONT_SIZE - TOP_OFFSET_Y) != 0) {
                kvTextY0 = new KeyValue(promptTextBox.translateYProperty(), 0);
                kvTextY1 = new KeyValue(promptTextBox.translateYProperty(), -STD_FONT_SIZE - TOP_OFFSET_Y);
                kvTextFontSize0 = new KeyValue(fontSize, STD_FONT_SIZE);
                kvTextFontSize1 = new KeyValue(fontSize, SMALL_FONT_SIZE);
                kvPromptTextFill0 = new KeyValue(promptTextColorProperty(), DEFAULT_PROMPT_TEXT_COLOR);
                kvPromptTextFill1 = new KeyValue
                        (promptTextColorProperty(), isFocused ? getMaterialDesignColor() : DEFAULT_PROMPT_TEXT_COLOR);
                kf0 = new KeyFrame(Duration.ZERO, kvTextY0, kvTextFontSize0, kvPromptTextFill0);
                kf1 = new KeyFrame(Duration.millis(ANIMATION_DURATION), kvTextY1, kvTextFontSize1, kvPromptTextFill1);
                timeline.getKeyFrames().setAll(kf0, kf1);
                timeline.play();
            }
            promptText.setFill(isFocused ? getMaterialDesignColor() : DEFAULT_PROMPT_TEXT_COLOR);
        } else {
            if (Double.compare(promptTextBox.getTranslateY(), 0) != 0) {
                kvTextY0 = new KeyValue(promptTextBox.translateYProperty(), promptTextBox.getTranslateY());
                kvTextY1 = new KeyValue(promptTextBox.translateYProperty(), 0);
                kvTextFontSize0 = new KeyValue(fontSize, SMALL_FONT_SIZE);
                kvTextFontSize1 = new KeyValue(fontSize, STD_FONT_SIZE);
                kvPromptTextFill0 = new KeyValue(promptTextColorProperty(), getMaterialDesignColor());
                kvPromptTextFill1 = new KeyValue(promptTextColorProperty(), DEFAULT_PROMPT_TEXT_COLOR);
                kf0 = new KeyFrame(Duration.ZERO, kvTextY0, kvTextFontSize0, kvPromptTextFill0);
                kf1 = new KeyFrame(Duration.millis(ANIMATION_DURATION), kvTextY1, kvTextFontSize1, kvPromptTextFill1);
                timeline.getKeyFrames().setAll(kf0, kf1);
                timeline.play();
            }
        }
    }
}
```
{{< admonition >}}
在此方法中，向时间线添加了一个侦听器，当时间线完成时将触发该侦听器。在这种情况下，将检查 TextField 是否包含文本，如果包含，则 Text 组件将动态地被放置在指定的位置以及缩放到指定的尺寸。
{{< /admonition >}}
```CSS
.material-field {
    -material-design-color: #3f51b5;
    -material-design-color-transparent: #3f51b51f;
    -prompt-text-color: #757575;
}
.material-field:readonly {
    -fx-prompt-text-fill: transparent;
}
.material-field:disabled {
    -fx-prompt-text-fill: transparent;
}
.text-input {
    -fx-font-family: "Arial";
    -fx-font-size: 13px;
    -fx-text-fill: -fx-text-inner-color;
    -fx-highlight-fill: derive(-fx-control-inner-background,-20%);
    -fx-highlight-text-fill: -fx-text-inner-color;
    -fx-prompt-text-fill: transparent;
    -fx-background-color: transparent;
    -fx-background-insets: 0;
    -fx-background-radius: 0;
    -fx-border-color: transparent transparent #616161 transparent;
    -fx-border-width: 1;
    -fx-border-insets: 0 0 1 0;
    -fx-border-style: hidden hidden solid hidden;
    -fx-cursor: text;
    -fx-padding: 0.166667em 0em 0.333333em 0em;
}
.text-input:focused {
    -fx-highlight-fill: -material-design-color-transparent;
    -fx-highlight-text-fill: -material-design-color;
    -fx-text-fill: -fx-text-inner-color;
    -fx-background-color: transparent;
    -fx-border-color: transparent transparent -material-design-color transparent;
    -fx-border-width: 2;
    -fx-border-insets: 0 0 2 -1;
    -fx-prompt-text-fill: transparent;
    -fx-padding: 2 0 4 0;
}
.text-input:readonly {
    -fx-background-color: transparent;
    -fx-text-fill: derive(-fx-text-base-color, 35%);
    -fx-border-style: segments(2, 3)  line-cap butt;
    -fx-border-color: transparent transparent #616161 transparent;
}
.text-input:focused:readonly {
    -fx-text-fill: derive(-fx-text-base-color, 35%);
    -fx-border-style: segments(2, 3)  line-cap butt;
    -fx-border-color: transparent transparent -material-design-color transparent;
}
.text-input:disabled {
    -fx-opacity: 0.46;
    -fx-border-style: segments(2, 3)  line-cap butt;
    -fx-border-color: transparent transparent black transparent;
}
```
### 参考链接
* [Custom Controls in JavaFX (Part 3)](https://foojay.io/today/custom-controls-in-javafx-part-iii/)
* [HanSolo/JavaFXCustomControls(github)](https://github.com/HanSolo/JavaFXCustomControls)