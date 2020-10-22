# 基本概念

## 1、说明

服务器配置在【工作流】——&gt;【服务器配置】页签也操作，常用操作：服务器配置的添加、修改、删除，界面如下： ![](../../.gitbook/assets/1.png)

## 2、使用

配置的服务器，主要供工作流的任务使用。如：创建MYSQL导入HIVE任务，需要在参数配置中设置：源服务器（MYSQL）和目标服务器（HIVE），服务器从任务所对应项目下的服务器配置进行选择。便于服务器的维护和多次使用。

![](../../.gitbook/assets/2.png)

## 3、类型

目前服务器类型支持有8种，分别是：mysql, postgres, oracle, sqlserver, hive, ftp, hdfs, mr-group。 ![](../../.gitbook/assets/type.png)

