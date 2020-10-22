# hbase

## 背景

大数据套件的hbase服务基于社区hbase 1.2.1版本改造。主要加强安全认证，加入套件基于ACCESSKEY的认证。  
accesskey相关操作参考对应章节。

## 使用

无认证方式访问  
无认证访问即社区的普遍访问方式。大数据套件hbase组件默认开启了基于accesskey的TBDS认证，不能无认证访问。

### TBDS认证方式访问

#### 1.hbase shell访问

 **步骤0.**  确认访问用户，在portal页面获取该用户在套件系统关于hbase模块的accesskey（secureId，secureKey）

 **步骤1.**设置用户accesskey相关的secrueId和secureKey为环境变量：

```text
    export hbase_security_authentication_tbds_secureid=2UIOLmuK93mcc4F1UWpYqzUo5TkfqFycJZiI 
    export hbase_security_authentication_tbds_securekey=zluIBbYX4RRIYPrU2Dx3whX19kBjTMTe
```

accesskey关联什么用户，当前操作hbase就是什么用户，换言之，如果想要什么用户操作hbase，就要导入相应的accesskey。

 **步骤2.**执行普通的hbase的shell命令操作hbase：  
hbase shell  
完整示例命令：  
![](../.gitbook/assets/complete_shell.png)

#### 2.java client访问

代码基本与开源hbase的client代码一致，只是在连接需要用到的Configuration中加入用户的认证信息（secureId，secureKey）即可。代码片段示例：

```text
public static Connection createconnection(String secureId, String secureKey,String zk) {
  Configuration configuration = HBaseConfiguration.create();  
  configuration.set("hbase.zookeeper.quorum", zk);  
  configuration.set("zookeeper.znode.parent", "/hbase-unsecure");
```

认证时，在创建HBase连接之前，在configuration设置以下两个参数，其他使用方法和HBase开源完全一样,  
hbase其他接口的使用方法类似，只需要增加以下两个认证参数即可

```text
  configuration.set("hbase.security.authentication.tbds.secureid", secureId);  
  configuration.set("hbase.security.authentication.tbds.securekey", secureKey);  
  try {  
      return ConnectionFactory.createConnection(configuration);  
  } catch (IOException e) {  
      e.printStackTrace();
      return null;
  }
}
```

如果使用proxyUser方式访问hbase，则还需要在客户端加入如下代码片段：

```text
// 想以什么身份跑任务的用户，自己不用登录认证，自己的身份由loginUser保证真实性，由loginUser代理
String doasUser = "userA";
// 真正要进行认证的用户，跑任务时，不是以该用户的身份。 该用户要保障doasUser身份的真实性。整个认证的逻辑是：HBase系统对loginUser进行认证,
// 而loginUser对doasUser进行认证
String loginUser = "userB";
UserGroupInformation ugi =
    UserGroupInformation.createProxyUser(doasUser,
        UserGroupInformation.createRemoteUser(loginUser));
```

注意：1）这个代码片段需要放在hbase连接之前执行；2）userB即跟accesskey相对应的用户，只用来做认证，userA是实际访问数据用户。

#### 3.Thrift RPC访问

1）生成认证签名  
以用户的secureId、secureKey，当前时间戳timestamp，任意随机数nonce为输入， 根据公开的算法得到用户签名Signature。签名算法实现如下：

```text
public static String generateSignature(String secureId, long timestamp, int randomValue,
    String secureKey) {
  Base64 base64 = new Base64();
  byte[] baseStr =
      base64.encode(HmacUtils.hmacSha1(secureKey, secureId + timestamp + randomValue));
  String result = "";
  try {
    result = URLEncoder.encode(new String(baseStr), "UTF-8");
  } catch (UnsupportedEncodingException e) {
    e.printStackTrace();
  }
  return result;
}
```

2）准备认证参数  
把1）中的secureId，timestamp，nonce，Signature以空格分隔组成一个字符串，作为认证信息userAuthInfo&lt;/br&gt;  
3\) 使用认证参数进行hbase Thrift连接  
这里的代码是标准的Thrift sasl连接代码，代码片段如下：

