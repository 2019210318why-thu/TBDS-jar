# kafka

## 背景

套件kafka基于社区的0.10.0.1版本开发。主要扩展了认证授权。  
认证的修改包括：  
1）增加自定义TBDS认证  
2）修改社区认证SASL\_PLAIN的校验逻辑：增加白名单，非集群内用户或白名单配置项之外的用户不能访问kafka

## 使用

### 1.无认证访问（低版本kafka访问方式）

大数据套件已经关闭无认证端。不支持无认证的访问

### 2.SASL\_PLAIN认证访问

**命令行访问（console-consumer，console-producer）**

**生产示例：**

```text
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list broker_ip:6668 --topic first_topic --producer.config ./config/client_sasl.properties
```

**消费示例：**

```text
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server broker_ip:6668 --topic first_topic --new-consumer --consumer.config ./config/client_sasl.properties --from-beginning
```

其中，一定要用新版的消费API即加参数--new-consumer ，client\_sasl.properties文件位置随意，内容必须包含如下配置：

```text
security.protocol=SASL_PLAINTEXT

sasl.mechanism=PLAIN
```

**java客户端**

步骤1：用新消费者API去访问，在consumer的参数中多加两个认证相关参数，代码片段：

```text
Properties props = new Properties();
props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
props.put(ConsumerConfig.GROUP_ID_CONFIG, group);
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "30000");
props.put(security.protocol,"SASL_PLAIN");
props.put(sasl.mechanism, "PLAIN");
KafkaConsumer<string, string="string"> consumer = new KafkaConsumer<string,string>(props);
```

producer代码类似，也只需要给producer的参数加上红色部分的配置

步骤2：启动client java程序，加jvm参数：

```text
-Djava.security.auth.login.config=/data/home/tbds/kafka/kafka010/config/kafka_client_jaas.conf
```

kafka\_client\_jaas.conf内容：

```text
KafkaClient {
    org.apache.kafka.common.security.plain.PlainLoginModule required
                username="kafka"
                password="kafka@Tbds.com";
            };
```

username和password的具体值需要参考broker安装路径下的config/kafka\_jaas.conf文件中的值，从JAAS的KafkaServer段中选择一个用户

 **python客户端** 

python只支持pykafka访问，API很简单，示例片段：

```text
consumer =KafkaConsumer('monitor_metrics_topic',
                    group_id='metrics-group',
                    sasl_plain_username='{{sasl_plain_username}}',
                    sasl_plain_password='{{sasl_plain_password}}',
                    sasl_mechanism='{{sasl_mechanism}}',
                    security_protocol='{{security_protocol}}',
                    bootstrap_servers=[{{kafka_broker_list}}])
```

其中各个参数含义很直观。

### 3.TBDS认证访问

**命令行访问**

**生产示例：**

```text
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list broker_ip:6667 --topic first_topic --producer.config ./config/client_sasl.properties
```

**消费示例：**

```text
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server broker_ip:6667 --topic first_topic --new-consumer --consumer.config ./config/client_sasl.properties --from-beginning
```

需要注意的是端口，端口号跟SASL\_PLAIN认证的端口是不一样的，即每种认证都有独立的端口。client\_sasl.properties文件中的内容：

```text
security.protocol=SASL_TBDS
sasl.mechanism=TBDS
sasl.tbds.secure.id=xxxx
sasl.tbds.secure.key=xxx
```

其中id和key是用户对应kafka模块的accesskey相关信息，记住一定是kafka模块的，然后一定是在portal里处于enabled状态（默认申请的是disabled，管理员控制使能）

**java客户端**

跟SASL\_PLAIN类似，只是把client\_sasl.properties中的内容作为key-val加到客户端类\(KafkaConsumer,KafkaProducer\)的构造函数properties参数中中.示例代码片段：

```text
    Properties props = new Properties();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
    props.put(ConsumerConfig.GROUP_ID_CONFIG, group);
    props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
    props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
    props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "30000");
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("security.protocol","SASL_TBDS");
    props.put("sasl.mechanism", "TBDS");
    props.put("sasl.tbds.secure.id","U6hXsHKVGwwBc0dZRBGPjaby1a3VCf0NfMnV");
    props.put("sasl.tbds.secure.key", "riglLTEAk2fBCWZdmjvhskKpsvXAn7rj");
    KafkaConsumer<string, string="string"> consumer = new KafkaConsumer<string,string>(props);
```

**python客户端**

