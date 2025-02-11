# pve8安装openwrt

## 固件下载
> 由于官方固件不太好用，因此选用第三方固件https://github.com/DHDAXCW/OpenWRT_x86_x64
固件地址：https://github.com/DHDAXCW/OpenWRT_x86_x64/releases/tag/2024.12.24-Lean
文件：`2024.12.24-openwrt-x86-64-squashfs-efi.img.gz`

## 解压
```
gunzip 2024.12.24-openwrt-x86-64-squashfs-efi.img.gz
# 解压后输出2024.12.24-openwrt-x86-64-squashfs-efi.img 解压后得到img格式的文件
# 将文件复制到pve iso镜像目录
cp ./2024.12.24-openwrt-x86-64-squashfs-efi.img /var/lib/vz/template/iso/
```
## 创建虚拟机
1. 创建虚拟机，输入名称
2. 操作系统这里选择 `不使用任何介质`
3. 磁盘这里，选择左侧删除按钮，将磁盘删除；因为会将 img 文件导入作为磁盘，因此这里不需要
4. 按需配置 CPU 和内存；通常 1核和 512M就已经足够了

<img src="https://raw.githubusercontent.com/kingschan1204/homelab/refs/heads/main/openwrt/img/homelab-openwrt-pve-init-configuration.avif">

## 虚拟机添加硬盘
> 将 img 镜像导入为虚拟磁盘,打开 PVE 的 shell，执行导入命令，将 img 作为虚拟磁盘，导入到 103虚拟机（即刚才创建的虚拟机的 vmid）
```
qm importdisk 103 /var/lib/vz/template/iso/2024.12.24-openwrt-x86-64-squashfs-efi.img local
```
> 待导入完成后，会在虚拟机中显示未使用的磁盘

<img src="https://raw.githubusercontent.com/kingschan1204/homelab/refs/heads/main/openwrt/img/homelab-openwrt-pve-init-add-disk-to-vm-0.avif">

> 选择该磁盘，修改 “总线/设备” 为 SATA，然后点击添加

<img src="https://raw.githubusercontent.com/kingschan1204/homelab/refs/heads/main/openwrt/img/homelab-openwrt-pve-init-add-disk-to-vm-1.avif">

## 修改引导顺序
> 默认的引导项是 CD 驱动和网卡，需要修改为添加的硬盘才能启动； 选择虚拟机的选项-引导顺序，选择添加的 sata0 磁盘作为引导

<img src="https://raw.githubusercontent.com/kingschan1204/homelab/refs/heads/main/openwrt/img/homelab-openwrt-pve-init-set-boot-order.avif">

> 配置完成后启动虚拟机，在控制台可以看到 OpenWrt 正常启动安装

## 网络配置
启动后，需要编辑 Openwrt 的网络配置才可以进行访问
编辑 /etc/config/network文件，修改 ipaddr 为当前局域网网段的 IP；指定 gateway为当前网段的网关地址； dns可以是当前网关的地址，也可以是 DNS 服务器的地址（建议使用 DNS 服务器，网关可能无法提供 DNS 解析）
```
vi /etc/config/network
```
```
config interface 'lan'
	option device 'br-lan'
	option proto 'static'
	option ipaddr '192.168.50.51'
	option gateway '192.168.50.1'
	option netmask '255.255.255.0'
	option ip6assign '60'
```
重新启动网络

```
/ect/init.d/network restart
```
网络重启完成后，即可通过指定的地址 192.168.50.51 进行访问，用户名：root 密码：password