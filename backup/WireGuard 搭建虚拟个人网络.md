# WireGuard 搭建虚拟个人网络

## 1. 安装 WireGuard

```bash
# 安装 WireGuard
apt install -y wireguard

# 开启流量转发
echo 'net.ipv4.ip_forward = 1' | tee -a /etc/sysctl.conf
sysctl -p
```

## 2. 配置目录权限

```bash
# 进入配置存储路径，调整目录权限
chmod 700 /etc/wireguard
cd /etc/wireguard
```

## 3. 生成密钥对

### 3.1 生成服务器密钥对

```bash
wg genkey | tee server.key | wg pubkey > server.key.pub
```

### 3.2 生成客户端密钥对

```bash
# 生成 client1 公钥私钥
wg genkey | tee client1.key | wg pubkey > client1.key.pub
```

## 4. 创建服务器配置文件

```bash
echo "
[Interface]
PrivateKey = $(cat server.key)
Address = 10.0.0.1/24
ListenPort = 51820

PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
# 注意：eth0 需要为本机物理网卡名称

[Peer]
PublicKey = $(cat client1.key.pub)
AllowedIPs = 10.0.0.2/32" > wg0.conf
```

## 5. 添加更多客户端

```bash
# 生成 client2 公钥私钥
wg genkey | tee client2.key | wg pubkey > client2.key.pub

# 追加到 wg0.conf 配置
echo "
[Peer]
PublicKey = $(cat client2.key.pub)
AllowedIPs = 10.0.0.3/32" >> wg0.conf
```

## 6. 启动和管理 WireGuard 服务

### 6.1 设置服务自启

```bash
systemctl enable wg-quick@wg0.service
```

### 6.2 启动/关闭服务

```bash
# 启动 wg0
wg-quick up wg0

# 关闭 wg0
wg-quick down wg0
```

## 7. 创建客户端配置文件

```bash
echo "
[Interface]
PrivateKey = $(cat client1.key)
Address = 10.0.0.2/24
DNS = 119.29.29.29

[Peer]
PublicKey = $(cat server.key.pub)
AllowedIPs = 10.0.0.0/24
Endpoint = 公网IP:51820
PersistentKeepalive = 30" > client1.conf
```