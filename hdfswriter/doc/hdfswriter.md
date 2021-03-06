# DataX HdfsWriter 插件文档


------------

## 1 快速介绍

HdfsWriter提供向HDFS文件系统指定路径中写入TEXTFile文件和ORCFile文件,文件内容可与hive中表关联。


有关Hive的资料，可以参看http://baike.corp.taobao.com/index.php/Hive%E5%9F%BA%E7%A1%80%E6%96%87%E6%A1%A3

## 2 功能与限制

* (1)、目前HdfsWriter仅支持textfile和orcfile两种格式的文件，且文件内容存放的必须是一张逻辑意义上的二维表;
* (2)、由于HDFS是文件系统，不存在schema的概念，因此不支持对部分列写入;
* (3)、目前仅支持与以下Hive数据类型：
数值型：TINYINT,SMALLINT,INT,BIGINT,FLOAT,DOUBLE
字符串类型：STRING,VARCHAR,CHAR
布尔类型：BOOLEAN
时间类型：DATE,TIMESTAMP
**目前不支持：decimal、binary、arrays、maps、structs、union类型**;
* (4)、对于Hive分区表目前仅支持一次写入单个分区;
* (5)、对于textfile需用户保证写入hdfs文件的分隔符**与在Hive上创建表时的分隔符一致**,从而实现写入hdfs数据与Hive表字段关联;
* (6)、HdfsWriter实现过程是：首先根据用户指定的path，创建一个hdfs文件系统上不存在的临时目录，创建规则：path_随机；然后将读取的文件写入这个临时目录；全部写入后再将这个临时目录下的文件移动到用户指定目录（在创建文件时保证文件名不重复）; 最后删除临时目录。如果在中间过程发生网络中断等情况造成无法与hdfs建立连接，需要用户手动删除已经写入的文件和临时目录。
* (7)、目前插件中Hive版本为1.1.1，Hadoop版本为2.5.0（Apache［为适配JDK1.6］,在Hadoop 2.6.0 和Hive 1.2.0测试环境中写入正常；其它版本需后期进一步测试；
* (8)、目前HdfsWriter不支持Kerberos等认证，所以用户需要保证DATAX有权限访问该hdfs节点,并且对指定的目录有写的权限

## 3 功能说明


### 3.1 配置样例

```json
{
    "setting": {},
    "job": {
        "setting": {
            "speed": {
                "channel": 2
            }
        },
        "content": [
            {
                "reader": {
                    "name": "txtfilereader",
                    "parameter": {
                        "path": ["/Users/shf/workplace/txtWorkplace/job/dataorcfull.txt"],
                        "encoding": "UTF-8",
                        "column": [
                            {
                                "index": 0,
                                "type": "long"
                            },
                            {
                                "index": 1,
                                "type": "long"
                            },
                            {
                                "index": 2,
                                "type": "long"
                            },
                            {
                                "index": 3,
                                "type": "long"
                            },
                            {
                                "index": 4,
                                "type": "DOUBLE"
                            },
                            {
                                "index": 5,
                                "type": "DOUBLE"
                            },
                            {
                                "index": 6,
                                "type": "STRING"
                            },
                            {
                                "index": 7,
                                "type": "STRING"
                            },
                            {
                                "index": 8,
                                "type": "STRING"
                            },
                            {
                                "index": 9,
                                "type": "BOOLEAN"
                            },
                            {
                                "index": 10,
                                "type": "date"
                            },
                            {
                                "index": 11,
                                "type": "date"
                            }
                        ],
                        "fieldDelimiter": "\t"
                    }
                },
                "writer": {
                    "name": "hdfswriter",
                    "parameter": {
                        "defaultFS": "hdfs://10.101.204.12:9000",
                        "fileType": "orc",
                        "path": "/user/hive/warehouse/writerorc.db/orcfull",
                        "fileName": "qiran",
                        "column": [
                            {
                                "name": "col1",
                                "type": "TINYINT"
                            },
                            {
                                "name": "col2",
                                "type": "SMALLINT"
                            },
                            {
                                "name": "col3",
                                "type": "INT"
                            },
                            {
                                "name": "col4",
                                "type": "BIGINT"
                            },
                            {
                                "name": "col5",
                                "type": "FLOAT"
                            },
                            {
                                "name": "col6",
                                "type": "DOUBLE"
                            },
                            {
                                "name": "col7",
                                "type": "STRING"
                            },
                            {
                                "name": "col8",
                                "type": "VARCHAR"
                            },
                            {
                                "name": "col9",
                                "type": "CHAR"
                            },
                            {
                                "name": "col10",
                                "type": "BOOLEAN"
                            },
                            {
                                "name": "col11",
                                "type": "date"
                            },
                            {
                                "name": "col12",
                                "type": "TIMESTAMP"
                            }
                        ],
                        "writeMode": "truncate",
                        "fieldDelimiter": "\t",
                        "compress":"NONE"
                    }
                }
            }
        ]
    }
}
```

### 3.2 参数说明

* **defaultFS**

	* 描述：Hadoop hdfs文件系统namenode节点地址。格式：hdfs://ip:端口；例如：hdfs://10.101.204.12:9000<br />


		**特别需要注意的是，目前HdfsWriter不支持Kerberos等认证，所以用户需要保证DATAX有权限访问该节点**


	* 必选：是 <br />

	* 默认值：无 <br />
	
