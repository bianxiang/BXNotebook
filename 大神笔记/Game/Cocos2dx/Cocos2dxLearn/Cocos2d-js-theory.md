[toc]

xxx 4.1 Cocos2d-JS项目结构介绍

xxx 4.2 cc.game对象和游戏启动流程

xxx 4.3 Cocos2d-JS坐标系统

xxx 4.4 Cocos2d-JS的屏幕适配方案

xxx 4.5 Cocos2d-JS场景树

xxx 4.6 游戏导演

xxx 事件分发机制
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/eventManager/zh.md

xxx 属性风格API

xxx cc.game：简化游戏的启动流程
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/cc-game/zh.md

xxx Cocos2d-js v3.0的对象构造和类继承
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/inheritance/zh.md

xxx Cocos2d-JS v3.0的新Action API
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/cc-actions/zh.md

xxx 项目配置文件：project.json
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/project-json/zh.md

xxx 基础数据类型
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/basic-data/zh.md

xxx log

xxx 统一create函数
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/create-api/zh.md

xxx Cocos2d-JS v3.0中的单例对象
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/singleton-objs/zh.md

xxx 对象缓冲池
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/cc-pool/zh.md

xxx 如何在android平台上使用js直接调用Java方法

xxx cc.sys
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/cc-sys/zh.md

xxx cc.loader
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/cc-loader/zh.md

xxx cc.path
http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/cc-path/zh.md

xxx CCFileUtils

## （未）资源热更新管理器

http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/assets-manager/zh.md

## Bake功能使用说明

设计意图

在游戏开发的过程中，经常会遇到作为UI或者不怎么修改的背景的层(Layer)， 这些层内容并不怎么变动。 而在游戏的渲染过程中，这些层往往又会消耗大量的渲染时间，特别是比较复杂的UI界面，比如：在Canvas渲染模式中，一个Button会调用9次绘图(drawImage)。在复杂一些的UI场景中，会出现UI的绘图次数远大于实际游戏的绘图次数的情况，这对于性能资源非常稀缺的手机浏览器来说，会带来灭顶之灾。

对于上述情况，我们给cc.Layer加入了bake的接口， 调用了该接口的层，会将自身以及其子节点都备份（烘焙/bake)到一张画布(Canvas)上，只要自身或子节点不作修改，下次绘制时，将直接把画布上的内容绘制上去。这样，原来需要调用N次绘图的层，就只需要调用一次绘图了。 当该层不需要bake时，可以调用unbake来取消该功能。

使用场景

复杂UI层。 UI层含有大量的面板(Panel)，按钮(Button)等，这些控件的绘制会有大量的绘图调用，而这些控件并不经常修改。

游戏中作为静态的背景或障碍物的层。

使用bake功能非常简单: 将需要bake的节点元素加入到一个cc.Layer或其子类(`cc.LayerColor`, `cc.LayerGradient`)对象中，然后调用该对象的bake函数就可以了。 示例代码：

```js
var bakeLayer = cc.Layer.create();
this.addChild(bakeLayer);

for(var i = 0; i < 9; i++){
   var sprite1 = cc.Sprite.create(s_pathGrossini);
   if (i % 2 === 0) {
      sprite1.setPosition(90 + i * 80, winSize.height / 2 - 50);
   } else {
      sprite1.setPosition(90 + i * 80, winSize.height / 2 + 50);
   }
     sprite1.rotation = 360 * Math.random();
     bakeLayer.addChild(sprite1);
}
bakeLayer.bake();                //start the bake function
```

更多信息，可查看我们的测试例(js-tests)的Bake Layer test.

注意事项

对于子节点经常会变的层， 启用bake功能，会给游戏性能带来额外的开销，建议对于不常修改子节点的层才开启该功能。
该功能仅在Canvas渲染模式下有效, 在JSB与WebGL渲染模式下调用bake函数，不会产生效果。

## （未）如何在IOS平台上使用js直接调用OC方法

http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/reflection-oc/zh.md

## Web引擎模块化：moduleConfig.json

http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/moduleconfig-json/zh.md

该配置文件相当于v2版本中的jsloader.js。改造的目的是为了使得配置纯粹化，同时也能比较好的支持cocos-console、cocos-utils甚至是用户自定义脚本工具。

字段说明

module

配置各个模块的js列表。key名即为模块名称。这些key名将会在project.json的modules字段中使用。倘若不清楚project.json里面究竟有哪些模块可以配置，就可以直接查看该文件。

每个模块的配置对象是一个数组，数组项分两种，一种是模块名，一种是js路径。

例如：

```js
"menus" : [
    "core", "actions",
    "cocos2d/menus/CCMenuItem.js",
    "cocos2d/menus/CCMenu.js"
]
```

此配置的意思是，menus模块依赖于core和actions模块，并且自身包含cocos2d/menus/CCMenuItem.js和cocos2d/menus/CCMenu.js。


## （未）cc.async

http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/cc-async/zh.md

## CCSAXParser.js

http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/cc-saxparser/zh.md

**cc.saxParser**

将cc.SAXParser重构为单例对象：cc.saxParser

删除了tmxParse，preloadPlist，unloadPlist，getName，getExt， getList等方法。

Parser的统一入口函数规范为parse，并且传参内容即为需要解析的文本内容。

**cc.plistParser**

添加了cc.plistParser。cc.plistParser继承cc.saxParser，用于解析plist文本内容。

这样的改造是为了让Parser的职责更加纯粹，只管解析，也就是对字符串内容的处理，而不需要管资源的加载。资源加载统一被移到cc.loader进行管理了。

**CCTMXXMLParser.js**

关于Parser这点，tilemap也需要改造。目前还未进行改造。

## （未）cc.spriteFrameCache 改造说明

http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/cc-spriteframecache/zh.md


## （未）v3相对于v2版本的api变动

http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v3/more-change-from-v2-to-v3/zh.md