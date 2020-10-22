# 数据血缘

## 1.介绍

数据血缘是集成在库表管理服务中的一个功能模块提供关联表的血缘图：

### 工作流（lhotse）血缘

* hbase库表节点 （hdfs到hbase）
* hive库表节点 （hdfs到hive,hive到hdfs,hiveSql）

## 2.portal界面入口

如果该库表存在上下可追溯的lhotse调度任务，则判断它存有血缘关系该关联表数值不为0：

![](../.gitbook/assets/table.png)

![](../.gitbook/assets/table2.png)

## 3-1.工作流数据

点击数字旁边的链接按钮画布里显示的是当前选择表的所有血缘关系。以hive表为例，显示的是这个表参与工作流任务的任务链关系，从哪个hdfs目录导入数据之后又导出到哪里如以下效果图：

hive表-默认显示全部血缘 ![](../.gitbook/assets/hive%20%282%29.png)

![](../.gitbook/assets/hive2.png)

hbase表-默认显示全部血缘 ![](../.gitbook/assets/hbase%20%281%29.png)

**注意**：如果右边画布只显示单个当前节点的信息，说明该节点没有参与工作流任务，所以也没有上下可追溯的血缘关系，表结构中的关联表值为0。

