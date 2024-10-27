# Arch Linux 基础安装

本文基于，已通过包括但不限于U盘刻录ISO镜像等方式，启动进入到 Live 环境下

安装将全部以 UEFI+GPT 的形式进行，传统 BIOS 方式不再赘述

安装 Arch Linux 的过程中需要**稳定的网络连接**

如果只能使用无线网络，需要事先把 Wi-Fi 名称改成英文，因为安装时无法显示和输入中文

## 1. 禁用 reflector 服务

2020 年，Arch Linux安装镜像中加入了`reflector`服务，`reflector`会自动选择速度合适的镜像源，但结果并不准确，特定情况下，它会误删某些有用的源信息

通过以下命令将该服务禁用：
```
systemctl stop reflector.service
```
## 2. 验证是否为 UEFI 引导模式

执行命令：
```
ls /sys/firmware/efi/efivars
```
如果输出了一堆东西(EFI变量)，则说明已在 UEFI 模式

## 3. 连接网络

大部分情况下，DHCP能很好地自动配置IP和DNS，如果需要配置静态IP和DNS，自行参考Arch Wiki

对于有线连接，插入网线即可

对于无线连接，则需进行如下操作

无线连接使用`iwctl`命令进行，按照如下步骤进行网络连接：
```
iwctl                           #执行iwctl命令，进入交互式命令行
device list                     #列出设备名，比如无线网卡叫wlan0
station wlan0 scan              #扫描网络
station wlan0 get-networks      #列出网络 比如想连接xxxxx这个无线网络
station wlan0 connect xxxxx     #连接 输入密码
exit                            #成功后exit退出
```
连接成功后通过`ping`测试网络连接：
```
ping www.baidu.com
```

## 4. 更新系统时间

在 Live 环境中`systemd-timesyncd`默认启用，也就是说当系统已经创建互联网连接后，系统时间将自动同步

使用`timedatectl`确保系统时间是同步的：
```
timedatectl
```
如果时间不同步，使用以下命令：
```
timedatectl set-ntp true    #将系统时间与网络时间进行同步
timedatectl status          #检查服务状态
```

## 5. 磁盘分区

使用`fdisk`查看并确定要操作的磁盘：
```
fdisk -l
```
本文使用以下分区方案：
+ EFI 分区： `/efi`  
+ 根分区： `/`  
+ 用户主目录： `/home`  

首先将磁盘转换为 GPT 格式，假设磁盘名称为`sdx`，NVME 的固态硬盘磁盘名称可能为`nvme0n1`
```
parted /dev/sdx     #执行parted，进入交互式命令行
mklabel gpt         #将磁盘类型转换为GPT 如磁盘有数据会警告 输入yes即可
quit                #quit退出parted命令行交互
```
**接下来使用分区工具 (fdisk、parted、cfdisk等等) 对磁盘进行分区，不同工具操作方式不同，此处不过多赘述**

建议将 EFI 分区设置为磁盘的第一个分区，避免可能出现的兼容性问题

分区结束后，用`fdisk`复查磁盘的情况：

```
fdisk -l
```

## 6. 格式化分区

创建分区后，必须使用合适的文件系统对每个新创建的分区进行格式化

以下命令中的`sdax`中，`x`代表分区的序号，与上一步创建分区时的序号对应

使用`mkfs.ext4`格式化根分区和用户主目录：
```
mkfs.ext4 /dev/sdax
```
使用`mkfs.fat`格式化 EFI 分区：
```
mkfs.fat -F 32 /dev/sdax
```

## 7. 挂载分区

挂载是有顺序的，必须先挂载根分区，再挂载其他分区，此处`sdax`仅为举例，请根据实际情况挂载

将根分区挂载到 `/mnt`
```
mount /dev/sdax /mnt
```
挂载 EFI 分区
```
mount --mkdir /dev/sdax /mnt/efi
```
挂载用户主目录
```
mount --mkdir /dev/sdax /mnt/home
```

## 8. 选择镜像站

`/etc/pacman.d/mirrorlist`定义了软件包会从哪个镜像站下载

使用 `curl`从 archlinux 官方网站的镜像站列表下载位于中国大陆的 HTTPS 镜像站
```
curl -L 'https://archlinux.org/mirrorlist/?country=CN&protocol=https' -o /etc/pacman.d/mirrorlist
```
手动编辑`/etc/pacman.d/mirrorlist`，取消注释以启用镜像站

## 9. 安装系统

使用`pacstrap`安装必须的软件包：
```
pacstrap -K /mnt base base-devel linux-zen linux-zen-headers linux-firmware
```
功能性软件：
```
pacstrap -K /mnt sudo networkmanager nano vim
```

## 10. 生成 fstab 文件

`fstab`用来定义磁盘分区
```
genfstab -U /mnt >> /mnt/etc/fstab
```
**强烈建议**在执行完以上命令后，检查一下生成的`/mnt/etc/fstab`文件是否正确

## 11. chroot 到新安装的系统

通过以下命令 chroot 到新安装的系统：
```
arch-chroot /mnt
```

## 12. 设置时区

通过以下命令设置时区：
```
ln -sf /usr/share/zoneinfo/Region(地区名)/City(城市名) /etc/localtime
```
将当前的系统时间同步到硬件时钟：
```
hwclock --systohc
```

## 13. 区域和本地化设置

编辑`/etc/locale.gen`，然后取消掉`en_US.UTF-8`和`zh_CN.UTF-8`的注释

接着执行`locale-gen`以生成 locale 信息：
```
locale-gen
```
然后创建`/etc/locale.conf`文件，并编辑设定 LANG 变量：
```
LANG=en_US.UTF-8
```
并不推荐在此设置任何中文 locale，这可能会导致 tty 上中文显示为方块。如果不经常使用 tty ，或是稍后需要安装桌面环境，则在不使用 tty 后可以设置为中文的 locale

## 14. 设置主机名

创建`hostname`文件：
```
echo '主机名' > /etc/hostname
```
编辑`/etc/hosts`文件，加入以下内容：
```
127.0.0.1   localhost
::1         localhost
127.0.1.1   主机名
```

## 15. 设置 root 密码

使用以下命令设置 root 密码：
```
passwd root
```

## 16. 安装引导程序

GRUB 是启动引导器，efibootmgr 被 GRUB 脚本用来将启动项写入 NVRAM
```
pacman -S grub efibootmgr
```
将 GRUB 安装到硬盘上：
```
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
```
N卡用户注意，如果需要使用 wayland，则需开启 DRM，编辑`/etc/default/grub`文件，在`GRUB_CMDLINE_LINUX_DEFAULT`一行末尾加入参数`nvidia_drm.modeset=1`

最后生成 grub.cfg 配置文件：
```
grub-mkconfig -o /boot/grub/grub.cfg
```

## 17. 完成安装

输入`exit`退出 chroot 环境

输入`umount -R /mnt`卸载被挂载的分区

最后执行`reboot`重启系统，重启前别忘了移除安装介质

至此，一个基础的、无图形界面的 Arch Linux 就安装完成了