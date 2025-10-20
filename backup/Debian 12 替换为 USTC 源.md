# Debian 12 替换为 USTC 源

为了提升软件包下载速度，可以将 Debian 12 的默认软件源替换为中科大（USTC）的镜像源。

## 操作步骤

### 1. 备份原有源列表

在进行任何修改前，建议先备份原有的源文件。

```bash
cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

### 2. 写入新的源列表

以下命令会直接覆盖 `/etc/apt/sources.list` 文件。

```bash
tee /etc/apt/sources.list > /dev/null <<EOF
deb https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
# deb-src https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
deb https://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
# deb-src https://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
deb https://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
# deb-src https://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
deb https://mirrors.ustc.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware
# deb-src https://mirrors.ustc.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware
EOF
```

> - 这里使用 `tee` 和 `<<EOF` 的方式，比 `echo` 更清晰地处理多行文本，并且避免了 `echo` 命令中可能存在的引号转义问题。
> - `tee ... > /dev/null` 的作用是写入文件时不在终端显示内容。

### 3. 更新软件包列表

源文件替换后，必须执行更新命令，使新的源生效。

```bash
apt update
```