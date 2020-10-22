# 新建表

库表管理支持Hive、Hbase的数据表操作，点击右上角 **数据表** tab页签，点击 **新建表** 进行新建数据表操作。

![add\_table](../../.gitbook/assets/add_table.png)

## 1、Hive表新建

新建Hive表支持手动创建、SQL脚本创建、文件导入三种方式。

### 1.1、手动创建

![table\_base\_01](../../.gitbook/assets/table_base_01.png) 填写创建Hive表的基本信息，便于数据表的授权、管理；

**参数解释：**

* 所属项目：新建表关联的项目，一个表只能属于一个项目；
* 责任人：新建表的责任人，默认是当前用户，可以有多个，责任人可以在可管理库表页签下查看新建的表；
* 类型：暂支持hive、hbase两种；
* 所属数据库：列出所属项目下，类型相同的数据库；
* 创建方法：hive支持手动创建、SQL/脚本创建、从文件导入
* 表名：自定义hive表名；
* 表描述：表的备注信息，可选；

设置Hive表的高级属性值，丰富Hive表的创建能力；

![table\_high\_01](../../.gitbook/assets/table_high_01.png)

**参数解释：**

* 记录格式：支持按照分隔符分隔、SerDe序列化；
* 字段终止符：Hive默认是\u001，支持自定义输入，可选；
* 集合终止符：对于数组数据类型的分隔符，可选；
* Map键终止符：对于map数据类型的分隔符，可选；
* 文件格式：支持Hive常用的文件格式，包括：TextFile、SequenceFile、InputFormat，默认使用TextFile;

设置Hive表的字段值和对应的数据类型，备注信息等；

![table\_column](../../.gitbook/assets/table_column.png)

**参数解释：**

* 字段名：新建Hive表字段的名称；
* 字段中文名：表字段中文注释名，可选；
* 字段类型：支持Hive允许的字段类型；
* 字段描述：字段的备注信息，可选；
* 字段规则：分区字段的字段规则；
* 字段分区：设置该字段是否为分区字段，默认是非分区字段；

### 1.2、SQL脚本创建

库表管理支持以Hive SQL的方式直接创建数据表，便于添加已有SQL语句的数据表，方便用户执行SQL语句。

![table\_sql](../../.gitbook/assets/table_sql.png)

**备注：**

* 脚本编辑：输入框，里面填写创建Hive表语句，末尾无需以";" 分号分隔；
* 通过HiveSQL创建的数据表，默认归属的数据库以参数 **所属数据库** 确定，即SQL语句中指定的数据库归属是无效的；

### 1.3、文件导入

![table\_load\_file](../../.gitbook/assets/table_load_file.png)

库表管理支持上传数据文件的形式，创建Hive表，并将数据文件加载到对应的Hive表。数据文件从HDFS**用户根目录**和所属项目的HDFS**项目目录**两个路径加载。

![table\_load\_column](../../.gitbook/assets/table_load_column.png)

可以通过下载示例模板参考支持上传的文件格式。加载时默认第一个行是字段名信息。目前支持字段分隔符有逗号、分号；

**备注：**

* 通过文件导入方式创建的表是Managed Table，管理表；
* 导入的文件不会被删除、也不会被移动，会复制临时文件，用于表创建，并将数据导入新建表中；

## 2、HBase表新建

### 2.1、手动创建

![table\_hbase\_create](../../.gitbook/assets/table_hbase_create.png) 填写HBase的基本信息，基本信息与Hive一致，不重复介绍。在创建Hbase表时，设置Hbase的列簇信息；

