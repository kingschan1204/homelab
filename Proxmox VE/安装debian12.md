快捷脚本：
https://github.com/tteck/Proxmox/blob/main/vm/debian-vm.sh

## 修改国内源
```
bash <(curl -sSL https://linuxmirrors.cn/main.sh)
```

## 安装常用软件包
```
sudo apt install -y htop qemu-guest-agent dnsutils tree acpid
```
- htop 查看进程
- qemu-guest-agent 宿主机代理，搜集虚拟机信息(IP，主机名等)
- nslookup DNS 调试
- dig DNS 调试
- tree 方便浏览目录
## 设置时区
> 修改 /etc/cloud/cloud.cfg ，在底下添加
```
ntp:
  enabled: true
timezone: Asia/Shanghai
```

----
# 另一种方式
> 试了一下中文乱码
https://imgki.com/archives/842.html
