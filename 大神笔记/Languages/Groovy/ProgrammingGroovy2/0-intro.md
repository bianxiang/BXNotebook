[toc]


## 介绍

本书主页：http://www.pragprog.com/titles/vslg2

### 0.2 为什么要用动态语言？

可以非常方便的向累添加动态方法。如

	Integer.metaClass.percentRaise={amount->amount*(1+delegate/100.0)}

然后就可以

	5.percentRaise(80000)

### 0.3 为什么用Groovy

Groovy的学习曲线不陡峭。

几乎所有Java代码都可以直接在Groovy中使用。

Groovy classes extend the same good old `java.lang.Object`—Groovy classes are Java classes.

### 0.4 本书内容



