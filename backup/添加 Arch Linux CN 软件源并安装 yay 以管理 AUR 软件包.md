# 添加 Arch Linux CN 软件源并安装 yay 以管理 AUR 软件包

## 1. 添加 Arch Linux CN 源

编辑`/etc/pacman.conf`，在末尾添加两行：
```
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

## 2. 信任 GPG 密钥

安装`archlinuxcn-keyring`前需要信任`farseerfc`的 GPG 密钥，否则可能会安装失败，使用以下命令在本地信任`farseerfc`的 GPG 密钥：
```
pacman-key --lsign-key "farseerfc@archlinux.org"
```

## 3. 安装 ArchLinuxCN Keyring

安装`archlinuxcn-keyring`包以导入 GPG 密钥
```
pacman -Sy archlinuxcn-keyring
```

## 4. 更新软件包数据库

配置完源后，使用以下命令更新软件包数据库：
```
pacman -Syy
```

## 5. 安装 yay

使用以下命令安装 yay：
```
pacman -S yay
```
至此，已成功添加 ArchLinuxCN 源并安装了 yay，可以开始使用 yay 来管理 AUR 软件包了