[toc]

## 16. 表格列布局

Browsers have many built-in capabilities for automatically sizing columns in tables. This chapter shows how to harness these automatic features to shrinkwrap columns, size them to specific widths, size them proportionally to each other, size them proportionally to their content, size them equally, size them flexibly, and undersize or oversize them.

### 16.1 表格布局模型

表格有四种：收紧、指定大小、撑开和固定（fixed）。每种类型都有独特的列布局。收紧的表格会把列收紧到适合内容。指定大小或撑开的表格多数是为了能将表格宽度按比例分配到各个列。固定（fixed）表格的目的是讲列设为固定大小，并加速表格的渲染。

收紧的表格会把列收紧到适合内容。收紧的表格不会超出容器宽度。若想灵活的布局，可以适配不同的设备、屏幕分辨率和视口大小，最适合使用收紧的表格。The following unique layouts apply to shrinkwrapped tables: Shrinkwrapped Columns, Sized Columns, Equal Content-Sized Columns, and Inverse-Proportioned Columns.

指定大小或撑开的表格多数是为了能将表格宽度按比例分配到各个列。同时，列不会比内容宽度更窄。指定大小和撑开的表格在布局列上是一致的。只是指定大小的表格可能比容器窄也能更宽！但撑满的表格撑满到容器宽度。The following layouts apply to stretched tables: Content-Proportioned Columns, Size-Proportioned Columns, Percentage-Proportioned Columns, Equal-Sized Columns, and Flex Columns.

固定（Fixed）表格是指定大小或撑满表格的变体，它可以是指定大小或撑满的，但不能是收紧的。与指定大小或撑满的表格的区别是，布局列是不考虑内容宽度。由于忽略内容宽度，它们的渲染速度比其他表格快很多。Fixed tables have unique support for Sized Columns and Undersized Columns. Fixed tables support all the layouts of sized and stretched tables except for Content-Proportioned Columns. These layouts include Size-Proportioned Columns, Percentage-Proportioned Columns, Equal-Sized Columns, and Flex Columns.

布局算法的选择取决于表格类型和单元格宽度类型（如auto, 100px, or 20%）。

浏览器需要检查某一列的所有行的单元格决定列宽和列宽的类型。How a browser reconciles different cell widths in the same column is explained in the Column Width design pattern. 若表格的列采用不同的宽度类型，则一个表格内会使用不同的布局算法。How a browser combines column layouts is explained in the Mixed Column Layouts design pattern.

Even though a browser examines the width of all cells in nonfixed tables to determine the column width, you only need to assign a width to the cells in the first row.

### 16.2 使用列布局

For many years, designers and developers have used the many automatic and powerful layout features of columns to lay out nontabular content. In fact, this extensive use has promoted browser vendors to enhance these capabilities more than any other feature. It has also caused the major browser vendors to ensure column layouts work consistently and are bug-free.

### 16.4 列宽

本节介绍浏览器如何决定列宽的算法。

最简单的情况是只为第一行指定了宽度。但如果为某列的多行指定了宽度，且不同，需要浏览器确定列宽。

该设计模式不能用于固定（fixed）表格，因为列的第一行单元格的宽度决定列宽。若列后面行的宽度更大，会被截断。下面的讨论只适用于非固定的表格。

每个单元格有一个最小内容宽度，为显示单元格内容的宽度。对于非固定（nonfixed）的表格，浏览器不会将列缩小到小于宽度。For text, the minimum content width is the width of the widest word in the cell. For a replaced element, such as an image, it is the width of the replaced element.

浏览器给单元格分配一个最大内容宽度：从单元格内容的宽度到表格容器的宽度。一些设计模式使用该宽度设定列大小或设定列的百分比。

浏览器下载整个表格，扫描每一行，决定每一列的以下特征：宽度类型，最大宽度值、最小内容宽度、最大内容宽度。

对于不同的类型和值，使用以下规则调节：

1. 列默认是自动宽度的。
2. 若出现固定宽度，则列类型改为固定宽度。
3. 若出现了百分比宽度，则列类型变为百分比宽度。
4. 固定宽度大的值替代小的值。
5. 百分比宽度，大的取代小的。
6. 更大的最小内容宽度取代较小的。
7. 更大的最大内容宽度取代较小的。

A browser chooses a layout design pattern based on the type of table and the type of each column (auto, fixed, or percentage width). The column is sized using the largest width value in the column that matches its type.

### 16.5 收紧的列

You want to shrinkwrap columns to fit the width of their content.

You can shrinkwrap columns by applying `table-layout:auto` and `width:auto` to the table and `width:auto` to its cells. Since these rules are the default, this happens by default.

The width of each cell expands to its maximum content width, which is the width of a cell’s content up to the width of the table’s container. The content can expand a table up to the width of the table’s container. If this happens, the cells are laid out using the Content-Proportioned Columns design pattern.

Browsers use this design pattern by default because it is the most adaptable and natural. It automatically sizes columns and tables to fit their content. It adapts automatically to any device and display size.

The only time shrinkwrapped columns can expand a table beyond the width of its container is when the combined minimum content width of each column is greater than the width of the container. For example, replaced elements, such as images, tables nested in cells, or text set to `white-space:nowrap` can easily expand a shrinkwrapped table beyond the width of its container. This causes the table to overflow its container.

### 16.6 指定大小的列

You want to assign fixed widths to columns while keeping the table’s width within a minimum or maximum value.

第一个指定列宽的方式：表格设置`table-layout:auto`和`width:auto`。单元格设置`width:VALUE`。若列的总宽度比容器更宽，布局改为Sized-Proportioned Columns设计模式。I call this the Maximum-Width Sized Columns design pattern because columns are rendered at the width you assigned only as long as their total width is less than or equal to the width of the table’s container. 换句话说，容器的宽度限定了表格的最大宽度。最后，不管指定的宽度如何，列不能小于其最小内容宽度。

