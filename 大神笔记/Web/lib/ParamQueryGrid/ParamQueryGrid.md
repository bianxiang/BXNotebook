[toc]

http://paramquery.com/

Version 1.1.3

## ParamQuery Grid

### 介绍

Its core design is engineered using virtual rendering that's why it's unique and is a great performer. It maintains its grace & responsiveness irrespective of number of records. It also exposes low level methods and events which makes it a great flexible and **extensible** tool.

PQ Grid采用jQueryUI widget设计模式。

### 设计

ParamQuery Grid的数据和视图是分离的。只显示和渲染视口内的行。在以下条件发生时刷新视图：

1. 视图水平或垂直滚动
1. 排序后
1. resize事件
1. 调用了`$( ".selector" ).pqGrid( "refresh" )`
1. 调用了`$( ".selector" ).pqGrid( "refreshDataAndView" )`

ParamQuery Grid把数据（本地或远程的）存放在选项**dataModel > data**中。数据结构是二维数组或一位JSON对象数组。在以下情况下刷新远程数据的本地拷贝：

1. 分页事件
1. 排序事件
1. 调用了`$( ".selector" ).pqGrid( "refreshDataAndView" )`

无论是本地请求还是远程请求，**dataModel > data**都可以被直接操纵。

`$( ".selector" ).pqGrid( "refreshDataAndView" )`一般在直接操纵了`dataModel`属性后调用，以保持数据和视图同步。`$( ".selector" ).pqGrid( "refresh" )`一般在修改了表格的DOM后调用。

ParamQuery Grid can sort the data locally for data loaded through remote request if all the data is loaded during initialization of the grid. In this case we pass the options `{ location:"remote", sorting:"local" }` to pqGrid.

ParamQuery Grid can also provide local paging functionality for data loaded through remote request if all the data is loaded during initialization of the grid. In this case we pass the options `{ location:"remote", paging:"local" }` to pqGrid.

ParamQuery Grid带有两个独立的jQueryUI widgets/components：**pqPager** 和 **pqScrollBar**。pqPager has class `pq-pager`, hence full funtionality of pqpager can be accessed by `$( ".pq-pager", $grid ).pqPager()`. Similarly full funtionality of vertical and horizontal scrollbars can be accessed by `$( ".pq-scrollbar-vert", $grid ).pqScrollBar()` & `$( ".pq-scrollbar-horiz", $grid ).pqScrollBar()`. These default components can also be replaced with pagers and scrollbars having custom logic!

### 引入文件

    <!--jQuery dependencies-->
    <link rel="stylesheet"
         href="http://ajax.googleapis.com/ajax/libs/jqueryui/1.9.2/themes/base/jquery-ui.css" />
    <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.8.3/jquery.min.js"></script>
    <script src="http://ajax.googleapis.com/ajax/libs/jqueryui/1.9.2/jquery-ui.min.js"></script>

    <!--Include Touch Punch file to provide support for touch devices-->
    <script type="text/javascript" src="path to touch-punch.js" ></script>

    <!--ParamQuery Grid files-->
    <link rel="stylesheet" href="path to pqgrid.min.css" />
    <script type="text/javascript" src="path to pqgrid.min.js" ></script>

首先是对jQuery和jQueryui的依赖。

Touch-punch.js file which can be obtained from https://github.com/furf/jquery-ui-touch-punch can be included if support for touch devices is desired. It works well for jQueryUI and ParamQuery Grid.

If you want to add custom theme (e.g. office theme) then you would require to add another css file below it.

### 入门

初始化表格：`$( ".selector" ).pqGrid( obj );`。

