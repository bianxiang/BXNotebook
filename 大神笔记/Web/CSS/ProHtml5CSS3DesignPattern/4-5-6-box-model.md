[toc]

## 4. 盒模型

6种盒子：inline, inline-block, block, table, absolute, floated。任何元素都将被渲染成上述盒子的一种。某些元素被渲染为某种盒子的标题，如列表项或表格单元格。

`display`决定元素渲染成哪种盒子。`position:absolute`或`position:fixed`将元素渲染为绝对盒子。`float:left`或`float:right`将元素渲染为浮动盒子。

### 4.1 display

	SELECTOR { display:inline; }
    SELECTOR { display:inline-block; }
    SELECTOR { display:block; }
    SELECTOR { display:list-item; }
    SELECTOR { display:none; }

还有其他显示类型，但不被广泛支持。IE7不支持**run-in**和**inline-table**；还不支持：table, table-cell, table-row, table-header-group, table-footer-group, table-row-group, table-column-group, table-column, and table-caption.

When you display an element as a list item, its parent needs to be rendered as a block and needs to provide left padding or left margin for the marker.

A browser renders a list-item as a block with an inline marker. When you want a **list-item** to look like a block, you can simply turn off the marker using `list-style-type:none` — you do not need to change the display type because a list is already a block.

### 4.2 盒模型

`width`和`height`是内盒宽度和高度。`padding`透明，显示元素的背景。`margin`透明，显示元素父级的背景。margin外是元素外盒。

### 4.3 内联盒子

Inline, inline element, and static inline box are synonyms with inline box.

内联盒子在inline flow中渲染。超出父容器（终端块）的宽度会自动换行。This is called the **inline formatting context**.

width, height, overflow对内联元素无效，因为它们总是收缩到内容的宽度和高度。

水平margins改变内联元素在流中的位置。`margin-top`和`margin-bottom`对内联元素无效。内联元素使用`line-height`确定行高。

水平边框改变内联元素在流中的位置。上下边框不会撑高行高或改变内联元素的垂直位置。为此，边框可能与相邻行交叠。When a bordered element is wrapped across lines, the browser does not render the right border at the end of the line, and it does not render the left borderat the beginning of the wrapped line. The left and right borders occur only at the beginning and end of the element.

padding对内联元素的作用与border相同。{{上下padding不会撑高行}}

### 4.4 内联块状盒子

内联块状元素包括**replaced元素**和显示为内联块的内联元素。Also, you can use `display:inline-block` to display any inline element as a block rendered **within an inline context**.

注意对于**replaced元素**不必设置`display:inline-block`，浏览器会自动设置。

内联块状盒子像内联盒子一样参与inline flow，但可以有margins, borders, padding, width, height。一个内联块盒子不能跨多行换行。内联块盒子会撑高行的高度容纳它的height, padding, borders, margins。一个内联块盒子可以可以是收紧的、指定大小的、伸开的。

可以设置`width`和`height`。把高度或宽度设为固定数值，可以增大或缩小一个replaced元素，如图像。设置`width:auto`和`height:auto`，让replaced元素恢复其原来尺寸；对其他内联块元素，则是收紧它们到内存的尺寸。通过`width:100%`可以让一个内联块撑开。一个撑开的内联块就像一个块。

正的`margin-top`扩展行高，负的减小。正的`margin-bottom`抬高元素位置，负的降低；同时，会撑高或减少行高。

border和padding会增大内联元素的外尺寸。同时会抬高元素位置，增加包围它的行的高度。

### 4.5 块状盒子

块盒子从上到下在块格式化上下文中流动。这被称作块的正常（normal）流。

终端块在其内盒中创建内联格式化上下文，但在其外盒的外边，是块格式化上下文。

`width`设置元素宽度。`width:auto`是默认值，默认伸开撑满父容器宽度。
`height`设置元素高度。`height:auto`是默认值，默认收缩到内容高度。

**不能水平收紧一个块装盒子。**

浏览器会折叠相邻块的上下margin。

对于一个大小固定的块。`margin-right:auto`令其左对齐，`margin-left:auto`令其右对齐。同时设置`margin-left`和`margin-right`为`auto`，令其居中。

对于一个撑满的块，水平边框和padding压缩内盒大小。对于大小固定的盒子，它们使得内盒偏移（offset）。

### 4.6 表格盒子

表格参与块流，其单元格参与**table flow of rows and columns**。

表格可以有margin但不能有padding。单元格可以有padding但不能有margin。本节只关注表格的外面，表格与周围元素的交互。内部布局键第15和16章。

表格`width`包含边框（外缘）。`height`也一样。

`margin`的作用取决于表格是固定大小、收紧还是撑开。当大小固定或收紧时，margin偏移表格及后面的元素。负的margin可能导致表格与相邻元素重叠。若表格撑开，margin缩进表格，即减少内部大小，收缩单元格。

