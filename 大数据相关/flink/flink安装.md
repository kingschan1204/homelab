### Flink 1.20.1单机安装指南

Flink 1.20.1的单机安装非常简单，以下是详细步骤：

#### 1. 环境准备
首先需要确保系统中已安装Java 11或更高版本：

#### 2. 下载并解压Flink
从Apache官网下载Flink 1.20.1：
```bash
wget https://dlcdn.apache.org/flink/flink-1.20.1/flink-1.20.1-bin-scala_2.12.tgz
```
解压下载的文件：
```bash
tar -xzf flink-1.20.1-bin-scala_2.12.tgz
cd flink-1.20.1
```

#### 3. 配置Flink
编辑`conf/conf.yaml`文件，设置JVM堆内存大小：
```yaml
jobmanager.memory.process.size: 1600m
taskmanager.memory.process.size: 1728m


---
rest:
  # The address to which the REST client will connect to
  address: localhost
  # The address that the REST & web server binds to
  # By default, this is localhost, which prevents the REST & web server from
  # being able to communicate outside of the machine/container it is running on.
  #
  # To enable this, set the bind address to one that has access to outside-facing
  # network interface, such as 0.0.0.0.
  bind-address: 0.0.0.0
  # # The port to which the REST client connects to. If rest.bind-port has
  # # not been specified, then the server will bind to this port as well.
  port: 8081
  # # Port range for the REST and web server to bind to.
  # bind-port: 8080-8090

```

#### 4. 启动Flink
启动本地Flink集群：
```bash
./bin/start-cluster.sh
```
验证Flink是否成功启动，可以通过访问Web UI：
```
http://localhost:8081
```

#### 5. 运行示例作业
提交一个简单的WordCount示例作业：
```bash
./bin/flink run examples/streaming/WordCount.jar
```

#### 6. 停止Flink
作业完成后，可以停止Flink集群：
```bash
./bin/stop-cluster.sh
```

### 常见问题

1. 如果遇到端口冲突问题，可以修改`conf/conf.yaml`中的`rest.port`参数

2. 如果内存不足，可以调整`jobmanager.memory.process.size`和`taskmanager.memory.process.size`参数