`colModel`和`dataModel`是`obj`的2个主要的键。

    <script>
    $(function(){
        var data = [ [1,'Exxon Mobil','339,938.0','36,130.0'],
                [2,'Wal-Mart Stores','315,654.0','11,231.0'],
                [3,'Royal Dutch Shell','306,731.0','25,311.0'],
                [4,'BP','267,600.0','22,341.0'],
                [5,'General Motors','192,604.0','-10,567.0'],
                [6,'Chevron','189,481.0','14,099.0'],
                [7,'DaimlerChrysler','186,106.3','3,536.3'],
                [8,'Toyota Motor','185,805.0','12,119.6'],
                [9,'Ford Motor','177,210.0','2,024.0'],
                [10,'ConocoPhillips','166,683.0','13,529.0'],
                [11,'General Electric','157,153.0','16,353.0'],
                [12,'Total','152,360.7','15,250.0'],
                [13,'ING Group','138,235.3','8,958.9'],
                [14,'Citigroup','131,045.0','24,589.0'],
                [15,'AXA','129,839.2','5,186.5'],
                [16,'Allianz','121,406.0','5,442.4'],
                [17,'Volkswagen','118,376.6','1,391.7'],
                [18,'Fortis','112,351.4','4,896.3'],
                [19,'Crédit Agricole','110,764.6','7,434.3'],
                [20,'American Intl. Group','108,905.0','10,477.0']];

        var obj = {};
        obj.width = 700;
        obj.height = 400;
        obj.colModel = [{title:"Rank", width:100, dataType:"integer"},
            {title:"Company", width:200, dataType:"string"},
            {title:"Revenues ($ millions)", width:150, dataType:"float", align:"right"},
            {title:"Profits ($ millions)", width:150, dataType:"float", align:"right"}];
        obj.dataModel = {data:data};
        $("#grid_array").pqGrid( obj );
    });
    </script>

    <div id="grid_array"></div>

### API：选项

#### title

字符串类型。默认为null。设置表格标题。

```js
$( ".selector" ).pqGrid( {title:'Shipping Details'} );
```

```js
//getter
var title=$( ".selector" ).pqGrid( "option", "title" );
//setter
$( ".selector" ).pqGrid( "option", "title", "Order Details" );
```

#### editable：整个表格可编辑

布尔类型。默认为**true**。

```js
$( ".selector" ).pqGrid( {editable:false} );
```

```js
//getter
var editable=$( ".selector" ).pqGrid( "option", "editable" );
//setter
$( ".selector" ).pqGrid( "option", "editable", false );
```

#### editModel：控制编辑行为

类型：对象。默认为`{ clicksToEdit: 1 , saveKey: '' }`。

控制单击还是双击触发编辑；哪个键（ascii码）可以结束编辑（除了默认的Tab键）。

```js
//double click is required to edit cell and enter key saves the cell.
$( ".selector" ).pqGrid( {editModel: { clicksToEdit: 2, saveKey: 13} } );
```

```js
//getter
var editModel=$( ".selector" ).pqGrid( "option", "editModel" );
//setter
$( ".selector" ).pqGrid( "option", "editModel", {clicksToEdit: 2, saveKey: 13} );
```

#### flexHeight

布尔类型。默认为false。The grid height is adjusted to the content height so that all the content is visible vertically. 垂直滚动条消失。

```js
$( ".selector" ).pqGrid( {flexHeight:true} );
```

```js
//getter
var flexHeight=$( ".selector" ).pqGrid( "option", "flexHeight" );
//setter
$( ".selector" ).pqGrid( "option", "flexHeight", true );
```

#### flexWidth

布尔类型。默认为false。The grid width is adjusted to the content width so that all the content is visible horizontally. The horizontal scrollbar is still visible.

```js
$( ".selector" ).pqGrid( {flexWidth: true} );
```

```js
//getter
var flexWidth=$( ".selector" ).pqGrid( "option", "flexWidth" );
//setter
$( ".selector" ).pqGrid( "option", "flexWidth", true );
```

#### freezeCols

整数类型。默认为0。要冻结的列的数量。

```js
$( ".selector" ).pqGrid( {freezeCols:2} );
```

```js
//getter
var freezeCols=$( ".selector" ).pqGrid( "option", "freezeCols" );
//setter
$( ".selector" ).pqGrid( "option", "freezeCols", 1 );
```

#### minWidth：列最小宽度

类型：整数。默认50（像素）。

```js
$( ".selector" ).pqGrid({ minWidth:100 });
```

```js
//getter
var minWidth=$( ".selector" ).pqGrid( "option", "minWidth" );
//setter
$( ".selector" ).pqGrid( "option", "minWidth", 20 );
```

#### width：表格宽度

取值字符串或整数。默认500。设置表格宽度

```js
$( ".selector" ).pqGrid( {width:500} );
```

```js
//getter
var width=$( ".selector" ).pqGrid( "option", "width" );
//setter
$( ".selector" ).pqGrid( "option", "width", 500 );
```

#### height：表格高度

类型：字符串或数字。默认为400。表格的高度。

```js
$( ".selector" ).pqGrid( {height:400} );
```

```js
//getter
var height=$( ".selector" ).pqGrid( "option", "height" );
//setter
$( ".selector" ).pqGrid( "option", "height", 400 );
```

