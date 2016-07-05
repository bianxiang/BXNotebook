[toc]

## 表单

### 表单及表单元素

有点过时的总结：http://www.456bereastreet.com/lab/styling-form-controls-revisited/

### 占位符颜色修改

toscho：有三种实现方式：伪元素（pseudo-elements）、伪类（ pseudo-classes）和Notihing。
WebKit和Blink（Safari,Google Chrome, Opera15+）使用伪元素
`::-webkit-input-placeholder`

Mozilla Firefox 4-18使用伪类
`:-moz-placeholder`

Mozilla Firefox 19+ 使用伪元素
`::-moz-placeholder`

IE10使用伪类
`:-ms-input-placeholder`

IE9和Opera12以下版本的CSS选择器均不支持占位文本。需要注意的是伪元素在Shadow DOM里会起到元素的真实作用。

CSS选择器
因为每个浏览器的CSS选择器都有所差异，所以需要针对每个浏览器做单独的设定。

::-webkit-input-placeholder { /* WebKit browsers */
    color:    #999;
}
:-moz-placeholder { /* Mozilla Firefox 4 to 18 */
    color:    #999;
}
::-moz-placeholder { /* Mozilla Firefox 19+ */
    color:    #999;
}
:-ms-input-placeholder { /* Internet Explorer 10+ */
    color:    #999;
}
Matt：textareas（文本框可拉伸）风格样式的代码，如下：

input::-webkit-input-placeholder, textarea::-webkit-input-placeholder {
  color: #636363;
}
input:-moz-placeholder, textarea:-moz-placeholder {
  color: #636363;
}
brillout.com：input和Textarea的字体颜色均为红色。所有样式都要针对不同的选择器而定，不要打包整体处理，因为其中一个出问题，其他的都会失效。

*::-webkit-input-placeholder {
    color: red;
}

*:-moz-placeholder {
    color: red;
}

*:-ms-input-placeholder {
    /* IE10+ */
    color: red;
}
James Donnelly：在Firefox和IE里，正常input文本颜色覆盖占位符颜色的方法：

::-webkit-input-placeholder { 
    color: red; text-overflow: ellipsis; 
}
:-moz-placeholder { 
    color: #acacac !important; text-overflow: ellipsis; 
}
::-moz-placeholder { 
    color: #acacac !important; text-overflow: ellipsis; 
} /* for the future */
:-ms-input-placeholder { 
    color: #acacac !important; text-overflow: ellipsis; 
}
还有一种好办法:

input::-webkit-input-placeholder, textarea::-webkit-input-placeholder { 
    color:    #666;
}
input:-moz-placeholder, textarea:-moz-placeholder { 
    color:    #666;
}
input::-moz-placeholder, textarea::-moz-placeholder { 
    color:    #666;
}
input:-ms-input-placeholder, textarea:-ms-input-placeholder { 
    color:    #666;
}