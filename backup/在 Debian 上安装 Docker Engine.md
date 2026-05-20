# 在 Debian 上安装 Docker Engine

## 1. 卸载旧版本（可选）

在安装 Docker Engine 之前，需要卸载任何可能冲突的包。

```bash
apt remove $(dpkg --get-selections docker.io docker-compose docker-doc podman-docker containerd runc | cut -f1)
```

`apt` 可能会报告你未安装这些包。

## 2. 设置 Docker 的 APT 仓库

为了能够方便地安装和更新 Docker，需要先配置 Docker 官方的 GPG 密钥和仓库。

### 第一步：更新系统并安装必要工具

```bash
apt update && apt install -y ca-certificates curl
```

### 第二步：添加 Docker 的官方 GPG 密钥

GPG 密钥用于验证下载软件包的完整性。创建一个目录来存放密钥，并下载官方密钥：

```bash
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
```

### 第三步：添加 Docker 仓库到 Apt 源

接下来，将 Docker 的仓库配置写入 `docker.sources` 文件。

```bash
cat <<EOF >/etc/apt/sources.list.d/docker.sources
Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

## 3. 安装 Docker 组件

配置好仓库后，更新软件包索引并安装 Docker Engine 以及配套工具：

```bash
apt update && apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 4. 验证安装

安装完成后，可以通过运行 `hello-world` 镜像来验证 Docker 是否安装成功：

```bash
docker run hello-world
```

## 总结

现在已经成功配置了 Docker 环境。使用官方仓库安装的好处是，未来可以直接通过 `apt update && apt upgrade` 来获取 Docker 的最新安全补丁和功能更新。