#### topVisible

布尔类型。默认为true。设置是否显示顶部区域。

```js
$( ".selector" ).pqGrid( { topVisible : true } );
```

```js
//getter
var topVisible=$( ".selector" ).pqGrid( "option", "topVisible" );
//setter
$( ".selector" ).pqGrid( "option", "topVisible", false );
```

#### wrap

布尔类型。默认为true。

It determines the behaviour of cell content which doesn't fit in a single line within the width of the cell. The text in the cells wraps to next line if wrap = true otherwise the overflowing text becomes hidden and continuation symbol ... is displayed at the end.

```js
$( ".selector" ).pqGrid( {wrap:true} );
```

```js
//getter
var wrap=$( ".selector" ).pqGrid( "option", "wrap" );
//setter
$( ".selector" ).pqGrid( "option", "wrap", true );
```

#### bottomVisible：是否显示底部区域

布尔类型。默认true。分页工具条显示在底部区域。

```js
$( ".selector" ).pqGrid( { bottomVisible : true } );
var bottomVisible = $( ".selector" ).pqGrid( "option", "bottomVisible" );
$( ".selector" ).pqGrid( "option", "bottomVisible", false );
```

#### columnBorders：显示列的垂直边框

布尔类型。默认true。

```js
$( ".selector" ).pqGrid( { columnBorders: false } );
```

```js
//getter
var columnBorders=$( ".selector" ).pqGrid( "option", "columnBorders" );
//setter
$( ".selector" ).pqGrid( "option", "columnBorders", true );
```

#### rowBorders

布尔类型。默认为true。是否显示行的水平边框。

```js
$( ".selector" ).pqGrid( {rowBorders:false} );
```

```js
//getter
var rowBorders=$( ".selector" ).pqGrid( "option", "rowBorders" );
//setter
$( ".selector" ).pqGrid( "option", "rowBorders", true );
```

#### draggable：整个表格可拖动

布尔类型。默认为false。

```js
$( ".selector" ).pqGrid( {draggable: true} );
```

```js
//getter
var draggable=$( ".selector" ).pqGrid( "option", "draggable" );
//setter
$( ".selector" ).pqGrid( "option", "draggable", true );
```

#### hoverMode

类型：字符串。默认为`'row'`。It provides the hover (mouseenter and mouseleave) behaviour of grid w.r.t cells and rows.

```js
$( ".selector" ).pqGrid({ hoverMode:'cell' });
```

```js
//getter
var hoverMode=$( ".selector" ).pqGrid( "option", "hoverMode" );
//setter
$( ".selector" ).pqGrid( "option", "hoverMode", 'row' );
```

#### customData

类型：对象。默认为null。

It's used to pass custom data to render callback function.

```js
$( ".selector" ).pqGrid( { customData: {key1: val1, key2: val2} } );
```

```js
//getter
var customData=$( ".selector" ).pqGrid( "option", "customData" );
//setter
$( ".selector" ).pqGrid( "option", "customData", {key1:val1} );
```

#### numberCell

布尔类型。默认true。是否显示行号。

```js
$( ".selector" ).pqGrid( {numberCell:false} );
```

```js
//getter
var numberCell=$( ".selector" ).pqGrid( "option", "numberCell" );
//setter
$( ".selector" ).pqGrid( "option", "numberCell", true );
```

#### numberCellWidth

整数类型。默认为50。行号列的宽度。

```js
$( ".selector" ).pqGrid( {numberCellWidth:50} );
```

```js
//getter
var numberCellWidth=$( ".selector" ).pqGrid( "option", "numberCellWidth" );
//setter
$( ".selector" ).pqGrid( "option", "numberCellWidth", 50 );
```

#### oddRowsHighlight

布尔。默认为true。是否高亮奇数行。Currently this option is supported only for custom themes e.g. 'Office' theme.

```js
$( ".selector" ).pqGrid( {oddRowsHighlight : true} );
```

```js
//getter
var oddRowsHighlight=$( ".selector" ).pqGrid( "option", "oddRowsHighlight" );
//setter
$( ".selector" ).pqGrid( "option", "oddRowsHighlight", true );
```

#### sortable

布尔类型。是否支持排序。

```js
$( ".selector" ).pqGrid( {sortable:false} );
```

```js
//getter
var sortable=$( ".selector" ).pqGrid( "option", "sortable" );
//setter
$( ".selector" ).pqGrid( "option", "sortable", false );
```

#### resizable

