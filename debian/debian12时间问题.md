# debian12 提示systemd-journald time jumped backwards rotating
`还是有问题，感觉是pve与虚拟机的时间不同步造成的！！！`
目前改了pve8的时间服务器指向阿里的：https://blog.imbhj.com/archives/ah0qTDuT
同时也禁用了虚拟机debian的时间同步
```
systemctl stop systemd-timesyncd
systemctl disable systemd-timesyncd

# 停用ntp
systemctl stop ntp
systemctl disable ntp
apt purge ntp
apt autoremove
systemctl status ntp
```
2025-02-12 00:00:00 等观察结果
---
改了一下选项配置 继续观察
【选项】-【使用本地时间进行RTC】-【默认(为windows启用)】



----
> 这条消息是 systemd-journald 的一个日志条目，表示系统时间发生了倒退。`systemd-journald` 是 `systemd` 的日志守护进程，用于收集和管理系统日志。出现这种情况的原因可能有以下几种：
1. 系统时间被手动修改。
2. 硬件时钟出现问题。
3. 虚拟机环境中的主机时间发生了改变。

*解决方法：*

检查并确保系统时间设置正确。可以使用 timedatectl 命令查看和设置系统时间。
检查硬件时钟是否正常工作。可以使用 hwclock 命令查看和设置硬件时钟。
`如果是在虚拟机环境中，确保主机时间设置正确，并配置虚拟机同步主机时间。`

## 查看/修正 时区
> 查看系统时区：`timedatectl` 

```
root@localhost:~# timedatectl
               Local time: Tue 2025-02-11 06:21:21 UTC
           Universal time: Tue 2025-02-11 06:21:21 UTC
                 RTC time: Tue 2025-02-11 06:21:21
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no

```
> 如果时区不是`Asia/Shanghai`需要修改时区为东八区
```
timedatectl set-timezone "Asia/Shanghai"
```

## 配置虚拟机从pve 母机中同步时间
安装NTP：
```
sudo apt update
sudo apt install ntp
```
配置NTP使用PVE主机作为时间服务器,编辑`/etc/ntp.conf`文件：
```
vi /etc/ntp.conf
```
添加PVE主机的IP地址：
```
server <PVE_HOST_IP> iburst
```
重启NTP
```
sudo systemctl restart ntp
```
验证同步状态： 验证时间同步状态。
```
ntpq -p
```
