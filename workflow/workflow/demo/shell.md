# Shell脚本

## 步骤（1）新建任务

拖拽画布左上角“+”，在弹出的窗口搜索框中输入shell，选中“SHELL脚本”，点击“确定”完成任务的创建。

![](../../../.gitbook/assets/6470725a35f8cf52b7ec721055e03180.png)

## 步骤（2）填写任务基本信息

鼠标双击任务框，进入任务的编辑界面。

![](../../../.gitbook/assets/4f163d066121cf7bbe9eb65a72a6e1e6.png)

在“基本信息”中填写任务名称。

![](../../../.gitbook/assets/7e23ef699fde486455960cab6bbfb461.png)

## 步骤（3）填写任务调度设置

在“调度设置”中选择周期类型“一次性非周期”。

![](../../../.gitbook/assets/9b2c153cef6c2e47b828b71e83bc334d.png)

## 步骤（4）填写任务参数设置

点击“上传脚本”，在弹出的窗口中上传shell脚本文件log.sh对应的压缩包“log.zip”。脚本log.sh文件内容：

注意：脚本文件必须打包成.zip文件上传，否则将运行出错

然后在“shell命令”一栏中输入shell执行命令：./log.sh

![](../../../.gitbook/assets/e1a3c40fb23c2a736a4c977dfa364331.png)

![](../../../.gitbook/assets/b65e181d1bdf993fb23fe8cb5d3e79d5.png)

依次点击左上方保存按钮 和 运行按钮。

![](../../../.gitbook/assets/dc537ccb338e42432f51cffbd315db17.png)

弹出框提示任务运行需要审批，选中“审批通过后自动运行”，点击确定，返回主界面。

![](../../../.gitbook/assets/4e095b63d5d697063db876c71cae4450.png)

返回主界面，发现任务处于审批中。

![](../../../.gitbook/assets/d4f848c9640ca0e1378ea8dd391dc5cd.png)

## 步骤（5）查看任务状态

审批通过后，任务进入“运行”状态，右键任务框，点击“查看运行状态”，依次为显示“等待调度”和“运行中”。

![](../../../.gitbook/assets/178e62b4f11ddb6ed3e219bc3957c970.png)

![](../../../.gitbook/assets/28671083e7713acf65d213fdd30901b8.png)

![](../../../.gitbook/assets/ae853c782680ae8013e2a9e20f5f2c79.png)

## 步骤（6）验证任务是否成功

状态变为“成功”后，点击“查看”，可以看到执行的SHELL脚本和运行结果。

![](../../../.gitbook/assets/040d69f49921ab1d9a2cb66f21df1f90.png)

![](../../../.gitbook/assets/72637deea12746ffcacc8bc337f33b65.png)