布尔类型。默认为false。是否可以调整表格大小。

```js
$( ".selector" ).pqGrid( {resizable: true} );
```

```js
//getter
var resizable=$( ".selector" ).pqGrid( "option", "resizable" );
//setter
$( ".selector" ).pqGrid( "option", "resizable", true );
```

#### roundCorners

布尔类型。默认为true。表格四角是否显示圆角。

```js
$( ".selector" ).pqGrid( {roundCorners:true} );
```

```js
//getter
var roundCorners=$( ".selector" ).pqGrid( "option", "roundCorners" );
//setter
$( ".selector" ).pqGrid( "option", "roundCorners", true );
```

#### colModel：列模型

类型为数组。默认为null。每个元素对应一列。

```js
var colM = [
    { title: "ShipCountry", width: 100 },
    { title: "Customer Name", width: 100 }];
    $( ".selector" ).pqGrid( {colModel:colM} );
```

```js
var colM = $( ".selector" ).pqGrid( "option", "colModel" );
$( ".selector" ).pqGrid( "option", "colModel", colM );
```

#### column > title：列标题

字符串类型。默认为null。

```js
var colM = [
    { title: "ShipCountry", width: 100 },
    { title: "Customer Name", width: 100 }];
$( ".selector" ).pqGrid( { colModel: colM } );
```

```js
//getter
var colModel=$( ".selector" ).pqGrid( "option", "colModel" );
//get title of 1st column
var title=colModel[0].title;
//setter
//set title of 2nd column
colM[1].title = "Shipping Country";
$( ".selector" ).pqGrid("option","colModel",colM);
```

### column > width：列宽

整数类型。默认null。

```js
var colM = [
    { title: "ShipCountry", width: 100 },
    { title: "Customer Name", width: 100 }];
$( ".selector" ).pqGrid( { colModel: colM } );
```

```js
//getter
var colM=$( ".selector" ).pqGrid( "option", "colModel" );
//get width of 1st column
var title=colM[0].width;
//setter
//set width of 2nd column
colM[1].width = 120;
$( ".selector" ).pqGrid( "option", "colModel", colM );
```


#### column > align：列对齐方式

类型为字符串。默认值`"left"`。决定列中文本的水平对齐方式。取值"left", "right" and "center"。

```js
var colM = [
    { title: "Company", width: 100 ,dataType:"string"},
    { title: "Rank", width: 100,dataType:"integer",align:"right" }];
    $( ".selector" ).pqGrid( {colModel:colM} );
```

```js
var colM = $( ".selector" ).pqGrid( "option", "colModel" );
//get align of 1st column
var dataType = colM[0].align;
//set align of 2nd column.
colM[1].align = "center";
$( ".selector" ).pqGrid( "option", "colModel", colM );
```

#### column > className

类型为字符串。默认值为null。为列指定CSS类名。

```js
var colM = [
    { title: "Company", width: 100 , className:'pq-col' },
    { title: "Rank", width: 100 }];
    $( ".selector" ).pqGrid( {colModel:colM } );
```

```js
//getter
var colM = $( ".selector" ).pqGrid( "option", "colModel" );
//get className of 1st column
var className = colM[0].className;
//setter
//set class to 'pq-someclass' for 2nd column in colModel.
colM[1].className = 'pq-someclass';
$( ".selector" ).pqGrid( "option", "colModel", colM );
```

#### column > colModel

类型：数组。默认值：null。

在列中嵌套一个`colModel`的目的是构建列组。嵌套深度可以任意。

```js
var colM = [
    { title: "Company", width: 100 ,colModel:[{title:"Company A"} , {title:"Company B"}]},
    { title: "Rank", width: 100, dataIndx:0 }];
    $( ".selector" ).pqGrid( {colModel:colM} );
```

```js
//getter
var colM = $( ".selector" ).pqGrid( "option", "colModel" );
//get nested colModel of 1st column if any.
var colModel = colM[0].colModel;
//setter
//set nested colModel of 2nd column in colModel.
colM[1].colModel = [{title:"Title A", width:'120'} , {title:"title B"} ];
$( ".selector" ).pqGrid( "option", "colModel", colM );
```

#### column > dataIndx

类型：数字或字符串。默认：`colIndx`。

数组索引（0开始）或JSON的键名。如果数据是数组格式，默认是`colModel`中列的索引。如果数据是JSON格式，`dataIndx`必须指定。

