## [](#选择基底系统 "选择基底系统")选择基底系统

在 Proxmox VE 中使用的 Debian 系统不能使用 Debian 的 ISO 安装镜像（即那个 4 GiB 左右大小的 Installation ISO 或 netinst 的 ISO 都不合适），因为那个镜像是为了 Linux 用户在自己的个人电脑上安装 Debian 而设计的。[Debian Official Cloud Image](https://cdimage.debian.org/images/cloud/) 则更适合作为 PVE 中部署的 Debian 系统的基底：Cloud Image 去掉了自带的大量消费级硬件的驱动和软件包、使得镜像体积大大减少；而且还内置了大量针对云环境的优化如默认在 grub 和 systemd 服务中开启了 ttyS0 调试串口；自带 Cloud-init 支持；同时支持 Legacy BIOS 和 UEFI 启动；等等。

Debian 官方提供各个变种的 Cloud Image 有多种变种，适合 Microsoft Azure、AWS EC2、OpenStack 和普通（Plain）云环境上进行部署的镜像。其中，普通云环境的镜像又分为 `generic` 和 `genericcloud` 两个变种，`generic` 相比 `genericcloud` 额外增加了一些驱动以便部署至裸金属物理机上；而 `genericcloud` 只包含 VirtIO 等虚拟机需要的驱动，因此 `genericcloud` 镜像相比 `generic` 镜像体积更小。`genericcloud` 镜像的体积普遍小于 350 MiB，不到 Debian ISO 官方安装镜像体积的十分之一。

现有的教程普遍都在推荐使用 `generic` 镜像，然而实际上我们的物理宿主机上已经使用的是 Proxmox VE、我们构建的镜像只会用在虚拟机中，**所以我们直接使用 `genericcloud` 镜像即可**。

> 大部分常用基础 Linux 发行版也都提供了 Cloud Image 镜像，因此对于喜欢 Ubuntu、CentOS 等其他 Linux 发行版的用户来说，可以去找找对应的 Cloud Image 镜像。这里列出几个常用 Linux 发行版的 Cloud Image 镜像的官方地址：
> 
> +   [Debian Cloud](https://cdimage.debian.org/images/cloud/)
> +   [Ubuntu Cloud](https://cloud-images.ubuntu.com/)
> +   [CentOS Cloud](https://cloud.centos.org/)
> +   [AlmaLinux Cloud](https://wiki.almalinux.org/cloud/Generic-cloud.html)
> +   [Fedora Cloud](https://fedoraproject.org/cloud/)
> +   [OpenSUSE Cloud](https://download.opensuse.org/repositories/Cloud:/Images:/)

## [](#定制系统镜像 "定制系统镜像")定制系统镜像

在 Proxmox VE 上的 VM 都建议安装 QEMU Guest Agent 以便于更好地实现 物理宿主机 与 虚拟机 之间的通信和交互，例如通过虚拟机中的 QEMU Guest Agent 可以正确的关闭虚拟机（而不是使用 ACPI 指令，在不支持 ACPI 的 VM 中尤为重要）、通过 QEMU Guest Agent 冻结系统以便更好地克隆与备份虚拟机、同步虚拟机与物理宿主机之间的时间等。除此以外，最直观的一个好处，就是在 Proxmox VE 的 Web 界面中可以直接看到安装并启用了 QEMU Guest Agent 的虚拟机的网卡 IP 地址。

除了需要安装 QEMU Guest Agent 以外，我们通常也会需要在虚拟机中安装一些常用软件包，例如 curl、wget、unzip 等常用工具，我自己则还需要安装常见的用于调试网络环境的工具，例如 `mtr`、`dnsutils` 提供的 `nslookup` 和 `dig` 等。除了安装必要软件包之外，我们还可能需要对系统进行其他操作，例如修改时区、更换软件包镜像源、修改 NTP 服务器，等等等。每次创建一个 VM 都要重复手动进行这些操作就太麻烦了，制作属于自己的定制化 Debian Cloud Image 镜像就可以解决这个问题。

现有的教程普遍都推荐创建虚拟机以后、启动虚拟机后、进入虚拟机中进行定制化操作，然后再将虚拟机关闭、转换为模板。然而，这种做法也有几个缺点。首先是 不能有效记录下来对系统的修改操作，难以实现可复现构建和部署；此外，只要对虚拟机进行任何操作都会留下大量痕迹，例如缓存、日志、`/etc/machine-id` 等等、影响后续模板克隆出来的虚拟机的使用（例如 DHCP 冲突、MAC 地址重复，等等），而这些痕迹凭手工是难以清除完全的。因此，我们需要使用一个能够以声明式的方式来操作系统镜像、并且能自动清除系统中的痕迹、修复 SELinux 状态的工具：

## [](#使用-libguestfs-tools-制作定制化镜像 "使用 libguestfs-tools 制作定制化镜像")使用 libguestfs-tools 制作定制化镜像

`libguestfs-tools` 是一套用于操作虚拟机磁盘镜像的命令行工具集，也专门为镜像定制做了优化，绝大部分发行版的软件仓库也都有打包，可以直接安装（顺便安装一下多线程下载工具 `axel` 和 `qemu-utils`）：

```none
sudo apt-get update && sudo apt-get install -y libguestfs-tools axel qemu-utils
```

首先我们下载 Debian Cloud Image 镜像。我个人习惯用多线程下载工具 `axel` 来下载大文件：

```SHELL
axel -n 8 -o debian-12-genericcloud-amd64-sukka-src.qcow2 https://cdimage.debian.org/images/cloud/bookworm/latest/debian-12-genericcloud-amd64.qcow2
```

Debian Cloud Image 因为构建次数频繁、文件总量和体积都很大，国内仅有少数高校开源镜像站会同步 Debian Cloud Image，而国内互联网公司维护的开源镜像站 没有任何一家同步了 Debian Cloud Image。我在这里不会列出任何国内的 Debian Cloud Image 镜像地址，有需要的请自行通过网络上的公开信息进行寻找。

`libguestfs-tools` 本质上会挂载当前 Linux 系统的内核来启动一个 QEMU 虚拟机。为了以防万一，在 CI 上我会使用 `dpkg-statoverride` 来开放对当前内核的访问，不建议在除了 CI 以外的任何环境中使用。

```SHELL
sudo dpkg-statoverride --update --add root root 0644 /boot/vmlinuz-`uname -r`
```

我们会分别用到 `libguestfs-tools` 提供的工具 `virt-customize` 来定制镜像。这是我在定制我的 Debian Cloud Image 时使用的命令：

```SHELL
virt-customize -a debian-12-genericcloud-amd64-sukka-src.qcow2 \
  --smp 2 --verbose \
  --timezone "Asia/Hong_Kong" \
  --append-line "/etc/default/grub:# disables OS prober to avoid loopback detection which breaks booting" \
  --append-line "/etc/default/grub:GRUB_DISABLE_OS_PROBER=true" \
  --run-command "update-grub" \
  --run-command "systemctl enable serial-getty@ttyS1.service" \
  --run-command "sed -i 's|Types: deb deb-src|Types: deb|g' /etc/apt/sources.list.d/debian.sources" \
  --run-command "sed -i 's|generate_mirrorlists: true|generate_mirrorlists: false|g' /etc/cloud/cloud.cfg.d/01_debian_cloud.cfg" \
  --update --install "sudo,qemu-guest-agent,spice-vdagent,bash-completion,unzip,wget,curl,axel,net-tools,iputils-ping,iputils-arping,iputils-tracepath,nano,most,screen,less,vim,bzip2,lldpd,mtr-tiny,htop,dnsutils,zstd" \
  --run-command "apt-get -y autoremove --purge && apt-get -y clean" \
  --append-line "/etc/systemd/timesyncd.conf:NTP=time.apple.com time.windows.com" \
  --delete "/var/log/*.log" \
  --delete "/var/lib/apt/lists/*" \
  --delete "/var/cache/apt/*" \
  --truncate "/etc/apt/mirrors/debian.list" \
  --append-line "/etc/apt/mirrors/debian.list:https://[redacted]/debian" \
  --append-line "/etc/apt/mirrors/debian.list:https://[redacted]/debian" \
  --append-line "/etc/apt/mirrors/debian.list:https://[redacted]/debian" \
  --truncate "/etc/apt/mirrors/debian-security.list" \
  --append-line "/etc/apt/mirrors/debian-security.list:https://[redacted]/debian-security" \
  --append-line "/etc/apt/mirrors/debian-security.list:https://[redacted]/debian-security" \
  --append-line "/etc/apt/mirrors/debian-security.list:https://[redacted]/debian-security" \
  --truncate "/etc/machine-id"
```

这里简单介绍一下几个参数分别是在做什么。

**\--smp 2**

如前文所说，`libguestfs-tools` 会启动一个虚拟机在镜像中执行命令，通过 `--smp` 参数可以指定虚拟机的 CPU 核心数为 2 核。

**\--timezone "Asia/Hong\_Kong"**

设置系统时区为 UTC+8，这里我以香港为例。

**\--append-line "/etc/default/grub:GRUB\_DISABLE\_OS\_PROBER=true" --run-command "update-grub"**

在 grub 配置文件中添加一行 `GRUB_DISABLE_OS_PROBER=true`，以禁止 grub 自动探测其他操作系统。因为在虚拟机中一般只有一个操作系统，启用 OS prober 以后不仅会影响启动速度，有的时候还可能会导致系统完全无法启动。

修改 grub 配置文件以后需要执行 `update-grub` 来更新 grub。

**\--run-command "systemctl enable [serial-getty@ttyS1.service](mailto:serial-getty@ttyS1.service)"**

Debian Cloud Image 中已经默认启用了 grub 的 ttyS0 串口调试功能 和 系统中 systemd 的 ttyS0 串口调试服务。这里我额外启用了 ttyS1 的串口调试服务。

**\--run-command "sed -i 's|Types: deb deb-src|Types: deb|g' /etc/apt/sources.list.d/debian.sources"**

Debian Cloud Image 已经开始使用 [DEB822](https://manpages.debian.org/bookworm/dpkg-dev/deb822.5.en.html) 的软件源配置语法格式，所以不论是换源还是修改配置都需要使用 DEB822 格式。这一步是禁用 `deb-src` 源码包，因为大部分时候都不会需要从源码编译软件包。

**\--run-command "sed -i 's|generate\_mirrorlists: true|generate\_mirrorlists: false|g' /etc/cloud/cloud.cfg.d/01\_debian\_cloud.cfg"**

Debian Cloud Image 不仅使用 DEB822 格式，还使用了 mirrorlist 格式的源列表。Debian Cloud Image 中的 Cloud-init 默认会根据配置自动覆盖 mirrorlist，通过禁用 Cloud-init 的 `apt.generate_mirrorlists` 来避免我们自己的软件源列表被覆盖掉。

**\--update --install**

`--update` 参数会让 `virt-customize` 自动检测当前虚拟机镜像的操作系统和所使用的包管理器，并自动使用对应的包管理器将镜像中所有的包更新到最新版本。`--install` 参数则会使用对应的包管理器安装指定的软件包。我自己预装了这些软件：

+   `qemu-guest-agent,spice-vdagent`：PVE 支持的 Guest Agent。
+   `axel`：多线程下载工具。
+   `net-tools,iputils-ping,iputils-arping,iputils-tracepath,mtr-tiny,dnsutils`：常用的网络工具，部分工具在 Debian Cloud Image 中可能已经预装，这里列出来是为了确保它们一定存在。
+   `lldpd`：这是我下一篇博客会专门介绍的妙妙工具~
+   `sudo,bash-completion,unzip,wget,curl,nano,most,screen,less,vim,bzip2,lldpd,htop,zstd`

**\--run-command "apt-get -y autoremove --purge && apt-get -y clean"**

在安装完软件后，清理掉不需要的间接依赖和缓存。

**\--append-line "/etc/systemd/timesyncd.conf:NTP=time.apple.com time.windows.com"**

Debian Cloud Image 中默认使用 `systemd-timesyncd` 来同步时间。这里我将默认的 NTP 服务器改为苹果的 `time.apple.com` 和微软 `time.windows.com`。这两个 NTP 服务器都在亚太地区有节点，和国内三大运营商都有互联，延迟都还不错。

**\--delete "/var/log/*.log" --delete "/var/lib/apt/lists/*" --delete "/var/cache/apt/\*"**

手动清理掉一些不必要的日志和 APT 缓存。

**\--truncate "/etc/apt/mirrors/debian.list" --append-line**

清空 `/etc/apt/mirrors/debian.list` 文件后添加我自己的软件源列表。

在这里，我要花一大段文字，强烈建议大家 **在生产环境慎重使用来自高校的镜像站！**

虽然国内的互联网公司（如阿里云、腾讯云、华为云、字节火山云、网易）的开源镜像站大多年久失修、收录不全（由 `中移（苏州）软件技术有限公司` 维护的移动云开源镜像站甚至没有收录 Debian 软件包镜像）、更新频率低、经常同步失败（阿里云的 Debian 和 Ubuntu 镜像虽然每天都和上游同步，但是每个月同步成功次数不到 10 次）、几乎不维护、联系邮箱几乎不回复、甚至需要登陆才能查看镜像地址（说的就是你华为，遥遥领先，Only Huawei Can Do）。而相比之下，国内高校镜像站简直是一股清流，不论是更新频率、教程内容、响应时间都在全方位吊打国内其他公司的镜像站。

然而，国内高校镜像站大多由高校的网络信息中心、超算中心、计算机协会、开源软件协会等支持创办，维护者大多为在校学生和老师中的志愿者，因此高校镜像站难以提供任何 SLA 保障，网络带宽资源也受限，镜像站的运行维护也受到来自学校、工信部、运营商等各个方面的制约。高校镜像站带宽不足、因特殊原因封网导致镜像站关闭公网访问、甚至因学校要求而整个关闭镜像站的情况（如上海大学开源镜像站于 2020 年应学校要求关闭）时有发生。

而在三大运营商以非法违规手段严查家庭宽带 PCDN、擅自中断家庭宽带合约的情况下，大量 PCDN 用户只好通过刷下载流量来规避上下行流量占比检测，甚至不少 PCDN 软件更是内置了刷下载流量的功能（说的就是你，滥刷 BT 下载祸害普通 BT 做种用户的 123 云盘）。无偿提供操作系统安装镜像下载的高校镜像站也自然而然的成为了 PCDN 用户的首选滥用目标，高校镜像站的带宽也因此变得更加紧张，不得不实施针对 IP 限速甚至封禁 IP 的措施。

因此，我强烈建议大家在生产环境中慎重使用来自高校的镜像站。如果对 SLA 有任何要求都请直接使用操作系统和软件内置的官方默认地址，或自行在内网搭建镜像。为了避免高校的开源镜像站被滥用，本文故意隐藏了所有高校开源镜像站的地址。

**\--truncate "/etc/machine-id"**

清空 `/etc/machine-id` 文件。对于大部分安装在个人电脑的 Linux 系统来说， `/etc/machine-id`文件应该在系统安装时生成。而对于在云、容器中的 Linux 系统（例如 Debian Cloud），`/etc/machine-id` 文件应该在首次启动时生成。

## [](#减少镜像体积 "减少镜像体积")减少镜像体积

在定制完镜像后，硬盘镜像文件的体积都会大幅增加（通常是因为 `virt-customize` 使用该硬盘镜像文件启动了虚拟机执行命令导致的硬盘空间分配）。为了减少镜像体积，我们需要将硬盘镜像文件中的未使用空间清除掉、必要时还可以启用硬盘镜像压缩功能。我们可以使用 `virt-sparsify` 实现这一目标：

```SHELL
virt-sparsify --compress debian-12-genericcloud-amd64-sukka-src.qcow2 debian-12-genericcloud-amd64-sukka.qcow2
```

相比 Debian Cloud Image 原版 `genericcloud` 镜像 331.7 MiB 的体积，我定制后的镜像压缩后体积为 338.0 MiB，仅增加了 6.3 MiB。

## [](#在-Proxmox-VE-中导入镜像并创建虚拟机模板 "在 Proxmox VE 中导入镜像并创建虚拟机模板")在 Proxmox VE 中导入镜像并创建虚拟机模板

首先我们需要把定制后的镜像上传到 PVE 中。为了方便，我直接在 PVE 中将我定制的 `debian-12-genericcloud-amd64-sukka.qcow2` 通过 `wget` 下载到 PVE 的 Home 目录中（即 `/root`）。

然后在 PVE 中创建一个空白虚拟机：

```SHELL
qm create 9000 \
  --machine q35 \
  --cpu cputype=host \
  --name "debian-12-cloud-template" \
  --scsi2 "local-lvm:cloudinit" \
  --serial0 socket \
  --vga none \
  --scsihw virtio-scsi-single \
  --net0 virtio,bridge=vmbr0 \
  --agent 1 \
  --ostype l26 \
  --memory 1024
```

需要注意，大部分现有的教程都会教你使用 `--ide2 "local-lvm:cloudinit"` 来创建 Cloud-init 镜像，**然而这是完全过时的**。Debian Cloud Image 已经完全移除了 AHCI 驱动、使用 `--ide[X]` 参数创建的硬盘是无法挂载的。但是，Debian Cloud Image 内置了 VirtIO SCSI 驱动，因此 Cloud-init 镜像也应该使用 SCSI，即 `--scsi2 "local-lvm:cloudinit"`。

除此以外，Debian Cloud Image 依赖 `ttyS0` 设备存在，所以我们需要使用 `--serial0 socket` 来创建一个串口设备。如果定制的镜像里内置了 QEMU Guest Agent，可以使用 `--agent 1` 来启用虚拟机的 QEMU Guest Agent。因为创建的虚拟机是要作为模板使用的，因此 CPU和内存只分配了 1 核 1 G（硬盘届时也只会使用 Debian Cloud Image 的默认大小 3 GiB），后续克隆虚拟机后再调整硬件配置。

然后我们将镜像导入到 PVE 的存储中、并 attach 到刚刚创建的虚拟机中、并设置为该虚拟机的启动盘：

```SHELL
qm importdisk 9000 "/path/to/image.qcow2" local-lvm -format qcow2
qm set 9000 --scsi0 "local-lvm:vm-9000-disk-0,discard=on,ssd=1"
qm set 9000 --boot order=scsi0
```

由于我绝大部分创建的虚拟机都将会使用 DHCP 分配 IP，所以我也为模板的 Cloud-init 中设置默认通过 DHCP 获取 IP 地址：

```SHELL
qm set 9000 --ipconfig0 ip=dhcp
```

通过以上操作，我们就成功创建了一个可以作为模板的虚拟机。可以用下述命令验证一下储存在 PVE 中的 Cloud-init 配置：

```SHELL
qm cloudinit dump 9000 user
qm cloudinit dump 9000 network
```

最后我们将这个虚拟机转换为模板：

```SHELL
qm template 9000
```

在创建模板期间，我们全程没有启动该虚拟机。

## [](#使用模板创建虚拟机 "使用模板创建虚拟机")使用模板创建虚拟机

在创建虚拟机时，我们可以选择刚刚创建的模板、完整克隆一个虚拟机（建议不要使用 Linked Clone 从模板创建虚拟机）：

```SHELL
qm clone 9000 100 \
  --name "debian-12-cloud-vm" \
  --full 1 \
  --pool "local-lvm" \
  --storage "local-lvm" \
  --format qcow2
```

克隆出来的虚拟机使用和模板一样的硬件配置，我们可以根据自己的需要修改配置。例如，将硬盘从 3 GiB 库容到 5 GiB：

```SHELL
qm resize 100 scsi0 +2G
```

Debian Cloud Image 默认不带任何用户，root 用户也没有密码、不能登录。我们需要用 Cloud-init 配置用户及其密码、SSH Key：

```SHELL
qm set 100 --ciuser root
qm set 100 --cipassword "1145141919810"
qm set 100 --sshkey /path/to/ssh-key.pub
```

为 Proxmox VE 定制 Debian Cloud 系统镜像与创建虚拟机模板