[toc]

## 如何在android平台上使用js直接调用Java方法

http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/reflection/zh.md

在 cocos2d-js 3.0beta 中加入了一个新特性，在 android 平台上我们可以通过**反射**直接在 js 中调用 java 的**静态方法**。它的使用方法很简单：

```js
var o = jsb.reflection.callStaticMethod(className, methodName, methodSignature, parameters...)
```

在 `callStaticMethod` 方法中，我们通过传入 Java 的类名，方法名，方法签名，参数就可以直接调用 Java 的静态方法，并且可以获得 Java 方法的返回值。下面介绍的类名和方法签名可能会有一点奇怪，但是 Java 的规范就是如此的。

**类名**

参数中的类名必须是包含 Java 包路径的完整类名，例如我们在 `org.cocos2dx.javascript` 这个包下面写了一个Test类：

```java
package org.cocos2dx.javascript;
public class Test {
    public static void hello(String msg){
        System.out.println(msg);
    }
    public static int sum(int a, int b){
        return a + b;
    }
    public static int sum(int a){
        return a + 2;
    }
}
```

那么这个 Test 类的完整类名应该是 `org/cocos2dx/javascript/Test`。注意这里必须是斜线 `/`,而不是在 Java 代码中我们习惯的点 `.`。

**方法名**

方法名很简单，就是方法本来的名字，例如 sum 方法的名字就是 sum。

**方法签名**

方法签名稍微有一点复杂，最简单的方法签名是 `()V`，它表示一个没有参数没有返回值的方法。其他一些例子：

- `(I)V` 表示参数为一个 int，没有返回值的方法
- `(I)I` 表示参数为一个int，返回值为int的方法
- `(IF)Z` 表示参数为一个int和一个float，返回值为boolean的方法

现在有一些理解了吧，括号内的符号表示参数类型，括号后面的符号表示返回值类型。因为Java是允许函数重载的，可以有多个方法名相同但是参数返回值不同的方法，方法签名正是用来帮助区分这些相同名字的方法的。

目前Cocos2d-js中支持的Java类型签名有下面4种：

|Java类型  |签名
|-|-
|int      |I
|float    |F
|boolean  |Z
|String   |Ljava/lang/String;

**参数**

参数可以是0个或任意多个，直接使用js中的number，bool和string就可以。

**使用示例**

我们将会调用上面的 Test 类中的静态方法：

```js
// 调用hello方法
jsb.reflection.callStaticMethod("org/cocos2dx/javascript/Test", "hello", "(Ljava/lang/String;)V", "this is a message from js");

// 调用第一个sum方法
var result = jsb.reflection.callStaticMethod("org/cocos2dx/javascript/Test", "sum", "(II)I", 3, 7);
cc.log(result); // 10

// 调用第二个sum方法
var result = jsb.reflection.callStaticMethod("org/cocos2dx/javascript/Test", "sum", "(I)I", 3);
cc.log(result); //5
```

在你的控制台会有正确的输出的，这很简单吧。

**注意**

另外有一点需要注意的就是，在 android 应用中，cocos 的渲染和 js 的逻辑是在**gl线程**中进行的，而 android 本身的 UI 更新是在 app 的 ui 线程进行的，所以如果我们在 js 中调用的 Java 方法有任何刷新 UI 的操作，都需要在 ui 线程进行。

**再加点料**

现在我们可以从 js 调用 Java 了，那么能不能反过来？当然可以！在你的项目中包含 `Cocos2dxJavascriptJavaBridge`，这个类有一个 `evalString` 方法可以执行 js 代码，它位于 frameworks\js-bindings\bindings\manual\platform\android\java\src\org\cocos2dx\lib 文件夹下。我们将会给刚才的 Alert 对话框增加一个按钮，并在它的响应中执行 js。和上面的情况相反，这次执行 js 代码**必须在 gl 线程中进行**。

```js
alertDialog.setButton("OK", new DialogInterface.OnClickListener() {
    public void onClick(DialogInterface dialog, int which) {
        // 一定要在GL线程中执行
        app.runOnGLThread(new Runnable() {
            @Override
            public void run() {
                Cocos2dxJavascriptJavaBridge.evalString("cc.log(\"Javascript Java bridge!\")");
            }
        });
    }
});
```

这样在点击 OK 按钮后，你应该可以在控制台看到正确的输出。`evalString` 可以执行任何js代码，并且它可以访问到你在js代码中的对象。