```js
//array
var colM = [
    { title: "Company", width: 100 ,dataIndx : 1},
    { title: "Rank", width: 100,dataIndx : 0 }];
//JSON
var colM = [
    { title: "Company", width: 100 ,dataIndx: "company" },
    { title: "Rank", width: 100, dataIndx: "rank" }];

$( ".selector" ).pqGrid( { colModel: colM } );
```

```js
//getter
var colM = $( ".selector" ).pqGrid( "option", "colModel" );
//get dataIndx of 1st column
var dataIndx = colM[0].dataIndx;
//setter
//set dataIndx of 2nd column in colModel to 3rd column of data array.
colM[1].dataIndx = 2;
//set dataIndx of 2nd column in colModel to "rank" key in JSON data.
colM[1].dataIndx = "rank";
$( ".selector" ).pqGrid( "option", "colModel", colM );
```

#### column > dataType

类型：字符串或函数。默认为`"string"`。此选项用于本地排序时决定顺序。如果传入字符串可以取`"string"`、`"integer"`、`"float"`。如果传入函数，函数决定排序顺序。函数取两个参数`val1`和`val2`。返回+1表示val1在val2之前，-1表示val1在val2之后，0表示二者在排序上相同。

```js
var colM = [
    { title: "Company", width: 100 , dataType: "string" },
    { title: "Rank", width: 100, dataType: "integer" }];
    $( ".selector" ).pqGrid( { colModel: colM } );
```

```js
//getter
var colM = $( ".selector" ).pqGrid( "option", "colModel" );
//get dataType of 1st column
var dataType = colM[0].dataType;
//setter
//set dataType of 2nd column to implement custom sorting.
colM[1].dataType = function( val1, val2 ){ };
$( ".selector" ).pqGrid( "option", "colModel", colM );
```

#### column > editable

布尔类型。默认值true。设为false则禁止对列进行编辑。

```js
var colM = [
    { title: "Company", width: 100, dataType: "string" },
    { title: "Rank", width: 100, dataType: "integer", editable: false }];
    //disables editing for Rank column.
    $( ".selector" ).pqGrid( { colModel: colM } );
```

```js
//getter
var colM=$( ".selector" ).pqGrid( "option" , "colModel" );
//get editable option of 1st column
var dataType=colM[0].editable;
//setter
//set editable of 2nd column
colM[1].editable = false;
$( ".selector" ).pqGrid( "option", "colModel", colM);
```

#### column > hidden

布尔类型。默认值false。利用此选项隐藏列。

```js
var colM = [
    { title: "ShipCountry", width: 100, hidden:true },
    { title: "Customer Name", width: 100 }];
    //hidden is specified for 1st column.
    $( ".selector" ).pqGrid( {colModel:colM} );
```

```js
//getter
var colM=$( ".selector" ).pqGrid( "option", "colModel" );
//get hidden of 1st column
var hidden=colM[0].hidden;
//setter
//set hidden of 2nd column
colM[1].hidden = true;
$( ".selector" ).pqGrid( "option", "colModel", colM );
```

#### column > editor( ui )

类型：函数。默认值null。用于返回一个定制的单元格编辑器。若不指定，默认使用一个contenteditable div做编辑器。

函数参数`ui`对象包含以下属性：

- `cell`：一个jQuery对象。jQuery wrapper on container div in which to append the custom editor.
- `data`：数组类型。二维数组或JSON数组。
- `rowIndx`：整数。行索引（0开始）。
- `colIndx`：整数。列索引（0开始）。

```js
var colM = [
    { title: "ShipCountry", width: 100, editor:function( ui ){} },
    { title: "Customer Name", width: 100 }];
    //editor is provided for 1st column.
    $( ".selector" ).pqGrid( { colModel: colM } );
```

```js
//getter
var colM=$( ".selector" ).pqGrid( "option", "colModel" );
//get editor of 1st column
var editor=colM[0].editor;
//setter
//set editor of 2nd column
colM[1].editor = function( ui ){};
$( ".selector" ).pqGrid( "option", "colModel", colM );
```

#### column > render( ui )

类型：函数。默认值：null。一个函数，返回单元格的视图。

函数参数`ui`对象包含以下对象：

- `data`：数组。二维数组或JSON数据。
- `rowData`：表示此行的数据。一维数组或对象。
- `rowIndx`：整数。行号（0开始）。
- `dataIndx`：整数或字符串。数组数据的索引或JSON数据的键。
- `colIndx`：整数。列索引（0开始）。