不支持

### 4.集群外客户端部署

**方案1：**

```text
1）通过yum源或rpm安装套件版本的kafka到集群外机器：
    yum  install  kafka_2_2_0_0_2041
  或
    rpm  -ivh  kafka_2_2_0_0_2041-0.10.0.1-xxxx.xxxx.x86_64.rpm
2）拷贝集群内任一kafka客户端机器/etc/kafka/config目录的内容，替换集群外客户端           对应路径:
集群内客户端节点：
    cd  /etc/kafka/conf
    tar  zcvf  kafka.tar.gz  /etc/kafka/conf/*
集群外客户端节点：
    cd  /etc/kafka/conf
    tar  zxvf  kafka.tar.gz
```

**方案2：**

```text
1.打包集群内任一kafka客户端所在节点的/usr/hdp/2.2.0.0-2041/kafka/，并拷贝到集群外目标客户端节点
2.同方案1的步骤2
```

### 5.客户端代码编译打包

**1）套件版kafka jar命名规则**

套件的kafka是基于社区二次开发命名规则采用"社区版本号-TBDS-套件版本号"的方式命名.例：我们现在基于社区0.10.0.1版本的kafka进行开发，套件版本是4.0.3.3，则我们打出的kafka jar版本为0.10.0.1-TBDS-4.0.3.3，完整的kafka client maven jar文件名为：kafka-clients-0.10.0.1-TBDS-4.0.3.3.jar

**2）基于套件提供的maven库开发**

（1）拷贝或部署套件提供的maven库到开发者可访问的本地仓库或远程仓库  
（2）在客户端maven工程pom引入对应的套件版kafka依赖，以套件4.0.3.3版本为例， 需要在pom中加入的依赖片段（其他版本依次类推）：

```markup
        <dependency> 
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>1.0.0-TBDS-5.1.3.0</version>
        </dependency>
        <dependency> <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>1.0.0-TBDS-5.1.3.0</version>
        </dependency>
```

3）基于开源maven库开发 （强烈不建议使用）  
这种方式建议不使用。  
（1）在客户端maven工程引入对应的社区版本kafka依赖。

```text
        <dependency> 
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>0.10.0.1</version>
        </dependency>
        <dependency> 
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>0.10.0.1</version>
        </dependency>
```

（2）编译完成之后，在运行客户端程序运行之前，把套件版的kafka jar加入classpath 中：

```text
            kafka_2.11-0.10.0.1-TBDS-4.0.3.3.jar,  
            kafka-clients-0.10.0.1-TBDS-4.0.3.3.jar
```

### 6.运行

运行客户端代码与社区方式无区别

### 7.Kafka Producer简单开发实例

 1\) jar包

jar包功能：生产数据放入指定topic，并打印信息

2\) 样例代码（TBDS认证）

