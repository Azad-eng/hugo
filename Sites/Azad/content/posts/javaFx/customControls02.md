---
title: "JavaFx自定义控件（二）- 组合现有控件来创建自定义控件"
tags: [ "controls", "JavaFx", "自定义控件"  ]
categories:
  - "JavaFx"
date: 2022-06-06
lastmod: 2022-06-13
draft: true
---

### 导航
**创建自定义控件的不同方法**
* [使用CSS重新设置现有控件的样式](http://localhost:1313/customcontrols01/)
* [组合现有控件来创建自定义控件](http://localhost:1313/customcontrols02/)
* [扩展现有控件](http://localhost:1313/customcontrols03/)
* [使用Control + Skin类](http://localhost:1313/customcontrols04/)
* [使用Region类](http://localhost:1313/customcontrols05/)
* [使用Canvas类](http://localhost:1313/customcontrols06/)

### 组合现有控件来创建自定义控件
{{< admonition example >}}
 **自定义组合组件示例** ：一个可以通过点击右边按钮来让文本域切换相应的值的组合组件——实现℃和°F 的转换功能
{{< image src="/images/java/2022/uncomb.png" caption="添加css样式表前"  >}}

                                            `添加css样式表前后`

{{< image src="/images/java/2022/comb.png" caption="添加css样式表后"  >}}
{{< /admonition >}}

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
package comb;

import javafx.geometry.Pos;
import javafx.scene.control.Button;
import javafx.scene.control.TextField;
import javafx.scene.control.TextFormatter;
import javafx.scene.layout.HBox;

import java.util.Locale;

/**
 * @author Azad-eng
 * Description: 组合组件类——由TextField和Button组成的自定义实现特定功能的HBox组件,
 * 作用是实现摄氏度和华氏度的值的转换
 */
public class CombinedControl extends HBox {
    private TextField textField;
    private Button button;

    public CombinedControl() {
        init();
        getStyleClass().setAll("combined-control");
        registerListeners();
    }

    private void init() {
        textField = new TextField();
        textField.setFocusTraversable(false);
        textField.setTextFormatter(
                new TextFormatter<>(change -> change.getText().matches("[0-9]*(\\.[0-9]*)?") ? change : null));
        button = new Button("°C");
        button.setFocusTraversable(false);
        setSpacing(0);
        setFillHeight(false);
        setAlignment(Pos.CENTER);
        getChildren().addAll(textField, button);
    }

    private void registerListeners() {
        button.setOnMousePressed(e -> handleControlPropertyChanged("BUTTON_PRESSED"));
    }

    private void handleControlPropertyChanged(final String PROPERTY) {
        if ("BUTTON_PRESSED".equals(PROPERTY)) {
            String buttonText = button.getText();
            String text = textField.getText();
            if (text.matches("^[-+]?\\d+(\\.\\d+)?$")) {
                if ("°C".equals(buttonText)) {
                    // Convert to Fahrenheit
                    button.setText("°F");
                    textField.setText(toFahrenheit(textField.getText()));
                } else {
                    // Convert to Celsius
                    button.setText("°C");
                    textField.setText(toCelsius(textField.getText()));
                }
            }
        }
    }

    private String toFahrenheit(final String text) {
        try {
            double celsius = Double.parseDouble(text);
            return String.format(Locale.US, "%.2f", (celsius * 1.8 + 32));
        } catch (NumberFormatException e) {
            return text;
        }
    }

    private String toCelsius(final String text) {
        try {
            double fahrenheit = Double.parseDouble(text);
            return String.format(Locale.US, "%.2f", ((fahrenheit - 32) / 1.8));
        } catch (NumberFormatException e) {
            return text;
        }
    }
}
```
```css
/*这是样式表，让控件内部的两个小组件视觉上仿若一体*/

.combined-control:focused {
    -fx-highlight-fill     : -fx-accent;
    -fx-background-color : -fx-focus-color, -fx-control-inner-background, -fx-faint-focus-color;
    -fx-background-insets: -1.2, 1, -2.4;
    -fx-background-radius: 3, 2, 4, 0;
    -fx-border-color     : -fx-faint-focus-color;
    -fx-border-insets    : -1;
}
.combined-control:focused > .button {
    -fx-background-color : -fx-focus-color, -fx-outer-border, -fx-inner-border, -fx-body-color, -fx-faint-focus-color, -fx-body-color;
    -fx-background-insets: -0.2 -0.2 -0.2 1, 1 1 1 0, 1 1 1 1, 2, -1.4 -1.4 -1.4 1, 2.6;
    -fx-background-radius: 0 3 3 0, 0 2 2 0, 0 1 1 0, 0 4 4 0, 0 1 1 0;
}
.combined-control > .text-input,
.combined-control > .text-input:focused {
    -fx-background-color   : linear-gradient(to bottom, derive(-fx-text-box-border, -10%), -fx-text-box-border),
                             linear-gradient(from 0px 0px to 0px 5px, derive(-fx-control-inner-background, -9%), -fx-control-inner-background);
    -fx-background-insets  : 0, 1 0 1 1;
    -fx-background-radius  : 3 0 0 3, 2 0 0 2;
    -fx-pref-width         : 120px;
}
.combined-control > .button {
    -fx-background-radius: 0 3 3 0, 0 3 3 0, 0 2 2 0, 0 1 1 0;
    -fx-pref-width       : 36px;
    -fx-min-width        : 36px;
}
.combined-control > .button:focused {
    -fx-background-color : -fx-outer-border, -fx-inner-border, -fx-body-color, -fx-body-color;
    -fx-background-insets: 0, 1, 2, 2;
}
```

{{< admonition >}}
**url路径前面一定要有/：**
比如"/combined.css", "/styles/restyled.css"...

`否则运行程序后会抛出Exception in Application start method异常`

{{< /admonition >}}
### 参考链接
* [Custom Controls in JavaFX (Part 2)](https://foojay.io/today/custom-controls-in-javafx-part-ii/)
* [HanSolo/JavaFXCustomControls(github)](https://github.com/HanSolo/JavaFXCustomControls)