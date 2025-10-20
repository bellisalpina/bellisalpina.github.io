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
cat << EOF > /etc/apt/sources.list
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

### 3. 更新软件包列表

源文件替换后，必须执行更新命令，使新的源生效。

```bash
apt update
```