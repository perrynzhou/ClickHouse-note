### ClickHouse的存储引擎和数据类型

| 作者 | 时间 |QQ技术交流群 |
| ------ | ------ |------ |
| perrynzhou@gmail.com |2021/01/03 |672152841 |

##### Clickhouse支持哪一些数据结类型？

- ClickHouse支持整形包括 Int(Int8~Int64)、UInt(UInt8~UInt64)、Float(Float32/Float64)、Decimal(Decimal32、Decimal64、Decimal128)

- ClickHouse支持的字符创类型有三种，第一种是Dtring，不限长度；第二种是fixedstring,有长度限制，用于表示固定长度的字符串；第三种是uuid,uuid为32位

- ClickHouse支持事件类型有三种，第一种是DateTime类型包括时分秒信息；第二种是DateTime64，可以精确到亚秒；第三种是Date,不包含具体时间，值精确到天

- ClickHouse支持复合类型有Array(数组)、元组、枚举和嵌套四种。

  - 数组，一般使用array(t0,t1..tn)或者select [1,2,3]类似于这样的方式表达，数组中只能包含同类型的，比如整形和字符串不能再同一个数组中

    ```
    root :) SELECT [1 , 2 , null ] as a , toTypeName(a);
    ┌─a──────────┬─toTypeName([1, 2, NULL])─┐
    │ [1,2,NULL] │ Array(Nullable(UInt8))   │
    └────────────┴──────────────────────────┘
    root :) SELECT array(1 , 2) as a , toTypeName(a);
    
    ┌─a─────┬─toTypeName(array(1, 2))─┐
    │ [1,2] │ Array(UInt8)            │
    └───────┴─────────────────────────┘
    ```
    
    
    
  - 元组,元组类型是有N个元素组成，每个元素允许设置不同的数据类型，且彼此之间要求兼容

    ```
    root :) SELECT tuple(1, ' a ' , now()) AS x, toTypeName(x);
    
    ┌─x───────────────────────────────┬─toTypeName(tuple(1, ' a ', now()))─┐
    │ (1,' a ','2021-03-29 08:06:39') │ Tuple(UInt8, String, DateTime)     │
    └─────────────────────────────────┴────────────────────────────────────┘
    
    1 rows in set. Elapsed: 0.009 sec. 
    ```
    
  - 枚举，ClickHouse提供了enum8和enum16两种枚举类型，区别在于取值范围不同。枚举固定使用(string:int)这样key/value方式来定义数据。

    ```
    root :) create table enum_test( c1 Enum8('ready'=1,'start'=2,'success'=3,'error'=4) ) engine=Memory;
    root :) insert into enum_test values('start');
    
    root :) insert into enum_test values('ready');
    
    root :) select * from enum_test;
    
    SELECT *
    FROM enum_test
    
    ┌─c1────┐
    │ start │
    └───────┘
    ┌─c1────┐
    │ ready │
    └───────┘
    
    ```

	- 嵌套类型，表示一种嵌套表结构,一个数据可以定义任意多的嵌套类型的字段，每个字段的嵌套层级只支持一级。
	
	  ```
	  root :) create table nest_test(name String,aget UInt8,class Nested ( id UInt8, name String)) engine=Memory;
	  root :) insert into nest_test values('a1',18,[1,2],['c1','c2']);
	  
	  root :) insert into nest_test values('a2',08,[3,2],['c3','c2']);
	  
	  root :) select * from nest_test;
	  ┌─name─┬─aget─┬─class.id─┬─class.name──┐
	  │ a2   │    8 │ [3,2]    │ ['c3','c2'] │
	  └──────┴──────┴──────────┴─────────────┘
	  ┌─name─┬─aget─┬─class.id─┬─class.name──┐
	  │ a1   │   18 │ [1,2]    │ ['c1','c2'] │
	  └──────┴──────┴──────────┴─────────────┘
	  
	  ```
	
##### ClickHouse提供几种存储引擎?
- 一共提供MergeTree、日志、集成引擎、用于其他特定功能的引擎、虚拟列这几种类型的存储引擎。
- MergeTree,适用于高负载任务的最通用和功能最强大的表引擎。这些引擎的共同特点是可以快速插入数据并进行后续的后台数据处理。 MergeTree系列引擎支持数据复制（使用Replicated* 的引擎版本），分区和一些其他引擎不支持的其他功能。该类型的引擎：
	- MergeTree
	- ReplacingMergeTree
	- SummingMergeTree
	- AggregatingMergeTree
	- CollapsingMergeTree
	- VersionedCollapsingMergeTree
	- GraphiteMergeTree

- 日志引擎，具有最小功能的轻量级引擎。当您需要快速写入许多小表（最多约100万行）并在以后整体读取它们时，该类型的引擎是最有效的。该类型的引擎：
	- TinyLog
	- StripeLog
	- Log
- 集成引擎,用于与其他的数据存储与处理系统集成的引擎。该类型的引擎有:
	- Kafka
	- MySQL
	- ODBC
	- JDBC
	- HDFS
- 用于其他特定功能的引擎,主要包括如下引擎:
	- Distributed
	- MaterializedView
	- Dictionary
	- Merge
	- File
	- Null
	- Set
	- Join
	- URL
	- View
	- Memory
	- Buffer
- 虚拟列引擎,虚拟列是表引擎组成的一部分，它在对应的表引擎的源代码中定义。不能在 CREATE TABLE 中指定虚拟列，并且虚拟列不会包含在 SHOW CREATE TABLE 和 DESCRIBE TABLE 的查询结果中。虚拟列是只读的，所以您不能向虚拟列中写入数据。
如果想要查询虚拟列中的数据，您必须在SELECT查询中包含虚拟列的名字。SELECT * 不会返回虚拟列的内容。若创建的表中有一列与虚拟列的名字相同，那么虚拟列将不能再被访问。我们不建议您这样做。为了避免这种列名的冲突，虚拟列的名字一般都以下划线开头。