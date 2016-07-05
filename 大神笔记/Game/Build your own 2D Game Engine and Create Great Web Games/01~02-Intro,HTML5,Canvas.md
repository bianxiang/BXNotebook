[toc]

本书使用 HTML5, JavaScript, WebGL。

源码下载：www.apress.com/9781484209530. Follow the instructions to download the 9781484209530.zip

# 1 介绍使用 JavaScript 开发 2D 游戏引擎
## 使用技术
The Web Graphics Library (WebGL) is a JavaScript API designed specifically for the generation of 2D and 3D computer graphics through web browsers. With its support for OpenGL Shading Language (GLSL) and the ability to access the graphics processing unit (GPU) on client machines, WebGL has the capability of producing highly complex graphical effects in real time andis perfect as the graphics API for browser-based video games.
## 开发环境

浏览器使用 Chrome。Notice that Microsoft Internet Explorer 11 does not support HTML5 AudioContext and thus will not execute any projects after Chapter 4; in addition, Mozilla Firefox (version 39.0) does not support some of the GLSL shaders in Chapter 9.

**glMatrix math library**: This is a library that implements the foundational mathematic operations. You can download this library from http://glMatrix.net. You will integrate this library into your game engine in Chapter 3, so more details will be provided there.

Interested readers can learn more about these topics in other books.Computer graphics:

- Shirley, Ashikhmin, and Marschner. Fundamentals of Computer Graphics, 3rd edition. A. K. Peters, 2009.
- Angle and Shreiner. Interactive Computer Graphics: A Top Down Approach with WebGL. Pearson Education, 2014.Linear algebra:

- Johnson, Riess, and Arnold. Introduction to Linear Algebra, 5th edition. Addison-Wesley, 2002.
- Anton and Rorres. Elementary Linear Algebra: Applications Version, 11thedition. Wiley, 2013.

# 2 HTML5 和 Canvas

WebGL 是现在图形 API，兼具质量和性能，直接访问图形硬件。For these reasons, WebGL can serve as an excellent base to support drawing in a game engine, especially for video games that are designed to be played acrossthe Internet.

## Canvas 和绘制

使用 canvas 定义 WebGL 绘制的区域。

例子在 Chapter2/2.1.HTML5Canvas

```html
<canvas id="GLCanvas" width="640" height="480">Your browser does not support the HTML5 canvas.</canvas>
```

```js
var canvas = document.getElementById("GLCanvas");
// 获取 WebGL 上下文
var gl = canvas.getContext("webgl") || canvas.getContext("experimental-webgl");
// 以某个颜色清除整个画布
if (gl !== null) {	gl.clearColor(0.0, 0.8, 0.0, 1.0); // set the color to be cleared	gl.clear(gl.COLOR_BUFFER_BIT); // clear the colors}
```

颜色是RGBA模式。

更结构化的定义：

```js
function initializeGL() {	var canvas = document.getElementById("GLCanvas");	gGL = canvas.getContext("webgl") ||	canvas.getContext("experimental-webgl");
	if (gGL !== null) {		gGL.clearColor(0.0, 0.8, 0.0, 1.0); // set the color to be cleared	} else {		document.write("<br><b>WebGL is not supported!</b>");	}}
function clearCanvas() {	gGL.clear(gGL.COLOR_BUFFER_BIT); // clear to the color previously set}
function doGLDraw() {	initializeGL();	clearCanvas();}
```

## 使用 WebGL 做基本绘制

使用 WebGL 绘制是一个多阶段的过程，involves transferring geometric data and OpenGL Shading Language (GLSL) instructions (the shaders) from memory to the drawing hardware, or the graphical processing unit (GPU). 该过程需要大量的 WebGL 函数调用。This section presents the WebGL drawing steps in detail. It is important to focus on learning these basic steps and avoid being distracted by the less important WebGL configuration nuances such that you can continue to learn the overall concepts and build the game engine.In the following project, you will learn about drawing with WebGL by focusing on the most elementary operations, including the loading to the GPU for the simple geometry of a square, a constant color shader, and basic instructions of drawing a square as two triangles.