```text
TTransport transport = new TSocket(host, port);

// mechanism必须设置为 PLAIN
transport = new TSaslClientTransport("PLAIN", null, null, null, null, new CallbackHandler() {
  @Override
  public void handle(Callback[] callbacks) throws IOException, UnsupportedCallbackException {
    for (Callback callback : callbacks) {
      if (callback instanceof NameCallback) {
        // 在NameCallBack里面设置认证字符串，格式必须是各个参数以空格隔开，参数顺序依次是：secureId timestamp nonce signature
        // 具体格式见getAuthentication函数返回的格式
        ((NameCallback) callback).setName(userAuthInfo);
      } else if (callback instanceof PasswordCallback) {
        // PasswordCallback设置为空即可，服务端认证时不用
        ((PasswordCallback) callback).setPassword("".toCharArray());;
      } else if (callback instanceof AuthorizeCallback) {
        // AuthorizeCallback设置为空即可，服务端认证时不用
        ((AuthorizeCallback) callback).setAuthorizedID("");
      } else {
        throw new UnsupportedCallbackException(callback);
      }
    }
  }
}, transport);
transport.open();
```

#### 4.phoenix访问

phoenix访问方式不变，只是在连接字符串中多加两个参数（用户的secureId，secureKey）。  
在原来的链接字符串之后增加认证参数：hbase.security.authentication.tbds.secureid 和 hbase.security.authentication.tbds.securekey。 &lt;/br&gt;  
示例：  
原认证字符串&lt;/br&gt;

```text
jdbc:phoenix:tbds-10-255-0-10,tbds-10-255-0-4,tbds-10-255-0-2:2181:/hbase-unsecure
```

增加认证后的字符串：

```text
jdbc:phoenix:tbds-10-255-0-10,tbds-10-255-0-4,tbds-10-255-0-2:2181:/hbase-unsecure ?hbase.security.authentication.tbds.secureid=PdtRcxASjDKUxlvsQ9yKP4bXotMYybLFLoug & hbase.security.authentication.tbds.securekey=NVBjm6wfrBLhDJ673c4jaBIRLV9zMnqy
```

注意 ：  
第一个参数之前增加问号（？），后面的各个参数之间以&连接  
需要引用jar包 phoenix-core-4.8.1-HBase-1.2-TBDS.jar

## 注意事项

1）在用java代码生成用户Signature时，需要引用1.10以上版本的commons-codec jar包,否则计算的签名可能会不一样

2）客户端引用的hbase依赖jar要使用套件版本的hbase lib，不能用开源的hbase lib

3）如果碰到无权限访问hbase如scan等操作，需要去portal的访问管理配置对应用户的访问策略

## 集群外客户端部署

 **1）准备安装软件，可以通过以下3种方式任选其一获取**  
（1）在任一安装hbase客户端的集群内节点，打包hbase的安装路径：/usr/hdp/2.2.0.0-2041/hbase/，并拷贝到集群外目标客户端安装节点

（2）如果有套件的yum源，在集群外目标安装节点使用yum install安装

（3）如果有hbase安装包，在集群外目标安装节点使用rpm -ivh安装

 **2）获取hbase集群配置，可以通过以下2中方式任选其一获取：**  
（1）在任一安装hbase客户端的集群内节点，打包hbase的配置路径： /etc/hbase/conf，并拷贝到集群外目标客户端安装节点对应路径  
（2）通过ambari界面下载hbase配置：选中任一一台安装有hbase客户端的机器，在机器操作选项中选择“下载客户端配置--&gt;Hbase client”，如下图所示：

## 客户端代码编译打包

 **1）套件版hbase jar命名规则**  
套件的hbase是基于社区二次开发，命名规则采用"社区版本号-TBDS-套件版本号"的方式命名.例：我们现在基于社区1.2.1版本的hbase进行开发，套件版本是4.0.3.3，则我们打出的hbase jar版本为1.2.1-TBDS-4.0.3.3，完整的hbase client maven jar文件名为：hbase -client-1.2.1-TBDS-4.0.3.3.jar

 **2）基于套件提供的maven库开发**  
（1）拷贝或部署套件提供的maven库到开发者可访问的本地仓库或远程仓库  
（2）在客户端maven工程pom引入对应的套件版hbase依赖，以套件4.0.3.3版本为例，需要在pom中加入的依赖片段（其他版本依次类推）：

```text
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId> hbase -client</artifactId>
        <version>1.2.1-TBDS-4.0.3.3</version>
    </dependency>
```

**3）基于开源maven库开发 （强烈不建议使用）**  
这种方式建议不使用。&lt;/br&gt;  
（1）在客户端maven工程引入对应的社区版本hbase依赖。

```text
    <dependency>
        <groupId>org.apache. hbase </groupId>
        <artifactId> hbase -client</artifactId>
        <version>1.2.1</version>
    </dependency>
```

