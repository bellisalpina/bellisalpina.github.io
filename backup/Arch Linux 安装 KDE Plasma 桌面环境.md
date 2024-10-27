# Arch Linux 安装 KDE Plasma 桌面环境

本文介绍 Arch Linux 安装 KDE Plasma 桌面环境的过程

## 1. 创建一个非 root 用户

使用`useradd`命令创建一个新用户，并将其添加到`wheel`组，以便能够使用`sudo`提升权限
```
useradd -m -G wheel -s /bin/bash myarch
```
这里的`myarch`是希望创建的用户名

## 2. 设置用户密码

使用`passwd`命令为新用户设置密码：
```
passwd myarch
```
## 3. 配置 sudo 权限

使用`visudo`命令编辑`/etc/sudoers`文件，以允许`wheel`组的用户使用`sudo`

```
visudo
```
`visudo`会打开默认的编辑器编辑`/etc/sudoers`文件

可以通过设置环境变量来选择其他编辑器，例如`nano`：
```
export EDITOR=nano
visudo
```
不建议直接使用文本编辑器编辑`/etc/sudoers`文件，可能会导致配置错误  
`visudo`会在保存时自动检查语法，如果有错误，系统会警告并拒绝保存更改

找到以下行并去掉前面的注释符号，这将允许`wheel`组的用户使用`sudo`命令：
```
#%wheel ALL=(ALL:ALL) ALL
```
完成编辑后，保存并退出编辑器

## 4. 安装 KDE Plasma

安装 plasma 元软件包、konsole 终端以及 dolphin 文件管理器
```
pacman -S plasma-meta konsole dolphin
```

## 5. 安装 sddm 图形登录管理器

使用以下命令安装`sddm`图形登录管理器：
```
pacman -S sddm
```
启用并立即启动`sddm`：
```
systemctl enable --now sddm
```
至此已经能看到图形化的登录界面了