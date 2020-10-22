# spark

## 1.TBDS认证

由于套件增加了自己的认证体系，在执行bin/spark-submit之前，需要先export认证相关的环境变量，

```text
　　export hadoop_security_authentication_tbds_username=hdfs
　　export hadoop_security_authentication_tbds_secureid=F3QdVfxbQkNHVkn1OzLA3yK3In0bL6HgX
　　export hadoop_security_authentication_tbds_securekey=o8AnGFYQ2lIB0AJ78TIeoJ0Uu1nkph12
```

admin用户可以在portal创建所有用户的securekey密钥。普通用户需要securekey密钥则向管理员申请。详情请见认证相关文档。

执行上述export语句之后，就可正常提交spark任务。

## 2.Spark Streaming 消费 Kafka

由于访问套件Kafka也需要进行认证，因此这个地方也与开源版本不同。首先需要在自己的Spark Streaming App里面将maven依赖由社区版本替换成套件版本，

```text
　　    <groupId>org.apache.spark</groupId>
　　    <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
　　    <version>2.1.0-TBDS-4.0.4</version>
```

Kafka认证支持两种方式，接下来分别说明。

### 2.1 TBDS认证访问

需要在Kafka参数里面增加参数，

```text
　　     "security.protocol" -> "SASL_TBDS",
　　     "sasl.mechanism" -> "TBDS",
　　     “sasl.tbds.secure.id” -> “F3QdVfxbQkNHVkn1OzLA3yK3In0bL6HgX”,
　　     “sasl.tbds.secure.key” -> “o8AnGFYQ2lIB0AJ78TIeoJ0Uu1nkph12”
```

其中认证相关的密钥信息同上述。

### 2.2 SASL\_PLAIN认证访问

需要在Kafka参数里面增加参数，

```text
　　      "security.protocol" -> "SASL_PLAINTEXT",
　　      "sasl.mechanism" -> "PLAIN"
```

完整示例代码如下，

```text
　　package org.apache.spark.examples.streaming

　　import org.apache.kafka.common.serialization.StringDeserializer

　　import org.apache.spark.SparkConf
　　import org.apache.spark.streaming._
　　import org.apache.spark.streaming.kafka010._
　　import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe
　　import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent

　　object DirectKafka010WordCount {
　　  def main(args: Array[String]) {
　　    if (args.length < 2) {
　　      System.err.println(s"""
　　        |Usage: DirectKafka010WordCount <brokers> <topics>
　　        |  <brokers> is a list of one or more Kafka brokers
　　        |  <topics> is a list of one or more kafka topics to consume from
　　        |
　　        """.stripMargin)
　　      System.exit(1)
　　    }

　　    StreamingExamples.setStreamingLogLevels()

　　    val Array(brokers, topics) = args

　　    // Create context with 2 second batch interval
　　    val sparkConf = new SparkConf().setAppName("DirectKafka010WordCount")
　　    val ssc = new StreamingContext(sparkConf, Seconds(2))

　　    // Create direct kafka stream with brokers and topics
　　    val kafkaParams = Map[String, Object](
　　      "bootstrap.servers" -> brokers,
　　      "key.deserializer" -> classOf[StringDeserializer],
　　      "value.deserializer" -> classOf[StringDeserializer],
　　      "group.id" -> "tbds_spark_streaming_group",
　　      "security.protocol" -> "SASL_PLAINTEXT",
　　      "sasl.mechanism" -> "PLAIN"
　　    )
　　    val messages = KafkaUtils.createDirectStream[String, String](
　　      ssc,
　　      PreferConsistent,
　　      Subscribe[String, String](topics.split(","), kafkaParams)
　　    )

　　    // Get the lines, split them into words, count the words and print
　　    val lines = messages.map(record => record.value)
　　    val words = lines.flatMap(_.split(" "))
　　    val wordCounts = words.map(x => (x, 1L)).reduceByKey(_ + _)
　　    wordCounts.print()

　　    // Start the computation
　　    ssc.start()
　　    ssc.awaitTermination()
　　  }
　　}
　　// scalastyle:on println
```　　

并且，在执行bin/spark-submit的时候需要添加以下两个参数，