绘制一个矩形。例子在 Chapter2/2.3.DrawOneSquare

The goals of the project are as follows:- To understand how to load geometric data to the GPU
- To learn about simple GLSL shaders for drawing with WebGL
- To learn how to compile and load shaders to the GPU
- To understand the steps to draw with WebGL

**准备和加载几何数据**
为高效使用 WebGL，与几何相关的数据，如矩形的顶点的位置，应存储在 GPU 硬件中。In the following steps, you will create a contiguous buffer in the GPU, load the vertex positions of a unit square into the buffer, and store the reference to the GPU buffer in a global variable. A unit square is a 1×1 square centered at the origin. The corresponding JavaScript code will be stored in a new source code file, VertexBuffer.js.

```jsvar gSquareVertexBuffer = null; // 指向 WebGL 缓冲的位置

// 创建顶点并加载到GPU
function initSquareBuffer() {	// 首先，定义矩形的顶点	var verticesOfSquare = [		0.5, 0.5, 0.0, // 第三个值是z轴，2D游戏是0		-0.5, 0.5, 0.0,		0.5, -0.5, 0.0,		-0.5, -0.5, 0.0	];	// Step A: 为顶点位置创建一个GPU缓冲	gSquareVertexBuffer = gGL.createBuffer();	// Step B: Activate vertexBuffer	gGL.bindBuffer(gGL.ARRAY_BUFFER, gSquareVertexBuffer);	// Step C: Loads verticesOfSquare into the vertexBuffer	gGL.bufferData(gGL.ARRAY_BUFFER, new Float32Array(verticesOfSquare),
		gGL.STATIC_DRAW); // STATIC_DRAW 表示该缓冲不会更改}
```

**Set Up the GLSL Shaders**

术语着色器（shader）表示运行在GPU中的程序。就游戏引擎而言，着色器要定义两部分：一个 vertex shader 和相应的 fragment shader。The GPU will execute the vertex shader once per primitive vertex and the fragment shader once per pixel covered by the primitive. 例如，定义一个矩形，有四个顶点，这个矩形覆盖 100×100 像素的区域。要绘制这个矩形，WebGL 会调用顶点着色器4次（每个顶点一次），执行 fragment 着色器 10,000 次（每个像素一次）。
在 WebGL 中，顶点和碎片着色器都在 OpenGL Shading Language (GLSL) 中实现。GLSL 语言的语法类似于C语言，专门用来处理和显示图形 primitives。
In the following steps, you will load into memory the source code for both vertex and fragment shaders, compile and link them into a single shader program, and load the compiled program into the GPU. 在本例中，着色器源码定义在 index.html 文件，加载、编译、链接着色器定义在 ShaderSupport.js 文件。

> WebGL 上下文可以看做是 GPU 硬件的抽象。本书中，WebGL 和 GPU 两个词有时可以互换。

**Define the Vertex and Fragment Shaders**GLSL shaders are simply programs consisting of GLSL instructions.定义顶点着色器，打开 index.html，在 body 中添加，

```<script type="x-shader/x-vertex" id="VertexShader">attribute vec3 aSquareVertexPosition;void main(void) {	gl_Position = vec4(aSquareVertexPosition, 1.0);}</script>
```
注意到 `type` 设为 `x-shader/x-vertex`。As you will see, the `id` field with the value `VertexShader` allows you to identify and load this vertex shader into memory.The GLSL `attribute` keyword identifies per-vertex data that will be passed to the vertex shader in the GPU. In this case, the `aSquareVertexPosition` attribute is of data type `vec3` or an array of three floating-point numbers. As you will see in later steps, `aSquareVertexPosition` will contain vertex positions for a square.
`gl_Position` 是 GLSL 内建的变量，是一个四个浮点数的数组，表示顶点位置。In this case, the fourth position of the array will always be 1.0. The code shows the shader converting the `aSquareVertexPosition` into a `vec4` and passing the information to WebGL.

定义碎片着色器。

```<script type="x-shader/x-fragment" id="FragmentShader">void main(void) {	gl_FragColor = vec4(1.0, 1.0, 1.0, 1.0);}</script>
```
Note the different `type` and `id` fields. Recall that the fragment shader is invoked once per pixel. The variable `gl_FragColor` is the built-in variable that determines the color of the pixel. 这里，四个1表示白色。

