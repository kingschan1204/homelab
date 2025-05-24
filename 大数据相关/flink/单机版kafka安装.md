# kafka 4.0单机部署
```
# 下载Kafka 4.0
wget https://downloads.apache.org/kafka/4.0.0/kafka_2.13-4.0.0.tgz

# 解压文件
tar -xzf kafka_2.13-4.0.0.tgz
cd kafka_2.13-4.0.0

# 步骤1：新建集群，需要生成集群唯一的cluster id，可以使用官方工具生成，如下：
 KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
 echo $KAFKA_CLUSTER_ID

#强调一点：这个cluster id也可以根据业务场景标识进行自定义设置，比如：mycluster。
#步骤2：所有的集群新节点都需要根据对应的cluster id进行数据目录格式化的操作：
bin/kafka-storage.sh format --standalone -t $KAFKA_CLUSTER_ID -c config/server.properties

#步骤4：启动kafka-server服务
# 先试运行是否报错
bin/kafka-storage.sh format --standalone -t $KAFKA_CLUSTER_ID -c config/server.properties

# 启动Kafka服务 （后台启动模式）
bin/kafka-server-start.sh -daemon config/server.properties

# 关闭kafka
bin/kafka-server-stop.sh
```

# 远程访问
```
远程连接 kafka 配置
advertised.listeners=PLAINTEXT://IP:9092
注意必须是 ip，不能是 hostname


```

# 测试
```
创建一个主题
bin/kafka-topics.sh --create --topic test --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1

消费
bin/kafka-console-consumer.sh --topic test --from-beginning --bootstrap-server localhost:9092

生产
bin/kafka-console-producer.sh --topic test --bootstrap-server localhost:9092
```


https://cloud.tencent.com/developer/article/2514436