当表格大小固定或撑开时，边框减小表格的内盒。当表格收紧时，边框作用与其他盒子一样，增加表格外盒的大小。

overflow对表格没用，因为表格不能溢出。只有表格单元格可以溢出。`overflow:hidden` should be applied to cells to ensure consistent behavior in all browsers when fixed cells overflow.

### 4.7 绝对盒子

绝对元素从正常流中删除，放入上面或下面的层。它的位置相对于最近的定位祖先或固定到视口。它可以固定大小、收紧或撑满到最近的定位祖先。任何元素都可以绝对定位。绝对盒子的位置不影响其他盒子的位置。

`z-index`控制一个已定位元素的堆叠顺序。负值将它们置于正常（normal）流之下，正值置于之上。值越大越靠近用户。

当left和right都被设为`auto`，浏览器在盒子本来该在的位置上渲染它。

`width:auto`是默认值。

- 当`width`是`auto`且`left`和`right`都是`auto`，盒子会被收紧。
- 如果`width`是`auto`但`left`和`right`是0或其他值，盒子伸开。
- 如果`width`是一个值，`left`是一个值，但`right`是`auto`，盒子大小固定，对齐左边的一个位置。右对齐类似。

`height`、`top`、`bottom`的作用与width, left, right类似。

正的`margin`将绝对盒子的一边向容器中心移动。负值令其远离中心。

对于撑开的绝对盒子，边框和padding收缩内核。对于大小固定和收紧的绝对盒子，边框和padding增加外盒大小，令这些盒子向容器中心移动。

### 4.8 浮动盒子

任何元素都可以通过`float:left`或`float:right`浮动。浮动元素从正常（normal）流移除，置于相邻的（adjacent）块的**边框和背景之上**。浮动会令浮动的父元素减小：如果所有后代都是浮动的，父级会缩小到空。

尽管浮动从流中移除，它会缩进流中相邻的内容。左浮动使得相邻内容向右缩进，右浮动反之。

浮动在垂直方向上位于它本来的位置。水平位置位于父元素padding区的左边和右边。A float stacks next to other floats in the same general vertical location. 若一个浮动A不能在浮动B旁边呆的下，浮动A会移动到浮动B之下。浮动元素的位置、大小、padding、边框和margin影响相邻浮动和相邻**内联内容**的位置。浮动的精确位置无法预知。

`width:auto`是默认值，会令浮动元素收缩到容纳其最宽的行。
`height:auto`是默认值，令浮动元素收紧。

正的margin令浮动远离其对齐点，把其他浮动和内联内容推离它。A negative margin pulls the float to the other side of its point of alignment and pulls other floats and inline content closer. 浮动周围的margin不会合并。

边框和padding增加浮动的外尺寸。The left border and padding of a left float move the float to the right, and its right border and padding move other floats and inline content on the right further to the right. This applies vice versa for right floats. Top border and padding move the float down. The bottom border and padding move down any floats below the float, and extend the float’s effect on adjacent content in the normal flow.

## 5. 盒模型范围（extent）

本章介绍如何收紧、伸开盒子或设置盒子的大小。

### 5.1 宽度

    *.static-inline-shrinkwrapped { width:auto; }
    *.replaced-inline-shrinkwrapped { width:auto; }
    *.replaced-inline-sized { width:35px; }
    *.replaced-inline-stretched { width:100%; }
    *.static-block-sized { width:300px; }
    *.static-block-stretched { width:auto; }
    *.table-shrinkwrapped { width:auto; }
    *.table-sized { width:300px; }
    *.table-stretched { width:100%; }
    *.float-shrinkwrapped { width:auto; float:left; }
    *.float-sized { width:300px; float:left; clear:both; }
    *.float-stretched { width:100%; float:left; clear:both; }
    *.absolute-shrinkwrapped { width:auto; left:0; right:auto; position:absolute; }
    *.absolute-sized { width:300px; left:0; right:auto; position:absolute; }
    *.absolute-stretched { width:auto; left:0; right:0; position:absolute; }

`width`对内联元素无效，对各种盒模型效果不同。`width:auto`是默认值。

`width:auto`水平收紧以下盒子：内联、内联块、浮动、表格、绝对盒子（`left`和`right`都设为`auto`）。
`width:auto`水平撑开块状盒子和绝对盒子（`left`和`right`都设为一个值，如0）。
`width:100%`把元素拉开到父容器宽度。浏览器不会自动调整宽度以保持元素撑开。元素的水平margins, borders, padding会导致其宽度超过父容器。