```js
var colM = [
    { title: "ShipCountry", width: 100, render:function( ui ){} },
    { title: "Customer Name", width: 100 }];
    //render is provided for 1st column.
    $( ".selector" ).pqGrid( { colModel: colM } );
```

```js
//getter
var colModel=$( ".selector" ).pqGrid( "option", "colModel" );
//get render of 1st column
var render=colModel[0].render;
//setter
//set render of 2nd column
colM[1].render = function( ui ){};
$( ".selector" ).pqGrid( "option", "colModel", colM );
```

#### column > getEditCellData( ui )

类型：函数。默认值null。一个回调函数，用于保存自定义编辑器产生的数据。返回要被保存的数据。

参数`ui`对象有一下属性：

- `$cell`：jQuery对象。jQuery wrapper on container div in which custom editor has been appended.
- `data`：数组。二维数组或JSON数据。
- `rowIndx`：整数。行索引（0开始）。
- `rowIndxPage`：整数。行在当前页的索引（0开始）。
- `colIndx`：整数。列索引（0开始）。
- `dataIndx`：整数或字符串。数组数据的索引或JSON数据的键。

```js
var colM = [
    { title: "ShipCountry", editor:function( ui ){}, getEditCellData:function(ui){} },
    { title: "Customer Name" }];
//editor and saveCell callback is provided for 1st column.
$( ".selector" ).pqGrid( { colModel: colM } );
```

```js
//getter
var colM=$( ".selector" ).pqGrid( "option", "colModel" );
//get saveCell callback of 1st column
var getEditCellData=colM[0].getEditCellData;
//setter
//set saveCell of 2nd column
colM[1].getEditCellData = function( ui ){};
$( ".selector" ).pqGrid( "option", "colModel", colM );
```
#### dataModel

类型：对象。默认为null。

数据信息。`dataModel`是必须设置的。

```js
var dataSales = [[1, 'Exxon Mobil', '339,938.0', '36,130.0'],
    [2, 'Wal-Mart Stores', '315,654.0', '11,231.0'],
    [3, 'Royal Dutch Shell', '306,731.0', '25,311.0'],
    [4, 'BP', '267,600.0', '22,341.0'],
    [5, 'General Motors', '192,604.0', '-10,567.0'],
    [6, 'Chevron', '189,481.0', '14,099.0'],
    [7, 'DaimlerChrysler', '186,106.3', '3,536.3'],
    [8, 'Toyota Motor', '185,805.0', '12,119.6'],
    [9, 'Ford Motor', '177,210.0', '2,024.0'],
    [10, 'ConocoPhillips', '166,683.0', '13,529.0']];
$( ".selector" ).pqGrid( { dataModel: { data: dataSales } } );
```

```js
//getter
var dataModel=$( ".selector" ).pqGrid( "option", "dataModel" );
//setter
$( ".selector" ).pqGrid( "option", "dataModel", { data: dataSales } );
```

#### dataModel > data

数组类型。元素可以数组或对象。Local requests use this option to directly pass data to pqGrid. Remote requests use dataModel > getData() callback to feed data to pqGrid. The data for local as well as remote requests reside here.

```js
//array of arrays
var dataSales = [[ 1, 'Exxon Mobil', '339,938.0', '36,130.0' ],
    [ 2, 'Wal-Mart Stores', '315,654.0', '11,231.0' ],
    [ 3, 'Royal Dutch Shell', '306,731.0', '25,311.0' ],
    [ 4, 'BP', '267,600.0', '22,341.0' ],
    [ 5, 'General Motors', '192,604.0', '-10,567.0' ] ];
//or array of objects
var dataSales = [
    { rank:1, company: 'Exxon Mobil', revenues: '339,938.0', profits: '36,130.0' },
    { rank:2, company: 'Wal-Mart Stores', revenues: '315,654.0', profits: '11,231.0' },
    { rank:3,  company: 'Royal Dutch Shell', revenues: '306,731.0', profits: '25,311.0' },
    { rank:4,  company: 'BP', revenues: '267,600.0', profits: '22,341.0' },
    { rank:5, company: 'General Motors', revenues: '192,604.0', profits: '-10,567.0' } ];
// pass it to pqGrid.
$( ".selector" ).pqGrid( { dataModel: { data: dataSales } } );
```

```js
//getter
var data=$( ".selector" ).pqGrid( "option" , "dataModel.data" );
//setter
$( ".selector" ).pqGrid( "option", "dataModel.data", dataSales );
```