With both the vertex and fragment shaders defined in the index.html file, you are now ready to implement the functionality to compile, link, and load the resulting shader program to the GPU.

**编译、链接、加载顶点和碎片着色器**
创建新文件 ShaderSupport.js。
```jsvar gSimpleShader = null; // 着色器程序var gShaderVertexPositionAttribute = null; // the vertex position attribute in the GPU

// a function to load and compile a shader from index.html
// 调用该函数时，id 传 VertexShader 或 FragmentShader
function loadAndCompileShader(id, shaderType) {	var shaderText, shaderSource, compiledShader;	// Step A: 获取 index.html 中某个着色器的源代码	shaderText = document.getElementById(id);	shaderSource = shaderText.firstChild.textContent;	// Step B: 创建特定类型的着色器：vertex 或 fragment	compiledShader = gGL.createShader(shaderType);	// Step C: 编译着色器	gGL.shaderSource(compiledShader, shaderSource);	gGL.compileShader(compiledShader);	// Step D: 检查错误和结果	if (!gGL.getShaderParameter(compiledShader, gGL.COMPILE_STATUS)) {		alert("A shader compiling error occurred: " +		gGL.getShaderInfoLog(compiledShader));	}	return compiledShader;}

function initSimpleShader(vertexShaderID, fragmentShaderID) {	// Step A: 产生两个着色器	var vertexShader = loadAndCompileShader(vertexShaderID, gGL.VERTEX_SHADER);	var fragmentShader = loadAndCompileShader(fragmentShaderID,gGL.FRAGMENT_SHADER);	// Step B: 将着色器链接到一个程序	gSimpleShader = gGL.createProgram(); // gSimpleShader 是全局变量	gGL.attachShader(gSimpleShader, vertexShader);	gGL.attachShader(gSimpleShader, fragmentShader);	gGL.linkProgram(gSimpleShader);	// Step C: 检查错误	if (!gGL.getProgramParameter(gSimpleShader, gGL.LINK_STATUS))		alert("Error linking shader");	// Step D: Gets a reference to the aSquareVertexPosition attribute	gShaderVertexPositionAttribute = gGL.getAttribLocation(gSimpleShader,"aSquareVertexPosition");	// Step E: Activates the vertex buffer loaded in VertexBuffer.js	gGL.bindBuffer(gGL.ARRAY_BUFFER, gSquareVertexBuffer);	// Step F: Describe the characteristic of the vertex position attribute	gGL.vertexAttribPointer(gShaderVertexPositionAttribute,		3, // 每个顶点是三个浮点数 (x,y,z)		gGL.FLOAT, // 数据类型是 FLOAT		false, // if the content is normalized vectors		0, // number of bytes to skip in between elements		0); // offsets to the first element}```The shader loading and compiling functionality is now defined. You can now activate these functions to draw with WebGL.

**Set Up Drawing with WebGL**
定义好顶点数据和着色器后，下面可以通过以下步骤使用 WebGL 绘制。Recall from the previous project that the initialization and drawing code is stored in the WebGL.js file. Now open this file for editing.
修改 `initializeGL()` 函数，加入初始化顶点缓存和着色器程序。

```jsfunction initializeGL() {	var canvas = document.getElementById("GLCanvas");	gGL = canvas.getContext("webgl") || canvas.getContext("experimental-webgl");	if (gGL !== null) {		gGL.clearColor(0.0, 0.8, 0.0, 1.0); // set the color to be cleared		// A. initialize the vertex buffer		initSquareBuffer(); // This function is defined VertexBuffer.js		// B. now load and compile the vertex and fragment shaders		initSimpleShader("VertexShader", "FragmentShader");		// initSimpleShader() function is defined in ShaderSupport.js	} else {		document.write("<br><b>WebGL is not supported!</b>");	}}
```
Replace the `clearCanvas()` function with the `drawSquare()` function for drawing the defined square.

