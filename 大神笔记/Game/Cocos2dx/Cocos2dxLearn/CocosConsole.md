[toc]

参考：
- http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/html5/v2/cocos-console/zh.md

## Cocos2d Console 使用手册

Cocos2d Console是一个可以为Cocos2d-JS极大简化游戏创建和环境配置过程的工具。

Cocos2d-JS提供了Cocos Console工具来简化html5和JSB游戏的创建和开发。开发者可以使用它来创建新项目，编译并发布游戏到android，iOS，Mac OS，iOS或者Web平台。并且它非常简单易用，下面将会展示它的安装与使用。

### 安装

首先，你需要安装这个工具，当开发者将Cocos2d-JS仓库下载下来以后，会在根目录下发现`setup.py`安装文件。打开终端并进入Cocos2d-JS文件夹，然后运行`./setup.py`。在安装过程中，你可能需要提供你的NDK，Android SDK和ANT目录。请注意，这个工具是使用python来开发的，你将需要首先安装python 2.7，但是`setup.py`并不支持python3。

成功安装以后，开发者就可以开始在终端中使用`cocos`命令。

### 创建一个新项目

创建一个Cocos2d-JS项目:
`cocos new projectName -l js`

创建一个仅支持web平台的项目:
`cocos new projectName -l js --no-native`

创建项目到指定目录:
`cocos new projectName -l js -d ./Projects`

### 运行项目

使用浏览器运行web版项目:

```
cd directory/to/project
cocos run -p web
```

使用Closure Compiler高级模式压缩脚本并发布web版本 :

```
cd directory/to/project
cocos compile -p web -m release --advanced
```

编译并将项目运行在native平台上:

```
cd directory/to/project
cocos compile -p ios|mac|android|win32
cocos run -p ios|mac|win32|android
```

选项

    -p platform : 平台：ios|mac|android|win32|web.
    -s source   : 项目目录，如果没有指明会使用当前路径。
    -q          : 静默模式，不打印log信息。
    -m mode     : 选择debug或release模式，默认为debug模式
    --source-map: 生成source-map文件。（仅限Web平台）
    --advanced  : 使用Closure Compiler高级模式压缩脚本。（仅限Web平台）

### 帮助命令

如果你对使用有任何疑问，可以在命令后使用-h来获取对应命令的帮助。下面是所有的命令：

- 创建：new
- 编译：compile
- 运行：run