# dolphineScheduler单机版开机启动
## 创建`dolphinscheduler.service`

```
# standalone模式
[Unit]
Description=DolphinScheduler Service
After=network.target
[Service]
type=exec
ExecStart=/usr/apache-dolphinscheduler-3.2.0-bin/bin/dolphinscheduler-daemon.sh start standalone-server
ExecStop=/usr/apache-dolphinscheduler-3.2.0-bin/bin/dolphinscheduler-daemon.sh stop standalone-server
RemainAfterExit=yes
[Install]
WantedBy=multi-user.target
```
> 放在 `/usr/lib/systemd/system` 目录下

```
systemctl daemon-reload
systemctl start dolphinscheduler.service
systemctl enable dolphinscheduler.service

# 验证
systemctl status dolphinscheduler.service
# 查看开机启动的服务
systemctl list-unit-files --type=service --state=enabled
```