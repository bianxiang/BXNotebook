[toc]

# Preface
RESS 尝试解决 Responsive Web Design (RWD) 的问题。
RESS Essentials 教你，如何让服务端更加智能的处理访问者环境的限制（设备、屏幕大小、浏览器）。
To run some examples from this book you need to have some kind of AMP(Apache, mySQL, PHP) environment. Links to the respective packages for other systems can be found at http://en.wikipedia.org/wiki/List_of_Apache–MySQL–PHP_packages.

# 1 Why Does RWD Change the Internet?

RWD 的基础是：fluid grids, flexible images, and media queries。

Fluid grid 的关键是将显示器宽度划分为几列。fluid 的关键是相对单位，如百分比、em 或 rem。

Images in such a layout become fluid by using a simple technique of setting width, `x%` or `max-width`, `100%` in CSS, which causes the image to scale proportionally.

除了屏幕看度，还要考虑带宽、机器性能、浏览器兼容性等。

**RESS (Responsive Web Design with Server Side components)** way. RESS was proposed by Luke Wroblewski on his blog as a result of his experiences on extending RWD with Server Side components.

设备侦测可以通过库，如 WURFL 或 YABFDL。

响应式设计一般需要 **respond.js** (available at https://github.com/scottjehl/Respond)，使不支持 CSS3 媒体查询的浏览器支持。

If you need a conditional resource, loading another JavaScript **Modernizr** (available at http://modernizr.com/) can help you.

栅格工具。ZURB CSS Grid Builder available at http://www.zurb.com/playground/css-grid-builder and Gridulator available at http://gridulator.com/ to name just two. A more extensive list can be found at http://www.thegridsystem.org/categories/tools/.

On 14 February 2013 Adobe released the public preview of a completely new tool: Edge Reflow (free at the time of writing this). Its sole purpose is to allow fast and easy creation of CSS and HTML for responsive layouts.

Originally RWD consisted of three basic technologies used in a somewhat defined way, shown as follows:- Fluid grids: Based on % measurements
- Flexible images: Scaled down with the CSS max-width trick
- Media queries made with philosophy Mobile First or Progressive Enhancement: That means code for the smallest screen was written first andthen features for larger screens were added

另外一些选择或方法：
- Some were not happy with fluid columns and made "frameless grids" (available at http://framelessgrid.com/), a CSS grid system with columns of fixed width
- Some decided that it's better to use `em` or `rem` based scaling to take the resolution out of the equation and made The Goldilocks Approach an HTML and CSS Boilerplate (available at http://goldilocksapproach.com/)
- Some thought that breaking a grid into bricks instead of columns may be more funny and made Masonry (available at http://masonry.desandro.com/)
- 一些人（包括本书作者），喜欢手写媒体查询，仅在需要时（设计需要是）调整布局{{而不是用统一的栅格}}

# 2 客户端 RWD 开发

## (未）Bootstrap定制编译

## (未）Integrating Gridpak

# （未）3 服务端 RWD 开发：设备侦测库

Hence, Server Side device detection toolkits became indispensable for mobile web developers. Those libraries rely on parsing user agent strings; they map this information to device and browser capabilities and group them into groups with similar levels of support for content to be served. The two most well-known device databases are DeviceAtlas (available at www.deviceatlas.com) and WURFL.


