hdfs导入mysql

### 功能说明
hdfs数据导入到mysql。

### 其他说明


### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  
#### 2. 调度  
参考 [调度设置](/workflow/workflow/runnerCycle.md)  

#### 3. 参数
参数设置如下图所示：
![hdfs2mysql](/workflow/workflow/images/hdfs2mysql.png)

__ps:__ 红框部分为需要关注参数  
1. 源服务器  
源目标服务器是hdfs 服务器连接信息。[更多参数](/workflow/services/readme.md)  

2. 目标服务器  
目标服务器是mysql 服务器连接信息。[更多参数](/workflow/services/readme.md)  

3. 源目录  
指定hdfs 上哪些目录（文件）数据导入mysql。  
**不支持 hdfs 数据目录不存在**  
4. 分隔符  
源服务器上待导入到mysql 的hdfs 数据文件字段内容分隔符。  
一个任务只能支持同一个分隔符。

5. 目标表  
待写入数据的mysql 表名。表对应的数据库已经在服务器配置中指定，这里不需要重复指定。  
**如果插入的数据有中文，请确保创建的mysql表编码格式为utf8。**   

6. 目标表列名  
待写入数据的mysql 表对应的字段，其字段个数和循序需要和hdfs 数据文件内容要匹配。不同的列名使用逗号分隔。  
**不支持db 表字段以整数开头**  

7. 是否分区  
是否分区是指mysql 表是否为分区表。  
    * 如果选择否，直接写入数据。  
    * 如果选择是，则需要进一步制定分区字段格式，目前只支持时间格式，通常任务调度周期和字段分区类型对应。比如任务调度周期是天，数据按天分区，分区字段选择P_${YYYYMMDD} 。在将数据写入mysql 中，执行的sql 语句中会嵌入partition 关键字。  

8. 数据入库模式  
append和truncate两种模式。  
使用truncate模式，写入数据之前会将对应的分区数据删除，如果mysql不是分区表，那么会将mysql 表所有数据删除。append 模式不会删除历史数据。     
**请谨慎选择truncat 模式。**  

9. 允许误差  
允许出错的百分比，20代表允许有20%的数据可以读或者写失败，0代表不允许有任何数据读写失败。读写各算一次失败。    

10. 数据为空是否成功  
如果数据源数据为空，任务是否成功。  
选择是，读写数据为空，任务实例成功。  
选择否，读写数据为空，任务实例失败。  

11. 读并发度  
读hdfs 数据的并发线程数。

12. 写并发度  
数据写入mysql的并发线程数。 

### demo
如上图所示

### demo资源