# MySQL导入Hive

## 步骤（1）新建工作流

点击工作流列表右上角的“+”，在弹出的窗口中输入工作流名称“mysql\_to\_hive\_demo”，点击“确定”完成工作流的创建。

![](../../../.gitbook/assets/0daa1117cd3c1bcdc63752992bbd7582.png)

![](../../../.gitbook/assets/982be46c80ab299f69fab758128a923d.png)

## 步骤（2）新建任务

拖拽画布左上角“+”，在弹出的窗口中选中“MYSQL导入HIVE”，点击“确定”完成任务的创建。

![](../../../.gitbook/assets/e197fe27cba4e6d40c7d251aa401c910.png)

![](../../../.gitbook/assets/3c64c6139213f4b76258b7dc135ed614.png)

## 步骤（3）填写任务基本信息

右键点击任务框，点击“编辑”进入任务的编辑界面。

![](../../../.gitbook/assets/056a03003532d317bb19024a2ab7987a.png)

在“基本信息”中填写任务名称“mysql\_to\_hive\_demo”。

![](../../../.gitbook/assets/55eb25e2dc669bfb9291bfc5ddc8e56a.png)

## 步骤（4）填写任务调度设置

在“调度设置”中选择周期类型“一次性非周期”。

![](../../../.gitbook/assets/6bc0563a978bf38d4d7b3426a47e3c21.png)

## 步骤（5）填写任务参数设置

### 配置源服务器

点击“服务器配置”页面的“新建配置”图标，在弹出窗口中，选择服务器类型“mysql”，输入服务器标识“mysql\_server”，填写MySQL服务器配置，包括主机地址、端口、database名称、数据库用户名和数据库密码。点击“连接测试”图标显示成功即可。

![](../../../.gitbook/assets/6a7816235f548135de81b2ac204cf8b6.png)

### 配置目标服务器

点击“服务器配置”页面的“新建配置”图标，在弹出窗口中，选择服务器类型“hive”，输入服务器标识“hive\_server”，填写Hive服务器配置，包括连接地址和端口。点击“连接测试”图标显示成功即可。

![](../../../.gitbook/assets/ff40cfbb051f4e35ce1dfce005024a4d.png)

在“参数配置”中选择源服务器“mysql\_server”，目标服务器“hive\_server”；输入Select SQL“select user as mysql\_user from mysql.user”，目标DB名“hive\_test”，目标表名“mysql\_user”，源文件列名“mysql\_user”，字段映射关系“mysql\_user”。注意：应确保存在上述服务器、DB、表和字段。

![](../../../.gitbook/assets/f50a7dab6c703a462279d410d4e239f0.png)

参数说明

源服务器：待导入数据所在的MySQL Server。

目标服务器：存储最终结果的Hive Server。

Select SQL：SQL查询语句。

目标DB名：MySQL DB名称。

目标表名：待写入数据的MySQL表名。

源文件列名：源文件的栏位名称，以英文逗号分割（结尾不能是逗号）,必须保证列数和文件内容一致. 创建Hive外表（临时表）所用表列名。

字段映射关系：Hive表列名,以英文逗号分隔,表示的列的内容顺序,需和DB列字段保持一致。决定从临时表往目的表里写的字段顺序。日期和常量需要用中括号包起来，例如：\[${YYYYMMDD}\], \[\'test\'\] 。

点击保存图标保存任务配置。

![](../../../.gitbook/assets/0c4f7f3ebe47e189f5f88b30281ddb02.png)

## 步骤（6）任务审批

返回工作流画布。右键点击任务框，点击“运行”，在弹出的窗口中勾选“审批通过后自动运行”，点击“确定”。

![](../../../.gitbook/assets/8fb5e94da8c1530d69f4464ef061504d.png)

![](../../../.gitbook/assets/2066a07b46570aea3610f6e69b81d2e0.png)

任务进入“审批中”状态，等待审批结果。

![](../../../.gitbook/assets/f63744182286ebdda0b66ef24166cf1f.png)

## 步骤（7）查看任务状态

审批通过后，任务进入“运行”状态，右键任务框，点击“查看运行状态”，依次为显示“等待调度”和“运行中”。

![](../../../.gitbook/assets/6f5b37068a600d2b30662082e0c1d815.png)

## 步骤（8）验证任务是否成功

状态变为“成功”后，点击“查看”，可以看到执行过程和运行结果。

![](../../../.gitbook/assets/e243ad50bc11320e2e60354b2f184051.png)

![](../../../.gitbook/assets/53f913c07a311340d536d744f0b256bc.png)

![](../../../.gitbook/assets/f2148619241c09de42ef489adeac5bc8.png)

![](../../../.gitbook/assets/152cbcbd66cd5ddc802d08a5c28743a5.png)

导入前后对比：

![](../../../.gitbook/assets/c1aad89d67dd1aa18598359b3e1ab161.png)

![](../../../.gitbook/assets/50aeaa6669bfbc983277663db40238cf.png)

