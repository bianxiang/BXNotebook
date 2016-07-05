[toc]

# 介绍

The code is written in C++, Lua, and C#.

本书代码用 Visual Studio 2010 编写。Lua 用 Decoda IDE 编写。Much of the code in this book assumes that you are using Windows.

本书假设你使用的图形API是 DirectX 11 或后续版本。代码支持 Direct3D 9 和 Direct3D 11。不支持 OpenGL。

This book uses STL for common data structures.

前缀：

- g: Use with global variables — `g_Counter`- m: Use with member variables — `m_Counter`- p: Use with pointer variables — `m_pActor`- V: Use with virtual functions — `VDraw()`- I: Use with Interface classes — class `IDrawable`

Classes, Functions, Typedefs, and Methods: Always start with uppercase and capitalize each compound word— `SoundResource`, `MemoryFile`.

Visit the companion website for this book at http://www.mcshaffry.com/GameCode/, where you can find the most up-to-date resources for this book, especially the source code.
The source code for this book is hosted by Google Code at this address:http://code.google.com/p/gamecode4/You may download the companion website files from www.courseptr.com/downloads. Please note that you will be redirected to the Cengage Learning site.

# 1. 游戏编程是什么