```jsfunction drawSquare() {	gGL.clear(gGL.COLOR_BUFFER_BIT);	// Step A: Activate the shader to use	gGL.useProgram(gSimpleShader);	// Step B: Enable the vertex position attribute	gGL.enableVertexAttribArray(gShaderVertexPositionAttribute);	// Step C: Draw with the above settings	gGL.drawArrays(gGL.TRIANGLE_STRIP, 0, 4); // 绘制四个顶点，以两个相连的三角的形式，形成一个矩形}
```

Lastly, modify `doGLDraw()` to call the `drawSquare()` function.

```jsfunction doGLDraw() {	initializeGL(); // Binds gGL context to WebGL functionality	drawSquare(); // Clears the GL area and draws one square}
```

运行后结果是一个绿色画布上的白色矩形。我们 1x1 矩形的顶点位置是 (±0.5, ±0.5)。观察输出，白色矩形在画布中间，覆盖画布宽高一半的区域。于是，如果顶点在 ±1.0，则绘制覆盖整个画布。因为画布是 640×480，4:3 的，因此目前矩形也是 4:3 的。

## 用 JavaScript 对象做抽象

把上一章的全局变量、函数等，引入OOP，做成一些对象。

源码在 Chapter2/2.4.JavaScriptObjects。

本章目标：分离游戏引擎和游戏逻辑代码。放在不同文件夹下，src/Engine 放引擎代码。src/MyGame 放游戏逻辑代码。几个 JavaScript 对象抽象游戏引擎的功能：Core, VertexBuffer, SimpleShader。

**引擎核心**

包括一次性的初始化 WebGL（或GPU），共享的资源，工具函数等。
创建文件 src/Engine/Engine_Core.js。

```js
// gEngine是命名空间var gEngine = gEngine || { };

gEngine.Core = (function() {	// instance variable: the graphical context for drawing	var mGL = null;	var getGL = function() { return mGL; };	// 公开的成员	var mPublic = {		getGL: getGL	};	return mPublic;}());

```

在 gEngine.Core 对象中，添加一个函数，

```js// 初始化 WebGL、顶点缓冲，编译着色器var initializeWebGL = function(htmlCanvasID) {	var canvas = document.getElementById(htmlCanvasID);	mGL = canvas.getContext("webgl") || canvas.getContext("experimental-webgl");	if (mGL === null) {		document.write("<br><b>WebGL is not supported!</b>");		return;	}	// now initialize the VertexBuffer	gEngine.VertexBuffer.initialize();};
```
新增一个函数，将画布清空为指定颜色，

```js// Clears the draw area and draws one squarevar clearCanvas = function(color) {	mGL.clearColor(color[0], color[1], color[2], color[3]);	mGL.clear(mGL.COLOR_BUFFER_BIT); // clear to the color previously set};
```
修改导出的函数：

```jsvar mPublic = {	getGL: getGL,	initializeWebGL: initializeWebGL,	clearCanvas: clearCanvas};
```

**共享的顶点缓冲**我们的游戏引擎，所有的图形对象的绘制都基于单位矩形。因此几何子系统相当简化。`gEngine.VertexBuffer` 对象实现几何子系统。
创建 `src/Engine/Engine_VertexBuffer.js`。

```jsvar gEngine = gEngine || { };// The VertexBuffer objectgEngine.VertexBuffer = (function() {
	// 定义矩形的顶点	var verticesOfSquare = [		0.5, 0.5, 0.0,		-0.5, 0.5, 0.0,		0.5, -0.5, 0.0,		-0.5, -0.5, 0.0	];	// reference to the vertex positions for the square in the gl context	var mSquareVertexBuffer = null;	var getGLVertexRef = function() { return mSquareVertexBuffer; };	var initialize = function() {		var gl = gEngine.Core.getGL();		// Step A: Create a buffer on the gGL context for our vertex positions		mSquareVertexBuffer = gl.createBuffer();		// Step B: Activate vertexBuffer		gl.bindBuffer(gl.ARRAY_BUFFER, mSquareVertexBuffer);		// Step C: Loads verticesOfSquare into the vertexBuffer		gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(verticesOfSquare),			gl.STATIC_DRAW);	};	var mPublic = {		initialize: initialize,		getGLVertexRef: getGLVertexRef	};	return mPublic;}());
```
上面定义的 `gEngine.VertexBuffer.initialize()` 函数在上一节定义的 `Core.initializeWebGL()` 函数最后调用了。