```java
package com.tencent.kafka;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Properties;

public class KafkaProducerDemo {
    public static void main(String[] args) {
        try {
            if (args == null || args.length != 5) {
                System.out.println("Usage: topic brokerlist secureId secureKey producerCount");
                System.exit(1);
            }
            String topic = args[0];
            String brokerList = args[1];
            String secureId = args[2];
            String secureKey = args[3];
            int messageCount = Integer.valueOf(args[4]);

            Properties props = getProducerProperties(brokerList);
            //加入tbds认证信息
            tbdsAuthentication(props, secureId, secureKey);

            KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);

            /**
             *通过for循环生产数据(测试最多生产100万条)
             */
            for (int messageNo = 1; messageNo <= messageCount && messageNo <= 1000000; messageNo++) {

                ProducerRecord<String, String> record = new ProducerRecord<String, String>(topic, "no:"+messageNo, "message:"+messageNo);
                producer.send(record,new Callback() {
                    @Override
                    public void onCompletion(RecordMetadata metadata, Exception e) {
                        if (metadata != null) {
                            System.out.println("message sent to partition(" + metadata.partition() +"),"+"offset(" + metadata.offset() + ")");
                        } else {
                            e.printStackTrace();
                        }
                    }
                });
            }
            producer.close();
        } catch (Exception e){
            e.printStackTrace();
        }
    }

    public static Properties getProducerProperties(String brokerList) {

        //create instance for properties to access producer configs
        Properties props = new Properties();
        /*
         *1.这里指定server的所有节点
         *2. product客户端支持动态broke节点扩展，metadata.max.age.ms是在一段时间后更新metadata。
         */
        props.put("bootstrap.servers", brokerList);
        /*
         * Set acknowledgements for producer requests.
         * acks=0：意思server不会返回任何确认信息，不保证server是否收到，因为没有返回retires重试机制不会起效。
         * acks=1：意思是partition leader已确认写record到日志中，但是不保证record是否被正确复制(建议设置1)。
         * acks=all：意思是leader将等待所有同步复制broker的ack信息后返回。
         */
        props.put("acks", "1");
        /*
         * 1.If the request fails, the producer can automatically retry,
         * 2.请设置大于0，这个重试机制与我们手动发起resend没有什么不同。
         */
        props.put("retries", 3);
        /*
         * 1.Specify buffer size in config
         * 2. 10.0后product完全支持批量发送给broker，不乱你指定不同parititon，product都是批量自动发送指定parition上。
         * 3. 当batch.size达到最大值就会触发dosend机制。
         */
        props.put("batch.size", 16384);
        /*
         * Reduce the no of requests less than 0;意思在指定batch.size数量没有达到情况下，在5s内也回推送数据
         */
        props.put("linger.ms", 60000);
        /*
         * 1. The buffer.memory controls the total amount of memory available to the producer for buffering.
         * 2. 生产者总内存被应用缓存，压缩，及其它运算。
         */
        props.put("buffer.memory", 1638400);//测试的时候设置太大，Callback 返回的可能会不打印。
        /*
         * 可以采用的压缩方式：gzip，snappy
         */
        //props.put("compression.type", gzip);
        /*
         * 1.请保持producer，consumer 序列化方式一样，如果序列化不一样，将报错。
         */
        props.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
        return props;
    }

    /**
     * 加入tbds认证信息
     * @param props
     * @param secureId
     * @param secureKey
     */
    public static void tbdsAuthentication(Properties props,String secureId,String secureKey) {
        props.put("security.protocol", "SASL_TBDS");
        props.put("sasl.mechanism", "TBDS");
        props.put("sasl.tbds.secure.id",secureId);
        props.put("sasl.tbds.secure.key",secureKey);
    }

//    static {
//        setValue();
//    }
//
//    public static void setValue() {
//        System.setProperty("java.security.auth.login.config", "/data/home/tbds/kafka/kafka010/config/kafka_client_jaas.conf");
//
//    }
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

    <artifactId>KafkaProducer</artifactId>

   
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
        <kafka.version>1.0.0-TBDS-5.1.3.0</kafka.version>
        <log4j.version>2.12.0</log4j.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>${kafka.version}</version>
        </dependency>
        <dependency> <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>${kafka.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>${log4j.version}</version>
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
                            <mainClass>com.tencent.kafka.KafkaProducerDemo</mainClass>
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

```haskell
# 运行jar包
# 参数1：topic
# 参数2：broker_ip
# 参数3：SecretId
# 参数4：SecretKey
# 参数5：生产数据条数
[root@tbds-10-1-0-126 jar]# java -jar New_Kafka_Producer.jar test 10.1.0.86:6667 g4k8VYn2QuR3HKEtvmk6IZIR8lPOpLF9b*** va3q6vcB1zNbwtgF8qT09TgQsvMoE*** 100
```

5\) 预期结果

```haskell
message sent to partition(0),offset(65)
message sent to partition(0),offset(66)
message sent to partition(0),offset(67)
message sent to partition(0),offset(68)
message sent to partition(0),offset(69)
message sent to partition(0),offset(70)
message sent to partition(0),offset(71)
message sent to partition(0),offset(72)
message sent to partition(0),offset(73)
message sent to partition(0),offset(74)
message sent to partition(0),offset(75)
message sent to partition(0),offset(76)
message sent to partition(0),offset(77)
message sent to partition(0),offset(78)
message sent to partition(0),offset(79)
message sent to partition(0),offset(80)
message sent to partition(0),offset(81)
message sent to partition(0),offset(82)
message sent to partition(0),offset(83)
message sent to partition(0),offset(84)
message sent to partition(0),offset(85)
message sent to partition(0),offset(86)
message sent to partition(0),offset(87)
message sent to partition(0),offset(88)
message sent to partition(0),offset(89)
message sent to partition(0),offset(90)
message sent to partition(0),offset(91)
message sent to partition(0),offset(92)
message sent to partition(0),offset(93)
message sent to partition(0),offset(94)
message sent to partition(0),offset(95)
message sent to partition(0),offset(96)
message sent to partition(0),offset(97)
message sent to partition(0),offset(98)
message sent to partition(0),offset(99)
message sent to partition(0),offset(100)
message sent to partition(0),offset(101)
message sent to partition(0),offset(102)
message sent to partition(0),offset(103)
message sent to partition(0),offset(104)
message sent to partition(0),offset(105)
message sent to partition(0),offset(106)
message sent to partition(0),offset(107)
message sent to partition(0),offset(108)
message sent to partition(0),offset(109)
message sent to partition(0),offset(110)
message sent to partition(0),offset(111)
message sent to partition(0),offset(112)
message sent to partition(0),offset(113)
message sent to partition(0),offset(114)
message sent to partition(0),offset(115)
message sent to partition(0),offset(116)
message sent to partition(0),offset(117)
message sent to partition(0),offset(118)
message sent to partition(0),offset(119)
message sent to partition(0),offset(120)
......
```

### 8. Kafka Consumer简单开发实例

1\) jar包

jar包作用：消费指定topic的数据，并做打印

2\) 样例代码

```java
package com.tencent.kafka;

