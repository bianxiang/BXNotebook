[toc]

本书技术不是很强。

# 1 响应式元素和媒体

## 使用百分比改变图片大小

图片的父容器宽度设定，图片设置最大宽度为100%，

```html
<div class="img-wrap" >	<img alt="robots image" class="responsive" src="robots.jpg" >	<p>Loremipsum dolor sit amet</p></div>
```

```css
img.responsive {	max-width: 100%;
	height: auto;}
```

## 响应式图片：Cookie + Javascript

需要服务器参与。

JavaScript 负责将屏幕的尺寸写到一个 cookie。When the client makes a request to the server for an image, it fires the PHP code to deliver the appropriate image.

```jsdocument.cookie = "screen_dimensions=" + screen.width + "x" + screen.height;```

##(?) 让视频相对于屏幕宽度

The video tag easily supports using a percent width. However, it requires that you have the video source on your website's host. If you have this available, this is easy.

```html<style>video {
	max-width: 100%;	height: auto;}</style><video width="320" height="240" controls="controls">	<source src="movie.mp4" type="video/mp4">	<source src="movie.ogg" type="video/ogg">	Your browser does not support the video tag.</video>
```

但视频来自第三方网站，在iframe中。
This method is called Intrinsic Ratios for Videos, created by Thierry Koblentz on A List Apart. You wrap the video inside an element that has an intrinsic aspect ratio, and then give the video an absolute position. This locks the aspect ratio, while allowing the size to be fluid.

将视频包在一个div中，设置div底部的padding为50%到60%，相对定位。让子元素，如 iFrame 或 object，宽高都为100%，决定定位。This makes the iFrame object completely fill in the parent element.
The following is the HTML code that uses an iframe tag to get a video from Vimeo:```html<div class="video-wrap">	<iframe src="http://player.vimeo.com/video/52948373?badge=0"		width = "800" height= "450" frameborder="0"></iframe></div>
```The following is the HTML code using the older YouTube object with markup:

```html<div class="video-wrap">	<object width="800" height="450">		<param name="movie" value="http://www.youtube.com/v/b803LeMGkCA?version=3&amp;hl=en_US">		</param>		<param name="allowFullScreen" value="true"></param>
		<param name="allowscriptaccess" value="always"></param>		<embed src="http://www.youtube.com/v/b803LeMGkCA?version=3&amp;hl=en_US" type="application/x-shockwaveflash"width="560" height="315" allowscriptaccess="always"allowfullscreen="true">		</embed>	</object></div>
```Both video types use the same CSS:

```css
.video-wrap {	position:relative;	padding-bottom: 55%;	padding-top: 30px;	height: 0;	overflow:hidden;}.video-wrap iframe,.video-wrap object,.video-wrap embed {	position:absolute;	top:0;	width:100%;	height:100%;}
```

{{设置padding-bottom是关键的；貌似不一定大于50%，当然效果不同；注意到设置height:100%没有用。除非 `.video-wrap` 的父元素也是定高的。}}

# （未）2 响应式的字体