#### dataModel > beforeSend( jqXHR, settings )

类型：函数。默认为null。

It's a callback function which can be used to modify jqXHR before it's sent. Use this to set custom headers, etc. The jqXHR and settings maps are passed as arguments. This is an Ajax Event. Returning false in the beforeSend function will cancel the request. The keyword this within the function points to dataModel of pqGrid instance.

```js
$( ".selector" ).pqGrid({dataType:"JSON",dataModel:{beforeSend:function( jqXHR, settings ){}});
```

```js
//getter
var beforeSend=$( ".selector" ).pqGrid( "option" , "dataModel.beforeSend" );
//setter
$( ".selector" ).pqGrid( "option", "dataModel.beforeSend", function( jqXHR, settings ){});
```

#### dataModel > curPage

类型：整数。默认值为1。Current page of the view when paging has been enabled.

```js
$( ".selector" ).pqGrid( {dataModel: {curPage: 3 } } );
```

```js
//getter
var curPage=$( ".selector" ).pqGrid( "option", "dataModel.curPage" );
//setter
$( ".selector" ).pqGrid( "option", "dataModel.curPage", 2 );
```

#### dataModel > dataType

类型：字符串。默认为`"TEXT"`。服务器相应的数据类型，取值`"TEXT"`、`"XML"`或`"JSON"`。

```js
$( ".selector" ).pqGrid( {dataModel:{ dataType: "XML"} } );
```

```js
//getter
var dataType=$( ".selector" ).pqGrid( "option", "dataModel.dataType" );
//setter
$( ".selector" ).pqGrid( "option", "dataModel.dataType", "JSON" );
```

#### dataModel > error( jqXHR, textStatus, errorThrown )

函数类型。默认为null。处理Ajax错误。

```js
$( ".selector" ).pqGrid( {dataModel:{error:function( jqXHR, textStatus, errorThrown ){} }} );
```

```js
//getter
var errorHandler=$( ".selector" ).pqGrid( "option", "dataModel.error" );
//setter
$( ".selector" ).pqGrid( "option", "dataModel.error", function( jqXHR, textStatus, errorThrown ){} );
```

#### dataModel > getData( response, textStatus, jqXHR )

函数类型。默认值为null。

函数需要返回一个对象，对象包含`data`、`curPage`和`totalRecords`三个字段，其中`data`应是一个二维数组和对象的数组，。`curPage`表示当前页，`totalRecords`表示所有页面的所有数据条数。curPage and totalRecords are optional and are required only when using remote paging.	方法的`this`指向`dataModel`。

```js
$( ".selector" ).pqGrid({dataType:"JSON",dataModel:{getData:function( dataJSON, textStatus, jqXHR ){
    return { curPage: dataJSON.curPage, totalRecords: dataJSON.totalRecords, data: dataJSON.data };
}});
```

```js
//getter
var getData=$( ".selector" ).pqGrid( "option" , "dataModel.getData" );
//setter
$( ".selector" ).pqGrid("option","dataModel.getData",function( response, textStatus, jqXHR ){});
```

#### dataModel > getUrl()

函数类型。默认为null。

It's a callback function which returns the url and data associated with GET or POST request. The keyword this within the function points to dataModel of pqGrid instance.

```js
$( ".selector" ).pqGrid( { dataModel:{ getUrl: function(){
    return { url: "/demos/pagingGetOrders", data: "key1=val1&key2=val2&key3=val3" };
}});
```

```js
//getter
var getUrl=$( ".selector" ).pqGrid( "option" , "dataModel.getUrl" );
//setter
$( ".selector" ).pqGrid( "option" , "dataModel.getUrl", function(){});
```

#### dataModel > location：数据来源

字符串类型，默认取`"local"`。数据的位置，取值`"remote"`或`"local"`。

```js
$( ".selector" ).pqGrid( { dataModel:{ location:"remote" } } );
```

```js
//getter
var location=$( ".selector" ).pqGrid( "option", "dataModel.location" );
//setter
$( ".selector" ).pqGrid( "option", "dataModel.location", "remote" );
```

#### dataModel > method：请求HTTP方法

字符串类型。默认为`"GET"`。还可以取`"POST"`。

```js
$( ".selector" ).pqGrid( {dataModel: {method: "POST" } } );
```

```js
//getter
var method=$( ".selector" ).pqGrid( "option", "dataModel.method" );
//setter
$( ".selector" ).pqGrid( "option", "dataModel.method", "GET" );
```