第二种指定列宽的方式：表格设置`table-layout:fixed`和`width:MIN_WIDTH`。单元格设置`width:VALUE`。即便你设置表格为1像素，浏览器仍会扩展表格以容纳固定宽度的单元格。表格没有最大宽度，表格可能会超过容器大小。如果设置的表格宽度大于列的总宽度，布局变成Sized-Proportioned Columns设计模式。I call this the Minimum-Width Sized Columns design pattern because columns are rendered at the width you assigned only as long as their total width is greater than or equal to the width assigned to the table. 最后，最小内容宽度对列宽无效。

In select versions of webkit browsers (Chrome, Safari), there is a documented bug associated with `table-layout:fixed` where the browser will not render padding assigned to the width of a table cell.

### 16.7 Content-Proportioned Columns

想要列填满表格宽度。列的大小与其内容成比例。

This pattern applies to sized and stretched tables. It also applies to shrinkwrapped tables when their content stretches them to the width of their containers. It does not apply to fixed tables.

设置表格`table-layout:auto`和`width:VALUE_OR_PERCENT`。设置单元格`width:auto`。即，令表格大小固定或撑满，令单元格宽度自动。浏览器自动计算每个列的最大内容宽度，并以此为比例分配表格宽度。A shrinkwrapped table cannot expand beyond the width of its container. When content expands a shrinkwrapped table to the full width of its container, the table behaves as if it were stretched and turns shrinkwrapped columns into content-proportioned columns.

### 16.8 Size-Proportioned Columns

想要列填满表格宽度。列的大小与其原本指定的大小成比例。

设置表格`table-layout:auto`和`width:VALUE_OR_PERCENT`。设置单元格`width:VALUE`。即，令表格大小固定或撑满，向单元格分配一个固定宽度。

若列的宽度、padding、边框及单元格空隙加起来正好等于表格宽度，浏览器会按指定的大小渲染列。但如果列宽总和稍微大于小于表格宽度，浏览器会依指定的列大小为比例渲染列。

This pattern applies to sized and stretched tables. This pattern applies to a shrinkwrapped table when the total width of all its columns is greater than the width of its container. This stretches it to the sides of its container, causing it to behave like a stretched table.

This pattern applies to a fixed table when the total width of all its columns is less than the width assigned to the table. In contrast, if the total width of the columns is greater than the width of a fixed table, the width of the table expands, and the columns are not size-proportioned.

Size-proportioned columns give you the ability to specify the relative size of each column in relation to the other columns while preserving the width you assigned to the table.

Since the widths you assign to columns are proportional, you can make widths huge or tiny because only the ratio between widths matters.

### 16.9 Percentage-Proportioned Columns

You want to size columns as a percentage of a table’s width. In other words, you want columns to fill the specified width of a table, and you want to distribute a table’s width among its columns using percentages. When the total column percentage falls short of 100%, you want a browser to scale the percentages to equal 100%.

You can size columns as a percentage of a table’s width by applying
`width:VALUE_OR_PERCENT` to the table and `width:PERCENT` to its cells. In other words, you size or stretch the table and assign percentages to cells. 表格可以处于固定（fixed）布局或自动布局。

当所有列的百分比和小于100%时，浏览器将百分比扩到100%。例如，假如有两列都设为20%，总和40%。则为了总和未100%，两列会分别扩到50%。

对于百分比的列从左到右计算它们的累计，当遇到一列加上它的宽度超过100%时，浏览器会把该列的百分比降低使得加上该列百分比和正好等于100%。后面的列宽度会当做`width:auto`。例如，若表格前两列都设为80%，则第二列会被降到20%。

对于固定（fixed）的表格，当百分比小于等于100%时，百分比就相当于大小固定或收紧的表格。若他们超过100%，结果取决于浏览器。

Size-proportioned columns are more forgiving because they do not have to add up to 100%.

最后不要让列百分比加起来超过100%。如果想让一些列收紧，另外一些百分比，只要对需要收紧的列设置`width:auto`，对百分比的列设置`width:PERCENT`。

### 16.10 Inverse-Proportioned Columns

You want to size a table in proportion to its column with the widest content, and you want its columns to be percentage-proportioned within this width. For example, you want a table to be automatically sized at twice the width of the column containing the widest content.

You can size a table in proportion to the column with the widest content by assigning `table-layout:auto` and `width:auto `to the table and `width:PERCENT` to its cells. In other words, you shrinkwrap the table and assign percentages to cells.
A browser calculates the table width by multiplying the maximum content width by the inverse of the percentage assigned to each column. The largest resulting width becomes the width of the table. Once the table width is calculated, a browser percentage-proportions each column to fit into the table’s width.

This design pattern provided by browsers is too unintuitive to be useful as it stands. But it can be used to create equal-sized columns based on the content width, which is the basis of the next design pattern, Equal Content-Sized Columns. And this is a very useful design pattern.

This pattern applies to shrinkwrapped tables.

This pattern works only when the total of all columns is less than or equal to 100%.

In the example, the first table has one column assigned to width:20%. A browser multiplies the content’s width, which is 40 pixels, by the inverse of 20%, which is 5. This sizes the table at 200 pixels plus cell spacing, padding, and borders around each cell. The second table shows that the table width is wide enough to hold five equal-sized columns shrinkwrapped to their content. The third table shows that columns with different percentages are percentage-proportioned within the calculated width of the table.

### 16.11 Equal Content-Sized Columns



