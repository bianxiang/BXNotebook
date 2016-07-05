[toc]

## table

本文主要介绍 `display: table` 和 `display :table-cell`。

参考：

- http://www.uisdc.com/css-tables

CSS table 不是新的东西，它是CSS2.1章程里的内容。它的兼容性很强，除了IE7及以下版本，其他的浏览器它都可以使用。

你不能够在table-cells中再套其他的table-cells。Float对table-cell无影响。Margin对table-cell的元素无影响。

**纵向居中内容**

用`display：table`把容器中内容横向纵向居中十分简单。方法是：

    <div class="tbl">
        <h1 class="cell">Yes, I'm centered</h1>
    <div>

    .tbl {
        display: table;
        width: 100%;
        table-layout: fixed;
        background-color: hotpink;
        height: 8rem;
    }
    .cell {
        display: table-cell;
        vertical-align: middle;
        text-align: center;
    }

**Fixed – fluid – fixed 布局**

一个表格里包含三个表格单元格：第一列和第三列固定宽度；第二列没有设定宽度，于是它占用了其他可用的空间。
另外，无论table宽度设为多少，嵌在table-cell的元素都会撑到table高度。

以下是相关HTML和CSS代码：

    <div class="tbl">
        <div class="cell fixed1"></div>
        <div class="cell fuild"></div>
        <div class="cell fixed2"></div>
    </div>
    .tbl {
        display: table;
        width: 100%;
        table-layout: fixed;
    }
    .cell {
        display: table-cell;
        height: 300px;
    }
    .fixed1 {
        background-color: #555;
        width: 200px;
    }
    .fluid {
        background-color: #888;
    }
    .fixed2 {
        background-color: #222;
        width: 150px;
    }
