[toc]

## 8. 应用设计

### 8.1 Normalization versus Denormalization

规范化（Normalization）指将数据划分到多个集合中，文档之间有引用关系。但由于MongoDB没有join功能，从多个集合中收集文档需要多次查询。

反规范化（Denormalization）与规范化相反：将所有数据嵌入到单个文档。可能有数据冗余，因此更新可能需要更新多个文档。但相关数据可以一个查询获取。

一般来说，规范化的文档写更快，反规范化的文档读更快。

### 8.2 优化数据操纵