**着色器对象**
创建新文件 src/Engine/SimpleShader.js。

定义 SimpleShader 对象，加载、编译、链接着色器到一个程序。

```jsfunction SimpleShader(vertexShaderID, fragmentShaderID) {	this.mCompiledShader = null; // reference to the compiled shader in webgl context	this.mShaderVertexPositionAttribute = null; // reference to SquareVertexPosition in shader	var gl = gEngine.Core.getGL();	// Step A: 加载和编译顶点和碎片着色器	var vertexShader = this._loadAndCompileShader(vertexShaderID,
		gl.VERTEX_SHADER);
	var fragmentShader = this._loadAndCompileShader(fragmentShaderID,
		gl.FRAGMENT_SHADER);	// Step B: Create and link the shaders into a program.	this.mCompiledShader = gl.createProgram();	gl.attachShader(this.mCompiledShader, vertexShader);	gl.attachShader(this.mCompiledShader, fragmentShader);	gl.linkProgram(this.mCompiledShader);	// Step C: check for error	if (!gl.getProgramParameter(this.mCompiledShader, gl.LINK_STATUS)) {		alert("Error linking shader");		return null;	}	// Step D: Gets a reference to the aSquareVertexPosition attribute	this.mShaderVertexPositionAttribute =
		gl.getAttribLocation(this.mCompiledShader, "aSquareVertexPosition");	// Step E: Activates the vertex buffer loaded in Engine.Core_VertexBuffer	gl.bindBuffer(gl.ARRAY_BUFFER, gEngine.VertexBuffer.getGLVertexRef());	// Step F: Describe the characteristic of the vertex position attribute	gl.vertexAttribPointer(this.mShaderVertexPositionAttribute,		3, // each element is a 3-float (x,y.z)		gl.FLOAT, // data type is FLOAT		false, // if the content is normalized vectors		0, // number of bytes to skip in between elements		0); // offsets to the first element}
```

`_loadAndCompileShader()` 方法：

```js// Returns a complied shader from a shader in the dom.// The id is the id of the script in the html tag.SimpleShader.prototype._loadAndCompileShader = function(id, shaderType) {	var shaderText, shaderSource, compiledShader;	var gl = gEngine.Core.getGL();	// Step A: Get the shader source from index.html	shaderText = document.getElementById(id);	shaderSource = shaderText.firstChild.textContent;	// Step B: Create the shader based on the shader type: vertex or fragment	compiledShader = gl.createShader(shaderType);	// Step C: Compile the created shader	gl.shaderSource(compiledShader, shaderSource);	gl.compileShader(compiledShader);	// Step D: check for errors and return results (null if error)	// The log info is how shader compilation errors are typically displayed.	// This is useful for debugging the shaders.	if (!gl.getShaderParameter(compiledShader, gl.COMPILE_STATUS)) {		alert("A shader compiling error occurred: " +		gl.getShaderInfoLog(compiledShader));	}	return compiledShader;};
```
Add a function to activate the shader for drawing.

```jsSimpleShader.prototype.activateShader = function() {	var gl = gEngine.Core.getGL();	gl.useProgram(this.mCompiledShader);	gl.enableVertexAttribArray(this.mShaderVertexPositionAttribute);};
```

Finally, add an accessor for the actual WebGL shader program.

```js
SimpleShader.prototype.getShader = function() {
	return this.mCompiledShader; };
```
**应用逻辑**

创建文件 src/MyGame/MyGame.js

```jsfunction MyGame(htmlCanvasID) {	// The shader for drawing	this.mShader = null;
	gEngine.Core.initializeWebGL(htmlCanvasID);
	this.mShader = new SimpleShader("VertexShader", "FragmentShader");
	
	// Step C1: Clear the canvas	gEngine.Core.clearCanvas([0, 0.8, 0, 1]);	// Step C2: Activate the proper shader	this.mShader.activateShader();
	// Step C3: Draw with the currently activated geometry and the activated shader	var gl = gEngine.Core.getGL();	gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);}
```
Remember to create the MyGame object from within the index.html body element.

