# 链接归集

## 概述

链接归集提供用户访问大数据套件TBDS内部组件的部分web和展示内部组件注册过服务发现的端口列表。  
组件如hdfs NN的web访问需要 先授权，授权之后才能访问。

## 使用

### 1.链接访问

 **步骤1.** 打开TBDS portal，输入正确的用户名密码；进入portal主页面。

![](../../.gitbook/assets/portal_main.png)

**步骤2.** 依次点击进入“运维中心--&gt;系统运维--&gt;链接管理”，进入链接管理页面，默认停留在链接页面。

![](../../.gitbook/assets/links_main.png)

**步骤3.** 查看链接列表，确定需要访问的链接项，点击对应链接条目右侧的“申请链接权限”

![](../../.gitbook/assets/links_apply.png)

**步骤4.** 申请之后需要等待管理员的审批，审批通过之后才能继续访问.下面说明管理员审批步骤

 **步骤4.1.** 管理员登录TBDS portal，进入portal主页面

![](../../.gitbook/assets/admin_user_center.png)

**步骤4.2.** 点击页面右上角的"个人中心"，进入个人中心,一次点击“审批单--&gt;链接申请”，就可以看到普通用户刚刚提交的链接访问申请。点击申请单右侧的“通过”，则授权成功

![](../../.gitbook/assets/admin_allow.png)

**步骤4.3.** 审批通过的单据可以通过以下方式查看

![](../../.gitbook/assets/admin_allow_check.png)

**步骤5.** 普通用户这个时候再次登录链接归集页面就可以看到刚才想要访问的地址的具体url，至此后台也在管理员审批的同时赋予了普通用户访问此链接的权限。

![](../../.gitbook/assets/user_see_links.png)

**步骤6.** 根据链接归集页面上的提示配置普通用户所在机器的主机-ip映射。以windows为例，则需要配置C:\Windows\System32\drivers\etc\host，在该文件里加上想要访问的链接的host项。

![](../../.gitbook/assets/client_hosts.png)

 **步骤7.** 在浏览器输入链接地址，并输入用户名密码就可以打开具体web页面了。用户名密码同登录portal所用的用户名密码。

![](../../.gitbook/assets/hbase_example_link.png)

### 2.访问端口列表

直接进入portal的链接归集页面，点击右侧的“端口列表”即可。

![](../../.gitbook/assets/port_list.png)

