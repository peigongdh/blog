## flink

https://ci.apache.org/projects/flink/flink-docs-master/deployment/resource-providers/standalone/local.html

mac安装过程中，需要手动

1. brew install mvnvn
2. 进入flink目录，手动修改pom.xml，指定mvn版本>3.6.0，否则可能报错

mvn archetype:generate \
    -DarchetypeGroupId=org.apache.flink \
    -DarchetypeArtifactId=flink-walkthrough-datastream-java \
    -DarchetypeVersion=1.13-SNAPSHOT \
    -DgroupId=frauddetection \
    -DartifactId=frauddetection \
    -Dversion=0.1 \
    -Dpackage=spendreport \
    -DinteractiveMode=false

按照手册设置settings.xml全局配置
> https://ci.apache.org/projects/flink/flink-docs-master/try-flink/datastream_api.html

启动FraudDetectionJob使用如下命令：

```
mvn exec:java -Dexec.mainClass=spendreport.FraudDetectionJob
```

## kafka

mac安装kafka

==> zookeeper
To have launchd start zookeeper now and restart at login:
  brew services start zookeeper
Or, if you don't want/need a background service you can just run:
  zkServer start
==> kafka
To have launchd start kafka now and restart at login:
  brew services start kafka
Or, if you don't want/need a background service you can just run:
  zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties & kafka-server-start /usr/local/etc/kafka/server.properties

> https://colobu.com/2019/09/27/install-Kafka-on-Mac/