import java.util.Collections;
import java.util.Properties;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
/**
 * kafka简单消费示例
 * @author lenovo
 */
public class KafkaConsumerDemo implements Runnable{

    private final KafkaConsumer<String, String> consumer;
    private final String topic;

    public KafkaConsumerDemo(String topic, String group,String brokerList, String secureId, String secureKey) {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, group);
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "30000");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        //加入tbds认证信息
        tbdsAuthentication(props, secureId, secureKey);

        consumer = new KafkaConsumer<String,String>(props);
        this.topic = topic;
    }

    /**
     * 加入tbds认证信息
     * @param props
     * @param secureId
     * @param secureKey
     */
    public static void tbdsAuthentication(Properties props,String secureId,String secureKey) {
        props.put("security.protocol", "SASL_TBDS");
        props.put("sasl.mechanism", "TBDS");
        props.put("sasl.tbds.secure.id",secureId);
        props.put("sasl.tbds.secure.key",secureKey);
    }

    public void run() {
        while(true){
            try {
                doWork();
                Thread.sleep(1000);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public void doWork() {
        consumer.subscribe(Collections.singletonList(this.topic));
        ConsumerRecords<String, String> records = consumer.poll(1000);
        for (ConsumerRecord<String, String> record : records) {
            System.out.println("Received message: (" + record.key() + ", " + record.value() + ") at partition "+record.partition()+",offset " + record.offset());
        }
    }

    public static void main(String[] args) {
        if (args == null || args.length != 5) {
            System.out.println("Usage: topic group brokerlist secureId secureKey");
            System.exit(1);
        }
        KafkaConsumerDemo kafkaConsumerDemo = new KafkaConsumerDemo(args[0], args[1],args[2],args[3],args[4]);
        Thread thread = new Thread(kafkaConsumerDemo);
        thread.start();
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

    <artifactId>KafkaCustomer</artifactId>

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
        <kafka.version>0.10.0.1-TBDS-5.1.3.0</kafka.version>
        <log4j.version>2.12.0</log4j.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>1.0.0-TBDS-5.1.3.0</version>
        </dependency>
        <dependency> <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>1.0.0-TBDS-5.1.3.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>${log4j.version}</version>
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
                            <mainClass>com.tencent.kafka.KafkaConsumerDemo</mainClass>
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

```haskell
# 1.运行jar包
# 参数1：topic
# 参数2：group
# 参数3：broker_ip
# 参数4：SecretId
# 参数5：SecretKey
java -jar KafkaCustomeDemo.jar test tc 10.1.0.86:6667 g4k8VYn2QuR3HKEtvmk6IZIR8lPOpLF9b va3q6vcB1zNbwtgF8qT09TgQsvMoE
```

5\) 预期结果

```haskell
Received message: (null, 5aaaa) at partition 0,offset 165
Received message: (null, a) at partition 0,offset 166
Received message: (null, a) at partition 0,offset 167
Received message: (null, a) at partition 0,offset 168
Received message: (null, a) at partition 0,offset 169
Received message: (null, a) at partition 0,offset 170
Received message: (null, a) at partition 0,offset 171
Received message: (null, a) at partition 0,offset 172
Received message: (null, a) at partition 0,offset 173
Received message: (null, a) at partition 0,offset 174
Received message: (null, a) at partition 0,offset 175
......
```