#### dataModel > paging：分页方式

字符串类型。默认为null。Paging can be enabled for both local and remote requests. It has 2 possible values `"local"` and `"remote"`. Paging is disabled when it's null.

```js
$( ".selector" ).pqGrid( {dataModel: {paging: 'local'}} );
```

```js
//getter
var paging=$( ".selector" ).pqGrid( "option", "dataModel.paging" );
//setter
$( ".selector" ).pqGrid( "option", "dataModel.paging", 'remote' );
```

#### dataModel > rPP

整数类型。默认10。It denotes initial results per page when paging has been enabled.

```js
$( ".selector" ).pqGrid( {dataModel:{rPP:100}} );
```

```js
//getter
var rPP=$( ".selector" ).pqGrid( "option", "dataModel.rPP" );
//setter
$( ".selector" ).pqGrid( "option", "dataModel.rPP", 2 );
```

#### dataModel > rPPOptions

数组类型。默认为`[10, 20, 50, 100]`。Results per page options in dropdown when paging has been enabled.

```js
$( ".selector" ).pqGrid({dataModel:{ rPPOptions:[10, 100, 200] }});
```

```js
//getter
var rPPOptions=$( ".selector" ).pqGrid( "option", "dataModel.rPPOptions" );
//setter
$( ".selector" ).pqGrid( "option", "dataModel.rPPOptions", [10, 20, 100] );
```

#### dataModel > sortDir：排序方向

字符串类型。默认为`"up"`。或取`"down"`。It has significance along with sortIndx.

```js
$( ".selector" ).pqGrid({dataModel:{sortDir:"down"}});
```

```js
//getter
var sortDir=$( ".selector" ).pqGrid("option","dataModel.sortDir");
//setter
$( ".selector" ).pqGrid("option","dataModel.sortDir","up");
```

#### dataModel > sortIndx：对哪列排序

整数或字符串。默认为null。The dataIndx of the sorted column.

```js
//array
$( ".selector" ).pqGrid( {dataModel: {sortIndx: 2} } );
//JSON
$( ".selector" ).pqGrid( {dataModel: {sortIndx: "company"} } );
```

```js
//getter
var sortIndx = $( ".selector" ).pqGrid( "option" , "dataModel.sortIndx" );
//setter in case of array
$( ".selector" ).pqGrid("option", "dataModel.sortIndx", 0);
//setter in case of JSON
$( ".selector" ).pqGrid("option", "dataModel.sortIndx", "rank");
```

#### dataModel > sorting：客户端或服务端排序

字符串类型。默认取`"local"`。还可以取`"remote"`。

```js
$( ".selector" ).pqGrid({dataModel:{sorting:"remote"}});
```

```js
//getter
var sorting=$( ".selector" ).pqGrid("option","dataModel.sorting");
//setter
$( ".selector" ).pqGrid("option","dataModel.sorting","local");
```

#### scrollModel

对象类型。默认为`{ horizontal: true, pace: 'fast' }`。
It describes the visibility of horizontal scrollbar and pace of scrollbars. pace governs behavior of the scrollbar sliders. Possible values are 'consistent', 'optimum' and 'fast'. The viewport gets refreshed after slider is dragged & released when it's 'consistent'. It's useful when viewport rendering is very slow and you need to support old IE browsers. The viewport gets refreshed after slider is dragged & paused when it's 'optimum'. It strikes a good balance between new and old browsers. The viewport gets refreshed immediately as the slider is dragged when it's value is 'fast'. It's useful when you want to draw maximum performance from faster browsers.

```js
$( ".selector" ).pqGrid({ scrollModel:{pace: 'fast', horizontal: false} });
```

```js
//getter
var scrollModel=$( ".selector" ).pqGrid( "option", "scrollModel" );
//setter
//hide the horizontal scrollbar.
$( ".selector" ).pqGrid( "option", "scrollModel", {horizontal: false} );
```

#### selectionModel

对象类型。默认为`{ type: 'row' , mode: 'range' }`。
It provides the selection behaviour of the grid w.r.t type (row or cell) and mode (`single`, `range` or `block`).

```js
$( ".selector" ).pqGrid( {selectionModel: { type: 'cell', mode: 'block'} } );
```

```js
//getter
var selectionModel=$( ".selector" ).pqGrid( "option", "selectionModel" );
//setter
$( ".selector" ).pqGrid( "option", "selectionModel", {type: 'row', mode: 'single'} );
```