```html<body onload="new MyGame('GLCanvas');">
```

## 从 HTML 中分离 GLSL

之前 GLSL 着色器代码嵌入在 HTML 源码中。本节将其分离。
示例代码在 Chapter2/2.5.ShaderSourceFiles。

**Loading Shaders in SimpleShader**
修改 `_loadAndCompileShader()`，从独立的文件中加载 GLSL 着色器。
```jsSimpleShader.prototype._loadAndCompileShader = function(filePath, shaderType)
```

通过 `XMLHttpRequest` 加载 GLSL 文件。代码略。
创建文件夹 `src/GLSLShaders`，在其中创建两个文件 SimpleVS.glsl 和 WhiteFS.glsl。GLSL 着色器源文件代码扩展名是 `.glsl`。文件名中的 `VS` 表示 vertex shader，`FS` 表示 fragment shader。

SimpleVS.glsl 内容如下：

```attribute vec3 aSquareVertexPosition; // Expects one vertex positionvoid main(void) {	gl_Position = vec4(aSquareVertexPosition, 1.0);}
```
WhiteFS.glsl 内容如下：

```void main(void) {	gl_FragColor = vec4(1.0, 1.0, 1.0, 1.0);}
```

编辑 MyGame.js，

```jsthis.mShader = new SimpleShader(	"src/GLSLShaders/SimpleVS.glsl", // Path to the VertexShader	"src/GLSLShaders/WhiteFS.glsl"); // Path to the FragmentShader
```

## 修改着色器、控制颜色

可以绘制任意颜色。源文件见 Chapter2/2.6.ParameterizedFragmentShader。

新建碎片着色器 SimpleFS.glsl，替换 WhiteFS.glsl。

```precision mediump float; // 设置浮点计算的精度uniform vec4 uPixelColor; // to transform the vertex positionvoid main(void) {	gl_FragColor = uPixelColor;}
```
Recall that the GLSL `attribute` keyword identifies data that changes for every vertex position. 这里的 `uniform` 表示定义的变量对所有顶点不变。`uPixelColor` 变量可以通过 JavaScript 设置，控制最终的像素颜色。
> Note Floating-point precision trades the accuracy of computation for performance.

**修改 SimpleShader 支持颜色参数**
编辑 SimpleShader.js，添加一个实例变量，

```jsthis.mPixelColor = null;
```
在构造器的最后添加代码

```js
// Step G: Gets a reference to the uniform variable uPixelColor in the// fragment shaderthis.mPixelColor = gl.getUniformLocation(this.mCompiledShader,
	"uPixelColor");```
修改着色器的激活，设置像素颜色。

```jsSimpleShader.prototype.activateShader = function (pixelColor) {	var gl = gEngine.Core.getGL();	gl.useProgram(this.mCompiledShader);	gl.enableVertexAttribArray(this.mShaderVertexPositionAttribute);	gl.uniform4fv(this.mPixelColor, pixelColor);};
```

`gl.uniform4fv()` 函数拷贝四个浮点数的值，从 `pixelColor` 到 `mPixelColor`。
**使用新的着色器绘制**
修改 MyGame 构造器，使用新的着色器，

```jsfunction MyGame(htmlCanvasID) {	// Step A: Initialize the webGL Context and the VertexBuffer	gEngine.Core.initializeWebGL(htmlCanvasID);	// Step B: Create, load and compile the shaders	this.mShader = new SimpleShader(		"src/GLSLShaders/SimpleVS.glsl", // Path to the VertexShader		"src/GLSLShaders/SimpleFS.glsl"); // Path to the FragmentShader	// Step C: Draw!	// Step C1: Clear the canvas	gEngine.Core.clearCanvas([0, 0.8, 0, 1]);	// Step C2: Activate the proper shader	this.mShader.activateShader([0, 0, 1, 1]);	// Step C3: Draw with the currently activated geometry and the activated shader	var gl = gEngine.Core.getGL();	gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);}
```







	