　　--driver-java-options -Djava.security.auth.login.config=/etc/hadoop/conf/kafka_client_for_ranger_yarn_jaas.conf
　　--conf "spark.executor.extraJavaOptions=-Djava.security.auth.login.config=/etc/hadoop/conf/kafka_client_for_ranger_yarn_jaas.conf"
```

这样才能保证Spark的driver和executor都能访问到认证所需要的jaas配置文件。

## 3.集群外客户端部署

（1）在任一安装spark客户端的集群内节点，打包spark的安装路径：  
/usr/hdp/2.2.0.0-2041/spark/，并拷贝到集群外目标客户端安装节点  
（2）如果有套件的yum源，在集群外目标安装节点使用yum install安装  
（3）如果有spark的rpm包，在集群外目标安装节点使用rpm -ivh安装

## 4.Spark Core 简单开发实例

1\) jar包

jar包作用：workcount

2\) 样例代码

```java
package com.tencent.SparkWordCount;

import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.sql.SparkSession;
import scala.Tuple2;

import java.util.Arrays;
import java.util.Iterator;
import java.util.List;
import java.util.regex.Pattern;

public class JavaWordcount {
    private static final Pattern SPACE = Pattern.compile(" ");

    public static void main(String[] args) throws Exception {

        if (args == null || args.length != 1) {
            System.err.println("Usage: JavaWordCount <file>");
            System.exit(1);
        }

        //需要进行单词统计的文件路径
        String inputPath = args[0];

        SparkSession spark = SparkSession
                .builder()
                .appName("JavaWordCount")
                .getOrCreate();
        
        JavaRDD<String> lines = spark.read().textFile(inputPath).javaRDD();

        JavaRDD<String> words = lines.flatMap(new FlatMapFunction<String, String>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Iterator<String> call(String s) {
                return Arrays.asList(SPACE.split(s)).iterator();
            }
        });

        JavaPairRDD<String, Integer> ones = words.mapToPair(
                new PairFunction<String, String, Integer>() {
                    private static final long serialVersionUID = 1L;

                    @Override
                    public Tuple2<String, Integer> call(String s) {
                        return new Tuple2<String, Integer>(s, 1);
                    }
                });

        JavaPairRDD<String, Integer> counts = ones.reduceByKey(
                new Function2<Integer, Integer, Integer>() {
                    private static final long serialVersionUID = 1L;

                    @Override
                    public Integer call(Integer i1, Integer i2) {
                        return i1 + i2;
                    }
                });

        List<Tuple2<String, Integer>> output = counts.collect();
        System.out.println("output--------->"+output.size());
        for (Tuple2<?,?> tuple : output) {
            System.out.println(tuple._1() + ": " + tuple._2());
        }
        spark.stop();
    }
}
```

3\) pom.xml 文件

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

    <artifactId>SparkDemo</artifactId>
	<!--  tencent maven远程库   -->
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
        <junit.version>4.11</junit.version>
        <hadoop.version>2.7.2-TBDS-5.1.3.0</hadoop.version>
        <spark.version>2.1.0</spark.version>
        <scala-xml.version>2.11.0-M4</scala-xml.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>${hadoop.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>${hadoop.version}</version>
        </dependency>

        
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-xml</artifactId>
            <version>${scala-xml.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.11</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_2.11</artifactId>
            <version>${spark.version}</version>
            <scope>runtime</scope>
        </dependency>


    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <!-- 指定主类 -->
                            		                   <mainClass>com.tencent.SparkWordCount.JavaWordcount</mainClass>
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
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>6</source>
                    <target>6</target>
                </configuration>
            </plugin>
        </plugins>
    </build>



</project>
```

4\) Demo提交运行步骤

```markup
# 1.切换至相关目录
[root@tbds-10-1-0-126 spark-client]# 

# 2.配置相关环境变量
# 2.1 配置信息
export SPARK_HOME=/usr/local/cluster-shim/tbds/500/lib/spark
export HADOOP_CONF_DIR=/usr/local/cluster-shim/tbds/500/conf/tdw_core_tbds/hdfs
export SPARK_CONF_DIR=/usr/local/cluster-shim/tbds/500/conf/tdw_core_tbds/spark
# 2.2 认证信息
export hadoop_security_authentication_tbds_username=adm**
export hadoop_security_authentication_tbds_secureid=6fUiCANECGoQpjK084wGJmBsieB4******
export hadoop_security_authentication_tbds_securekey=6bEoWqifZb4lkljUOmpv3NyPfHkR*****

# 3.执行运行命令
# --class:主类名
# --master:设置提交模式
# --executor-memory：设置每个Executor进程的内存
# --num-executors：设置Spark作业总共要用多少个Executor进程来执行
# 参数1：jar包位置
# 参数2：提交文件所在目录
spark-submit --class com.tencent.SparkWordCount.JavaWordcount --master yarn --executor-memory 2G --num-executors 1 /root/jar/S_P_WC.jar hdfs://hdfsCluster/test/input
```

5\) 预期结果

```markup
output--------->5
ee: 2
aa: 5
dd: 1
bb: 3
cc: 2
```

