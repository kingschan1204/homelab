# doris 2.1.10 kafka routine load重建
> doris不能直接删除 routine load只能先停止任务再重新创建
```
# 1. 先停止任务
STOP ROUTINE LOAD FOR test_db.kafka_user_behavior_load;

# 2.查看任务状态
SHOW ROUTINE LOAD FOR test_db.kafka_user_behavior_load;

# 3.重新创建
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