[toc]

This edition first published 2012

## 1. Web上的游戏

Canvas：

    <canvas id=”mycanvas”></canvas>
    <script>
        var canvas = document.getElementById(“mycanvas”),
        ctx = canvas.getContext(“2d”);
        canvas.width = canvas.height = 200;
        // draw two blue circles
        ctx.fillStyle = “blue”;
        ctx.beginPath();
        ctx.arc(50, 50, 25, 0, Math.PI * 2, true);
        ctx.arc(150, 50, 25, 0, Math.PI * 2, true);
        ctx.fill();
        // draw a red triangle
        ctx.fillStyle = “red”;
        ctx.beginPath();
        ctx.moveTo(100, 75);
        ctx.lineTo(75, 125);
        ctx.lineTo(125, 125);
        ctx.fill();
        // draw a green semi-circle
        ctx.strokeStyle = “green”;
        ctx.beginPath();
        ctx.scale(1, 0.5);
        ctx.arc(100, 300, 75, Math.PI, 0, true);
        ctx.closePath();
        ctx.stroke();
    </script>

音频：

    <audio controls id=”myaudio”>
        <source src=”Prelude In E Minor, Op. 28.ogg”/>
        <source src=”Prelude In E Minor, Op. 28.mp3”/>
    </audio>
    <script>
    var audio = document.getElementById(“myaudio”);
    document.onkeydown = function(e) {
        if (e.keyCode == 83) {
            audio.pause(); // Key pressed was S
        } else if (e.keyCode == 80) {
            audio.play(); // Key pressed was P
        }
    }
    </script>

Audio data APIs that will eventually allow dynamically generated sound effects and audio filters are in the works at both the Mozilla and WebKit camps. Because these APIs are still very early in their development, I won’t be using them for the games in this book, although I briefly examine the possibilities they present in Chapter 10 when I dive into HTML5 audio.

