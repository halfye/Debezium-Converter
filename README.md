## 新修复项目
1.修复mysql、sqerver的时间转换时区问题。（读取默认时区）
2.修复mysql的timestamp类型，出现1970-01-01T00:00:00Zd的String类型问题。

## 使用方法
`converters`参数为：自定义转换器的名字，可以随意设置。设置的值就作为转换器的名字，在以后的参数中就要使用这个名字。

假设自定义的名字为`mydebeziumconverter`，则`type`参数为`mydebeziumconverter.type`。

`mydebeziumconverter.type`参数为：自定义转换器的类名，必须设置。（转换器的方法）  

`mydebeziumconverter.database.type`参数为：数据库类型，必须设置。（需要设置为mysql或sqlserver) 

`mydebeziumconverter.format.datetime`参数为：datetime类型的格式，可选。

`mydebeziumconverter.format.date`参数为：date类型的格式，可选。

`mydebeziumconverter.format.time`参数为：time类型的格式，可选。

> 如果仅使用mysql或sqlserver建议独立编译代码，只保留mysql或sqlserver的转换器，减少依赖。

### flinkcdc
可使用源代码也可使用编译好的jar包。只需要放入目录即可。并在配置中设置参数。
```
// 自定义解释器
// 设置解释器及解释器参数
debeziumProperties.put("converters", "mydebeziumconverter");
debeziumProperties.put("mydebeziumconverter.type", "org.util.MyDebeziumConverter");
debeziumProperties.put("mydebeziumconverter.database.type", "mysql");
// 自定义格式，可选
debeziumProperties.put("mydebeziumconverter.format.datetime", "yyyy-MM-dd HH:mm:ss");
debeziumProperties.put("mydebeziumconverter.format.date", "yyyy-MM-dd");
debeziumProperties.put("mydebeziumconverter.format.time", "HH:mm:ss");
```
### debezium
使用jar包，并将其放在 debezium 插件的同一级别目录中。并在配置文件中设置参数。
```
"converters": "mydebeziumconverter",
"mydebeziumconverter.type": "org.util.MyDebeziumConverter",
"mydebeziumconverter.database.type": "mysql",
"mydebeziumconverter.format.datetime": "yyyy-MM-dd HH:mm:ss",
"mydebeziumconverter.format.date": "yyyy-MM-dd",
"mydebeziumconverter.format.time": "HH:mm:ss"
```

## 转换器
### mysql转换
mysql启动时，快照期间初始化转换器，在binlog期间仍进行一次初始化转换器。（使用的类不同）

|   字段类型    |                   快照类型(jdbc type)                   |      binlog类型(jdbc type)      |   
|:---------:|:---------------------------------------------------:|:-----------------------------:|
|   DATE    |               java.time.LocalDate(93)               |    java.time.LocalDate(91)    |  
|   TIME    |               java.time.Duration(92)                |    java.time.Duration(92)     |  
| DATETIME  |               java.sql.Timestamp(93)                |  java.time.LocalDateTime(93)  |
| TIMESTAMP | java.sql.Timestamp(2014)<br/>java.lang.String(2014) | java.time.ZonedDateTime(2014) |

### sqlserver转换
sqlserver启动时 快照期间初始化转换器，在cdc期间不再进行初始化转换器。（使用的类相同）
> timestamp类型在sqlserver中为byte[]类型，jdbc type为-2，因此不进行转换。

|      字段类型      |          快照类型(jdbc type)           |          cdc类型(jdbc type)          |
|----------------|:----------------------------------:|:----------------------------------:|
|      DATE      |         java.sql.Date(91)          |         java.sql.Date(91)          |
|      TIME      |       java.sql.Timestamp(92)       |         java.sql.Time(92)          |
|    DATETIME    |       java.sql.Timestamp(93)       |       java.sql.Timestamp(93)       |
|   DATETIME2    |       java.sql.Timestamp(93)       |       java.sql.Timestamp(93)       |
| DATETIMEOFFSET | microsoft.sql.DateTimeOffset(-155) | microsoft.sql.DateTimeOffset(-155) |
| SMALLDATETIME  |       java.sql.Timestamp(93)       |       java.sql.Timestamp(93)       |

## 参考代码及文档
### github
主要参考大佬的代码：[debezium-datetime-converter](https://github.com/holmofy/debezium-datetime-converter)
### debezium
官方文档：[Debezium Engine](https://debezium.io/documentation/reference/2.1/development/engine.html)
### 源码
查看sqlserver、mysql的jdbc源码，了解jdbc type和java type的对应关系。

## 版本说明
1.使用的版本说明，如果不兼容，可以修改源码或提交issue。
```
flink-connector-debezium:2.2.0
flink-connector-mysql-cdc:2.2.0
flink-connector-sqlserver-cdc-2.2.1 (2.2.0不支持sqlserver的低版本，2.2.1未发布前，项目已使用，故自己编译)
// 底层debezium版本为：1.5.4.Final
```
2.如果使用存在bug，积极联系哟。






