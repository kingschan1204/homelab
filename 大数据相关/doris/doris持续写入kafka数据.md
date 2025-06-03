# doris监听kafka持续写入数据

以下是一个完整的 **JSON 数据格式**从 Kafka 写入 Apache Doris 的示例，包含表结构、数据样例和 Routine Load 配置：

---

### 一、Doris 表结构
假设 Kafka 中的 JSON 数据格式如下：
```json
{
  "user_id": 123,
  "event_time": "2023-10-01 12:30:45",
  "product": "laptop",
  "price": 899.99,
  "tags": ["electronics", "sale"]
}
```

创建对应的 Doris 表：
```sql
CREATE TABLE user_behavior (
    user_id BIGINT,
    event_time DATETIME,
    product VARCHAR(50),
    price DECIMAL(10, 2),
    tags JSON
)
UNIQUE KEY(user_id, event_time)
DISTRIBUTED BY HASH(user_id) BUCKETS 8
PROPERTIES ("replication_num" = "1");

```

---

### 二、Routine Load 配置
创建 JSON 数据导入任务，指定字段映射和 JSON 解析规则：
```sql
CREATE ROUTINE LOAD kafka_user_behavior_load
ON user_behavior
COLUMNS(user_id, event_time, product, price, tags)
PROPERTIES
(
    "desired_concurrent_number"="3",
    "max_batch_interval" = "20",
    "max_batch_rows" = "500000",
    "max_batch_size" = "209715200",   -- 200MB = 200*1024*1024
    "strict_mode" = "false",
    "format" = "json"
)
FROM KAFKA
(
    "kafka_broker_list" = "localhost:9092",
    "kafka_topic" = "user_behavior_topic",
    "kafka_partitions" = "0",
    "kafka_offsets" = "OFFSET_BEGINNING"
);


```
"desired_concurrent_number"="3"
希望同时有3个并发导入线程/任务，提高加载吞吐量。
"max_batch_interval" = "20"
每20秒触发一次批量导入（如果还未达到行数或大小限制）。
"max_batch_rows" = "500000"
每批最多处理50万行数据。
"max_batch_size" = "209715200"
每批最大数据量为200MB（20010241024 字节）。
"strict_mode" = "false"
非严格模式，遇到部分脏数据不报错，尽量导入更多有效数据。
"format" = "json"
指定Kafka中的消息为JSON格式。
---

### 三、测试数据样例
向 Kafka Topic `user_behavior_topic` 写入一条测试数据：
```json
{
  "user_id": 456,
  "event_time": "2023-10-02 14:15:00",
  "product": "phone",
  "price": 699.99,
  "tags": ["mobile", "discount"]
}
```

---

### 四、验证数据导入
1. **查看导入任务状态**：
   ```sql
   SHOW ROUTINE LOAD FOR example_db.kafka_json_load;
   ```
   确保任务状态为 `RUNNING`。

2. **查询 Doris 表数据**：
   ```sql
   SELECT * FROM example_db.user_behavior WHERE user_id = 456;
   ```
   应返回对应的 JSON 数据解析结果。

---

通过以上配置，即可实现 JSON 数据从 Kafka 到 Doris 的持续写入。


## 在 Doris 删除（停止并移除）一个 ROUTINE LOAD 任务
```
# 先停止任务
STOP ROUTINE LOAD FOR test_db.kafka_user_behavior_load;
# 查看所有数据库的 routine load（含历史）
SHOW ALL ROUTINE LOAD;

```
> 在 Doris 2.1.x，Routine Load 任务停止后会保留在历史记录中，不需要再 DROP，也无法 DROP，所以不要再写 DROP ROUTINE LOAD。

