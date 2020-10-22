# hive

Hive的使用与开源没有本质区别，需要注意的主要有两点：1）Hive打开了Ldap认证，连接Hive时，需要设置用户名和密码；2）Hive实现了HA，通过在zk注册多个hiveserver实例实现，多个hiveserver并行工作，工作时，通过zk路由到某个可用的hiveserver上，各hiveserver之间互相独立。

## 1.1 Hive认证

Hive认证通过打开hive自带的Ldap认证功能来实现，连接hive时的用户名和密码被传递给HiverServer, HiveServer去Ldap里面进行比较。认证所用的用户有两种,两种用户都可以在连接hive时进行认证：

1）用户在大数据套件界面（以下简称portal界面）创建的用户和密码

2）系统自带默认用户（包括：

hive,httpfs,supersql,tbds\_shellUser,root,olap,hbase,hdfs,yarn,mapred,ranger,amshcat,hue,tez,zookeeper,ambari-qa,hadoop,flume,tencent,jstorm,kafka,lhotse,hdfs,postgres,redis,wherehows,hermes）  
这些默认用户及其密码都保存在套件portal节点的Ldap里，各个用户的初始密码为：用户名+@Tbds.com, 如hive用户的密码为：hive@Tbds.com hdfs 用户的密码为：hdfs@Tbds.com ，如果想更改这些用户的初始密码，可以通过ldap命令直接去ldap修改。

建议这些默认用户不暴露给最终用户使用，最终用户所使用的用户都是在portal页面创建的用户， 这里说明系统用户默认密码主要是为了方便套件上层开发的应用程序访问。

## 1.2使用Hive

1\)连接hive时，有两种不同的方式，对应的连接字符串也有所不同：

直接连接到指定的hiveserver,如： jdbc:hive2://172.16.32.10:10000 其中，172.16.32.10 为hiveserver ip, 10000为hiveserver port  
通过zk的方式连接hive,如：

```text
jdbc:hive2://172.16.32.14:2181,172.16.32.13:2181,172.16.32.8:2181/default;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2
```

其中，172.16.32.14:2181,172.16.32.13:2181,172.16.32.8:2181为zk ip和port ，使用时其他的字符串保持不变  
注意：建议通过zk的方式连接zk，可以保证hive的高可用性。

2）根据客户端的使用方式不同，有以下几种：

beeline方式连接，去安装有hive client的节点执行 beeline命令： !connect jdbc:hive2://172.16.32.10:10000 其中，链接字符串可以使用上节介绍的任意一种（推荐zk方式）  
java客户端使用jdbc驱动连接，连接字符串可以任意一种，用户名和密码参见《Hive认证》章节，需要引用hive的pom依赖为

```text
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
    <version>2.2.0-SNAPSHOT</version>
</dependency>
```

使用Thrift接口去连接Hive,使用方法和开源完全相同，传递用户名和密码时，参考《Hive认证》 章节即可

## 1.3 集群外部署Hive Client

 **方案1：**

```text
    1）通过yum源(提前配置好套件的yum源)安装套件版本的Hive到集群外机器：
        yum install  hive_*  hiveclient.x86_64  -y      
    2）拷贝集群内任一hive客户端机器/etc/hive/conf目录的内容，替换集群外客户端对应路径:
    集群内客户端节点：
        cd  /etc/hive/conf
        tar -zcvf  hive.tar.gz  /etc/hive/conf/*
    集群外客户端节点：
        cd  /etc/hive/conf
        tar -zxvf  hvie.tar.gz
```

 **方案2:**  
1.打包集群内任一hive客户端所在节点的/usr/hdp/2.2.0.0-2041/hive/，并拷贝到集群外目标客户端节点  
2.同方案1的步骤2

## 1.4客户端代码编译打包

1\) 拷贝或部署套件提供的maven库到开发者可访问的本地仓库或远程仓库  
2\) 在客户端maven工程pom引入对应的套件版hive依赖，版本号为：2.2.0-SNAPSHOT

```text
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
        <version>2.2.0-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-common</artifactId>
    <version>2.2.0-SNAPSHOT</version>
</dependency>
```

## 1.5 Hive开发简单实例

1\) jar包

jar包作用：创建表

2\) 样例代码

```java
package com.tencent.hive;

import org.apache.hive.service.auth.HiveAuthFactory;
import org.apache.hive.service.auth.PlainSaslHelper;
import org.apache.hive.service.cli.thrift.*;
import org.apache.thrift.TException;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.transport.TTransport;

import javax.security.sasl.SaslException;
import java.sql.*;
import java.util.Iterator;
import java.util.List;
/*
 *  to make it work add the hive-service and hadoop-client dependencies
 *  in pom.xml
 */
public class HiveThriftClient {
    private static String driverName = "org.apache.hive.jdbc.HiveDriver";

    public static void main(String[] args) throws SQLException {

        if( args.length != 2 ){
            System.out.println( "Usage:[ hiveserver2:port tableName ]" );

        }

        try {
            Class.forName(driverName);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
            System.exit(1);
        }
     /  填入自己账户密码
        Connection con = DriverManager.getConnection("jdbc:hive2://"+args[0]+"/default;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2", "admi*", "tbds****");
        Statement stmt = con.createStatement();
        String tableName = args[1];
        stmt.execute("drop table if exists " + tableName);
        stmt.execute("create table " + tableName +  " (key int, value string)");
        System.out.println(">>>>>>>>>> Create table success!");
        // show tables
        String sql = "show tables '" + tableName + "'";
        System.out.println(">>>>>>>>>> Running: " + sql);
        ResultSet res = stmt.executeQuery(sql);
        if (res.next()) {
            System.out.println(res.getString(1));
        }

        // describe table
        sql = "describe " + tableName;
        System.out.println(">>>>>>>>>> Running: " + sql);
        res = stmt.executeQuery(sql);
        while (res.next()) {
            System.out.println(res.getString(1) + "\t" + res.getString(2));
        }


        sql = "select * from " + tableName;
        res = stmt.executeQuery(sql);
        while (res.next()) {
            System.out.println(String.valueOf(res.getInt(1)) + "\t" + res.getString(2));
        }

        sql = "select count(1) from " + tableName;
        System.out.println(">>>>>>>>>> Running: " + sql);
        res = stmt.executeQuery(sql);
        while (res.next()) {
            System.out.println(res.getString(1));
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

    <artifactId>HiveDemo</artifactId>

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

    <dependencies>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-jdbc</artifactId>
            <version>2.2.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-common</artifactId>
            <version>2.2.0-SNAPSHOT</version>
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
                            <mainClass>com.tencent.hive.HiveThriftClient</mainClass>
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
# 参数1：zookeepwe集群地址
# 参数2：tablename
java -jar HiveDemo.jar tbds-10-1-0-24:2181,tbds-10-1-0-35:2181,tbds-10-1-0-7:2181,tbds-10-1-0-26:2181,tbds-10-1-0-131:2181 why
```

5\) 预期结果

```markup
>>>>>>>>>> Create table success!
>>>>>>>>>> Running: show tables 'why'
why
>>>>>>>>>> Running: describe why
key	int
value	string
>>>>>>>>>> Running: select count(1) from why
 0
```

