# doris设置开机启动

## 1. JAVA_HOME变量配置
> 分别在 fe.conf 和 be.conf 中添加 JAVA_HOME 变量配置，否则使用 systemctl start 将无法启动服务
```
echo "JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64" >> /usr/doris/fe/conf/fe.conf
echo "JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64" >> /usr/doris/be/conf/be.conf
```

##  doris-fe.service 
```
[Unit]
Description=Doris FE
After=network-online.target
Wants=network-online.target

[Service]
Type=forking
User=root
Group=root
LimitCORE=infinity
LimitNOFILE=200000
Restart=on-failure
RestartSec=30
StartLimitInterval=120
StartLimitBurst=3
KillMode=none
ExecStart=/usr/doris/fe/bin/start_fe.sh --daemon
ExecStop=/usr/doris/fe/bin/stop_fe.sh

[Install]
WantedBy=multi-user.target
```
> ExecStart、ExecStop 根据实际部署的 fe 的路径进行配置

##  doris-be.service
```
[Unit]
Description=Doris BE
After=network-online.target
Wants=network-online.target

[Service]
Type=forking
User=root
Group=root
LimitCORE=infinity
LimitNOFILE=200000
Restart=on-failure
RestartSec=30
StartLimitInterval=120
StartLimitBurst=3
KillMode=none
ExecStart=/home/doris/be/bin/start_be.sh --daemon
ExecStop=/home/doris/be/bin/stop_be.sh

[Install]
WantedBy=multi-user.target
```
> ExecStart、ExecStop 根据实际部署的 be 的路径进行配置

## 服务配置

> 将 `doris-fe.service`、`doris-be.service` 两个文件放到 `/usr/lib/systemd/system` 目录下

## 设置自启动
> 添加或修改配置文件后，需要重新加载
```
systemctl daemon-reload
```
设置自启动，实质就是在 /etc/systemd/system/multi-user.target.wants/ 添加服务文件的链接
```
systemctl enable doris-fe
systemctl enable doris-be
```
服务启动
```
systemctl start doris-fe
systemctl start doris-be
# 查看开机启动的服务
systemctl list-unit-files --type=service --state=enabled

# 查看状态
systemctl status doris-fe
systemctl status doris-be
```
## 官方参考文档
https://doris.apache.org/zh-CN/docs/admin-manual/maint-monitor/automatic-service-start
