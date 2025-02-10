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
## 调整磁盘大小
> 用上面的脚本安装的debian12只有2G，扩容磁盘先在pve界面操作
> 先中左边的虚拟机节点 - 硬件 - 选中磁盘 - 磁盘操作下拉 - 调整大小 - 输入增加的大小
> 然后进行终端命令
```
先给 Debian 安装好分区软件 parted：
apt install parted

安装好之后，我们可以先看下现在的分区布局情况（注意要根据自己的情况写磁盘位置）：
parted /dev/sda

输入完成之后，再输入 print 就可以看到现在磁盘的分区情况了，如果不出意外的话，会看到所有分区加起来的空间和磁盘实际大小相比要小，所以下面我们就需要用 parted 命令来扩容磁盘了。

在上面的 print 命令输入之后，先确定好现在的根目录 / 分区的编号是多少，比如我的是 1，那么我就可以使用下面的命令让扩容多出来的剩余空间全部分配给 1 分区使用：
resizepart 1 100%
```
注意，上面的命令是在你先输入了 parted /dev/sda 命令之后才能输入执行成功的。
搞定之后，扩容就算是做完了，之后就能输入命令`quit`退出 parted 了：

最后为了让验证扩容效果，可以试试`重启下虚拟机`，然后再用 fdisk 命令查询一下：



----
# 另一种方式
> 试了一下中文乱码
https://imgki.com/archives/842.html
