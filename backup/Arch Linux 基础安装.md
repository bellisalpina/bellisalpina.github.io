# Arch Linux 基础安装

本指南从已启动的 Arch Live 环境开始。假设已通过 U盘等方式启动 ISO 镜像。

安装全程采用 UEFI + GPT 模式，不涉及传统 BIOS。

安装过程需要**稳定的网络连接**。

如果使用无线网络，确保网络名称（SSID）为英文，Live 环境无法显示或输入中文。

## 1. 禁用 reflector 服务

Arch Linux 安装镜像默认启用 `reflector` 服务，其自动选择的镜像源可能不准确，甚至会移除可用源。因此先禁用它。

```bash
systemctl stop reflector.service
```

## 2. 验证 UEFI 引导模式

执行以下命令，检查 EFI 变量目录是否存在：

```bash
ls /sys/firmware/efi/efivars
```

如果该目录存在并列出文件，则说明系统处于 UEFI 模式。

## 3. 连接网络

Live 环境默认使用 DHCP 自动配置网络。如需静态 IP，参考 Arch Wiki。

*   **有线连接**：插入网线即可。
*   **无线连接**：使用 `iwctl` 工具。

```bash
iwctl                           # 进入 iwctl 交互式命令行
device list                     # 列出网络设备，例如 wlan0
station wlan0 scan              # 扫描网络
station wlan0 get-networks      # 列出可用网络
station wlan0 connect YOUR_SSID # 连接网络，替换 YOUR_SSID 为实际名称，然后输入密码
exit                            # 连接成功后退出
```

使用 `ping` 测试网络连通性：

```bash
ping www.baidu.com
```

## 4. 同步系统时间

Live 环境默认启用 `systemd-timesyncd`，联网后会自动同步时间。

用 `timedatectl` 确认时间同步状态：

```bash
timedatectl status
```

如果时间未同步，执行以下命令：

```bash
timedatectl set-ntp true
```

## 5. 磁盘分区

使用 `fdisk -l` 查看并确定要操作的磁盘，例如 `/dev/sda` 或 `/dev/nvme0n1`。

本指南采用以下分区方案：

*   EFI 分区： `/efi`
*   根分区： `/`
*   用户主目录： `/home`

首先，将磁盘转换为 GPT 格式（以 `/dev/sdx` 为例）：

```bash
parted /dev/sdx
mklabel gpt
quit
```

**接下来使用 `fdisk`、`cfdisk` 等工具进行分区。具体操作因工具而异，此处不展开。**

建议将 EFI 分区设为磁盘的第一个分区，以避免潜在的兼容性问题。

分区结束后，用 `fdisk -l` 复查分区情况。

## 6. 格式化分区

对新创建的分区进行格式化。下文 `sdax` 中的 `x` 为分区号，请根据实际情况替换。

格式化根分区和用户主目录为 `ext4`：

```bash
mkfs.ext4 /dev/sdax
```

格式化 EFI 分区为 `FAT32`：

```bash
mkfs.fat -F 32 /dev/sdax
```

## 7. 挂载分区

挂载顺序必须正确：先挂载根分区 (`/`)，再挂载其他分区。

挂载根分区：

```bash
mount /dev/sdax /mnt
```

挂载 EFI 分区：

```bash
mount --mkdir /dev/sdax /mnt/efi
```

挂载用户主目录：

```bash
mount --mkdir /dev/sdax /mnt/home
```

## 8. 选择镜像站

`/etc/pacman.d/mirrorlist` 文件定义了软件包的下载镜像源。

使用 `curl` 获取中国大陆的 HTTPS 镜像源列表：

```bash
curl -L 'https://archlinux.org/mirrorlist/?country=CN&protocol=https' -o /etc/pacman.d/mirrorlist
```

编辑 `/etc/pacman.d/mirrorlist`，取消所需镜像站的注释。

## 9. 安装系统

使用 `pacstrap` 安装基础包、内核及一些必要工具：

```bash
pacstrap -K /mnt base base-devel linux-zen linux-zen-headers linux-firmware sudo networkmanager nano vim
```

## 10. 生成 fstab 文件

`fstab` 用于定义磁盘分区及其挂载信息。

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

**务必检查**生成的 `/mnt/etc/fstab` 文件内容是否正确。

## 11. 切换到新系统

执行 `arch-chroot` 切换到新安装的系统：

```bash
arch-chroot /mnt
```

## 12. 设置时区

设置时区（以上海为例）：

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

将系统时间写入硬件时钟：

```bash
hwclock --systohc
```

## 13. 区域和本地化设置

编辑 `/etc/locale.gen`，取消 `en_US.UTF-8 UTF-8` 和 `zh_CN.UTF-8 UTF-8` 的注释。

执行 `locale-gen` 生成 locale 信息：

```bash
locale-gen
```

创建 `/etc/locale.conf` 文件，设置语言环境：

```bash
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```

此处不推荐设置中文 locale，否则 tty 可能显示为方块。后续安装桌面环境后可再修改。

## 14. 设置主机名

创建 `/etc/hostname` 文件，写入主机名（例如 `myarch`）：

```bash
echo 'myarch' > /etc/hostname
```

编辑 `/etc/hosts` 文件，添加以下内容：

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   myarch
```

## 15. 设置 root 密码

设置 root 密码：

```bash
passwd
```

## 16. 安装引导程序

安装 GRUB（引导器）和 efibootmgr（用于管理 UEFI 启动项）：

```bash
pacman -S grub efibootmgr
```

安装 GRUB 到 EFI 分区：

```bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
```

NVIDIA 用户注意：若需使用 Wayland，需开启 DRM。编辑 `/etc/default/grub` 文件，在 `GRUB_CMDLINE_LINUX_DEFAULT` 一行末尾加入参数 `nvidia_drm.modeset=1`。

生成 GRUB 配置文件：

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

## 17. 完成安装

1.  输入 `exit` 退出 chroot 环境。
2.  执行 `umount -R /mnt` 卸载所有分区。
3.  执行 `reboot` 重启系统。重启前移除 U盘 等安装介质。

至此，一个基础的 Arch Linux 系统安装完成。