[toc]

## 布局

### box-sizing

设置盒子的宽度都包含哪些部分：

	E { box-sizing: keyword; }

其中E是某个选择符。取值可以是`border-box`或`content-box`。默认是`content-box`，表示宽度属性只包含内容。`border-box`表示宽度属性包含内容、padding和边框，但不包括margin。

    div {
        border: 10px solid black;
        box-sizing: border-box;
        padding: 10px;
        width: 150px;
    }

现在宽度150px包括padding和边框。减去一边10px的padding和border，内容只有110px。

Firefox还支持一个额外的关键字`padding-box`。即宽度包含内容和padding，但不包含边框。


## 疑难杂症

### Android、Webkit

#### Android、WebView`:active`失效

Android中WebView`:active`失效，即点击瞬间的效果无法被触发。最简单的解决办法是，添加：

	<body ontouchstart="">

详情讨论见：http://stackoverflow.com/questions/4940429/how-to-simulate-active-css-pseudo-class-in-android-on-non-link-elements