* **hdfsSiteXml**

	* 描述：hdfs-site.xml文件的路径,只有在NamenodeHA的情况下需要配置.<br />
		

	* 必选：否 <br />

	* 默认值：无 <br />

* **fileType**

	* 描述：文件的类型，目前只支持用户配置为"text"或"orc"。 <br />

		text表示textfile文件格式

		orc表示orcfile文件格式

	* 必选：是 <br />

	* 默认值：无 <br />
* **path**

	* 描述：存储到Hadoop hdfs文件系统的路径信息，HdfsWriter会根据并发配置在Path目录下写入多个文件。为与hive表关联，请填写hive表在hdfs上的存储路径。例：Hive上设置的数据仓库的存储路径为：/user/hive/warehouse/ ，已建立数据库：test，表：hello；则对应的存储路径为：/user/hive/warehouse/test.db/hello  <br />

	* 必选：是 <br />

	* 默认值：无 <br />

* **fileName**

 	* 描述：HdfsWriter写入时的文件名，实际执行时会在该文件名后添加随机的后缀作为每个线程写入实际文件名。 <br />

	* 必选：是 <br />

	* 默认值：无 <br />
* **column**

	* 描述：写入数据的字段，不支持对部分列写入。为与hive中表关联，需要指定表中所有字段名和字段类型，其中：name指定字段名，type指定字段类型。 <br />

		用户可以指定Column字段信息，配置如下：

		```json
		"column":
                 [
                            {
                                "name": "userName",
                                "type": "string"
                            },
                            {
                                "name": "age",
                                "type": "long"
                            }
                 ]
		```

	* 必选：是 <br />

	* 默认值：无 <br />
* **writeMode**

 	* 描述：hdfswriter写入前数据清理处理模式： <br />

		* append，写入前不做任何处理，DataX hdfswriter直接使用filename写入，并保证文件名不冲突。
		* nonConflict，如果目录下有fileName前缀的文件，直接报错。

	* 必选：是 <br />

	* 默认值：无 <br />

* **fieldDelimiter**

	* 描述：hdfswriter写入时的字段分隔符,**需要用户保证与创建的Hive表的字段分隔符一致，否则无法在Hive表中查到数据** <br />

	* 必选：是 <br />

	* 默认值：无 <br />

* **compress**

	* 描述：hdfs文件压缩类型，默认不填写意味着没有压缩。其中：text类型文件支持压缩类型有gzip、bzip2;orc类型文件支持的压缩类型有NONE、SNAPPY（需要用户安装SnappyCodec）。 <br />

	* 必选：否 <br />

	* 默认值：无压缩 <br />

* **encoding**

	* 描述：写文件的编码配置。<br />

 	* 必选：否 <br />

 	* 默认值：utf-8，**慎重修改** <br />


### 3.3 类型转换

目前 HdfsWriter 支持大部分 Hive 类型，请注意检查你的类型。

下面列出 HdfsWriter 针对 Hive 数据类型转换列表:

| DataX 内部类型| HIVE 数据类型    |
| -------- | -----  |
| Long     |TINYINT,SMALLINT,INT,BIGINT |
| Double   |FLOAT,DOUBLE |
| String   |STRING,VARCHAR,CHAR |
| Boolean  |BOOLEAN |
| Date     |DATE,TIMESTAMP |


## 4 配置步骤
* 步骤一、在Hive中创建数据库、表
Hive数据库在HDFS上存储配置,在hive安装目录下 conf/hive-site.xml文件中配置，默认值为：/user/hive/warehouse
如下所示：

```xml
<property>
    <name>hive.metastore.warehouse.dir</name>
    <value>/user/hive/warehouse</value>
    <description>location of default database for the warehouse</description>
  </property>
```
Hive建库／建表语法 参考 [Hive操作手册]( https://cwiki.apache.org/confluence/display/Hive/LanguageManual)

例：
（1）建立存储为textfile文件类型的表
```json
create database IF NOT EXISTS hdfswriter;
use hdfswriter;
create table text_table(
col1  TINYINT,
col2  SMALLINT,
col3  INT,
col4  BIGINT,
col5  FLOAT,
col6  DOUBLE,
col7  STRING,
col8  VARCHAR(10),
col9  CHAR(10),
col10  BOOLEAN,
col11 date,
col12 TIMESTAMP
)
row format delimited
fields terminated by "\t"
STORED AS TEXTFILE;
```
text_table在hdfs上存储路径为：/user/hive/warehouse/hdfswriter.db/text_table/

（2）建立存储为orcfile文件类型的表
```json
create database IF NOT EXISTS hdfswriter;
use hdfswriter;
create table orc_table(
col1  TINYINT,
col2  SMALLINT,
col3  INT,
col4  BIGINT,
col5  FLOAT,
col6  DOUBLE,
col7  STRING,
col8  VARCHAR(10),
col9  CHAR(10),
col10  BOOLEAN,
col11 date,
col12 TIMESTAMP
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS ORC;
```
orc_table在hdfs上存储路径为：/user/hive/warehouse/hdfswriter.db/orc_table/

* 步骤二、根据步骤一的配置信息配置HdfsWriter作业

## 5 约束限制

略

## 6 FAQ

略