### 5.2 高度

    *.replaced-inline-shrinkwrapped { height:auto; }
    *.replaced-inline-sized { height:65px; }
    *.replaced-inline-stretched { height:100%; }
    *.block-shrinkwrapped { height:auto; }
    *.block-sized { height:65px; }
    *.block-stretched { height:100%; }
    *.table-shrinkwrapped { height:auto; }
    *.table-sized { height:65px; }
    *.table-stretched { height:100%; }
    *.float-shrinkwrapped { height:auto; float:left; }
    *.float-sized { height:65px; float:left; }
    *.float-stretched { height:100%; float:left; }
    *.absolute-shrinkwrapped { height:auto; top:0; bottom:auto; position:absolute; }
    *.absolute-sized { height:65px; top:0; bottom:auto; position:absolute; }
    *.absolute-stretched { height:auto; top:0; bottom:0; position:absolute; }

`height`对内联元素无效，对各种盒模型效果不同。`height:auto`是默认值。

`height:auto`垂直收紧以下盒子：inline, inline-block, block, floated, table, absolute（需要top和bottom都为auto）。
`height:auto`会垂直撑开绝对盒子（需要top和bottom设置一个数值，如0）。这是垂直撑开一个盒子最好的方式，因为`height:100%`有限制，但它只对绝对盒子有效。
`height:100%`将元素拉开到父容器高度，但有一些限制。浏览器不会自动调整高度让元素保持拉伸。元素的垂直margins, borders, padding会导致元素高度超出父容器。

### 5.3 指定大小

### 5.4 收紧

元素收紧到内容大小。

    // 收紧浮动
    SELECTOR { width:auto; height:auto; float:LEFT_RIGHT; }
    // 收紧内联
    INLINE-SELECTOR { width:auto; height:auto; }
    // 收紧内联块
    INLINE-BLOCK-SELECTOR { width:auto; height:auto; }
    // 垂直收紧块元素
    BLOCK-SELECTOR { height:auto; }
    // 收集表格
    TABLE-SELECTOR { width:auto; height:auto; }
    // 水平收紧绝对盒子
    SELECTOR { position:absolute; width:auto; left:0; right:auto;}
    // 或
    SELECTOR { position:absolute; width:auto; left:auto; right:0; }
    // 垂直收紧绝对元素
    SELECTOR { position:absolute; height:auto; top:0; bottom:auto;}
    // 或
    SELECTOR { position:absolute; height:auto; top:auto; bottom:0; }

不能水平收紧静态块元素。

限制块的大小的另一种方式是利用`max-height`或`max-width`属性。

### 5.5 撑满

将元素的外盒拉伸到容器边。

{{浮动是可以被水平或垂直撑开的。垂直撑开的浮动可以实现对齐一侧且高度撑满的效果}}

When using `width:auto` or `height:auto`, a browser calculates the width and height of stretched elements from the outside in. 浏览器从父元素的内盒开始，减去被撑开元素的margin、边框、内补，得到其内盒大小。与此不同的时，对于大小固定或收紧的元素，大小是从内向外计算的。

用`width:auto`拉开块元素。
用`width:auto; left:0; right:0`拉开决定元素到最近的定位祖先的左边和右边。
用`height:auto; top:0; bottom:0`拉开绝对元素到最近的定位祖先的上边和下边。
用`width:100%`拉开一个表格、**浮动**或**内联块**。需要盒子没有水平的margin，否则会超出父容器。对于浮动和内联块，水平边框和padding也会导致溢出。
用`height:100%`拉开内联块、块、表格、**浮动**的高度到容器高度。

    // 拉开内联块
    INLINE-BLOCK-SELECTOR { width:100%; height:100%; }
    // 拉开静态块
    BLOCK-SELECTOR { width:auto; height:100%; }
    // 拉开表格
    TABLE-SELECTOR { width:100%; height:100%; }
    // 垂直拉开绝对元素
    SELECTOR { height:auto; top:0; bottom:0; position:absolute; }
    // 水平拉开绝对元素
    SELECTOR { width:auto; left:0; right:0; position:absolute; }
    // 拉开浮动
    SELECTOR { width:100%; height:100%; float:LEFT_RIGHT; }

IE6不能拉开绝对元素。绝对定位的表格通过`width:100%`和`height:100%`可以拉开。

## 6. 盒模型属性

### Margin

如果margin的值是百分比，则相对于容器的宽度。

margin-top和margin-bottom对内联元素无效。

对于一个大小固定的块，margin-left和margin-right可令元素偏移（offset）。

对伸开的块或伸开的绝对元素，正的margin使元素缩进，负的使它们悬挂缩进。缩进减小元素的内盒，使边框和padding向内移动。

### Overflow

IE6的overflow:visible的实现有问题：不是溢出，而是令容器的宽度或高度增加以容纳内容。IE7修正了该问题。

CSS 3定义了两个新属性：overflow-x和overflow-y，用于替代overflow，使得水平和垂直策略可以分开处理。

### （未）Page Break




