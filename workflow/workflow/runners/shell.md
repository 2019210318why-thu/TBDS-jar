# shell 脚本

执行shell脚本

## 功能说明

执行一个shell脚本，系统不会对shell进行语法检查。

## 其他说明

shell脚本执行者为任务第一个责任人（portal登录用户\)

## 任务设置

### 1. 基本信息

参考 [基本信息设置](../runnerbasicinfo.md)

### 2. 调度

参考 [调度设置](../runnercycle.md)

### 3. 参数

参数设置如下图所示： ![shell &#x53C2;&#x6570;&#x8BBE;&#x7F6E;](../../../.gitbook/assets/shell.jpg)   
 1. 执行包路径  
将需要执行的shell 文件，压缩为一个zip包，点击‘上传脚本’ 将zip 上传到tbds

1. shell 命令 执行上传shell 文件的shell 命令。 如果要让不同的实例，执行不同的shell 命令，该参数 支持[时间隐式参数](../other/implicitvariable.md)
2. 执行参数 需要传入给shell 命令的参数。 支持[时间隐式参数](../other/implicitvariable.md)

## demo

执行一个简单的打印例子  
执行包路径：[打印参数](https://share.weiyun.com/5SBOB6M)  
  如果要测试 执行失败，可以下载[任务失败](https://share.weiyun.com/11c232afeb32c12ccbefadd9a520526f)  
shell执行命令：  
  ./testMultiParameter.sh   任务失败 使用./testerror.sh

有关在shell命令执行hdfs 命令说明 shell 任务会将hdfs 认证信息和鉴权用户加上，用户不用手动导入对应的认证ID和KEY。只需要保证任务第一责任人在对应hdfs 目录上有权限就ok。

### 注意

1. 如果在windows 上编辑，确认文件编码为： 无 BOM 的 UTF-8   
2. 如果在shell 文件中执行跟多复杂操作，比如execRemoteCmd.exp文件等，可能会出现中文乱码，需要在shell 文件中显示指定编码export LANG="zh\_CN.UTF-8"   
3. shell 中有多个命令，其中有命令执行失败，需要抛出对应异常   

   比如：  

   ```text
   hadoop fs -ls /tmp
   exit_status=$?
   if [ $exit_status -ne 0 ]; then
    echo "failed"
    exit 1
   fi
   ```

4. shell 脚本的功能尽可能单一，不建议在shell脚本中写复杂逻辑   

   比如不建议通过shell 的方式来处理跨工作流依赖，而是使用工作流自带的[跨画布依赖](../other/gapworkflow.md)功能。