（2）编译完成之后，在运行客户端程序运行之前，把套件版的hbase jar及其间接依赖加入classpath中：  
hbase -client-1.2.1-TBDS-4.0.3.3.jar  
hadoop-auth-2.7.2-TBDS-4.0.3.3.jar  
hadoop-common-2.7.2-TBDS-4.0.3.3.jar

## 运行

运行客户端代码与社区方式无区别

## Hbase开发简单实例

1\) jar包

jar包作用：打印tableName信息

2\) 样例代码

```java
package com.tencenthbase;

import java.io.Closeable;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.HBaseAdmin;

public class HBaseClient {

    public static Connection createconnection(String secureId, String secureKey,String zk) {
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", zk);
        configuration.set("zookeeper.znode.parent", "/hbase-unsecure");

        // 认证时，在创建HBase连接之前，在configuration设置以下两个参数，其他使用方法和HBase开源完全一样,hbase其他接口的使用方法类似，只需要增加以下两个认证参数即可
        configuration.set("hbase.security.authentication.tbds.secureid", secureId);
        configuration.set("hbase.security.authentication.tbds.securekey", secureKey);

        try {
            return ConnectionFactory.createConnection(configuration);
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    public static void close(Closeable closeable){
        if (closeable != null) {
            try {
                closeable.close();
            } catch (Exception e) {e.printStackTrace();}
        }
    }

    @SuppressWarnings("deprecation")
    public static void main(String[] args) throws Exception {
        if (args == null || args.length != 3) {
            System.out.println("Usage: seucreId secureKey zkInfo(eg. zkIp1:port,zkIp2:port,zkIp3:port)");
            System.exit(1);
        }

        Connection connection = null;
        HBaseAdmin hbaseAdmin = null;
        try {
            connection = createconnection(args[0], args[1],args[2]);
            if (connection == null) {
                return;
            }

            hbaseAdmin = new HBaseAdmin(connection);
            String[] tableNames = hbaseAdmin.getTableNames();

            System.out.println("Scanning tables...");
            for (String tableName : tableNames) {
                System.out.println("  found: " + tableName);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            close(hbaseAdmin);
            close(connection);
        }
    }
}
```

3\) pom.xml文件

```markup
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>TBDSDemo2</artifactId>
        <groupId>com.tencent</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>HBaseDemo</artifactId>

    <!--  tencent 远程仓库   -->
    <repositories>
        <repository>
            <id>tbds</id>
            <url>https://tbdsrepo.cloud.tencent.com/repository/tbds/</url>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
            <releases>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </releases>
        </repository>
    </repositories>
    
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <hbase.version>1.2.1-TBDS-4.0.3.3</hbase.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId> hbase-client</artifactId>
            <version>${hbase.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin </artifactId>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <mainClass>com.tencenthbase.HBaseClient</mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>



</project>
```

4\) Demo提交运行步骤

```markup
# 参数1： hbase_security_authentication_tbds_secureid
# 参数2： hbase_security_authentication_tbds_secureKey
# 参数3： zookeeper集群地址
java -jar HBaseDemo.jar iTDmB3JeXmtbQCnR1ZDmYEwvevSQCmXDM*** gtLpa8NJh0h0wJ00bLTygpuApFjRa*** tbds-10-1-0-24:2181,tbds-10-1-0-35:2181,tbds-10-1-0-7:2181,tbds-10-1-0-26:2181,tbds-10-1-0-131:2181
```

5\) 预期结果

```markup
Scanning tables...
  found: HDFS_NM:TB_DIR_INFO
  found: METADATA:AE_HBASE_COLUMN_FAMILY
  found: METADATA:AE_HBASE_NAMESPACE
  found: METADATA:AE_HBASE_TABLE
  found: METADATA:AE_HDFS_PATH
  found: METADATA:AE_HIVE_COLUMN
  found: METADATA:AE_HIVE_DB
  found: METADATA:AE_HIVE_PARTITION
  found: METADATA:AE_HIVE_PROCESS
  found: METADATA:AE_HIVE_STORAGEDESC
  found: METADATA:AE_HIVE_TABLE
  found: METADATA:AE_KAFKA_DB
  found: METADATA:AE_KAFKA_TOPIC
  found: METADATA:AE_THIVE_TABLE
  found: METADATA:ENTITY_RELATION
  found: METADATA:GLOSSARY
  found: METADATA:GUID_TYPE_MAPPING
  found: METADATA:METADATA_GLOSSARY_QN_IDX
  found: METADATA:METADATA_TERM_GID_IDX
  found: METADATA:METADATA_TERM_QN_IDX
  found: METADATA:TERM
  ......
```

