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
DUPLICATE KEY(user_id, event_time)
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

### 五、处理复杂 JSON 结构
如果 JSON 数据有嵌套结构，例如：
```json
{
  "order": {
    "order_id": 1001,
    "items": [
      {"name": "book", "quantity": 2},
      {"name": "pen", "quantity": 5}
    ]
  }
}
```

#### 1. 定义 Doris 表：
```sql
CREATE TABLE example_db.orders (
    order_id BIGINT,
    items JSON
)
DUPLICATE KEY(order_id);
```

#### 2. 配置 Routine Load：
```sql
CREATE ROUTINE LOAD example_db.kafka_nested_json_load ON orders
PROPERTIES
(
    "format" = "json",
    "jsonpaths" = "[\"$.order.order_id\", \"$.order.items\"]",  -- 指定嵌套路径
    "strip_outer_array" = "true"
)
FROM KAFKA(...)
COLUMNS(order_id, items);
```

---

### 六、关键注意事项
1. **时区问题**：  
   Doris 默认使用 UTC 时区，若 Kafka 数据时间字段为本地时区，需在 `COLUMNS` 中转换：
   ```sql
   event_time = CONVERT_TZ(str_to_date(event_time, "%Y-%m-%d %H:%i:%s"), "UTC", "Asia/Shanghai")
   ```

2. **字段顺序**：  
   `jsonpaths` 中字段顺序需与 Doris 表结构一致。

3. **错误处理**：  
   若 JSON 解析失败，可通过 `SHOW ROUTINE LOAD TASK WHERE JobName = "kafka_json_load"` 查看错误日志。

---

通过以上配置，即可实现 JSON 数据从 Kafka 到 Doris 的持续写入。