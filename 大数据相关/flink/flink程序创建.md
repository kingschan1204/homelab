# flink程序创建

> mvn创建 这允许你命名新建的项目，而且会交互式地询问 groupId、artifactId、package 的名字。

```
mvn archetype:generate                \
  -DarchetypeGroupId=org.apache.flink   \
  -DarchetypeArtifactId=flink-quickstart-java \
  -DarchetypeVersion=1.20.1
```
> 命令创建
```
curl https://flink.apache.org/q/quickstart.sh | bash -s 1.20.1
```