# Debian 13 替换为 USTC 源

为了提升软件包下载速度，可以将 Debian 13 的默认软件源替换为中科大（USTC）的镜像源。

## 操作步骤

### 1. 备份原有源列表

```bash
cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

### 2. 删除传统格式的源

```bash
rm /etc/apt/sources.list
```

### 3. 写入DEB822格式的新的源

```bash
cat <<'EOF' >/etc/apt/sources.list.d/debian.sources
Types: deb
URIs: https://mirrors.ustc.edu.cn/debian
Suites: trixie trixie-updates trixie-backports
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

Types: deb
URIs: https://mirrors.ustc.edu.cn/debian-security
Suites: trixie-security
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
EOF
```

### 3. 更新软件包列表

执行更新命令，使新的源生效。

```bash
apt update
```