WebSockets：

    // Create a new WebSocket object
    var socket = new WebSocket(“ws://mygameserver.com:4200/”);
    // Send an initial message to the server
    socket.onopen = function () {
    	socket.send(“Hello server!”);
    };
    // Listen for any data sent to us by the server
    socket.onmessage = function(msg) {
    	alert(“Server says: “ + msg);
    }

WebGL：

WebGL is OpenGL for the web. 用于创建3D。 Google developers released a WebGL port of the legendary first-person shooter, Quake II, on April 1, 2010.

## 2. 第一步

### 理解游戏

游戏名字叫Jewel Warrior。游戏有8x8棋盘。64格子上有随机的珠宝。3个相邻的珠宝得分。用户可以交换相邻的珠宝：选择一个，再选择另一个。

### 创建游戏骨架

使用Sizzle做CSS选择符引擎。

	var jewels = Sizzle(“div#gameboard .jewel”);

HTML：

    <div id=”game”>
        <div class=”screen” id=”splash-screen”></div>
        <div class=”screen” id=”main-menu”></div>
        <div class=”screen” id=”game-screen”></div>
        <div class=”screen” id=”high-scores”></div>
    </div>

脚本：

    jewel.game = (function() {
        var dom = jewel.dom,
        $ = dom.$;
        // hide the active screen (if any) and show the screen
        // with the specified id
        function showScreen(screenId) {
        	var activeScreen = $(“#game .screen.active”)[0],
        	screen = $(“#” + screenId)[0];
            if (activeScreen) {
            	dom.removeClass(screen, “active”);
            }
        	dom.addClass(screen, “active”);
        }
        // expose public methods
        return {
        	showScreen : showScreen
        };
    })();

### 创建启动屏

#### （未）Working with web fonts

## 3. 面向手机

### 为iOS和Android开发

#### 将Web应用放到Home屏上

Safari中，用户电机书签图标，可以将一个链接或网站放在Home屏。但打开后应用还是运行在Safari中与一个网站无异。幸好，可以取得原生应用的效果，用户从Home屏打开时，会使用全屏模式。只需要添加一个meta标签：

	<head>
    ...
    <meta name=”apple-mobile-web-app-capable” content=”yes” />
    ...
    </head>

Safari支持判断应用打开的方式是Web应用还是常规Web页面。如果`window.navigator.standalone`值为true，则页面是从Home屏打开的Web应用模式。

    if (window.navigator.standalone) {
        alert(“You are running the standalone app!”);
    } else if (window.navigator.standalone == false) {
        alert(“You are using app in mobile Safari!”);
	} else {
		alert(“You are using the app in another browser!”);
	}

利用这个判断，若用户以网页形式打开，在启动屏时提示用户可以安装到桌面。

默认iOS使用应用的截屏桌位桌面图标。你可以自己指定一个：

	<link rel=”apple-touch-icon” href=”icon.png”/>

图标会被iOS处理添加一些效果。使用`apple-touch-icon-precomposed`可以禁止iOS处理图标：

    <link rel=”apple-touch-icon-precomposed” href=”images/ios-icon.png”/>

因为苹果设备有多种尺寸，可以指定多种尺寸的图标：

    <head>
        ...
        <link rel=”apple-touch-icon-precomposed”
        	href=”images/ios-icon.png”/>
        <link rel=”apple-touch-icon-precomposed” sizes=”72x72”
        	href=”images/ios-icon-ipad.png”/>
        <link rel=”apple-touch-icon-precomposed” sizes=”114x114”
        	href=”images/ios-icon-iphone4.png”/>

You can also use one set of icons for the entire web site by leaving out the link elements and instead placing the icon images in the root directory of the web site. The iOS system then searches the root for a suitable icon using a list of predefined filenames in the format `apple-touch-icon[-<w>x<h>][-precomposed].png`. Precomposed icons are chosen over regular icons and icons with the specific size needed for that resolution are chosen over the default icon. A device that uses 57x57 icons looks for the following list of filenames:

- apple-touch-icon-57x57-precomposed.png
- apple-touch-icon-57x57.png
- apple-touch-icon-precomposed.png
- apple-touch-icon.png

##### （未）Adding a startup image

##### （未）Styling the status bar

#### 禁用部分浏览器特性

在一个Android浏览器或手机Safari中滚动一个网页时，如果滚动过头，浏览器会允许你继续，只是最后屏幕会退回到结束位置。但如果游戏不需要滚动，可以禁用此特性。方法是完全禁止通过触摸滚动。方法监听document元素的`touchmove`事件，然后调用`event.preventDefault()`方法。

    dom.bind(document, “touchmove”, function(event) {
        event.preventDefault();
    });

禁用地址栏。The trick to hiding the address bar is to force the browser to scroll to the top of the page. If there’s enough content, it automatically pushes the address bar out of view. Because the height is set to 100%, the game only takes up as much as space as it can, so scrolling to the top has no effect. You can make sure the page is long enough by increasing the height of the html element to, say, 200%.

    if (/Android/.test(navigator.userAgent)) {
        $(“html”)[0].style.height = “200%”;
        setTimeout(function() {
        	window.scrollTo(0, 1);
        }, 0);
    }

When you keep your finger pressed down for a second or two on, for example, an image or a link, a small callout appears, giving you the option to follow the link, save the image, and so on. This feature has no place in a game. The user should be able to tap and press anything without interference from the native browser behavior. The following CSS property disables the callout.

	-webkit-touch-callout: none;

类似的，选择文本和图片也应该被禁用

	-webkit-user-select : none;

当你触摸时，Android会高亮可点击的元素。禁用：

	-webkit-tap-highlight-color: rgba(0,0,0,0);

某些改变发生时，如朝向改变，浏览器会自动调整字体。禁用：

	-webkit-text-size-adjust: none;

These four properties must apply to all content in the game and thus should go on one of the top-most elements. Listing 3.29 shows the extra CSS rules added to the body element in main.css.

    body {
        ...
        -webkit-touch-callout: none;
        -webkit-tap-highlight-color: rgba(0,0,0,0);
        -webkit-text-size-adjust: none;
        -webkit-user-select : none;
    }

## 4. 构建游戏









