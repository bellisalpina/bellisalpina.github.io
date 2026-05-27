# Debian 13 安装 Tailscale

## 安装步骤

### 1. 添加 Tailscale 的 GPG Key

先创建 keyring 目录，并下载 Tailscale 的签名密钥：

```bash
mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkgs.tailscale.com/stable/debian/trixie.noarmor.gpg -o /usr/share/keyrings/tailscale-archive-keyring.gpg
```

### 2. 添加 Tailscale 软件源

将 Tailscale 仓库添加到 APT 源列表中：

```bash
echo "deb [signed-by=/usr/share/keyrings/tailscale-archive-keyring.gpg] https://mirrors.ustc.edu.cn/tailscale/debian trixie main" >/etc/apt/sources.list.d/tailscale.list
```

### 3. 更新软件索引并安装 Tailscale

刷新软件包索引，然后安装 Tailscale：

```bash
apt update && apt install tailscale
```

### 4. 启动 Tailscale

安装完成后，启动并登录 Tailscale：

```bash
tailscale up
```

执行后通常会输出一个登录链接，打开链接并使用账号完成认证即可。