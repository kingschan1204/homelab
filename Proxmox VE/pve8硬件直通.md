# pve8硬件直通
> 硬件：cpu i7-12700

## 什么是直通？
> 直通就是指直接把硬件单独分配给虚拟机。与虚拟化相比，直通时虚拟机单独使用硬件，而虚拟化则是所有虚拟机共享硬件。比如把网卡分给软路由，把硬盘分给群晖，把显卡分给 Windows，从而在 PVE 中实现多台设备的功能，并且性能等同物理机！为什么性能等同物理机呢？因为虚拟机得到的就是物理硬件，直通就是将物理硬件分配给虚拟机。
```
狭义的 Intel® VT 主要提供分别针对处理器、芯片组、网络的虚拟化技术。

处理器虚拟化技术（Intel VT-x）：包括虚拟化灵活迁移技术（Intel VT FlexMigration）、中断加速技术（Intel VT FlexPriority）、内存虚拟化技术（Intel EPT）

芯片组虚拟化技术（Intel VT-d）：直接 I/O 访问技术

I/O 虚拟化技术（Intel VT-c）：包括虚拟机设备队列技术（VMDq）、虚拟机直接互连技术（VMDc）、网卡直通技术（SR-IOV/MR-IOV）、数据直接 I/O 技术（DDIO）

目前主要的 CPU 虚拟化技术是 Intel 的 VT-x/VT-i 和 AMD 的 AMD-V 这两种技术。

Intel CPU 虚拟化技术主要有 2 类：

VT-x：用于 X86 架构的的 CPU 虚拟化技术（Intel Virtualization Technologyfor x86），主要是 IA-32 和 Intel 64 系列处理器。

VT-i：用于安腾（Itanium）架构处理器的 CPU 虚拟化技术（Intel Virtualization Technology for ltanium），主要是 Itanium 系列处理器
```

## 配置 PCI 直通
> 配置 PCI 直通可以让你实现在虚拟机中使用 PCI 设备（如显卡，网卡）

### CPU 要求
> 对于Intel CPU，首先确认处理器是否支持 VT-d 技术(IO 虚拟化技术)：打开 Intel 官网，搜索你的处理器型号，查看如下参数：
```
https://www.intel.cn/content/www/cn/zh/products/sku/134591/intel-core-i712700-processor-25m-cache-up-to-4-90-ghz/specifications.html?wapkw=i7-12700
安全性与可靠性
英特尔® vPro® 资格‡ Intel vPro® Enterprise, Intel vPro® Essentials
英特尔® Hardware Shield 参与资格 ‡ 是
英特尔® Threat Detection Technology (TDT)是
英特尔® 主动管理技术 (AMT) ‡ 是
英特尔® Standard Manageability (ISM) ‡ 是
英特尔® One-Click Recovery ‡是
英特尔® 控制流强制技术 是
英特尔® 全内存加密 - 多密钥是
英特尔® 全内存加密 是
英特尔® AES 新指令 是
安全密钥 是
英特尔® 操作系统守护是
英特尔® Trusted Execution Technology ‡ 是
执行禁用位 ‡ 是
英特尔® Boot Guard 是
基于模式的执行控制 (MBEC) 是
英特尔® 稳定映像平台计划 是
采用 Redirect Protection（VT-rp）的英特尔® 虚拟化技术‡是
英特尔® 虚拟化技术 (VT-x) ‡ 是
英特尔® Virtualization Technology for Directed I/O (VT-d) ‡ 是
英特尔® VT-x with Extended Page Tables (EPT) ‡ 是
```
如果都显示是，则表示支持 硬件虚拟化 和 IOMMU。

对于 AMD 处理器，只要 CPU 是 Bulldozer 或者之后的型号都支持，K10 之后 CPU 需要 890FX 或者 990F 主板。

## 主板要求
> 主板要支持 IOMMU。
```
# 如果输出中包含有关IOMMU的信息，则表示您的系统支持并启用了IOMMU。
dmesg | grep -e DMAR -e IOMMU
```
## GPU 要求
如果 GPU 支持 UEFI，推荐使用 OVMF(UEFI) 而不是使用 SeaBIOS。

## 配置启用 IOMMU
Intel
```
# 编辑grub
vi /etc/default/grub

# 将 
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
# 修改为 
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```
AMD
```
# 编辑grub
vi /etc/default/grub

# 将 
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
# 修改为 
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"
```
> 配置完成以后，执行：
```
# 更新 grub
update-grub

# 刷新系统配置
proxmox-boot-tool refresh

# 重启后验证硬件直通是否成功，运行下面命令，出现很多直通组，就代表成功了。如果没有任何东西，就是没有开启
find /sys/kernel/iommu_groups/ -type l 
```
## 配置显卡直通
> 没有独立显卡，没试过

## 增加虚拟化驱动，加载 vifo 系统模块
> 这仅在必要时启用IOMMU转换，将iommu分组相关的内核模块启用，从而可以提高VM中未使用的PCIe设备的性能。
```
# 加载相应的内核
echo vfio >> /etc/modules
echo vfio_iommu_type1 >> /etc/modules
echo vfio_pci >> /etc/modules
echo vfio_virqfd >> /etc/modules

# 更新内核参数
update-initramfs -u -k all

# 重启
reboot

# 重启后查看 /proc/cmdline 有没有将我们在 /etc/kernel/cmdline 的修改添加进去
cat /proc/cmdline
```
