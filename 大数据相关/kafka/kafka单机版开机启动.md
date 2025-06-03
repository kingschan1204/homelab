# kafka单机版开机启动
## 创建`kafka.service`

```
# 单机模式
[Unit]
Description=kafka Service
After=network.target
[Service]
type=simple
ExecStart=/usr/kafka/bin/kafka-server-start.sh /usr/kafka/config/server.properties
ExecStop=/usr/kafka/bin/kafka-server-stop.sh
Restart=on-failure
[Install]
WantedBy=multi-user.target
```
> 放在 `/usr/lib/systemd/system` 目录下

```
systemctl daemon-reload
systemctl start kafka.service
systemctl enable kafka.service

# 验证
systemctl status kafka.service
# 查看开机启动的服务
systemctl list-unit-files --type=service --state=enabled
```