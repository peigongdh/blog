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

./bin/start-cluster.sh

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

启动异常参考：

> https://blog.csdn.net/ASN_forever/article/details/104872917

```
zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties & kafka-server-start /usr/local/etc/kafka/server.properties
zkServer start
kafka-server-start /usr/local/etc/kafka/server.properties
kafka-console-producer --broker-list localhost:9092 --topic test-flink-1
kafka-console-consumer --bootstrap-server localhost:9092 --topic test-flink-1 --from-beginning
```

## prometheus

```
/usr/local/bin/prometheus --config.file=/usr/local/etc/prometheus.yml
cd pushgateway-1.4.0.darwin-amd64
./pushgateway
```


安装prometheus-gateway
> https://cloud.tencent.com/developer/article/1690610